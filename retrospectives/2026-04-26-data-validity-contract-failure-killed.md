# Activist-Intent Classifier — Data-Validity Contract Failure (Killed Postmortem)

*This is a sanitized public version of a private postmortem of an activist-intent classification project (Project Y, killed 2026-04-26). Methodology details (the three input-layer data-validity contracts, the layered abandon-criteria framework, the worked-example trigger analysis) are preserved; signal-specific content (exact κ values, exact filing counts, internal repo names, file paths, commit SHAs) has been stripped or expressed in qualitative bands. This is the **second sacrificial-pilot retrospective** in this archive — a companion to [2026-04-25-sacrificial-pilot-48h-kill-gate.md](2026-04-25-sacrificial-pilot-48h-kill-gate.md) — and documents a **distinct failure mode**: where that retrospective showed an output-layer kill gate firing as designed, this one shows an input-layer data-validity contract collapse that the output-layer triggers were structurally unable to detect.*

**Date**: 2026-04-26
**Project status**: **Killed** (user decision, data-layer contract failure)
**Audience**: future LLM-classification alpha designers; abandon-criteria framework maintainers
**Total effort**: ~18 hours (continuous burst, single overnight cycle)
**LLM cost**: well under the pre-registered $25 cap (qualitative: ~13% of cap consumed)
**Headline**: Five pre-registered abandon criteria fired zero mechanical triggers — but three input-layer data-validity contracts broke simultaneously, making statistical evaluation of the output-layer gates structurally meaningless. First exposure of an **input-layer blind spot** in the abandon-criteria framework.

---

## 1. Executive Summary

### Project definition (original)

A 2-week scoped project to classify Schedule 13D Item 4 intent text into a 5-class schema (hostile / strategic / passive / governance / boilerplate), labeled by a 3-model LLM ensemble with confidence weighting, and tested for cross-sectional CAR predictive power against the published Brav-Jiang-Partnoy-Thomas (2008) baseline. Distinct from a 2-day sacrificial pilot — this was scoped as a "main weapon" project with five pre-registered abandon criteria.

### Kill reason

Three input-layer contracts violated simultaneously:

| Contract | Assumption | Actual | Gap |
|---|---|---|---|
| Sample availability | tens of thousands of filings (broad universe, 11-year window) | a few hundred filings (narrow seed pool after 13D filter) | order of ~1% of target |
| Label rater methodology | Multi-rater consensus, κ ≥ 0.7 | Single non-domain-expert labeler, κ near 0.5 | well below the 0.7 target |
| Coverage / end-to-end pipeline | CAR computable on 100% of sample | ~25% of acquired filings yielded computable CAR (CUSIP→ticker mapping bottleneck) | -75% |

