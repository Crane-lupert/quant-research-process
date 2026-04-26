# Multi-Agent Fan-Out Runbook

Methodology runbook for parallel signal research using one orchestrator session and N sub-agents. Project-agnostic; adapted from internal practice across multiple research repos. Routinely shortens 1.5-3 hour serial work into ~30 minutes of parallel work, with measured speedups in the 3-6x range.

This runbook documents the *concrete execution procedure*. The underlying parallelism principles (when to parallelize, dependency analysis, resumability) live separately in the harness's parallelism guide. See `research-promotion-gates.md` for the validation gates each sub-agent's output is then run through, and `standalone-live-research-bootstrap.md` for the surrounding repo scaffolding that makes fan-out safe.

## 1. Sub-agent prompt self-containment — 7 elements

A sub-agent starts with **zero** conversation context from the orchestrator. The same quality of output as in-session work requires all seven elements below to be present in the dispatch prompt.

```
1. Project absolute path
   e.g. <repo-root>

2. Data file absolute path + exact column / item name
   e.g. <repo-root>/data_<vendor>/<file>.CSV
        Item Name: '<exact field name>'

3. Data layout (encoding, header row, key cols, quirks)
   e.g. encoding='cp949', header=7, Item Name col index = 4,
        date columns from index 6, values are quoted thousand-separator
        strings ('353,000') that must be stripped and cast to float

4. Existing-pattern referent (point at a research/<existing-slug>/
   directory the sub-agent should mimic)
   e.g. follow research/<reference-slug>/ four-file structure:
        signal.py + run.py + NOTES.md + VERDICT.md

5. Deliverable spec (output paths and required format)
   e.g. research/<new-slug>/{signal.py, run.py, NOTES.md, VERDICT.md} —
        VERDICT.md MUST contain PROMOTE/REJECT/TRACKED verdict +
        Sharpe number + the 7-gate pass/fail table

6. Live-feasibility constraint (where applicable)
   e.g. "the historical-data vendor is backtest-only. Live signal
        production must be reproducible from the live broker API +
        public disclosure API alone. VERDICT.md's Live-feasibility
        section must list endpoint + sample call + measured latency."

7. CLAUDE.md reading instruction
   e.g. "Before starting, read <repo-root>/CLAUDE.md sections §1-§8
        (absolute rules), §3 (horizon constraints: intraday + max 5d),
        and §5 (7-gate promotion checklist). Self-REJECT if violated."
```

Omit any of the seven and you get output divergence, naming collisions, or rework. Treat the 7-element template as a brake — its only job is to make sub-agent outputs interchangeable.

## 2. Standard 4-agent fan-out launch pattern

```
Orchestrator session:
  1. Identify N work units (4 is the visual / managerial sweet spot)
  2. Verify independence (no shared file writes between sub-agents)
  3. Launch all N Task tool calls in a single message (foreground)
  4. Collect results, aggregate, update manifest, commit

Serial vs parallel:
  - 1.5-3h serial per work batch → ~30min parallel = 3.6x-6x speedup
  - At scale: dozens of orchestrated waves x 4 agents per wave
  - Measured: 4 agents on a single phase, ~3.6x wall-clock reduction
```

## 3. Atomic commit per phase

When all 4 sub-agent results are in, the orchestrator wraps them in a single commit. On interrupt, the last commit is the resume point.

```bash
# After collecting results from phase N
git add research/<slug_a>/ research/<slug_b>/ research/<slug_c>/ research/<slug_d>/
git add audits/<phase_N>/manifest.json audits/<phase_N>/checkpoint.json

# Commit message via .trash/ pattern (next section)
```

**Atomicity rules**:
- A sub-agent's status stays `in_progress` until the orchestrator commits
- Only the orchestrator flips status to `done` (after verifying outputs)
- Failed sub-agents are retried in a separate batch — successful ones are already committed

## 4. The `.trash/` pattern — `rm` deny rule bypass + HEREDOC quote-safe

Autonomous research repos commonly install a PreToolUse hook that blocks `rm`. The `.trash/` pattern handles both that constraint and the commit-message-apostrophe-corruption problem in one move.

