# Backlog

Unspecced work items for future specs. Each entry becomes a spec when prioritized.

## Pending

### Gap-audit `--fix` flag

When `--fix` is passed to gap-audit and findings exist, check the extensions registry for the backtrace extension. If backtrace is registered and enabled, invoke it with the findings, spec directory, and scope. If backtrace is not installed, error with install instructions.

**Depends on:** spec 002 (backtrace extension)
**Origin:** brainstorm session 01 (backtrace-extension), 2026-04-28
