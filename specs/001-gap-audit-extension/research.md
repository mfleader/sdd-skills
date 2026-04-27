# Research: SDD Gap Audit Extension

## R1: Extension Structure

**Decision**: Follow the standard speckit extension layout: `extension.yml` at root, `commands/` directory with markdown command file.

**Rationale**: All existing extensions (git, spex, spex-gates, spex-deep-review, spex-teams, spex-worktrees) follow this exact pattern. Constitution Principle III (Speckit-Native) requires it.

**Alternatives considered**:
- Custom plugin structure: Rejected. Violates Principle III.
- Standalone skill without extension packaging: Rejected. Not installable via `specify extension add`.

## R2: Command Naming Convention

**Decision**: Command name is `speckit.gap-audit.audit`.

**Rationale**: All existing commands use `speckit.<extension-id>.<command-name>` (e.g., `speckit.spex-gates.review-spec`, `speckit.spex-worktrees.manage`). Extension ID is `gap-audit`, command suffix is `audit`.

**Alternatives considered**:
- `speckit.gap-audit.run`: Less descriptive than `audit`.
- `speckit.gap-audit.review`: Conflicts conceptually with spex-gates review commands.

## R3: Hook Registration

**Decision**: No lifecycle hooks. The command is invoke-only.

**Rationale**: Per the spec (FR-015), the extension registers no hooks. The gap audit is invoked explicitly, not triggered at phase transitions. The existing `sdd-gap-audit` skill already exists as a standalone skill that can be invoked directly.

**Alternatives considered**:
- Register as `after_specify` hook (optional): Rejected. The spec explicitly says no hooks. The gap audit should run after the review-spec gate, not as part of the specify flow.

## R4: Argument and Flag Handling

**Decision**: The command receives arguments via the `$ARGUMENTS` placeholder in the command markdown. The audit scope (`spec` or `plan`) is a positional argument. The `--output` flag is parsed from arguments, consumed, and the remainder is passed through.

**Rationale**: This matches the pattern used by `speckit.spex-gates.review-code` which parses `--no-external`, `--coderabbit` etc. flags from `$ARGUMENTS`. The spex-worktrees command also parses positional arguments (`list`, `cleanup`) from `$ARGUMENTS`.

**Alternatives considered**:
- Separate positional and flag arguments: No speckit mechanism for this. All input comes through `$ARGUMENTS`.

## R5: SKILL.md Generation

**Decision**: Create a corresponding SKILL.md at `.claude/skills/speckit-gap-audit-audit/SKILL.md` with frontmatter including `name`, `description`, `compatibility`, `metadata.source`, `user-invocable: true`. Set `disable-model-invocation: false` so the command can be invoked programmatically (e.g., by hooks or other commands).

**Rationale**: All extension commands have corresponding SKILL.md files. The `disable-model-invocation` should be `false` because the gap audit may need to be invoked by automated workflows (unlike core speckit commands which are user-initiated).

**Alternatives considered**:
- `disable-model-invocation: true`: Would prevent programmatic invocation, which limits integration with the sdd-drive orchestrator and hook-based workflows.

## R6: Subagent Prompt Construction

**Decision**: The command markdown file contains the full subagent prompt inline, including all checklist items, classification scheme (blocking/non-blocking), output format (JSON array of GapFinding objects), and the directive to not modify files. The prompt is constructed by reading the artifact files and embedding their content.

**Rationale**: Constitution Principle V (Precise Skill Instructions) requires the prompt to include the full checklist, classification scheme, and output constraints. Embedding these in the command file ensures they ship with the extension and cannot drift from the command logic.

**Alternatives considered**:
- External checklist files referenced by the prompt: Adds indirection and risk of drift. Rejected per Principle V.
- Checklist items as configuration: Over-engineering for 12 items that change rarely.

## R7: False Positive Filter Implementation

**Decision**: The 8 false positive filters are embedded in the subagent prompt alongside the checklist. The subagent finds gaps AND applies filters before returning findings. The main agent receives already-filtered results.

**Rationale**: The subagent has full tool access (read files, grep codebase, cross-reference tasks.md). There is no reason to split gap-finding and filtering into separate execution contexts. A single dispatch with both checklist and filters is simpler and matches FR-020 (single dispatch).

**Alternatives considered**:
- Post-processing filters in the main agent after subagent returns: Unnecessary complexity. The subagent can do everything the main agent can.
- A second subagent for filtering: Unnecessary complexity for 8 checks.

## R8: Output Format

**Decision**: Default output is a grouped presentation to the user (blocking findings first, then non-blocking). When `--output` is set, the command additionally writes the findings as a JSON array to the spec directory.

**Rationale**: Per FR-007 and FR-011, presentation is always grouped. JSON persistence is opt-in (US5). The JSON format matches the GapFinding schema (FR-008).

**Alternatives considered**:
- JSON-only output: Not user-friendly for the common case.
- Markdown report file: Additional format to maintain with no clear benefit over JSON for automation or grouped presentation for humans.

## R9: Example gap-patterns.md

**Decision**: Ship an example `gap-patterns.md` file in the `specs/` directory demonstrating the format: canonical name, checklist category, trigger description.

**Rationale**: The spec defines the format in assumptions but a concrete example makes it unambiguous for implementers and users. This was flagged as a planning deliverable.

**Alternatives considered**:
- Document-only (no example file): Users would have to infer the format from the spec's prose description.
