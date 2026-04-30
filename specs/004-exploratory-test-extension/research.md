# Research: Exploratory Test Extension

## Source Skill Analysis

**Decision**: Mechanical repackaging of source skill into speckit extension format.
**Rationale**: The source skill is already portable and well-structured. All behavioral decisions were resolved during brainstorming.
**Alternatives considered**: None. The spec explicitly constrains this to a repackaging (SC-004).

## Key Design Decisions (from brainstorming)

### Command Name: `speckit.exploratory-test.test`

**Decision**: Use `test` as the verb.
**Rationale**: Follows the established `<technique>.<technique-root-verb>` convention: gap-audit.audit, backtrace.trace, exploratory-test.test. Audited and confirmed.
**Alternatives considered**: `hunt` (breaks convention by mixing goal into verb slot), `explore` (redundant with "exploratory"), `run` (generic, adds no meaning).

### Schema: Keep exploratory-specific fields

**Decision**: Keep severity/description/reproduction/expected/actual/spec_gap schema.
**Rationale**: Each finding producer writes its own schema suited to its finding type. The `source` field is the routing key. The pattern collector and backtrace handle normalization downstream. Aligning schemas at the producer level is unnecessary and would lose execution-grounded fields.
**Alternatives considered**: Align with gap-audit schema (loses reproduction/expected/actual), hybrid (adds classification but redundant with severity).

### Test Mode: Active testing

**Decision**: Subagent writes tests to /tmp and runs them, plus runs the project test suite.
**Rationale**: This is the source skill's design and what differentiates it from gap-audit. Gap-audit reasons about spec quality statically; exploratory test exercises the implementation.
**Alternatives considered**: Report-only (loses execution grounding, overlaps with gap-audit), hybrid (active tests but no project test suite run).

### Scope Field: Intentionally omitted

**Decision**: Exploratory findings include only `"source": "exploratory"`, not `"scope"`.
**Rationale**: The extension has no scope concept (always operates on a spec directory). Downstream consumers (backtrace) fall back to their own scope argument when scope is absent.

## Section-by-Section Mapping

The command's 11 sections map to gap-audit's 10-section structure with one addition (Section 5: Changed File Discovery is new because gap-audit doesn't need git diff).

| Gap-audit section | Exploratory test section | Differences |
|---|---|---|
| 1. Version Check | 1. Version Check | Identical |
| 2. Argument Parsing | 2. Argument Parsing | Adds --base flag parsing |
| 3. Spec Dir Resolution | 3. Spec Dir Resolution | Adds --output constraint on ask-the-user |
| 4. Artifact Loading | 4. Artifact Loading | Spec-only (no plan/tasks scope) |
| 5. Gap-Patterns Loading | 6. Defect Catalog Loading | Different file name and section name |
| 6. Prompt Construction | 7. Prompt Construction | Different checklist, adds test vectors + self-check gate |
| 7. Subagent Dispatch | 8. Subagent Dispatch + Integrity Check | Identical dispatch pattern |
| 8. Response Parsing | 9. Response Parsing | Different required fields |
| 9. JSON Persistence | 10. JSON Persistence | source only (no scope), different filename |
| 10. Output Formatting | 11. Output Formatting | Groups by severity instead of classification |
| N/A | 5. Changed File Discovery | New section (git diff + base branch validation) |
