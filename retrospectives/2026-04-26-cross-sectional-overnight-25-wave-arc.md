# Cross-Sectional Overnight Research — 25-Wave Methodology Arc

> This is a sanitized public version of a 25-wave overnight research progression
> originally captured as an internal audit. The developmental arc is preserved;
> signal-specific details (names, exact IC/Sharpe values, thresholds, universe
> compositions, broker / endpoint / cron specifics) are stripped. The discovery
> rate (~15%, 28 archived nulls + 5 promotable candidates across 25 waves) and
> the methodology lessons are the focus — they are what generalizes across
> projects.

## Snapshot

Source project: a crypto live-research repo (private). Window: 4 days, 25 waves,
35 atomic commits. Pattern: 4-agent parallel fan-out per wave (~100 agent-runs
total). Tests: 165 → 409 (+244, 0 failures). 50+ signal modules / 40+
backtests / 50+ markdown reports.

**Promotion ledger at end of Wave 25**:
- **Lead R→L candidate (1)**: a directional signal in the cross-sectional alt
  universe with a regime overlay.
- **Tracked candidates (3)**: an implied-vs-realized vol signal with regime
  overlay; a session-event signal (two horizon variants); a funding-rate
  cross-sectional signal with a stepwise drawdown-conditional sizing schedule.
- **Archived nulls (28 rows)**: each carries an explicit reopen criterion.
- **In-flight infrastructure (4)**: an event-prediction-market gap pipeline,
  an on-chain whale-flow pipeline, an options-vol path (cron-data-accumulation
  gated), and a liquidation-data pipeline (paid feed required).

The ratio (28 archived : 5 promotable) is the headline discovery-rate signal of
the arc — it is the value the rest of this document builds on.

---

## §0. Foundation Evolution (Phase 0 → Wave 14)

This section reconstructs the manual progression that preceded the auto-wave
mode (Wave 16+). Other live-research repos are likely to follow a very similar
developmental curve, so it is preserved as reference.

### §0.1 Phase 0–3: baseline decisions
- **Phase 0**: standalone scoping decision (no shared imports from other
  in-house quant packages; duplication tolerated, isolation prioritized).
  Two skills installed (karpathy-guidelines + an overnight-autonomy skill).
  A PreToolUse hook blocking writes to live-trading paths wired in.
  CLAUDE.md kept under the 200-line ceiling.
- **Phase 1**: shared/ utilities — 4-module pattern locked in
  (types / validation / data_io / risk).
- **Phase 2-3**: two CEX/derivatives public-data fetchers stood up; a
  9-hypothesis catalog written as the research-artifact starting point.

**Generalization**: every live-research repo we have reviewed follows the same
pattern — standalone decision → shared/ 4-module utilities → market-specific
fetchers → hypothesis catalog. New repos can mirror this.

### §0.2 Wave 1–2: first promotable
- **Wave 1**: IC-spurious-gate sweep across 9 hypotheses. Most cells below the
  spurious-IC band; one directional ratio signal showed a strong negative IC
  at the lead horizon.
- **Wave 2** (4-agent parallel): one gated cell of the directional ratio
  emerged with strongly positive net Sharpe in the promote band, surviving
  the typical retail-tier cost band that proved tight. This became the first
  lead R→L candidate and has remained the lead through 25 waves. Three
  alternative hypotheses archived as cost-eaten.

**Pattern**: "9 hypotheses in one sweep → 1–2 promotable + many archives" —
typical first-wave outcome, in line with a ~10–20% discovery rate.

### §0.3 Wave 3–5: walk-forward harness + cloud cron
- **Wave 3** (4-agent parallel): first walk-forward harness, first
  permutation-null harness (both per-script ad-hoc, standardized later);
  engine extracted into a research/ directory; paper-trading scaffold.
- **Wave 4** (4-agent parallel): testnet platform PoCs; a market-state filter
  (regime gate) added — became the core gate for the lead candidate; cost
  sensitivity analysis (taker vs maker).
- **Wave 5**: cloud cron registration (hourly open-interest snapshot on cloud
  VPS, event-prediction-market snapshot at multi-hour cadence, weekly WF
  rerun).

