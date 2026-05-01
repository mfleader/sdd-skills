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

### Audit plan and tasks for gaps

```
/speckit.gap-audit.audit plan
```

Reads `spec.md`, `plan.md`, and `tasks.md` and checks for: missing contract tests, integration gaps, edge case coverage gaps, implicit behavior, and spec coverage gaps.

### Save findings to JSON

```
/speckit.gap-audit.audit spec --output
/speckit.gap-audit.audit plan --output
```

Writes findings to `.gap-audit-spec-findings.json` or `.gap-audit-plan-findings.json` in the spec directory. Each finding includes `source` and `scope` fields. Overwrites any existing file from a previous run.

## Defect Catalog

**Breaking change (v1.0.0):** The extension now reads `specs/defect-catalog.md` instead of `specs/gap-patterns.md`. Projects using the old file must migrate to the new format.

The extension reads the "Gap Audit Patterns" section from the project-level `specs/defect-catalog.md`. When present, the auditor tags matching findings with the canonical pattern name.

The defect catalog is externally produced (each project brings its own collector tooling). The catalog uses TOML frontmatter (`+++` delimited) for metadata, with consumer-specific sections under H2 headings.

Format:

```markdown
+++
version = 1
last_updated = "2026-05-01"
specs_analyzed = ["042", "043", "044"]
pattern_count = 3
+++

# Defect Catalog

## Gap Audit Patterns

(Consumed by this extension)

### missing-condition-false-path

**Category**: Weak ACs
**Trigger**: For every conditional FR (if X then Y), check that an AC exists for the condition-false path.
**Remediation**: Test both boundaries explicitly.

## Exploratory Testing Probes

(Consumed by the exploratory-test extension)

### off-by-one-boundary

**Category**: Edge case coverage
**Trigger**: Check boundary conditions for fencepost errors in loops and range checks.
**Remediation**: Verify loop bounds and range checks handle +/- 1 correctly.
```

Entry fields: `canonical-name` (H3 heading, required), `Category` (required), `Trigger` (required), `Occurrences`, `Source`, `Also known as`, `Remediation`, `Example` (all optional).

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
