# Brainstorm: Defect Catalog Unification

**Date:** 2026-05-01
**Status:** spec-created
**Spec:** specs/005-defect-catalog-unification/

## Problem Framing

The exploratory test skill has a feedback loop with external pattern collectors: findings feed into a collector, the collector aggregates recurring patterns into a catalog, and the skill reads the catalog to seed future probes. Spec 060 (wh40k) decoupled the skill from the wh40k-specific path `specs/sdd-patterns.md` by changing it to read `<spec_dir>/defect-catalog.md`. But the catalog is project-level accumulated knowledge, not per-spec, and nothing produces per-spec files. The feedback loop broke.

Meanwhile, gap-audit reads `specs/gap-patterns.md` with a different entry format (name/Category/Trigger). Both files originated from the same `sdd-patterns.md` in the wh40k project, split at different times during extension packaging. Deep research into the wh40k collector confirmed that both skills do the same thing with catalog data (embed raw section content into subagent prompts, tell the LLM to match findings against patterns), and both reference "trigger" in their matching instructions.

## Approaches Considered

### A: Fix exploratory path only (not chosen)

Change the exploratory test path from per-spec to project-level. Leave gap-audit and its separate file alone.

- Pros: Minimal change, fixes the broken loop immediately
- Cons: Two files, two schemas, two contracts for the same concept. Gap-audit's schema mismatch with the collector persists.

### B: Fix path + unify catalog (chosen)

Change both skills to read from a single `specs/defect-catalog.md` with consumer-specific sections and a converged entry format.

- Pros: One file, one schema, one contract. Both skills aligned. Collector produces one output. Existing fields preserved (Occurrences, Source, Example) plus new fields added (Category, Trigger).
- Cons: Breaking change for gap-audit (path change requires major version bump). Collector needs to produce two new fields.

### C: Configurable path

Add a `.specify/` config pointing to the catalog file. Flexible but adds complexity for a problem that fixed convention solves.

- Pros: Works for any project structure
- Cons: Complexity for no real use case. Violates Constitution Principle III (don't invent custom patterns).

## Decision

**Approach B: Fix path and unify catalog.** Research confirmed the schemas are accidentally different (built at different times), not intentionally different. Both subagent prompts reference "trigger." The converged format is a union of both schemas (nothing lost, Category and Trigger added). The collector already has the data needed to produce the new fields.

Key decisions:
- **File location**: `specs/defect-catalog.md` (fixed conventional path)
- **Entry format**: Union of both schemas. Required: canonical-name, Category, Trigger. Optional: Occurrences, Source, Also known as, Remediation, Example.
- **Frontmatter**: TOML. Metadata for tooling, not parsed by skills.
- **Breaking change**: Gap-audit path change requires major version bump. Clean break, no deprecation fallback.
- **Producer**: Externally produced. No generic collector ships with speckit.

## Open Threads

- Backtrace cannot consume exploratory findings due to schema incompatibility (GapFinding vs ExploratoryFinding). Carried forward from brainstorm #03.
- Should sdd-skills itself ship a `specs/defect-catalog.md` example/template, or does the catalog only exist in consuming projects?
