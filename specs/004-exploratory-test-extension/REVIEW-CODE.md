# Code Review: Exploratory Test Extension

## Code Review Guide (30 minutes)

> This section guides a code reviewer through the implementation changes,
> focusing on high-level questions that need human judgment.

**Changed files:** 3 files created (extension.yml, README.md, command markdown), 2 spec files updated (spec.md parity checklist, tasks.md completion marks)

### Understanding the changes (8 min)

- Start with `extension.yml`: Minimal metadata file. Confirm the command name `speckit.exploratory-test.test` follows the convention established by gap-audit (`speckit.gap-audit.audit`) and backtrace (`speckit.backtrace.trace`).
- Then the [command file](../../.specify/extensions/exploratory-test/commands/speckit.exploratory-test.test.md): This is the entire implementation. 12 sections, each modeled after a corresponding gap-audit section. Read sections 1-6 (boilerplate and input gathering) first, then section 7 (prompt construction, the largest section), then sections 8-12 (dispatch, integrity check, and output).
- Question: Does the 12-section structure (gap-audit has 10, this adds Section 5 for git diff and splits dispatch from integrity check) feel like the right decomposition, or should changed file discovery be folded into artifact loading?

### Key decisions that need your eyes (12 min)

**Prompt construction verbosity** (`commands/speckit.exploratory-test.test.md` Section 7, relates to [FR-007](spec.md))

The test vector checklist and self-check gate are transplanted verbatim from the source SKILL.md, wrapped in the speckit artifact-tag convention. The prompt is long (7 subsections) but each maps to a specific FR. [SC-004](spec.md) constrains against behavioral additions.
- Question: Is the prompt decomposition clear enough that an LLM executing it would follow all instructions, or would consolidating related sections (e.g., merging the testing approach into the preamble) improve adherence?

**Empty changed file handling** (`commands/speckit.exploratory-test.test.md` Section 7 changed files, relates to [FR-005](spec.md))

When `git diff` returns no files, the subagent is told to "proceed with spec-only testing." This is a degraded mode: the subagent has a spec to test against but no implementation files to read.
- Question: Is the "spec-only testing" instruction specific enough for the subagent to know what to do, or would it benefit from more concrete guidance about what spec-only testing means in practice?

**Defect catalog section name** (`commands/speckit.exploratory-test.test.md` Section 6, relates to [FR-006](spec.md))

The extension looks for "Exploratory Testing Probes" as a section heading in `defect-catalog.md`. This is a hard-coded string. If users name the section differently, patterns won't load.
- Question: Is a hard-coded section name acceptable, or should we document the exact heading requirement more prominently in the README?

### Areas where I'm less certain (5 min)

- `commands/speckit.exploratory-test.test.md` Section 10 (Response Parsing): The bracket-matching JSON extraction from prose is described in prose instructions to an LLM. Whether the LLM executing this command will faithfully implement bracket matching (finding `[` at line start and its matching `]`) is uncertain. Gap-audit has the same approach, so this is at least consistent.
- `commands/speckit.exploratory-test.test.md` Section 7 (additionalProperties enforcement): The prompt says "Return ONLY the specified fields. Do not add additional fields." This is a soft constraint (LLM instruction) for what the [contract](contracts/exploratory-findings.json) defines as a hard constraint (`additionalProperties: false`). There is no post-parse validation that rejects extra fields.
- `commands/speckit.exploratory-test.test.md` Section 11 (jq validation): The jq validation checks parseability, not schema conformance. A syntactically valid JSON array with wrong field names would pass validation. This matches gap-audit's behavior but is worth noting.

### Deviations and risks (5 min)

- Deviation from [plan.md](plan.md): The command now has 12 sections (plan specified 11). Section 8 was split into Section 8 (Dispatch) and Section 9 (Post-Dispatch Integrity Check) per deep review findings. This is a structural improvement, not a behavioral change.
- Risk: The [backtrace integration constraint](plan.md) (backtrace parser cannot consume exploratory findings) is documented but not resolved. If a user runs backtrace with auto-detect globbing, `.exploratory-findings.json` will match and parsing will fail. This is accepted per [plan.md](plan.md) as a follow-up task on the backtrace extension.

---

## Deep Review Report

> Automated multi-perspective code review results. This section summarizes
> what was checked, what was found, and what remains for human review.

**Date:** 2026-04-30 | **Rounds:** 1/3 | **Gate:** PASS

### Review Agents

| Agent | Findings | Status |
|-------|----------|--------|
| Correctness | 3 | completed |
| Architecture & Idioms | 5 | completed |
| Security | 4 | completed |
| Production Readiness | 5 | completed |
| Test Quality | 4 | completed |
| CodeRabbit (external) | 0 | skipped (--no-external) |
| Copilot (external) | 0 | skipped (--no-external) |

### Findings Summary

| Severity | Found | Fixed | Remaining |
|----------|-------|-------|-----------|
| Critical | 0 | 0 | 0 |
| Important | 7 | 7 | 0 |
| Minor | 14 | 14 | 0 |

### What was fixed automatically

Fixed 21 findings in one round. Key improvements: quoted `$BASE_BRANCH` in bash templates (security), added `jq` and `git` dependency checks with clear error messages (production readiness), split dispatch from integrity check into separate sections (architecture), upgraded integrity check to `git diff --name-only HEAD` to catch staged changes (correctness), added `--base` value validation and ask-the-user decline handling (correctness), added explicit findings channel instruction (test quality), added low-confidence marking convention (test quality), added edge case coverage checklist to spec (test quality), fixed write failure handling in JSON persistence (production readiness), fixed unactionable PARSE_FAILED message (production readiness), aligned README command format with gap-audit convention (architecture).

### What still needs human attention

All Critical and Important findings were resolved. No Minor findings remain. No further review action needed, but reviewers may want to verify:

- Does the 12-section structure (up from plan's 11) make sense, or should the integrity check stay merged with dispatch?
- Is the `[LOW CONFIDENCE]` prefix convention for the description field sufficient, or should a proper `confidence` field be added to the schema?

### Recommendation

All findings addressed. Code is ready for human review with no known blockers. See [review-findings.md](review-findings.md) for details.
