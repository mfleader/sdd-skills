# SDD Exploratory Test Extension

Adversarial exploratory tester for SDD implementations. Dispatches a subagent to exercise a feature beyond its success criteria, finding edge cases and bugs the spec missed.

## Requirements

- speckit >= 0.5.2
- Claude Code with `superpowers:code-reviewer` subagent type

## Installation

From a git URL:

```bash
specify extension add exploratory-test --from <git-url>
```

For local development:

```bash
specify extension add exploratory-test --dev
```

## Usage

### Run exploratory tests

```
/speckit.exploratory-test.test <spec_dir>
```

Reads `spec.md`, discovers changed files via `git diff`, dispatches an adversarial subagent that writes and runs tests against the implementation, and reports findings.

### Specify a base branch

```
/speckit.exploratory-test.test <spec_dir> --base develop
```

Uses `develop` instead of `main` for changed file discovery. Useful for non-standard branch workflows.

### Save findings to JSON

```
/speckit.exploratory-test.test <spec_dir> --output
```

Writes findings to `.exploratory-findings.json` in the spec directory. Each finding includes a `"source": "exploratory"` field. Overwrites any existing file from a previous run. Writes `[]` when no findings exist.

## Output Format

Findings are grouped by severity:

- **Critical**: Crash or data loss
- **Important**: Wrong result
- **Moderate**: Unexpected but non-breaking
- **Minor**: Cosmetic or style

Each finding includes: description, reproduction steps, expected behavior, actual behavior, and spec gap reference.

### JSON Schema

When `--output` is passed, findings conform to the schema in `contracts/exploratory-findings.json`:

```json
{
  "severity": "critical|important|moderate|minor",
  "description": "what happened",
  "reproduction": "command or code snippet",
  "expected": "what should have happened",
  "actual": "what did happen",
  "spec_gap": "which FR/AC/SC should have caught this",
  "source": "exploratory",
  "pattern_match": "(optional) canonical name from defect catalog"
}
```

## Defect Catalog Integration

Create a `defect-catalog.md` file in the spec directory (adjacent to `spec.md`) with an "Exploratory Testing Probes" section to track recurring defect patterns. When present, the tester uses these as additional test vectors and tags matching findings with `"pattern_match"`.

Format:

```markdown
# Defect Catalog

## Exploratory Testing Probes

### missing-condition-false-path
For every conditional FR, check that the implementation handles the condition-false path.

### off-by-one-boundary
Check boundary conditions for fencepost errors in loops and range checks.
```

## Notes

- This extension registers no lifecycle hooks. It is invoke-only.
- The tester completes in a single subagent dispatch (no multi-round iteration).
- The subagent writes tests to `/tmp/exploratory-tests/` and runs them against the implementation.
- The subagent also runs the project test suite for regression checking.
- A post-dispatch integrity check warns if any repo files were modified or created during testing.
- This extension does not depend on spex-gates or any other extension.