**Key insight**: without cron registered in Wave 3–5, the lead candidate would
not have accumulated enough data to clear the WF gate by Wave 25. Cron
registration immediately after first promotable discovery is the highest-ROI
infrastructure step.

### §0.4 Wave 6–8: paper trading + first short-window collapse
- **Wave 6-7**: paper v0 testnet smoke loop; an event-prediction-market
  convergence pilot archived (wrong-sign + KPSS-spurious null); a sibling repo
  split off when a daily-horizon hypothesis violated the repo's intraday-only
  invariant.
- **Wave 7.5–7.7**: a volume-momentum cell at a multi-day horizon entered the
  promotable IC band; qualified as a second promotable candidate.
- **Wave 8**: same cell at extended window — IC shrank ~2.4x, block-perm failed
  Bonferroni; walk-forward 1 of 4 folds carried sign. **First short-window
  collapse confirmed.** Reproduced three more times later in the arc.

### §0.5 Wave 9–13: rescue attempt + cloud migration
- **Wave 9**: regime gate added to the volume-momentum cell; partial
  single-fold rescue; WF sign-stability still failed; demoted.
- **Wave 10**: a new on-chain CEX-flow pilot started. An external on-chain
  transaction API V2 migration was forced by V1 deprecation, with a page-cap
  workaround via block-range bisection to stay in free tier. Short-sample IC at
  the lead horizon was negative and significant — flagged.
- **Wave 11**: cloud-deploy handoff (SSH key + operator-side cron migration).
- **Wave 12**: satellite hypothesis catalog for an adjacent project; 3 of 5
  hypotheses shelved on prior-art grounds; one macro pilot carried forward.
- **Wave 13**: CEX-flow pilot at extended window — honest null. **Second
  short-window collapse confirmed.**

**Generalization**: V1 deprecation events on external data APIs are inevitable
— build in V2 fallbacks or periodic health checks.

### §0.6 Wave 14: the larger parallel wave — funding cross-sectional discovery
- **Wave 14** (4 parallel): wallet universe expanded for the on-chain CEX-flow
  pilot; a tier-2 chain pilot started; macro pilot scaffold using a mid-tier
  open LLM via a routing API; adjacent-project hypothesis catalog updated.
- **Wave 14B** (7 parallel — the largest wave of the arc): a funding-rate
  cross-sectional signal discovered (rank z-score across a top-cap crypto
  universe at daily hold; modest IC, block-permutation cleanly passed
  Bonferroni at the project's cell count; promotable gross Sharpe collapsing
  to negative net under the typical retail-tier cost band). Plus: a 7-family
  microstructure hypothesis catalog; macro-event surprise infrastructure (a
  public macro-data series pipeline + surprise-z computation); the tier-2
  chain pilot confirmed as a data-limited null.

**Architectural pattern**: a large parallel wave often produces one milestone
discovery alongside the catalog of the *next* hypothesis set.

### §0.7 Foundation cumulative state (end of Wave 14)
- Promotable candidates: 1. Archived nulls: 17 rows (later +11 by Wave 25 → 28).
- Cron jobs running: 4. Shared/ modules: 4, all standalone, zero external
  in-house dependencies. Hypothesis catalogs: 4. Tests: ~120 (409 by end of
  Wave 25).

### §0.8 Foundation → auto-progression transition
Wave 15 was the first fully automated wave. From Wave 16 onward, the next wave
fired automatically on commit. The brakes that make manual → auto-progression
safe are exactly two: (1) an atomic commit per wave (interrupt-safe restart
point); (2) a memory note for auto-wave-progression (fresh session resumes
immediately). **Generalization**: any long-horizon research repo can evolve
along the same curve once the harness and skills are solid.

---

## §1. Principles checklist (audit results)

- [x] CLAUDE.md under the 200-line ceiling.
- [x] All rules verifiable (absolute rules + R→L 7-gate checklist).
- [x] Standalone principle stated (no in-house external imports).
- [x] MCP list documented with rationale (minimal config; trading-API MCPs
      explicitly disallowed for everyday use).
- [x] Sub-agent usage policy defined (main agent does orchestration only).
- [x] Parallelism mandate (R-mode parallel) and serial L-mode (live-order path)
      both stated.
