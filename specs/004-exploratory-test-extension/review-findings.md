# Deep Review Findings

**Date:** 2026-04-30
**Branch:** 004-exploratory-test-extension
**Rounds:** 1
**Gate Outcome:** PASS
**Invocation:** quality-gate

## Summary

| Severity | Found | Fixed | Remaining |
|----------|-------|-------|-----------|
| Critical | 0 | 0 | 0 |
| Important | 7 | 7 | 0 |
| Minor | 14 | 14 | 0 |
| **Total** | **21** | **21** | **0** |

**Agents completed:** 5/5 (+ 0 external tools)
**Agents failed:** none
**External tools:** CodeRabbit skipped (--no-external), Copilot skipped (--no-external)

## Findings

### FINDING-1 (Security)
- **Severity:** Important
- **File:** commands/speckit.exploratory-test.test.md:86,98
- **Source:** security-agent
- **Resolution:** fixed (round 1)

Unquoted `$BASE_BRANCH` in bash templates. Fixed by quoting: `"$BASE_BRANCH"`.

### FINDING-2 (Test Quality)
- **Severity:** Important
- **File:** specs/004-exploratory-test-extension/spec.md:181
- **Source:** test-quality-agent
- **Resolution:** fixed (round 1)

Parity checklist item 10 marked as parity when it elaborates implicit behavior. Fixed by adding PASS* with footnote.

### FINDING-3 (Correctness)
- **Severity:** Important
- **File:** commands/speckit.exploratory-test.test.md (Section 9)
- **Source:** correctness-agent
- **Resolution:** fixed (round 1)

Integrity check used bare `git diff --name-only` missing staged changes. Fixed by using `git diff --name-only HEAD`.

### FINDING-4 (Architecture)
- **Severity:** Important
- **File:** commands/speckit.exploratory-test.test.md (Section 8)
- **Source:** architecture-agent
- **Resolution:** fixed (round 1)

Compound section heading "Dispatch + Integrity Check" bundled two concerns. Fixed by splitting into Section 8 (Dispatch) and Section 9 (Post-Dispatch Integrity Check), renumbering to 12 sections.

### FINDING-5 (Security)
- **Severity:** Important
- **File:** commands/speckit.exploratory-test.test.md:108
- **Source:** security-agent
- **Resolution:** fixed (round 1, partial)

Spec dir path in shell commands. The `<spec_dir>` placeholder in the defect catalog check is already within double quotes in the bash template. Quoting of `$BASE_BRANCH` was the primary injection vector and is now fixed.

### FINDING-6 (Production Readiness)
- **Severity:** Important
- **File:** commands/speckit.exploratory-test.test.md (Section 1)
- **Source:** production-readiness-agent
- **Resolution:** fixed (round 1)

`jq` and `git` dependencies not checked. Fixed by adding dependency checks at the start of Section 1 with clear error messages.

### FINDING-7 (Production Readiness)
- **Severity:** Important
- **File:** commands/speckit.exploratory-test.test.md (Section 5)
- **Source:** production-readiness-agent
- **Resolution:** fixed (round 1)

Git availability not checked, producing misleading errors. Fixed by adding `git rev-parse --is-inside-work-tree` check in Section 1.

### FINDING-8 (Correctness)
- **Severity:** Minor
- **File:** commands/speckit.exploratory-test.test.md (Section 4)
- **Source:** correctness-agent
- **Resolution:** fixed (round 1)

Section 4 said "extract user stories, FRs, ACs, SCs" but Section 7 embeds full spec content. Fixed by aligning Section 4 to say "store full content for inclusion in the subagent prompt."

### FINDING-9 (Correctness)
- **Severity:** Minor
- **File:** commands/speckit.exploratory-test.test.md (Section 2)
- **Source:** correctness-agent
- **Resolution:** fixed (round 1)

`--base` without a value not validated. Fixed by adding validation: "The --base flag requires a branch name argument."

### FINDING-10 (Architecture)
- **Severity:** Minor
- **File:** commands/speckit.exploratory-test.test.md (Section 3)
- **Source:** architecture-agent
- **Resolution:** fixed (round 1)

Ask-the-user fallback missing decline handling. Fixed by adding explicit decline/empty handling matching gap-audit convention.

### FINDING-11 (Architecture)
- **Severity:** Minor
- **File:** README.md
- **Source:** architecture-agent
- **Resolution:** fixed (round 1)

README used hyphen format for commands, gap-audit uses dot format. Fixed to use dot format for consistency.

### FINDING-12 (Production Readiness)
- **Severity:** Minor (mapped from Moderate)
- **File:** commands/speckit.exploratory-test.test.md (Section 11)
- **Source:** production-readiness-agent
- **Resolution:** fixed (round 1)

No write failure handling in JSON persistence. Fixed by adding write failure detection and error message.

### FINDING-13 (Production Readiness)
- **Severity:** Minor (mapped from Moderate)
- **File:** commands/speckit.exploratory-test.test.md (Section 9)
- **Source:** production-readiness-agent
- **Resolution:** fixed (round 1)

Integrity check silently swallowed git failures. Fixed by adding git command failure detection with distinct warning.

### FINDING-14 (Production Readiness)
- **Severity:** Minor
- **File:** commands/speckit.exploratory-test.test.md (Section 12)
- **Source:** production-readiness-agent
- **Resolution:** fixed (round 1)

PARSE_FAILED message suggested "review subagent output manually" which is unactionable. Fixed to say "Re-run the test."

### FINDING-15 (Test Quality)
- **Severity:** Minor (mapped from Moderate)
- **File:** commands/speckit.exploratory-test.test.md (Section 7)
- **Source:** test-quality-agent
- **Resolution:** fixed (round 1)

No mechanism to express low-confidence findings. Fixed by adding convention: prefix description field with "[LOW CONFIDENCE] ".

### FINDING-16 (Test Quality)
- **Severity:** Minor (mapped from Moderate)
- **File:** specs/004-exploratory-test-extension/spec.md
- **Source:** test-quality-agent
- **Resolution:** fixed (round 1)

Spec edge cases not covered by parity checklist. Fixed by adding "Spec Edge Case Coverage Checklist" with 5 items (E1-E5).

### FINDING-17 (Test Quality)
- **Severity:** Minor
- **File:** commands/speckit.exploratory-test.test.md (Section 10)
- **Source:** test-quality-agent
- **Resolution:** fixed (round 1)

No explicit negative instruction for findings channel. Fixed by adding explicit "Parse findings ONLY from response text" instruction.

### FINDING-18-21 (Architecture, Security)
- **Severity:** Minor
- **Resolution:** accepted or not applicable

Remaining minor findings: Section 7 navigability (bold labels match gap-audit convention, accepted), explicit "do NOT include source" instruction (kept as defensive improvement), prompt injection via artifact tags (pre-existing pattern, accepted), path traversal (addressed by existing path validation).
