# Signal R-to-L Promotion Gate Runbook

Methodology runbook for promoting a research-stage signal into a live-trading path. Project-agnostic; adapted from internal practice across multiple research repos (cross-sectional crypto, KR equities long-only, factor replication, sentiment-LLM, HFT/LOB).

A signal that survives an exploratory backtest is not yet eligible for capital. This document defines the standard gate sequence every candidate must pass before it can move from `research/` into a live execution path. Each gate is scored as a binary yes/no. A single failure halts promotion; the candidate is archived with an explicit reopen criterion.

Companion runbooks: see `multi-agent-fanout-runbook.md` for parallel signal research, and `standalone-live-research-bootstrap.md` for the surrounding repo scaffolding. The retrospectives in `../retrospectives/` document real cases where these gates fired (or failed to fire).

## 1. The 7-gate standard checklist

| # | Gate | Tool / function | Pass criterion |
|---|---|---|---|
| 1 | IC spurious (ADF + KPSS) | `shared/validation.py::ic_spurious_check` | ADF rejects + KPSS fails to reject |
| 2 | Turnover sanity | same function | turnover != 0; intraday turnover < intended horizon |
| 3 | Block24-perm | `shared/perm_test.py::permute_block(block_size=24)` | p < 0.05 (Bonferroni alpha/cells when multiple cells) |
| 4 | Walk-forward sign-stability | `shared/walkforward.py::walk_forward_eval` | net Sharpe lift same sign in **>= 3/4 folds** |
| 5 | Cost-cliff sweep | ad-hoc (per-strategy) | no cliff inside default cost +/- 5 bps |
| 6 | Short-window extension | per-strategy | sign and perm-significance survive at 1.5x-2x the original IC sample window |
| 7 | LS-vs-LO asymmetry | per-strategy | `|LS Sharpe| <= LO Sharpe` (single-side markets only) |

**Important**: gates 1-3 alone catch only the most obvious failures. Gates 4-7 are where the bulk of in-sample lottery, overfit, and survivorship-bias cases get filtered. Do not promote on gates 1-3 alone.

## 2. Gate 1 — ADF + KPSS combined (stationarity)

ADF (null = unit root) and KPSS (null = stationary) test in opposite directions. Combining them avoids one-sided false-positive bias.

```python
adf_p = adfuller(ic_series)[1]      # null: non-stationary
kpss_p = kpss(ic_series)[1]          # null: stationary
robust_stationary = (adf_p < 0.05) and (kpss_p > 0.05)
```

ADF alone has weak small-sample power; KPSS alone over-rejects in large samples. The combination is the Hamilton (1994) standard.

## 3. Gate 3 — Block24-perm vs standard-perm

For overlapping forward-return windows (e.g. 24h forward IC), the lag-1 autocorrelation in returns inflates the standard permutation null tail. The pattern is robustly reproduced: standard-perm reports p = 0.001 PASS while block24-perm on the same data reports p = 0.06 to 0.30 FAIL.

**Rule**: any signal with horizon >= 4h uses **block24-perm as primary**. Standard-perm is retained only as a sanity comparison.

```python
def ic_null_distribution(ic, signal, fwd, scheme="block24", n_iter=1000):
    if scheme == "block24":
        return permute_block(ic, signal, fwd, block_size=24, n_iter=n_iter)
    elif scheme == "standard":
        return permute_standard(ic, signal, fwd, n_iter=n_iter)
```

LOB / sub-second signals are largely unaffected. This gate matters for daily and swing horizons.

## 4. Gate 4 — Walk-forward sign-stability

The single most reliable filter for in-sample lottery wins. Procedure:

- 4-fold (or 11-window) walk-forward
- compute per-fold net Sharpe lift
- PASS = >= 3/4 folds (or >= 75% of windows) same sign and > 0
- FAIL = a single fold carries the bulk of the lift (in-sample lottery)

**Separate failure mode — overlay zero-out**: if a drawdown-conditional sizing overlay collapses to overlay-size <= 0.02 within most folds, the signal is DEGENERATE — full-window aggregate looks fine but the overlay is doing nothing. Inspect overlay statistics per-fold, not just aggregate.

## 5. Gate 5 — Cost-cliff sweep

A representative single-region cost sensitivity (illustrative numbers, not from any specific deployed signal):

