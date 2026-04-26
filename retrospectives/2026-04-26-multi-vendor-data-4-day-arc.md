# Multi-Vendor KR Equity Research — 4-Day Methodology Arc

> Sanitized public version of a 4-day manual-progression research arc on KR equities.
> The arc is preserved; signal-specific details are stripped.
> The discovery profile (5 PROMOTE provisional + 11 archived REJECTs + 7 validation-blocked
> scaffolds in 4 days, with PIT-correction haircuts) and 12 methodology lessons are the focus.
> Snapshot date: 2026-04-26.

## Scope

- Subject: a KR live-research repo, manual-progression mode (Day 1 → Day 4)
- Format: progress retrospective + methodology audit
- Companion sanitized doc: a parallel cross-sectional overnight 25-wave arc on a separate
  market venue (auto-progression mode). The two arcs converge on overlapping methodology
  lessons from different microstructures.

## Headline numbers (Day 4 close, PIT-corrected)

- **5 PROMOTE provisional**: a disclosure-text drift signal (long-only, mid-range haircut
  from a higher biased reading); an earlier disclosure-text drift variant (similarly
  promotable, smaller haircut); a sector-pair cointegration long-only signal (PIT-immune as
  a pair strategy); a price-extreme-drift signal at swing horizon (PIT re-validation
  pending); a foreign-investor flow momentum signal at swing horizon (broker-endpoint
  integration blocker + PIT re-validation pending).
- **1 tracked**: a preferred-common share spread signal (capacity-blocked by retail borrow
  constraints on the short leg).
- **11 archived REJECTs with explicit lesson**: directional gap-follow, open-fade
  mean-reversion, overnight-reversal, short-window reversal, sector-pair cointegration
  L/S, long-only preferred-common spread, v1 disclosure-text drift, two later L/S
  disclosure-text drift variants, peer-laggard daily-horizon, preliminary-earnings drift.
- **7 validation-blocked scaffolds**: minute-bar strategies (opening-range breakout,
  intraday momentum, VWAP reversion, closing imbalance, futures-lead, intraday
  oscillators) blocked on a broker-API limitation; unblock target ~60+ accumulated
  WS-forward-log sessions.

---

## §0. Foundation Evolution (Day 1 → Day 4)

The developmental arc of 4 manual-progression days, preserved as a step sequence.

### §0.1 Day 0 (pre-Day 1) baseline decisions
Standalone (duplication tolerated, isolation prioritized); no imports from external
core/platform packages. Skills installed: karpathy-guidelines, overnight-autonomy.
PreToolUse hook (block_live_writes) blocks any LLM write into live/, positions/, orders/,
.env.live. CLAUDE.md ~150 lines. shared/ self-implemented (types / validation / data_io /
risk). Every live-research repo starts the same way — standalone decision → shared/
4-module skeleton → market-specific fetcher entry.

### §0.2 Day 1 (overnight): top-cap universe + 8 daily-validatable strategies enumerated

Universe + infra: top-cap KR cap-weighted universe (~30 names, ~8 effective) cache-warmed;
shared/cache/daily.py (DuckDB) + shared/kis/{rest,ws,client,auth,token_store}.py on paper
account; ~270 trading days (a year); cost assumption a typical KR retail round-trip cost
band (CLAUDE.md-pinned).

Strategy catalogue (8 daily-validatable, IC + Sharpe gate):

```
Directional:
  - directional gap-follow         REJECT (wrong direction)
  - short-window reversal          REJECT (weak on cap-weighted top universe)
Mean-reversion:
  - open-fade                      REJECT (correct direction, cost-eaten)
  - overnight reversal             REJECT (weak signal)
Stat-arb:
  - preferred-common spread (L/S)  PROMOTE (needs short-leg borrow)
  - sector-pair cointegration L/S  REJECT (had been provisional PROMOTE on a shorter
                                            window — sample artifact)
Stat-arb long-only:
  - preferred-common spread LO     REJECT (~half PnL, cost-eaten)
  - sector-pair cointegration LO   PROMOTE (long-leg cheap-name in semis keeps the edge
                                            that L/S destroys)
```

