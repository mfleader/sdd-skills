# Feature Specification: SDD Gap Audit

**Feature Branch**: `001-gap-audit-extension`  
**Created**: 2026-04-27  
**Status**: Draft  
**Input**: User description: "A speckit extension that packages a single command: an adversarial gap auditor for SDD artifacts"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Audit a Spec for Gaps (Priority: P1)

A skill author has written a spec for a new SDD skill and wants to catch specification-level gaps before investing time in planning. They invoke the gap audit command with `spec` focus. The auditor reads the spec, dispatches an adversarial subagent, applies false positive filters to the raw findings, and presents them grouped by blocking/non-blocking.

**Why this priority**: Spec-level auditing is the first point where gaps can be caught. Finding orphan FRs, weak ACs, and implicit assumptions before planning prevents cascading errors downstream.

**Independent Test**: Can be fully tested by creating a spec with known gaps (an orphan FR, a weak AC, an unverifiable SC) and confirming the auditor identifies each one without false positives.

**Acceptance Scenarios**:

1. **Given** a spec directory containing `spec.md` with orphan FRs, weak ACs, and cross-reference gaps, **When** the user invokes the gap audit command with focus `spec`, **Then** the auditor presents at least one finding per gap category present in the spec, each with classification, category, description, evidence, and suggested_fix.
2. **Given** a spec directory containing `spec.md` with no gaps, **When** the user invokes the gap audit command with focus `spec`, **Then** the auditor reports no issues found.
3. **Given** a spec directory containing `spec.md` where an FR references an existing function but does not state the specific behavior it depends on, **When** the user invokes the gap audit command with focus `spec`, **Then** the auditor reports an "implicit assumptions" finding citing the FR and the undocumented dependency.

---

### User Story 2 - Audit Plan and Tasks for Gaps (Priority: P1)

A skill author has completed the spec, plan, and tasks phases and wants to verify that the plan and tasks adequately cover the spec before implementation. They invoke the gap audit command with `plan` focus. The auditor reads `spec.md`, `plan.md`, and `tasks.md`, dispatches an adversarial subagent, applies false positive filters, and presents the findings.

**Why this priority**: Plan-level auditing catches integration gaps, missing contract tests, and spec coverage gaps that would otherwise surface during implementation or code review.

**Independent Test**: Can be fully tested by creating a spec with edge cases and a plan/tasks set that deliberately omits a contract test and an integration scenario. Confirm the auditor identifies the missing coverage.

**Acceptance Scenarios**:

1. **Given** a spec directory containing `spec.md`, `plan.md`, and `tasks.md` where one spec edge case has no corresponding task, **When** the user invokes the gap audit command with focus `plan`, **Then** the auditor reports a finding in the "edge case coverage" category citing the missing edge case.
2. **Given** a spec directory where an FR modifies a computed output but no task asserts the final output against an independently calculated expected value, **When** the user invokes the gap audit command with focus `plan`, **Then** the auditor reports an "edge case coverage" finding specifying that end-to-end output verification is missing.
3. **Given** a spec directory where all spec requirements are fully covered by plan steps and tasks, **When** the user invokes the gap audit command with focus `plan`, **Then** the auditor reports no issues found.
4. **Given** a spec directory where two components interact but no contract test exists at their API boundary, **When** the user invokes the gap audit command with focus `plan`, **Then** the auditor reports a "contract tests" finding citing the boundary and the missing argument/contract verification.
5. **Given** a spec directory where a scenario requires real components but no task covers integration testing for it, **When** the user invokes the gap audit command with focus `plan`, **Then** the auditor reports an "integration gaps" finding citing the scenario and the missing coverage.

---

### User Story 3 - False Positive Filters Remove Noise (Priority: P2)

A skill author invokes the gap audit and the raw subagent findings include false positives: a technical claim that doesn't match the actual API, a gap that existing tasks already cover, or grouped findings with different root causes. The false positive filters remove these before presenting results.

**Why this priority**: Precision over recall is the first constitutional principle. If the auditor produces noisy findings, users lose trust and stop using it.

