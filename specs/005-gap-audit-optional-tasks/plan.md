# Implementation Plan: Gap Audit Optional tasks.md

**Branch**: `006-gap-audit-optional-tasks` | **Date**: 2026-05-26 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/005-gap-audit-optional-tasks/spec.md`

## Summary

Make tasks.md optional in the gap-audit extension's plan scope. When tasks.md is absent, run only categories 4-5 (implicit behavior, spec coverage gaps). When present, run all 5 categories. Changes are confined to the command markdown file (LLM-executed prose), spec artifacts, and documentation. No traditional source code is involved.

## Technical Context

**Language/Version**: Markdown (LLM-executed prose), not traditional code  
**Primary Dependencies**: speckit >= 0.5.2, Claude Code Agent tool  
**Storage**: JSON files (findings persistence)  
**Testing**: Manual invocation against test spec directories  
**Target Platform**: Claude Code CLI  
**Project Type**: speckit extension (LLM skill)  
**Performance Goals**: N/A (single subagent dispatch, unchanged)  
**Constraints**: Backward compatibility with existing bare JSON array output  
**Scale/Scope**: Single command file (~409 lines), plus spec/doc updates

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Correctness of Findings | Pass | FR-017 adds host-side category validation to prevent incorrect findings in degraded mode |
| II. Evidence-Based Claims | Pass | No change to evidence requirements |
| III. Speckit-Native | Pass | Modifies existing extension command, no new infrastructure |
| IV. One Code Path Per Operation | Pass | Modifies existing command, no parallel implementation |
| V. Precise Skill Instructions | Pass | FR-005, FR-007 specify exact prompt text and constraints |
| VI. Architectural Decisions | Pass | Per-finding `categories_audited` vs wrapper object approved with rationale |
| VII. Flag Uncertainty | Pass | Known limitation (backtrace) documented |
| VIII. Conventional Commits | Pass | FR-016 requires version bump for schema change |

No violations. No complexity tracking needed.

## Project Structure

### Documentation (this feature)

```text
specs/005-gap-audit-optional-tasks/
├── spec.md
├── plan.md              # This file
├── research.md
├── data-model.md
├── contracts/
│   └── command-schema.md
├── checklists/
│   └── requirements.md
├── sdd-workflow-checklist.md
├── REVIEW-SPEC.md
└── .gap-audit-spec-findings.json
```

### Source Code (files to modify)

```text
.specify/extensions/gap-audit/
├── extension.yml                    # Version bump (FR-016)
├── commands/
│   └── speckit.gap-audit.audit.md   # Core changes (FR-001 through FR-018)
└── README.md                        # Usage docs update

specs/001-gap-audit-extension/
├── spec.md                          # Original spec updates (FR cross-refs)
├── contracts/
│   └── command-schema.md            # Input artifacts table
└── data-model.md                    # GapFinding entity update

.claude/skills/speckit-gap-audit-audit/
└── SKILL.md                         # Wrapper description update

README.md                            # Project-level description update
```

**Structure Decision**: No new files created. All changes modify existing files in the gap-audit extension, original spec artifacts, and documentation.

## Design Decisions

### D1: Conditional logic in command markdown

The command file is LLM-executed prose. Conditional behavior (tasks.md present vs absent) is expressed as prose instructions with explicit if/then branching, not as code. Each section that varies by tasks.md presence has two clearly labeled subsections.

### D2: Category validation in Section 8 (post-parse)

FR-017 requires host-side rejection of out-of-category findings. This is implemented as a new validation step in Section 8 (Response Parsing) that checks each finding's `category` field against the active categories before proceeding to Section 9 (persistence) and Section 10 (output).

### D3: Modification approach

Changes to the command file are surgical edits to specific sections, not a rewrite. Each section is modified independently:
- Section 4: Add tasks.md optional loading
- Section 6: Conditional preamble, checklist, artifacts, validation, filter, patterns
- Section 8: Add category validation step
- Section 9: Add `categories_audited` field
- Section 10: Conditional summary line

### D4: Original spec (001) updates are documentation-only

The original spec in `specs/001-gap-audit-extension/` is updated to reflect the new behavior but these are documentation changes (the spec describes what the command does, not implementation code). The actual behavior change is in the command file.