→ 1-day enumeration: 4 REJECT + 1 PROMOTE (LS) + 1 PROMOTE (LO) + 2 LO sibling REJECT.
Discovery rate ~12.5% (1/8) on directional+LS, ~12.5% on LO. **Pattern**: cap-weighted
top-N showed insufficient cross-sectional power for ~8 effective names — KR's
retail-feasible top universe is too narrow → universe-expansion trigger.

### §0.3 Day 2: cointegration pair scan + mid-cap universe + tick infra
Coint pair scanner: ~7,700 raw Engle-Granger pairs over a 300-name candidate set,
Bonferroni at strict alpha → 2 survivors (a tech-sector pair, another sector-pair with
lumpier behavior). Mid-cap KR universe cache-warmed (~20 names). 4 archived REJECTs
re-tested on mid-cap: short-window reversal IC roughly doubled vs cap-weighted top
universe → reversal-to-momentum direction flip on mid-cap names → quantitative evidence
of cap-weight effect. shared/cache/ticks.py + a WS execution-tick logger launched
(one trading day × 6 symbols accumulated to start). Tick-to-minute aggregator scaffolded.
**Broker paper-account minute-endpoint current-day-only LIMITATION DISCOVERED**: the
minute-bar REST endpoint on paper host ignores cursor parameters → cannot fetch historical
minute. Two alternatives: live-account switch, or WS forward-log accumulation.

**Lesson surfaced**: the broker-API limitation validation-blocked 7 minute-bar scaffolds.
Chosen path: WS forward-log accumulation (preserves paper-forward verification rather than
swapping to live capital).

**Generalization**: trade-off between live-account switch cost vs forward-log accumulation
latency is worth documenting up-front for any broker-API limitation discovery.

### §0.4 Day 3: disclosure-API integration + signal evolution + overnight tier audit + collaborator brief

The largest day, 5 sub-phases.

```
§0.4.1 Disclosure OpenAPI integration: shared/kis/dart.py + shared/cache/disclosures.py.
  Batch warm: ~190K disclosures cached over a year-window in <1hr / ~2K API calls,
  state-cached in DuckDB.

§0.4.2 Disclosure-text drift signal evolution (v1 → v2 → v3):
  v1 REJECT (50-name universe missed ~70% of filing days → universe-expansion trigger);
  universe builder by disclosure-filing volume (300 names) → (Day 4 finding: a static
  "top by filing volume" snapshot constitutes survivorship bias, see Lesson 1);
  expanded daily-cache warm ~290 symbols / ~78K bars in <20 min;
  v2 PROMOTE provisional (50→300 universe, weighted small set of disclosure-event
  keywords [sales / equity issuance / treasury share / convertible / dividend], 3-day
  accumulation hold; mid-positive Sharpe net of default cost);
  v3 PROMOTE LO (keyword scheme simplified after v2 ablation; LS variant in low-positive
  range, long-only the best non-pair candidate pre-PIT).

§0.4.3 Tier 1+2+3 overnight audit (12-task manifest):
  - manifest.json + checkpoint.json discipline (12 tasks done overnight from
    a single launch). Tier 1 (fast), Tier 2 (deep), Tier 3 (exploratory).
  - Key outcomes: v2 long-only in promotable Sharpe range; **cost-cliff sweep**
    found a sharp Sharpe cliff at default+5bps that disqualifies promotion if
    real cost exceeds the assumption; period-stability quartile sweep showed
    one severely negative quartile; **keyword ablation found a single keyword
    in the keyword scheme carried virtually all the alpha — drop-one ablation
    collapsed Sharpe to near zero**; a separate keyword behaved as anti-signal.
    Pair walk-forward (15 × 30d × 14d): tech-sector pair survivor ~9/15 wins
    (most consistent), the other sector-pair survivor ~6/15 (lumpy), preferred-
    common spread ~7/15 (weaker than IS), sector-pair LO ~8/15. 4 archived
    REJECTs re-tested on 300-name universe — all stayed REJECT. Bug fix in
    cross-sectional intraday harness + regression tests. 22 symbols registered
    in daily forward-test; 2 new minute-bar scaffolds added (validation-blocked).

§0.4.4 Regime overlay battery (6 overlays × disclosure-drift v3 LO, 11 WF windows):
  baseline ~8/11 wins, mid-positive avg; low-vol-only ~1/5, near-zero, HURTS
  (dead days); high-dispersion-only ~5/6, high-positive, ROBUST; broad-market-only
  ~2/4, mid-positive (was IS double-digits — **textbook overfit detection**);
  trend-up-only ~7/8, high-positive, ROBUST; staged-reentry ~8/11, mid-positive,
  neutral. high-dispersion + trend-up intersection deferred for sample-shrinkage.

§0.4.5 External collaborator brief integration:
  8 ideas classified — in-scope (2): peer-laggard scalp, regime-change reentry;
  out-of-scope (6 cluster): swing momentum, macro overlay, HFT order-book, earnings
  event PEAD, themed-name baskets, crypto. Peer-laggard daily REJECT (deeply
  negative Sharpe, correct-direction IC, very high turnover) → **brief's own claim
  ('30-60s window edge, not daily') confirmed**. Staged-reentry overlay (per
  §0.4.4) was neutral.
```

