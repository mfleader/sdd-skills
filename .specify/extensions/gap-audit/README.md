# SDD Gap Audit Extension

Adversarial gap auditor for SDD artifacts. Dispatches a prejudiced subagent to find gaps the standard review step missed.

## Requirements

- speckit >= 0.5.2
- Claude Code with `superpowers:code-reviewer` subagent type

## Installation

From a git URL:

```bash
specify extension add gap-audit --from <git-url>
```

For local development:

```bash
specify extension add gap-audit --dev
```

## Usage

### Audit a spec for gaps

```
/speckit.gap-audit.audit spec
```

Reads `spec.md` and checks for: orphan FRs, weak ACs, unverifiable SCs, cross-reference gaps, implicit assumptions, naming collisions, and implicit behavior.

### Audit plan for gaps

```
/speckit.gap-audit.audit plan
```

Reads `spec.md` and `plan.md` (required), plus `tasks.md` if present. When `tasks.md` is available, checks all 5 categories: missing contract tests, integration gaps, edge case coverage gaps, implicit behavior, and spec coverage gaps. When `tasks.md` is absent, checks categories 4-5 only (implicit behavior, spec coverage gaps).

**Note**: The backtrace extension's plan scope still requires `tasks.md`. If you run gap-audit plan without `tasks.md` and get findings, use `backtrace spec` (not `backtrace plan`) to trace them, or generate `tasks.md` first.

### Save findings to JSON

```
/speckit.gap-audit.audit spec --output
/speckit.gap-audit.audit plan --output
```

Writes findings to `.gap-audit-spec-findings.json` or `.gap-audit-plan-findings.json` in the spec directory. Each finding includes `source`, `scope`, and `categories_audited` fields. Overwrites any existing file from a previous run.

## Gap Patterns

Create a project-level `specs/gap-patterns.md` file to track recurring gap types across audits. When present, the auditor tags matching findings with the canonical pattern name.

Format:

```markdown
# Gap Audit Patterns

## missing-condition-false-path
**Category**: Weak ACs
**Trigger**: For every conditional FR (if X then Y), check that an AC exists for the condition-false path.

## missing-derived-fr
**Category**: Orphan FRs
**Trigger**: User story describes behavior but no FR explicitly requires it.
```

## Output

Findings are grouped by classification:

- **Blocking**: Gaps that would cause incorrect or incomplete implementation
- **Non-blocking**: Quality issues or minor omissions

Each finding includes: category, description, evidence (with artifact reference), and suggested fix.

## Notes

- This extension registers no lifecycle hooks. It is invoke-only.
- The auditor completes in a single subagent dispatch (no multi-round iteration).
- 8 false positive filters are applied by the subagent before returning findings.
- This extension does not depend on spex-gates or any other extension.
