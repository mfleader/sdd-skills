# Brainstorm: Backtrace Extension

**Date:** 2026-04-28
**Status:** spec-created
**Spec:** specs/002-backtrace-extension/

## Problem Framing

Gap audits identify missing coverage in spec artifacts, but there's no automated way to fix those gaps. The backtrace skill exists in a development worktree but hasn't been packaged as a speckit extension. Users must manually trace findings back to spec items and edit artifacts by hand.

## Approaches Considered

### A: Standalone Extension
- Pros: Clean separation from gap-audit. Backtrace can accept findings from any source. Gap-audit stays focused and unchanged.
- Cons: Two extensions to install.

### B: Gap-Audit Subcommand
- Pros: Single extension, simpler install.
- Cons: Couples trace to audit. Can't use trace with findings from other sources. Breaks gap-audit's "no hooks" design.

### C: Merged Extension (New Name)
- Pros: Single coherent package.
- Cons: Breaking change for gap-audit users. Scope creep. Naming problem.

## Decision

Approach A: Standalone extension. Backtrace is conceptually different from auditing (it fixes, not finds). Separation means backtrace can accept findings from any source, and gap-audit doesn't need modification.

Key design decisions:
- One pass only, no iteration loop
- No JSON output persistence
- Auto-detect findings input (check spec dir for `.sdd-findings-{scope}.json`, fall back to asking user)
- Backtrace implements its own hook checking for `after_backtrace` hooks, following the plan/implement pattern
- Gap-audit `--fix` flag deferred to a separate spec (tracked in BACKLOG.md)

## Open Threads

- Should `after_backtrace` hooks fire even when no additions were applied?
- Gap-audit `--fix` flag needs its own spec (BACKLOG.md)
