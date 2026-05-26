# Review Guide: Gap Audit Optional tasks.md

**Spec:** [spec.md](spec.md) | **Plan:** [plan.md](plan.md) | **Tasks:** [tasks.md](tasks.md)
**Generated:** 2026-05-26

---

## What This Spec Does

Makes the gap-audit extension's `plan` scope work without tasks.md. Today, running `/speckit.gap-audit.audit plan` requires spec.md, plan.md, and tasks.md. After this change, tasks.md becomes optional: when absent, the auditor runs categories 4-5 only (implicit behavior, spec coverage gaps). When present, all 5 categories run as before. This lets users catch plan-level gaps before investing in task generation.

**In scope:** Command file changes (Sections 4, 6, 8, 9, 10), new `categories_audited` finding field, version bump, original spec/doc updates.

**Out of scope:** Backtrace extension changes (it still requires tasks.md for plan scope). This is a documented follow-up.

## Bigger Picture

This is the fifth spec in the sdd-skills project, which provides adversarial quality extensions for the speckit/spex SDD workflow. The gap-audit extension was built in [spec 001](../001-gap-audit-extension/spec.md), and this change addresses a workflow limitation discovered during use: plan-level gaps can't be caught until after tasks exist, but tasks are generated from the plan. The backtrace extension ([spec 002](../002-backtrace-extension/spec.md)) also has plan scope that requires tasks.md and will need a matching change eventually.

The upstream spex project's `review-plan` gate requires tasks.md by design (it's a readiness gate). This extension diverges intentionally because adversarial auditing serves a different purpose than readiness checks.

---

## Spec Review Guide (30 minutes)

### Understanding the approach (8 min)

Read the [User Story 1 acceptance scenarios](spec.md#user-story-1---audit-plan-against-spec-before-task-generation-priority-p1) and the [edge cases](spec.md#edge-cases) section. As you read:

- Does graceful degradation (same scope name, different behavior based on artifact presence) feel intuitive, or would a separate scope name be clearer for users?
- The [info message](spec.md#functional-requirements) (FR-004) is emitted "before dispatching the subagent." Is this the right timing, or should it appear earlier (during artifact loading)?

### Key decisions that need your eyes (12 min)

**Per-finding `categories_audited` vs wrapper object** ([FR-013](spec.md#functional-requirements), [Assumptions](spec.md#assumptions))

The spec adds `categories_audited` to each finding object rather than wrapping findings in a metadata envelope. The rationale is backward compatibility: backtrace and sdd-drive expect bare JSON arrays. The trade-off is denormalization (every finding in the same file repeats the same list).

- Is the denormalization acceptable, or should we consider a wrapper with a version field to enable future schema evolution?

**Host-side category validation** ([FR-017](spec.md#functional-requirements))

The spec requires both prompt-level (FR-007) and post-parse (FR-017) category restriction. The prompt tells the subagent which categories are valid, but since LLMs are non-deterministic, FR-017 catches violations after the fact.

- Is silent rejection of out-of-category findings correct, or should these be logged as warnings?

**"No issues found" without partial qualifier** ([FR-018](spec.md#functional-requirements), [US1 AC-3](spec.md#user-story-1---audit-plan-against-spec-before-task-generation-priority-p1))

When degraded mode finds no gaps, the output says "No issues found." without indicating that only 2 of 5 categories were checked. The rationale: a clean result needs no qualifier. But a user might not remember they're in degraded mode.

- Should the clean output also say "(categories 4-5 only)" to prevent false confidence?

### Areas where I'm less certain (5 min)

- [FR-012](spec.md#functional-requirements): Adding `categories_audited` to spec mode output is scope expansion beyond the stated problem (plan scope). The Assumptions section explains this is for consistency, but it means every existing spec-scope audit will now produce findings with a new field. Is this the right time to make that change?

- [Plan design decision D3](plan.md#design-decisions): T002-T010 are nine sequential edits to one file. In practice, the implementer will likely make these changes in a single editing pass. Should these be consolidated into fewer tasks?

### Risks and open questions (5 min)

- The backtrace follow-up is documented but not tracked. If a user runs degraded gap-audit, gets findings, then tries `backtrace plan`, it fails. Should the gap-audit info message warn about this? ("Note: backtrace plan scope still requires tasks.md.")
- [SC-001](spec.md#measurable-outcomes) was reframed from "identical findings" to "same prompt structure and filter logic." This is more realistic for LLM subagents, but it means there's no way to regression-test output correctness, only structural equivalence. Is that sufficient?

---

*Full context in linked [spec](spec.md) and [plan](plan.md).*
