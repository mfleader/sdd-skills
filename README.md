# sdd-skills

Custom extensions for [speckit](https://github.com/github/spec-kit), a CLI framework for Specification-Driven Development (SDD) with AI agent workflows.

## Prerequisites

- [speckit](https://github.com/github/spec-kit) (>= 0.5.2) initialized with [spex](https://github.com/github/spec-kit#spex) extensions (spex, spex-gates, git)
- [Claude Code](https://claude.ai/claude-code) CLI

## Where These Extensions Fit

```mermaid
flowchart TD
    Specify["/speckit-specify"] --> ReviewSpec["review-spec<br>(auto)"]
    ReviewSpec --> GapSpec["/speckit-gap-audit-audit spec<br>Find spec gaps"]
    GapSpec -->|"findings JSON"| BacktraceSpec["/speckit-backtrace-trace spec<br>Trace & fix gaps"]
    BacktraceSpec -->|"updated spec"| ReviewSpec

    ReviewSpec -->|"spec clean"| Plan["/speckit-plan + /speckit-tasks"]
    Plan --> ReviewPlan["review-plan<br>(auto)"]
    ReviewPlan --> GapPlan["/speckit-gap-audit-audit plan<br>Find plan gaps"]
    GapPlan -->|"findings JSON"| BacktracePlan["/speckit-backtrace-trace plan<br>Trace & fix gaps"]
    BacktracePlan -->|"updated spec + tasks"| ReviewPlan

    ReviewPlan -->|"plan clean"| Implement["/speckit-implement"]
    Implement --> ReviewCode["review-code<br>(auto)"]

    style GapSpec fill:#f96,stroke:#333
    style GapPlan fill:#f96,stroke:#333
    style BacktraceSpec fill:#69f,stroke:#333
    style BacktracePlan fill:#69f,stroke:#333
```

**Gap audit** (orange) finds gaps after review gates pass. **Backtrace** (blue) traces those gaps back to the spec, proposes additions, and feeds the updated artifacts back through review. Both are optional, manual invocations that complement the automatic review gates.

## Extensions

This repo provides two extensions:

| Extension | Description | Command |
|-----------|-------------|---------|
| **gap-audit** | Adversarial gap auditor for specs and plans | `/speckit-gap-audit-audit` |
| **backtrace** | Trace gap-audit findings back to spec gaps and propose additions | `/speckit-backtrace-trace` |

### Installation

```bash
# Install both extensions from a local clone
specify extension add /path/to/sdd-skills/.specify/extensions/gap-audit
specify extension add /path/to/sdd-skills/.specify/extensions/backtrace
```

## Gap Audit

Dispatches an adversarial subagent to find gaps the standard review missed.

```bash
# Audit a spec for orphan FRs, weak ACs, unverifiable SCs, implicit assumptions, etc.
/speckit-gap-audit-audit spec

# Audit plan + tasks for missing contract tests, integration gaps, edge case coverage
/speckit-gap-audit-audit plan

# Save findings to JSON (consumed by backtrace)
/speckit-gap-audit-audit spec --output
```

The auditor applies 8 false positive filters and groups findings as blocking or non-blocking. See `.specify/extensions/gap-audit/README.md` for details.

## Backtrace

Closes the loop between finding gaps and fixing them. Traces gap-audit findings back to the spec artifacts that should have caught them, proposes additions, gets adversarial auditor approval, and applies approved changes.

```bash
# Run gap-audit with --output first to persist findings
/speckit-gap-audit-audit spec --output

# Then run backtrace to trace and fix the gaps
/speckit-backtrace-trace spec

# Plan-scope (traces against spec + plan + tasks)
/speckit-gap-audit-audit plan --output
/speckit-backtrace-trace plan
```

After applying additions, backtrace invokes follow-up reviews (review-spec, review-plan, analyze) automatically. See `.specify/extensions/backtrace/README.md` for details.

## Project Structure

```
sdd-skills/
├── .specify/
│   ├── extensions/
│   │   ├── backtrace/       # Backtrace extension
│   │   └── gap-audit/       # Gap audit extension
│   └── memory/
│       └── constitution.md  # Synced from specs/constitution.md
├── specs/
│   ├── constitution.md      # Project governance principles
│   ├── gap-patterns.md      # Recurring gap audit patterns
│   ├── 001-gap-audit-extension/
│   └── 002-backtrace-extension/
└── brainstorm/              # Brainstorm session documents
```

## Constitution

The project follows 8 core principles defined in `specs/constitution.md`:

1. **Correctness of Findings.** Precision over recall.
2. **Evidence-Based Claims.** Every finding cites specific evidence.
3. **Speckit-Native.** Follow speckit conventions.
4. **One Code Path Per Operation.** Call existing speckit commands.
5. **Precise Skill Instructions.** Imperative directives with explicit output formats.
6. **Architectural Decisions Require Explicit Approval.** Present options with trade-offs first.
7. **Flag Uncertainty.** Distinguish certain from uncertain claims.
8. **Conventional Commits.** All commits follow [Conventional Commits v1.0.0](https://www.conventionalcommits.org/).

## Contributing

Feature branches use sequential numbering: `001-feature-name`, `002-another-feature`. Commits follow [Conventional Commits](https://www.conventionalcommits.org/), signed and GPG-signed.

## License

Apache 2.0. See [LICENSE](LICENSE).
