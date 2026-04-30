# Review Guide: Backtrace Improvements

**Spec:** [spec.md](spec.md) | **Plan:** [plan.md](plan.md) | **Tasks:** [tasks.md](tasks.md)
**Generated:** 2026-04-30

---

## What This Spec Does

Backtrace takes gap-audit findings and traces them back to the spec items that should have caught them, then proposes additions. This improvement spec addresses three weaknesses in v0.1.0: the tracing sometimes targets the wrong spec layer (proposing an AC when the real gap is a missing FR), all proposed additions appear equally confident regardless of trace quality, and clustered additions that signal systemic issues go unnoticed.

**In scope:** Barrier enumeration walk (Section 6), trace certainty field on ProposedAddition (Sections 6/7/11), cluster warning in output (Section 11), version bump, data model update.

**Out of scope:** Replacing tracing categories with STPA-derived ones, iterative refinement, dual auditors, configurable thresholds, any gap-audit changes. See [Out of Scope](spec.md#out-of-scope) for the full list with rationale.

## Bigger Picture

This is the first improvement cycle for backtrace after its initial v0.1.0 release (specs/002-backtrace-extension). The three changes were selected from 17 candidates researched across computer science, safety engineering, and LLM agent design (see [improvement-research.md](improvement-research.md)). An adversarial auditor reviewed and rejected 6 of the 17, challenged 4, and accepted 3.

The barrier enumeration concept comes from nuclear safety's defense-in-depth / barrier analysis. Trace certainty implements [Constitution Principle VII](../../specs/constitution.md) (Flag Uncertainty). Cluster detection is a simplified Pareto analysis.

Future work (v0.3+) might include Toulmin-structured rationales and STPA-derived tracing categories, both of which were explored and deferred.

---

## Spec Review Guide (30 minutes)

> Focus your time on whether the barrier walk and certainty definitions will work in practice when an LLM executes them as markdown instructions.

### Understanding the approach (8 min)

Read [FR-018 through FR-020](spec.md#functional-requirements) for the barrier walk mechanics, then [FR-021 through FR-024](spec.md#functional-requirements) for trace certainty. Consider:

- The barrier walk is a 4-step checklist (FR → AC → SC → task). Is this ordering always correct, or are there spec structures where the hierarchy breaks down?
- FR-018 delegates relevance determination to "LLM judgment." Is this the right call for a skill that prioritizes correctness (Principle I), or should there be more structure?
- The barrier walk and tracing categories "operate together in a single pass" ([FR-020](spec.md#functional-requirements)). Is this distinction from "preliminary step" meaningful, or is it implementation semantics that shouldn't be in the spec?

### Key decisions that need your eyes (12 min)

**Relevance via LLM judgment** ([FR-018](spec.md#functional-requirements))

The barrier walk determines "relevance" through semantic association, not keyword matching. This is the only feasible approach for natural language spec items, but it means the walk's accuracy depends entirely on the LLM's reasoning quality. The [gap-audit adversarial auditor](improvement-research.md#i-001-barrier-enumeration-step) called this the strongest recommendation but noted it could be undercut by poor relevance matching.

- Does the definition in FR-018 ("the spec item's stated scope covers the behavior or domain described by the finding") give enough guidance, or is it too abstract?

**Three certainty levels** ([FR-022](spec.md#functional-requirements))

The spec defines `direct`, `plausible`, and `uncertain`. The [improvement research](improvement-research.md#i-002-categorical-trace-certainty) explains why numeric scores were rejected (LLM calibration is unreliable). But three categories is a design choice.

- Is `plausible` distinct enough from `direct` and `uncertain` to be useful, or will the LLM collapse most traces into `plausible` as a safe middle ground?

**Fixed cluster threshold of 3** ([FR-027](spec.md#functional-requirements))

The threshold is fixed at 3 with no configuration. At typical finding volumes (5-15), this seems reasonable. But gap-audit can produce 20+ findings on a large spec.

- At 20+ findings, is 3 too low (noisy) or still appropriate?

### Areas where I'm less certain (5 min)

- [FR-019](spec.md#functional-requirements): The interaction between barrier walk and existing tracing categories is the most complex part of this spec. "The walk determines the target layer, the categories determine new vs amended" is clean in theory, but I'm not confident the instruction text will be clear enough for consistent LLM execution. The instruction may need examples.

- [FR-024](spec.md#functional-requirements): The output format `[approve] FR-018: <summary> [direct]` puts certainty at the end in brackets. Is this visually distinct enough from the verdict brackets at the start, or will it read as noise?

- [Assumptions](spec.md#assumptions): The spec assumes ACs are associated with FRs "through document proximity and domain relevance." This is true for speckit-generated specs but may not hold for hand-written specs with different structures.

### Risks and open questions (5 min)

- If the barrier walk consistently assigns `direct` certainty (because the LLM is confident about its own reasoning), [US-002](spec.md#us-002-trace-certainty-surfaces-ambiguity-p1) delivers no value. Is there a way to validate that the LLM actually produces a distribution across the three levels?

- The cluster warning fires on applied additions only ([Edge Cases](spec.md#edge-cases)). If the auditor rejects 4 additions targeting the same FR, the developer gets no cluster signal. Should rejected additions also count toward clustering?

---
*Full context in linked [spec](spec.md) and [plan](plan.md).*