**Pattern**: Day 3 ran 4 layers in parallel — external data-source integration; overnight
12-task checkpointed audit; regime overlay WF battery; external brief integration. Each
was a different-scope first-practice artifact.

### §0.5 Day 4: vendor-bundle integration + PIT loader + 4-agent fan-out

```
§0.5.1 Vendor-bundle integration: FnGuide's bulk CSV bundle (~50 GB
  across ~30 files) identified, 6-tier categorized; ~480 unique Item Names
  inventoried; vendor bundle migration tool ran 31 files / ~700s / 0 mismatches.
  cp949 + thousand-separator + dual-layout quirks documented (Lesson 2).

§0.5.2 PIT universe loader: a PIT cache covering ~770 symbols × ~22 months built
  from 3 core bundles. DuckDB pandas register('df', df) → C-level bulk insert
  ~6 min wall time (vs executemany ~250s+ → ~40x speedup). Symbol normalization
  across vendor / broker conventions. API: PIT-aware universe loader + PIT
  metadata getter. Cross-sectional harness gained a universe_at(yyyymmdd)
  callable parameter for per-date universe filtering.

§0.5.3 Disclosure-drift PIT re-validation: PIT-vs-original-universe overlap was
  ~54% (significant survivorship inflation in original). v2 LS REVERSE on PIT
  (DOWNGRADE → REJECT); v2 LO HOLD with minor haircut; v3 LS REJECT on PIT;
  v3 LO HOLD with mid-range haircut.

§0.5.4 4-agent parallel sub-agent fan-out (Phase 1):
  P0 (~30 min) PIT loader + disclosure-drift PIT re-validation; P1.A (~15 min)
  price-extreme-drift swing-horizon → provisional PROMOTE; P1.B (~15 min)
  foreign-flow momentum swing-horizon → provisional PROMOTE (broker investor-flow
  endpoint not yet wired, blocker noted); P1.C (~25 min) preliminary-earnings
  drift → REJECT (held longer than horizon rule allows). Sequential 1.5-3hr →
  parallel ~30 min = **~3.6x speedup**. Self-containment 7-element prompt
  template stabilized.

§0.5.5 Live-feasibility constraint as design criterion: "vendor bundle is
  backtest-only; live signal must be reconstructible from broker REST+WS +
  disclosure OpenAPI alone" — pinned in every sub-agent prompt. The PEAD signal
  was rejected on horizon grounds even after live-feasibility passed — design
  criteria layered (live-feasibility / horizon / cost in that order).
```

Day 1-3 manual → Day 4 multi-agent fan-out (4 strategies/session). **Next step is
auto-progression-eligible** — manifest+checkpoint infra in place since Day 3.

### §0.6 Cumulative state at Day 4 close
Promotable 5 (provisional); tracked 1 (capacity-blocked); archived REJECTs 11 (each with
explicit lesson); validation-blocked scaffolds 7 (minute-bar). Cron: daily forward-test
runner (post-close). shared/ modules: 7 self-implemented, 0 external deps. Hypothesis
catalogues: 3 (original CATALOGUE + collaborator brief + vendor 6-tier categorization).
WF harness per-strategy ad-hoc (standardization candidate); permutation harness ad-hoc
per-script (backport candidate from companion repo). Tests: smoke + regression. Vendor
data ~30 bundles / ~50 GB ingested, ~360k-row PIT cache. Disclosure data ~190k cached.

### §0.7 Manual → auto-progression transition

