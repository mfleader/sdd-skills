# Review Guide: SDD Gap Audit

**Spec:** [spec.md](spec.md) | **Plan:** [plan.md](plan.md) | **Tasks:** [tasks.md](tasks.md)
**Generated:** 2026-04-27

---

## What This Spec Does

An adversarial gap auditor for SDD artifacts, packaged as a speckit extension. It dispatches a prejudiced subagent to find gaps in specs or plans that the standard review step missed, filters the raw findings for false positives, and presents the results grouped by severity. The entire implementation is prompt engineering (markdown + YAML), not compiled code.

**In scope:** One command with two audit scopes (spec and plan), 12 checklist items, 8 false positive filters, opt-in JSON persistence, optional pattern matching against recurring gap types.

**Out of scope:** Fixing findings, re-running review commands, back-trace, review loop/convergence, auto-fix, sdd-drive orchestrator integration.

## Bigger Picture

This is the first extension in the sdd-skills repo. The repo already has a constitution, a standalone `sdd-gap-audit` skill, and a `sdd-artifact-review-loop` skill. This extension packages the gap audit as a speckit-installable artifact so it can be distributed to other projects. The existing standalone skill will eventually be superseded by this extension, but both will coexist during the transition.

The extension follows patterns established by spex-gates (review commands), but deliberately avoids lifecycle hooks. It is invoke-only because the gap audit should run after the review-spec/review-plan gates, not as part of them.

---

## Spec Review Guide (30 minutes)

> Focus your time on the parts that need human judgment most.

### Understanding the approach (8 min)

Read the [user stories](spec.md#user-scenarios--testing-mandatory) and the [false positive filters](spec.md#functional-requirements) (FR-006). As you read, consider:

- The spec prioritizes precision over recall (Constitution Principle I). Is an 80% recall floor (SC-001) too low, too high, or about right for an adversarial auditor?
- The [false positive filters](spec.md#functional-requirements) are lessons from real PR reviews. Do they capture your experience with false positives in code review tools, or are there common false positive patterns they miss?
- The command completes in a [single subagent dispatch](spec.md#functional-requirements) (FR-020). Is this the right constraint, or would a refinement round improve quality enough to justify the cost?

### Key decisions that need your eyes (12 min)

**Subagent type: superpowers:code-reviewer** ([assumptions](spec.md#assumptions))

The spec uses `superpowers:code-reviewer` instead of `general-purpose` because it loads structured review discipline. The spex-deep-review extension uses `general-purpose` for its agents.
- Does the code-reviewer subagent type actually provide the claimed benefits? Has anyone tested the difference for this use case?

**Opt-in JSON persistence** ([US5](spec.md#user-story-5---persist-findings-and-match-patterns-priority-p3), [FR-007](spec.md#functional-requirements))

The default behavior is presentation-only. JSON output requires `--output`. This was a deliberate decision to avoid overwhelming users with files.
- Is this the right default? If the gap audit runs as part of an automated pipeline later, would the pipeline need JSON output by default?

**GapFinding vs Finding naming** ([key entities](spec.md#key-entities))

The entity is named `GapFinding` to avoid collision with spex-deep-review's `Finding` (which has a different schema). Both entities have a `category` and `description` field but differ on severity classification.
- Is the naming distinction sufficient, or should the schemas be aligned for future interoperability?

**Extension name** (throughout)

The extension name is `gap-audit`.
- What should the extension be named? This blocks implementation.

### Areas where I'm less certain (5 min)

- [FR-025](spec.md#functional-requirements): The version check requires the command to determine the installed speckit version at runtime. I don't know how speckit exposes its version to extensions. This might need research during implementation.
- [False positive filter implementation](plan.md#project-structure): The plan says filters run in the main agent context after the subagent returns. For filters 1 and 2 (verify technical claims, cross-reference tasks), this means the main agent reads files and greps the codebase. If the codebase is large, this could be slow. The spec doesn't set a time budget for filtering.
- [Pattern matching](spec.md#assumptions): The matching mechanism is "category-based" but the trigger condition alignment is vague. Two findings in the same category might not match the same pattern. The implementation will need to make judgment calls about what "aligns with" means.

### Risks and open questions (5 min)

- ~~The extension name is TBD.~~ Resolved: `gap-audit`.
- If `superpowers:code-reviewer` is unavailable in a user's environment, the command fails entirely (no fallback). Is that acceptable, or should there be a degraded mode using `general-purpose`?
- The 8 false positive filters are embedded in the command markdown. If filters need updating based on new false positive patterns, every installation needs to update the extension. Is there a mechanism for filter updates without full extension upgrades?

---

*Full context in linked [spec](spec.md) and [plan](plan.md).*
