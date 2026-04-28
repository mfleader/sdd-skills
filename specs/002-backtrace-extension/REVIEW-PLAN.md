# Review Guide: Backtrace Extension

**Spec:** [spec.md](spec.md) | **Plan:** [plan.md](plan.md) | **Tasks:** [tasks.md](tasks.md)
**Generated:** 2026-04-28

---

## What This Spec Does

Backtrace traces gap-audit findings backward to the spec items that should have caught them, proposes additions (new FRs, ACs, SCs, tasks), gets adversarial auditor approval, and applies the approved changes. It's packaged as a speckit extension following the gap-audit pattern. The "code" is a markdown command file that instructs Claude how to perform the tracing, not traditional source code.

**In scope:** Backtrace command, auditor-gated spec edits, follow-up review invocations, extension packaging, docs

**Out of scope:** Iteration/looping, JSON persistence, gap-audit `--fix` flag (backlogged), modifying gap-audit

## Bigger Picture

This is the second extension in the sdd-skills project. Gap-audit (spec 001) finds gaps. Backtrace (spec 002) fixes them. Together they form a find-and-fix cycle for SDD artifacts. The backlogged `--fix` flag on gap-audit would chain the two into a single invocation.

Backtrace was originally a standalone skill in a different project (warhammer/wh40k). This spec refactors it into the speckit extension format. The key change from the original: one pass only (no iteration loop), and follow-up reviews are direct invocations rather than hooks.

The hook approach changed during planning. The original spec assumed hooks could declare multiple follow-up commands. Research revealed speckit only allows one command per hook point per extension. This is documented in [research.md R-005](research.md#r-005-extensionyml-for-backtrace-revised-after-gap-audit).

---

## Spec Review Guide (30 minutes)

### Understanding the approach (8 min)

Read [Purpose](spec.md#purpose) and [FR-001 through FR-011](spec.md#functional-requirements) for the core flow. As you read, consider:

- Does the tracing logic in [FR-003](spec.md#functional-requirements) cover the right categories? Are there gap types that don't fit into "missing test coverage, missing integration scenario, implicit assumption, edge case"?
- The spec says additions target [spec.md or tasks.md](spec.md#functional-requirements) (FR-004). Should plan.md ever be directly edited, or is "new tasks referencing existing plan phases" sufficient?
- [FR-010](spec.md#functional-requirements) says one pass only. After using this in practice, will that feel like a limitation?

### Key decisions that need your eyes (12 min)

**Direct invocation instead of hooks** ([FR-012, FR-013](spec.md#functional-requirements), [research.md R-005](research.md#r-005-extensionyml-for-backtrace-revised-after-gap-audit))

The original spec assumed hook-based follow-ups. Research discovered speckit only allows one command per hook point per extension. The revised approach directly invokes review-spec, review-plan, and analyze from the command file, checking the extensions registry for availability first.
- Question: Is direct invocation acceptable, or should we pursue a single composite follow-up command registered as a hook?

**Auditor as single dispatch** ([FR-006](spec.md#functional-requirements))

All proposed additions are sent to the auditor in one prompt. This mirrors gap-audit's single-dispatch pattern.
- Question: For specs with many findings (10+), will a single dispatch produce reliable verdicts, or should we batch?

**Auto-detect findings input** ([FR-001](spec.md#functional-requirements))

Backtrace checks for `.sdd-findings-{scope}.json` first, then asks the user. This creates a convention dependency on gap-audit's output filename.
- Question: Is the filename convention stable enough to depend on?

**Scope terminology** ([FR-002](spec.md#functional-requirements))

Backtrace uses "scope" while gap-audit uses "focus" for the same `spec`/`plan` parameter. We chose to keep "scope" and align gap-audit later.
- Question: Should we align the terminology now to avoid confusion?

### Areas where I'm less certain (5 min)

- [FR-003](spec.md#functional-requirements): The tracing categories (missing test coverage, missing integration scenario, etc.) are guidance, not rules. In practice, the AI will need to exercise judgment about which category fits. I'm not sure the categories are exhaustive or that the spec provides enough structure to produce consistent tracing results.

- [FR-009](spec.md#functional-requirements): Reset trigger thresholds ("SC count change by 3+", "material scope expansion") are somewhat arbitrary. "Material scope expansion" in particular is subjective. How will the AI reliably detect this?

- [US-004](spec.md#us-004-follow-up-review-hooks-p2): The acceptance scenarios say hooks fire even when all additions are rejected. This means follow-up reviews run on unmodified artifacts, which is technically a no-op but adds token cost. Is that the right behavior?

### Risks and open questions (5 min)

- If the auditor subagent returns unreliable verdicts (approving bad additions or rejecting good ones), backtrace has no recourse within one pass. Is one pass really sufficient for production use, or will users quickly want iteration back?
- The [GapFinding schema](spec.md#key-entities) is consumed from gap-audit but never formally versioned. If gap-audit changes its schema, backtrace breaks silently. Should there be a schema version check?
- 28 tasks for a prompt-engineering project (one markdown command file + supporting files). Is the task granularity right, or are some tasks too fine-grained for what amounts to writing different sections of one document?

---
*Full context in linked [spec](spec.md) and [plan](plan.md).*