- [x] Overnight resumption protocol defined (manifest + checkpoint).
- [x] karpathy-guidelines and overnight-autonomy skills installed; deny rules
      reflected in `.claude/settings.json`.
- [x] Reference-validation checklist applied per signal (ADF + KPSS + turnover
      + permutation + cost-aware Sharpe).
- [ ] **drift-watchdog skill not installed** — recommendation §6.1.
- [ ] **abandon-criteria skill not installed** — recommendation §6.2.
- [x] LLM role boundary stated (research/ is free; live/ order paths are blocked
      by a PreToolUse hook).
- [x] /memory load confirmed; CLAUDE.md and auto-memory cleanly separated.

---

## §2. Core lessons (10, transferable across projects)

### Lesson 1: in-sample / walk-forward divergence — 5 reproductions
Cells that pass all promotion gates in-sample can still flip on walk-forward,
with only 1 of 4 folds carrying a positive lift. Reproduced 5 times. **Action**:
walk-forward sign-stability gate (≥3 of 4 folds carrying same-sign net Sharpe
lift) added to the promotion checklist. **Generalization**: any project whose
promotion checklist lacks this gate is exposed — the in-sample pass is fold-1
luck on average.

### Lesson 2: short-window IC collapse — 4 reproductions
A cell that sits in the promotable IC band on a sub-6-month window will often
sign-flip, lose magnitude, and fail permutation when the window is extended
1.5x–2x. **Action**: first IC read on under 6 months of sample treated as an
upper bound; 1.5x–2x window extension mandatory before any promotion-gate
evaluation. **Generalization**: applies to every cross-sectional pilot with
limited in-sample history; especially factor-replication and multi-asset repos.

### Lesson 3: standard vs block permutation — 2x disagreement, 3 reproductions
A signal can pass standard-permutation but fail block-permutation at the
project's chosen block size. Forward-return overlap inflates the standard-perm
tail. **Action**: every signal at a horizon ≥ 4h uses block-permutation as
primary, with the block size as a parameter on the perm-test dispatcher.
**Generalization**: sub-second signals are not exposed; daily and swing signals
should use block-permutation as primary.

### Lesson 4: linear vs HP detrend sign-flip diagnostic
A slow series detrended linearly produced a sign-flip on one archived row's IC;
HP-detrending recovered the original sign. When the sign moves with the choice
of detrend, the detrend itself is the artifact. **Action**: run multiple
detrend variants as a diagnostic; if two independent detrends both flip versus
baseline, the baseline is a non-stationary level artifact. **Generalization**:
any cross-sectional pilot using slow series (sentiment indices, supply,
market-share variables) needs this diagnostic.

### Lesson 5: LLM-mediated event signal can be pure overhead
An LLM-bias variant on a small event sample was not significant under
permutation; the direct numeric variant on a much larger sample was also not
significant. The LLM did not extract signal that the raw numeric channel could
not see. **Action**: every LLM-mediated signal pilot must measure the raw
numeric baseline first, before evaluating the LLM variant.
**Generalization**: any sentiment-LLM or text-classifier pilot — justify the
LLM only when it beats the raw baseline statistically.

### Lesson 6: mechanism portability ≠ sign portability
A well-known low-vol-vs-high-vol academic anchor was ported directly to a
top-cap cross-sectional crypto universe; the sign came out opposite.
**Action**: when porting an academic invariant from one asset class to another,
the mechanism may carry but the sign must be empirically re-verified.
**Generalization**: directly applicable to any factor-zoo or cross-asset
replication project porting equity factors to crypto, FX, or KR equities.

### Lesson 7: overlay rescue is mostly in-sample lottery
Two batteries of overlay-rescue attempts (a regime gate, then a stepwise
drawdown-conditional sizing schedule) produced multiple RESCUED-RISK results
in-sample. A subsequent 4-fold walk-forward reverted 4 of 5 cells; only one
combined cell carried. **Action**: every overlay-rescue claim must pass the
same 4-fold WF gate before entering the promotion track. **Generalization**:
every reformulation (regime gate, vol filter, drawdown sizing) needs the WF
gate; in-sample rescue is not enough.

