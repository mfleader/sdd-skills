# Answer Key: spec-with-gaps.md

Ground truth for measuring SC-001 (recall >= 80%) and SC-002 (FP rate <= 20%).
A single qualified reviewer (spec author or domain expert) determines pass/fail.

## Planted Gaps

| ID | Category | Description | Location | Classification |
|----|----------|-------------|----------|----------------|
| G01 | Orphan FRs | US1 describes "warning banner when widget data is stale (older than 5 minutes)" but no FR covers stale data warnings | US1 narrative, line 12 | blocking |
| G02 | Orphan FRs | US1 describes "every 30 seconds" refresh interval but FR-004 only says "refresh automatically" without the interval | US1 narrative vs FR-004 | blocking |
| G03 | Weak ACs | US1-AC1 "they see metrics" does not verify correctness of values, only presence | US1 AC1 | blocking |
| G04 | Weak ACs | US3-AC1 "a CSV file is downloaded" does not verify CSV content, format, or column structure | US3 AC1 | blocking |
| G05 | Weak ACs | US2-AC1 has no edge case ACs (empty result, clear filter, invalid category) | US2 AC1 | non-blocking |
| G06 | Unverifiable SCs | SC-001 "acceptable time" has no numeric threshold or measurement procedure | SC-001 | blocking |
| G07 | Unverifiable SCs | SC-004 "good user experience" is subjective and not mechanically verifiable | SC-004 | blocking |
| G08 | Cross-reference gaps | FR-005 (WidgetService API) has no corresponding SC | FR-005 vs SC section | non-blocking |
| G09 | Cross-reference gaps | SC-003 (10,000 concurrent users) has no corresponding FR for concurrency | SC-003 vs FR section | non-blocking |
| G10 | Cross-reference gaps | FR-006 (local timezone) has no corresponding SC | FR-006 vs SC section | non-blocking |
| G11 | Implicit assumptions | FR-005 references "WidgetService API" but Assumptions section does not document API contract, endpoint, or availability | FR-005 vs Assumptions | blocking |
| G12 | Implicit assumptions | FR-006 assumes system has access to user's timezone, not listed in Assumptions | FR-006 vs Assumptions | non-blocking |
| G13 | Naming collisions | "metrics" is used to mean both "dashboard metrics display" (FR-001) and "CSV file contents" (SC-002) without disambiguation | FR-001 vs SC-002 | non-blocking |
| G14 | Implicit behavior | No error handling defined: what happens when WidgetService API is unreachable? | Entire spec, no error section | blocking |
| G15 | Implicit behavior | No empty state defined: what happens when zero active widgets exist? | US1 assumes widgets exist | non-blocking |
| G16 | Implicit behavior | CSV export format (encoding, delimiter, header row, line endings) is unspecified | FR-003, US3 | blocking |

## Scoring

**Recall** = (gaps found by auditor that match an answer-key entry) / 16

**False positive rate** = (findings that do not match any answer-key entry) / (total findings)

A finding "matches" an answer-key entry when it identifies the same gap (same category and substantially the same issue), even if the description wording differs. Category assignment must match.

## Notes

- G02 could be classified as Orphan FRs (the interval is an unstated requirement) or Weak ACs (FR-004 is underspecified). The primary classification is Orphan FRs because the 30-second interval from the user story has no FR.
- G13 is a naming collision within the spec itself (not against the codebase) since this is a standalone test fixture with no accompanying codebase.
