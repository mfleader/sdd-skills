# sdd-skills

Custom extensions for [speckit](https://github.com/github/spec-kit), a CLI framework for Specification-Driven Development (SDD) with AI agent workflows.

## Prerequisites

- [speckit](https://github.com/github/spec-kit) (>= 0.5.2) initialized with [spex](https://github.com/github/spec-kit#spex) extensions (spex, spex-gates, git)
- [Claude Code](https://claude.ai/claude-code) CLI

## Where These Extensions Fit

These extensions plug into the [spex workflow](https://github.com/rhuss/cc-spex#the-workflow) as quality loops after the automatic review gates.

```mermaid
flowchart TD
    A["spec, plan, tasks"] --> B["review gate<br>(review-spec, review-plan, analyze)"]
    B --> C["gap-audit<br>find what reviews missed"]
    C -->|"findings JSON"| D["backtrace<br>trace gaps to spec, propose fixes"]
    D -->|"updated artifacts"| B
    B --> E["exploratory-test<br>exercise implementation"]
    E -->|"findings JSON"| D

    style C fill:#fdd,stroke:#b00,stroke-width:2px,color:#000
    style D fill:#ddf,stroke:#006,stroke-width:2px,color:#000
    style E fill:#fdf,stroke:#606,stroke-width:2px,color:#000
```

1. Write your spec, plan, and tasks. Review gates run automatically.
2. Run **gap-audit** to find gaps the standard reviews missed (orphan FRs, weak ACs, implicit assumptions, missing edge cases).
3. Run **backtrace** to trace each gap back to the spec item that should have caught it, propose additions with adversarial auditor approval, and apply approved changes.
4. After implementation, run **exploratory-test** to exercise the implementation beyond its success criteria and find edge cases the spec missed.
5. Run **backtrace** on exploratory findings to trace gaps back to the spec.
6. Review gates re-run on the updated artifacts. Repeat until clean.

## Extensions

This repo provides three extensions:

| Extension | Description | Command |
|-----------|-------------|---------|
| **gap-audit** | Adversarial gap auditor for specs and plans | `/speckit.gap-audit.audit` |
| **backtrace** | Trace findings back to spec gaps and propose additions | `/speckit.backtrace.trace` |
| **exploratory-test** | Adversarial tester that exercises implementations beyond success criteria | `/speckit.exploratory-test.test` |

### Installation

```bash
# Install all extensions from a local clone
specify extension add /path/to/sdd-skills/.specify/extensions/gap-audit
specify extension add /path/to/sdd-skills/.specify/extensions/backtrace
specify extension add /path/to/sdd-skills/.specify/extensions/exploratory-test
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

## Exploratory Test

Dispatches an adversarial subagent to exercise a feature beyond its success criteria, writing and running tests to find edge cases and bugs the spec missed.

```bash
# Run exploratory tests against an implementation
/speckit.exploratory-test.test specs/004-feature/

# Specify a base branch for changed file discovery
/speckit.exploratory-test.test specs/004-feature/ --base develop

# Save findings to JSON (consumed by backtrace)
/speckit.exploratory-test.test specs/004-feature/ --output
```

The tester applies 7 test vector categories (boundary value analysis, type edges, invariant violations, feature interaction, negative testing, mock fidelity, regression) with a self-check gate. See `.specify/extensions/exploratory-test/README.md` for details.

## Backtrace

Closes the loop between finding gaps and fixing them. Traces findings back to the spec artifacts that should have caught them, proposes additions, gets adversarial auditor approval, and applies approved changes.

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
│   │   ├── backtrace/          # Backtrace extension
│   │   ├── exploratory-test/   # Exploratory test extension
│   │   └── gap-audit/          # Gap audit extension
│   └── memory/
│       └── constitution.md  # Synced from specs/constitution.md
├── specs/
│   ├── constitution.md      # Project governance principles
│   ├── gap-patterns.md      # Recurring gap audit patterns
│   ├── 001-gap-audit-extension/
│   ├── 002-backtrace-extension/
│   └── 004-exploratory-test-extension/
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
