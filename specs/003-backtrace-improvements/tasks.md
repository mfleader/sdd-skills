# Tasks: Backtrace Improvements

**Input**: Design documents from `specs/003-backtrace-improvements/`
**Prerequisites**: plan.md, spec.md, data-model.md

**Tests**: Not applicable. This is a markdown skill, not code. Verification is through manual invocation.

**Organization**: Tasks grouped by user story. US-001 and US-002 modify Section 6 together (barrier walk informs certainty), so US-002 depends on US-001.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Phase 1: Setup

**Purpose**: Version bump and data model update (prerequisites for all user story work)

- [x] T001 [P] Bump version from 0.1.0 to 0.2.0 in `.specify/extensions/backtrace/extension.yml`
- [x] T002 [P] Add `trace_certainty` field to ProposedAddition entity in `specs/002-backtrace-extension/data-model.md` per `specs/003-backtrace-improvements/data-model.md`

**Checkpoint**: Schema updated, version bumped. Command file modifications can begin.

---

## Phase 2: User Story 1 - Barrier Enumeration (Priority: P1)

**Goal**: Walk the spec hierarchy top-down (FR → AC → SC → task) for each finding before applying tracing categories. The walk determines the target layer; the categories determine new vs amended within that layer.

**Independent Test**: Run backtrace with a finding that describes behavior with no relevant FR. Verify it proposes a new FR, not an AC on an unrelated FR.

### Implementation for User Story 1

- [x] T003 [US1] Add barrier walk instructions to Section 6 of `.specify/extensions/backtrace/commands/speckit.backtrace.trace.md`. Insert before the four tracing categories. The walk checks: (1) relevant FR exists? (2) adequate ACs? (3) corresponding SC? (4) task (plan scope only)? Identify the first missing layer. When scope is spec, the walk terminates at step 3 (SC check); step 4 is skipped because tasks.md is not loaded. Relevance is semantic association per FR-018.
- [x] T004 [US1] Update the tracing category logic in Section 6 of `.specify/extensions/backtrace/commands/speckit.backtrace.trace.md` to use the barrier walk result. The barrier walk result takes precedence over category-specific default target layers. If the walk determines the gap is at the FR level, all four tracing categories propose at the FR layer (new_FR or amended_FR), not at their default layer (e.g., category 2 defaults to SC but must yield to the walk result). The walk determines the target layer per FR-019; the categories determine new vs amended within that layer.

**Checkpoint**: Barrier enumeration functional. Additions target the correct spec layer.

---

## Phase 3: User Story 2 - Trace Certainty (Priority: P1)

**Goal**: Every ProposedAddition includes a `trace_certainty` field (`direct`, `plausible`, `uncertain`). The auditor applies more skepticism to uncertain traces.

**Independent Test**: Run backtrace with a mix of findings. Verify that findings naming specific FRs get `direct` and cross-cutting findings get `uncertain`.

**Depends on**: Phase 2 (barrier walk informs certainty assignment)

### Implementation for User Story 2

- [x] T005 [US2] Add trace_certainty assignment to Section 6 of `.specify/extensions/backtrace/commands/speckit.backtrace.trace.md`. After each trace, assign certainty per-addition (not per-finding; a single finding may produce additions with different certainty values). Use the barrier walk result as the primary input: if the walk found a spec item whose scope explicitly covers the finding's domain, certainty is `direct`; if the association is domain-related but not explicit, certainty is `plausible`; if the walk fell through to inferential matching across multiple candidates, certainty is `uncertain`. Per FR-021, FR-022.
- [x] T006 [US2] Add trace_certainty to the ProposedAddition field table in Section 6 of `.specify/extensions/backtrace/commands/speckit.backtrace.trace.md` (schema definition) and ensure the field appears in the actual JSON output that Section 6 produces (the ProposedAddition array sent to the auditor in Section 7).
- [x] T007 [US2] Update the auditor prompt in Section 7 of `.specify/extensions/backtrace/commands/speckit.backtrace.trace.md`. Add instruction: "For additions with trace_certainty: uncertain, verify the rationale more carefully. Prefer reject or revise when the causal link between the finding and the proposed spec item is weak." Per FR-023.
- [x] T008 [US2] Update the output format in Section 11 of `.specify/extensions/backtrace/commands/speckit.backtrace.trace.md`. Add `[certainty]` in brackets after each addition summary line. Format: `1. [approve] FR-018: <summary> [direct]`. Apply to both Applied Additions and Rejected Additions sections. Per FR-024.

**Checkpoint**: Every ProposedAddition has trace_certainty. Auditor weighs uncertain traces with more skepticism. Output displays certainty.

---

## Phase 4: User Story 3 - Cluster Detection (Priority: P2)

**Goal**: Output notes when 3+ applied additions target the same `target_section` within the same `target_artifact`.

**Independent Test**: Run backtrace with findings that produce 4+ additions targeting the same FR. Verify the output mentions the cluster.

**Depends on**: Phase 3 (T008 also modifies Section 11; T009 must be applied after T008 to avoid conflicts in the Applied Additions subsection).

### Implementation for User Story 3

- [x] T009 [US3] Add cluster detection logic to Section 11 of `.specify/extensions/backtrace/commands/speckit.backtrace.trace.md`. After formatting the Applied Additions list, count additions per `target_section` within `target_artifact`. If 3+ target the same item, append a cluster warning after the Applied Additions section per FR-025, FR-026. Only count applied additions (not rejected). Threshold is fixed at 3 per FR-027.

**Checkpoint**: Cluster warnings appear when 3+ additions target the same spec item.

---

## Phase 5: Polish

**Purpose**: Final verification and cleanup

- [x] T010 Regenerate the Claude Code skill wrapper at `.claude/skills/speckit-backtrace-trace/SKILL.md` from the updated command file. The SKILL.md inlines the command file content and is already out of sync (pre-existing drift from source-named findings convention change). Full regeneration required, not incremental patching.

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: No dependencies. Start immediately.
- **Phase 2 (US1 - Barrier Enumeration)**: Depends on Phase 1 (data model must reflect trace_certainty before command file references it).
- **Phase 3 (US2 - Trace Certainty)**: Depends on Phase 2 (barrier walk informs certainty assignment).
- **Phase 4 (US3 - Cluster Detection)**: Depends on Phase 3 (T008 modifies Section 11; T009 must follow to avoid conflicts).
- **Phase 5 (Polish)**: Depends on all previous phases.

### User Story Dependencies

- **US1 (Barrier Enumeration)**: Independent after Setup
- **US2 (Trace Certainty)**: Depends on US1 (barrier walk informs certainty)
- **US3 (Cluster Detection)**: Depends on US2 (T008 modifies Section 11 before T009)

### Parallel Opportunities

- T001 and T002 (Setup) can run in parallel [P] (different files)
- T005-T008 (US2) must be sequential after T003-T004 (US1)
- T009 (US3) must follow T008 (both modify Section 11)

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (T001, T002)
2. Complete Phase 2: Barrier enumeration (T003, T004)
3. **STOP and VALIDATE**: Run backtrace with test findings, verify correct layer targeting

### Incremental Delivery

1. Setup → version and schema ready
2. US1 (barrier enumeration) → test layer targeting → most impactful improvement
3. US2 (trace certainty) → test certainty assignment → constitution compliance
4. US3 (cluster detection) → test cluster warnings → output quality
5. Polish → verify skill wrapper

---

## Notes

- All modifications are to markdown instruction files, not code
- The command file is ~420 lines; changes touch Sections 6, 7, and 11
- T001 and T002 are the only changes outside the command file
- Total estimated change: ~30 lines of new/modified instruction text
