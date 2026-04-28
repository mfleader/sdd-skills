# Feature: Backtrace Extension

## Purpose

Backtrace closes the loop between finding gaps and fixing them. When a gap audit (or other source) identifies missing coverage in spec artifacts, backtrace traces each gap back to the spec item that should have caught it, proposes additions, gets adversarial auditor approval, and applies the approved changes.

## User Scenarios & Testing

### US-001: Trace gap-audit findings back to spec (P1)

**As** a developer using the SDD workflow,
**I want** to trace gap-audit findings back to the spec artifacts that should have caught them,
**so that** I can strengthen the spec to prevent similar gaps in the future.

**Acceptance Scenarios:**

1. **Given** a set of gap-audit findings and a spec directory, **When** the user invokes the backtrace command with scope `spec`, **Then** each finding is traced to a specific FR, AC, or SC that should have caught it, and proposed additions are drafted.
2. **Given** proposed additions, **When** the adversarial auditor reviews them, **Then** only approved additions are applied to the spec artifacts.
3. **Given** all proposals are rejected by the auditor, **When** backtrace completes, **Then** no artifacts are modified and the user sees the rejection reasons.

**Why this priority:** Core functionality. Without tracing, backtrace has no purpose.

### US-002: Trace findings with plan scope (P1)

**As** a developer who has completed planning,
**I want** to trace findings against spec, plan, and task artifacts,
**so that** gaps in planning are also addressed.

**Acceptance Scenarios:**

1. **Given** findings and a spec directory containing spec.md, plan.md, and tasks.md, **When** the user invokes backtrace with scope `plan`, **Then** findings are traced against all three artifacts and additions may be proposed for any of them.
2. **Given** a required artifact is missing (e.g., plan.md not found), **When** the user invokes backtrace with scope `plan`, **Then** backtrace stops with a clear error identifying the missing file.

**Why this priority:** Plan-scope tracing is essential for the full SDD workflow.

### US-003: Auditor-gated changes (P1)

**As** a developer,
**I want** an adversarial auditor to review proposed spec additions before they are applied,
**so that** I don't introduce redundant, inconsistent, or incorrect requirements.

**Acceptance Scenarios:**

1. **Given** proposed additions, **When** the auditor classifies a proposal as "revise" with a suggested revision, **Then** the revision is incorporated and the revised addition is applied.
2. **Given** proposed additions, **When** the auditor rejects a proposal with a reason, **Then** the rejection and reason are reported to the user and the proposal is not applied.

**Why this priority:** Auditor gating prevents spec degradation from unchecked automated edits.

### US-004: Follow-up review hooks (P2)

**As** a developer,
**I want** review-spec, review-plan, and analyze to run automatically after backtrace completes,
**so that** I can verify the spec edits didn't introduce inconsistencies.

**Acceptance Scenarios:**

1. **Given** the backtrace extension is installed with hooks declared, **When** backtrace completes and additions were applied, **Then** follow-up review commands are triggered via the speckit hook system.
2. **Given** the backtrace extension is installed with hooks declared, **When** backtrace completes but all additions were rejected, **Then** follow-up hooks still fire (they verify the artifacts are consistent even without changes).
3. **Given** spex-gates is not installed, **When** backtrace completes, **Then** hooks are skipped with a warning and backtrace exits normally.

**Why this priority:** Hooks automate verification. Without them the user must manually invoke reviews, which is viable but tedious.

### US-005: Reset trigger detection (P2)

**As** a developer,
**I want** backtrace to warn me if proposed additions cross a reset trigger threshold,
**so that** I can decide whether to re-plan before continuing implementation.

**Acceptance Scenarios:**

1. **Given** applied additions include a new user story, **When** backtrace checks reset triggers, **Then** it reports the trigger to the user without taking action.
2. **Given** applied additions include a new non-refinement FR, **When** backtrace checks reset triggers, **Then** it reports the trigger.
3. **Given** applied additions change the SC count by 3 or more, **When** backtrace checks reset triggers, **Then** it reports the trigger.
4. **Given** applied additions represent material scope expansion, **When** backtrace checks reset triggers, **Then** it reports the trigger.

