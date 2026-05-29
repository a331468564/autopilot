---
name: autopilot
description: >
  Universal project automation orchestrator. Works in any project.
  Scans the project, checks closure readiness, reports gaps, configures
  automation on confirmation, then executes. Use when the user says
  "/autopilot", "automate this", "run the project", or "/autopilot 30m".
---

<!-- DOC_META
lifecycle:  long-term
audience:   agent
write_when: Automation logic or phases change
read_when:  Invoking /autopilot
delete_when: Never
-->

# Auto — Universal Project Automation

## Overview

This skill automates any project through 4 phases:

1. **Project Intelligence** — Understand what the project is and how it runs
2. **Closure Check** — Can this project run end-to-end autonomously?
3. **Gap Report** — What's missing? Ask user what to configure.
4. **Execute** — Run the project pipeline, generate report.

**No project-specific logic is hardcoded.** Everything is discovered at runtime.

## Invocation

```
/autopilot           → Full cycle: Phase 1 → 2 → 3 → (user confirms) → 4
/autopilot check     → Phase 1 + 2 only (no execution)
/autopilot 30m       → Execute every 30 minutes (skip Phase 1-2 after first run)
/autopilot loop      → Adaptive interval (succeeds → longer, fails → shorter)
```

## Phase 1: Project Intelligence

**Goal:** Build a project profile in ~1-2 minutes.

**Steps:**

1. Read entry files (first 50 lines each):
   - `README.md`, `CLAUDE.md`, `AGENTS.md`
   - `package.json`, `requirements.txt`, `pyproject.toml`, `Makefile`, `Dockerfile`
   - `.claude/settings.json`, `.claude/settings.local.json`

2. Scan `scripts/` directory:
   - List all `.py`, `.sh`, `.js` files
   - Run `--help` on each to identify purpose and arguments
   - Categorize: data pipeline / extraction / reporting / utility

3. Check dependencies:
   - Python: `pip list 2>/dev/null | wc -l` and `pip check 2>/dev/null`
   - Node: `npm ls --depth=0 2>/dev/null`
   - Check if `.venv` exists and is activated

4. Identify data flow:
   - Input files: `data/*.csv`, `data/*.json`, `*.db`
   - Processing: which scripts read which inputs
   - Output files: `reports/`, `exports/`, `data/output*`
   - State files: `docs/current-progress.md`, `*.log`, `*_metrics.json`

5. Check existing automation:
   - `.claude/hooks/*.py` — list and describe each hook
   - `.github/workflows/` — CI/CD
   - `cron` / scheduled tasks
   - `/loop` or cron hooks in settings

**Output format:**

```
## Project Profile

| Field | Value |
|-------|-------|
| Name | {from README or directory name} |
| Type | {Python CLI / Node API / Data Pipeline / Frontend / etc.} |
| Entry | {main command to run} |
| Language | {Python 3.x / Node 20 / etc.} |
| Dependencies | {N packages, status: OK / broken} |

### Data Flow
{input} → {processor} → {output}

### Sub-processes
1. {name}: {description} — `command`
2. {name}: {description} — `command`
...

### Existing Automation
- {type}: {description}
...
```

## Phase 2: Closure Check

**Goal:** Score the project on 6 dimensions.

| Check | Pass Criteria | How to Verify |
|-------|--------------|---------------|
| Runnable | Clear entry command exists and executes | Run entry with `--help` or `--dry-run` |
| Executable | Dependencies installed, no import errors | `pip check` / `npm ls` / test import |
| Verifiable | Has tests, dry-run, or success criteria | Look for `tests/`, `--dry-run`, exit codes |
| Recoverable | Has error handling, retry, or rollback | Check for try/except, backup files, git |
| Observable | Has logs, reports, or progress tracking | Check for log files, progress docs, metrics |
| Data-safe | Has backups, append-only, or snapshots | Check for `.bak` files, git commits, hooks |

