# sdd-skills

Extensions for Specification-Driven Development (SDD) built on [speckit](https://github.com/github/spec-kit), a CLI framework for AI agent workflows. SDD treats specifications as the primary artifact: code is derived from specs, and automated gates verify that code stays aligned with the spec at every stage. Everything runs inside Claude Code.

## Prerequisites

- [speckit](https://github.com/github/spec-kit) (>= 0.5.2)
- [Claude Code](https://claude.ai/claude-code) CLI

## Quick Start

```bash
# Clone the repo
git clone git@github.com:mfleader/sdd-skills.git
cd sdd-skills

# The repo is already initialized. Start with a brainstorm:
/speckit-spex-brainstorm
```

Or go fully autonomous from an existing brainstorm document:

```bash
/speckit-spex-ship brainstorm/001-my-feature.md --ask smart
```

## The SDD Workflow

Every feature follows the same lifecycle. Each stage produces artifacts that feed the next.

```
brainstorm ─> specify ─> clarify ─> plan ─> tasks ─> implement ─> review ─> verify
```

| Stage | Command | What it does | Artifacts produced |
|-------|---------|-------------|-------------------|
| Brainstorm | `/speckit-spex-brainstorm` | Explore an idea, refine scope | `brainstorm/NN-topic.md` |
| Specify | `/speckit-specify` | Create the feature spec | `spec.md`, `checklists/requirements.md` |
| Clarify | `/speckit-clarify` | Resolve ambiguities in the spec | Updated `spec.md` |
| Plan | `/speckit-plan` | Design the technical approach | `plan.md`, `research.md`, `data-model.md`, `contracts/` |
| Tasks | `/speckit-tasks` | Break the plan into actionable tasks | `tasks.md` |
| Implement | `/speckit-implement` | Execute tasks phase by phase | Source code |
| Review | `/speckit-spex-gates-review-code` | Check code against spec compliance | `REVIEW-CODE.md` |
| Verify | `/speckit-spex-gates-verify` | Final verification gate | Verification report |

Quality gates run automatically between stages (spec review after specify, plan review after tasks, code review after implement). You don't need to invoke them manually unless you want to re-run one.

### Autonomous Mode

`/speckit-spex-ship` chains all stages into a single autonomous pipeline. State is tracked in `.specify/.spex-state`, which enables resuming interrupted runs.

```bash
# Full autopilot from brainstorm to verified code
/speckit-spex-ship brainstorm/001-my-feature.md --ask smart

# Options:
#   --ask always   Pause on every finding (full human oversight)
#   --ask smart    Auto-fix clear issues, pause on ambiguous ones (default)
#   --ask never    Auto-fix everything, pause only on blockers
#   --create-pr    Auto-create PR after completion
#   --resume       Resume an interrupted pipeline
#   --start-from   Start from a specific stage (specify, clarify, review-spec, plan, tasks, review-plan, implement, review-code, verify)
```

## Extensions

The project ships 8 extensions. All are active by default.

| Extension | Description | Key Commands |
|-----------|-------------|-------------|
| **spex** | Core SDD workflow orchestration | `brainstorm`, `ship`, `help`, `init`, `evolve`, `extensions` |
| **spex-gates** | Quality gates at each workflow stage | `review-spec`, `review-plan`, `review-code`, `verify`, `stamp` |
| **spex-deep-review** | Multi-agent code review (5 specialized agents) with autonomous fix loop | `review` |
| **spex-teams** | Parallel implementation via agent teams | `orchestrate`, `research`, `implement` |
| **spex-worktrees** | Git worktree isolation for feature work | `manage` |
| **git** | Branch creation, auto-commits, validation | `initialize`, `feature`, `commit`, `validate`, `remote` |
| **gap-audit** | Adversarial gap auditor for specs and plans | `audit` |
| **backtrace** | Trace gap-audit findings back to spec gaps and propose additions | `trace` |

### Managing Extensions

```bash
# List installed extensions
/speckit-spex-extensions

# Enable/disable
specify extension disable spex-deep-review
specify extension enable spex-deep-review

# Install from git
specify extension add gap-audit --from <git-url>
specify extension add backtrace --from <git-url>
```

## Gap Audit

The gap audit extension dispatches an adversarial subagent to find gaps the standard review missed. It supports two scopes:

```bash
# Audit a spec for orphan FRs, weak ACs, unverifiable SCs, implicit assumptions, etc.
/speckit-gap-audit-audit spec

# Audit plan + tasks for missing contract tests, integration gaps, edge case coverage
/speckit-gap-audit-audit plan

# Save findings to JSON
/speckit-gap-audit-audit spec --output
```

The auditor applies 8 false positive filters before reporting and groups findings as blocking or non-blocking. See `.specify/extensions/gap-audit/README.md` for details.

## Backtrace

The backtrace extension closes the loop between finding gaps and fixing them. It traces gap-audit findings back to the spec artifacts that should have caught them, proposes additions, gets adversarial auditor approval, and applies approved changes.

```bash
# First, run gap-audit with --output to persist findings
/speckit-gap-audit-audit spec --output

# Then run backtrace to trace and fix the gaps
/speckit-backtrace-trace spec

# For plan-scope (traces against spec + plan + tasks)
/speckit-gap-audit-audit plan --output
/speckit-backtrace-trace plan
```

After applying additions, backtrace automatically invokes follow-up reviews (review-spec, review-plan, analyze) if spex-gates is installed. See `.specify/extensions/backtrace/README.md` for details.

## Utility Commands

| Command | What it does |
|---------|-------------|
| `/speckit-analyze` | Cross-artifact consistency check (spec vs plan vs tasks) |
| `/speckit-checklist` | Generate quality checklists ("unit tests for requirements") |
| `/speckit-constitution` | Define or update project governance principles |
| `/speckit-taskstoissues` | Sync tasks.md to GitHub Issues |
| `/speckit-spex-help` | Quick reference for all commands |

## Project Structure

```
sdd-skills/
├── .specify/
│   ├── extensions/          # 8 installed extensions
│   │   ├── backtrace/       # Trace findings back to spec gaps
│   │   ├── gap-audit/       # Adversarial gap auditor
│   │   ├── git/             # Git workflow automation
│   │   ├── spex/            # Core SDD workflow
│   │   ├── spex-deep-review/# Multi-agent code review
│   │   ├── spex-gates/      # Quality gates
│   │   ├── spex-teams/      # Parallel agent teams
│   │   └── spex-worktrees/  # Git worktree isolation
│   ├── extensions.yml       # Extension registry and hook configuration
│   ├── memory/
│   │   └── constitution.md  # Synced from specs/constitution.md for agent context
│   └── templates/           # Spec, plan, task templates
├── .claude/
│   └── skills/              # 35 Claude Code skills
├── specs/
│   ├── constitution.md      # Project governance
│   ├── gap-patterns.md      # Recurring gap audit patterns
│   └── NNN-feature-name/    # Feature spec directories
│       ├── spec.md
│       ├── plan.md
│       ├── tasks.md
│       ├── research.md
│       ├── data-model.md
│       └── contracts/
└── brainstorm/              # Brainstorm session documents
```

## Constitution

The project follows 8 core principles defined in `specs/constitution.md`:

1. **Correctness of Findings.** Precision over recall. Wrong findings corrupt decision-making.
2. **Evidence-Based Claims.** Every finding cites specific evidence (file, line, quote).
3. **Speckit-Native.** Follow speckit conventions. Don't invent custom patterns.
4. **One Code Path Per Operation.** Call existing speckit commands. Don't reimplement.
5. **Precise Skill Instructions.** Imperative directives with explicit output formats.
6. **Architectural Decisions Require Explicit Approval.** Design choices with multiple valid approaches get approved first.
7. **Flag Uncertainty.** Distinguish certain from uncertain claims.
8. **Conventional Commits.** All commits follow [Conventional Commits v1.0.0](https://www.conventionalcommits.org/).

## Contributing

### Branching

Feature branches use sequential numbering: `001-feature-name`, `002-another-feature`.

```bash
# Create a feature branch (handled by the git extension)
/speckit-git-feature
```

### Commits

All commits follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat(gap-audit): implement adversarial gap audit extension
fix(spex-gates): correct review-code compliance scoring
docs: update README with workflow overview
```

### Development Workflow

1. Start with `/speckit-spex-brainstorm` to refine the idea
2. The workflow guides you through specify, plan, tasks, implement
3. Quality gates run automatically. Fix any blocking findings before proceeding.
4. Commit with `git commit -s -S` (signed, GPG-signed)

## License

Apache 2.0. See [LICENSE](LICENSE).
