# Data Model: SDD Gap Audit

## Entities

### GapFinding

A single gap identified by the adversarial auditor subagent, after false positive filtering.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| classification | string | yes | `"blocking"` or `"non-blocking"` |
| category | string | yes | Checklist item name (e.g., `"Orphan FRs"`, `"Contract tests"`) |
| description | string | yes | What the gap is |
| evidence | string | yes | File reference, section, or quote from the audited artifact. When a false positive filter cannot verify the finding, prefixed with `"unverified: [reason]"` (FR-023) |
| suggested_fix | string | yes | What to add or change to close the gap |
| source | string | yes (on persist) | Origin of the finding (e.g., `"audit"`). Added when `--output` persists findings to JSON. Not present in subagent output. |
| scope | string | yes (on persist) | Audit scope that produced the finding: `"spec"` or `"plan"`. Added when `--output` persists findings to JSON. |
| pattern_match | string | no | Canonical name from `gap-patterns.md` when the finding matches a known recurring pattern |

**Validation rules**:
- `classification` MUST be one of: `"blocking"`, `"non-blocking"`
- `category` MUST be one of the 12 checklist categories (7 spec + 5 plan, depending on audit scope)
- `evidence` MUST NOT be empty
- `pattern_match` is only present when `gap-patterns.md` exists and a match is found

### Spec Checklist Categories

The 7 categories used when audit scope is `spec`:

| Category | Description |
|----------|-------------|
| Orphan FRs | FRs implied by user stories but not explicitly stated |
| Weak ACs | Acceptance criteria that can pass while behavior is wrong |
| Unverifiable SCs | Success criteria that aren't mechanically verifiable |
| Cross-reference gaps | Orphan FRs with no SC, unmapped SCs with no FR |
| Implicit assumptions | Assumptions that should be explicit constraints |
| Naming collisions | New names conflicting with existing codebase terms |
| Implicit behavior | Load-bearing behavior the spec never mentions |

### Plan Checklist Categories

The 5 categories used when audit scope is `plan`:

| Category | Description |
|----------|-------------|
| Contract tests | Correct arguments at each API boundary |
| Integration gaps | Scenarios requiring real components with no covering task |
| Edge case coverage | Spec edge cases with corresponding tasks |
| Implicit behavior | Undocumented interactions between components |
| Spec coverage gaps | FRs/ACs/SCs implying architectural choices the plan doesn't account for |

### False Positive Filter

There are 8 named filters applied to raw subagent findings. Each filter is a validation check, not a data entity. They do not persist as data, they are logic in the command.

| Filter # | Name | What it checks |
|----------|------|----------------|
| 1 | Verify technical claims | Claims about APIs or codebase behavior are verified against actual code |
| 2 | Cross-reference tasks | "No task covers X" claims are checked against tasks.md |
| 3 | Distinguish FRs from ACs | Orphan ACs are framed correctly, not as missing FRs |
| 4 | Separate root causes | Grouped findings with different root causes are split or rejected |
| 5 | Classify pre-existing behavior | Existing documented behavior is not flagged as a new gap |
| 6 | Accept human-verified criteria | Non-automatable criteria with observable outcomes are not flagged as unverifiable |
| 7 | Acknowledge existing coverage | Findings acknowledge what's covered before stating what's missing |
| 8 | Default to plan-diverges framing | Spec/plan contradictions are framed as plan divergence, not spec error |

### Gap Pattern (in gap-patterns.md)

A recurring gap type cataloged at the project level.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | yes | Canonical name (e.g., `missing-condition-false-path`) |
| category | string | yes | Checklist category this pattern applies to (e.g., `Weak ACs`) |
| trigger | string | yes | Description of what to look for |

## Relationships

```text
GapFinding --[matched by]--> Gap Pattern (via pattern_match field)
GapFinding --[categorized by]--> Spec Checklist Category | Plan Checklist Category
GapFinding --[filtered by]--> False Positive Filter (8 filters applied sequentially)
```

## State Transitions

GapFinding has no state transitions. It is created once by the subagent, filtered, and presented. Findings are immutable after filtering.

The `--output` flag controls whether the findings array is persisted to JSON. This is a command-level decision, not an entity state.
