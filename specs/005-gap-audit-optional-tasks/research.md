# Research: Gap Audit Optional tasks.md

## R1: Current command file structure for plan scope

**Decision**: Modify Sections 4, 6, 8, 9, and 10 of the existing command file.

**Rationale**: The command file has 10 clearly delineated sections. Plan scope behavior is concentrated in Sections 4 (artifact loading, lines 69-84), 6 (subagent prompt, lines 249-302), 8 (response parsing, lines 326-337), 9 (JSON persistence, lines 339-355), and 10 (output formatting, lines 357-409). Each section can be modified independently.

**Alternatives considered**: Rewriting the entire command file was rejected as unnecessary risk. The existing structure cleanly separates concerns.

## R2: How to express conditional behavior in LLM prose

**Decision**: Use explicit "### Plan scope (tasks.md present)" and "### Plan scope (tasks.md absent)" subsections within Section 4 and Section 6 where behavior diverges.

**Rationale**: The command file already uses similar patterns (separate subsections for spec vs plan scope in Sections 4 and 6). Adding a second level of branching within plan scope follows the established pattern.

**Alternatives considered**: Using inline "If tasks.md exists..." prose was considered but rejected because it's harder to follow for multi-paragraph conditional blocks.

## R3: Where to add category validation (FR-017)

**Decision**: Add a new validation step at the end of Section 8 (Response Parsing), after JSON extraction but before Section 9 (persistence).

**Rationale**: Section 8 already handles parse validation (valid JSON, required fields). Category validation is a natural extension of this section. It must run before Section 9 to prevent invalid findings from being persisted.

**Alternatives considered**: Adding validation in Section 6 (as a subagent instruction) is already done via FR-007, but LLM subagents are non-deterministic, so host-side enforcement is needed as a safety net.

## R4: Extension version bump

**Decision**: Bump from 0.1.0 to 0.2.0 (minor version for new feature, no breaking changes).

**Rationale**: The `categories_audited` field is additive (new optional field on persisted findings). Existing consumers ignore unknown fields. This is a minor feature addition, not a breaking change.

**Alternatives considered**: Patch bump (0.1.1) was rejected because this adds a new field to the finding schema, which is a feature. Major bump (1.0.0) was rejected because there are no breaking changes.

## R5: Original spec (001) update scope

**Decision**: Update FR-003, FR-013, US2, Edge Cases, Key Entities, and Assumptions in the original spec to reflect the new behavior. Add the `categories_audited` field to the data model.

**Rationale**: The original spec documents the command's intended behavior. When the command behavior changes, the spec should be updated to remain the source of truth.

**Alternatives considered**: Creating a separate "amendment" document was rejected because it fragments the source of truth.
