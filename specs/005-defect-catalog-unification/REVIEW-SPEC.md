# Spec Review: Defect Catalog Unification

**Spec:** specs/005-defect-catalog-unification/spec.md
**Date:** 2026-05-01
**Reviewer:** Claude (speckit-spex-gates-review-spec)

## Overall Assessment

**Status:** SOUND

**Summary:** The spec is well-structured with clear requirements, a precise contract definition, and good edge case coverage. Three issues were identified and resolved in revision: FR-009 added for deprecation/breaking change, validation asymmetry explained in FR-006.

## Completeness: 4/5

### Structure
- All required sections present
- User stories with acceptance scenarios, edge cases, requirements, success criteria, assumptions, out of scope
- No placeholder text

### Coverage
- All functional requirements defined and numbered
- Error cases covered through FR-004/FR-005/FR-006/FR-007 and edge cases
- Success criteria specified for all key outcomes

**Issues:**
1. SC-005 requires `specs/gap-patterns.md` to no longer be referenced, but no FR covers deprecation or removal of that file. Add an FR stating the old file path is replaced and projects should migrate, or at minimum that the gap-audit command no longer reads it.
2. No FR addresses what happens to the existing `specs/gap-patterns.md` file in this repo (sdd-skills). It currently has 3 manually-written patterns. Should those patterns be migrated to a `specs/defect-catalog.md` in this repo, or is this repo not expected to have a catalog (since it's the tooling repo, not a project that runs audits on itself)?

## Clarity: 5/5

### Language Quality
- Uses "MUST" consistently for requirements
- Contract definition is precise: exact file path, section names, field names, required vs optional
- No vague terms ("user-friendly", "handle appropriately", etc.)
- Field definitions include purpose and consumer context

**Ambiguities Found:**
- None identified.

## Implementability: 4/5

### Plan Generation
- Can generate implementation plan from this spec
- Two extension commands to modify (exploratory-test, gap-audit), two READMEs to update
- Dependencies identified (external collector adaptation is out of scope)
- Scope is manageable

**Issues:**
1. **Breaking change not flagged**: FR-002 changes gap-audit's catalog path from `specs/gap-patterns.md` to `specs/defect-catalog.md`. Per Constitution Principle VIII, changing what file a command reads is a behavioral change that could break existing installations. Projects that have `specs/gap-patterns.md` but not `specs/defect-catalog.md` will lose pattern support on upgrade. The spec should acknowledge this as a breaking change and note the version bump requirement in extension.yml.
2. FR-006 requires gap-audit to validate `name`, `Category`, `Trigger` fields. FR-007 only requires exploratory test to validate non-emptiness. This asymmetry is intentional (gap-audit does structured matching, exploratory test does prose injection), but the spec should state why the validation levels differ, since a reader might wonder why one skill validates fields and the other doesn't.

## Testability: 5/5

### Verification
- Acceptance scenarios use Given/When/Then format
- Each SC is verifiable by running the skills and checking behavior
- SC-005 is verifiable by grepping for `gap-patterns.md` in extension commands
- Edge cases define expected behavior for each scenario

**Issues:**
- None identified.

## Constitution Alignment

- **Principle III (Speckit-Native)**: Follows extension conventions. Both skills are speckit extensions reading a conventional file path. Compliant.
- **Principle IV (One Code Path)**: Unifying two files into one shared contract aligns with this principle. Compliant.
- **Principle V (Precise Skill Instructions)**: Contract defines exact field names, section names, and validation behavior. Compliant.
- **Principle VI (Architectural Decisions)**: Brainstorming session explored 3 approaches (fixed path, configurable, multi-level) and obtained user approval. Compliant.
- **Principle VIII (Conventional Commits)**: See implementability issue #1. The path change is a breaking change that needs acknowledgment in the spec.

**Violations:**
- Principle VIII: Breaking change not flagged. FR-002 changes the file path gap-audit reads. This should be documented as a breaking change requiring a version bump.

## Recommendations

### Critical (Must Fix Before Implementation)
- [x] ~~Add FR covering deprecation of `specs/gap-patterns.md` path~~ — FR-009 added
- [x] ~~Acknowledge FR-002 as a breaking change per Constitution Principle VIII~~ — FR-009 flags breaking change and version bump

### Important (Should Fix)
- [x] ~~Add a brief note explaining why FR-006 and FR-007 have different validation levels~~ — FR-006 now explains the two-part vs prose-based matching distinction

### Optional (Nice to Have)
- [ ] Clarify whether sdd-skills itself should have a `specs/defect-catalog.md` or if the catalog only exists in consuming projects

## Conclusion

The spec is sound after revision. The contract definition is precise. All critical and important recommendations have been addressed. One optional clarification remains (whether sdd-skills itself needs a catalog file), which does not block implementation.

**Ready for implementation:** Yes

**Next steps:**
1. Proceed to `/speckit-plan`
