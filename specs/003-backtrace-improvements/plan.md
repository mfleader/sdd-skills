# Implementation Plan: Backtrace Improvements

**Branch**: `003-backtrace-improvements` | **Date**: 2026-04-30 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/003-backtrace-improvements/spec.md`

## Summary

Add barrier enumeration, categorical trace certainty, and cluster detection to the backtrace extension. All changes modify the existing command file (speckit.backtrace.trace.md), the data model (data-model.md), the spec (spec.md for the base 002 extension), and the extension version. Total estimated change: ~30 lines of markdown instruction, one additive field, one version bump.

## Technical Context

**Language/Version**: Markdown (Claude Code skill instructions), Bash (shell snippets within skill)
**Primary Dependencies**: Backtrace extension v0.1.0, speckit >= 0.5.2
**Storage**: N/A (markdown files, no database)
**Testing**: Manual invocation of backtrace with test findings; no automated test framework
**Target Platform**: Claude Code CLI (LLM agent executing markdown instructions)
**Project Type**: Speckit extension (markdown command files + YAML metadata)
**Performance Goals**: N/A (LLM execution, not latency-sensitive)
**Constraints**: Changes must be additive to v0.1.0; no breaking changes to GapFinding schema or existing ProposedAddition consumers
**Scale/Scope**: Single command file (~420 lines), one data model file, one extension.yml

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Correctness of Findings | PASS | Barrier enumeration improves trace correctness (targets correct spec layer) |
| II. Evidence-Based Claims | PASS | trace_certainty flags uncertain claims per this principle |
| III. Speckit-Native | PASS | All changes follow speckit extension conventions (command file, extension.yml) |
| IV. One Code Path Per Operation | PASS | No new commands; modifications to existing command file |
| V. Precise Skill Instructions | PASS | Barrier walk is a checklist; certainty levels are defined precisely (FR-022) |
| VI. Architectural Decisions Require Approval | PASS | All three improvements were approved through brainstorm + spec review |
| VII. Flag Uncertainty | PASS | trace_certainty directly implements this principle |
| VIII. Conventional Commits | PASS | Version bump specified (FR-028) |

No violations. No complexity tracking needed.

## Project Structure

### Documentation (this feature)

```text
specs/003-backtrace-improvements/
├── plan.md              # This file
├── spec.md              # Feature specification
├── improvement-research.md  # Research backing the three improvements
├── checklists/
│   └── requirements.md  # Spec quality checklist
├── REVIEW-SPEC.md       # Spec review results
├── data-model.md        # Phase 1: amended ProposedAddition schema
└── tasks.md             # Phase 2 output (created by /speckit-tasks)
```

### Source Code (files to modify)

```text
.specify/extensions/backtrace/
├── extension.yml                          # Version bump: 0.1.0 → 0.2.0
└── commands/
    └── speckit.backtrace.trace.md         # Sections 6, 7, 11 modified

specs/002-backtrace-extension/
└── data-model.md                          # ProposedAddition entity amended
```

**Structure Decision**: No new files created in the extension. All changes are modifications to existing files. The data-model.md lives in the 002 spec directory because it documents the backtrace extension's data model (not this improvement spec's).

## Phase 0: Research

No NEEDS CLARIFICATION items in the Technical Context. The research was completed prior to this spec in `improvement-research.md`.

**Decisions already made:**

| Decision | Rationale | Alternatives Rejected |
|----------|-----------|----------------------|
| Barrier enumeration as checklist walk | Mechanical, fits LLM execution well; from RCA barrier analysis | FTA decomposition (too expensive per finding), STPA categories (orthogonal, not replacement) |
| Categorical certainty over numeric scores | LLM self-assessed numeric confidence unreliable (ECE 0.24-0.47); categorical avoids calibration problem | Numeric confidence (unreliable), no certainty (violates Principle VII) |
| Fixed threshold of 3 for clusters | Typical finding volumes 5-15; at these volumes clustering is trivially visible below 3 | Configurable threshold (over-engineered for v0.2.0) |
| Relevance determined by LLM judgment | Spec items are natural language; no formal ontology or keyword index available | Keyword matching (brittle), embedding similarity (no infrastructure) |

No research.md needed. Referencing improvement-research.md for full backing.

## Phase 1: Design

### Data Model Changes

**ProposedAddition** (amended entity, additive change):

| Field | Type | Required | Description | New? |
|-------|------|----------|-------------|------|
| addition_id | string | yes | Unique ID (e.g., "1.1", "1.2") | no |
| finding_ref | string | yes | Reference to the GapFinding | no |
| target_artifact | string | yes | `spec.md` or `tasks.md` | no |
| target_section | string | yes | Section heading where addition goes | no |
| addition_type | string | yes | `new_FR`, `amended_FR`, `new_AC`, `amended_AC`, `new_SC`, `new_task` | no |
| content | string | yes | The actual text to add or amend | no |
| rationale | string | yes | Why this addition addresses the gap | no |
| **trace_certainty** | **string** | **yes** | **`direct`, `plausible`, or `uncertain`** | **yes** |

No changes to GapFinding or AuditorVerdict entities.

### Command File Changes

Three sections of `speckit.backtrace.trace.md` require modification:

**Section 6 (Core Tracing Logic)**: Insert barrier walk instructions before the four tracing categories. The walk and categories operate together in a single pass (FR-020). Add trace_certainty assignment after each trace.

**Section 7 (Subagent Dispatch)**: Add one paragraph to the auditor preamble instructing more skepticism for `uncertain` traces.

**Section 11 (Output Formatting)**: Add `[certainty]` display to each addition line. Add cluster warning subsection after Applied Additions.

### Contracts

No external interfaces. The backtrace extension is invoked as a Claude Code skill command. The only "contract" is the ProposedAddition schema (documented in data-model.md) and the AuditorVerdict schema (unchanged).

No contracts/ directory needed for this feature.

### Agent Context Update

Update CLAUDE.md to reference this plan:

```text
<!-- SPECKIT START -->
For project context, read specs/constitution.md for governance principles
and README.md for the SDD workflow, extensions, and project structure.
Current plan: specs/003-backtrace-improvements/plan.md
<!-- SPECKIT END -->
```

## Phase 2: Task Breakdown

Task generation deferred to `/speckit-tasks`. The implementation is straightforward: modify three sections of one command file, update one data model file, bump one version number.
