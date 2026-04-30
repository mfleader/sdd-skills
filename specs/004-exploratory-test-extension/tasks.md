# Tasks: Exploratory Test Extension

**Input**: Design documents from `specs/004-exploratory-test-extension/`
**Prerequisites**: plan.md, spec.md, data-model.md, contracts/exploratory-findings.json, research.md

**Tests**: Not applicable. This is a prompt-engineering artifact (LLM-executed prose), not compiled code. Verification is via the behavioral parity checklist (SC-001).

**Organization**: All user stories are implemented by sections of a single command file (`speckit.exploratory-test.test.md`). Tasks are organized by implementation phase (command file sections) with user story labels showing which stories each section serves.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (US1-US5)
- Source skill reference: `/home/mleader/workspace/py/warhammer/wh40k/.claude/skills/sdd-exploratory-test/SKILL.md`
- Gap-audit reference: `.specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md`

---

## Phase 1: Setup

**Purpose**: Create extension directory structure and metadata

- [X] T001 Create extension directory structure: `.specify/extensions/exploratory-test/commands/`
- [X] T002 [P] Write extension metadata in `.specify/extensions/exploratory-test/extension.yml` (schema_version, id: exploratory-test, name, version: 0.1.0, description, author, license, requires speckit >= 0.5.2, provides commands speckit.exploratory-test.test, tags)

**Checkpoint**: Extension directory exists with valid extension.yml

---

## Phase 2: Command File — Input Sections (1-6)

**Purpose**: Speckit boilerplate and input gathering. These sections run before the subagent dispatch.

- [X] T003 [US5] Write Section 1 (Version Check) in `.specify/extensions/exploratory-test/commands/speckit.exploratory-test.test.md`. Check speckit version from `.specify/init-options.json` against 0.5.2. Strip pre-release suffixes. Proceed with warning if undetermined. Model after gap-audit Section 1.
- [X] T004 [US1] [US2] [US3] [US5] Write Section 2 (Argument Parsing) in the command file. Parse $ARGUMENTS: extract `--output` flag (set OUTPUT_FLAG), extract `--base <branch>` flag (set BASE_BRANCH, default `main`), remaining token is spec directory path. Parsing order: flags first, then positional. Per FR-001.
- [X] T005 [US5] Write Section 3 (Spec Directory Resolution) in the command file. Three-step fallback: (1) argument path, (2) `.specify/feature.json` feature_directory, (3) ask user. When --output is passed, disable ask-the-user fallback (error instead). Validate path exists and contains spec.md. Per FR-002. Model after gap-audit Section 3.
- [X] T006 [US1] Write Section 4 (Artifact Loading) in the command file. Read spec.md (existence already validated in Section 3), extract user stories, FRs, ACs, SCs. Store full content. Per FR-004.
- [X] T007 [US1] [US3] Write Section 5 (Changed File Discovery) in the command file. Validate base branch with `git rev-parse --verify <base>`. Run `git diff --name-only <base> -- .`. Store changed file list. Per FR-005. This section has no gap-audit equivalent (new section).
- [X] T008 [US4] Write Section 6 (Defect Catalog Loading) in the command file. Check for `defect-catalog.md` in spec directory. If present, read "Exploratory Testing Probes" section, validate non-empty, store patterns. If absent, proceed without patterns. Per FR-006. Model after gap-audit Section 5 (Gap-Patterns Loading).

**Checkpoint**: All input sections complete. Command can gather all required context.

---

## Phase 3: Command File — Dispatch Section (7-8)

**Purpose**: Subagent prompt construction, dispatch, and integrity check. This is the largest section — transplant the source skill's test vector checklist, self-check gate, and adversarial directive.

