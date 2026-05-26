# Brainstorm: Gap-Audit Optional tasks.md

**Date:** 2026-05-26
**Status:** active

## Problem Framing

The gap-audit `plan` scope hard-requires `tasks.md` alongside `spec.md` and `plan.md`. This creates a dependency inversion: users cannot audit their plan against the spec until they've generated tasks from it. The audit would be most useful *before* task generation, catching architectural gaps early.

The current design mirrors spex's `review-plan` gate, which also requires tasks.md. That's appropriate for `review-plan` (a readiness gate), but gap-audit is an adversarial probe with different timing needs.

## Approaches Considered

### A: Add a third scope (e.g., `plan-only`)
- Pros: clean separation, no behavior change to existing `plan` scope
- Cons: adds a new scope value, more surface area, users need to learn when to use `plan` vs `plan-only`

### B: Make tasks.md optional in existing `plan` scope (graceful degradation)
- Pros: no new scope names, backward compatible, single concept to learn
- Cons: `plan` scope behavior now varies based on artifact presence (implicit mode)

### C: Split into separate commands
- Pros: each command has a single responsibility
- Cons: proliferates commands, breaks existing workflows

## Decision

**Approach B: Graceful degradation.** The plan scope's 5 checklist categories split cleanly: categories 1-3 need tasks.md (contract tests, integration gaps, edge case coverage), categories 4-5 don't (implicit behavior, spec coverage gaps). When tasks.md is absent, run categories 4-5 only. When present, run all 5. No new scopes, no breaking changes.

Key design decisions:
- `categories_audited` field added per-finding (not a wrapper object) to maintain bare JSON array backward compatibility with backtrace and sdd-drive
- Gap-patterns filtered to active categories only in degraded mode
- Summary output indicates partial audit explicitly
- Backtrace plan scope still requires tasks.md (documented as known limitation, follow-up issue)

## Open Threads

- Backtrace plan scope needs a matching change to consume degraded-mode findings (follow-up issue)
