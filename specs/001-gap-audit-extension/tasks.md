# Tasks: SDD Gap Audit

**Input**: Design documents from `specs/001-gap-audit-extension/`
**Prerequisites**: plan.md (required), spec.md (required), research.md, data-model.md, contracts/

**Tests**: No automated tests requested. Validation is manual (test specs with planted gaps, human review of precision/recall).

**Organization**: Tasks grouped by user story. All deliverables are prompt engineering artifacts (markdown + YAML), not compiled code.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to

## Path Conventions

All paths relative to repository root. Extension name is `gap-audit`.

- Extension files: `.specify/extensions/gap-audit/`
- Skill files: `.claude/skills/speckit-gap-audit-audit/`
- Example patterns: `specs/gap-patterns.md`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Create the extension directory structure and metadata.

- [ ] T001 Create extension directory structure: `.specify/extensions/gap-audit/`, `.specify/extensions/gap-audit/commands/`, `.claude/skills/speckit-gap-audit-audit/`
- [ ] T002 [P] Create `.specify/extensions/gap-audit/extension.yml` with extension id, name, version 0.1.0, description, speckit >= 0.5.2 requirement, single command registration (`speckit.gap-audit.audit`), no hooks section

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Base command structure that all user stories build on. MUST complete before any user story.

- [ ] T003 Create `.claude/skills/speckit-gap-audit-audit/SKILL.md` with frontmatter (name, description, compatibility, metadata.source pointing to `gap-audit:commands/speckit.gap-audit.audit.md`, user-invocable: true, disable-model-invocation: false) and body that loads and executes the extension command
- [ ] T004 Create `.specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md` with: frontmatter (description), argument parsing section ($ARGUMENTS, extract --output flag, extract scope), spec directory resolution (FR-024: argument > feature.json > ask user), artifact file existence checks (FR-013), error messages per contracts/command-schema.md

**Checkpoint**: Extension skeleton is installable and command responds to invocation with argument validation.

---

## Phase 3: User Story 1 - Audit a Spec for Gaps (Priority: P1) MVP

**Goal**: Invoke with `spec` scope, dispatch adversarial subagent against spec.md, present findings grouped by blocking/non-blocking.

**Independent Test**: Create a spec with known gaps (orphan FR, weak AC, unverifiable SC) and confirm the auditor identifies each one.

### Implementation for User Story 1

- [ ] T005 [US1] Add subagent prompt construction for spec scope to `.specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md`: inline the complete 7-item spec checklist (orphan FRs, weak ACs, unverifiable SCs, cross-reference gaps, implicit assumptions, naming collisions, implicit behavior) with descriptions from data-model.md directly in the prompt, embed full artifact content (spec.md), classification scheme (blocking/non-blocking), GapFinding JSON output format (per data-model.md), DO NOT modify files directive, per-category minimum (FR-021)
- [ ] T006 [US1] Add subagent dispatch section to command file: Agent tool call with subagent_type `superpowers:code-reviewer`, description `Adversarial gap audit (spec)`, full prompt from T005
- [ ] T007 [US1] Add response parsing and output formatting to command file per section order in contracts/command-schema.md: (a) parse subagent response defensively — handle valid JSON array, empty array, non-JSON prose, and malformed JSON (log warning and treat as empty on parse failure), (b) group findings by classification (blocking first, then non-blocking), present each with category/description/evidence/suggested_fix, summary line with counts, "No issues found." when empty (FR-011)

**Checkpoint**: `speckit.gap-audit.audit spec` works end-to-end on a test spec. Findings are presented grouped by blocking/non-blocking.

---

## Phase 4: User Story 2 - Audit Plan and Tasks for Gaps (Priority: P1)

**Goal**: Extend the command to handle `plan` scope, reading spec.md + plan.md + tasks.md and auditing against the 5-item plan checklist.

**Independent Test**: Create a spec with edge cases and a plan/tasks set that omits a contract test and an integration scenario. Confirm the auditor identifies the missing coverage.

### Implementation for User Story 2

