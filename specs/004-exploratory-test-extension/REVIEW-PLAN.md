# Review Guide: Exploratory Test Extension

**Spec:** [spec.md](spec.md) | **Plan:** [plan.md](plan.md) | **Tasks:** [tasks.md](tasks.md)
**Generated:** 2026-04-30

---

## What This Spec Does

Packages an existing exploratory test skill as a speckit extension. The skill dispatches an adversarial subagent that reads a feature's spec, writes tests against the implementation, runs them, and reports what broke. Unlike gap-audit (static spec analysis), this extension exercises running code.

**In scope:** Extension packaging (extension.yml, command markdown, README), speckit boilerplate (version check, spec dir resolution, integrity check, defensive parsing), behavioral parity with the source skill.

**Out of scope:** New features beyond the source skill, schema alignment with gap-audit, lifecycle hooks, automated tests (this is a prompt-engineering artifact).

## Bigger Picture

This is the third finding-producing extension in the SDD tooling ecosystem, joining gap-audit (spec quality) and backtrace (tracing gaps to spec additions). Together they form a feedback loop: skills produce findings, the pattern collector aggregates them into recurring patterns, and patterns inject avoidance directives into new specs.

The exploratory test fills a gap the other two can't: execution-grounded testing. Gap-audit reasons about what's missing from the spec. Backtrace proposes spec additions for gaps found. Exploratory test exercises the actual implementation to find bugs neither static analysis nor spec review would catch. The three extensions cover the full quality spectrum: spec quality, spec completeness, and implementation correctness.

Downstream, the `sdd_collect_patterns.py` collector already knows how to read `.*-findings.json` files and route by `source` field. The `sdd-drive` orchestrator reads findings to make flow control decisions. Both will pick up exploratory findings automatically once the extension writes `.exploratory-findings.json` with `"source": "exploratory"`.

---

## Spec Review Guide (30 minutes)

> Focus your 30 minutes on the parts that need human judgment most.

### Understanding the approach (8 min)

Read [User Story 1](spec.md#user-story-1---run-exploratory-tests-against-implementation-priority-p1) and the [Implementation Approach](plan.md#implementation-approach) table for the core design. As you read, consider:

- Does the 11-section command structure (modeled after gap-audit) feel right for a skill that actively runs tests, or does the active-testing nature warrant structural differences?
- The spec constrains this to a mechanical repackaging ([SC-004](spec.md#measurable-outcomes)). Is that constraint too tight? Are there obvious improvements that should be made during packaging?
- The source skill uses absolute paths (`/tmp/exploratory-tests/`). Is this portable enough for all environments where the extension might run?

### Key decisions that need your eyes (12 min)

**Active testing preserved** ([spec.md US1 AC1](spec.md#user-story-1---run-exploratory-tests-against-implementation-priority-p1))

The extension preserves the source skill's active testing behavior: the subagent writes test files and runs them, plus runs the project's test suite. This is fundamentally different from gap-audit/backtrace (report-only). The integrity check ([FR-012](spec.md#functional-requirements)) only catches repo modifications, not side effects from running tests.
- Question for reviewer: Are there environments where running the project's test suite from a subagent could cause problems (destructive tests, database mutations, external API calls)?

**Schema divergence from gap-audit** ([spec.md Key Entities](spec.md#key-entities))

ExploratoryFinding uses severity/description/reproduction/expected/actual/spec_gap. GapFinding uses classification/category/evidence/suggested_fix. These are deliberately different. The `source` field is the only shared routing key.
- Question for reviewer: When backtrace consumes exploratory findings, it needs to map from the exploratory schema to ProposedAdditions. The spec documents that backtrace falls back when `scope` is absent. Has anyone verified that backtrace actually handles the exploratory schema's different fields (reproduction/expected/actual instead of evidence/suggested_fix)?

**Scope field intentionally omitted** ([spec.md Assumptions](spec.md#assumptions))

Gap-audit findings include both `source` and `scope`. Exploratory findings include only `source`. The spec documents this as intentional (no scope concept) and says backtrace falls back to its own scope argument.
- Question for reviewer: Is this documented in backtrace's spec or just assumed? If backtrace's behavior changes, exploratory findings could silently break.

### Areas where I'm less certain (5 min)

- [FR-002 --output constraint](spec.md#functional-requirements): The spec says when `--output` is passed, the ask-the-user fallback is disabled. This came from the source skill ("the spec directory path must be unambiguous"). But the gap-audit extension doesn't have this constraint. Is this a meaningful safety measure or an unnecessary divergence from the gap-audit pattern? It was flagged by the gap auditor and fixed, but I'm not confident it adds real value.

- [Edge Cases](spec.md#edge-cases): ~~Resolved.~~ The spec now specifies "overwrite the existing directory contents" for `/tmp/exploratory-tests/`. The disjunction was resolved during the spec audit cycle.

- [plan.md Section-by-Section Mapping](plan.md#section-by-section-mapping): The mapping table shows Section 5 (Changed File Discovery) as "new section" with no gap-audit equivalent. This is the only section where the implementer can't copy structure from an existing extension. It's straightforward (`git diff` + branch validation), but it's the one place where the "just model after gap-audit" instruction doesn't apply.

### Risks and open questions (5 min)

- The subagent writes tests to `/tmp/exploratory-tests/` and runs the project's test suite. If a test suite has side effects (sends emails, modifies a database, calls external APIs), the exploratory test could trigger those side effects. The spec has no guardrail for this. Is a warning in the README sufficient, or should the prompt instruct the subagent to check for destructive test behaviors?

- The behavioral parity checklist has 33 items ([SC-001](spec.md#behavioral-parity-checklist)). Who runs this checklist? If the implementer runs it on their own work, it's a self-review. If a reviewer runs it, it's time-consuming. Should T015 specify that a subagent runs the checklist?

- The source skill reference is an absolute path on one developer's machine (`/home/mleader/workspace/py/warhammer/wh40k/...`). If someone else implements this, they won't have that path. Should the source skill content be copied into the spec directory or the extension itself?

---
*Full context in linked [spec](spec.md) and [plan](plan.md).*
