# Tasks: Backtrace Extension

**Input**: Design documents from `specs/002-backtrace-extension/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/command-schema.md

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Phase 1: Setup (Extension Structure)

**Purpose**: Create the extension directory structure and metadata

- [X] T001 Create extension directory structure: `.specify/extensions/backtrace/commands/`
- [X] T002 Create extension.yml at `.specify/extensions/backtrace/extension.yml` per research.md R-005 (schema_version, extension metadata, requires, provides, tags; no hooks section)
- [X] T003 [P] Create SKILL.md wrapper at `.claude/skills/speckit-backtrace-trace/SKILL.md` per research.md R-006 (front matter: name, description, compatibility, metadata.source, user-invocable: true, disable-model-invocation: false; body delegates to command file)

---

## Phase 2: Foundational (Command File Skeleton)

**Purpose**: Create the command file with all 10 sections as stubs, then fill in the non-story-specific sections

- [X] T004 Create command file skeleton at `.specify/extensions/backtrace/commands/speckit.backtrace.trace.md` with YAML front matter (`description: "Trace gap-audit findings back to missing spec items and propose additions"`) and 10 section headings matching research.md R-002
- [X] T005 Fill Section 1 (Ship Pipeline Guard): check `.specify/.spex-state` for autonomous mode per the pattern in speckit-spex-gates-review-spec (read status and ask fields, set AUTONOMOUS_MODE)
- [X] T006 Fill Section 2 (Overview): brief description of backtrace purpose and usage
- [X] T007 Fill Section 3 (Prerequisites): speckit version check from `.specify/init-options.json` per FR-015a; validate `.specify/` directory exists
- [X] T008 Fill Section 4 (Argument Parsing): parse `$ARGUMENTS` for scope (`spec`/`plan`), optional spec directory path, and `--output` flag handling (error: not supported). Validate scope per FR-002. Error messages per contracts/command-schema.md
- [X] T009 Fill Section 5 (Spec Directory Resolution): three-step resolution per FR-002a and research.md R-004 (argument, feature.json, ask user). Validate spec.md exists. For plan scope, validate plan.md and tasks.md exist

**Checkpoint**: Command file has all infrastructure sections filled. Core execution sections (6-10) are stubs.

---

## Phase 3: User Story 1 - Trace Findings to Spec (Priority: P1) MVP

**Goal**: Trace gap-audit findings back to spec items and propose additions

**Independent Test**: Invoke backtrace with scope `spec` against a spec with known gaps. Verify each finding is traced to a specific FR/AC/SC and a ProposedAddition is drafted.

### Implementation for User Story 1

- [X] T010 [US1] Fill Section 5b (Findings Resolution): within the resolved spec directory, check for `.sdd-findings-{scope}.json` per FR-001. If not found, ask user for file path. Validate file exists and is valid GapFinding JSON per data-model.md schema. Handle errors: file not found, invalid format, empty findings list
- [X] T011 [US1] Fill Section 6a (Artifact Loading): read spec.md (both scopes) and optionally plan.md + tasks.md (plan scope). Store full content for subagent prompt
- [X] T012 [US1] Fill Section 6b (Core Tracing Logic): for each GapFinding, identify which AC/FR/SC should have caught it using the four tracing categories from FR-003 (missing test coverage, missing integration scenario, implicit assumption, edge case). For untraceable findings, propose a new FR or AC. Draft ProposedAddition per data-model.md schema
- [X] T013 [US1] Fill Section 7 (Subagent Dispatch): construct the auditor prompt per research.md R-003 and contracts/command-schema.md auditor contract. Include: original findings in XML tags, proposed additions in XML tags, spec artifacts in XML tags, classification instructions (approve/reject/revise), directive "DO NOT modify any files". Dispatch via Agent tool with `subagent_type: "superpowers:code-reviewer"`
- [X] T014 [US1] Fill Section 8 (Response Parsing): parse auditor response as JSON array of AuditorVerdict per data-model.md. Handle: valid JSON, non-JSON (extract from prose), malformed JSON (warn, treat as empty), partial responses (apply valid verdicts, warn about rest)
- [X] T015 [US1] Fill Section 8b (Apply Additions): for each approved/revised verdict, edit the target artifact (spec.md or tasks.md) to add the content. Preserve existing content per FR-005. For revisions, incorporate the auditor's suggested revision
- [X] T016 [US1] Fill Section 9 (Output Formatting): present results per contracts/command-schema.md output format. Applied additions section (with approval status), rejected additions section (with reasons). Post-dispatch verification: `git diff --name-only` to verify no unauthorized file modifications

**Checkpoint**: Backtrace can trace findings, get auditor approval, and apply additions to spec.md.

---

## Phase 4: User Story 2 - Plan Scope Tracing (Priority: P1)

**Goal**: Trace findings against spec, plan, and task artifacts

**Independent Test**: Invoke backtrace with scope `plan` against a spec directory with spec.md, plan.md, and tasks.md. Verify findings are traced against all three artifacts.

### Implementation for User Story 2

- [X] T017 [US2] Extend Section 6a to load plan.md and tasks.md when scope is `plan`. Include all three artifacts in subagent prompt
- [X] T018 [US2] Extend Section 6b tracing logic to propose additions targeting tasks.md (new tasks) in addition to spec.md. Plan.md additions limited to new tasks referencing existing plan phases per FR-004
- [X] T019 [US2] Extend Section 8b to apply additions to tasks.md in addition to spec.md

**Checkpoint**: Backtrace works for both spec and plan scopes.

---

## Phase 5: User Story 3 - Auditor-Gated Changes (Priority: P1)

**Goal**: Ensure auditor review gates all changes

**Independent Test**: Provide proposed additions where some should be rejected. Verify rejected additions are not applied and reasons are reported.

### Implementation for User Story 3

- [X] T020 [US3] Verify Section 8b correctly skips rejected additions (no-op for rejects). Verify revisions are incorporated. Add explicit logging for each verdict type
- [X] T021 [US3] Add edge case handling: all proposals rejected (report "All proposals rejected" with reasons, no artifacts modified per edge case spec)

**Checkpoint**: Auditor gating is fully functional. Rejects never applied, revisions correctly incorporated.

---

## Phase 6: User Story 4 - Follow-Up Reviews (Priority: P2)

**Goal**: Automatically invoke review-spec, review-plan, and analyze after backtrace completes

**Independent Test**: Run backtrace with spex-gates installed. Verify follow-up reviews are invoked. Run backtrace without spex-gates. Verify warning is shown and backtrace exits normally.

### Implementation for User Story 4

- [X] T022 [US4] Fill Section 10 (Follow-Up Invocations): after output formatting, check extensions registry (`.specify/extensions/.registry`) for spex-gates availability per FR-012. If available, invoke review-spec. For plan scope, also invoke review-plan. Invoke analyze if available
- [X] T023 [US4] Add soft dependency handling: if spex-gates is not installed, log warning "spex-gates not installed, skipping follow-up reviews" and continue. No error per FR-013
- [X] T024 [US4] Handle autonomous mode in follow-up invocations: if AUTONOMOUS_MODE is true from Section 1, suppress user prompts during follow-up invocations

**Checkpoint**: Follow-up reviews fire after backtrace. Graceful degradation when spex-gates absent.

---

## Phase 7: User Story 5 - Reset Trigger Detection (Priority: P2)

**Goal**: Warn user when additions cross reset trigger thresholds

**Independent Test**: Apply additions that include a new user story and 3+ new SCs. Verify reset trigger warning is displayed.

### Implementation for User Story 5

- [X] T025 [US5] Add Section 8c (Reset Trigger Check) after applying additions: count new user stories, new non-refinement FRs, SC count delta, and material scope expansion per FR-009. Read constitution thresholds from `.specify/memory/constitution.md` if exists; use defaults otherwise
- [X] T026 [US5] Add reset trigger output to Section 9: if triggers fired, append "Reset Trigger Warning" section per contracts/command-schema.md output format

**Checkpoint**: Reset triggers detected and reported. No action taken, user decides.

---

## Phase 8: Documentation

**Purpose**: Extension README and project README update

- [X] T027 [P] Create extension README at `.specify/extensions/backtrace/README.md` with: requirements, installation, usage (command syntax and arguments), findings input format, output format, follow-up reviews, artifact interactions per FR-017
- [X] T028 [P] Update project README.md: add backtrace to extensions table, add usage examples, add copy-paste installation commands for both gap-audit and backtrace extensions per FR-016

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies
- **Foundational (Phase 2)**: Depends on Phase 1
- **US1 (Phase 3)**: Depends on Phase 2 (command file skeleton)
- **US2 (Phase 4)**: Depends on Phase 3 (extends tracing logic)
- **US3 (Phase 5)**: Depends on Phase 3 (verifies auditor flow)
- **US4 (Phase 6)**: Depends on Phase 3 (adds post-execution step)
- **US5 (Phase 7)**: Depends on Phase 3 (adds post-apply check)
- **Docs (Phase 8)**: Depends on all user stories

### User Story Dependencies

- **US1 (P1)**: Foundational. All other stories depend on this.
- **US2 (P1)**: Extends US1 for plan scope. Can start after US1.
- **US3 (P1)**: Verifies US1 auditor flow. Can start after US1.
- **US4 (P2)**: Adds follow-up step. Can start after US1. Independent of US2/US3.
- **US5 (P2)**: Adds reset check. Can start after US1. Independent of US2/US3/US4.

### Parallel Opportunities

- T002 and T003 can run in parallel (different files)
- US2, US3, US4, US5 can all start after US1 completes (they extend different sections)
- T027 and T028 can run in parallel (different files)

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (T001-T003)
2. Complete Phase 2: Foundational (T004-T009)
3. Complete Phase 3: User Story 1 (T010-T016)
4. **STOP and VALIDATE**: Test backtrace with spec scope against test fixtures
5. If working: MVP is deliverable

### Incremental Delivery

1. Setup + Foundational → Skeleton ready
2. US1 → Core tracing works → MVP
3. US2 → Plan scope works
4. US3 → Auditor gating verified
5. US4 → Follow-up reviews automated
6. US5 → Reset triggers detected
7. Docs → Ready for distribution

---

## Notes

- This is a prompt-engineering project. "Implementation" means writing markdown command sections, not compiled code.
- Each task fills one or more sections of the command file at `.specify/extensions/backtrace/commands/speckit.backtrace.trace.md`
- The command file follows the 10-section structure from research.md R-002
- Test fixtures from gap-audit (`specs/001-gap-audit-extension/test-fixtures/`) can be adapted for backtrace testing
