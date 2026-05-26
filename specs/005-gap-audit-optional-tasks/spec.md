# Feature Specification: Gap Audit Optional tasks.md

**Feature Branch**: `006-gap-audit-optional-tasks`  
**Created**: 2026-05-26  
**Status**: Draft  
**Input**: User description: "Make tasks.md optional in gap-audit plan scope with graceful degradation"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Audit Plan Against Spec Before Task Generation (Priority: P1)

A skill author has written a spec and plan for a new feature but has not yet generated tasks. They want to validate that the plan adequately covers the spec's requirements before investing effort in task generation. They invoke the gap audit command with `plan` scope. The auditor reads `spec.md` and `plan.md`, emits an info message that tasks.md is not available, dispatches an adversarial subagent with categories 4-5 only, applies false positive filters, and presents findings with a partial-audit indicator in the summary.

**Why this priority**: This is the core new capability. Running plan-vs-spec audits before task generation catches architectural gaps early, preventing wasted effort on tasks derived from a flawed plan.

**Independent Test**: Can be tested by creating a spec directory containing `spec.md` and `plan.md` (no `tasks.md`) where the plan omits architectural coverage for a spec FR that implies retry semantics. Confirm the auditor identifies a "spec coverage gaps" finding and the summary indicates a partial audit.

**Acceptance Scenarios**:

1. **Given** a spec directory containing `spec.md` and `plan.md` but no `tasks.md`, **When** the user invokes the gap audit command with scope `plan`, **Then** the auditor emits "tasks.md not found, running plan-vs-spec audit only (categories 4-5)." and presents findings using only the "Implicit behavior" and "Spec coverage gaps" categories.
2. **Given** a spec directory containing `spec.md` and `plan.md` but no `tasks.md`, **When** the auditor completes and findings exist, **Then** the summary line reads "N blocking, M non-blocking findings (categories 4-5 only, tasks.md not available)."
3. **Given** a spec directory containing `spec.md` and `plan.md` but no `tasks.md`, **When** the auditor completes with no findings, **Then** the output reads "No issues found." (no partial-audit qualifier needed when clean).
4. **Given** a spec directory containing `spec.md` and `plan.md` but no `tasks.md`, **When** the user invokes the gap audit with `--output`, **Then** each persisted finding includes `"categories_audited": ["Implicit behavior", "Spec coverage gaps"]` alongside existing `source` and `scope` fields.

---

### User Story 2 - Full Plan Audit With Tasks Present (Priority: P1)

A skill author has completed the spec, plan, and tasks phases and invokes the gap audit command with `plan` scope. The auditor reads all three files and audits against all 5 plan checklist categories, producing the same behavior as before this change.

**Why this priority**: Backward compatibility is equally critical. Existing workflows (including sdd-drive step 8) must produce identical results when tasks.md is present.

**Independent Test**: Can be tested by running the gap audit with scope `plan` on a spec directory containing all three files with known gaps in categories 1-3. Confirm findings are produced for all 5 categories and no partial-audit indicator appears in the summary.

**Acceptance Scenarios**:

1. **Given** a spec directory containing `spec.md`, `plan.md`, and `tasks.md` where one spec edge case has no corresponding task, **When** the user invokes the gap audit command with scope `plan`, **Then** the auditor reports findings across all 5 plan checklist categories and the summary says "N blocking, M non-blocking findings." (no partial-audit qualifier).
2. **Given** a spec directory containing all three files, **When** the user invokes the gap audit with `--output`, **Then** each persisted finding includes `"categories_audited"` listing all 5 plan checklist categories.
3. **Given** a spec directory containing all three files where two components interact but no contract test exists at their API boundary, **When** the user invokes the gap audit with scope `plan`, **Then** the auditor reports a "Contract tests" finding citing the boundary and the missing verification.

---

### User Story 3 - Pattern Filtering in Degraded Mode (Priority: P2)

A skill author has a project-level `gap-patterns.md` file with recurring patterns cataloged across previous audits. Some patterns target categories 1-3 (e.g., "missing-contract-boundary-test" under "Contract tests"). When running plan scope without tasks.md, patterns for inactive categories are excluded from the subagent prompt so they cannot produce spurious matches.

**Why this priority**: Pattern matching is a power-user feature (P3 in the original spec). Ensuring it works correctly in degraded mode is important but secondary to the core capability.

