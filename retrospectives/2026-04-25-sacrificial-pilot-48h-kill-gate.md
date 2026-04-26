# Sacrificial Pilot — 48-Hour Kill Gate Fired (Abandon Retrospective)

*This is a sanitized public version of a private retrospective. Methodology details (kill-gate mechanics, process learnings, reusable infrastructure framing) are preserved; signal-specific content (exact metric values, ticker lists, model names, universe counts, internal repo names) has been stripped or expressed in qualitative bands.*

**Date**: 2026-04-25
**Project status**: **Abandoned** (Day 2 EOD kill gate)
**Audience**: future sacrificial-pilot designers
**Total effort**: ~6 hours (Day 1 environment setup + Day 2 cache wait + scoring + Sharpe)
**LLM cost**: under $0.05 in API calls (~150 calls; cap was $5, well below the $8 abandon ceiling)
**Headline**: pre-registered kill gate fired as designed. Negative result + reusable infrastructure preserved.

---

## 1. Executive Summary

### Project definition (original)
A 2-day sacrificial pilot testing whether an LLM-derived controversy-intensity score predicts cross-sectional returns *within* a well-known sin universe (tobacco / alcohol / gambling / defense). Hypothesis H1: firms under stronger ESG-exclusion pressure carry a larger premium for unconstrained investors — a within-universe extension of the classic between-universe sin premium literature.

### Pre-registered kill gate
At Day 0 (CLAUDE.md draft time), the following was committed in writing:
> in-sample Sharpe (top decile − bottom decile spread) > 0.3 → survive → merge into a parent appendix
> ≤ 0.3 → immediate abandon, repo freeze, agents released

### Outcome
- Universe: ~60 curated CIKs across 4 sin sub-industries
- Filing-text cache hits at scoring time: most of universe (one filing-audit pipeline served as upstream)
- Successfully scored: ~50 firms (a 3-model ensemble across 70B-class open + frontier providers)
- Returns coverage: ~40 firms (8 ticker symbol failures — class-share notation, ADR mismatch, delistings)
- In-sample window: 2015-01..2022-12, ~95 monthly observations
- **Result: negative Sharpe (sign-reversed), well below the 0.3 pre-registered gate**

The result was not a marginal miss — the spread had the *opposite sign* of H1. Both magnitude and direction failed.
Pre-registered gate fired → abandon.

### Meta-lesson (one line)
> **"When a hypothesis appears to confirm with the wrong sign, sign-flipping in-sample is the textbook definition of overfitting. The value of a sacrificial pilot is not the result — it is your future self being bound by rules your past self pre-registered."**

---

## 2. Assets worth preserving

### 2.1 Reusable infrastructure (lift-and-shift candidates)

| Asset | Reusable for |
|---|---|
| **3-model ensemble client + JSON parser** | other LLM-scoring alpha pipelines |
| **10-K HTML → Item 1A/3 section extractor** | other filing-form pipelines (13D/13G, comment-letter correspondence) — section regex needs per-form tuning |
| **Distribution bunching detector** (≥80% concentration in one bin → abandon) | schema sanity check for any LLM-scoring pipeline |
| **Sharpe runner with auto-ABANDONED.md emission** | any spread-portfolio test — both pass and fail paths produce artifacts automatically |
| **Daemon-mediated SEC fetch + cache pattern** | shared-utils internalized; this pilot was the first user |
| **Cron-driven poll-and-chain pattern** | other daemon-bound pipelines — wait for upstream daemon, auto-advance to next stage |
| **Pre-registered kill gate worked example** | sacrificial-pilot template + abandon-criteria skill — first successful execution case study |

### 2.2 Negative result (literature value)

> **Within-sin-universe controversy sub-ranking is sign-reversed in-sample over 2015-2022 on ~40 US-listed sin issuers (negative annualized Sharpe).**

Two interpretations (both require separate OOS testing):
1. **No within-universe effect**: the classic sin premium is a between-universe (sin vs non-sin) phenomenon and does not survive within-universe sub-ranking.
2. **Reverse signal — quality**: low-controversy firms may win because they have cleaner balance sheets, lower leverage — 5 bottom-decile firms (lower-controversy quality names) populated the winning leg. But in-sample sign-flip is the textbook overfitting signature, so this interpretation must be tested OOS before any belief is updated.

Detailed calculation log + retrospective live in the abandoned-pilot repo's `ABANDONED.md`.

---

## 3. Process learnings

### 3.1 What worked

- **Pre-registered kill gate's self-binding force.** At outcome-reveal time, the temptation to sign-flip / change the threshold / add explanatory variables was strong. The 0.3 threshold pre-committed in CLAUDE.md bound the future self. Validates the core premise of the abandon-criteria skill.
- **Daemon-mediated coordination.** A shared coordination repo's daemon + semaphore pattern produced zero SEC rate-limit conflicts across 4 concurrent repos. This pilot's filing-audit work queued behind an earlier filing-type queue and waited — that is ordering, not conflict, and is exactly the intended behaviour.
- **Scope discipline.** Strong urges to add variants (extra industry-code bins, OOS test, factor-model adjust) within the 2-day box were resisted. The CLAUDE.md framing — "this is not the main weapon / abandonment IS the deliverable" — held the line against scope creep.
- **Cron + chain prompt pattern.** A 30-minute self-loop that fired when cache reached threshold count completed score → Sharpe → checkpoint inside a single fire, with no user attention required mid-pipeline.