| cost (bps RT) | Sharpe |
|---|---|
| 15 | +1.12 |
| 20 (default) | +0.58 |
| 22 | +0.32 |
| **25** | **+0.04** ← cliff |
| 30 | -0.45 |

A 5-bps shift can flip a candidate from PROMOTE to REJECT. **Mandate a default +/- 5 bps cost sweep on every PROMOTE candidate** plus paper-forward measurement of realized cost.

Decompose the regional default into spread + transaction tax + commission + slippage so you can update the sweep when any component changes. Crypto VIP-0 retail bands sit closer to the cliff than typical institutional equity bands — be more conservative there.

## 6. Gate 6 — Short-window extension

The most consistent single failure: a signal first observed on a < 6 month sample collapses or sign-flips when the window is extended 1.5x-2x. This pattern has reproduced across volume, flow, volatility, and pair-cointegration signal families.

**Rule**: "the first IC read on a < 6 month sample is an **upper bound**. A 1.5x-2x window extension is **mandatory** before any promotion-gate evaluation."

If extension produces sign-flip, magnitude collapse, or perm-fail, archive immediately with a window-condition reopen criterion.

See also the cross-sectional retrospective in `../retrospectives/` for a multi-wave case study where this gate fired repeatedly.

## 7. Gate 7 — LS-vs-LO asymmetry (single-side markets only)

In markets where retail short-selling is restricted, a long-only universe is enforced by structure. This restriction creates a *coincidental immunity* to one common survivorship-bias pattern: when point-in-time universe construction is later applied, **L/S variants of biased signals often reverse entirely**, while long-only variants take only a 5-17% haircut.

**Mechanism**: a non-PIT universe drops the genuinely weak names (delisted, watch-listed) and retains "quiet large-cap survivors". The short leg therefore appears to generate alpha that is in fact survivorship.

**Rule**:
```
For every cross-sectional candidate, compute long-only and L/S in parallel:
  if abs(LS_Sharpe) > LO_Sharpe:
    flag "POSSIBLE SURVIVORSHIP BIAS — investigate short leg"
    require PIT universe re-validation before PROMOTE
```

24/7 two-sided markets (e.g. crypto) are not subject to this gate. Apply only in markets with structural single-side bias.

## 8. IC-PnL gap diagnostic (auxiliary)

`shared/validation.py::pnl_ic_gap(ic, pnl, cost_bps)` computes a break-even cost given trading-cost and frequency assumptions. A signal can pass IC-significance and still fail on PnL once costs are charged. This is a diagnostic, not an eighth gate — but a candidate that fails it will fail Gate 5.

## 9. Audit checklist (target project)

When auditing a project against this runbook:

- [ ] CLAUDE.md §5 promotion checklist enumerates all 7 gates?
- [ ] `shared/validation.py::ic_spurious_check` implements ADF+KPSS combined?
- [ ] `shared/perm_test.py` defaults to block24 via dispatcher?
- [ ] `shared/walkforward.py::walk_forward_eval` documents the >= 3/4 fold sign-stability threshold?
- [ ] Every PROMOTE candidate's VERDICT.md contains a +/- 5 bps cost sweep table?
- [ ] First-discovery short-window signals are extension-validated (1.5x-2x) before promotion?
- [ ] Single-side-market repos include the LS-vs-LO asymmetry check?

## 10. Application matrix (by repo category)

The same 7 gates are not all relevant to every research style. Generic guidance by category:

| Repo category | Gates 1-3 | Gate 4 (WF sign) | Gate 5 (cost cliff) | Gate 6 (short-window) | Gate 7 (LS vs LO) |
|---|---|---|---|---|---|
| Crypto live-research (24/7, two-sided) | applies (standardized in foundation phase) | applies | applies (cost-aware design) | applies (multiple reproductions) | **N/A** (24/7 two-sided) |
| KR equities live-research (single-side) | applies | recommended addition | applies | recommended addition | **applies** (key gate for this market) |
| Factor-replication audit | applies | applies | applies (capacity-aware) | recommended addition | typically skip (PIT-clean source) |
| Cross-sectional factor platform | standard | standard | standard | standard | branch by market |
| Sentiment-LLM signal | review | review | review | **mandatory** (short-sample risk) | n/a unless cross-sectional |
| HFT / LOB strategy | applies | applies | applies | n/a (sub-second horizon) | n/a |

Use this matrix to prioritize gate work when bringing a new project into compliance — adopt all gates eventually, but in the order suggested by category.
