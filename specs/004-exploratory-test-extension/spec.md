# Feature Specification: Exploratory Test Extension

**Feature Branch**: `004-exploratory-test-extension`
**Created**: 2026-04-30
**Status**: Draft
**Input**: Package the exploratory test skill as a speckit extension

## User Scenarios & Testing

### User Story 1 - Run Exploratory Tests Against Implementation (Priority: P1)

A developer has finished implementing a feature and wants to find edge cases and bugs the spec missed. They invoke the exploratory test command, which dispatches an adversarial subagent to exercise the implementation beyond its success criteria.

**Why this priority**: Core functionality. Without this, the extension has no purpose.

**Independent Test**: Invoke `/speckit-exploratory-test-test spec_dir` against a feature with known edge cases. The subagent should find at least one issue not covered by existing tests.

**Acceptance Scenarios**:

1. **Given** a spec directory containing `spec.md` and implementation changes on the current branch, **When** the user invokes `speckit.exploratory-test.test <spec_dir>`, **Then** the subagent reads the spec, identifies changed files via git diff, writes tests to `/tmp/exploratory-tests/`, runs them, runs the project test suite, and reports findings with severity, description, reproduction, expected, actual, and spec_gap fields.

2. **Given** a clean implementation with no edge case bugs, **When** the user invokes the command, **Then** the subagent reports "No issues found." without manufacturing low-severity findings.

3. **Given** a spec directory without `spec.md`, **When** the user invokes the command, **Then** the extension reports an error and stops.

---

### User Story 2 - Persist Findings to JSON (Priority: P1)

A developer wants findings written to a file so downstream tools (pattern collector, drive orchestrator) can consume them programmatically.

**Why this priority**: Required for integration with the SDD feedback loop (collector aggregates findings into patterns that inject into new specs).

**Independent Test**: Invoke with `--output` flag. Verify `.exploratory-findings.json` is written to the spec directory with correct schema including `"source": "exploratory"` on each finding.

**Acceptance Scenarios**:

1. **Given** the user passes `--output`, **When** findings exist, **Then** the extension writes them to `<spec_dir>/.exploratory-findings.json` as a JSON array with `"source": "exploratory"` added to each finding.

2. **Given** the user passes `--output`, **When** no findings exist, **Then** the extension writes an empty array `[]` to the file.

3. **Given** the user does NOT pass `--output`, **When** the command completes, **Then** no findings file is written.

4. **Given** a previous `.exploratory-findings.json` exists, **When** the user runs the command with `--output`, **Then** the existing file is overwritten.

---

### User Story 3 - Specify Base Branch for Changed File Discovery (Priority: P2)

A developer working on a branch that diverged from a branch other than `main` needs to specify the base branch for accurate changed-file discovery.

**Why this priority**: Important for non-standard branch workflows, but `main` default covers the common case.

**Independent Test**: Invoke with `--base develop` on a branch that diverged from `develop`. Verify git diff runs against `develop`, not `main`.

**Acceptance Scenarios**:

1. **Given** the user passes `--base develop`, **When** the command runs, **Then** `git diff --name-only develop -- .` is used to discover changed files.

2. **Given** the user does not pass `--base`, **When** the command runs, **Then** `git diff --name-only main -- .` is used (default).

3. **Given** the user passes `--base nonexistent-branch`, **When** the command runs, **Then** the extension reports an error ("Base branch 'nonexistent-branch' does not exist") and stops.

---

### User Story 4 - Leverage Defect Catalog for Targeted Probing (Priority: P3)

A developer has accumulated recurring defect patterns in a `defect-catalog.md` file. The exploratory tester should use these patterns as additional test vectors to catch known problem categories.

**Why this priority**: Enhancement that improves findings quality over time, but the extension works without it.

**Independent Test**: Create a `defect-catalog.md` with an "Exploratory Testing Probes" section containing a known pattern. Invoke the command. Verify the subagent receives the pattern as an additional test vector and tags matching findings with `pattern_match`.

**Acceptance Scenarios**:

1. **Given** `defect-catalog.md` exists in the spec directory with an "Exploratory Testing Probes" section, **When** the command runs, **Then** each listed pattern is included as an additional test vector in the subagent prompt.

2. **Given** a finding matches a known defect catalog pattern, **When** findings are reported, **Then** the finding includes `"pattern_match": "<canonical-name>"`.

3. **Given** no `defect-catalog.md` exists in the spec directory, **When** the command runs, **Then** the extension proceeds without patterns (no error).

---

### User Story 5 - Speckit Boilerplate Compliance (Priority: P1)

The extension follows all speckit conventions: version check, spec directory resolution with fallbacks, post-dispatch integrity check, and defensive response parsing. This ensures the extension behaves consistently with gap-audit and backtrace.