### 3.2 Inefficiencies / improvement points

| Issue | What happened | Recommended mitigation |
|---|---|---|
| **LLM provider model deprecation** | Two configured models returned 404 at first scoring call. Discovered late (Day 2) instead of at startup. | Add a startup-time model probe to the shared LLM client. Daily sanity check that listed models return 200 OK. |
| **10-K section regex failed on 9 firms** | Cache existed but Risk Factors / Legal Proceedings extraction failed (foreign issuers, non-standard Item heading variants). | Add fallback strategy to the extractor: (a) plain-text keyword matching, (b) Item-heading variant dictionary, (c) raw-text first 6k characters as last resort. |
| **Hand-typed CIK seed had fabrication risk** | Universe-builder used a hand-typed CIK list. 2 entries mismatched their declared industry bin and were caught only by tests; 6 invalid CIKs surfaced as cache misses (daemon 404). | Add a workflow rule: any hand-typed external ID must be schema-invariant tested in CI. Prefer authoritative-source files (e.g. SEC company_tickers.json) over manual entry whenever available. |
| **`.env` deny rule conflicted with dotenv loading** | The pilot's hook configuration denied `.env` write, which also blocked `.env.example` copy. Workable because the shared-utils client reads from process env, but explicit init-time loading was needed. | Keep the `.env` write deny in the sacrificial-pilot template, but make shared-utils auto-load the parent coordination repo's `.env` via dotenv as the default pattern. |
| **Daemon priority order opaque** | The upstream daemon processed a different filing type before this pilot's queue, producing ~80 minutes of wait. Priority logic was internal to the daemon and not externally visible. | Expose per-filing-type queue depth in the daemon manifest; surface it in the status report. |
| **Returns-source ticker symbol failures** | 8 ticker symbol failures (class-share notation, ADR mismatch, delistings) — silently dropped to stderr. | Add a ticker-fallback table for known notation variants. Promote dropped tickers to an explicit per-firm report at the lookup stage instead of stderr. |

### 3.3 Sacrificial-pilot pattern validation

This pilot was the first execution of the sacrificial-pilot pattern. The pattern itself worked:
- Timebox was unambiguous (2 days)
- Kill gate was a single metric, threshold pre-committed, statistically justified
- Survival path was specified (merge into parent appendix); abandon path was specified (`ABANDONED.md`)
- "Not the main weapon" framing actively suppressed scope creep

→ Recommend extracting into a dedicated `sacrificial-pilot-CLAUDE.md` template for future use.

---

## 4. Workflow-archive recommendations

### 4.1 Immediate
- [x] **Write this retrospective** (this file).
- [x] **Update the project register**: `status: planned → abandoned`, add `abandoned_at` / `abandoned_reason` fields, link the retrospective.
- [x] **Add a Worked Examples section to the abandon-criteria skill** with a success/failure comparison table.

### 4.2 Short-term (before next sacrificial pilot)
- [ ] **Create a `sacrificial-pilot-CLAUDE.md` template** with mandatory sections:
  - `## Kill Gate (absolute EOD basis / pre-registered threshold / statistical justification)`
  - `## Abandon Criteria (beyond the kill gate)` — data collection failure / cost overrun / distribution degeneracy
  - `## Survival absorption path` (which parent repo / branch / appendix)
  - `## Abandonment artifact` (`ABANDONED.md` format spec: reason + calc log + retrospective)
  - `## Interview positioning` — pre-write both the survival and abandon narratives
- [ ] **Add to principles doc**: "A sacrificial pilot's deliverable is process, not outcome. State up front that `ABANDONED.md` IS the deliverable."

### 4.3 Long-term
- [ ] Daemon priority visibility upgrade (see §3.2 table).
- [ ] LLM client model-probe sanity check (see §3.2 table).
- [ ] On the next sacrificial pilot, reload these three artifacts together at kickoff: this retrospective + abandon-criteria skill + sacrificial-pilot template.

---

## 5. Rollback / resume path

This pilot is frozen. If resumed:
- Repo itself is preserved (`ABANDONED.md` + reproducible code + scores data).
- Resume triggers: (a) testing the same hypothesis on a different universe / OOS window, (b) explicitly pre-registering the sign-reversed (low-controversy → premium) hypothesis and OOS-testing it.
- A fresh harness should be written if resumed — this pilot's CLAUDE.md assumes a 2-day timebox.

Final commits (commit hashes redacted): Day 1 scaffold (universe, fetch, score, sharpe modules); Day 2 prep (extractor, runners, ~18 unit tests, ensemble fix); ABANDONED commit (Sharpe below pre-registered gate).

---

## 6. Next-audit triggers

- At the next sacrificial-pilot design time, reload this document together with the abandon-criteria skill.
- At the next coordination-daemon weekly retrospective, revisit the daemon-priority visibility item.
- When other alpha projects (in adjacent research streams) reuse the same LLM-ensemble + section-extractor pattern, evaluate cross-project reuse value in a separate retrospective.
