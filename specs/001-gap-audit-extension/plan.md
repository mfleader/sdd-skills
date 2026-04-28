# Implementation Plan: SDD Gap Audit

**Branch**: `001-gap-audit-extension` | **Date**: 2026-04-27 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/001-gap-audit-extension/spec.md`

## Summary

A speckit extension that provides a single invoke-only command: an adversarial gap auditor for SDD artifacts. The command accepts an audit scope (`spec` or `plan`), dispatches a prejudiced subagent via `superpowers:code-reviewer` to find gaps the standard review step missed, applies 8 false positive filters to the raw findings, and presents the filtered results grouped by blocking/non-blocking. An optional `--output` flag persists findings as JSON. The implementation is entirely prompt engineering artifacts (markdown command files + YAML metadata), not traditional source code.

## Technical Context

**Language/Version**: Markdown (speckit command files are prompt engineering artifacts, not compiled code)
**Primary Dependencies**: speckit >= 0.5.2, Claude Code Agent tool with `superpowers:code-reviewer` subagent type
**Storage**: JSON files (`.sdd-findings-spec.json`, `.sdd-findings-plan.json`) written to spec directory when `--output` flag is set
**Testing**: Manual testing with test specs containing planted gaps. Precision/recall measured by human review.
**Target Platform**: Claude Code CLI (any platform running Claude Code)
**Project Type**: Speckit extension (prompt engineering artifact)
**Performance Goals**: Single subagent dispatch per audit (FR-020)
**Constraints**: >= 80% recall (SC-001), <= 20% false positive rate (SC-002), single dispatch only
**Scale/Scope**: 1 extension, 1 command, 2 audit scopes, 12 checklist items (7 spec + 5 plan), 8 false positive filters

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Evidence |
|-----------|--------|----------|
| I. Correctness of Findings | PASS | 8 false positive filters (FR-006), precision/recall thresholds (SC-001/002) |
| II. Evidence-Based Claims | PASS | FR-008 requires evidence field, SC-003 verifies citability |
| III. Speckit-Native | PASS | extension.yml + commands/*.md structure (FR-015), installable via `specify extension add` |
| IV. One Code Path | PASS | FR-017 no dependency on other extensions, FR-018 report-only |
| V. Precise Skill Instructions | PASS | FR-004 requires full checklist, classification scheme, and output constraints in subagent prompt |
| VI. Architectural Decisions | PASS | Subagent type choice documented in assumptions with rationale |
| VII. Flag Uncertainty | PASS | FR-023 passes unverified findings through with reason noted |
| VIII. Conventional Commits | N/A | Development workflow, not plan scope |

**Gate result: PASS. No violations.**

## Project Structure

### Documentation (this feature)

```text
specs/001-gap-audit-extension/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── contracts/           # Phase 1 output
│   └── command-schema.md
├── quickstart.md        # Phase 1 output
└── tasks.md             # Phase 2 output (/speckit.tasks - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
.specify/extensions/
└── gap-audit/
    ├── extension.yml                          # Extension metadata, command registration, version requirement
    ├── commands/
    │   └── speckit.gap-audit.audit.md          # The gap audit command (prompt engineering artifact)
    └── README.md                              # Usage documentation

.claude/skills/
└── speckit-gap-audit-audit/
    └── SKILL.md                               # Claude Code skill wrapper for the command

specs/
└── gap-patterns.md                            # Example gap-patterns.md (project-level, not per-feature)
```

**Structure Decision**: This is a speckit extension, not a traditional software project. The deliverables are markdown files (command prompt) and YAML (extension metadata). The skill wrapper (SKILL.md) is the entry point that Claude Code loads. The extension directory is what `specify extension add` installs. The extension name is `gap-audit`.

## Complexity Tracking

No constitution violations to justify.
