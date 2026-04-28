# Brainstorm: Speckit Extension Packaging

**Date:** 2026-04-27
**Status:** parked

## Problem Framing

Two standalone Claude Code skills (sdd-gap-audit, sdd-artifact-review-loop) exist as local files on the user's filesystem. They need to be packaged for distribution to coworkers. The coworker who wrote spex chose speckit's extension system over building a custom plugin platform, so this extension should follow that pattern.

## Approaches Considered

### A: Single Speckit Extension (Chosen)
One extension with both skills. Declares speckit and spex-gates as dependencies. Installs via `specify extension add`.
- Pros: Single install, lifecycle hooks, catalog-discoverable, follows coworker's pattern
- Cons: Gap audit loses standalone portability (minor, since target users all have speckit)

### B: Two Separate Extensions
Split into sdd-review (review loop) and sdd-audit (gap audit), each with its own extension.yml.
- Pros: Gap audit stays portable, users install only what they need
- Cons: Two things to install, maintain, and version for 2 small skills

### C: Extension + Standalone Fallback
Speckit extension for primary distribution, plus raw SKILL.md files for non-speckit users.
- Pros: Maximum reach
- Cons: Two distribution paths to maintain, standalone review loop can't call speckit commands

## Decision

Approach A chosen. Single speckit extension. Name deferred.

## Open Threads
- Extension name undecided. User explicitly deferred this.
- Individual command names within the extension undecided.
- Whether gap audit should hook into speckit lifecycle or remain invoke-only.
- "Gap audit" may not be the right term for the adversarial review skill.
- Constitution needs to be created before resuming spec work.