Because these three contracts broke at the input stage, the pre-registered output-layer triggers became **structurally impossible to evaluate**:
- Oracle κ landed just above the floor of 0.5 (so trigger #2 did not fire mechanically) but well below the 0.7 target (so the gate failed in spirit)
- Effective sample collapsed to a quarter of acquired filings, making CI estimation noisy enough that the "CI ∋ 0" trigger could not be evaluated meaningfully
- Spend stayed well under the pre-registered $25 cap (so trigger #4 did not fire mechanically)
- Short-side feasibility evaluation never ran — hostile-class sample was too small to evaluate

In short: every trigger was designed to fire **only when results are measurable**. A scenario where measurement itself is impossible is a sunk-cost rationalization blind spot.

### Meta-lesson (one line)

> **Abandon criteria must be layered. Output-layer triggers (effect size, CI, p-value, budget) cannot detect input-layer validity collapse. Phase 0 entry must validate input-layer contracts (sample availability / rater methodology / end-to-end pipeline coverage) and declare triggers for each.**

---

## 2. Mechanical abandon criteria check (5 pre-registered triggers)

| # | Trigger | Layer | Threshold | Actual | Fired? | Notes |
|---|---|---|---|---|---|---|
| 1 | Day 2 EOD n < 3000 | data-resource hybrid | n ≥ 3000 | ~250 | (eval timing ambiguous) | Continuous 18h burst made "Day 2 EOD" undefined. De facto miss but no mechanical fire |
| 2 | Oracle κ < 0.5 | output | κ ≥ 0.5 | κ near 0.5 | **No** | Just above floor; gap from 0.7 target ignored. Floor-only, no target-gap trigger |
| 3 | Main signal CI ∋ 0 + regime null | output | — | unevaluable | **N/A** | Effective n collapsed; CI noisy; regime split weaker still |
| 4 | LLM spend > $25 cap | resource | < $25 | ~13% of cap | **No** | 87% of budget unused at kill |
| 5 | Short-side infeasible | output | — | not evaluated | **N/A** | Hostile-class n too small to evaluate |

**Observation**: Of 5 triggers, 3 (#1, #3, #5) were unevaluable, 2 (#2, #4) returned mechanical no. The actual failure mode (input validity collapse) was never in the trigger set.

**Core framework gap**: Trigger #1 was the only data-resource hybrid, but (a) it captured only an absolute floor (n ≥ 3000), not a *target gap* (a few hundred actual vs tens of thousands target ≈ ~1%), and (b) the evaluation timing was ambiguous in a continuous burst.

---

## 3. Data validity contracts — all three broken

### 3.1 Sample availability contract

**Assumption**: tens of thousands of filings. Justification — broad equity universe × 11-year window × combined activist-disclosure form base rate.

**Actual**: ~250 filings, after a shared SEC fetch daemon's FIFO queue (multi-project shared) processed the 13D-only universe down to a ~50-CIK seed pool.

**Root cause**: Sample sourcing depended on **the processing order of a shared daemon queue** across multiple sibling projects. Queue priority was not externally visible, so pre-task capacity estimation was not possible.

**Reference-class signal**: Every other related project that depends on the same daemon faces the same bottleneck. Daemon priority + capacity-probe capability must be elevated to an input-layer prerequisite for any project that consumes from a shared fetch queue.

### 3.2 Label rater methodology contract

**Assumption**: A 5-class schema is unambiguous to a "domain-aware labeler"; κ ≥ 0.7 is achievable.

**Actual**:
- Single labeler (the author, non-domain-expert), 30-item human oracle
- κ (human ↔ ensemble) near 0.5
- κ (human ↔ frontier model) lowest
- κ (frontier model ↔ ensemble) ~0.65 (decent)

The pattern across the three κ values: the human labeler is the bottleneck. Schema-internal agreement (frontier↔ensemble) reached ~0.65 (decent). Human↔frontier was the weakest pair → "domain expertise gap" outranks "schema ambiguity" as the dominant cause.

**Root cause**: Multi-rater capacity must be secured *before project start*. With a single labeler, inter-rater agreement is structurally unmeasurable (self-vs-self κ has no meaning). A well-written labeler guide is necessary but not sufficient — *guide quality and schema validity are independent*.

**Reference-class signal**: Every LLM-classification alpha project has the same risk. The finer the class schema, the sharper the single-rater limitation. Future LLM-classification projects must either secure multiple raters in advance, or simplify the schema to binary.

### 3.3 Coverage / end-to-end pipeline contract

**Assumption**: ~96% CUSIP extraction × adequate CUSIP→ticker mapping → ~100% CAR coverage.

**Actual**:
- CUSIP→ticker mapping was hand-coded for a small fraction of filings
- CAR computed on roughly a quarter of acquired filings — the rest blocked at the lookup stage
- Effective sample collapsed below the already-shrunk acquired sample

**Root cause**: Pipeline-stage dropout was never measured in a Phase 0 dry run. A 1-2 hour OpenFIGI lookup integration would have lifted coverage to 80%+, but because no Phase 0 dry run was conducted, the bottleneck wasn't discovered until the event-study stage on day 5.

**Reference-class signal**: Every LLM-text-mining alpha is a multi-stage pipeline (fetch → parse → label → join → return). Each stage's yield must be measured in a Phase 0 dry run (n=10) before commitment. A single 25%-yield stage compounds — effective sample lands at a fraction of the assumed total.

---

## 4. Salvageable assets

The kill was clean — meaning, code and data artifacts produced during the 18-hour burst remain reusable for future related projects. Without listing internal paths:

### 4.1 Data artifacts

- **30-item human-labeled oracle file** (5-class, bilingual). Reusable for re-labeling once additional raters are secured.
- **Labeler guide** (decision tree + SEC legal-term glossary). Reusable as the schema-spec template for any LLM-classification alpha.
- **3-model ensemble outputs over the full acquired sample** (per-model labels + confidence + disagreement scores). Reusable as a baseline for any follow-on analysis on the same sample.
- **Strategic-subtype label set** (substantive vs boilerplate split on the strategic class). Reusable as a binary robustness-check template.
- **Three-way κ triangulation file** (human / frontier / ensemble). Reusable as a comparison baseline for any future schema evaluation.
- **CAR file with FF3-adjusted 60-day horizons** for the covered subset. Reusable as a baseline for sample expansion after the OpenFIGI fix.
- **Korean translation + glossary of the oracle**. Reusable when bilingual raters join.

### 4.2 Code modules

- An SEC Item 4 parser and amendment-chain handler (reusable across any 13D/A pipeline)
- A 3-model ensemble client with confidence weighting (reusable across any LLM-classification project)
- A daemon-mediated SEC fetch wrapper (reusable across any project consuming from the shared fetch queue)
- A defensive raw-decode workaround for the shared usage-tracking file race condition (a stopgap; the root-cause fix was applied separately at the coordination layer after the kill — see §5.5)

### 4.3 Negative results (literature value)

1. **Single-labeler 5-class κ ≈ 0.5** — direct evidence that a non-domain-expert single rater cannot operationalize a 5-class SEC-text schema to the κ ≥ 0.7 publishable bar. Justifies the rater-pre-secure mandate for future LLM-classification alpha repos.
2. **Hostile-CAR sign-flip on a tiny n (sign reversed vs published Brav-2008 +5–7% baseline)** — directionally opposite to the canonical headline, but n is too small to discriminate reversal from small-sample artifact. If the reversal replicates on a larger sample, the result is a paper-headline candidate ("Why the Brav-2008 hostile signal disappeared post-2024"). If it doesn't replicate, it's a small-sample artifact. Either way the file survives as a calibration anchor.
3. **Strategic-class skew with non-trivial boilerplate share** — quantifies confidence-weighting effect direction for any successor schema. Specific values redacted; the qualitative observation (LLM ensemble over-classifies the strategic bucket, of which a meaningful slice is boilerplate) is the methodology takeaway.
4. **Probe-sample selection bias quantified** — small initial probe showed ~88% unanimous-vote agreement vs ~73% on the full acquired sample. Direct evidence that small probes systematically overstate ensemble agreement — a generalizable lesson for any pilot-sizing decision.

---

## 5. Framework patches added (workflow-archive side)

The kill was treated as a framework-improvement opportunity. The following patches were applied to the workflow-archive at the same time:

### 5.1 Reference-validation checklist — new §4.5 (data validity contracts)

Three input-layer contracts must be specified **before Phase 0 entry**:
- 4.5.1 Sample-availability contract (assumption N + justification + dry run + miss-trigger)
- 4.5.2 Label-rater-methodology contract (rater count + κ pilot + miss-trigger)
- 4.5.3 Coverage / end-to-end-pipeline contract (assumption % + n=10 dry run + miss-trigger)

Phase 0 pass-judgment now requires §4.5 completion for any LLM-text-mining alpha project.

### 5.2 Abandon-criteria skill — new trigger category

- Added a fourth trigger category: **data-validity** (alongside mechanism, product-market, resource).
- Each phase must declare ≥ 1 trigger in each of the four categories.
- New trigger catalog section: sample / rater / coverage / dependency / labeler-secured / queue-priority.
- Worked-examples table updated with this project as the input-layer failure case (counterpart to the output-layer worked example from the companion sacrificial-pilot retrospective).

### 5.3 Principles doc — new §6.8 (Data Validity Contracts)

The one-line meta-lesson elevated to a framework principle. Applies to all future projects with an LLM-classification component, and to any currently-active project's Phase-0 re-entry.

### 5.4 Project register

The project's status row was updated: `active → abandoned`, with `abandoned_at` and `abandoned_reason` fields populated, and a link to this retrospective.

### 5.5 Coordination-layer follow-up tickets — closed same day

Three coordination-layer tickets opened by this postmortem were closed within hours:
- A shared-utility race condition on the LLM-usage-tracking file → resolved with an atomic-write + filelock pattern in the shared utilities module.
- Daemon-priority and capacity-probe visibility → resolved by adding a per-prefix queue-depth probe script and exposing depth in the daemon status report.
- A shared filer-ontology module (CIK → activist-vs-passive classification) → published as a shared utility with tests.

After these three closures, the input-layer contracts in §5.1 become *mechanically passable* by future projects: §4.5.1 can be satisfied by the new daemon-capacity-probe; §4.5.2 / §4.5.3 by multi-rater pre-secure plus a Phase 0 dry run. The framework gap is closed *and* the framework-passing tooling exists.

---

## 6. Adjacent-layer observations (not the cause of the kill)

### 6.1 Reference age vs duration ratio (passed)

The project's anchor references (Brav-Jiang-Partnoy-Thomas 2008, plus the 2024 SEC disclosure-window tightening) had a healthy age-vs-duration ratio. The reference-validation layer passed cleanly. This kill is *not* a reference-validation failure — it is an input-layer failure inside an otherwise well-grounded project frame.

### 6.2 Sacrificial-pilot pattern (worked, but not applied here)

The sister sacrificial pilot retrospectived in [2026-04-25-sacrificial-pilot-48h-kill-gate.md](2026-04-25-sacrificial-pilot-48h-kill-gate.md) used a 2-day timebox and a single-metric kill gate, and was abandoned cleanly when its output-layer gate fired. This project was scoped at 2 weeks, so the sacrificial-pilot pattern was deliberately not applied. **The lesson is that 2-week-scope projects need *more* input-layer rigor than 2-day pilots, not less** — the longer the timebox, the more expensive an input-layer collapse becomes.

### 6.3 Labeler-guide value (independent of the kill)

The 5 cross-cutting principles distilled from domain-labeler feedback at project start (now archived as principles-doc §6.7) survive the kill, and are independently applicable to other related projects.

---

## 7. Next-repo checklist (before any future LLM-classification alpha)

1. Reference-validation document with §4.5 (data validity contracts) populated — no Phase 0 entry without it.
2. Abandon-criteria triggers declared across all 4 categories (mechanism / product-market / resource / **data-validity**) — each with ≥ 1 trigger.
3. For shared-fetch-queue-dependent projects: file an explicit capacity-probe ticket against the coordination layer before declaring sample-availability assumption.
4. For multi-rater labeling projects: pre-secure ≥ 2 raters, OR simplify schema to binary.
5. For multi-stage pipeline projects: run a Phase 0 dry run (n=10) and measure stage-by-stage yield before scope commitment.

---

## 8. Comparison with the output-layer retrospective

The pair of retrospectives in this archive — [2026-04-25-sacrificial-pilot-48h-kill-gate.md](2026-04-25-sacrificial-pilot-48h-kill-gate.md) (output-layer) and this document (input-layer) — together demonstrate that abandonment triggers come in two structurally distinct types:

| Dimension | Output-layer retrospective | Input-layer retrospective (this doc) |
|---|---|---|
| Timebox | 2-day sacrificial pilot | 2-week main-weapon scope |
| Trigger that fired | Pre-registered Sharpe gate (single metric, hard threshold) | None of 5 triggers fired mechanically |
| Reason for abandonment | Result was sign-reversed in-sample — gate fired as designed | Input contracts collapsed; output gates were structurally unable to evaluate |
| Framework status | Validated the abandon-criteria pattern | Exposed an input-layer blind spot in the abandon-criteria pattern |
| Salvage value | Reusable LLM-ensemble + section-extractor + Sharpe runner | Reusable oracle + labeler guide + ensemble outputs + parser |
| Generalizable lesson | "A sacrificial pilot's deliverable is process, not outcome" | "Abandon criteria must be layered: input-layer contracts before output-layer triggers" |

The methodology value of having **both** retrospectives published is the demonstration that abandonment triggers cover both layers. A practitioner who has only experienced output-layer triggers will not have the framework to detect input-layer collapse — and vice versa. The four-category trigger taxonomy added to the abandon-criteria skill (mechanism / product-market / resource / data-validity) is the synthesized output of these two retrospectives.

---

**END OF POSTMORTEM**

Future LLM-classification alpha projects in this workflow stream are required to apply §5.1 + §5.2 + §5.3 before Phase 0 entry. This kill is the first live invocation of the drift-prevention framework installed earlier in the same week.