**Independent Test**: Can be tested by creating a `gap-patterns.md` with patterns in categories 1-3 and categories 4-5, running plan scope without tasks.md, and confirming only category 4-5 patterns appear in the subagent prompt.

**Acceptance Scenarios**:

1. **Given** a `gap-patterns.md` with patterns in categories 1, 3, and 5, and a spec directory with `spec.md` and `plan.md` (no `tasks.md`), **When** the user invokes the gap audit with scope `plan`, **Then** only the category 5 pattern is included in the subagent prompt; categories 1 and 3 patterns are excluded.
2. **Given** the same `gap-patterns.md` and a spec directory with all three files present, **When** the user invokes the gap audit with scope `plan`, **Then** all patterns (categories 1, 3, and 5) are included in the subagent prompt.

---

### Edge Cases

- When `plan.md` is missing but scope is `plan`, the command returns an error naming the missing file (unchanged behavior).
- When `spec.md` is missing but scope is `plan`, the command returns an error naming the missing file (unchanged behavior).
- When `tasks.md` is missing and scope is `plan`, the command does NOT return an error. It emits an info message and runs in degraded mode (new behavior).
- When `tasks.md` is missing and scope is `spec`, behavior is unchanged (tasks.md was never loaded for spec scope).
- When `gap-patterns.md` exists but all patterns target categories 1-3, and scope is `plan` without tasks.md, the pattern matching section is omitted entirely from the subagent prompt (no applicable patterns).
- When the subagent produces a finding with a category not in the active checklist (e.g., "Contract tests" when only categories 4-5 are active), the finding is rejected during category validation.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: When scope is `plan`, the command MUST read `spec.md` and `plan.md` from the spec directory. If `tasks.md` exists in the spec directory, the command MUST also read it.
- **FR-002**: When scope is `plan` and `tasks.md` is present, the command MUST audit against all 5 plan checklist categories (contract tests, integration gaps, edge case coverage, implicit behavior, spec coverage gaps). This is identical to existing behavior.
- **FR-003**: When scope is `plan` and `tasks.md` is absent, the command MUST audit against categories 4 (implicit behavior) and 5 (spec coverage gaps) only.
- **FR-004**: When scope is `plan` and `tasks.md` is absent, the command MUST emit an info message to the user before dispatching the subagent: "tasks.md not found, running plan-vs-spec audit only (categories 4-5)."
- **FR-005**: The subagent prompt preamble MUST be conditional on tasks.md presence. When tasks.md is present: "Check the plan and tasks against each of these 5 categories, using the spec as the source of truth." When absent: "Check the plan against the spec using these 2 categories."
- **FR-006**: The subagent prompt MUST only include the `tasks.md` artifact block when tasks.md was loaded.
- **FR-007**: The category validation directive in the subagent prompt MUST restrict valid categories to those in the active checklist. When tasks.md is absent, valid categories are "Implicit behavior" and "Spec coverage gaps" only.
- **FR-008**: False positive filter 2 (cross-reference tasks) MUST be skipped when tasks.md is not an input artifact. The skip condition changes from "when audit scope is spec" to "when tasks.md is not an input artifact." This is logically equivalent for spec scope (which never loads tasks.md) and only changes behavior for plan scope without tasks.md.
- **FR-009**: When `gap-patterns.md` is loaded and scope is `plan`, the command MUST filter included patterns to only those whose `category` matches an active checklist category. Patterns for inactive categories MUST be excluded from the subagent prompt. If after filtering no applicable patterns remain, the pattern matching section MUST be omitted entirely from the subagent prompt.
- **FR-010**: When scope is `plan` and tasks.md is absent, the summary line MUST read: "N blocking, M non-blocking findings (categories 4-5 only, tasks.md not available)."
- **FR-011**: When scope is `plan` and tasks.md is present, the summary line MUST read: "N blocking, M non-blocking findings." (no partial-audit qualifier, identical to existing behavior).
- **FR-012**: When `--output` is set, each persisted finding MUST include a `categories_audited` field (string array) listing the checklist categories that were active in the audit run, in checklist order. For degraded plan mode: `["Implicit behavior", "Spec coverage gaps"]`. For full plan mode: `["Contract tests", "Integration gaps", "Edge case coverage", "Implicit behavior", "Spec coverage gaps"]`. For spec mode: `["Orphan FRs", "Weak ACs", "Unverifiable SCs", "Cross-reference gaps", "Implicit assumptions", "Naming collisions", "Implicit behavior"]`.
- **FR-013**: The `categories_audited` field MUST be added per-finding alongside existing `source` and `scope` fields, maintaining the bare JSON array output format. A top-level metadata wrapper is not used.
- **FR-014**: When scope is `plan` and `plan.md` is missing, the command MUST return an error naming the missing file (unchanged behavior).
- **FR-015**: When scope is `spec`, all existing behavior MUST remain unchanged. tasks.md is never loaded for spec scope.
- **FR-016**: The extension MUST bump the version in `extension.yml` when the `categories_audited` field is added to the finding schema, per the constitution Development Workflow requirement that finding schema changes require a version bump.
- **FR-017**: After parsing subagent findings (Section 8 of the command), the command MUST reject any finding whose `category` field does not match one of the active checklist categories. Rejected findings MUST be excluded from output and persistence.
- **FR-018**: When scope is `plan`, tasks.md is absent, and no findings survive filtering, the output MUST read "No issues found."

