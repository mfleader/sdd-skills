# Review Brief: Defect Catalog Unification

**Spec:** specs/005-defect-catalog-unification/spec.md
**Generated:** 2026-05-01

> Reviewer's guide to scope and key decisions. See full spec for details.

---

## Feature Overview

The exploratory test skill's defect catalog feedback loop is broken. Spec 060 (in the wh40k project) decoupled the skill from `specs/sdd-patterns.md` and pointed it at `<spec_dir>/defect-catalog.md`, but no per-spec catalog files exist because the catalog is project-level knowledge. Meanwhile, the gap-audit skill reads a separate file (`specs/gap-patterns.md`) with a different entry format. This spec unifies both into a single `specs/defect-catalog.md` with a converged entry format and consumer-specific sections, fixing the broken loop and aligning the two skills on one contract.

## Scope Boundaries

- **In scope:** Fix exploratory test catalog path, migrate gap-audit to same file, define the defect catalog contract (file location, TOML frontmatter, sections, entry format), update both extension READMEs
- **Out of scope:** Modifying the wh40k pattern collector, changing findings output schemas, building a generic collector, per-spec catalog files, unifying informational sections (Spec Generation Weaknesses, Git-Derived Patterns) with any skill
- **Why these boundaries:** The spec defines the consumer contract. The producer (wh40k collector) adapts separately. Findings schemas are output-side and unrelated to catalog input format.

## Critical Decisions

### Unified file vs separate files
- **Choice:** Single `specs/defect-catalog.md` with consumer-specific sections
- **Trade-off:** Both skills depend on one file. A malformed catalog affects both, but the contract is precise enough to prevent this and both skills gracefully degrade.
- **Feedback:** Is the graceful degradation approach sufficient, or should one skill's section failure be completely isolated from the other?

### Converged entry format (union of both schemas)
- **Choice:** All fields preserved: Category, Trigger, Occurrences, Source, Also known as, Remediation, Example. Category and Trigger are new required fields. Occurrences, Source, Also known as, Remediation, Example are optional.
- **Trade-off:** The collector must produce two new fields (Category, Trigger) that it doesn't currently generate. More work for the producer, but enables accurate pattern matching in both consumers.
- **Feedback:** Is requiring Category and Trigger as mandatory fields the right call, given the collector doesn't produce them yet?

## Areas of Potential Disagreement

### Breaking change for gap-audit
- **Decision:** FR-009 replaces `specs/gap-patterns.md` with `specs/defect-catalog.md`, requiring a major version bump
- **Why this might be controversial:** Existing projects with `gap-patterns.md` lose pattern support until they migrate
- **Alternative view:** Could support both paths during a deprecation period (read new file, fall back to old)
- **Seeking input on:** Is a clean break acceptable, or should we add a deprecation fallback?

### Asymmetric validation between skills
- **Decision:** Gap-audit validates Category/Trigger fields (FR-006). Exploratory test only checks non-emptiness (FR-007).
- **Why this might be controversial:** Inconsistent behavior between skills reading the same file
- **Alternative view:** Both skills could validate the same fields for consistency
- **Seeking input on:** Is the rationale (gap-audit needs structured matching, exploratory test uses prose injection) sufficient?

## Naming Decisions

| Item | Name | Context |
|------|------|---------|
| Catalog file | `specs/defect-catalog.md` | Fixed conventional path at project root |
| Gap-audit section | "Gap Audit Patterns" | H2 heading, exact match required |
| Exploratory section | "Exploratory Testing Probes" | H2 heading, exact match required |
| Pattern identifier | canonical-name | Lowercase hyphenated, used as `pattern_match` value |

## Open Questions

- [ ] Should sdd-skills itself ship a `specs/defect-catalog.md` example/template, or does the catalog only exist in consuming projects?

## Risk Areas

| Risk | Impact | Mitigation |
|------|--------|------------|
| wh40k collector not updated to produce new fields | Medium | Skills gracefully degrade without catalog. Contract is documented so collector team knows the target format. |
| Projects with gap-patterns.md break on upgrade | Medium | FR-009 flags as breaking change with major version bump. Migration is documented. |
| TOML frontmatter parsing confusion | Low | Skills explicitly do not parse frontmatter. It's metadata for external tooling only. |

---
*Share with reviewers before implementation.*
