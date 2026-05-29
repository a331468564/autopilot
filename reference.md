<!-- DOC_META
lifecycle:  long-term
audience:   agent
write_when: Phase details or formats change
read_when:  Need detailed autopilot reference
delete_when: Never
-->

# Autopilot — Detailed Reference

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
   - Check if `.venv` exists

4. Identify data flow:
   - Input files: `data/*.csv`, `data/*.json`, `*.db`
   - Processing: which scripts read which inputs
   - Output files: `reports/`, `exports/`, `data/output*`
   - State files: `docs/current-progress.md`, `*.log`, `*_metrics.json`

5. Check existing automation:
   - `.claude/hooks/*.py` — list and describe each hook
   - `.github/workflows/` — CI/CD
   - `cron` / scheduled tasks

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

### Existing Automation
- {type}: {description}
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

**Run report format:**

```
## Autopilot Run Report — {date} {time}

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
