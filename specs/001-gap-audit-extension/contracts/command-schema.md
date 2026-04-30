# Command Contract: Gap Audit

## Command File Section Order

The command markdown file MUST follow this canonical section order. Tasks that add sections reference this ordering.

1. Version check (FR-025, T013)
2. Argument parsing ($ARGUMENTS, --output flag, scope) (T004)
3. Spec directory resolution (FR-024, T004)
4. Artifact loading and existence checks (FR-013, T004/T009)
5. Gap-patterns loading (FR-010, T017)
6. Subagent prompt construction with checklist + false positive filters + pattern matching (T005/T008/T010-T012/T018)
7. Subagent dispatch (T006)
8. Response parsing with defensive handling (T007)
9. JSON persistence when --output set (T016)
10. Output formatting and presentation (T007)

## Invocation

```
/speckit.gap-audit.audit <scope> [--output]
```

### Arguments

| Argument | Position | Required | Values | Description |
|----------|----------|----------|--------|-------------|
| scope | 1 | yes | `spec`, `plan` | Audit scope selector |
| --output | flag | no | (presence) | When set, write findings to JSON file |

### Argument Parsing

Arguments arrive via `$ARGUMENTS` in the command markdown. Parsing order:

1. Extract `--output` flag if present (consume it)
2. Remaining text is the scope argument
3. If scope is empty, ask the user
4. If scope is not `spec` or `plan`, return error listing valid values (FR-019)

## Input Artifacts

### Spec scope

| File | Required | Source |
|------|----------|--------|
| `<spec_dir>/spec.md` | yes | Feature specification |
| `specs/gap-patterns.md` | no | Project-level recurring patterns |

### Plan scope

| File | Required | Source |
|------|----------|--------|
| `<spec_dir>/spec.md` | yes | Feature specification |
| `<spec_dir>/plan.md` | yes | Implementation plan |
| `<spec_dir>/tasks.md` | yes | Task breakdown |
| `specs/gap-patterns.md` | no | Project-level recurring patterns |

## Output

### Default (no --output flag)

Grouped text presentation to the user:

```
## Blocking

### [Category]
- **Description**: ...
- **Evidence**: ...
- **Suggested fix**: ...

## Non-blocking

### [Category]
- **Description**: ...
- **Evidence**: ...
- **Suggested fix**: ...

---
Summary: N blocking, M non-blocking findings.
```

When no findings survive filtering: `No issues found.`

### With --output flag

Same presentation as above, plus a JSON file written to the spec directory:

- Spec scope: `<spec_dir>/.gap-audit-spec-findings.json`
- Plan scope: `<spec_dir>/.gap-audit-plan-findings.json`

JSON schema:

```json
[
  {
    "classification": "blocking",
    "category": "Orphan FRs",
    "description": "US1 describes X but no FR requires it",
    "evidence": "US1 AC2: '...' — no corresponding FR",
    "suggested_fix": "Add FR: System MUST X",
    "pattern_match": "missing-derived-fr"
  }
]
```

Empty array `[]` when no findings survive filtering.

## Error Conditions

| Condition | Error message |
|-----------|--------------|
| Missing scope argument | "Audit scope required. Valid values: spec, plan" |
| Invalid scope | "Invalid audit scope '[value]'. Valid values: spec, plan" |
| spec.md not found | "Required file not found: <spec_dir>/spec.md" |
| plan.md not found (plan scope) | "Required file not found: <spec_dir>/plan.md" |
| tasks.md not found (plan scope) | "Required file not found: <spec_dir>/tasks.md" |
| speckit version < 0.5.2 | "This extension requires speckit >= 0.5.2. Installed: [version]" |
| Spec directory not resolved | "Could not determine spec directory. Provide path as argument or set feature.json" |

## Subagent Contract

The command dispatches one subagent via the Agent tool:

| Parameter | Value |
|-----------|-------|
| subagent_type | `superpowers:code-reviewer` |
| description | `Adversarial gap audit (<scope>)` |

The subagent prompt includes:
- Full artifact content (spec.md, and optionally plan.md + tasks.md)
- The complete checklist for the selected scope (7 or 5 items)
- Classification scheme: blocking vs non-blocking
- Output format: JSON array of GapFinding objects
- Directive: DO NOT modify any files
- Gap patterns from gap-patterns.md (if loaded)
