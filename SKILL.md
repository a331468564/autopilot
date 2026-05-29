---
name: autopilot
description: >
  Use when the user says "/autopilot", "automate this", "run the project",
  "check closure", or "/autopilot 30m". Orchestrates any project through
  4 phases: project intelligence, closure check, gap report, execute.
when_to_use: >
  Triggers: "/autopilot", "automate this", "run the project", "check closure",
  "/autopilot 30m", "/autopilot loop", "is this project ready to run".
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - Agent
  - WebFetch
  - WebSearch
---

<!-- DOC_META
lifecycle:  long-term
audience:   agent
write_when: Automation phases or rules change
read_when:  Invoking /autopilot
delete_when: Never
-->

# Autopilot — Universal Project Automation

Works in any project. No project-specific logic is hardcoded. Everything is discovered at runtime.

## Invocation

```
/autopilot           → Full cycle: Phase 1 → 2 → 3 → (user confirms) → 4
/autopilot check     → Phase 1 + 2 only (no execution)
/autopilot 30m       → Execute every 30 minutes (skip Phase 1-2 after first run)
/autopilot loop      → Adaptive interval (succeeds → longer, fails → shorter)
```

Arguments: `$ARGUMENTS` is passed as the execution target. If empty, auto-detect from project.

## Phases

| Phase | What | Auto? |
|-------|------|-------|
| 1. Project Intelligence | Scan entry files, scripts, dependencies, data flow | Yes |
| 2. Closure Check | Score on 6 dimensions | Yes |
| 3. Gap Report | List missing components, ask user what to configure | Yes (asks before configuring) |
| 4. Execute | Run the project pipeline, generate report | After user confirms |

## Closure Score

6 dimensions: Runnable / Executable / Verifiable / Recoverable / Observable / Data-safe

- 6/6: Ready for automation
- 4-5/6: Can run with known risks
- < 4/6: Needs work first

## Safety Rules

1. Never modify data without backup
2. Never delete master data — append-only
3. Never commit credentials
4. Never retry destructive operations
5. Always report errors
6. Respect existing hooks

## Detailed Reference

For phase details, scheduling logic, output formats, and project-specific adaptation patterns, see [reference.md](reference.md).