**Why this priority**: Required for the extension to function within the speckit ecosystem.

**Independent Test**: Invoke the command without arguments. Verify spec directory resolution falls back to `feature.json`, then prompts the user. Verify version check runs against speckit >= 0.5.2.

**Acceptance Scenarios**:

1. **Given** speckit version is below 0.5.2, **When** the command is invoked, **Then** it stops and reports the version requirement.

2. **Given** no spec directory argument and no `feature.json`, **When** the command is invoked, **Then** it prompts the user for the spec directory path.

3. **Given** the subagent modifies files in the repo during execution, **When** the subagent returns, **Then** the extension warns "Files were modified during testing (integrity violation). Review changes with `git diff`."

4. **Given** the subagent creates new untracked files in the repo (but does not modify existing files), **When** the subagent returns, **Then** the extension warns "Files were created during testing (integrity violation). Review changes with `git diff`."

5. **Given** the subagent returns non-JSON prose, **When** the extension parses the response, **Then** it attempts JSON extraction via bracket matching. If extraction fails or validation fails, it sets PARSE_FAILED and treats as empty array.

---

### Edge Cases

- What happens when `git diff` returns no changed files? The subagent should still run (it can test based on spec alone) but findings may be limited without implementation context.
- What happens when the subagent's tests fail to compile or run? The subagent should report this as a finding (the implementation may have issues), not silently swallow the error.
- What happens when `/tmp/exploratory-tests/` already exists from a previous run? The subagent should overwrite the existing directory contents.
- What happens when the project has no test suite to run for regression? The subagent should skip the regression check and note it was skipped.
- What happens when the subagent cannot write to `/tmp/exploratory-tests/` (permissions, disk full)? The subagent should report this as a finding rather than silently skipping test execution.

## Requirements

### Functional Requirements

- **FR-001**: The extension MUST parse `$ARGUMENTS` to extract spec directory path, `--base <branch>` flag (default: `main`), and `--output` flag. Parsing order: extract flags first, remaining token is spec directory path.

- **FR-002**: The extension MUST resolve the spec directory via three-step fallback: (1) argument path, (2) `.specify/feature.json` `feature_directory` value, (3) ask the user. The resolved path MUST exist and contain `spec.md`. When `--output` is passed, the spec directory MUST be resolved from argument or `feature.json` only. The ask-the-user fallback MUST NOT be used with `--output`. If the spec directory cannot be determined without user input and `--output` is passed, the extension MUST report an error and stop.

- **FR-003**: The extension MUST check speckit version from `.specify/init-options.json` against minimum `0.5.2`. Pre-release suffixes are stripped before comparison. If version cannot be determined, proceed with a warning.

- **FR-004**: The extension MUST read `spec.md` from the resolved spec directory and extract user stories, FRs, ACs, and SCs for inclusion in the subagent prompt.

- **FR-005**: The extension MUST run `git diff --name-only <base> -- .` to discover changed files, where `<base>` is the `--base` argument or `main`. The base branch MUST be validated with `git rev-parse --verify <base>`.

- **FR-006**: The extension MUST check for `defect-catalog.md` in the spec directory. If present, read the "Exploratory Testing Probes" section and include each pattern as an additional test vector in the subagent prompt. If absent, proceed without patterns.

- **FR-007**: The extension MUST dispatch a single `superpowers:code-reviewer` subagent with a prompt containing: spec content (user stories, FRs, ACs, SCs), changed file list, instruction to read changed files and understand the implementation, the full test vector checklist (7 categories: boundary value analysis, type edges, invariant violations, feature interaction, negative testing, mock fidelity, regression), instruction to write tests to `/tmp/exploratory-tests/`, instruction to execute the written exploratory tests and report results as findings, instruction to run the project test suite for regression, output format specification, the directive "Be adversarial. Try to break it. Do not assume the implementation is correct.", and (when defect catalog patterns are loaded) instruction to tag matching findings with `"pattern_match": "<canonical-name>"` from the defect catalog.

- **FR-008**: The subagent prompt MUST include the self-check gate: before reporting findings, re-examine each test for assertion mismatches (trace expected values through source code), uncaught exceptions (assert specific exception types), and parameter errors (correct function signatures). Low-confidence findings are marked as such rather than discarded.

- **FR-009**: The extension MUST parse the subagent response defensively: empty or whitespace-only response (set PARSE_FAILED, treat as empty array), valid JSON array (parse and proceed), empty array (no findings), non-JSON prose (extract JSON via bracket matching, validate first element has required ExploratoryFinding fields: severity, description, reproduction, expected, actual, spec_gap; if extraction fails OR validation fails, set PARSE_FAILED, treat as empty array), malformed JSON (set PARSE_FAILED, treat as empty array). First-element validation is a structural sanity check confirming the response contains well-formed findings. Individual finding schema compliance is not enforced at parse time.

