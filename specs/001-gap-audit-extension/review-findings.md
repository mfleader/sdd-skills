# Deep Review Findings

**Date:** 2026-04-27
**Branch:** 001-gap-audit-extension
**Rounds:** 1
**Gate Outcome:** PASS (after fixes)
**Invocation:** quality-gate

## Summary

| Severity | Found | Fixed | Remaining |
|----------|-------|-------|-----------|
| Critical | 2 | 2 | 0 |
| Important | 12 | 10 | 2 |
| Minor | 9 | - | 9 |
| **Total** | **23** | **12** | **11** |

**Agents completed:** 5/5 (+ 0 external tools)
**Agents failed:** none

## Fixed Findings

### Correctness: Argument path parsing unreachable (Important, fixed)
- **File:** .specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md:28-36
- **Source:** correctness-agent
- **What was wrong:** Section 2 treated all remaining text after --output as scope, making FR-024 step 1 (argument path) unreachable.
- **How resolved:** Added multi-token parsing: first token = scope, second token = spec directory path.

### Correctness: Plan scope missing stop instruction (Important, fixed)
- **File:** .specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md:69-84
- **Source:** correctness-agent (also reported by: production-readiness-agent)
- **What was wrong:** Plan scope reported missing files but had no explicit stop instruction before the read step.
- **How resolved:** Added "If any files are missing, stop. Do not proceed to subsequent sections."

### Correctness: Spec dir resolution conflates ask with error (Important, fixed)
- **File:** .specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md:55
- **Source:** correctness-agent
- **What was wrong:** FR-024 step 3 "ask the user" was implemented as an error message instead of an interactive prompt.
- **How resolved:** Split into interactive prompt followed by error fallback.

### Architecture: disable-model-invocation convention break (Important, fixed)
- **File:** .claude/skills/speckit-gap-audit-audit/SKILL.md:9
- **Source:** architecture-agent
- **What was wrong:** Set to `false`, breaking convention with all 34 other skills in the project.
- **How resolved:** Changed to `true`.

### Architecture: Missing registry entry (Important, fixed)
- **File:** .specify/extensions/.registry
- **Source:** architecture-agent
- **What was wrong:** gap-audit extension was not registered, unlike all other extensions.
- **How resolved:** Added registry entry with version 0.1.0, enabled, single command.

### Security: Prompt injection from embedded artifacts (Important, fixed)
- **File:** .specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md:125-135, 255-273
- **Source:** security-agent
- **What was wrong:** Artifact content embedded verbatim without structural separation. Subagent has file write access.
- **How resolved:** Added XML-style `<artifact>` delimiters and post-dispatch `git diff` verification.

### Production Readiness: Silent degradation to "No issues found" (Important, fixed)
- **File:** .specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md:389-399
- **Source:** production-readiness-agent
- **What was wrong:** Subagent failure silently produced "No issues found" output.
- **How resolved:** Added PARSE_FAILED flag with distinct message for parse failures vs genuine empty results.

### Production Readiness: Brittle JSON extraction (Important, fixed)
- **File:** .specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md:333
- **Source:** production-readiness-agent
- **What was wrong:** JSON extraction matched any `[`/`]` pair, risking false extraction from prose.
- **How resolved:** Tightened to line-start `[` and validate first element has required GapFinding fields.

### Test Quality: No answer key for test fixture (Critical, fixed)
- **File:** specs/001-gap-audit-extension/test-fixtures/
- **Source:** test-quality-agent
- **What was wrong:** No ground truth document for measuring SC-001 recall and SC-002 precision.
- **How resolved:** Created spec-with-gaps.answer-key.md with 16 documented gaps across all 7 categories.

### Test Quality: No naming collision coverage (Critical, fixed)
- **File:** specs/001-gap-audit-extension/test-fixtures/spec-with-gaps.md
- **Source:** test-quality-agent
- **What was wrong:** Category 6 (naming collisions) was structurally untestable.
- **How resolved:** Added G13 (within-spec naming collision on "metrics" term) to fixture and answer key.

### Test Quality: Fixture improvements (Important, fixed)
- **File:** specs/001-gap-audit-extension/test-fixtures/spec-with-gaps.md
- **Source:** test-quality-agent (FINDING-7, 12, 14)
- **What was wrong:** Ambiguous orphan FR, thin implicit assumption coverage, only one unverifiable SC.
- **How resolved:** Added stale data warning (unambiguous orphan FR), FR-006 timezone (implicit assumption), SC-004 "good UX" (second unverifiable SC).

## Remaining Findings

### Important (2 remaining)

**Test Quality: No mechanism to test SC-007 deterministically**
SC-007 (FP filter demo) cannot be tested deterministically because filters run inside the non-deterministic subagent. Would require mock raw findings as a separate test artifact.

**Test Quality: No plan-focus test fixtures**
The test fixture only covers spec-focus audit. Plan-focus audit (US2) has no test fixtures with planted gaps across the 5 plan checklist categories.

### Minor (9 remaining)

- Correctness: Redundant filter 2 adjustment note (line 298)
- Architecture: README.md inconsistency with peer extensions
- Architecture: gap-patterns path hardcoded to specs/
- Architecture: code-reviewer subagent type choice undocumented
- Security: No path validation on user-supplied spec directory
- Security: Unconditional file overwrite without symlink check
- Production Readiness: No action on JSON_VALID=false
- Production Readiness: No bash snippet for semver comparison
- Test Quality: No plan-focus test fixture
