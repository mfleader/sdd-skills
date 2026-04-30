# Feature Specification: Backtrace Improvements

**Feature Branch**: `003-backtrace-improvements`
**Created**: 2026-04-30
**Status**: Draft
**Input**: Improve backtrace extension with barrier enumeration, trace certainty, and cluster detection (improvement-research.md items I-001 through I-003)

## Purpose

Improve backtrace's tracing accuracy and output quality with three targeted changes: a barrier enumeration step that prevents misplaced additions, categorical trace certainty that flags ambiguous traces, and cluster detection that surfaces systemic spec weaknesses. All changes are additive to the existing v0.1.0 design. Version bump to 0.2.0.

## User Scenarios & Testing

### US-001: Barrier enumeration prevents misplaced additions (P1)

**As** a developer running backtrace,
**I want** each finding to be traced top-down through the spec hierarchy (FR → AC → SC → task),
**so that** additions target the correct spec layer rather than proposing ACs when the real gap is a missing FR.

**Why this priority**: Tracing to the wrong spec layer produces technically incorrect additions even when the content is reasonable. This directly violates Constitution Principle I (correctness over coverage).

**Independent Test**: Run backtrace with a finding that describes behavior with no relevant FR. Verify it proposes a new FR, not an AC on an unrelated FR.

**Acceptance Scenarios:**

1. **Given** a finding about unvalidated input and no FR's stated scope covers input validation, **When** backtrace traces the finding, **Then** it proposes a new FR (not an AC on an unrelated FR).
2. **Given** a finding about an edge case and an FR whose scope covers that domain exists but lacks an AC for that boundary, **When** backtrace traces the finding, **Then** it proposes a new AC on that FR.
3. **Given** a finding about a missing integration check and a relevant FR and AC both exist, **When** backtrace traces the finding, **Then** it proposes a new SC (not a duplicate AC).
4. **Given** a finding about implemented behavior and a relevant FR, AC, and SC all exist but no task covers the implementation (plan scope), **When** backtrace traces the finding, **Then** it proposes a new_task (not a duplicate SC).

---

### US-002: Trace certainty surfaces ambiguity (P1)

**As** a developer reviewing backtrace output,
**I want** each proposed addition to indicate how confident the trace is,
**so that** I can focus scrutiny on uncertain traces.

**Why this priority**: Directly implements Constitution Principle VII (Flag Uncertainty). Without certainty markers, all proposed additions appear equally confident regardless of whether the trace is obvious or inferential.

**Independent Test**: Run backtrace with a mix of findings, some naming specific FRs and others describing vague cross-cutting concerns. Verify direct traces get `direct` and ambiguous traces get `uncertain`.

**Acceptance Scenarios:**

1. **Given** a finding that explicitly names a spec item (e.g., "FR-003 does not cover negative inputs"), **When** backtrace traces it, **Then** the ProposedAddition has `trace_certainty: direct`.
2. **Given** a finding about a cross-cutting concern that could trace to multiple spec items, **When** backtrace traces it, **Then** the ProposedAddition has `trace_certainty: uncertain`.
3. **Given** a finding that relates to a spec item's domain but does not name it, **When** backtrace traces it, **Then** the ProposedAddition has `trace_certainty: plausible`.
4. **Given** proposed additions sent to the auditor, **When** the auditor prompt is constructed, **Then** it includes explicit instructions to verify rationale more carefully for `uncertain` traces and prefer `reject` or `revise` when the causal link is weak.

---

### US-003: Cluster detection surfaces systemic issues (P2)

**As** a developer reviewing backtrace output,
**I want** to be told when multiple additions target the same spec item,
**so that** I can consider structural rework instead of incremental amendments.

**Why this priority**: Useful but not critical. The developer can already see clustering by reading the output list. This automates pattern recognition that is otherwise manual.

**Independent Test**: Run backtrace with findings that produce 4+ additions targeting the same FR. Verify the output summary mentions the cluster.

**Acceptance Scenarios:**

1. **Given** 3 or more applied additions all target the same `target_section` within the same `target_artifact`, **When** backtrace formats output, **Then** it notes the cluster and suggests considering structural rework.
2. **Given** fewer than 3 additions target any single `target_section` within the same `target_artifact`, **When** backtrace formats output, **Then** no cluster warning is emitted.

