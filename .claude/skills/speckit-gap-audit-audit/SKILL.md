---
name: speckit-gap-audit-audit
description: Adversarial gap audit for SDD artifacts — dispatches a prejudiced subagent to find gaps in specs and plans
compatibility: Requires spec-kit project structure with .specify/ directory
metadata:
  author: github-spec-kit
  source: gap-audit:commands/speckit.gap-audit.audit.md
user-invocable: true
disable-model-invocation: false
---

# SDD Gap Audit

## Overview

Dispatches an adversarial auditor subagent to find gaps the standard review step missed in SDD artifacts. Supports two audit scopes: `spec` (spec.md only) and `plan` (spec.md + plan.md + tasks.md).

## Usage

```
/speckit.gap-audit.audit <scope> [--output]
```

- `scope`: `spec` or `plan`
- `--output`: persist findings to JSON file in the spec directory

## Execution

Load and execute the extension command file:

1. Read `.specify/extensions/gap-audit/commands/speckit.gap-audit.audit.md`
2. Follow the instructions in the command file exactly
3. Pass through any arguments provided by the user via `$ARGUMENTS`

**$ARGUMENTS**: The text the user provided after the command name. Pass this value to the command file for argument parsing.