**Why this priority:** Prevents silent scope creep. The user retains control over re-planning decisions.

## Requirements

### Functional Requirements

**Backtrace command (`speckit.backtrace.trace`):**

- **FR-001**: Accept a list of findings (GapFinding JSON array) and a spec directory path. Findings are resolved in this order: (1) check the spec directory for `.sdd-findings-{scope}.json` (produced by gap-audit `--output`), (2) if not found, ask the user for a file path. If the user-provided path does not exist or is not readable, stop with "Findings file not found: `<path>`".
- **FR-002**: Accept a scope argument (`spec` or `plan`) indicating which artifacts to trace against. If scope is missing or invalid, stop with an error listing the valid values (`spec`, `plan`).
- **FR-002a**: Resolve the spec directory path in this order: (1) path provided as an argument, (2) `.specify/feature.json` `feature_directory` value, (3) ask the user. This follows the same resolution pattern as gap-audit.
- **FR-003**: For each finding, trace it back to identify which AC, FR, or SC should have caught it:
  - Missing test coverage: what FR or AC should have required it?
  - Missing integration scenario: what SC should have verified it?
  - Implicit assumption: what constraint should be explicit in the spec?
  - Edge case not covered: what AC should have specified the boundary?
  - If a finding cannot be traced to any existing spec item, propose a new FR or AC rather than an amendment. This may trigger a reset trigger (FR-009).
- **FR-004**: For each traced gap, draft a specific addition: new or amended FR, AC, SC, or task. Additions target spec.md (for FRs, ACs, SCs) or tasks.md (for tasks). Plan.md additions are limited to new tasks referencing existing plan phases.
- **FR-005**: Additions are edits to existing artifacts, not regeneration. Preserve all existing content. Add, don't replace.
- **FR-006**: Dispatch an adversarial auditor subagent (via `superpowers:code-reviewer`) to review all proposed additions. The auditor prompt must include: the original findings, the proposed additions, instructions to verify each addition addresses the gap it claims to, no addition introduces new inconsistencies, no addition is redundant with existing content.
- **FR-007**: Auditor classifies each proposal as: **approve**, **reject** (with reason), or **revise** (with suggested revision).
- **FR-008**: Apply approved additions to the spec artifacts. Incorporate auditor revisions where suggested. Do not apply rejected additions.
- **FR-009**: After applying additions, check reset triggers: new user story, new non-refinement FR, SC count change by 3+, or material scope expansion. If a constitution file exists (`.specify/memory/constitution.md`), use its thresholds. If no constitution exists, use these defaults. Report if triggered but do not take action.
- **FR-010**: One pass only. No iteration.
- **FR-011**: Console output: list of additions applied (with approval status for each), list of additions rejected (with reasons).

**Follow-up hooks:**

- **FR-012**: After completing (whether additions were applied or all were rejected), the backtrace command file directly invokes follow-up reviews: review-spec, review-plan (plan scope only), and analyze. Before invoking each, it checks the extensions registry (`.specify/extensions/.registry`) for spex-gates availability. This follows the same pattern as review-code invoking deep-review.
- **FR-013**: Follow-up reviews are a soft dependency on spex-gates. If spex-gates is not installed, follow-ups are skipped with a warning. No error. No hooks are declared in backtrace's extension.yml (speckit's hook schema only allows one command per hook point per extension, making hook-based follow-ups infeasible for three commands).

**Extension packaging:**

- **FR-014**: Packaged as a speckit extension with `extension.yml`, a `commands/` directory, and a Claude Code skill wrapper.
- **FR-015**: Follows the same structure as gap-audit: extension.yml, command markdown file, skill SKILL.md, extension README.
- **FR-015a**: At invocation, check the speckit version from `.specify/init-options.json`. If below 0.5.2, stop with "This extension requires speckit >= 0.5.2. Installed: [version]". If version cannot be determined, proceed with a warning.

**Documentation:**