- **FR-010**: The extension MUST add `"source": "exploratory"` to each finding object before writing to JSON.

- **FR-011**: When `--output` is passed, the extension MUST write findings to `<spec_dir>/.exploratory-findings.json`. Overwrite existing file. Write `[]` when no findings exist. Validate output JSON after writing. If JSON validation fails after writing, warn: "Output file written but JSON validation failed. The file may be corrupt. Re-run the command." Do not delete the file. When `--output` is not passed, no findings file is written.

- **FR-012**: After the subagent returns, the extension MUST run `git diff --name-only HEAD` and `git ls-files --others --exclude-standard` to detect repo modifications. If tracked files were modified, warn: "Files were modified during testing (integrity violation). Review changes with `git diff`." If new untracked files were created in the repo, warn: "Files were created during testing (integrity violation). Review changes with `git status`."

- **FR-013**: The extension MUST present findings grouped by severity (critical first, then important, moderate, minor). Include `pattern_match` when present. When PARSE_FAILED is true, present: "Testing completed but the subagent did not return structured findings. Re-run the test." When `--output` was set, report the output file path.

- **FR-014**: Finding zero issues MUST be accepted as a valid outcome. The subagent prompt MUST instruct: do not manufacture low-severity findings to avoid reporting clean. A clean report after exhausting the checklist is more valuable than noise.

- **FR-015**: The subagent prompt MUST instruct: generate focused, high-confidence tests. Each test should target a specific behavior or edge case with a clear expected outcome. Do not generate exhaustive test matrices.

### Key Entities

- **ExploratoryFinding**: A single finding produced by the subagent. Fields: `severity` (critical/important/moderate/minor), `description` (what happened), `reproduction` (command or code snippet), `expected` (what should have happened), `actual` (what did happen), `spec_gap` (which FR/AC/SC should have caught this), `source` ("exploratory", injected by the extension), `pattern_match` (optional, canonical name from defect catalog).

## Success Criteria

### Measurable Outcomes

- **SC-001: Behavioral parity.** Every behavior in the source skill (SKILL.md at `/home/mleader/workspace/py/warhammer/wh40k/.claude/skills/sdd-exploratory-test/SKILL.md`) MUST be present in the extension command. Verified by the parity checklist below. Each checklist item passes when the extension command contains prose that would cause the LLM to produce the described behavior. Verification is by human review of the command markdown against the source SKILL.md.

- **SC-002: Speckit boilerplate present.** The command includes all standard speckit extension sections: version check, argument parsing with flag extraction, spec directory resolution (arg/feature.json/ask), post-dispatch integrity check, defensive response parsing, JSON validation after write.

- **SC-003: Source field convention.** Every finding in the output JSON contains `"source": "exploratory"`.

- **SC-004: No behavioral additions.** The extension does not add behaviors absent from the source skill, except for speckit boilerplate (version check, spec dir resolution fallbacks, integrity check, defensive parsing, dependency checking). Any addition MUST be traceable to a speckit convention or a production readiness improvement identified during review, not a feature decision.

- **SC-005: Post-dispatch integrity check correctness.** The integrity check detects both modified tracked files (`git diff --name-only` returns non-empty) and newly created untracked files (`git ls-files --others --exclude-standard` returns non-empty), and produces the correct warning for each case.

### Behavioral Parity Checklist

Verified at implementation end by diffing the source SKILL.md against the extension command:

| # | Source behavior | Extension FR | Pass |
|---|---|---|---|
| 1 | Reads spec.md for user stories, FRs, ACs, SCs | FR-004 | PASS |
| 2 | Runs `git diff --name-only <base> -- .` for changed files | FR-005 | PASS |
| 3 | Validates spec directory exists and contains spec.md | FR-002 | PASS |
| 4 | Validates base branch exists via `git rev-parse --verify` | FR-005 | PASS |
| 5 | `--output` controls whether findings file is written | FR-011 | PASS |
| 6 | Unambiguous spec dir required when `--output` passed | FR-002 | PASS |
| 7 | Subagent receives spec content + changed file list | FR-007 | PASS |
| 8 | Subagent instructed to read changed files and understand implementation | FR-007 | PASS |
| 9 | Subagent writes tests to `/tmp/exploratory-tests/` | FR-007 | PASS |
| 10 | Subagent runs written tests against implementation | FR-007 | PASS* |
| 11 | Subagent runs project test suite for regression | FR-007 | PASS |
| 12 | Subagent instructed to be adversarial | FR-007 | PASS |
| 13 | Test vector: boundary value analysis | FR-007 | PASS |
| 14 | Test vector: type edges | FR-007 | PASS |
| 15 | Test vector: invariant violations | FR-007 | PASS |
| 16 | Test vector: feature interaction | FR-007 | PASS |
| 17 | Test vector: negative testing | FR-007 | PASS |
| 18 | Test vector: mock fidelity | FR-007 | PASS |
| 19 | Test vector: regression | FR-007 | PASS |
| 20 | Self-check gate: assertion mismatch detection | FR-008 | PASS |
| 21 | Self-check gate: uncaught exception check | FR-008 | PASS |
| 22 | Self-check gate: parameter error check | FR-008 | PASS |
| 23 | Low-confidence findings marked, not discarded | FR-008 | PASS |
| 24 | Findings schema: severity, description, reproduction, expected, actual, spec_gap | FR-009 | PASS |
| 25 | `defect-catalog.md` loaded when present | FR-006 | PASS |
| 26 | `defect-catalog.md` absence is not an error | FR-006 | PASS |
| 27 | Matching findings tagged with `pattern_match` | FR-006 | PASS |
| 28 | Output file is `.exploratory-findings.json` | FR-011 | PASS |
| 29 | Empty array written when no findings and `--output` passed | FR-011 | PASS |
| 30 | "No issues found." when no findings | FR-013 | PASS |
| 31 | Clean report accepted without manufacturing findings | FR-014 | PASS |
| 32 | Focused tests preferred over exhaustive matrices | FR-015 | PASS |
| 33 | Findings read from subagent response text, not test output files | FR-009 | PASS |

*Item 10: The source SKILL.md instructs the subagent to "write exploratory tests" and describes them as "reproducible evidence artifacts" (line 85). The extension makes test execution explicit ("Execute the written exploratory tests and report results as findings"). This is an elaboration of implicit source behavior, not a behavioral addition. The source clearly intends test execution (writing tests without running them would be pointless), but does not state it in those words.

### Spec Edge Case Coverage Checklist

Behaviors required by the spec's Edge Cases section, verified against the command file:

| # | Edge case | Command section | Pass |
|---|---|---|---|
| E1 | No changed files: subagent proceeds with spec-only testing | Section 7 (Changed Files, empty case) | PASS |
| E2 | Tests fail to compile/run: reported as finding | Section 7 (Preamble, rule 5) | PASS |
| E3 | /tmp/exploratory-tests/ already exists: overwritten | Section 7 (Preamble, rule 2) | PASS |
| E4 | No test suite discoverable: regression check skipped | Section 7 (Preamble, rule 4) | PASS |
| E5 | /tmp not writable: reported as finding | Section 7 (Preamble, rule 2) | PASS |

## Assumptions

- The `superpowers:code-reviewer` subagent type is available in the Claude Code environment (same assumption as gap-audit and backtrace extensions).
- speckit >= 0.5.2 is installed (provides extension infrastructure, feature.json, init-options.json).
- The project is a git repository (required for changed file discovery and post-dispatch integrity check).
- The source skill at `/home/mleader/workspace/py/warhammer/wh40k/.claude/skills/sdd-exploratory-test/SKILL.md` is the authoritative reference for behavioral parity.
- The extension registers no lifecycle hooks and requires no config-template.
- The extension structure follows the established pattern: `extension.yml`, `README.md`, `commands/speckit.exploratory-test.test.md`.
- The subagent has filesystem write access to `/tmp/` for writing exploratory tests.
- Test files written to `/tmp/exploratory-tests/` are not cleaned up by the extension. The source skill does not specify cleanup behavior. Users are responsible for managing `/tmp` contents. This matches the source skill's behavior of treating `/tmp` test files as reproducible evidence artifacts.
- The project has a discoverable test suite that the subagent can invoke. Test suite discovery is delegated to the subagent using standard project conventions (Makefile test target, pytest, go test, npm test, etc.). The spec does not prescribe a specific test runner or discovery algorithm. If no test suite is discoverable, the subagent skips the regression check and notes it was skipped.
- Downstream consumers (pattern collector, drive orchestrator) route findings by the `"source"` field, not by filename. The source field value `"exploratory"` is registered in the project-wide findings source namespace alongside `"audit"` (gap-audit) and `"verify"` (SC verify). New extensions producing findings must choose unique source values.
- The defect catalog (`defect-catalog.md`) is per-feature, located in the spec directory adjacent to `spec.md`. The gap-audit pattern catalog (`gap-patterns.md`) is project-wide, located at `specs/gap-patterns.md`. Both serve pattern-matching purposes but at different scopes. Unification of pattern catalogs is out of scope for this extension.
- Exploratory findings intentionally omit the `"scope"` field (unlike gap-audit findings which include both `"source"` and `"scope"`). The exploratory test extension has no scope concept (it always operates on a spec directory, not spec vs plan). Downstream consumers that expect a scope field (e.g., backtrace) should use their own scope argument as fallback when scope is absent.
