# Tasks: Defect Catalog Unification

**Input**: Design documents from `specs/005-defect-catalog-unification/`
**Prerequisites**: plan.md (required), spec.md (required)

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Phase 1: Setup

**Purpose**: Create the unified catalog file that both skills will read from.

- [x] T001 Create `specs/defect-catalog.md` with TOML frontmatter (`+++` delimited), "Gap Audit Patterns" H2 section (migrated from `specs/gap-patterns.md`: convert pattern name headings from H2 to H3 to conform to the converged entry format, preserve existing Category and Trigger fields, add optional fields Occurrences/Source/Also known as/Remediation/Example where available), and empty "Exploratory Testing Probes" H2 section. Note: the empty Exploratory Testing Probes section will trigger the empty-section warning path when the exploratory test skill runs in this repo. This is expected initial state.
- [x] T002 Delete `specs/gap-patterns.md` after confirming content migrated to `specs/defect-catalog.md`. Run `grep -r "gap-patterns.md" .specify/ .claude/ specs/` across the repo to verify no stale references remain in any file.

---

## Phase 2: User Story 1 - Exploratory test reads project-level catalog (P1)

**Story Goal**: Fix the broken feedback loop. The exploratory test skill reads `specs/defect-catalog.md` at the project level instead of `<spec_dir>/defect-catalog.md`.

**Independent Test**: Run `/speckit-exploratory-test-test` in this repo with the `specs/defect-catalog.md` created in Phase 1. Verify the subagent prompt includes the "Exploratory Testing Probes" section content.

- [x] T003 [US1] Update Section 6 (Defect Catalog Loading) in `.specify/extensions/exploratory-test/commands/speckit.exploratory-test.test.md`: change file path from `<spec_dir>/defect-catalog.md` to `specs/defect-catalog.md`. Update the bash conditional to check `specs/defect-catalog.md` instead. Keep section extraction ("Exploratory Testing Probes"), non-empty validation (at least one H3 heading), and warning/graceful degradation behavior. Add TOML frontmatter skipping (ignore content between `+++` markers). Section extraction reads from the target H2 heading line (inclusive) to the next H2 heading line (exclusive) or end of file, whichever comes first. Handle empty file (0 bytes) by logging a warning and proceeding without patterns. Malformed entries within a non-empty section are still injected per spec edge case guidance.
- [x] T004 [US1] Update the exploratory-test README at `.specify/extensions/exploratory-test/README.md`: replace the Defect Catalog Integration section (currently starting with "Create a defect-catalog.md file in the spec directory") with project-level `specs/defect-catalog.md` documentation. Document the converged entry format (canonical-name as H3 heading, Category, Trigger, Occurrences, Source, Also known as, Remediation, Example) and the TOML frontmatter convention.
- [x] T005 [P] [US1] Regenerate `.claude/skills/speckit-exploratory-test-test/SKILL.md` from the updated command file to ensure consistency, applying the same path change from T003 (`<spec_dir>/defect-catalog.md` → `specs/defect-catalog.md`).
- [x] T012 [US1] Verify US1 end-to-end: run `/speckit-exploratory-test-test` against a spec in this repo with the `specs/defect-catalog.md` created in T001. Verify the subagent prompt includes the Exploratory Testing Probes section content in the Defect Catalog Patterns prompt section, outside artifact tags. Check that pattern_match tags appear in findings output when findings match cataloged patterns (best-effort manual verification since subagent output is non-deterministic).

---

## Phase 3: User Story 2 - Gap-audit reads unified catalog (P2)

**Story Goal**: Migrate gap-audit from `specs/gap-patterns.md` to the unified `specs/defect-catalog.md`, extracting the "Gap Audit Patterns" section.

**Independent Test**: Run `/speckit-gap-audit-audit spec` in this repo with the `specs/defect-catalog.md` created in Phase 1. Verify the subagent prompt includes the "Gap Audit Patterns" section content.

