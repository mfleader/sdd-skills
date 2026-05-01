# Feature Specification: Defect Catalog Unification

**Feature Branch**: `005-defect-catalog-unification`
**Created**: 2026-05-01
**Status**: Draft
**Input**: Unify the defect catalog feedback loop: fix the broken exploratory test catalog path, converge the gap-audit and exploratory test pattern entry schemas, and define a single project-level defect catalog contract.

## User Scenarios & Testing

### User Story 1 - Exploratory test reads project-level catalog (Priority: P1)

A speckit user runs `/speckit-exploratory-test-test` on a new spec. The exploratory test skill reads `specs/defect-catalog.md` at the project level and injects the "Exploratory Testing Probes" section into the subagent prompt's Defect Catalog Patterns section. The subagent uses the patterns to seed additional test vectors and tags matching findings with `pattern_match`.

**Why this priority**: This is the broken feedback loop. Without this fix, the exploratory test skill never reads accumulated pattern knowledge, so every spec starts from zero.

**Independent Test**: Run `/speckit-exploratory-test-test` in a project with a `specs/defect-catalog.md` containing at least one pattern in the "Exploratory Testing Probes" section. Verify the subagent prompt includes the catalog patterns and output findings contain `pattern_match` tags where appropriate.

**Acceptance Scenarios**:

1. **Given** a project with `specs/defect-catalog.md` containing an "Exploratory Testing Probes" section with 3 patterns, **When** the exploratory test skill runs, **Then** the subagent prompt includes all 3 patterns in the Defect Catalog Patterns section of the prompt, outside of artifact tags, with the pattern matching instruction preamble.
2. **Given** a project with no `specs/defect-catalog.md`, **When** the exploratory test skill runs, **Then** the skill proceeds without patterns (graceful degradation, no error).
3. **Given** a project with `specs/defect-catalog.md` that has an "Exploratory Testing Probes" section with zero patterns (empty section), **When** the exploratory test skill runs, **Then** the skill logs a warning and proceeds without patterns.
4. **Given** a project with `specs/defect-catalog.md` containing a "Gap Audit Patterns" section but no "Exploratory Testing Probes" section, **When** the exploratory test skill runs, **Then** the skill logs a warning and proceeds without patterns.
5. **Given** a catalog with pattern X in "Exploratory Testing Probes", **When** the subagent produces a finding that matches pattern X's trigger, **Then** the finding includes `pattern_match: X`.

---

### User Story 2 - Gap-audit reads unified catalog (Priority: P2)

A speckit user runs `/speckit-gap-audit-audit` on a spec. The gap-audit skill reads `specs/defect-catalog.md` at the project level and injects the "Gap Audit Patterns" section into the subagent prompt's Pattern Matching section. The subagent uses the patterns to inform its audit and tags matching findings with `pattern_match`.

**Why this priority**: Gap-audit currently reads `specs/gap-patterns.md` with a different entry format. Migrating it to the unified catalog aligns both skills on the same contract. Lower priority than P1 because gap-audit's current path works (it's not broken, just inconsistent).

**Independent Test**: Run `/speckit-gap-audit-audit` in a project with a `specs/defect-catalog.md` containing at least one pattern in the "Gap Audit Patterns" section. Verify the subagent prompt includes the catalog patterns and output findings contain `pattern_match` tags where appropriate.

**Acceptance Scenarios**:

1. **Given** a project with `specs/defect-catalog.md` containing a "Gap Audit Patterns" section with 5 patterns, **When** the gap-audit skill runs, **Then** the subagent prompt includes all 5 patterns in the Pattern Matching section of the prompt.
2. **Given** a project with no `specs/defect-catalog.md`, **When** the gap-audit skill runs, **Then** the skill proceeds without patterns (graceful degradation, no error).
3. **Given** a project with `specs/defect-catalog.md` where the "Gap Audit Patterns" section is missing, **When** the gap-audit skill runs, **Then** the skill logs a warning and proceeds without patterns.
4. **Given** both `specs/gap-patterns.md` and `specs/defect-catalog.md` exist, **When** the gap-audit skill runs, **Then** it reads only `specs/defect-catalog.md` and does not read `specs/gap-patterns.md`.
5. **Given** a catalog with pattern Y in "Gap Audit Patterns", **When** the subagent produces a finding that matches pattern Y's category and trigger, **Then** the finding includes `pattern_match: Y`.

---

### User Story 3 - Catalog contract is documented (Priority: P3)

A project maintainer who operates an external pattern collector reads the defect catalog contract documentation and produces a `specs/defect-catalog.md` file that both skills can consume. The contract defines the file location, TOML frontmatter schema, section names, and entry format.