Day 4 fan-out is the first auto-style wave. From Day 5+ a `commit → next-batch` loop is
feasible without explicit wave-trigger commands. Transition brake has 2 components:
(1) atomic commit per phase as interrupt-recovery anchor; (2) project-specific memory
notes (PIT universe, broker account, etc.) for immediate session-resumption.

---

## §1. Principles checklist (audit)

- [x] CLAUDE.md ≤ 200 lines (~150)
- [x] All rules verifiable (absolute rules + R→L 5-gate checklist)
- [x] Standalone declared (no external core/platform imports)
- [x] MCP list rationale documented (Filesystem + DuckDB only; broker MCP banned)
- [x] Sub-agent usage policy defined (main = orchestration; fan-out first applied Day 4)
- [x] Parallelism enforced; live-mode serial path declared
- [x] Overnight resume protocol defined (manifest + checkpoint, first proven Day 3)
- [x] karpathy-guidelines + overnight-autonomy skills installed; deny rules reflected
- [x] Reference-validation checklist passed per signal (ADF + KPSS + IC-PnL gap + Sharpe
      ≥ promotion threshold)
- [ ] **drift-watchdog skill not installed** — see §6.1
- [ ] **abandon-criteria skill not installed** — see §6.2 (R→L 5-gate covers part)
- [x] LLM role boundary declared (research/ free; live/ PreToolUse hook blocks writes)
- [x] /memory loaded; auto-memory vs manual-CLAUDE.md separation maintained

---

## §2. 12 Lessons (transferable methodology)

### Lesson 1: LS-variant survivorship-bias inflation — KR retail short-blocking as accidental immunity

**Phenomenon**: ~54% of the original universe was survivor-selected. On PIT re-validation,
the L/S variant of the disclosure-text drift signal **reversed sign**, while the long-only
variant absorbed only a minor haircut.

**Mechanism**: the biased universe drops truly distressed names (delisting-bound,
admin-flagged) and keeps "quietly-large surviving names" — these populated the bottom-decile
short leg of the L/S sort and produced what *looked* like alpha. The KR retail short-blocked
microstructure forces all retail strategies to long-only, which *accidentally* immunizes
them against this survivorship contamination.

**Action**: every cross-sectional candidate is reported in *both* long-only and L/S
variants. **If LS Sharpe > LO Sharpe, that asymmetry is a survivorship-bias trigger**:
in a clean sample, LS should not exceed LO meaningfully (LS includes LO's long leg plus a
short leg with extra cost). LS > LO points to spurious alpha on the short leg. Short-enabled
markets do NOT inherit this accidental immunity → LS bias appears normally → independent
PIT-universe validation required. PIT universe + LS-vs-LO asymmetry check should be
mandatory in every cross-sectional factor pipeline.

### Lesson 2: cp949 + thousand-separator + dual-layout vendor-data quirks

FnGuide's bulk CSV bundles uniformly: (1) cp949 encoding (not UTF-8);
(2) price values quoted as strings with thousand-separator commas; (3) two layouts coexist
(a single-Item wide format + a Bundle CSV long-format with Item Name in a fixed column);
(4) single-Item files cover only major-index constituents — using them quietly re-introduces
a universe survivorship problem.

**Action**: pandas `read_csv(encoding='cp949')` then explicit string-strip of comma
separators and cast to float. Bundle layout enforced. The price-extreme-drift sub-agent
discovered the issue when its initial integer comparison silently failed. Other KR vendors
exhibit the same quirks frequently. A KR-vendor-data CLAUDE.md template should standardize
cp949 + thousand-separator + dual-layout handling.

### Lesson 3: Python `signal.py` stdlib collision in research/<slug>/

A sub-agent wrote `research/<numeric-prefixed-slug>/signal.py`. When `run.py` adds the
strategy folder to `sys.path`, stdlib `signal` gets shadowed. Folder names starting with
a digit cannot be imported as a package either. Action: rename to a non-shadowing module
name (e.g. `signal_<short>.py`) and use `importlib.spec_from_file_location`. Generalization:
every research/<slug>/ pattern repo should add a naming rule — no stdlib-shadowing module
names + no numeric-prefixed folder names.

### Lesson 4: Live-feasibility constraint as design criterion (vendor-backtest-only)

Bulk vendor exports are batch-only with no real-time feed → unusable for live signal
computation. Live signals must be reconstructible from broker REST+WS + disclosure
OpenAPI alone. This was pinned in every sub-agent prompt → every PROMOTE candidate
documents backtest-data vs live-data sources separately.

