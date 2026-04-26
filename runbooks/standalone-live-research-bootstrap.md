# Standalone Live-Research Repo Bootstrap Runbook

Methodology runbook for the Day-0 bootstrap of a new live-trading research repository. Project-agnostic; adapted from internal practice across multiple live-research repos in different markets. Two independent repos following this pattern produced identical foundation curves over their first ~14-25 atomic build steps — strong signal that the structure is reusable.

Companion runbooks: see `research-promotion-gates.md` for the gate sequence each candidate signal must pass, and `multi-agent-fanout-runbook.md` for the parallel research pattern that becomes available once Day-0 is complete.

## 1. The standalone principle

External shared packages (a centralized "core" library, a shared cross-sectional factor platform, etc.) are **not imported**. Every market-specific fetcher, validation function, and risk module is **implemented inside the repo**.

**Why duplication is accepted**: strong isolation of the live path is preferred over code reuse. A validation bug in one repo must not auto-propagate into another. The cost is duplicated implementation; the benefit is independent blast radius.

## 2. Day-0 directory structure

```
<repo_root>/
  CLAUDE.md                  <= 200 lines (harness rule)
  .claude/
    settings.json            minimal MCP + PreToolUse hook
    skills/
      karpathy-guidelines/   required (link or vendor from workflow archive)
      overnight-autonomy/  required when autonomous mode is used
      drift-watchdog/        recommended for research repos
      abandon-criteria/      recommended for research repos
    hooks/
      block_live_writes.py   PreToolUse — denies LLM writes to live/
  shared/                    self-implemented (zero external imports)
    types.py
    validation.py
    data_io.py (or cache/, broker subpackage, etc.)
    risk.py
  research/                  R-mode free area; signal catalog
  live/                      L-mode serial; LLM writes blocked here
    .env.live                gitignored + hook-denied
    positions/               hook-denied
    orders/                  hook-denied
  audits/                    audit artifacts
  memory/                    auto-memory area (Claude-managed)
  CATALOGUE.md               (or hypothesis_sources.md) — hypothesis catalog
  targets.yaml entry         registered in the workflow archive
```

## 3. shared/ — the 4-module pattern

| Module | Responsibility | Core functions |
|---|---|---|
| `types.py` | Frozen dataclasses (CanonicalBar, Tick, Position, etc.) | dataclass definitions only |
| `validation.py` | IC-spurious + Sharpe + IC-PnL gap | `ic_spurious_check`, `sharpe_daily`, `pnl_ic_gap` |
| `data_io.py` (or `cache/<resol>.py` + `<broker>/rest.py`) | DuckDB cache + market fetcher | `warm_<universe>`, `get_<bars>` |
| `risk.py` | RiskLimits (Frozen) + ExchangeOutageGuard + DrawdownLadder | env-var override **forbidden** |

**Recommended naming for validation functions**:
- `calc_sharpe(daily_pnl)` — daily aggregation × √252. **Per-bar Sharpe is forbidden** (a known LOB pitfall is roughly 4,500x Sharpe over-statement when annualizing per-bar).
- `check_spurious_ic(ic_series, turnover)` — combined ADF + KPSS test plus turnover=0 red flag.
- `check_ic_pnl_gap(ic, pnl, cost_bps)` — break-even cost given trading cost and frequency assumptions.

Recommended extension modules (added once foundation phase is complete):
- `walkforward.py` — `walk_forward_eval(...)` (see `research-promotion-gates.md` §4)
- `perm_test.py` — `permute_block(block_size=24)` dispatcher (see `research-promotion-gates.md` §3)

## 4. .claude/settings.json — minimal MCP + hook

```json
{
  "mcp": {
    "filesystem": { "enabled": true },
    "duckdb":     { "enabled": true }
  },
  "deny": {
    "tools": ["WebSearch", "WebFetch"],
    "//": "research repo: internet blocked to prevent hallucinated references.
           When entering overnight mode, the overnight-autonomy skill
           grants temporary CRU + web access."
  },
  "hooks": {
    "PreToolUse": [
      { "match": "Write|Edit", "command": ".claude/hooks/block_live_writes.py" }
    ]
  }
}
```

Broker MCPs (e.g. CCXT-based crypto MCP, broker-REST MCPs) are **not enabled by default**. Enable them only inside the code-generation step that requires them, then disable. Minimal-MCP is a deliberate context-rot prevention measure.

## 5. PreToolUse hook — block_live_writes.py

```python
#!/usr/bin/env python3
import json, sys, re

DENY_PATTERNS = [
    r"(^|/)live/",
    r"(^|/)positions/",
    r"(^|/)orders/",
    r"\.env\.live$",
]

evt = json.load(sys.stdin)
path = evt.get("tool_input", {}).get("file_path", "")
if any(re.search(p, path) for p in DENY_PATTERNS):
    print(f"BLOCKED: {path} (live path, LLM write denied)", file=sys.stderr)
    sys.exit(2)  # exit 2 = block
sys.exit(0)
```

