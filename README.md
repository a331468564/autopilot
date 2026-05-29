# autopilot

Claude Code skill for universal project automation. Works in any project — scans the project, checks closure readiness, reports gaps, configures automation on confirmation, then executes.

## Installation

```bash
mkdir -p ~/.claude/skills/autopilot
curl -sSL https://raw.githubusercontent.com/a331468564/autopilot/main/SKILL.md \
  -o ~/.claude/skills/autopilot/SKILL.md
```

## Usage

```
/autopilot           → Full cycle: project intel → closure check → gap report → execute
/autopilot check     → Phase 1 + 2 only (no execution)
/autopilot 30m       → Execute every 30 minutes
/autopilot loop      → Adaptive interval
```

## Closure Score

6-dimension check: Runnable / Executable / Verifiable / Recoverable / Observable / Data-safe

- 6/6: Ready for automation
- 4-5/6: Can run with known risks
- < 4/6: Needs work first

## License

MIT