**Scoring:**
- 6/6: Fully closed-loop — ready for automation
- 4-5/6: Mostly closed — can run with known risks
- 2-3/6: Partial — needs work before automation
- 0-1/6: Not ready — major gaps

**Output format:**

```
## Closure Score: {N}/6

| Check | Status | Evidence |
|-------|--------|----------|
| Runnable | ✅/✗ | {evidence} |
| Executable | ✅/✗ | {evidence} |
| Verifiable | ✅/✗ | {evidence} |
| Recoverable | ✅/✗ | {evidence} |
| Observable | ✅/✗ | {evidence} |
| Data-safe | ✅/✗ | {evidence} |
```

## Phase 3: Gap Report

**Goal:** List gaps with actionable suggestions. Ask user what to configure.

For each gap found:

```
## Gap: {check name}

**Problem:** {what's missing}

**Suggestion:**
- {specific action 1}
- {specific action 2}

**Effort:** {low / medium / high}
```

After listing all gaps, ask the user:

```
发现 {N} 个缺口。要我配置以下哪些？

1. {gap 1 suggestion — effort}
2. {gap 2 suggestion — effort}
3. {gap 3 suggestion — effort}

[全部配置] [选择性配置] [跳过，直接执行] [取消]
```

**Configuration rules:**
- Ask before configuring anything that touches security (API keys, credentials, external services)
- Ask before configuring anything that modifies data flow (new scripts, changed entry points)
- For standard infrastructure (hooks, tests, backups, progress tracking): configure on user confirmation

## Phase 4: Execute

**Goal:** Run the project pipeline once, generate report.

**Pre-conditions:**
- Closure score >= 4/6, OR user explicitly confirmed to proceed despite risks
- Phase 3 gaps either resolved or user chose to skip

**Execution strategy:**

1. Determine execution command from Phase 1 project profile
2. If project has sub-processes, run them in dependency order
3. Capture exit code, stdout, stderr
4. If exit code != 0:
   - Log error
   - If retryable (network timeout, etc.): retry once
   - If not retryable: report and stop
5. Generate run report

**Run report:**

```
## Auto Run Report — {date} {time}

| Metric | Value |
|--------|-------|
| Duration | {seconds}s |
| Exit code | {code} |
| Sub-processes | {N ran / N total} |
| New data | {rows added / files created} |
| Errors | {N} |

### Details
{per-subprocess results}
```

## Scheduling

### `/autopilot 30m`

Run every 30 minutes. After first run:
- Cache Phase 1 project profile (don't re-scan)
- Skip Phase 2-3 (closure already checked)
- Go directly to Phase 4

### `/autopilot loop`

Adaptive scheduling:
- After success: next run in {interval * 2} (max 2 hours)
- After failure: next run in {interval / 2} (min 5 minutes)
- 3 consecutive failures: stop and notify user
- Starting interval: 30 minutes

## Safety Rules

1. **Never modify data without backup** — always backup before writes
2. **Never delete master data** — append-only for input/output CSVs
3. **Never commit credentials** — check .gitignore before commit
4. **Never retry destructive operations** — if a write fails, don't retry
5. **Always report errors** — never silently swallow exceptions
6. **Respect existing hooks** — if pre_bash_safety.py blocks something, don't work around it

## Project-Specific Adaptation

This skill discovers project structure at runtime. Examples:

**Python data pipeline:**
- Entry: `python -m scripts.main`
- Sub-processes: scripts in `scripts/` with `--help`
- Data: `data/*.csv`
- Report: `scripts/reports/generate_run_report.py`

**Node.js API:**
- Entry: `npm start`
- Sub-processes: `npm run build`, `npm test`
- Data: database or API responses
- Report: test results + coverage

**Frontend:**
- Entry: `npm run dev`
- Sub-processes: `npm run build`, `npm run lint`
- Data: source files
- Report: build status + lint results

The skill adapts to whatever it finds. No hardcoded project logic.
