# Quant Research Process

This repo is a curated archive of methodology artifacts from a multi-project
quant research workflow. It is **not** a signal repository — none of the
content here is a deployable strategy, and none of the numbers are
calibrated to a live system. Instead it documents *how* research was
conducted: what worked, what didn't, what was abandoned and why.

The artifacts are grouped into:

- **[retrospectives/](retrospectives/)** — multi-day research arcs and
  abandoned pilots (sacrificial-pilot kill-gate firings, archived nulls,
  developmental progressions).
- **[runbooks/](runbooks/)** — project-agnostic methodology (promotion
  gates, multi-agent fan-out, standalone live-research bootstrap).
- **[case-studies/](case-studies/)** — single-topic deep dives (factor
  replication completion review, prior-art survey for an activist
  strategy).

Companion public repos (linked separately):
- A 212-factor Open Asset Pricing replication project with mechanism
  classification and a Streamlit dashboard.
- An in-flight Schedule 13D Item 4 intent classification project.

Operational live-trading repos are private by design — secrecy is part of
the discipline. This repo exists to share the *process*, not the alpha.

---

## Reading order for a 5-minute scan

If you only have 5 minutes:

1. **[retrospectives/2026-04-25-sacrificial-pilot-48h-kill-gate.md](retrospectives/2026-04-25-sacrificial-pilot-48h-kill-gate.md)**
   — fastest signal of process discipline. A pre-registered kill gate
   fired in 48 hours; the project was abandoned cleanly with a
   reusable-infrastructure carve-out and a public retrospective.
2. **[runbooks/research-promotion-gates.md](runbooks/research-promotion-gates.md)**
   — the 7-gate validation pipeline (ADF+KPSS, block-permutation,
   walk-forward sign-stability, cost-cliff sweep, short-window extension,
   long-vs-long-only asymmetry) applied across projects.
3. **[case-studies/2026-04-26-factor-zoo-completion-review.md](case-studies/2026-04-26-factor-zoo-completion-review.md)**
   — what shipping a complete research project looks like, including the
   honest reckoning between literature claims and replication reality.

If you have 20 minutes, add:

4. **[retrospectives/2026-04-26-cross-sectional-overnight-25-wave-arc.md](retrospectives/2026-04-26-cross-sectional-overnight-25-wave-arc.md)**
   — 25 waves of overnight research (28 archived nulls + 5 promotables
   + 10 methodology lessons) on a crypto live-research repo.
5. **[retrospectives/2026-04-26-multi-vendor-data-4-day-arc.md](retrospectives/2026-04-26-multi-vendor-data-4-day-arc.md)**
   — 4-day manual progression on a KR equities live-research repo,
   surfacing a survivorship-bias asymmetry specific to single-side
   markets (12 lessons).
6. **[runbooks/multi-agent-fanout-runbook.md](runbooks/multi-agent-fanout-runbook.md)**
   — the 7-element prompt template + atomic commit + manifest+checkpoint
   resumability pattern that produced the parallel research throughput
   in the retrospectives above.
7. **[runbooks/standalone-live-research-bootstrap.md](runbooks/standalone-live-research-bootstrap.md)**
   — Day 0 setup for a new live-research repo (directory layout,
   shared/ 4-module pattern, hooks, skill installs, promotion checklist).

---

## What this archive demonstrates

| Signal | Where to find it |
|---|---|
| **Pre-registration discipline** (kill gates, abandon criteria, hard timeboxes) | sacrificial-pilot retrospective; sacrificial-pilot pattern in standalone-bootstrap §10 |
| **Honest negative results** (28 archived nulls, 11 archived REJECTs, sign-reversed sacrificial-pilot result published as the deliverable) | 25-wave arc, 4-day arc, sacrificial-pilot retrospective |
| **Validation rigor** (ADF+KPSS, block-permutation, walk-forward sign-stability, short-window collapse extension, LS-vs-LO asymmetry, cost-cliff sweep) | research-promotion-gates runbook |
| **Honest stat→product separation** (no leaping from p-value to claim, no leaping from in-sample to OOS) | factor-zoo completion review; promotion-gates §2-§4 |
| **Reproducibility infrastructure** (manifest+checkpoint, atomic commits, .trash/ pattern, incremental save+resume) | multi-agent-fanout runbook §3-§5; factor-zoo completion review Finding 4 |
| **Methodology meta-doc** (process artifacts curated as their own deliverable, not just a side-effect of code) | this entire repo |

---

## What this archive does NOT contain

- Live trading code, signal backtest code, or paper-trading code
- Specific signal names, thresholds, universe lists, or cost calibrations
- Cron schedules, broker endpoints, or API credentials
- Personal P&L or position data
- Anything that could be used to reverse-engineer a deployed strategy

Source documents from the private repos these artifacts derive from are
not included. This is by design — a methodology archive is one form of
public artifact; signal commoditization through public release is a
different, mostly unwanted, form.

---

## Document conventions

All docs use a consistent header format:
- **Date** of the original artifact (YYYY-MM-DD)
- **Topic** in slug form
- **Project category** (descriptive, never the actual private repo name)

Cross-references between documents use relative links. Cross-references
to private internal sources are deliberately stripped.

Any specific number that appears (κ, fold count, time window, cost band)
is either a methodology constant (e.g., 4-fold walk-forward) or a
qualitative band illustration — never a live signal calibration.

---

## License

MIT. See [LICENSE](LICENSE).

The methodology described here may be freely adopted; the lessons are
specific instances of long-known finance and ML principles, applied with
the care that a small operator can afford and that a large institution
sometimes shortcuts. Forks, reformulations, and translations are welcomed.

If you use any of this material in a published paper or commercial
product, attribution is appreciated but not required.