### Lesson 8: overlay zero-out vs sign instability — a separate WF failure mode
A stepwise drawdown-sizing overlay can collapse to zero size after an early
loss, leaving 1–2 active folds carrying all the lift. This is a distinct WF
failure mode from sign-flipping. **Action**: classify cells with mean overlay
size below a threshold as degenerate, on a per-fold basis. Full-window
aggregates hide the zero-out. **Generalization**: any live system using
drawdown-conditional sizing needs the per-fold degenerate-overlay flag.

### Lesson 9: atomic commit + temp-HEREDOC + .trash/ pattern
Apostrophes inside commit-message HEREDOCs break `git commit`, and `rm` is
denied by the project's PreToolUse rules. Both problems were resolved together
by writing the commit message to a tempfile under `.git/`, committing with
`git commit -F`, and `mv`-ing the tempfile to a `.trash/` directory (in
`.gitignore`). **Generalization**: applies to any autonomous-mode repo using a
write-blocking PreToolUse hook.

### Lesson 10: do not stop R&D while the lead candidate waits on cron data
The lead candidate sat in a cron-data-accumulation hold (a public-API window
cap meant the historical sample had to accumulate forward). Had the project
paused on that one cell, Wave 16+ would not have happened. **Action**: keep
the 4-agent parallel reformulation/reopen/new-pilot pipeline running
independently of the lead candidate's hold state. Across 25 waves this produced
28 archived nulls + 4 additional promotables. **Generalization**: any
live-research repo with a cron-blocked lead candidate should split the
lead-candidate path from the R&D backlog and run them in parallel.

---

## §3. Infrastructure outputs (reusable across projects)

### shared/ modules

The project's standalone shared/ tree exposes four project-agnostic
ML/finance utility modules. Interfaces (project-agnostic, copyable as-is):

- **validation module**: `ic_spurious_check(signal, fwd, ...)` →
  ADF + KPSS + turnover gate; `sharpe_daily(returns)` → daily aggregation
  × √252 (per-bar Sharpe disallowed); `pnl_ic_gap(...)` → IC ≠ PnL diagnostic.
- **walk-forward module**:
  `walk_forward_folds(series_index, n_folds=4, warmup_days=30, ...)` plus
  a calendar variant and a `walk_forward_eval(...)` driver.
- **permutation module**: `permute_standard` / `permute_block` (block size
  parameterized); `two_sided_p` with a symmetric smoother;
  `ic_null_distribution` dispatcher with a `scheme` parameter.
- **risk module**: `RiskLimits` (frozen dataclass; no env-var override);
  `ExchangeOutageGuard` (deterministic now-time API); `DrawdownLadder`
  (frozen dataclass implementing a stepwise drawdown-conditional sizing
  schedule).

**Reuse recommendations**:
- KR live-research repos — same interface, swap the market layer.
- Factor-replication repos — block-permutation + walk-forward standard.
- Short-window-collapse audits — pull the validation module's IC tools.
- Higher-frequency repos — adopt the daily-aggregation Sharpe rule (no
  per-bar Sharpe) regardless of bar resolution.

### Cron infrastructure
- Cloud-side: hourly open-interest snapshot, hourly event-prediction-market
  snapshot, hourly options snapshot (added later for the options-vol path),
  daily nightly-rebuild.
- Local-side: a multi-hour gap-monitoring cron between two data sources;
  weekly walk-forward reruns (one per repo).

### Cofounder-brief adaptation workflow
A 130-strategy idea catalog (multi-market, multi-horizon) was filtered down to
a single-market, single-horizon repo by: (1) market-fixed + horizon-fixed
invariants as the first out-of-scope filter; (2) keep only mechanism-transferable
ideas (~9 of 130); (3) each in-scope idea taken to row-level reopen-criterion
granularity in the archived-nulls ledger. **Generalization**: the workflow
applies to any cofounder collaboration or cross-market idea transplant.

---

## §4. Open risks / next-wave candidates

**Time-dependent (cron data accumulation)**: an options-vol path waiting 30–60
days for first IC read; the event-prediction-market gap pipeline (multi-week
accumulation); the on-chain whale-flow pipeline (free-tier API + a couple of
weeks); a liquidation-cascade pipeline (paid feed required).

**Reopen-pending (next-method explicit)**: one archived row carries a
term-structure variant as primary reopen path (gated on cron data above); one
stablecoin row has per-issuer decomposition flagged as the next variant; one
sentiment-contrarian row passed the follow-up detrend diagnostic but the
signal itself collapsed and is closed.

