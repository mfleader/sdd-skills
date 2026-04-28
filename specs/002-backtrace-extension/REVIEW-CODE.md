# Code Review: Backtrace Extension

**Spec:** [spec.md](spec.md)
**Date:** 2026-04-28
**Reviewer:** Claude (speckit.spex-gates.review-code)

## Compliance Summary

**Overall Score: 100%** (after fixes)

- Functional Requirements: 17/17 (100%)
- Error Handling: 8/8 (100%)
- Edge Cases: 5/5 (100%)
- Success Criteria: 8/8 (100%)
- User Story Acceptance Scenarios: 11/11 (100%)

## Deviations Fixed During Review

1. **FR-016/SC-008**: Added `specify extension add backtrace --from <git-url>` to project README Managing Extensions section.
2. **FR-007**: Aligned auditor prompt with contract example. Approve reasons now optional, reject reasons mandatory (matching spec and contract).
3. **EDGE-002**: Changed "produce a ProposedAddition" to "produce one or more ProposedAdditions" for multi-artifact findings.

---

## Code Review Guide (30 minutes)

> This section guides a code reviewer through the implementation changes,
> focusing on high-level questions that need human judgment.

**Changed files:** 5 files (1 command file, 1 extension config, 1 skill wrapper, 1 extension README, 1 project README update)

### Understanding the changes (8 min)

- Start with `.specify/extensions/backtrace/commands/speckit.backtrace.trace.md`: This is the entire implementation. It's a 430-line prompt-engineering artifact that instructs Claude how to trace findings, dispatch an auditor, and apply changes. Every other file is packaging around this.
- Then `.specify/extensions/backtrace/extension.yml`: 20 lines of metadata. Note the absence of a `hooks:` section, which is a deliberate design decision.
- Question: Does the 10-section structure of the command file feel natural for the flow of operations, or would a different breakdown be clearer for the executing agent?

### Key decisions that need your eyes (12 min)

**No-verdict defaults to approved** (`speckit.backtrace.trace.md` Section 8b, lines 300-302, relates to [FR-008](spec.md))

When the auditor returns a verdict array that doesn't include a verdict for a particular addition, backtrace treats it as approved. The alternative was to treat missing verdicts as rejected (safer) or to error out (strictest). The "approved" default was chosen because a missing verdict likely means the auditor had no objections, and blocking additions on auditor omission would be frustrating.
- Question: Is "default to approved on missing verdict" the right call, or should missing verdicts be treated as "needs manual review"?

**Direct invocation instead of hooks for follow-ups** (`speckit.backtrace.trace.md` Section 10, relates to [FR-012](spec.md) and [FR-013](spec.md))

The [research](research.md) (R-005) discovered that speckit's hook schema only allows one command per hook point per extension. Since backtrace needs three follow-up commands (review-spec, review-plan, analyze), hooks are infeasible. Instead, Section 10 directly invokes them inline. This follows the same pattern as review-code invoking deep-review.
- Question: Is the direct invocation pattern acceptable for this use case, or should this limitation drive a change to the hook schema?

**Subjective "material scope expansion" trigger** (`speckit.backtrace.trace.md` Section 8c, lines 316 and 322, relates to [FR-009](spec.md))

Three of the four reset triggers are mechanical (new user story, new FR, SC count delta). The fourth, "material scope expansion," is described as "subjective, based on whether the additions collectively represent a significant broadening." This means the executing agent makes a judgment call.
- Question: Is a subjective trigger appropriate here, or should this be replaced with something mechanical (e.g., total additions count exceeding a threshold)?

**Plan.md is read-only** (`speckit.backtrace.trace.md` Section 6b, line 172, relates to [FR-004](spec.md))

Backtrace can add new tasks to tasks.md but never writes to plan.md. The rationale is that plan.md describes architecture and phasing, which shouldn't be modified by automated gap-fixing. New tasks can reference existing plan phases without modifying plan.md itself.
- Question: Does the read-only constraint on plan.md make sense, or are there cases where a gap finding should result in a new plan phase?

### Areas where I'm less certain (5 min)

- `speckit.backtrace.trace.md` Section 6b ([FR-003](spec.md)): The tracing logic describes four categories at a high level but doesn't provide detailed heuristics for how the executing agent should decide which category a finding falls into. An experienced agent will likely handle this well, but an ambiguous finding could be categorized inconsistently across runs. The spec doesn't prescribe heuristics either, so this may be intentional.
- `speckit.backtrace.trace.md` Section 8b: The "locate the `target_section` heading" instruction assumes section headings are unique and findable by exact match. If a spec has duplicate section headings or unusual formatting, the agent might insert content in the wrong location.
- `speckit.backtrace.trace.md` Section 8 (empty array handling): An empty verdict array `[]` is treated as "all additions implicitly approved." This could mask a bug where the auditor fails silently and returns nothing. The gap-audit command treats empty as "no findings" (safe), but for backtrace the semantics are different since empty means "no objections to proposed changes."

### Deviations and risks (5 min)

- No deviations from [plan.md](plan.md) were identified. The file structure matches the plan's Project Structure section exactly.
- The command file is 430 lines, within the plan's estimated range of 200-350 lines (slightly above the upper bound). The overshoot comes from defensive error handling and the auditor verification checklist, both of which are warranted.
- Risk: The extension is not yet registered in `.specify/extensions/.registry`. Until `specify extension add` is run, the skill wrapper will delegate to the command file but the extension won't appear in registry-based lookups. This means backtrace's own Section 10 registry check pattern (used for follow-up reviews) would not find backtrace itself if another extension tried to check for it. This is not a problem for backtrace's own operation but is worth noting for future extensions that might depend on backtrace.

---

## Deep Review Report

> Automated multi-perspective code review results. This section summarizes
> what was checked, what was found, and what remains for human review.

**Date:** 2026-04-28 | **Rounds:** 1/3 | **Gate:** PASS

### Review Agents

| Agent | Findings | Status |
|-------|----------|--------|
| Correctness | 7 | completed |
| Architecture & Idioms | 7 | completed |
| Security | 3 | completed |
| Production Readiness | 6 | completed |
| Test Quality | 5 | completed |
| CodeRabbit (external) | - | skipped (--no-external flag) |
| Copilot (external) | - | skipped (--no-external flag) |

### Findings Summary

| Severity | Found | Fixed | Remaining |
|----------|-------|-------|-----------|
| Critical | 1 | 1 | 0 |
| Important | 8 | 8 | 0 |
| Minor | 5 | - | 5 |

### What was fixed automatically

Fixed a Critical registry lookup bug (wrong jq path) reported by 4 of 5 agents that would have silently disabled all follow-up reviews. Fixed 8 Important issues: removed a git diff check that would false-positive on every successful run, corrected the speckit-analyze invocation (core command, not extension), added unique IDs for multi-addition verdict matching, added error handling for missing section headings and empty subagent responses, hardened incomplete verdict handling, reordered prerequisite checks, and added prompt-injection mitigation to the auditor prompt.

### What still needs human attention

5 Minor findings remain (see [review-findings.md](review-findings.md) for details). No further review action needed, but reviewers may want to consider:

- Is the Ship Pipeline Guard (AUTONOMOUS_MODE) warranted for backtrace, or is it YAGNI since the spec doesn't mention autonomous mode?
- The section numbering uses lettered subsections (5b, 6a, 6b, 8b, 8c) instead of gap-audit's strictly sequential pattern. Is this acceptable?
- The post-dispatch git diff only checks tracked files. Is this sufficient?

### Recommendation

All Critical and Important findings addressed. Code is ready for human review with no known blockers.
