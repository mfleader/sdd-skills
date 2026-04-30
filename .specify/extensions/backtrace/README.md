# SDD Backtrace Extension

Traces gap-audit findings back to missing spec items, proposes additions with adversarial auditor approval, and applies approved changes. Closes the loop between finding gaps and fixing them.

## Requirements

- speckit >= 0.5.2
- Claude Code with `superpowers:code-reviewer` subagent type
- Gap-audit findings in GapFinding JSON format (from `gap-audit --output` or manual creation)

## Installation

From a git URL:

```bash
specify extension add backtrace --from <git-url>
```

For local development:

```bash
specify extension add backtrace --dev
```

## Usage

### Trace spec-scope findings

```
/speckit-backtrace-trace spec
```

Auto-detects findings files (e.g., `.gap-audit-spec-findings.json`) in the spec directory. Traces each finding back to spec.md, proposes FR/AC/SC additions, gets auditor approval, and applies approved changes.

### Trace plan-scope findings

```
/speckit-backtrace-trace plan
```

Auto-detects findings files (e.g., `.gap-audit-plan-findings.json`). Traces against spec.md, plan.md, and tasks.md. May propose new tasks in addition to spec additions.

### Explicit spec directory

```
/speckit-backtrace-trace spec specs/002-my-feature
```

Overrides the default spec directory resolution (feature.json or interactive prompt).

## Findings Input Format

Backtrace consumes the GapFinding JSON format produced by `gap-audit --output`:

```json
[
  {
    "classification": "blocking",
    "category": "Weak ACs",
    "description": "AC on US-001 can pass while behavior is wrong",
    "evidence": "spec.md, US-001 scenario 2: checks presence but not correctness",
    "suggested_fix": "Add AC verifying the output value matches expected format"
  }
]
```

If no `*-findings.json` file matching the current scope is found, backtrace prompts for a file path. Legacy filenames (`.sdd-findings-{scope}.json`) are also detected as a fallback.

## Output Format

### Applied additions

Each approved or revised addition is listed with its target artifact, section, and the finding it addresses.

### Rejected additions

Rejected proposals are listed with the auditor's reason for rejection.

### Reset trigger warnings

If applied additions cross reset trigger thresholds (new user story, new non-refinement FR, SC count change by 3+, material scope expansion), a warning is displayed. No action is taken. The user decides whether to re-plan.

## Follow-Up Reviews

After completing, backtrace invokes follow-up reviews:

1. `review-spec` (both scopes, requires spex-gates)
2. `review-plan` (plan scope only, requires spex-gates)
3. `analyze` (cross-artifact consistency, core speckit command, always runs)

If `spex-gates` is not installed, review-spec and review-plan are skipped with a note. Analyze runs regardless.

## Artifact Interactions

| File | Read | Write |
|------|------|-------|
| `.*-findings.json` (e.g., `.gap-audit-spec-findings.json`) | yes (input) | no |
| `spec.md` | yes | yes (additions) |
| `plan.md` | yes (plan scope) | no (read-only) |
| `tasks.md` | yes (plan scope) | yes (new tasks, plan scope) |
| `.specify/feature.json` | yes (dir resolution) | no |
| `.specify/extensions/.registry` | yes (follow-up check) | no |
| `.specify/memory/constitution.md` | yes (reset triggers) | no |
| `.specify/init-options.json` | yes (version check) | no |

## Notes

- This extension registers no lifecycle hooks. Follow-up reviews are invoked directly from the command file (same pattern as review-code invoking deep-review).
- Single subagent dispatch per invocation. No multi-round iteration.
- Plan.md is read-only. The backtrace never modifies plan.md.
- This extension does not depend on gap-audit being installed. Any source producing GapFinding JSON works.
- Soft dependency on spex-gates for follow-up reviews (review-spec, review-plan). Graceful degradation when absent. Analyze runs unconditionally.
- Spec directory and findings file paths are validated to be within the project root (path traversal protection).
- The auditor prompt includes an anti-injection directive marking data within XML tags as opaque input.
- Post-dispatch integrity check verifies no files were modified or created during the audit.
- The SKILL.md at `.claude/skills/speckit-backtrace-trace/SKILL.md` inlines the command file content. After any edit to the command file, regenerate SKILL.md by extracting the body (without YAML frontmatter) and prepending the skill frontmatter. The SKILL.md is not tracked in the speckit manifest; drift detection is manual.
