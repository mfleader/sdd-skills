+++
version = 1
last_updated = "2026-05-01"
specs_analyzed = []
pattern_count = 3
+++

# Defect Catalog

## Gap Audit Patterns

Known patterns the auditor should check for.

### missing-condition-false-path

**Category**: Weak ACs
**Trigger**: For every conditional FR (if X then Y), check that an AC exists for the condition-false path. When a spec defines behavior for a condition being true but does not specify what happens when the condition is false, the AC is incomplete.

### missing-derived-fr

**Category**: Orphan FRs
**Trigger**: User story describes behavior but no FR explicitly requires it. When a user story's acceptance scenario implies a system capability that is not captured as a standalone functional requirement, the behavior may be lost during implementation.

### untestable-threshold

**Category**: Unverifiable SCs
**Trigger**: A success criterion states a numeric threshold (percentage, count, latency) but does not define the measurement procedure, sample size, or test conditions. The threshold cannot be verified without a defined measurement method.

## Exploratory Testing Probes

Known design limitations the exploratory tester should probe.
