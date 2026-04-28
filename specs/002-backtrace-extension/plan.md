# Implementation Plan: Backtrace Extension

**Branch**: `main` | **Date**: 2026-04-28 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/002-backtrace-extension/spec.md`

## Summary

Package the backtrace skill as a standalone speckit extension following the gap-audit pattern. The extension traces gap-audit findings back to missing spec items, proposes additions with auditor approval, and directly invokes follow-up reviews (review-spec, review-plan, analyze) as its final step. This is a prompt-engineering project: the deliverables are markdown command files and YAML configs, not compiled code.

**Key design revision (from research gap audit):** The original spec assumed hooks could declare multiple follow-up commands at one hook point. Speckit's hook schema only allows one command per hook point per extension. Instead, the backtrace command file directly invokes follow-up reviews inline (same pattern as review-code invoking deep-review). No hooks are declared in extension.yml. See research.md R-005 for details.

## Technical Context

**Language/Version**: Markdown (prompt-engineering artifacts) + YAML (extension config)
**Primary Dependencies**: speckit >= 0.5.2, spex-gates (soft, for follow-up hooks)
**Storage**: N/A (reads/writes spec artifact files)
**Testing**: Manual invocation against test specs with planted gaps (same pattern as gap-audit test fixtures)
**Target Platform**: Claude Code CLI
**Project Type**: Speckit extension (prompt-engineering artifact)
**Performance Goals**: N/A (single-invocation command)
**Constraints**: Must follow speckit extension conventions. Single subagent dispatch per invocation.
**Scale/Scope**: 1 command file (~400 lines, 12 sections), 1 extension.yml, 1 skill wrapper (gitignored, auto-generated on install), 1 README, project README update

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Constitution is the default unfilled template. No project-specific principles defined yet. The README documents 8 core principles that serve as informal constitution:

1. **Correctness of Findings** (precision over recall) - Applies: auditor review must not generate false positives
2. **Evidence-Based Claims** - Applies: proposed additions must reference the finding they address
3. **Speckit-Native** - Applies: must use extension.yml, commands/, skill wrapper pattern
4. **One Code Path Per Operation** - Applies: single command, no branching workflows
5. **Precise Skill Instructions** - Applies: command file must be unambiguous
6. **Architectural Decisions Require Explicit Approval** - Applies: auditor gates changes
7. **Flag Uncertainty** - Applies: untraceable findings must be reported, not silently dropped
8. **Conventional Commits** - Applies to all commits

No violations anticipated. All principles align with the spec.

## Project Structure

### Documentation (this feature)

```text
specs/002-backtrace-extension/
├── spec.md              # Feature specification (done)
├── plan.md              # This file
├── research.md          # Phase 0: hook schema, command file patterns
├── data-model.md        # Phase 1: entity definitions
├── contracts/
│   └── command-schema.md  # Phase 1: command invocation contract
├── REVIEW-SPEC.md       # Spec review (done)
├── review_brief.md      # Reviewer guide (done)
├── checklists/
│   └── requirements.md  # Quality checklist (done)
└── tasks.md             # Phase 2 output (via /speckit-tasks)
```

### Source Code (repository root)

```text
.specify/extensions/backtrace/
├── extension.yml                    # Extension metadata (no hooks, follow-ups are inline)
├── commands/
│   └── speckit.backtrace.trace.md   # The command file (prompt-engineering artifact)
└── README.md                        # Extension documentation

.claude/skills/speckit-backtrace-trace/   # gitignored, auto-generated on install
└── SKILL.md                         # Claude Code skill wrapper
```

**Structure Decision**: Follows the gap-audit extension pattern exactly. Two directories: `.specify/extensions/backtrace/` for the speckit extension, `.claude/skills/speckit-backtrace-trace/` for the Claude Code skill entry point. README.md at project root gets updated.

## Complexity Tracking

No complexity violations. Single command, single subagent dispatch, standard extension structure.
