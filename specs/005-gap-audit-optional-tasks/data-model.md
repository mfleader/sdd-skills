# Data Model: Gap Audit Optional tasks.md

## Changes to Existing Entities

### GapFinding (modified)

One new field added to the existing GapFinding entity:

| Field | Type | Required | Description | Change |
|-------|------|----------|-------------|--------|
| categories_audited | string[] | yes (on persist) | Checklist categories active in the audit run, in checklist order | **NEW** |

All existing fields (`classification`, `category`, `description`, `evidence`, `suggested_fix`, `source`, `scope`, `pattern_match`) are unchanged.

**Validation rules (updated)**:
- `category` MUST be one of the active checklist categories for the audit run:
  - Spec scope: 7 categories (unchanged)
  - Plan scope with tasks.md: 5 categories (unchanged)
  - Plan scope without tasks.md: 2 categories ("Implicit behavior", "Spec coverage gaps")
- `categories_audited` values MUST match the active categories exactly, in checklist order
- `categories_audited` is only present on persisted findings (when `--output` is set), same as `source` and `scope`

### Audit Scope (modified)

The `plan` scope definition changes from "spec + plan + tasks artifacts" to "spec + plan artifacts (required), plus tasks artifacts when available."

Plan scope now has two modes:
- **Full mode** (tasks.md present): 5 categories, all filters active
- **Degraded mode** (tasks.md absent): 2 categories, filter 2 skipped

### Plan Checklist Categories (new entity variants)

| Mode | Categories | Filter 2 | Pattern filtering |
|------|-----------|----------|-------------------|
| Full (tasks.md present) | Contract tests, Integration gaps, Edge case coverage, Implicit behavior, Spec coverage gaps | Active | All patterns included |
| Degraded (tasks.md absent) | Implicit behavior, Spec coverage gaps | Skipped | Only matching patterns |

## Relationships (unchanged)

```text
GapFinding --[matched by]--> Gap Pattern (via pattern_match field)
GapFinding --[categorized by]--> Active Checklist Categories (mode-dependent)
GapFinding --[filtered by]--> False Positive Filter (8 filters, filter 2 conditionally skipped)
```

## State Transitions (unchanged)

GapFinding has no state transitions. It is created once by the subagent, filtered, validated against active categories (FR-017), and presented. Findings are immutable after filtering.