- [X] T009 [US1] [US4] Write Section 7 (Subagent Prompt Construction) in the command file. Construct the full prompt with these components: (1) preamble with adversarial directive, (2) spec content wrapped in `<artifact>` tags, (3) changed file list (if empty, instruct subagent to proceed with spec-only testing and note limited implementation context), (4) instruction to read changed files and understand implementation, (5) test vector checklist (7 categories verbatim from source SKILL.md: boundary value analysis, type edges, invariant violations, feature interaction, negative testing, mock fidelity, regression), (6) instruction to write tests to `/tmp/exploratory-tests/` (overwrite if directory exists; if /tmp is not writable, report as a finding rather than silently skipping), (7) instruction to run project test suite for regression (attempt common test runners: make test, pytest, go test, npm test; if no test suite is discoverable, skip regression check and note it was skipped), (8) self-check gate (assertion mismatch, uncaught exceptions, parameter errors, low-confidence marking), (9) output format (ExploratoryFinding schema from data-model.md — field names MUST match exactly: severity, description, reproduction, expected, actual, spec_gap), (10) focused test instruction (no exhaustive matrices), (11) clean report acceptance, (12) when defect catalog loaded: pattern-match tagging instruction, (13) edge case handling: if tests fail to compile or run, report as a finding, do not silently swallow (per spec.md edge case #2), (14) instruct subagent to return ONLY the specified fields, no additional fields (per contracts/exploratory-findings.json additionalProperties: false). Per FR-007, FR-008, FR-014, FR-015.
- [X] T010 [US1] [US5] Write Section 8 (Subagent Dispatch + Integrity Check) in the command file. Dispatch single subagent: subagent_type `superpowers:code-reviewer`, description "Exploratory test", prompt from Section 7. Post-dispatch: run `git diff --name-only` and `git ls-files --others --exclude-standard`. Warn with specific messages for modified vs created files. Per FR-007, FR-012, SC-005.

**Checkpoint**: Command can dispatch subagent and verify repo integrity.

---

## Phase 4: Command File — Output Sections (9-11)

**Purpose**: Parse subagent response, inject source field, persist and present findings.

- [X] T011 [US1] [US5] Write Section 9 (Response Parsing) in the command file. Defensive parsing in this order: (1) empty or whitespace-only response (set PARSE_FAILED, treat as empty array), (2) valid JSON array (proceed), (3) empty array (no findings), (4) non-JSON prose (bracket matching extraction, validate first element has required fields: severity, description, reproduction, expected, actual, spec_gap; if extraction OR validation fails, set PARSE_FAILED, treat as empty array), (5) malformed JSON (PARSE_FAILED, empty array). First-element validation is a structural sanity check only. Per FR-009.
- [X] T012 [US2] Write Section 10 (JSON Persistence) in the command file. When --output: add `"source": "exploratory"` to each finding (FR-010), write to `<spec_dir>/.exploratory-findings.json`, overwrite existing, write `[]` when empty, validate with `jq`. If jq validation fails, warn: "Output file written but JSON validation failed. The file may be corrupt. Re-run the command." Do not delete the file (per FR-011). When not --output: skip entirely. Per FR-010, FR-011, SC-003.
- [X] T013 [US1] Write Section 11 (Output Formatting) in the command file. Present grouped by severity (critical, important, moderate, minor). Include pattern_match when present. When PARSE_FAILED: "Testing completed but the subagent did not return structured findings. Re-run the test or review the subagent output manually." When no findings and not PARSE_FAILED: "No issues found." When --output was set: "Findings written to: <path>". Per FR-013.

**Checkpoint**: Command file is complete. All 11 sections written.

---

## Phase 5: User Story 3 — Base Branch (Priority: P2)

**Goal**: `--base <branch>` flag support for non-standard branch workflows.

**Independent Test**: Invoke with `--base develop` on a branch diverged from develop. Verify git diff uses develop.

Already implemented in T004 (argument parsing) and T007 (changed file discovery). No additional tasks needed — this phase is a verification checkpoint only.

**Checkpoint**: `--base` flag works correctly with Section 2 and Section 5.

---

## Phase 6: User Story 4 — Defect Catalog (Priority: P3)

**Goal**: Load recurring defect patterns from `defect-catalog.md` and use them as additional test vectors.

**Independent Test**: Create a defect-catalog.md, invoke command, verify subagent receives patterns and tags matching findings.

Already implemented in T008 (catalog loading) and T009 (prompt construction includes pattern-match tagging). No additional tasks needed — this phase is a verification checkpoint only.

**Checkpoint**: Defect catalog integration works end-to-end.

---

## Phase 7: README and Polish

**Purpose**: Documentation and final verification

- [X] T014 [P] Write README.md in `.specify/extensions/exploratory-test/README.md`. Include: requirements (speckit >= 0.5.2, Claude Code with superpowers:code-reviewer), installation (specify extension add), usage examples (basic invoke, --base, --output), output format (severity grouping), defect catalog integration, findings JSON schema reference, notes (invoke-only, no hooks, single dispatch, active testing). Model after gap-audit README.md.
- [X] T015 Run behavioral parity checklist (SC-001): diff source SKILL.md against command file, verify all 33 checklist items pass. Document results in the parity checklist table in spec.md.
- [X] T016 Verify SC-002 (speckit boilerplate sections present), SC-003 (source field), SC-004 (no behavioral additions), SC-005 (integrity check correctness) by reading the command file. Additionally: (a) validate that a sample output would conform to `contracts/exploratory-findings.json` schema (required fields, severity enum, source const, no extra fields), (b) verify that the field names in Section 7's output format specification exactly match the field names validated in Section 9's parser (both must reference data-model.md as canonical source), (c) verify backtrace and sdd_collect_patterns.py can consume `.exploratory-findings.json` without error (if not feasible, document as follow-up task on backtrace extension).

**Checkpoint**: Extension is complete and verified.

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — can start immediately
- **Command Input Sections (Phase 2)**: Depends on T001 (directory must exist). T003-T008 are sequential within the command file.
- **Command Dispatch Section (Phase 3)**: Depends on Phase 2 completion (prompt needs all inputs)
- **Command Output Sections (Phase 4)**: Depends on Phase 3 (parses dispatch response)
- **US3 Verification (Phase 5)**: Verification only — depends on Phase 4
- **US4 Verification (Phase 6)**: Verification only — depends on Phase 4
- **README + Polish (Phase 7)**: T014 (README) can run in parallel with Phase 2-4. T015-T016 depend on Phase 4.

### User Story Mapping

- **US1 (Run Tests)**: T006, T007, T009, T010, T011, T013
- **US2 (Persist Findings)**: T004, T012
- **US3 (Base Branch)**: T004, T007
- **US4 (Defect Catalog)**: T008, T009
- **US5 (Boilerplate)**: T003, T004, T005, T010, T011

### Parallel Opportunities

- T001 and T002 can run in parallel (directory creation vs extension.yml writing)
- T014 (README) can be written in parallel with the command file (Phase 2-4)
- Within Phase 2, sections are sequential (each builds on prior context in the file)

---

## Parallel Example: Setup

```bash
# Launch setup tasks together:
Task: "Create extension directory structure"
Task: "Write extension.yml metadata"
```

---

## Implementation Strategy

### MVP First (US1 + US2 + US5)

1. Complete Phase 1: Setup (T001-T002)
2. Complete Phase 2: Input sections (T003-T008)
3. Complete Phase 3: Dispatch section (T009-T010)
4. Complete Phase 4: Output sections (T011-T013)
5. **STOP and VALIDATE**: Run parity checklist (T015)
6. US1, US2, US5 are now complete

### Incremental Delivery

1. Setup → Command file complete → Parity check → README
2. Each user story is testable after the command file is complete
3. US3 and US4 require no additional implementation (already woven into command sections)

---

## Notes

- This is a mechanical repackaging. Every command section should be modeled after the gap-audit equivalent.
- Section 7 (prompt construction) is the largest task. Transplant the source skill's test vector checklist and self-check gate verbatim.
- The source skill at `/home/mleader/workspace/py/warhammer/wh40k/.claude/skills/sdd-exploratory-test/SKILL.md` is the authoritative reference. Read it before writing each section.
- SC-004 constrains: no behavioral additions beyond speckit boilerplate.