**Operator action queue**: register the new options-snapshot cron; monitor the
open-interest snapshot accumulation (the lead candidate's walk-forward window
unblocks at a ~3-month threshold); paper-trade the lead candidate to measure
maker-fill rate (cost model assumes 100% maker fills; needs empirical
validation).

---

## §5. Audit conclusions

1. **CLAUDE.md absolute-rule compliance**: all hard rules held across 25 waves,
   zero violations. The PreToolUse hook blocking writes to live-trading paths
   was effective.
2. **Standalone principle held**: zero external in-house package imports;
   shared/ utilities implemented locally.
3. **Parallel-mandate (R-mode) compliance**: every wave used 4-agent parallel
   fan-out. Serial L-mode (live order path) has not yet been exercised — the
   project is still at the paper-trade stage.
4. **Atomic-commit + .trash/ pattern**: standardized from Wave 22 onward;
   resilience across overnight sessions confirmed.
5. **Skill installation**: the karpathy-guidelines and overnight-autonomy
   skills both installed; deny rules reflected in `.claude/settings.json`.

---

## §6. Recommendations

- **§6.1 Install drift-watchdog skill** — 25 waves ran without one. Promotion
  candidates can become stale (walk-forward not re-run as time passes) without
  a watchdog firing.
- **§6.2 Install abandon-criteria skill + declare ≥3 kill triggers** — without
  abandon-criteria, reopen-pending rows drift into "someday maybe"
  indefinitely. Recommended kill triggers: a tier-2-chain on-chain pilot
  (paid data not secured + 6 months → close); an event-prediction-market
  lead-lag pilot (event-tag filter not specified + 6 months → close); a
  machine-forecast-disagreement pilot (100+ feature breadth not secured +
  12 months → close).
- **§6.3 Cross-repo lesson propagation** — propagate as small targeted audit
  notes: walk-forward sign-stability gate retrofit (Lessons 1, 7);
  short-window-collapse rule for any cross-sectional / factor-replication
  project (Lessons 2, 6); LLM baseline mandate for any sentiment-LLM or
  transformer-text project (Lesson 5); block-permutation applicability check
  for any daily/swing-horizon project (Lesson 3).
- **§6.4 Strengthen the workflow archive templates** — pull the invariants from
  the auto-progression waves back into the live-research template: 7-gate
  promotion checklist (walk-forward gate explicit), 4-module shared/ utilities
  standard, block-permutation default with parameterized block size,
  atomic-commit + `.trash/` pattern documented.

---

## §7. Patches directly applicable to other repos

1. **Walk-forward sign-stability gate** — insert into any live-research repo's
   R→L promotion checklist as a new gate: net Sharpe lift carries the same
   sign on ≥3 of 4 folds (in-sample-only evaluation depends on a single
   fold's luck).
2. **shared/ module standard** — standardize `ic_spurious_check` and
   `sharpe_daily` interfaces across research repos; add `walk_forward` and
   `perm_test` modules to the shared template.
3. **Short-window-collapse operating rule** — for every cross-sectional pilot:
   first IC read on under 6 months of sample → 1.5x–2x window extension
   before promotion-gate evaluation.
4. **LLM baseline mandate** — for every LLM-mediated signal pilot: raw numeric
   baseline IC + permutation measured first → LLM variant compared directly →
   drop the LLM if the variant is not statistically better.

---

## §8. Meta-observation

Two architectural insights stand out from this 25-wave arc.

**"Honest nulls are 95% of the discoveries."** 28 archived nulls + 5
promotables = roughly a 15% discovery rate. This sits squarely with the
meta-analytic finding from Jensen-Kelly-Pedersen (2023), "Is There a
Replication Crisis in Finance?", where roughly a quarter of academic factors
fail OOS. A repo that enforces walk-forward OOS validation *before* promotion
should naturally land at a lower discovery rate than that academic baseline,
and this is what we observe.

**"Methodology lessons travel further than signal discoveries."** The
promotable candidates from this repo are not directly applicable to other
markets or other strategies. The 10 lessons in §2 are. A research-process
archive's value is in its methodology audit trail, not in its signal catalog.

---