- [x] T006 [US2] Update Section 5 (Gap-Patterns Loading) in `.specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md`: change file path from `specs/gap-patterns.md` to `specs/defect-catalog.md`. Add section extraction to read only the "Gap Audit Patterns" H2 section (currently reads the whole file). Section extraction reads from the target H2 heading line (inclusive) to the next H2 heading line (exclusive) or end of file, whichever comes first. Keep validation (entries must have name/H3 heading, Category, Trigger fields identified by bold label). Add TOML frontmatter skipping (ignore content between `+++` markers). Handle empty file (0 bytes) by logging a warning and proceeding without patterns. Update graceful degradation to handle missing section. After changes, verify the command no longer references `specs/gap-patterns.md` in any code path, satisfying AC US2.4 (coexistence scenario).
- [x] T007 [US2] Update Section 6 pattern matching prompt in `.specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md`: change "content of gap-patterns.md embedded here" to "content of 'Gap Audit Patterns' section embedded here". Update the conditional text from "if gap-patterns.md was loaded" to "if Gap Audit Patterns section was loaded from defect catalog".
- [x] T008 [P] [US2] Bump version in `.specify/extensions/gap-audit/extension.yml` from `0.1.0` to `1.0.0`
- [x] T009 [P] [US2] Update gap-audit README at `.specify/extensions/gap-audit/README.md`: replace the Gap Patterns section (currently starting with "Format:" and showing H2-level pattern entries) with `specs/defect-catalog.md` documentation. Document the converged entry format and TOML frontmatter convention. Note the breaking change from `gap-patterns.md`.
- [x] T010 [P] [US2] Verify `.claude/skills/speckit-gap-audit-audit/SKILL.md` does not contain catalog path references (expected: none, since it is a thin wrapper that delegates to the command file).
- [x] T013 [US2] Verify US2 end-to-end: run `/speckit-gap-audit-audit spec` against a spec in this repo with the `specs/defect-catalog.md` created in T001. Verify the subagent prompt includes the Gap Audit Patterns section content in the Pattern Matching prompt section. Check that pattern_match tags appear in findings output when findings match cataloged patterns (best-effort manual verification).

---

## Phase 4: User Story 3 - Catalog contract is documented (P3)

**Story Goal**: Ensure both READMEs document the full defect catalog contract consistently.

**Independent Test**: Read both READMEs. Both should describe the same file location, TOML frontmatter schema, section names, entry format, required/optional fields.

- [x] T011 [US3] Verify consistency between gap-audit README and exploratory-test README catalog documentation. Both must describe: file location (`specs/defect-catalog.md`), TOML frontmatter schema (`+++` delimited, version/last_updated/specs_analyzed/pattern_count), section names (table of 5 sections with consumer and purpose), entry format (H3 heading + bold-label fields), required fields (canonical-name, Category, Trigger), optional fields (Occurrences, Source, Also known as, Remediation, Example). Note: cross-section duplication edge case (same pattern in both sections) is not testable with the initial catalog since the Exploratory Testing Probes section is empty. Verify when a project produces a catalog with overlapping patterns.

---

## Dependencies

```text
T001 → T002 (delete old file after migration)
T001 → T003, T004, T005 (US1 depends on catalog file existing)
T001 → T006, T007, T008, T009, T010 (US2 depends on catalog file existing)
T003 → T005 (SKILL.md regenerated from updated command file)
T003, T004, T005 → T012 (US1 integration verification after all changes)
T006, T007 → T013 (US2 integration verification after command changes)
T004, T009 → T011 (US3 verifies consistency after both READMEs updated)
T006, T007, T009 → T011

T003 and T006 are independent (different extension command files)
T004 and T009 are independent (different README files)
T005 and T010 are independent (different SKILL.md files)
```

## Parallel Execution

**Within Phase 2 (US1)**: T003 and T004 can run in parallel. T005 depends on T003. T012 runs after T003+T004+T005.
**Within Phase 3 (US2)**: T006 and T007 must be sequential (same file). T008, T009, T010 can run in parallel with each other and with T006/T007. T013 runs after T006+T007.
**Across phases**: Phase 2 and Phase 3 can run in parallel after Phase 1 completes.

## Implementation Strategy

**MVP**: Phase 1 + Phase 2 (US1). This fixes the broken feedback loop, which is the primary problem.
**Full delivery**: All phases. Phase 3 (US2) is important for consistency but gap-audit's current path works. Phase 4 (US3) is verification only.
