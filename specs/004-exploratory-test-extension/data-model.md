# Data Model: Exploratory Test Extension

## ExploratoryFinding

A single finding produced by the adversarial subagent.

### Fields

| Field | Type | Required | Description |
|---|---|---|---|
| severity | enum | yes | `critical` (crash/data loss), `important` (wrong result), `moderate` (unexpected but non-breaking), `minor` (cosmetic/style) |
| description | string | yes | What happened |
| reproduction | string | yes | Command or code snippet to reproduce |
| expected | string | yes | What should have happened |
| actual | string | yes | What did happen |
| spec_gap | string | yes | Which FR/AC/SC should have caught this (if any) |
| source | string | yes | Always `"exploratory"`. Injected by the extension (FR-010), not by the subagent |
| pattern_match | string | no | Canonical name from defect catalog. Present only when a finding matches a known pattern |

### Validation Rules

- `severity` must be one of: `critical`, `important`, `moderate`, `minor`
- `source` is always `"exploratory"` (set by extension, not by subagent)
- `pattern_match` is optional; omit the field entirely when no match (do not set to null)
- All required string fields must be non-empty

### Differences from GapFinding (gap-audit)

| Aspect | ExploratoryFinding | GapFinding |
|---|---|---|
| Severity/classification | `severity` (4 levels) | `classification` (blocking/non-blocking) |
| Evidence type | `reproduction` + `expected` + `actual` | `evidence` + `suggested_fix` |
| Category | None (no audit checklist categories) | `category` (7 spec or 5 plan categories) |
| Scope field | Not present (no scope concept) | `scope` (spec or plan) |
| Source value | `"exploratory"` | `"audit"` |

### JSON Output Format

When `--output` is passed, findings are written to `<spec_dir>/.exploratory-findings.json`:

```json
[
  {
    "severity": "important",
    "description": "Division by zero when input count is 0",
    "reproduction": "process_batch(items=[], batch_size=0)",
    "expected": "Raise ValueError or return empty result",
    "actual": "ZeroDivisionError traceback",
    "spec_gap": "FR-003 has no AC for empty input",
    "source": "exploratory",
    "pattern_match": "missing-condition-false-path"
  }
]
```

Empty findings: `[]`
