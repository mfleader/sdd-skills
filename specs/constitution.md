<!-- Sync Impact Report
Version change: 0.0.0 → 1.0.0
Added sections:
  - I. Correctness of Findings
  - II. Evidence-Based Claims
  - III. Speckit-Native
  - IV. One Code Path Per Operation
  - V. Precise Skill Instructions
  - VI. Architectural Decisions Require Explicit Approval
  - VII. Flag Uncertainty
  - VIII. Conventional Commits
  - Development Workflow
  - Governance
Removed sections: none (initial version)
Templates requiring updates: none (speckit-owned templates, not controlled by this repo)
Follow-up TODOs: none
-->

# SDD Skills Constitution

## Core Principles

### I. Correctness of Findings
An audit skill that reports false positives or misattributes behavior is worse than one that misses a finding. Precision over recall. Every trade-off decision uses this order: correctness of findings > coverage of findings > speed of execution. Wrong findings corrupt decision-making silently and erode trust in the tooling.

### II. Evidence-Based Claims
Every finding MUST cite specific evidence: file path, line number, quote, or artifact reference. No vague claims. No assertions from general knowledge without verification against the actual codebase or API. If a claim cannot be substantiated, weaken it to "I believe" or "could you verify" rather than stating it as fact. This principle applies to audit findings, review comments, and gap reports equally.

### III. Speckit-Native
Follow speckit extension conventions. Use `extension.yml` for metadata, `commands/*.md` for command implementations, and lifecycle hooks for workflow integration. Do not invent custom patterns when speckit has one. Do not build custom plugin infrastructure, discovery mechanisms, or configuration formats. The extension MUST be installable via `specify extension add`.

### IV. One Code Path Per Operation
If a speckit command already performs an operation, call it. Do not reimplement the logic in the skill. The review loop calls `speckit.spex-gates.review-spec`, it does not contain its own spec review logic. When encountering friction with an existing command, fix the integration or file an issue upstream. A parallel implementation that makes an existing command redundant is a violation.

### V. Precise Skill Instructions
Skill files are prompt engineering artifacts. Instructions MUST be imperative ("Do X"), not advisory ("Consider X"). Every skill MUST define its expected output format explicitly. Avoid ambiguous directives like "be thorough" or "review carefully." If a skill dispatches a subagent, the prompt MUST include the full checklist, classification scheme, and output constraints. Vague prompts produce vague findings, which violates Principle I.

### VI. Architectural Decisions Require Explicit Approval
Design choices with multiple valid approaches (command boundaries, hook points, finding classification schemes, dependency choices) MUST be presented as options with trade-offs and approved before implementation. No contributor (human or AI) should unilaterally resolve architectural ambiguity.

### VII. Flag Uncertainty
When a claim, term, or recommendation is uncertain, say so. Do not present uncertain or invented information with the same confidence as verified facts. This applies to the skills themselves (audit findings MUST distinguish certain from uncertain claims) and to the development process (design proposals MUST flag assumptions). Wrong answers stated confidently are worse than honest uncertainty.

### VIII. Conventional Commits
All commits follow the [Conventional Commits v1.0.0 specification](https://www.conventionalcommits.org/en/v1.0.0/#specification). The 16 rules in that specification apply verbatim. Project-specific supplements: concise subject lines, no body for small changes. Signed-off and GPG-signed. Breaking changes (renamed commands, changed finding schemas, removed hook points, changed extension dependencies) MUST use `!` before the colon (`type(scope)!: description`) or a `BREAKING CHANGE:` footer. Before committing, ask: "would existing installations or workflows that depend on this break?" If yes, it is a breaking change.

## Development Workflow

- Finding schema changes require a version bump in extension.yml
- New commands require corresponding hook registration decisions (lifecycle hook or invoke-only)
- PRs require review from at least one coworker once a second contributor joins the project

## Governance

This constitution supersedes all other development practices for this repository. Amendments require:

1. A description of the change and its rationale
2. Approval from project contributors
3. A version bump (MAJOR for principle removal/redefinition, MINOR for additions, PATCH for clarifications)
4. A migration plan if the change affects existing skills or workflows

All PRs and reviews MUST verify compliance with these principles.

**Version**: 1.0.0 | **Ratified**: 2026-04-27 | **Last Amended**: 2026-04-27
