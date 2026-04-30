# Data Model: Backtrace Improvements

## Amended Entities

### ProposedAddition (v0.2.0)

One new field added: `trace_certainty`. All other fields unchanged from v0.1.0 (see specs/002-backtrace-extension/data-model.md for the full entity).

| Field | Type | Required | Description | Changed? |
|-------|------|----------|-------------|----------|
| addition_id | string | yes | Unique ID: finding index + sequential suffix (e.g., "1.1", "1.2") | no |
| finding_ref | string | yes | Reference to the GapFinding this addresses | no |
| target_artifact | string | yes | `spec.md` or `tasks.md` | no |
| target_section | string | yes | Section heading where addition goes | no |
| addition_type | string | yes | One of: `new_FR`, `amended_FR`, `new_AC`, `amended_AC`, `new_SC`, `new_task` | no |
| content | string | yes | The actual text to add or amend | no |
| rationale | string | yes | Why this addition addresses the gap | no |
| **trace_certainty** | **string** | **yes** | **One of: `direct`, `plausible`, `uncertain`** | **added in v0.2.0** |

### trace_certainty values

| Value | Definition | Example |
|-------|-----------|---------|
| `direct` | Finding explicitly names or unambiguously refers to a specific spec item | "FR-003 does not cover negative inputs" |
| `plausible` | Finding relates to a spec item's domain but does not name it directly | "Input validation is missing" traces to an FR about user input handling |
| `uncertain` | Finding could trace to multiple spec items, or the connection is inferential | "Error handling is inconsistent" could trace to FR-003, FR-007, or FR-009 |

## Unchanged Entities

- **GapFinding**: No changes (input entity from gap-audit)
- **AuditorVerdict**: No changes (output from auditor subagent)

## Schema Compatibility

This is an additive, non-breaking change. Existing consumers of ProposedAddition that do not read `trace_certainty` will continue to function. The auditor subagent receives `trace_certainty` as additional context but its AuditorVerdict schema is unchanged.
