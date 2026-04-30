# Implementation Plan: Exploratory Test Extension

**Branch**: `004-exploratory-test-extension` | **Date**: 2026-04-30 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/004-exploratory-test-extension/spec.md`

## Summary

Package the existing exploratory test skill as a speckit extension. The extension dispatches an adversarial subagent that writes and runs tests against an implementation to find edge cases and bugs the spec missed. This is a mechanical repackaging: the source skill's behavior transfers verbatim, with speckit boilerplate (version check, spec dir resolution, integrity check, defensive parsing) added around it.

## Technical Context

**Language/Version**: Markdown (LLM-executed prose, not compiled code)
**Primary Dependencies**: speckit >= 0.5.2, Claude Code with `superpowers:code-reviewer` subagent type
**Storage**: JSON files (`.exploratory-findings.json` output)
**Testing**: Manual invocation (prompt-engineering artifacts per constitution)
**Target Platform**: Claude Code CLI
**Project Type**: speckit extension
**Performance Goals**: N/A (single subagent dispatch, performance bounded by LLM latency)
**Constraints**: Behavioral parity with source skill (SC-001, 33-item checklist)
**Scale/Scope**: 3 files (extension.yml, README.md, command markdown)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|---|---|---|
| I. Correctness of Findings | PASS | FR-008 (self-check gate), FR-014 (clean reports valid), FR-015 (focused tests) serve precision over recall |
| II. Evidence-Based Claims | PASS | Findings schema requires reproduction, expected, actual fields |
| III. Speckit-Native | PASS | Uses extension.yml, commands/*.md, standard boilerplate |
| IV. One Code Path Per Operation | PASS | Single subagent dispatch, no reimplementation of existing commands |
| V. Precise Skill Instructions | PASS | FR-007 mandates full checklist, output format, and directive in subagent prompt |
| VI. Architectural Decisions | PASS | All decisions resolved in brainstorming (command name, schema, test mode) |
| VII. Flag Uncertainty | PASS | FR-008 marks low-confidence findings rather than discarding |
| VIII. Conventional Commits | PASS | Will follow on commit |

## Project Structure

### Documentation (this feature)

```text
specs/004-exploratory-test-extension/
├── plan.md              # This file
├── spec.md              # Feature specification
├── research.md          # Phase 0: source skill analysis
├── data-model.md        # Phase 1: ExploratoryFinding schema
├── contracts/           # Phase 1: JSON output contract
│   └── exploratory-findings.json
├── REVIEW-SPEC.md       # Spec review output
├── checklists/
│   └── requirements.md  # Quality checklist
└── tasks.md             # Phase 2 output (via /speckit-tasks)
```

### Source Code (repository root)

```text
.specify/extensions/exploratory-test/
├── extension.yml                              # Extension metadata and command declarations
├── README.md                                  # Installation, usage, output format docs
└── commands/
    └── speckit.exploratory-test.test.md        # Command implementation (11 sections)
```

**Structure Decision**: Matches the established extension pattern used by gap-audit and backtrace. No deviation.

## Implementation Approach

This is a mechanical repackaging. The implementation follows an 11-section command structure (gap-audit has 10; Section 5 is new for changed file discovery), transplanting the source skill's behavior into each section. Sections are written sequentially because they append to a single command markdown file; each section depends on prior sections being in place for the file to be well-formed.

| Section | Source | What it does |
|---|---|---|
| 1. Version Check | Speckit boilerplate | Check speckit >= 0.5.2 from init-options.json |
| 2. Argument Parsing | FR-001 | Extract `--base`, `--output` flags, spec dir token |
| 3. Spec Directory Resolution | FR-002 | Three-step fallback (arg/feature.json/ask), --output constraint |
| 4. Artifact Loading | FR-004 | Read spec.md, extract user stories/FRs/ACs/SCs |
| 5. Changed File Discovery | FR-005 | git diff --name-only, base branch validation |
| 6. Defect Catalog Loading | FR-006 | Optional defect-catalog.md, pattern extraction |
| 7. Subagent Prompt Construction | FR-007, FR-008, FR-014, FR-015 | Full prompt with spec, files, checklist, self-check gate |
| 8. Subagent Dispatch + Integrity Check | FR-007, FR-012 | Single dispatch, post-dispatch git checks |
| 9. Response Parsing | FR-009, FR-010 | Defensive JSON parsing, source field injection |
| 10. JSON Persistence | FR-011 | Write .exploratory-findings.json when --output |
| 11. Output Formatting | FR-013 | Severity-grouped presentation, PARSE_FAILED handling |

Section 7 (prompt construction) is the largest section. It transplants the source skill's test vector checklist, self-check gate, and adversarial directive verbatim, wrapped in the speckit artifact-tag convention.

## Known Integration Constraints

**Backtrace cannot consume exploratory findings.** Backtrace's parser validates the first element against GapFinding fields (classification, category, description, evidence, suggested_fix). ExploratoryFinding has a different schema (severity, reproduction, expected, actual, spec_gap). Backtrace's auto-detect glob (`.*-findings.json`) will match `.exploratory-findings.json` but parsing will fail. T016(c) should produce a concrete follow-up task on the backtrace extension to make its parser schema-aware by checking the `source` field before validating schema-specific fields.

**Bare `git diff --name-only` in integrity check.** FR-012 uses bare `git diff --name-only` (no revision argument), which only detects unstaged modifications. This matches the gap-audit and backtrace convention. Staged modifications by the subagent would be missed. Accepted as a consistency decision per SC-004 (no behavioral additions beyond speckit boilerplate). A project-wide fix would need to update all three extensions.

## Complexity Tracking

No constitution violations. No complexity justifications needed.
