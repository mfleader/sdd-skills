# Brainstorm: Exploratory Test Extension

**Date:** 2026-04-30
**Status:** spec-created
**Spec:** specs/004-exploratory-test-extension/

## Problem Framing

The exploratory test skill exists as a standalone Claude Code skill in the wh40k project. It dispatches an adversarial subagent that writes and runs tests against an implementation to find edge cases the spec missed. The skill needs to be packaged as a speckit extension so it can be installed and invoked consistently across projects, integrated into the SDD feedback loop (findings → collector → patterns → new specs).

## Approaches Considered

### A: Mechanical repackaging (chosen)

Transfer the source skill's behavior verbatim into speckit extension format, adding only speckit boilerplate (version check, spec dir resolution, integrity check, defensive parsing).

- Pros: Lowest risk, behavioral parity verifiable via checklist, no design decisions beyond packaging
- Cons: Can't improve the skill during packaging, any source skill quirks transfer verbatim

### B: Redesign as report-only

Strip the active testing (writing/running tests) and make it report-only like gap-audit. Subagent reads code and spec, reports potential issues without execution.

- Pros: Consistent with gap-audit/backtrace pattern, no side effects
- Cons: Loses execution grounding (the whole point of exploratory testing), overlaps with gap-audit

### C: Hybrid (write tests, don't run project suite)

Keep test writing and execution but remove project test suite regression checking.

- Pros: Execution-grounded findings without full test suite side effects
- Cons: Loses regression checking, partial value

## Decision

**Approach A: Mechanical repackaging.** The source skill's active testing behavior is what differentiates it from gap-audit. Removing or modifying it would reduce the extension's value. SC-004 explicitly constrains against behavioral additions.

Key decisions resolved during brainstorming:
- **Command name:** `speckit.exploratory-test.test` (follows `<technique>.<root-verb>` convention, auditor-confirmed)
- **Schema:** Keep exploratory-specific fields (severity/reproduction/expected/actual), don't align with gap-audit. Source field routes findings.
- **Scope field:** Intentionally omitted (no scope concept). Backtrace falls back to its own scope argument.
- **Test mode:** Active testing preserved (write tests, run them, run project suite)

## Open Threads

- Backtrace cannot currently consume exploratory findings due to schema incompatibility (GapFinding vs ExploratoryFinding). Follow-up task needed on backtrace to make its parser schema-aware.
- Pattern catalog divergence: defect-catalog.md (per-feature) vs gap-patterns.md (project-wide). Unification is future work.