**Action**: a Live-feasibility section is mandatory in every research/<slug>/VERDICT.md:
exact broker endpoint(s), sample call + response schema, latency (intraday / T+1),
confirmation that live-derived signal can match backtest signal value. Vendor-data-dependent
repos all benefit; real-time exchange-feed projects have a smaller gap, but the layering
pattern still holds.

### Lesson 5: Multi-agent fan-out prompt self-containment — 7 elements

The four Day-4 sub-agents started with zero conversation context yet produced
uniform-quality artifacts (~3.6x speedup). The brake is prompt self-containment.

**7 elements**: (1) project absolute path; (2) data file absolute path + exact column /
Item Name; (3) data layout (encoding, header row, key-column index); (4) existing-pattern
referent (imitate research/<existing-slug>/); (5) deliverable spec (signal module + run
script + NOTES.md + VERDICT.md); (6) live-feasibility constraint; (7) CLAUDE.md read
mandate.

Cost: minor naming inconsistencies across sub-agent outputs — acceptable trade-off for the
speedup. A separate runbook formalizes this template; this lesson kept short for
cross-reference.

### Lesson 6: ADF + KPSS combination for spurious-stationarity gating

The harness's gate combines ADF (null = unit root) and KPSS (null = stationarity). The
hypotheses point opposite ways, so simultaneous ADF-reject + KPSS-fail-to-reject is the
robust conclusion; either alone admits false positives. Mandatory for every IC-time-series
spurious check (Hamilton 1994). Standard for any cross-sectional factor pipeline.

### Lesson 7: Cost-cliff sweep — a typical KR retail cost band, with a sharp Sharpe cliff at default+5bps

Disclosure-drift v2 cost-sensitivity sweep showed a sharp Sharpe cliff such that a 5-bp
cost increase moved the strategy from solidly-promotable to disqualifying. The default
cost assumption is a typical KR retail round-trip band; the cliff sits within range of
plausible execution-cost variance. Action: every PROMOTE candidate gets a ±5 bp cost
sweep + paper-forward measured-cost verification. KR retail cost band decomposes as
bid-ask + transaction tax (sell-side only, dominant) + fees + slippage. Crypto retail
taker cost is closer to the cliff than KR — same cost-aware gate required. Live-research
CLAUDE.md template should include the cost-sweep mandate.

### Lesson 8: Keyword ablation methodology — one feature carried the entire alpha

The disclosure-text drift signal used a small set of weighted disclosure-event keywords
(sales / equity issuance / treasury share / convertible / dividend). Drop-one ablation:
**a single keyword in the keyword scheme carried virtually all the alpha — drop-one
ablation collapsed Sharpe to near zero**. A separate keyword behaved as anti-signal
(dropping it *lifted* Sharpe). After simplifying to a smaller keyword set in v3, long-only
Sharpe improved noticeably. Action: every multi-feature signal gets keyword/feature
ablation; dual purpose — detect single-feature dependence (fragility) + remove
anti-signal features. Generalization: multi-gate signals benefit from per-gate ablation;
factor-replication studies treat each factor as the ablation unit; LLM-mediated feature
pipelines should run per-feature ablations to detect dead weight.

### Lesson 9: Regime overlay walk-forward exposes textbook in-sample overfit

6 overlays × disclosure-drift v3 LO over 11 WF windows. One overlay's IS Sharpe was deep
double-digit territory but collapsed substantially on walk-forward — **textbook overfit
signature**. A low-vol-only filter actively HURT performance (counter-intuitive — the
alpha lives on high-information days, which the low-vol filter excludes). Two overlays
(high-dispersion-only, trend-up-only) were WF-robust. Every overlay-rescue must pass
N-fold WF before entering the promotion track; IS double-digit Sharpe values are a
selection-bias signature, never trusted without WF.

### Lesson 10: External-collaborator brief integration — 4-layer framework

An external collaborator's idea brief was processed in 4 layers: (1) CLAUDE.md
horizon-scope rule as hard first-pass filter (in-scope vs out-of-scope clusters);
(2) in-scope ideas implemented and tested; (3) honest reporting back to brief's own
claims (one idea's daily-horizon implementation REJECTed, *confirming the brief author's
own claim that the edge lives on sub-minute horizon, not daily*); (4) out-of-scope
clusters logged as candidate future-repo branches rather than crammed into current scope.
Any multi-stakeholder collaboration repo benefits from a standardized external-input
critical-evaluation framework.