- [ ] T008 [US2] Add subagent prompt construction for plan scope to command file: inline the complete 5-item plan checklist (contract tests, integration gaps, edge case coverage, implicit behavior, spec coverage gaps) with descriptions from data-model.md directly in the prompt, embed full artifact content (spec.md, plan.md, tasks.md), same classification scheme and output format as spec scope
- [ ] T009 [US2] Update subagent dispatch section to branch on scope: spec scope uses spec prompt (T005), plan scope uses plan prompt (T008), artifact loading checks for plan.md and tasks.md when scope is plan

**Checkpoint**: `speckit.gap-audit.audit plan` works end-to-end. Both scopes produce correctly scoped findings.

---

## Phase 5: User Story 3 - False Positive Filters Remove Noise (Priority: P2)

**Goal**: Embed 8 false positive filters in the subagent prompt so the subagent self-filters before returning findings.

**Independent Test**: Invoke the audit on a spec that produces findings including known false positives (claim about nonexistent API, gap already covered by a task). Confirm the subagent filters them out.

### Implementation for User Story 3

- [ ] T010 [US3] Add false positive filters 1-4 to the subagent prompt in the command file (both spec and plan scope prompts). Each filter is an imperative instruction: (1) verify technical claims against actual codebase via grep/read, reject if wrong, (2) cross-reference tasks.md before claiming gaps, reject if task exists (skip in spec scope), (3) distinguish FRs from acceptance scenarios, reframe orphan ACs correctly, (4) split or reject grouped findings with different root causes (FR-022)
- [ ] T011 [US3] Add false positive filters 5-8 to the subagent prompt: (5) classify pre-existing behavior correctly, (6) accept human-verified criteria with observable outcomes, (7) acknowledge existing coverage before claiming gaps, (8) default to plan-diverges-from-spec framing
- [ ] T012 [US3] Add unverified finding handling to the subagent prompt: when a filter cannot determine validity, pass finding through with evidence prefixed "unverified: [reason]" (FR-023)

**Checkpoint**: Audit produces higher-precision findings. Known false positives are filtered by the subagent.

---

## Phase 6: User Story 4 - Installable as Speckit Extension (Priority: P2)

**Goal**: Extension installs via `specify extension add`, command is discoverable, version check works.

**Independent Test**: Run `specify extension add` in a fresh speckit project and confirm the gap audit command is available.

### Implementation for User Story 4

- [ ] T013 [P] [US4] Add speckit version check to command file per section order in contracts/command-schema.md: first, determine how speckit exposes its version (check for `specify --version`, `.specify/init-options.json` speckit_version field, or environment variable). Then implement the check at invocation, returning a clear error if below 0.5.2 (FR-025)
- [ ] T014 [P] [US4] Create `.specify/extensions/gap-audit/README.md` with installation instructions, usage examples (both scopes, --output flag), gap-patterns.md format description
- [ ] T015 [US4] Finalize `.specify/extensions/gap-audit/extension.yml`: verify version 0.1.0, command registration points to correct file, no hooks section present, no `requires.extensions` section, speckit version requirement set

**Checkpoint**: Extension installs cleanly, command is listed, version check rejects old speckit.

---

## Phase 7: User Story 5 - Persist Findings and Match Patterns (Priority: P3)

**Goal**: When --output is set, write findings to JSON file. When gap-patterns.md exists, include patterns in the subagent prompt so the subagent tags matching findings.

**Independent Test**: Run with and without --output, verify JSON file presence. Provide gap-patterns.md, verify pattern_match appears on matching findings.

### Implementation for User Story 5

- [ ] T016 [US5] Add JSON persistence logic to command file per section order in contracts/command-schema.md: when --output flag is set, write findings array to `<spec_dir>/.sdd-findings-spec.json` (spec scope) or `<spec_dir>/.sdd-findings-plan.json` (plan scope). Overwrite if exists (FR-014). Write empty array when no findings survive filtering. After writing, validate JSON is parseable (SC-006).
- [ ] T017 [US5] Add gap-patterns.md loading to command file per section order in contracts/command-schema.md: check `specs/gap-patterns.md` (project-level only, not per-feature). If file exists and contains valid patterns (each with name, category, trigger fields), include them in the subagent prompt as additional checklist items. If file is empty or malformed (missing required fields, unparseable structure), log a warning and proceed without patterns.
- [ ] T018 [US5] Add pattern matching instructions to the subagent prompt: when gap-patterns.md patterns are loaded, instruct the subagent to check each finding against loaded patterns. If a finding's category matches a pattern's category and the description aligns with the pattern's trigger, add `pattern_match` field with the canonical pattern name.
- [ ] T019 [P] [US5] Create example `specs/gap-patterns.md` with 2-3 sample patterns demonstrating the format: canonical name, category, trigger description