---

### Edge Cases

- Barrier enumeration finds no relevant FR, AC, SC, or task: treat as an untraceable finding (existing behavior: propose a new FR or AC).
- All additions in a cluster are rejected by the auditor: no cluster warning is emitted (only applied additions count).
- A finding produces multiple ProposedAdditions at different certainty levels (e.g., one `direct` AC amendment and one `uncertain` new SC): each addition gets its own certainty independently.
- `trace_certainty` and barrier enumeration interact: the barrier walk naturally informs certainty. If the walk finds a spec item whose scope explicitly covers the finding's domain, certainty is `direct`. If the association is domain-related but not explicit, certainty is `plausible`. If the walk falls through to inferential matching across multiple candidates, certainty is `uncertain`.
- When the barrier walk finds multiple FRs whose stated scope covers the finding's domain, the finding traces to each relevant FR independently (producing separate ProposedAdditions). The trace_certainty for each is assessed independently per FR-022. If the finding's domain does not clearly favor one FR over another, this ambiguity is reflected in a lower trace_certainty value per existing Edge Case bullet 4.
- When multiple cluster warnings exist, present them in descending order by addition count (largest cluster first).

## Requirements

### Functional Requirements

**Barrier enumeration (Section 6 enhancement):**

- **FR-018**: Before applying the four tracing categories, walk the spec hierarchy top-down for each finding: (1) Is there a relevant FR? (2) If yes, does it have adequate ACs for this behavior? (3) If yes, is there a corresponding SC? (4) If yes (plan scope), is there a task? Identify the first missing layer. Relevance is determined by semantic association (the spec item's stated scope covers the behavior or domain described by the finding), not keyword matching. This is delegated to LLM judgment.
- **FR-019**: The barrier walk determines the target spec layer. The existing tracing categories then determine whether the addition at that layer is new or amended: if an existing item at the target layer is close but incomplete, propose an amendment (`amended_AC`, `amended_FR`); if no item at that layer covers the finding's domain, propose a new item (`new_FR`, `new_AC`, `new_SC`, `new_task`).
- **FR-020**: The barrier walk and tracing categories operate together in a single pass: the walk determines the target layer, and the categories determine the addition content and type within that layer.

**Trace certainty (ProposedAddition enhancement):**

- **FR-021**: Add a `trace_certainty` field to ProposedAddition with three valid values: `direct`, `plausible`, `uncertain`.
- **FR-022**: `direct`: the finding explicitly names or unambiguously refers to a specific spec item. `plausible`: the finding relates to a spec item's domain but does not name it directly. `uncertain`: the finding could trace to multiple spec items, or the connection is inferential.
- **FR-023**: The auditor prompt (Section 7) must instruct the auditor to apply more skepticism to `uncertain` traces: verify the rationale more carefully and prefer `reject` or `revise` when the causal link is weak.
- **FR-024**: The output (Section 11) must include the trace_certainty value for each applied and rejected addition, displayed in brackets after the addition summary. Format: `1. [approve] FR-018: <summary> [direct]`

**Cluster detection (Section 11 enhancement):**

- **FR-025**: After formatting the applied additions list, count how many applied additions target each spec item (by `target_section` within `target_artifact`).
- **FR-026**: If 3 or more applied additions target the same spec item, append a cluster warning after the Applied Additions section: "[spec item] is the target of N additions. Consider whether this item needs structural rework rather than incremental amendments."
- **FR-027**: The cluster threshold of 3 is fixed. No configuration mechanism.

**Version and schema:**

- **FR-028**: Bump extension version from 0.1.0 to 0.2.0 in extension.yml.
- **FR-029**: Update data-model.md to include `trace_certainty` in the ProposedAddition entity. This is an additive, non-breaking schema change.

### Key Entities

- **ProposedAddition** (amended): Gains one new field: `trace_certainty` (string, required, one of: `direct`, `plausible`, `uncertain`). All other fields unchanged from v0.1.0.

## Success Criteria

- **SC-009**: Each ProposedAddition targets the first missing layer in the spec hierarchy (FR → AC → SC → task). The barrier walk and tracing categories operate together per FR-020.
- **SC-010**: Every ProposedAddition includes a `trace_certainty` field with one of the three valid values (`direct`, `plausible`, `uncertain`).
- **SC-011**: The auditor prompt includes explicit instructions to weight uncertain traces with more skepticism.
- **SC-012**: Output summary notes clusters of 3+ additions targeting the same `target_section` within the same `target_artifact`.
- **SC-013**: Extension version is 0.2.0.
- **SC-014**: data-model.md reflects the `trace_certainty` field addition.

## Error Handling

No new error conditions are introduced. All three changes are additive to existing behavior:

- Barrier enumeration: if the walk finds no relevant spec item at any level, falls through to existing untraceable-finding behavior (propose new FR or AC).
- Trace certainty: always produces a value. No error state possible since the three categories cover all cases.
- Cluster detection: pure post-processing on applied additions. If no additions were applied, no cluster check runs.

## Dependencies

- Backtrace extension v0.1.0 (specs/002-backtrace-extension)
- No new external dependencies

## Assumptions

- The existing four tracing categories remain valid and are not replaced.
- The barrier enumeration step does not change the set of findings that can be traced, only the accuracy of which spec layer is targeted.
- The barrier walk uses semantic association, not explicit parent-child IDs, to determine the hierarchy between spec items. ACs are associated with FRs through document proximity and domain relevance. This association is delegated to LLM judgment.
- The cluster threshold of 3 is adequate for typical finding volumes (5-15 findings per audit).
- Trace certainty is assessed by the same LLM that performs tracing, not by the auditor subagent. The auditor uses certainty as an input signal, not as something it assigns.

## Out of Scope

- Replacing the four tracing categories with STPA-derived categories (explored in improvement-research.md, deferred)
- Toulmin-structured rationales (explored, deferred to v0.3+)
- Iterative refinement / relaxing FR-010 (explored, rejected)
- Dual specialized auditors (explored, rejected)
- Configurable cluster threshold
- Any changes to gap-audit

## Manual Verification Scenarios

Required manual test scenarios for verifying v0.2.0 behavior. Run backtrace with synthetic findings files against a test spec.

### MV-001: Untraceable finding (barrier walk finds nothing)

Run backtrace with a finding whose domain has no relationship to any FR, AC, SC, or task in the target spec. Verify: output is `new_FR` or `new_AC` with `trace_certainty: uncertain`.

### MV-002: Cluster threshold boundary (exactly 3)

Run backtrace with exactly 3 findings that all trace to the same spec item and are all approved by the auditor. Verify: cluster warning is emitted. Then run with exactly 2 targeting the same item. Verify: no cluster warning.

### MV-003: Scope-dependent walk termination (spec scope, all layers present)

Run backtrace with `scope=spec` and a finding where FR, AC, and SC all exist adequately. Verify: the barrier walk terminates at step 3 (does not attempt step 4), and either no addition is proposed or the finding is logged as "all spec layers present."

### MV-004: Plausible certainty as distinct outcome

Run backtrace with a finding that describes behavior in a domain clearly covered by a single FR but does not name that FR (e.g., finding about "timeout handling" when an FR covers "connection lifecycle management"). Verify: output is `trace_certainty: plausible`, not `direct` or `uncertain`.

### MV-005: Multiple-FR tie-breaking

Create a spec with two FRs whose scopes overlap (e.g., FR-A covers "user authentication" and FR-B covers "session management"). Run backtrace with a finding about "session tokens are not validated during authentication." Verify: two ProposedAdditions are produced, one per FR.

### MV-006: Cluster warning ordering (multiple clusters)

Run backtrace with 7+ findings producing two distinct clusters (e.g., 4 additions targeting FR-A and 3 targeting FR-B). Verify: both cluster warnings appear, FR-A's cluster listed first (descending by count).

### Known Limitations

- **Auditor skepticism for uncertain traces** (FR-023): The instruction's effect is non-deterministic and depends on LLM interpretation. Structural verification (instruction text exists in prompt) is the only repeatable check. Future improvement: track rejection rates for `uncertain` vs `direct` traces across runs.
