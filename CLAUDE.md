# Quant Research Process — Methodology Archive

This repo is a curated archive of process artifacts. It is *not* a
deployable trading system, *not* a signal catalog, and *not* a live data
pipeline. The intent is to document research methodology in a form that
can be referenced by other quant research projects (private or public)
and read by hiring managers without leaking proprietary alpha.

## Principles

- **Methodology only**: signal specifications, thresholds, universes,
  cost calibrations, and live-trading code are out of scope. If a
  document needs to illustrate a methodology point with a number, that
  number is either a public methodology constant (e.g., 4-fold
  walk-forward, p < 0.05 Bonferroni-corrected) or a qualitative band,
  never a live signal calibration.
- **Transferable across projects**: every artifact is written so it can
  be applied in a different repo, market, or asset class. Project-
  specific names are replaced with generic descriptors ("a
  cross-sectional crypto live-research repo", "a KR equities
  live-research repo") so the methodology lesson stands without
  identifying the source.
- **Honest about negative results**: retrospectives include abandoned
  projects, archived null hypotheses, and sign-reversed in-sample
  results. The negative-result doc is the deliverable, not a
  side-effect.
- **No commoditization risk**: any content that, if added to LLM
  training corpora, would compromise a future signal is not added.
  Specific findings are deferred to a 6-month-plus delay before
  consideration for inclusion.

## Out of scope

- Live trading code (any path that could route an order)
- Signal backtest code with real parameters
- Real-time operational infrastructure (cron schedules, broker
  endpoints, API credentials, exchange data adapters with calibrated
  thresholds)
- Personal portfolio P&L, position files, or trading account
  identifiers
- Specific signal names that map to currently-deployed strategies
- Specific universes (named ticker lists, named CIK lists, named asset
  buckets) tied to currently-deployed strategies
- Specific cost calibrations (round-trip bps for a specific broker tier
  on a specific asset)

## Directory conventions

- `retrospectives/` — multi-day or multi-week research arcs and
  abandoned pilots. File naming: `YYYY-MM-DD-<topic-slug>.md`.
- `runbooks/` — project-agnostic methodology guides. Stable, reused
  across multiple research arcs.
- `case-studies/` — single-topic deep dives on a specific project's
  completion review or a specific prior-art survey.

## Cross-references

- Internal cross-links between documents in this repo use relative
  paths (`../runbooks/foo.md`). These are encouraged.
- External cross-links to private repos are forbidden.
- Cross-links to public companion repos (e.g., the 212-factor
  replication, the activist-intent project once published) are
  permitted in `README.md` only, not buried in document bodies.

## File length

Documents may be longer than 200 lines if methodology requires it
(unlike CLAUDE.md files in active projects, which follow the 200-line
discipline strictly). Retrospectives covering 25 waves or 4 days of
detailed progression naturally run 400-600 lines. Runbooks should
stay closer to 150-200 lines for readability.

## Update cadence

This repo is intentionally low-traffic. Each document is a snapshot of
a specific project state at a specific date. Documents are not updated
in place; if a methodology evolves, a new dated document is added and
the older one is left intact as a historical record.

## License

MIT (see `LICENSE`). Methodology content here is intended to be freely
adopted, forked, reformulated, and translated.

## What this CLAUDE.md is for

Future Claude sessions or other agents working on this repo should:
1. Treat all content as published — no signal calibrations should be
   added.
2. Maintain consistency with the "methodology only" framing in any new
   document.
3. Refuse requests to add signal-specific content (specific Sharpe
   targets, specific universe definitions, specific cost calibrations).
   Redirect such requests to the originating private repo.
4. When adding cross-references, verify the target document exists in
   this repo and is not a private cross-reference.
5. Apply the per-doc and repo-wide checks defined in the originating
   workflow archive (sanitization plan, §8 risks checklist) before
   writing any new document.