- **FR-016**: Project README updated with backtrace in the extensions table, usage examples, and copy-paste installation commands for both gap-audit and backtrace extensions.
- **FR-017**: Extension README at `.specify/extensions/backtrace/README.md` with full reference: arguments, examples, artifact interactions.

### Key Entities

- **GapFinding**: A structured finding from gap-audit or other source. Contains at minimum: `classification` (blocking/non-blocking), `category`, `description`, `evidence`, `suggested_fix`.
- **ProposedAddition**: A drafted edit to a spec artifact. Contains: target artifact (spec.md or tasks.md), target section, addition type (new FR, amended FR, new AC, amended AC, new SC, new task), content, and the finding it addresses.
- **AuditorVerdict**: The auditor's classification of a proposed addition: approve, reject (with reason), or revise (with suggested revision).

## Success Criteria

- **SC-001**: Backtrace command can be invoked with findings from gap-audit and correctly traces each finding back to a specific spec artifact (AC, FR, or SC).
- **SC-002**: Proposed additions are reviewed by an adversarial auditor subagent before being applied.
- **SC-003**: Only approved or revised additions are applied to artifacts; rejected additions are not applied.
- **SC-004**: Existing artifact content is preserved. Additions only, no regeneration.
- **SC-005**: Reset trigger check fires and reports when thresholds are crossed.
- **SC-006**: Follow-up hooks trigger review-spec, review-plan, and analyze after backtrace completes.
- **SC-007**: Extension installs via `specify extension add` and follows speckit conventions.
- **SC-008**: README extensions table includes backtrace with usage examples and copy-paste install commands for both extensions.

## Error Handling

- Missing spec directory or spec.md: stop with "Required file not found: `<path>`"
- Invalid or missing scope argument: stop with "Invalid scope '[value]'. Valid values: spec, plan"
- User-provided findings file path does not exist: stop with "Findings file not found: `<path>`"
- Empty findings list: report "No findings to trace" and exit cleanly.
- Findings in unexpected format (not GapFinding JSON): stop with parse error describing the expected format.
- Auditor subagent returns non-JSON or malformed response: warn and treat as empty (no additions applied). If the response is a JSON array with some valid and some malformed verdicts, apply the valid ones and warn about the rest.
- Backtrace extension not installed when invoked: standard speckit "command not found" behavior.
- Constitution not found when checking reset triggers: use default thresholds (defined in FR-009) with a warning.

## Edge Cases

- All proposed additions rejected by auditor: report "All proposals rejected" with reasons, no artifacts modified.
- Finding traces to multiple artifacts (e.g., needs both a new FR and a new AC): propose separate additions for each.
- Finding cannot be traced to any existing spec item: propose a new FR or AC rather than an amendment.
- Reset trigger fires: report to user but do not take action. User decides whether to re-plan.
- Findings from non-gap-audit source (e.g., manual list): accepted as long as format matches GapFinding schema.

## Dependencies

- speckit >= 0.5.2
- gap-audit extension (backtrace consumes its GapFinding JSON schema, but does not require gap-audit to be installed; any source producing that schema works)
- spex-gates extension (soft dependency for follow-up hooks: review-spec, review-plan. Hooks are skipped with a warning if spex-gates is not installed.)
- speckit analyze command (for follow-up consistency check)

## Assumptions

- The GapFinding JSON schema is stable and documented by the gap-audit extension.
- The speckit hook system supports declaring hooks in an extension's `extension.yml` under a `hooks:` key with lifecycle event names (e.g., `after_specify`, `after_tasks`). Each hook entry has `command`, `optional`, and `description` fields. Core commands read these from `.specify/extensions.yml` at runtime. Verified against the spex-gates extension which declares `after_specify`, `after_tasks`, and `after_implement` hooks.
- The `superpowers:code-reviewer` subagent type is available for auditor dispatch.
- Constitution reset trigger thresholds may be defined in `.specify/memory/constitution.md`. If absent, FR-009 defines default thresholds.

## Out of Scope

- Iteration or looping (one pass only)
- `--output` JSON persistence for backtrace findings
- Gap-audit `--fix` flag (backlog, separate spec)
- Modifying the gap-audit extension