**Independent Test**: Can be tested by mocking raw subagent output containing known false positives (e.g., a claim about an API that doesn't exist, a gap for which a task already exists) and verifying the false positive filters remove them.

**Acceptance Scenarios**:

1. **Given** raw subagent findings that include a technical claim about an API that does not exist in the codebase, **When** false positive filters are applied, **Then** that finding is removed from the final output.
2. **Given** raw subagent findings that claim a gap for which a task already provides coverage, **When** false positive filters are applied, **Then** that finding is removed from the final output.
3. **Given** raw subagent findings that group two issues with different root causes into a single finding, **When** false positive filters are applied, **Then** the finding is either split into separate findings (one per root cause) or rejected from the output.

---

### User Story 4 - Installable as Speckit Extension (Priority: P2)

A user installs the gap audit extension into their speckit project using `specify extension add <name> --from <git-url>` or `--dev`. After installation, the gap audit command is available for invocation. The extension does not register lifecycle hooks.

**Why this priority**: The extension must be distributable and installable following speckit conventions to be useful to anyone beyond the author.

**Independent Test**: Can be tested by running `specify extension add` in a fresh speckit project and confirming the gap audit command is discoverable and executable.

**Acceptance Scenarios**:

1. **Given** a speckit project (>= 0.5.2) with no extensions installed, **When** the user runs `specify extension add <name> --from <git-url>`, **Then** the extension is installed and the gap audit command appears in the available commands list.
2. **Given** the extension is installed, **When** the user lists registered lifecycle hooks, **Then** no hooks from this extension are present.
3. **Given** the extension is installed in a speckit project running a version below 0.5.2, **When** the user attempts to invoke the gap audit command, **Then** a clear error message indicates the minimum speckit version requirement.

---

### User Story 5 - Persist Findings and Match Patterns (Priority: P3)

A skill author wants to save the audit findings for later reference or automated processing, or has a `gap-patterns.md` file with recurring patterns from previous audits. They invoke the gap audit command with the `--output` flag. The command writes findings to a JSON file in the spec directory. When patterns exist, matching findings are tagged with the canonical pattern name.

**Why this priority**: JSON persistence and pattern matching are power-user features. The core value is the audit itself (US1/US2). Persistence enables automation and tracking but is not required for the primary workflow.

**Independent Test**: Can be tested by running the gap audit with and without `--output` and verifying that the JSON file is only written when the flag is set. Pattern matching can be tested by providing a `gap-patterns.md` with known patterns and verifying the `pattern_match` field appears on matching findings.

**Acceptance Scenarios**:

1. **Given** a spec directory with gaps, **When** the user invokes the gap audit with `--output`, **Then** the command writes findings to `<spec_dir>/.sdd-findings-spec.json` (spec focus) or `<spec_dir>/.sdd-findings-plan.json` (plan focus) as valid JSON.
2. **Given** a spec directory with gaps, **When** the user invokes the gap audit without `--output`, **Then** no findings file is written to the spec directory.
3. **Given** a spec directory with a `gap-patterns.md` file containing recurring patterns, **When** the auditor finds a gap that matches a known pattern, **Then** the corresponding finding includes a `pattern_match` field with the canonical pattern name.
4. **Given** `--output` is set and a findings JSON file already exists from a previous run, **When** the command completes, **Then** the previous file is overwritten with the new findings.

---

### Edge Cases

- When `spec.md` does not exist in the provided directory, the command returns an error naming the missing file (covered by FR-013).
- When the user provides an invalid focus parameter (neither `spec` nor `plan`), the command returns an error listing the valid values (covered by FR-019).
- When `plan.md` or `tasks.md` is missing but focus is `plan`, the command returns an error naming each missing file (covered by FR-013).
- When `gap-patterns.md` exists but is empty or malformed, the command treats it as no patterns and proceeds without pattern matching.
- When the subagent returns no findings at all, the command reports no issues found.
- When the subagent returns findings that all fail false positive filters, the command reports no issues found.
- When `--output` is set and the findings JSON file already exists from a previous run, the command overwrites it (covered by FR-014).

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The command MUST accept a focus parameter with two valid values: `spec` and `plan`.
- **FR-002**: When focus is `spec`, the command MUST read `spec.md` from the spec directory and audit it against the spec checklist (orphan FRs, weak ACs, unverifiable SCs, cross-reference gaps, implicit assumptions, naming collisions, implicit behavior).
- **FR-003**: When focus is `plan`, the command MUST read `spec.md`, `plan.md`, and `tasks.md` from the spec directory and audit them against the plan checklist (contract tests, integration gaps, edge case coverage, implicit behavior, spec coverage gaps).
- **FR-004**: The command MUST dispatch an adversarial auditor subagent using the `superpowers:code-reviewer` subagent type with the full checklist, classification scheme, and output constraints embedded in the prompt.
- **FR-005**: The subagent MUST NOT modify any files.
- **FR-006**: The command MUST apply 8 false positive filters to raw subagent findings before reporting: (1) verify technical claims against actual APIs, (2) cross-reference tasks before claiming gaps, (3) distinguish FRs from acceptance scenarios, (4) don't group findings with different root causes, (5) classify pre-existing behavior correctly, (6) accept human-verified criteria, (7) acknowledge existing coverage before claiming gaps, (8) default to "plan diverges from spec" framing.
- **FR-007**: When the `--output` flag is set, the command MUST write findings to `<spec_dir>/.sdd-findings-spec.json` for spec focus and `<spec_dir>/.sdd-findings-plan.json` for plan focus. When the flag is not set, the command MUST NOT write a findings file.
- **FR-008**: Each finding MUST contain these fields: `classification` (blocking or non-blocking), `category`, `description`, `evidence`, `suggested_fix`.
- **FR-009**: When a finding matches a known recurring pattern from `gap-patterns.md`, the finding MUST include a `pattern_match` field with the canonical pattern name.
- **FR-010**: The command MUST optionally load `specs/gap-patterns.md` (project-level) if it exists and include its patterns in the subagent prompt.
- **FR-011**: The command MUST present findings to the user grouped by blocking/non-blocking classification.
- **FR-012**: Superseded by FR-024 (explicit resolution order).
- **FR-013**: The command MUST return an error if the required artifact files do not exist for the selected focus.
- **FR-014**: When `--output` is set and a findings JSON file already exists from a previous run, the command MUST overwrite it.
- **FR-015**: The extension MUST be packaged as a speckit extension with `extension.yml`, a `commands/` directory, and no lifecycle hooks.
- **FR-016**: The extension MUST declare a minimum speckit version requirement of >= 0.5.2.
- **FR-017**: The extension MUST NOT depend on spex-gates or other extensions.
- **FR-018**: The command MUST NOT fix any findings. It is report-only.
- **FR-019**: When the focus parameter is not `spec` or `plan`, the command MUST return an error listing the valid values.
- **FR-020**: The command MUST complete its audit in a single subagent dispatch. It MUST NOT iterate or perform multi-round refinement of findings.
- **FR-021**: When the audit identifies gaps, the command MUST produce at least one finding per gap category that has gaps. No category may be silently skipped.
- **FR-022**: When false positive filter 4 detects a finding that groups issues with different root causes, the filter MUST either split it into separate findings (one per root cause) or reject it from the output.
- **FR-023**: When a false positive filter cannot determine whether a finding is valid or invalid, the finding MUST pass through with the evidence field noting "unverified: [reason]".
- **FR-024**: The command MUST resolve the spec directory path in this order: (1) path provided as argument, (2) `.specify/feature.json` feature_directory value, (3) ask the user. If none succeeds, return an error.
- **FR-025**: At invocation, the command MUST check the speckit version and return a clear error stating the minimum version requirement (>= 0.5.2) if the installed version is below it.

### Key Entities

- **GapFinding**: A single gap identified by the auditor. Contains classification, category, description, evidence, suggested_fix, and optionally pattern_match. Named GapFinding to distinguish from spex-deep-review's Finding entity, which uses a different schema (severity/confidence instead of blocking/non-blocking classification).
- **False Positive Filter**: A validation check applied to raw subagent findings to remove false positives and improve precision. There are 8 named filters.
- **Audit Scope**: The scope selector. Either `spec` (spec artifacts only) or `plan` (spec + plan + tasks artifacts).
- **Spec Checklist**: The 7-item checklist used for spec-focus auditing (orphan FRs, weak ACs, unverifiable SCs, cross-reference gaps, implicit assumptions, naming collisions, implicit behavior).
- **Plan Checklist**: The 5-item checklist used for plan-focus auditing (contract tests, integration gaps, edge case coverage, implicit behavior, spec coverage gaps).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: The auditor identifies at least 80% of intentionally planted gaps in a test spec (recall floor), measured against a test suite of specs with known, documented gaps.
- **SC-002**: No more than 20% of reported findings are false positives (precision floor), measured by human review of findings against the audited artifacts. A false positive is a finding where the cited evidence does not support the claimed gap when read in context of the full artifact, or where the claimed gap does not exist in the artifact.
- **SC-003**: All findings cite specific evidence (file reference, section, or quote) verifiable by reading the referenced artifact.
- **SC-004**: The extension installs successfully via `specify extension add` in a speckit >= 0.5.2 project without manual configuration.
- **SC-005**: The gap audit command completes within the time limits of a single subagent dispatch (no multi-round iteration).
- **SC-006**: Findings output is valid JSON parseable by standard JSON tools without errors.
- **SC-007**: False positive filters demonstrably remove at least one false positive from a test run with known false positives in the raw subagent output.
- **SC-008**: When `gap-patterns.md` exists with known patterns, findings matching those patterns include the `pattern_match` field with the correct canonical name, verified against a test spec with planted pattern-matching gaps.

## Assumptions

- The user has a speckit project initialized with at least version 0.5.2.
- Spec, plan, and task artifacts follow the standard speckit template structure and naming conventions (`spec.md`, `plan.md`, `tasks.md`).
- The `superpowers:code-reviewer` subagent type is available in the user's Claude Code environment. This type is used instead of `general-purpose` because it loads the code-reviewer skill instructions, which provide structured review discipline (checklist adherence, evidence requirements, classification rigor) that a general-purpose agent does not have by default.
- `gap-patterns.md` is a project-level file (in the `specs/` directory, not per-feature) that catalogs gap types that have recurred across 2+ previous audits. Its purpose is to help the auditor recognize repeat offenders. Each pattern entry has three fields: a canonical name (e.g., `missing-condition-false-path`), the checklist category it applies to (e.g., `Weak ACs`), and a trigger description explaining what to look for. Pattern matching is category-based: a finding matches a pattern when the finding's category matches the pattern's category and the finding's description aligns with the pattern's trigger condition.
- The extension will be distributed as a git repository installable via speckit's extension mechanism.
- The subagent has read access to files in the spec directory and the project codebase for cross-referencing.