### Lesson 11: Broker paper-account minute-history limitation

A KR retail broker's paper-account API has a current-day-only limit on minute-bar
history — cursor parameters silently ignored on paper host. Cannot fetch 30+ days of
minute history; 7 minute-bar strategy scaffolds validation-blocked as a result.

Two alternative paths: (1) switch to live-account (explicit constant change, but loses
paper-forward verification safety); (2) WS forward-log accumulation (a daily WS
execution-tick logger runs market-hours, minute-bar strategies unblocking ~60+ accumulated
sessions later). Day 2 discovery, immediate pivot to forward-log path. Daily-resolution
strategies paper-forward in parallel meanwhile. Live-research CLAUDE.md templates should
mandate a "broker API limitation discovery + alternative path" section as standard.

### Lesson 12: PIT universe API + universe_at callable integration

The cross-sectional research lib gained a `universe_at(yyyymmdd) -> list[symbol]` callable
parameter. Per-date universe filtering applies PIT correction automatically — no need to
remember to filter at every call site. A PIT-aware universe loader provides the default
implementation. Naturally-PIT venues (dynamic top-N-by-volume) still benefit from an
explicit `universe_at` API for symmetry and documentation. Cross-sectional pipelines
should standardize on the interface.

---

## §3. Infrastructure outputs (reusable)

### shared/ modules (self-implemented; portable to other repos)

- `shared/data_io/<vendor>_pit.py` (Day 4): DuckDB-backed PIT cache; APIs
  `pit_universe_top_n(yyyymmdd, n, ...)` and `pit_metadata(yyyymmdd, symbol)`.
- `shared/cache/` (Day 1-3): DuckDB-backed `daily / minute / ticks / disclosures` (each
  paired with the corresponding broker-REST or vendor-API source).
- `shared/kis/` (Day 1): broker access — `rest / ws / client` (env=paper|live, token
  refresh + retry) / `dart` (Day 3) / `auth + token_store` (24h refresh + file-lock).
- `shared/risk.py` (Day 0): `RiskLimits` (Frozen dataclass; env-var override forbidden);
  Day 5 candidates `ExchangeOutageGuard`, `DrawdownLadder`.
- `research/lib.py`: `run_cross_sectional(..., universe_at=None)` (Day 4 extension).
- `research/_harness.py`: `_gate(...)` → ADF + KPSS + IC-PnL gap + Sharpe.
- `research/overlay.py`: 6 overlays.

**Reuse**: PIT API is vendor-agnostic; minute-bar / tick infra portable across markets
with data source swapped; factor-replication repos using vendor-cleaned PIT need no
additional work.

### Cron infrastructure (Day 3)
Daily forward-test runner (post-close, paper-forward z-score + position for the 5
PROMOTE candidates + 4 pair trackers); WS execution-tick logger (market-hours, forward-log
accumulation); tick-to-minute aggregator (post-close).

**Pending cron candidates (Day 5+)**: vendor quarterly refresh; disclosure incremental
warm (currently batch); broker investor-flow daily logging (after endpoint blocker wired).

### Manifest+checkpoint workflow (Day 3 Tier audit)
A `manifest.json` defines all tasks; `checkpoint.json` carries incremental per-task state
(pending/in_progress/done/aborted + intermediate artifacts + resume cursor). Robust against
external-dependency interruptions (LLM API hiccups, host sleep). Atomic commit per task →
last commit is the recovery anchor.

---

## §4. Open risks / next-action candidates

### Time-dependent (cannot accelerate)
Vendor quarterly refresh closes a 6-month PIT-data gap; WS-tick-infra ~60+ session
accumulation (~3 months out) unblocks 7 minute-bar scaffolds; paper-forward 30+ days
accumulation gives backtest-vs-live Sharpe agreement check for the 5 PROMOTE candidates.

### Reopen-pending (next-method declared)
Preliminary-earnings PEAD signal candidate for a separate swing-horizon repo branch
(20d hold violates current horizon rule); foreign-flow signal awaits broker investor-flow
endpoint wiring; price-extreme-drift + foreign-flow signals still need PIT re-validation
(loader exists, only re-runs needed); high-dispersion + trend-up overlay intersection
deferred for sample-shrinkage concern.

