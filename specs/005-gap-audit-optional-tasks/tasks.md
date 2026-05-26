# Tasks: Gap Audit Optional tasks.md

**Input**: Design documents from `specs/005-gap-audit-optional-tasks/`
**Prerequisites**: plan.md (required), spec.md (required), research.md, data-model.md, contracts/

**Tests**: Not explicitly requested. No test tasks included.

**Organization**: Tasks grouped by user story. All changes are to LLM-executed prose (command markdown) and documentation files.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Phase 1: Setup

**Purpose**: Version bump and prerequisite changes

- [ ] T001 Bump version from 0.1.0 to 0.2.0 in .specify/extensions/gap-audit/extension.yml (FR-016)

---

## Phase 2: Foundational (Command File Core Changes)

**Purpose**: Modify the command file sections that all user stories depend on. These changes to `speckit.gap-audit.audit.md` must be complete before user story verification.

- [ ] T002 Modify Section 4 (Artifact Loading) plan scope in .specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md: change tasks.md from required to optional. When tasks.md is missing, emit info message "tasks.md not found, running plan-vs-spec audit only (categories 4-5)." and continue. When present, load it as before. Keep plan.md and spec.md required. Establish a clear state indicator (e.g., "tasks.md was loaded" / "tasks.md was not loaded") that downstream sections (6, 8, 9, 10) can reference. (FR-001, FR-004, FR-014)
- [ ] T003 Modify Section 6 (Subagent Prompt Construction) plan scope preamble in .specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md: make the checklist preamble conditional on tasks.md presence. When present: "Check the plan and tasks against each of these 5 categories, using the spec as the source of truth." When absent: "Check the plan against the spec using these 2 categories." (FR-005)
- [ ] T004 Modify Section 6 plan checklist in .specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md: when tasks.md is absent, include only categories 4 (Implicit behavior) and 5 (Spec coverage gaps). When present, include all 5 categories. (FR-003, FR-002)
- [ ] T005 Modify Section 6 artifact embedding in .specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md: only include the tasks.md artifact block when tasks.md was loaded. (FR-006)
- [ ] T006 Modify Section 6 category validation directive in .specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md: restrict valid categories to the active checklist. When tasks.md absent, valid categories are "Implicit behavior" and "Spec coverage gaps" only. (FR-007)
- [ ] T007 Modify Section 6 Filter 2 (cross-reference tasks) in .specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md: change skip condition from "when audit scope is spec" to "when tasks.md is not an input artifact." (FR-008)
- [ ] T008 Modify Section 8 (Response Parsing) in .specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md: add a category validation step after JSON extraction. Reject any finding whose category does not match an active checklist category. Exclude rejected findings from output and persistence. Verify: if a finding has category "Contract tests" but only categories 4-5 are active, it is excluded. (FR-017)
- [ ] T009 Modify Section 9 (JSON Persistence) in .specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md: add categories_audited field to each finding when --output is set for ALL scopes (spec, plan full, plan degraded). Use exact arrays per FR-012: degraded plan ["Implicit behavior", "Spec coverage gaps"], full plan ["Contract tests", "Integration gaps", "Edge case coverage", "Implicit behavior", "Spec coverage gaps"], spec ["Orphan FRs", "Weak ACs", "Unverifiable SCs", "Cross-reference gaps", "Implicit assumptions", "Naming collisions", "Implicit behavior"]. (FR-012, FR-013)
- [ ] T010 Modify Section 10 (Output Formatting) in .specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md: when plan scope with tasks.md absent and findings exist, summary reads "N blocking, M non-blocking findings (categories 4-5 only, tasks.md not available)." When no findings, output "No issues found." When tasks.md present, summary unchanged. (FR-010, FR-011, FR-018)

**Checkpoint**: Command file changes complete. Gap-audit plan scope now supports degraded mode.

---

## Phase 3: User Story 1 - Degraded Plan Audit (Priority: P1) MVP

**Goal**: Plan scope works without tasks.md, auditing categories 4-5 only.

**Independent Test**: Invoke `/speckit.gap-audit.audit plan` on a spec directory with spec.md and plan.md but no tasks.md. Confirm info message, findings in categories 4-5 only, partial-audit summary.

- [ ] T011 [US1] Verify degraded mode end-to-end: run `/speckit.gap-audit.audit plan --output` against a spec directory with spec.md and plan.md (no tasks.md). Confirm: info message emitted before audit, findings only in categories 4-5, summary contains "(categories 4-5 only, tasks.md not available)", JSON file has categories_audited field with value ["Implicit behavior", "Spec coverage gaps"] on each finding.

**Checkpoint**: US1 independently testable.

---

## Phase 4: User Story 2 - Full Plan Audit Backward Compatibility (Priority: P1)

**Goal**: Plan scope with all three files produces identical behavior to pre-change implementation.