```
1. Orchestrator writes the commit message into .git/COMMIT_MSG_TMP.txt
2. git commit -F .git/COMMIT_MSG_TMP.txt
3. mv .git/COMMIT_MSG_TMP.txt .trash/COMMIT_MSG_TMP_phase<N>.txt
   (mv bypasses rm-deny; .trash/ is in .gitignore)
```

This pattern becomes mandatory for any repo running an autonomous-overnight skill with `rm` denied.

## 5. Manifest + checkpoint resumability

Every fan-out batch leaves a resumable footprint. Reference layout:

```
audits/<date>-<task>/
  manifest.json     # task definitions + status (pending|in_progress|done|aborted)
  checkpoint.json   # incremental state per task + intermediate artifacts +
                    # resume cursor
  raw/<task_id>.md  # one atomic artifact per task
  FINAL.md          # written after aggregation
```

**Resume protocol**: a fresh session scans `manifest.json` and re-processes only `pending` and `in_progress` entries; `done` is skipped.

**Atomic-write rules**:
- Each sub-agent writes only its own file (`raw/<task_id>.md`)
- Only the orchestrator writes `manifest.json` (sub-agents read-only)
- Status flips to `done` only at task completion (mid-write = `in_progress`)

## 6. Dispatch and aggregation example

Illustrative single-phase fan-out (sanitized; actual signal names omitted):

```
Orchestrator launches (4 Task calls in one message):
  ├─ Task A (~29 min): "PIT loader build + re-validate a disclosure-text
  │                    drift signal under PIT universe
  │                    | full 7-element prompt
  │                    | output research/<slug_A>/PIT_VERDICT.md"
  ├─ Task B (~15 min): "Long-only price-extremes drift signal at
  │                    short horizon | output research/<slug_B>/*"
  ├─ Task C (~15 min): "Foreign-flow regression signal at short horizon
  │                    | output research/<slug_C>/*"
  └─ Task D (~24 min): "Pre-event drift signal | self-REJECT if
                       horizon constraint in CLAUDE.md §3 is violated"

Results collected:
  Task A: PIT_VERDICT.md → LS variant REJECT, LO variant HOLD (haircut)
  Task B: VERDICT.md     → PROMOTE provisional, mid-positive Sharpe
  Task C: VERDICT.md     → PROMOTE provisional, higher Sharpe; live
                          endpoint blocker documented
  Task D: VERDICT.md     → REJECT (20d horizon — CLAUDE.md §3 violation,
                          self-flagged)

Orchestrator aggregates:
  - manifest.json updated (all 4 tasks done)
  - audits/<date>/FINAL.md written
  - git add + commit -F .git/COMMIT_MSG_TMP.txt
  - mv .git/COMMIT_MSG_TMP.txt .trash/

Serial-equivalent wall clock: ~83 min. Parallel wall clock: ~29 min.
Measured speedup: ~3.6x.
```

The PROMOTE candidates produced here then enter the gate sequence in `research-promotion-gates.md`.

## 7. Anti-patterns

- **Concurrent writes to a shared file**: two sub-agents writing the same `manifest.json` clobber each other. Only the orchestrator writes shared state.
- **Orchestrator does the work itself**: running four signal backtests serially in the orchestrator session burns context and time. Always delegate.
- **Missing 7-element prompt**: "make me research/<slug>/" with no layout / encoding hints leads to silent data-load failures. Layout is mandatory.
- **Naming inconsistency**: one sub-agent emits `signal.py`, another `signal_<topic>.py` → import collisions downstream. Force file names in the prompt.
- **Sub-agents launching sub-agents**: the orchestrator is the only orchestrator. Sub-agents are leaf nodes.
- **Non-atomic commits**: committing 2 of 4 sub-agent outputs and pushing the others into the next phase blurs the phase boundary on interrupt. Commit all four together.

## 8. Audit checklist (target project)

- [ ] CLAUDE.md has a "Sub-agent / parallelism" section?
- [ ] Sub-agent prompt template includes all 7 elements?
- [ ] If autonomous mode is used, `.trash/` is registered in `.gitignore`?
- [ ] PreToolUse `rm` deny rule supports `.trash/` bypass?
- [ ] `manifest.json` + `checkpoint.json` schema is defined for the audits directory?
- [ ] Resume protocol re-processes only `pending` / `in_progress` tasks?
- [ ] If autonomous mode is used, the live-order path is explicitly excluded from fan-out (live mode runs serially under operator control)?
