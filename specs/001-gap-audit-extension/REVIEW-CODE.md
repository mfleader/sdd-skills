# Code Review: SDD Gap Audit Extension

**Spec:** specs/001-gap-audit-extension/spec.md
**Date:** 2026-04-27
**Reviewer:** Claude (speckit.spex-gates.review-code)

## Compliance Summary

**Overall Score: 100%**

- Functional Requirements: 24/24 (100%)
- Error Handling: 7/7 (100%)
- Edge Cases: 7/7 (100%)
- Non-Functional: N/A (prompt engineering artifact, no runtime NFRs)

## Detailed Review

### Functional Requirements

All 24 active FRs (FR-012 superseded) are compliant. The command file follows the canonical section order from contracts/command-schema.md exactly.

### Error Handling

All 7 error conditions from the command contract are implemented with matching error messages.

### Edge Cases

All 7 edge cases from spec.md are covered: missing spec.md, invalid focus, missing plan.md/tasks.md, empty/malformed gap-patterns.md, no findings, all-filtered findings, overwrite existing JSON.

### Extra Features (Not in Spec)

None identified.

---

## Code Review Guide (30 minutes)

> This section guides a code reviewer through the implementation changes,
> focusing on high-level questions that need human judgment.

**Changed files:** 4 source files (1 command markdown, 1 extension YAML, 1 README, 1 example patterns file), plus 1 SKILL.md (gitignored, installed locally), 1 test fixture

### Understanding the changes (8 min)

- Start with [speckit.gap-audit.audit.md](../../.specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md): This is the entire implementation. The 10 sections follow the [canonical order](contracts/command-schema.md) exactly. Reading top-to-bottom gives the full execution flow.
- Then [extension.yml](../../.specify/extensions/gap-audit/extension.yml): Minimal metadata. Verify command registration points to the correct file.
- Question: Is a single command file the right structure, or should the spec/plan prompts be split into separate files for maintainability?

### Key decisions that need your eyes (12 min)

**Subagent prompt is fully inline** ([audit.md Section 6](../../.specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md), relates to [FR-004](spec.md))

The entire 7-item spec checklist, 5-item plan checklist, 8 false positive filters, classification scheme, and output format are inlined in the command file rather than referenced from external files. This follows Constitution Principle V and research decision R6.
- Question: As the checklists evolve, will maintaining them inline become burdensome? Is the tradeoff (no drift risk vs. large file) acceptable?

**Filter 2 scope-conditional behavior** ([audit.md Section 6 line 200](../../.specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md), relates to [FR-006](spec.md))

Filter 2 (cross-reference tasks) is skipped in spec scope because tasks.md is not an input artifact. This behavior is in the implementation but not explicitly stated in the spec (the dogfood audit flagged this as a blocking finding).
- Question: Should this be added to the spec before shipping, or is it sufficiently implied by FR-002 (spec scope reads only spec.md)?

**Version check reads init-options.json** ([audit.md Section 1](../../.specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md), relates to [FR-025](spec.md))

The version check reads from `.specify/init-options.json` and proceeds with a warning if the file is absent. The spec does not specify the version source.
- Question: Is init-options.json the right source? Could speckit expose version differently in future versions?

### Areas where I'm less certain (5 min)

- [audit.md Section 8](../../.specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md) ([FR-020](spec.md)): The response parsing treats non-JSON and malformed JSON as empty arrays with warnings. The spec does not define this behavior (the dogfood audit flagged it). Is silently degrading to "no findings" the right choice, or should subagent failure be surfaced more prominently?
- [audit.md Section 6 plan scope](../../.specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md): The plan scope prompt says "All other sections remain the same" rather than repeating the full prompt. This relies on the executing agent understanding the diff-based instruction. An agent that reads literally might miss the false positive filters in plan scope.

### Deviations and risks (5 min)

- No deviations from [plan.md](plan.md) were identified. The implementation follows the project structure, file paths, and architecture exactly as planned.
- Risk: The command file is a prompt engineering artifact interpreted by an LLM. Its behavior depends on the executing model's ability to follow complex multi-section instructions. Testing with different models may produce different results.

---

## Deep Review Report

> Automated multi-perspective code review results. This section summarizes
> what was checked, what was found, and what remains for human review.

**Date:** 2026-04-27 | **Rounds:** 1/3 | **Gate:** PASS

### Review Agents

| Agent | Findings | Status |
|-------|----------|--------|
| Correctness | 4 | completed |
| Architecture & Idioms | 5 | completed |
| Security | 3 | completed |
| Production Readiness | 5 | completed |
| Test Quality | 7 | completed |
| CodeRabbit (external) | - | skipped (not installed) |
| Copilot (external) | - | skipped (not installed) |

### Findings Summary

| Severity | Found | Fixed | Remaining |
|----------|-------|-------|-----------|
| Critical | 2 | 2 | 0 |
| Important | 12 | 10 | 2 |
| Minor | 9 | - | 9 |

### What was fixed automatically

Fixed argument parsing to support spec directory path as second argument (correctness). Added explicit stop instruction for plan scope when files are missing (correctness + production readiness). Split spec directory resolution into interactive prompt and error fallback (correctness). Changed `disable-model-invocation` to `true` to match project convention (architecture). Registered extension in `.registry` (architecture). Added XML-style artifact delimiters and post-dispatch `git diff` verification for prompt injection mitigation (security). Distinguished subagent parse failures from genuine empty results in output (production readiness). Tightened JSON extraction heuristic with field validation (production readiness). Created answer key with 16 documented gaps for test fixture (test quality). Added naming collision, implicit assumption, and unverifiable SC coverage to test fixture (test quality).

### What still needs human attention

- The test fixture has no mechanism to deterministically test SC-007 (false positive filter demonstration). The filters run inside the non-deterministic subagent. Is this acceptable for the current scope, or should mock raw findings be created?
- No plan-focus test fixtures exist. Is this a follow-up task or required before shipping?

### Recommendation

All Critical and Important findings from the command file, SKILL.md, and registry were resolved. 2 Important findings remain in the test infrastructure (non-deterministic FP testing and missing plan fixtures). 9 Minor findings remain. See [review-findings.md](review-findings.md) for details. Code is ready for human review with the caveat that test fixture coverage could be expanded.
