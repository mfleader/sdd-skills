# Quickstart: SDD Gap Audit Extension

## Install

```bash
specify extension add gap-audit --from <git-url>
```

Or for local development:

```bash
specify extension add gap-audit --dev
```

## Usage

### Audit a spec for gaps

```
/speckit.gap-audit.audit spec
```

Reads `spec.md` from the current feature's spec directory and checks for orphan FRs, weak ACs, unverifiable SCs, cross-reference gaps, implicit assumptions, naming collisions, and implicit behavior.

### Audit plan and tasks for gaps

```
/speckit.gap-audit.audit plan
```

Reads `spec.md`, `plan.md`, and `tasks.md` and checks for missing contract tests, integration gaps, edge case coverage gaps, implicit behavior, and spec coverage gaps.

### Save findings to JSON

```
/speckit.gap-audit.audit spec --output
/speckit.gap-audit.audit plan --output
```

Same audit, but also writes findings to `.sdd-findings-spec.json` or `.sdd-findings-plan.json` in the spec directory.

## Optional: Gap Patterns

Create a `specs/gap-patterns.md` file to track recurring gap types across audits. When present, the auditor tags matching findings with the pattern name.

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
