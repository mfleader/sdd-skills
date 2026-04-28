# Command Schema: speckit.backtrace.trace

## Invocation

```
/speckit-backtrace-trace <scope> [spec-directory]
```

### Arguments

| Position | Name | Required | Values | Description |
|----------|------|----------|--------|-------------|
| 1 | scope | yes | `spec`, `plan` | Which artifacts to trace against |
| 2 | spec-directory | no | path | Override spec directory resolution |

### Findings Resolution

If no spec directory argument is provided, resolve via:
1. `.specify/feature.json` → `feature_directory`
2. Ask user

Within the resolved spec directory, findings are loaded from:
1. `.sdd-findings-{scope}.json` (if exists)
2. Ask user for file path (if step 1 fails)

### Errors

| Condition | Message |
|-----------|---------|
| speckit not initialized | `speckit not initialized. Run specify init first.` |
| speckit version < 0.5.2 | `This extension requires speckit >= 0.5.2. Installed: [version]` |
| `--output` flag provided | `The --output flag is not supported by backtrace. Use gap-audit with --output to persist findings, then run backtrace against the persisted file.` |
| Missing scope | Interactive prompt: `Scope required. Valid values: spec, plan` |
| Invalid scope | `Invalid scope '[value]'. Valid values: spec, plan` |
| Spec directory outside project root | `Spec directory path resolves outside the project root: <resolved_path>` |
| Spec directory not found | `Required file not found: <path>` |
| spec.md missing | `Required file not found: <spec_dir>/spec.md` |
| plan.md missing (plan scope) | `Required file not found: <spec_dir>/plan.md` |
| tasks.md missing (plan scope) | `Required file not found: <spec_dir>/tasks.md` |
| Findings file not found | `Findings file not found: <path>` |
| Empty findings | `No findings to trace` (clean exit) |
| Invalid findings format | Parse error with expected format description |
| Audit integrity violation | `Audit integrity violation: files were modified or created during review-only audit. Aborting backtrace.` |

## Output

Console output with two sections:

### Applied Additions

```
## Applied Additions (N)

1. [approve] FR-018: <addition summary>
   Target: spec.md → Functional Requirements
   Finding: <finding description>

2. [revise] AC on US-003: <addition summary>
   Target: spec.md → US-003 Acceptance Scenarios
   Finding: <finding description>
   Revision: <what the auditor changed>
```

### Rejected Additions

```
## Rejected Additions (N)

1. [reject] New SC: <addition summary>
   Target: spec.md → Success Criteria
   Finding: <finding description>
   Reason: <auditor's rejection reason>
```

### Reset Trigger Warning (if applicable)

```
## Reset Trigger Warning

The following additions may warrant re-planning:
- New user story added (US-006)
- SC count changed by 4 (from 8 to 12)

Consider running /speckit-plan to re-plan.
```

## Auditor Subagent Contract

### Dispatch

- Subagent type: `superpowers:code-reviewer`
- Single dispatch (all proposed additions in one prompt)
- Directive: "DO NOT modify any files"

### Auditor Input

```
<findings>
[Original GapFinding JSON array]
</findings>

<proposed_additions>
[ProposedAddition JSON array]
</proposed_additions>

<spec_artifacts>
[Full text of spec.md and optionally plan.md, tasks.md]
</spec_artifacts>
```

### Auditor Output (expected)

```json
[
  {
    "addition_ref": "1.1",
    "verdict": "approve",
    "reason": null,
    "revision": null
  },
  {
    "addition_ref": "1.2",
    "verdict": "revise",
    "reason": "Overlaps with existing FR-005",
    "revision": "Amend FR-005 instead of creating new FR"
  },
  {
    "addition_ref": "2.1",
    "verdict": "reject",
    "reason": "This AC is already covered by US-002 scenario 1"
  }
]
```
