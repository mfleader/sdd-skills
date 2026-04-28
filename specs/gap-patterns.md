# Gap Audit Patterns

Recurring gap types cataloged from previous audits. The gap auditor uses these to tag matching findings with the canonical pattern name.

## missing-condition-false-path
**Category**: Weak ACs
**Trigger**: For every conditional FR (if X then Y), check that an AC exists for the condition-false path. When a spec defines behavior for a condition being true but does not specify what happens when the condition is false, the AC is incomplete.

## missing-derived-fr
**Category**: Orphan FRs
**Trigger**: User story describes behavior but no FR explicitly requires it. When a user story's acceptance scenario implies a system capability that is not captured as a standalone functional requirement, the behavior may be lost during implementation.

## untestable-threshold
**Category**: Unverifiable SCs
**Trigger**: A success criterion states a numeric threshold (percentage, count, latency) but does not define the measurement procedure, sample size, or test conditions. The threshold cannot be verified without a defined measurement method.
