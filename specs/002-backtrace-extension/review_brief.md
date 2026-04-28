# Review Brief: Backtrace Extension

**Spec:** specs/002-backtrace-extension/spec.md
**Generated:** 2026-04-28

> Reviewer's guide to scope and key decisions. See full spec for details.

---

## Feature Overview

Backtrace closes the feedback loop in the SDD workflow. When a gap audit finds missing coverage in spec artifacts, backtrace traces each gap back to the AC, FR, or SC that should have caught it, proposes additions, gets adversarial auditor approval, and applies the approved changes. It's packaged as a standalone speckit extension following the same structure as gap-audit.

## Scope Boundaries

- **In scope:** Backtrace command, auditor-gated spec edits, follow-up hook checking, extension packaging, documentation (README + extension README with copy-paste install commands)
- **Out of scope:** Iteration/looping (one pass only), JSON persistence, gap-audit `--fix` flag (backlog item for a follow-up spec), any modifications to the gap-audit extension itself
- **Why these boundaries:** Keeping backtrace standalone means it can accept findings from any source (not just gap-audit). The `--fix` flag is deferred because it couples two extensions and should be specced separately.

## Critical Decisions

### One pass, no iteration
- **Choice:** Backtrace runs once and reports. No automatic re-audit loop.
- **Trade-off:** Simpler and cheaper, but if follow-up reviews find new issues the user must re-run manually.
- **Feedback:** Is one pass sufficient, or will users expect automatic convergence?

### Standalone extension (not a gap-audit subcommand)
- **Choice:** Backtrace is its own extension, independent of gap-audit.
- **Trade-off:** Two extensions to install, but backtrace can accept findings from any source and gap-audit stays focused.
- **Feedback:** Does the install overhead matter for your users?

### Auto-detect findings input
- **Choice:** Backtrace checks the spec directory for `.sdd-findings-{scope}.json` first, falls back to asking the user for a path.
- **Trade-off:** Seamless after `gap-audit --output`, but adds a convention dependency on gap-audit's output filename.
- **Feedback:** Is the fallback (ask user) the right behavior, or should it error instead?

## Areas of Potential Disagreement

### Hook-based follow-ups vs. direct invocation
- **Decision:** Backtrace checks `.specify/extensions.yml` for `hooks.after_backtrace` and executes registered hooks. The extension declares review-spec, review-plan, and analyze as default hooks.
- **Why this might be controversial:** This creates a new hook point (`after_backtrace`) that doesn't exist in core speckit. It's the first extension to define its own hook point rather than hooking into an existing one.
- **Alternative view:** Backtrace could directly invoke the follow-up commands as hardcoded final sections, avoiding the new hook point entirely.
- **Seeking input on:** Is it appropriate for an extension to define its own hook point?

### Reset trigger checking
- **Decision:** Backtrace checks for scope expansion (new user story, new non-refinement FR, SC count change by 3+) and reports, but takes no action.
- **Why this might be controversial:** The user might expect backtrace to halt or roll back when a trigger fires, rather than just reporting.
- **Alternative view:** Backtrace could refuse to apply additions that cross a reset threshold without explicit user confirmation.
- **Seeking input on:** Is report-only the right level of response for reset triggers?

## Naming Decisions

| Item | Name | Context |
|------|------|---------|
| Extension | `backtrace` | Traces gaps backward to their spec origin |
| Command | `speckit.backtrace.trace` | Follows speckit naming: `speckit.{extension}.{command}` |
| Hook point | `after_backtrace` | New hook point, not in core speckit |
| Skill wrapper | `speckit-backtrace-trace` | Follows Claude Code skill naming convention |

## Open Questions

- [ ] Should `after_backtrace` hooks fire even when no additions were applied (all rejected)?

## Risk Areas

| Risk | Impact | Mitigation |
|------|--------|------------|
| Hook point `after_backtrace` not recognized by speckit | Med | Backtrace implements its own hook checking (same code pattern as plan/implement), so it doesn't depend on speckit core recognizing the hook point |
| Auditor rejects valid additions | Low | User sees rejection reasons and can re-run or manually edit |
| Spec additions introduce inconsistencies | Med | Follow-up hooks run review-spec, review-plan, analyze to catch them |

---
*Share with reviewers before implementation.*
