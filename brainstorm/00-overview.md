# Brainstorm Overview

Last updated: 2026-05-01

## Sessions

| # | Date | Topic | Status | Spec |
|---|------|-------|--------|------|
| 01 | 2026-04-27 | speckit-extension-packaging | parked | - |
| 02 | 2026-04-28 | backtrace-extension | spec-created | 002 |
| 03 | 2026-04-30 | exploratory-test-extension | spec-created | 004 |
| 04 | 2026-05-01 | defect-catalog-unification | spec-created | 005 |

## Open Threads

- Extension name undecided for packaging approach (from #01)
- Should `after_backtrace` hooks fire even when no additions were applied? (from #02)
- Gap-audit `--fix` flag needs its own spec (from #02)
- Backtrace cannot consume exploratory findings due to schema incompatibility (GapFinding vs ExploratoryFinding) (from #03, #04)
- Pattern catalog divergence resolved by spec 005 (from #03) — thread closed
- Should sdd-skills ship a `specs/defect-catalog.md` example/template? (from #04)

## Parked Ideas

- Speckit extension packaging (#01)
  Reason: Extension name undecided, user deferred
