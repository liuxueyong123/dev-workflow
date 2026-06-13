# Agent-Skills + Superpowers Dev Workflow Design

## Goal

Make `dev-workflow` explicitly compose Agent-Skills and Superpowers rather than becoming a broad compatibility fallback.

## Corrected Direction

The main value is the combination of two capability packs:

- Agent-Skills provides lifecycle quality gates: spec, plan, incremental implementation, tests, review, simplification, security, performance, documentation, CI/CD, git workflow, and shipping.
- Superpowers provides execution discipline: skill selection, brainstorming, written implementation plans, TDD, systematic debugging, worktree isolation, subagent-driven execution, verification before completion, and development-branch finishing.

Claude Code and Codex are supported entry points, but they are not the source of the workflow. The workflow should prefer both capability packs and stop when one is missing unless the user explicitly approves degraded execution.

## Pipeline

1. Intake and skill availability check.
2. Define with Agent-Skills `/spec` and Superpowers brainstorming.
3. Plan with Agent-Skills `/plan` and Superpowers writing-plans.
4. Isolate with Superpowers worktree guidance.
5. Build with Superpowers TDD and Agent-Skills incremental implementation.
6. Debug with Superpowers systematic debugging.
7. Review with Agent-Skills code-review-and-quality.
8. Simplify with Agent-Skills code-simplification.
9. Run security-and-hardening for sensitive changes.
10. Verify with Superpowers verification-before-completion.
11. Ship or hand off with Agent-Skills shipping and Superpowers branch completion.

## Approval Boundaries

Plan approval authorizes local execution only. Push, merge, PR creation, publishing, deployment, credential changes, third-party resource changes, paid jobs, and destructive data operations still require explicit user approval.

## Non-Goals

- Do not make a broad compatibility workflow the primary product.
- Do not treat Agent-Skills or Superpowers as optional decorations.
- Do not silently continue when either capability pack is missing.
