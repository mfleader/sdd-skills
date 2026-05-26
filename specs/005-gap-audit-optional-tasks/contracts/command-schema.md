# Command Contract Changes: Gap Audit

## Input Artifacts (Plan scope, updated)

| File | Required | Source | Change |
|------|----------|--------|--------|
| `<spec_dir>/spec.md` | yes | Feature specification | unchanged |
| `<spec_dir>/plan.md` | yes | Implementation plan | unchanged |
| `<spec_dir>/tasks.md` | no (degraded mode: categories 4-5 only) | Task breakdown | **CHANGED from required to optional** |
| `specs/gap-patterns.md` | no | Project-level recurring patterns | unchanged |

## Error Conditions (updated)

| Condition | Error message | Change |
|-----------|--------------|--------|
| tasks.md not found (plan scope) | ~~"Required file not found: \<spec_dir>/tasks.md"~~ → Info message: "tasks.md not found, running plan-vs-spec audit only (categories 4-5)." | **CHANGED from error to info** |

All other error conditions unchanged.

## Output (updated)

### Summary line (plan scope)

| Condition | Summary format | Change |
|-----------|---------------|--------|
| tasks.md present, findings exist | "N blocking, M non-blocking findings." | unchanged |
| tasks.md absent, findings exist | "N blocking, M non-blocking findings (categories 4-5 only, tasks.md not available)." | **NEW** |
| tasks.md absent, no findings | "No issues found." | **NEW** |

### JSON persistence (--output)

New field on each finding:

```json
{
  "categories_audited": ["Implicit behavior", "Spec coverage gaps"],
  "source": "audit",
  "scope": "plan",
  ...existing fields...
}
```

## Extension Version

| Field | Before | After |
|-------|--------|-------|
| version | 0.1.0 | 0.2.0 |
