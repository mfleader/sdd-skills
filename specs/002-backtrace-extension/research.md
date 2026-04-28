# Research: Backtrace Extension

## R-001: Hook Declaration Schema

**Decision**: Hooks are declared in extension.yml under a `hooks:` key. Each hook point maps to exactly **one** command (single object, not an array).

**Schema**:
```yaml
hooks:
  <hook_point>:
    command: <fully-qualified-command-name>
    optional: <boolean>  # defaults to true if omitted
    prompt: <string>     # auto-generated as "Execute {command}?" if omitted
    description: <string>
```

**Fields**:
- `command` (required): Fully qualified command name (e.g., `speckit.spex-gates.review-spec`)
- `optional` (optional, defaults to `true`): `false` = mandatory, `true` = user can skip
- `prompt` (optional, auto-generated): User-facing question. Auto-generated as "Execute {command}?" if omitted. Present in the merged config regardless of `optional` status.
- `description` (required): Human-readable explanation

**Constraint**: Each hook point maps to one command per extension. The specify CLI validates that hook config is a dict, not a list. Duplicate entries from the same extension at the same hook point are overwritten.

**Available hook points**: `before_/after_` + `constitution`, `specify`, `clarify`, `plan`, `tasks`, `implement`, `checklist`, `analyze`, `taskstoissues`

**Registration vs consumption**: Declaring a hook in extension.yml registers it in `.specify/extensions.yml` on install. But registration alone does nothing. The command that runs at that lifecycle step must contain inline hook-checking code that reads `.specify/extensions.yml` and dispatches registered hooks. Core commands (plan, implement, etc.) have this code built in. Custom hook points (like `after_backtrace`) require the backtrace command file to contain the same dispatch pattern.

**Rationale**: Verified against spex-gates, git, and spex-deep-review extension.yml files. Hook validation confirmed via specify CLI source (extensions.py).

**Alternatives considered**: Using existing hook points (rejected: backtrace is not a core lifecycle step), direct invocation without hooks (rejected: not composable).

## R-002: Command File Structure

**Decision**: Command files are markdown with YAML front matter, following a consistent pattern.

**Required front matter**:
```yaml
---
description: "Human-readable description"
---
```

**Standard sections** (applicable to backtrace):
1. Ship Pipeline Guard (check `.specify/.spex-state` for autonomous mode)
2. Overview
3. Prerequisites (speckit version check, file existence)
4. Argument parsing (scope, spec directory)
5. Artifact loading
6. Core execution (tracing, proposing additions)
7. Subagent dispatch (auditor)
8. Response parsing (JSON handling)
9. Output formatting
10. Hook checking (after_backtrace)

**Rationale**: Matches gap-audit command file (10 sections) and review-spec command file.

## R-003: Subagent Dispatch Pattern

**Decision**: Use `superpowers:code-reviewer` subagent type with a structured prompt including the auditor directive "DO NOT modify any files."

**Prompt structure** (from gap-audit pattern):
1. Preamble with CRITICAL RULES
2. Artifact content wrapped in XML-style tags
3. Classification scheme
4. Output format (JSON)
5. False positive filters (adapted for backtrace: verify additions address gaps, check for redundancy)

**Post-dispatch verification**: Check `git diff --name-only` to verify no files were modified during audit.

**Rationale**: Same pattern as gap-audit. Proven to work with the auditor subagent.

## R-004: Spec Directory Resolution

**Decision**: Three-step resolution: (1) argument, (2) `.specify/feature.json`, (3) ask user.

**Implementation**:
```bash
FEATURE_DIR=$(jq -r '.feature_directory // empty' .specify/feature.json 2>/dev/null)
```

**Rationale**: Matches gap-audit FR-024 and speckit conventions.

## R-005: Extension.yml for Backtrace (revised after gap audit)

**Constraint discovered**: Each hook point maps to one command per extension. Backtrace cannot declare separate hooks for review-spec, review-plan, and analyze at the same `after_backtrace` hook point.

**Solution**: Backtrace declares no hooks in extension.yml. Instead, the backtrace command file directly invokes review-spec, review-plan, and analyze as its final sections (same pattern as review-code invoking deep-review). This is simpler, avoids the untested cross-extension hook reference pattern, and handles spex-gates absence gracefully inline.

**Proposed extension.yml**:
```yaml
schema_version: "1.0"

extension:
  id: backtrace
  name: "SDD Backtrace"
  version: "0.1.0"
  description: "Trace gap-audit findings back to missing spec items and propose additions"
  author: cc-spex
  license: MIT

requires:
  speckit_version: ">=0.5.2"

provides:
  commands:
    - name: speckit.backtrace.trace
      file: commands/speckit.backtrace.trace.md
      description: "Trace findings back to spec gaps and propose additions"

tags:
  - "sdd"
  - "backtrace"
  - "quality"
```

**Follow-up invocations** (in command file, not hooks):
After applying additions, the command file checks if spex-gates is installed via the extensions registry, then sequentially invokes:
1. `speckit.spex-gates.review-spec` (if available)
2. `speckit.spex-gates.review-plan` (if available, plan scope only)
3. `speckit-analyze` (if available)

If spex-gates is not installed, these are skipped with a warning.

**Rationale**: One-hook-per-point constraint makes hook-based follow-ups impossible for backtrace. Direct invocation is the same pattern review-code uses for deep-review.

**Alternatives rejected**:
- Array hooks in extension.yml (invalid schema, CLI raises ValidationError)
- Multiple hook points (after_backtrace_1, after_backtrace_2) - not how the system works
- Composite follow-up command - unnecessary abstraction for 3 sequential invocations

## R-006: SKILL.md Schema

**Decision**: SKILL.md is a Claude Code skill wrapper with YAML front matter.

**Front matter schema**:
```yaml
---
name: speckit-backtrace-trace
description: Trace gap-audit findings back to missing spec items and propose additions
compatibility: Requires spec-kit project structure with .specify/ directory
metadata:
  author: github-spec-kit
  source: backtrace:commands/speckit.backtrace.trace.md
user-invocable: true
disable-model-invocation: false
---
```

**Naming convention**: `speckit-{ext-id}-{command}` with dots replaced by hyphens.

**Content strategy**: The specify CLI's `_register_extension_skills()` copies command file body directly into SKILL.md (inline pattern). Gap-audit uses a delegation pattern (thin wrapper that reads the command file). Since we're hand-writing the extension (not using `specify extension add` from a remote), we can use either pattern. The delegation pattern is simpler for development.

**Rationale**: Verified against gap-audit SKILL.md and other installed extension skill wrappers.