**LLM role boundary**: research/ is fully open to the LLM; the live order-execution path has zero LLM writes. Operator edits the live path directly. This is enforced by the hook, not by convention.

## 6. Required skills

| Skill | Source | Scope |
|---|---|---|
| `karpathy-guidelines` | workflow archive | required for every new repo (Think Before Coding / Simplicity First / Surgical Changes / Goal-Driven Execution) |
| `overnight-autonomy` | workflow archive | required when overnight autonomous mode is used (grants temporary CRU + web) |
| `drift-watchdog` | workflow archive | recommended for research repos (detects scale / scope / metric drift > 2x) |
| `abandon-criteria` | workflow archive | recommended for research repos (forces declaration of >= 3 kill triggers per phase) |

Installation = drop `SKILL.md` under `.claude/skills/<name>/`, then cross-link from CLAUDE.md.

## 7. First fetcher + hypothesis catalog

Immediately after Day-0:

1. **One market-specific fetcher.** Examples (broker / vendor names are publicly known and given here only as illustration of *categories*, not endorsements): a public crypto data fetcher built on `ccxt`; a regional broker REST adapter in paper-trading mode; an institutional-API wrapper; a vendor CSV ingester. Pick one for the first market and implement it in `shared/`.
2. **One hypothesis catalog.** `CATALOGUE.md` or `hypothesis_sources.md` enumerating 9-15 candidate hypotheses, each as a 4-tuple: `(mechanism, horizon, universe, cost_assumption)`.

Do not start coding signals before the catalog exists. The act of enumerating is itself the first wide-vs-deep decision.

## 8. R-to-L 7-gate checklist (CLAUDE.md §5)

CLAUDE.md §5 must enumerate the promotion gates as binary checkboxes:

```markdown
## §5 R-to-L promotion checklist
- [ ] In-house IC spurious check (ADF + KPSS + turnover=0) passed
- [ ] block24-perm p < 0.05 (Bonferroni alpha/cells when multiple cells)
- [ ] Walk-forward sign-stability >= 3/4 folds (`research-promotion-gates.md` §4)
- [ ] No cliff inside +/- 5 bps cost sweep
- [ ] Live-feasibility (broker endpoint + sample call + measured latency)
- [ ] (single-side market) LS-vs-LO asymmetry check passed
- [ ] Kill-switch + position-limit code + open/close scenario tests
```

The full gate definitions and supporting tooling are in `research-promotion-gates.md`.

## 9. targets.yaml registration

Register the new repo in the workflow archive's `targets.yaml`:

```yaml
<repo_name>:
  path: <absolute path to repo>
  type: live-research
  template: templates/live-research-CLAUDE.md.template
  status: planned         # flip to active on the first audit
  last_audit: null
  promotable_count: 0
  archived_nulls: 0
```

## 10. Manual-to-auto progression — transition condition

Two independent live-research repos showed the same transition timing pattern:

- **After ~14-25 atomic foundation steps**, auto-progression becomes safe
- Required brakes before auto-progression: (a) atomic-commit-per-phase, (b) a memory note (`auto_wave_progression.md` or equivalent) so a fresh session can resume immediately
- Auto entry requires: `overnight-autonomy` skill active + manifest+checkpoint infrastructure in place + sub-agent fan-out 7-element prompt template established (see `multi-agent-fanout-runbook.md`)

Skipping these brakes and entering auto-mode early reliably produces the failure cases documented in the retrospectives at `../retrospectives/` — see in particular the sacrificial-pilot reference case.

## 11. Audit checklist (Day-0 → Day-1 entry)

- [ ] CLAUDE.md <= 200 lines?
- [ ] Standalone principle + zero-external-imports stated explicitly?
- [ ] `shared/` 4-module pattern self-implemented (validation, risk, types, data_io|cache|broker)?
- [ ] `.claude/settings.json` — minimal MCP (Filesystem + DuckDB only) with rationale comment?
- [ ] PreToolUse hook `block_live_writes.py` verified to block (test with a dummy live/ write)?
- [ ] `live/`, `positions/`, `orders/`, `.env.live` denied in both `.gitignore` and the hook?
- [ ] `karpathy-guidelines` skill installed?
- [ ] (autonomous mode) `overnight-autonomy` skill installed + `.trash/` pattern in place?
- [ ] (research) `drift-watchdog` + `abandon-criteria` installed?
- [ ] Hypothesis catalog file exists with >= 9 hypotheses enumerated?
- [ ] CLAUDE.md §5 promotion checklist matches the 7 gates in `research-promotion-gates.md`?
- [ ] `targets.yaml` entry registered?
- [ ] LLM role boundary explicit — research free, live LLM-write blocked?