**Independent Test**: Invoke `/speckit.gap-audit.audit plan` on a spec directory with all three files. Confirm all 5 categories audited, no partial-audit indicator.

- [ ] T012 [US2] Verify full mode end-to-end: run `/speckit.gap-audit.audit plan --output` against a spec directory with spec.md, plan.md, and tasks.md. Confirm: no info message about degraded mode, findings across all 5 categories, summary has no partial-audit qualifier, JSON file has categories_audited with value ["Contract tests", "Integration gaps", "Edge case coverage", "Implicit behavior", "Spec coverage gaps"] on each finding.

**Checkpoint**: US2 independently testable. Backward compatibility confirmed.

---

## Phase 5: User Story 3 - Pattern Filtering in Degraded Mode (Priority: P2)

**Goal**: Gap-patterns for inactive categories are excluded from subagent prompt in degraded mode.

**Independent Test**: Create gap-patterns.md with patterns spanning categories 1-5. Run plan scope without tasks.md. Confirm only category 4-5 patterns appear in prompt.

- [ ] T013 [US3] Modify Section 6 (Subagent Prompt Construction) pattern matching in .specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md: when gap-patterns.md is loaded and scope is plan, filter patterns to only those matching active categories. If no patterns remain after filtering, omit pattern matching section entirely. Verify: with a gap-patterns.md containing only categories 1-3 patterns and tasks.md absent, confirm the pattern matching section is completely omitted from the subagent prompt. (FR-009)

**Checkpoint**: US3 independently testable.

---

## Phase 6: Documentation Updates

**Purpose**: Update all documentation and spec artifacts to reflect the new behavior.

- [ ] T014 [P] Update specs/001-gap-audit-extension/spec.md: FR-003 (tasks.md optional), FR-013 (scope narrows), US2 narrative and independent test, new acceptance scenario for degraded mode, edge cases split, key entities, assumptions.
- [ ] T015 [P] Update specs/001-gap-audit-extension/contracts/command-schema.md: input artifacts table (tasks.md required=no), error conditions (tasks.md missing = info not error).
- [ ] T016 [P] Update specs/001-gap-audit-extension/data-model.md: add categories_audited field to GapFinding, update validation rules.
- [ ] T017 [P] Update .specify/extensions/gap-audit/README.md: update "Audit plan and tasks for gaps" section to reflect tasks.md optionality. Add note under usage that backtrace plan scope still requires tasks.md (known limitation, follow-up issue).
- [ ] T018 [P] Update .claude/skills/speckit-gap-audit-audit/SKILL.md: update description from "plan (spec.md + plan.md + tasks.md)" to "plan (spec.md + plan.md, optionally tasks.md)".
- [ ] T019 [P] Update README.md: add parenthetical to Extensions table description noting tasks optional for plan scope.

- [ ] T020 [P] Verify spec scope regression: run `/speckit.gap-audit.audit spec` after all changes and confirm behavior is identical to pre-change (same prompt structure, same filters, same output format). Confirms FR-015 and guards against Section 4 changes affecting spec scope. (FR-015)

**Checkpoint**: All documentation reflects the new behavior. Spec scope regression confirmed.

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies, start immediately
- **Foundational (Phase 2)**: Depends on Phase 1 (version bump)
- **US1 Verification (Phase 3)**: Depends on Phase 2 (command changes)
- **US2 Verification (Phase 4)**: Depends on Phase 2 (command changes), parallel with Phase 3
- **US3 Implementation (Phase 5)**: Depends on Phase 2 (command changes), parallel with Phases 3-4
- **Documentation (Phase 6)**: Can start after Phase 2, parallel with Phases 3-5

### User Story Dependencies

- **US1 (P1)**: Depends on Foundational. No story dependencies.
- **US2 (P1)**: Depends on Foundational. No story dependencies. Can run parallel with US1.
- **US3 (P2)**: Depends on Foundational. No story dependencies. Can run parallel with US1/US2.

### Parallel Opportunities

- T014-T019 (all documentation tasks) can run in parallel with each other
- T011, T012, T013 can run in parallel (after Phase 2)
- T002-T010 are sequential within the same file but logically independent edits

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Version bump
2. Complete Phase 2: Command file changes (T002-T010)
3. Complete Phase 3: Verify degraded mode (T011)
4. **STOP and VALIDATE**: Test plan scope without tasks.md

### Incremental Delivery

1. Setup + Foundational → Command file ready
2. US1 verification → Degraded mode works
3. US2 verification → Backward compat confirmed
4. US3 implementation + verification → Pattern filtering works
5. Documentation → All artifacts updated

---

## Notes

- All "source code" changes are to LLM-executed markdown prose, not traditional code
- T002-T010 modify sections of a single file (speckit.gap-audit.audit.md) but are split by section for clarity
- T011 and T012 are verification tasks, not implementation — they confirm the Phase 2 changes work
- T013 is both implementation and verification (adds pattern filtering to the command file)