### Key Entities

- **GapFinding**: Unchanged from the original spec, with one new field: `categories_audited` (string array, required on persist). Lists which checklist categories were active in the audit run.
- **Audit Scope**: The scope selector. `spec` audits spec artifacts only. `plan` audits spec + plan artifacts (required), plus tasks artifacts when available. When tasks.md is absent in plan scope, only categories 4-5 are audited.
- **Plan Checklist Categories (full, 5 items)**: Contract tests, Integration gaps, Edge case coverage, Implicit behavior, Spec coverage gaps. Active when tasks.md is present.
- **Plan Checklist Categories (degraded, 2 items)**: Implicit behavior, Spec coverage gaps. Active when tasks.md is absent.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: When tasks.md is present, the gap audit plan scope uses the same 5-category checklist, the same subagent prompt structure, and the same filter logic as the current implementation, verified by comparing the constructed prompt and filter configuration against the pre-change implementation.
- **SC-002**: When tasks.md is absent, the gap audit plan scope produces findings only in categories 4 and 5, verified by checking that no finding has a category outside these two.
- **SC-003**: The summary line correctly indicates partial audit when tasks.md is absent, verified by running the command without tasks.md and checking the summary text contains "(categories 4-5 only, tasks.md not available)".
- **SC-004**: Persisted findings include the `categories_audited` field with the correct category list, verified by parsing the output JSON and checking the field on each finding.
- **SC-005**: Gap-patterns for inactive categories are excluded from the subagent prompt in degraded mode, verified by inspecting the constructed prompt when patterns exist for categories 1-3 and tasks.md is absent.
- **SC-006**: The command does not error when tasks.md is absent in plan scope, verified by running the command without tasks.md and confirming it completes successfully with findings or "No issues found."
- **SC-007**: Downstream consumers (backtrace, sdd-drive) that expect a bare JSON array can parse the output without errors, verified by running `jq` against the output file.
- **SC-008**: The extension.yml version field is higher than the pre-change version, verified by comparing the version in the modified extension.yml against the current version.

## Assumptions

- The user has a speckit project initialized with at least version 0.5.2.
- Spec and plan artifacts follow the standard speckit template structure and naming conventions (`spec.md`, `plan.md`).
- tasks.md is optional for plan scope. When absent, the audit runs in degraded mode covering categories 4-5 only. When present, all 5 categories are audited.
- The `superpowers:code-reviewer` subagent type is available in the user's Claude Code environment.
- The backtrace extension still requires tasks.md for plan scope. This is a known limitation documented as a follow-up issue, not addressed in this spec.
- A top-level metadata wrapper for the findings JSON was rejected to maintain backward compatibility with backtrace and sdd-drive, which expect a bare JSON array.
- Adding new fields to GapFinding objects (like `categories_audited`) is backward-compatible because downstream consumers (backtrace, sdd-drive) validate required fields and ignore unknown fields. The new field is additive and will not cause parse failures.
- FR-012 adds `categories_audited` to all scopes (spec, plan full, plan degraded) for consistency. Downstream tools can always rely on this field being present regardless of scope, rather than needing to infer categories from the scope value.