**Why this priority**: The contract enables external producers (like wh40k's `sdd_collect_patterns.py`) to generate catalog files that the skills can consume. Without the contract, producers have to reverse-engineer the expected format.

**Independent Test**: Create a `specs/defect-catalog.md` conforming to the contract documentation. Run both skills against it. Both skills correctly load their respective sections and use the patterns.

**Acceptance Scenarios**:

1. **Given** a catalog file conforming to the documented contract, **When** either skill loads it, **Then** the skill extracts and uses the patterns from its respective section without errors.
2. **Given** a catalog file missing the TOML frontmatter, **When** either skill loads it, **Then** the skill still reads the section content (frontmatter is metadata for tooling, not required for skill consumption).

---

### Edge Cases

- What happens when the catalog file exists but is completely empty (0 bytes)? Both skills log a warning and proceed without patterns.
- What happens when the catalog file contains sections but with malformed entry format (missing required fields)? The skills embed raw content into subagent prompts. The LLM interprets what it can. No crash, but pattern matching quality degrades.
- What happens when a pattern appears in both the "Gap Audit Patterns" and "Exploratory Testing Probes" sections? This is expected behavior. The same canonical pattern can appear in multiple sections with identical entries. Each skill reads only its own section.
- What happens when `specs/gap-patterns.md` still exists alongside `specs/defect-catalog.md`? The skills read only `specs/defect-catalog.md`. The old file is ignored because the commands no longer reference the old path. Migration documentation notes that `gap-patterns.md` should be removed after catalog unification.

## Requirements

### Functional Requirements

- **FR-001**: The exploratory test extension command MUST read the defect catalog from `specs/defect-catalog.md` relative to the working directory (the project root where the skill is invoked), not from `<spec_dir>/defect-catalog.md`.
- **FR-002**: The gap-audit extension command MUST read the defect catalog from `specs/defect-catalog.md` relative to the working directory, replacing the current `specs/gap-patterns.md` path.
- **FR-003**: Both skills MUST extract only their own section from the catalog: the exploratory test skill reads "Exploratory Testing Probes", the gap-audit skill reads "Gap Audit Patterns". Skills MUST extract sections by H2 heading only. TOML frontmatter (`+++` delimited blocks), if present, MUST be ignored.
- **FR-004**: Both skills MUST gracefully degrade when the catalog file does not exist, logging a warning and proceeding without patterns.
- **FR-005**: Both skills MUST gracefully degrade when their respective section is missing from the catalog file, logging a warning and proceeding without patterns.
- **FR-006**: The gap-audit skill MUST validate that extracted patterns contain at least `name` (H3 heading), `Category`, and `Trigger` fields, identified by bold label at the start of a line (`**FieldName**: value`). If validation fails, log a warning and proceed without patterns. The gap-audit subagent performs two-part matching (category + trigger alignment), so these fields are required for accurate pattern matching. The exploratory test subagent uses broader prose-based matching, so field-level validation is not required for it.
- **FR-007**: The exploratory test skill MUST validate that the extracted section is non-empty (contains at least one H3 pattern heading). If validation fails, log a warning and proceed without patterns.
- **FR-008**: The extension README for both extensions MUST document the defect catalog contract: file location, TOML frontmatter schema, section names, and entry format. Existing per-spec `defect-catalog.md` references in the exploratory test README MUST be replaced with the project-level path.
- **FR-009**: The gap-audit extension command MUST NOT reference `specs/gap-patterns.md`. All references to the old path MUST be replaced with `specs/defect-catalog.md`. This is a **breaking change**: projects that have `specs/gap-patterns.md` but not `specs/defect-catalog.md` will lose pattern support until they migrate. The gap-audit extension.yml version MUST be bumped to `1.0.0` to signal this breaking change.
- **FR-010**: The exploratory test skill MUST inject catalog patterns into the subagent prompt under the "Defect Catalog Patterns" heading, preserving the existing prompt structure. The gap-audit skill MUST inject catalog patterns under the "Pattern Matching" heading, preserving the existing prompt structure. Both injections MUST be outside artifact tags.

### Defect Catalog Contract

**File location**: `specs/defect-catalog.md` (relative to working directory, which MUST be the project root)

**TOML frontmatter** (delimited by `+++` markers, metadata for tooling, not parsed by skills):

```toml
+++
version = 1
last_updated = "2026-05-01"
specs_analyzed = ["042", "043", "044"]
pattern_count = 7
+++
```

Skills skip content between `+++` markers when extracting sections by heading.

**Sections** (H2 headings, order does not matter):

| Section Name | Consumer | Purpose |
|---|---|---|
| Spec Generation Weaknesses | Informational (human) | Patterns spec authors should avoid |
| Gap Audit Patterns | Gap-audit skill | Patterns the auditor checks for |
| Exploratory Testing Probes | Exploratory test skill | Patterns the exploratory tester probes |
| Git-Derived Patterns | Informational (human) | Recurring fix/refactor patterns from commit history |
| Trend Data | Informational (human) | Per-category-per-spec occurrence counts |

Only "Gap Audit Patterns" and "Exploratory Testing Probes" are consumed by skills. Other sections are informational and may be present or absent.

**Section extraction algorithm**: Read from the target H2 heading line (inclusive) to the next H2 heading line (exclusive) or end of file, whichever comes first. Content before the first H2 heading (including TOML frontmatter) is ignored.

**Entry format** (H3 heading under a section, fields identified by bold label at the start of a line):

```markdown
### canonical-name

**Category**: checklist category name
**Trigger**: what to look for when auditing or probing
**Occurrences**: N | **Source**: source label
**Also known as**: alias1, alias2
**Remediation**: what to do about it
**Example**: representative instance description
```

**Required fields per entry**: `canonical-name` (H3 heading), `Category`, `Trigger`
**Optional fields per entry**: `Occurrences`, `Source`, `Also known as`, `Remediation`, `Example`
**Field order within an entry does not matter.** Fields not matching the `**FieldName**: value` pattern are ignored.

**Field definitions**:

- **canonical-name**: Lowercase hyphenated identifier for the pattern (e.g., `weak-acs`, `orphan-frs`). Used as the value of `pattern_match` when a finding matches this pattern.
- **Category**: The checklist category this pattern belongs to (e.g., "Weak ACs", "Orphan FRs", "Unverifiable SCs"). Used by the gap-audit subagent for two-part matching (category + trigger).
- **Trigger**: A prose description of the condition that indicates this pattern is present. Used by both subagents to match findings against patterns.
- **Occurrences**: How many times this pattern has been observed, with the source label (e.g., "Spec findings", "Git history", "Defect taxonomy"). Collector metadata.
- **Also known as**: Alternative names or aliases that normalize to this canonical name. Collector metadata.
- **Remediation**: What to do when this pattern is found. Actionable guidance for the spec author or implementer.
- **Example**: A representative instance of this pattern from a real finding. Provides concrete context for pattern matching.

### Key Entities

- **Defect Catalog**: A project-level markdown file (`specs/defect-catalog.md`) containing accumulated cross-spec defect patterns organized by consumer section. Externally produced by a project-specific collector.
- **Pattern Entry**: A single defect pattern within a catalog section, identified by its canonical name, with Category, Trigger, and optional metadata fields.
- **Collector**: An external tool (not part of speckit) that aggregates findings across specs and produces the defect catalog. Each project brings its own collector.

## Success Criteria

### Measurable Outcomes

- **SC-001**: The exploratory test skill loads patterns from `specs/defect-catalog.md` and injects them into the subagent prompt's Defect Catalog Patterns section when the file and section exist.
- **SC-002**: The gap-audit skill loads patterns from `specs/defect-catalog.md` and injects them into the subagent prompt's Pattern Matching section when the file and section exist.
- **SC-003**: Both skills proceed without error when `specs/defect-catalog.md` does not exist.
- **SC-004**: A catalog file conforming to the documented contract is loadable by both skills, with each skill extracting and injecting its respective section into the subagent prompt.
- **SC-005**: The `specs/gap-patterns.md` file is no longer referenced by any extension command, and the gap-audit extension.yml version is `1.0.0`.
- **SC-006**: Both extension READMEs document the defect catalog contract including file location, TOML frontmatter schema, section names, and entry format.
- **SC-007**: When the gap-audit skill extracts patterns missing required fields (Category or Trigger), it logs a warning and proceeds without patterns.
- **SC-008**: When the exploratory test skill extracts an empty section (no H3 headings), it logs a warning and proceeds without patterns.

## Assumptions

- The defect catalog is always externally produced. No speckit extension ships a collector. Projects bring their own tooling to generate the file.
- The TOML frontmatter is delimited by `+++` markers (the standard TOML frontmatter convention). Skills skip frontmatter when extracting sections by heading.
- Skills are invoked from the project root (the directory containing `.specify/`). The path `specs/defect-catalog.md` is relative to the working directory.
- Existing projects using `specs/gap-patterns.md` will need to rename/regenerate that file as `specs/defect-catalog.md` with the converged entry format. This is a one-time migration handled by each project independently. Existing per-spec `defect-catalog.md` files should be consolidated into the project-level `specs/defect-catalog.md`.
- The collector in wh40k will need to be updated to produce `Category` and `Trigger` fields and write to `specs/defect-catalog.md`. That work is tracked separately in the wh40k project.
- Patterns can appear in multiple sections. The collector routes patterns to sections based on their originating sources. Skills read only their own section.

## Out of Scope

- Modifying the wh40k pattern collector (`sdd_collect_patterns.py`). The collector adapts to the contract defined here, but implementation changes are tracked in that project.
- Changing the findings output schemas for either skill. Gap-audit findings and exploratory test findings remain separate schemas.
- Building a generic collector that ships with speckit.
- Per-spec catalog files. The catalog is project-level only.
- Unifying the "Spec Generation Weaknesses" or "Git-Derived Patterns" sections with any skill. Those sections are informational for humans.