**Checkpoint**: --output produces valid JSON. Pattern matching tags findings correctly. No JSON written without --output.

---

## Phase 8: Polish and Cross-Cutting Concerns

**Purpose**: Final validation and documentation.

- [ ] T020 Run quickstart.md validation: walk through each quickstart scenario and verify it works. Include: attempt `specify extension add --dev` to verify SC-004.
- [ ] T021 Dogfood: run `/speckit.gap-audit.audit spec` on this feature's own spec.md and verify findings quality
- [ ] T022 Create a test spec at `specs/001-gap-audit-extension/test-fixtures/spec-with-gaps.md` with at least 5 intentionally planted gaps (one per category: orphan FR, weak AC, unverifiable SC, cross-reference gap, implicit assumption). Run the auditor against it and verify it catches them (SC-001) without excessive false positives (SC-002).
- [ ] T023 Run the auditor on the test spec and verify at least one false positive is filtered out (SC-007).

---

## Dependencies and Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies. Start immediately.
- **Foundational (Phase 2)**: Depends on Phase 1.
- **US1 (Phase 3)**: Depends on Phase 2. MVP.
- **US2 (Phase 4)**: Depends on Phase 3 (extends the command file US1 created).
- **US3 (Phase 5)**: Depends on Phase 3 (filters added to subagent prompt US1 created).
- **US4 (Phase 6)**: Depends on Phase 1 (extension.yml exists). Can run in parallel with US1-US3.
- **US5 (Phase 7)**: Depends on Phase 3 (adds to the command file).
- **Polish (Phase 8)**: Depends on all previous phases.

### User Story Dependencies

- **US1 (P1)**: Foundational only. No story dependencies. **MVP.**
- **US2 (P1)**: Depends on US1 (extends same command file with plan scope).
- **US3 (P2)**: Depends on US1 (filters added to US1's subagent prompt). Independent of US2.
- **US4 (P2)**: Independent of US1-US3 (packaging only). Can parallel with story work.
- **US5 (P3)**: Depends on US1 (adds features to command file). Independent of US2-US4.

### Parallel Opportunities

- T002 (extension.yml) runs in parallel with other setup.
- T013, T014 (US4) can run in parallel with each other and with US1-US3 work.
- T019 (example gap-patterns.md) can run in parallel with other US5 tasks.
- US3 and US4 can run in parallel (US3 modifies subagent prompt, US4 modifies extension.yml and README).

---

## Parallel Example: User Story 4

```
# These tasks target different files and can run together:
Task: T013 "Add version check to command file"
Task: T014 "Create README.md"
Task: T015 "Finalize extension.yml"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (T001-T002)
2. Complete Phase 2: Foundational (T003-T004)
3. Complete Phase 3: US1 Spec Audit (T005-T007)
4. **STOP and VALIDATE**: Run `speckit.gap-audit.audit spec` on a test spec with planted gaps
5. If findings quality is acceptable, the MVP is usable

### Incremental Delivery

1. Setup + Foundational: Extension skeleton exists
2. US1: Spec audit works (MVP)
3. US2: Plan audit works (both scopes complete)
4. US3: False positive filters improve precision
5. US4: Extension is installable and distributable
6. US5: JSON persistence and pattern matching available
7. Each increment adds value without breaking previous functionality

---

## Notes

- All tasks modify markdown or YAML files, not compiled code
- The command file (`.specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md`) is the primary deliverable and is touched by US1, US2, US3, US4 (version check), and US5
- Extension name is `gap-audit`
- Total: 23 tasks across 8 phases