### Operator action (LLM out)
After paper-forward 30+ days, select 1-2 PROMOTE candidates for live-account migration.
R→L promotion checklist has 3 unmet items: kill switch, hardcoded position-limit
constants, open/close transition scenario tests. live/ entry (LLM write-blocked, operator
direct): order path + position reconciliation + emergency shutdown procedure.

---

## §5. Audit conclusions

1. **CLAUDE.md absolute rules: 0 violations across 4 days**. Horizon rule fired correctly
   on the preliminary-earnings PEAD self-REJECT. PreToolUse hook performed as designed.
   live/ remained LLM-untouched.
2. **Standalone principle preserved**: 0 imports from external core/platform packages;
   shared/ fully self-implemented (7 modules).
3. **Parallel-mode discipline (R-mode)**: Day 4 4-agent fan-out was the first live-fire
   test. Live-mode serial path not yet engaged (still in paper-forward).
4. **Atomic-commit-per-phase pattern** standardized in Day 3 overnight audit.
5. **Karpathy-guidelines + overnight-autonomy skills** both installed.
6. **Validation-gate discipline**: all 13 strategies passed the 4-axis gate. Each of the
   11 REJECTs preserved with explicit lesson.
7. **Multi-vendor data integration**: 4 sources unified (broker REST+WS, disclosure API,
   vendor bundle, PIT cache); multi-resolution cache standardized on DuckDB.
8. **Honest abandon criteria**: the preliminary-earnings PEAD self-REJECT on horizon
   grounds, the foreign-flow blocker documented as a gap (not papered over), and the
   peer-laggard daily REJECT confirming the brief author's own claim — all evidence of
   sub-agent self-discipline.

---

## §6. Recommended actions

### §6.1 Immediate — install drift-watchdog skill
The 5 PROMOTE candidates can stale silently without WF re-validation (especially the
disclosure-drift v2's cost-cliff fragility and its single-keyword dependency). Install
drift-watchdog from the skill registry into the live-research template.

### §6.2 Immediate — install abandon-criteria skill + declare ≥3 kill triggers
Without explicit abandon criteria, reopen-pending items risk perpetual holding. R→L
5-gate covers only part of this role. Three suggested kill triggers: (1) disclosure-drift
v2 — paper-forward measured cost > default+2bps AND Sharpe < 0.3 → close; (2)
price-extreme-drift signal — post-PIT Sharpe < promotion threshold → close; (3)
foreign-flow signal — 6 months without broker investor-flow endpoint wired → close.

### §6.3 Short-term — propagate lessons across category
PIT-API-portability check across vendors (Lesson 1, 12); LS-bias-check audit for
short-enabled markets (Lesson 1, where accidental immunity does NOT exist); vendor-quirk
template (Lesson 2); multi-feature ablation policy (Lesson 8); broker-API limitation +
alternative-path documentation standard (Lesson 11).

### §6.4 Short-term — repo-status registry update
Update the workflow archive target registry: status `planned` → `active`, last-audit date,
promotable count, archived-REJECT count, validation-blocked-scaffold count.

### §6.5 Short-term — cross-arc lesson backport
From the companion arc: explicit N-fold WF gate (Day-3 broad-market-only collapse is the
first KR-side empirical evidence); "first IC read on < 6 months sample → 1.5x-2x extension
before promotion gate" (Day-1 sector-pair short-window-vs-full reversal is the first
KR-side evidence); block-permutation primary test for daily/swing signals; atomic commit
+ trash-bin pattern (already partial; complete adoption).

### §6.6 Long-term — template strengthening
Live-research CLAUDE.md template additions: promotion checklist 7-gate (5-gate + WF
sign-stability + block-perm); shared/ standard-module set (data_io, cache×4, broker,
risk, validation, perm_test, walkforward); Live-feasibility constraint (Lesson 4);
multi-agent fan-out 7-element prompt (Lesson 5); vendor-data quirk handling (Lesson 2).

---

## §7. Directly applicable patches (extracted)

### Patch 1: PIT universe build pattern (Lesson 12)
**Target**: every cross-sectional research repo
**Diff**:
```diff
+ shared/data_io/<vendor>_pit.py
+   class <Vendor>PITCache(db_path):
+       def pit_universe_top_n(yyyymmdd, n, exclude_<flag>=True) -> list[symbol]
+       def pit_metadata(yyyymmdd, symbol) -> dict
+
  research/lib.py::run_cross_sectional(
      panel, score_factory, cost_bps, min_symbols, long_short,
+     universe_at=None  # callable(yyyymmdd) -> list[symbol]
  )
```

### Patch 2: LS-variant survivorship-bias detection rule (Lesson 1)
**Target**: every short-blocked retail-market repo (asymmetric markets specifically)
**Rule**:
```
For every cross-sectional candidate, report both long-only and L/S variants:
  if abs(LS_Sharpe) > LO_Sharpe:
    flag "POSSIBLE SURVIVORSHIP BIAS — investigate short leg"
    require PIT universe re-validation before PROMOTE
```
24/7 short-enabled markets are not subject to this asymmetry — the rule is specific to
single-side-tradable markets.

### Patch 3: Multi-agent fan-out 7-element prompt template (Lesson 5)
**Target**: every long-running research repo using sub-agent fan-out. Template = the
7 elements from Lesson 5.

### Patch 4: Live-feasibility constraint as design criterion (Lesson 4)
**Target**: every vendor-data-dependent research repo
**Diff** (`research/<slug>/VERDICT.md` template):
```diff
+ ## Live-feasibility section (mandatory)
+ - Backtest data: <vendor> <Item Name>
+ - Live data: <broker_or_realtime_api> <endpoint_name>
+ - Sample call: <example>
+ - Latency: intraday / T+1 / T+N
+ - Confirmation: live signal == backtest signal? (yes / no / gap_documented)
```

### Patch 5: cp949 + thousand-separator vendor parsing (Lesson 2)
**Target**: every KR vendor-data integration
**Code snippet**:
```python
# Standard KR vendor CSV parsing
df = pd.read_csv(path, encoding='cp949', header=<vendor-specific row>)
for col in df.columns[<date-col-start>:]:
    if df[col].dtype == 'object':
        df[col] = df[col].str.replace(',', '').replace('-', '').astype(float)
# Bundle CSV: long-format, Item Name in a fixed column
df['item'] = df.iloc[:, <item-col-index>].str.strip()  # trailing spaces matter
```

### Patch 6: Cost-cliff sweep mandate (Lesson 7)
**Target**: every PROMOTE candidate's promotion checklist. **Rule**: ±5 bp cost sweep
before promotion; if a Sharpe cliff falling below the promotion threshold sits within
±5 bp of the default assumption, the candidate is NOT promotion-eligible. Paper-forward
measured-cost verification required before final decision.

---

## §8. Meta observations

The most important architectural insights from this 4-day arc:

**"Survivorship bias concentrates on the short side — a short-blocked retail microstructure
acts as accidental immunity."** This is the audit's largest standalone finding. On PIT
re-validation the L/S variants near-uniformly reverse, while long-only variants take
only minor haircuts. A market microstructure constraint that *looks* like a weakness
(no shorting) becomes a strength (no spurious short-leg alpha). This is a
KR-microstructure-specific asymmetry; markets with symmetric short access do not inherit it.

**"The brake on multi-agent fan-out is prompt self-containment."** Four sub-agents started
with zero conversation context yet produced uniform-quality artifacts. The 7-element
prompt template is the brake. Two data points (the 25-wave companion arc + this 4-day
arc) are already enough to call the pattern reproducible. Strong candidate for the
workflow archive template registry.

**"Methodology lessons travel further than signals."** This repo's 5 PROMOTE candidates
do not transfer to other markets directly. The 12 lessons in §2, however, transfer at the
mechanism level. The same conclusion was reached independently in the companion arc's
meta observations — **the real value of a workflow archive is in the methodology audit
catalogue, not the signal catalogue.**

**"Manual → auto-progression transition is an infrastructure-driven event."** Day 1-3
manual progression → Day 4 4-agent fan-out → Day 5+ auto-progression-eligible (the
manifest+checkpoint infra was put in place on Day 3). This timing matches the companion
arc's analogous transition — foundation phase ~14-25 atomic steps before automation
becomes safe. Automation is not enabled by ambition; it is enabled by the recovery-anchor
infrastructure being in place.

---
