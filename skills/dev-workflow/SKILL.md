---
name: dev-workflow
description: Use only when the user explicitly invokes dev-workflow, $dev-workflow, /dev-workflow, or dev-flow by name. Do not use for ordinary coding requests, defect fixes, cleanup work, sensitive changes, or generic requests to combine Agent-Skills and Superpowers unless dev-workflow/dev-flow is explicitly named.
---

# Dev Workflow

## Explicit Invocation Only

Use this skill only when the user explicitly invokes one of these names:

- `dev-workflow`
- `$dev-workflow`
- `/dev-workflow`
- `dev-flow`

Do not use this skill for ordinary coding requests, defect fixes, cleanup work, sensitive changes, or generic requests to use Agent-Skills and Superpowers unless the user explicitly names `dev-workflow` or `dev-flow`.

If the user does not explicitly name `dev-workflow` or `dev-flow`, use the appropriate individual skills instead of this orchestrator.

## Purpose

Coordinate **Agent-Skills** and **Superpowers** as one production development workflow.

This skill is not a broad compatibility layer. Claude Code and Codex can both run it, but the workflow is intentionally built around two capability packs:

- **Agent-Skills** sets the quality gates: spec, plan, review, simplify, security, performance, documentation, CI/CD, git workflow, and shipping.
- **Superpowers** enforces execution discipline: skill selection, brainstorming, written plans, TDD, systematic debugging, worktree isolation, subagent-driven development, verification before completion, and branch finishing.

## Capability Gate

At the start of every non-trivial task:

1. Check whether Agent-Skills is available.
2. Check whether Superpowers is available.
3. If either pack is missing, stop and tell the user what is missing.
4. Continue in degraded mode only if the user explicitly asks for it.

Do not silently replace Agent-Skills or Superpowers with a generic checklist.

## Phase Tracking (MANDATORY)

**Create a TodoWrite task for every phase before starting work.** Each phase task MUST be marked `completed` (or explicitly `deleted` with a skip reason) before the final report. A phase with a `pending` status at final-report time is a workflow violation.

Use exactly these task subjects:

| Task subject      | When to skip                                  |
| ----------------- | --------------------------------------------- |
| "Phase: Intake"   | Never                                         |
| "Phase: Define"   | Trivial change (see Scope Rules)              |
| "Phase: Plan"     | Trivial change                                |
| "Phase: Isolate"  | Greenfield project without existing branches  |
| "Phase: Build"    | Never                                         |
| "Phase: Debug"    | No failures encountered                       |
| "Phase: Review"   | Never                                         |
| "Phase: Security" | No security-gate triggers (see Security Gate) |
| "Phase: Simplify" | Never — this is mandatory after Review        |
| "Phase: Verify"   | Never                                         |
| "Phase: Ship"     | User has not requested merge/deploy/publish   |

**Simplify is the only phase that has no skip condition.** It always runs after Review.

## Workflow

```text
intake (capability check)
  -> define
  -> plan
  -> isolate
  -> build
  -> [debug, if needed]
  -> review ──┐
              ├── simplify -> verify -> ship or handoff
  -> security ─┘
```

Review and security are independent checks; run them in parallel when both are triggered.

Every arrow is a phase transition. Do not skip a phase without marking its task `deleted` with a reason. Do not enter Verify while Simplify is `pending`.

## Phase Ownership

| Phase               | Primary                                                     | Secondary                                    | Exit gate                                                                                                                                                                    |
| ------------------- | ----------------------------------------------------------- | -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Intake              | Capability check (see Capability Gate)                      | —                                            | Agent-Skills and Superpowers both confirmed, or degraded mode explicitly approved                                                                                            |
| Define              | Agent-Skills `spec-driven-development` or `/spec`           | Superpowers `brainstorming`                  | User approves the spec content; archive choice is recorded                                                                                                                    |
| Plan                | Agent-Skills `planning-and-task-breakdown` or `/plan`       | Superpowers `writing-plans`                  | User approves the task plan; plan is archived only if archive was chosen after spec approval                                                                                  |
| Isolate             | Superpowers `using-git-worktrees`                           | Agent-Skills `git-workflow-and-versioning`   | Work area is safe                                                                                                                                                            |
| Build               | Superpowers `test-driven-development`                       | Agent-Skills `incremental-implementation`    | Focused tests pass                                                                                                                                                           |
| Debug (conditional) | Superpowers `systematic-debugging`                          | Agent-Skills `debugging-and-error-recovery`  | Root cause fixed, regression test added                                                                                                                                      |
| Review ∥ Security   | Review: Agent-Skills `code-review-and-quality` or `/review` | Superpowers `requesting-code-review`         | All CRITICAL and HIGH findings fixed; remaining IMPORTANT and MINOR findings captured for Simplify                                                                           |
|                     | Security: Agent-Skills `security-and-hardening`             | Project audit commands                       | Security-sensitive findings addressed                                                                                                                                        |
| Simplify            | Agent-Skills `code-simplification` or `/code-simplify`      | Superpowers `verification-before-completion` | **Every review finding addressed** (fixed, deferred with comment, or rejected with reason). Duplication removed, dead code deleted, long functions split, behavior unchanged |
| Verify              | Superpowers `verification-before-completion`                | Agent-Skills `/test`                         | Fresh evidence collected                                                                                                                                                     |
| Ship                | Agent-Skills `shipping-and-launch` or `/ship`               | Superpowers `finishing-a-development-branch` | User-approved external action                                                                                                                                                |

## Scope Rules

| Change type                                                         | Workflow                                                                                                           |
| ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| Typo, formatting-only docs, metadata copy edit                      | Direct edit plus Superpowers verification                                                                          |
| Single-file behavior change                                         | Short Agent-Skills spec note, Superpowers TDD, focused Agent-Skills review                                         |
| Multi-file feature or refactor                                      | Full Agent-Skills spec and plan, Superpowers TDD execution, Agent-Skills review/simplify, Superpowers verification |
| Bug fix                                                             | Superpowers systematic debugging, regression test, TDD fix, Agent-Skills review                                    |
| Auth, payment, PII, secrets, permissions, public API, data deletion | Full workflow plus Agent-Skills security-and-hardening                                                             |
| Release, deployment, or production rollout                          | Agent-Skills shipping-and-launch plus Superpowers branch completion                                                |

## Define And Plan

For non-trivial work:

1. Use Agent-Skills `/spec` or `spec-driven-development` to define the work.
2. Use Superpowers `brainstorming` to resolve ambiguity.
3. Output the spec draft in the conversation and stop for user confirmation.
4. After the user approves the spec content, ask whether the user wants to archive the spec/plan documents.
5. If the user chooses archive, write the spec to `docs/<feature>/spec.md`. If the user chooses not to archive, do not write the spec, and do not write the later plan for this workflow run.
6. Use Agent-Skills `/plan` or `planning-and-task-breakdown` for lifecycle-level decomposition.
7. Use Superpowers `writing-plans` to produce exact implementation steps, file paths, commands, and expected outputs.
8. Output the plan draft in the conversation and stop for user confirmation.
9. If the user chose archive earlier, write the approved plan to `docs/<feature>/plan.md`. If the user chose not to archive, keep the plan in the conversation only.

Do not write implementation code before spec and plan approval unless the task is explicitly classified as trivial.

Spec approval authorizes only the archive-choice question and plan creation. Plan approval authorizes local execution only (see Approval Boundaries).

### Spec And Plan Document Rules

Do not archive spec or plan documents by default. Every spec or plan draft is first presented in the conversation for user confirmation. After the spec is approved, ask whether the user wants the workflow artifacts archived.

If the user chooses not to archive:

1. Do not create or update `docs/<feature>/spec.md`.
2. Do not create or update `docs/<feature>/plan.md`.
3. Continue the workflow using the confirmed conversation drafts.

If the user chooses archive, every spec or plan document created by Agent-Skills, Superpowers, or dev-workflow MUST follow these rules:

1. Write the document body in Chinese. Technical identifiers, file paths, command names, API names, and quoted source text may remain in their original language.
2. Store documents under `docs/<feature>/`, where `<feature>` is a short kebab-case feature or task name chosen from the user's request.
3. Use exactly `docs/<feature>/spec.md` for the confirmed spec.
4. Use exactly `docs/<feature>/plan.md` for the confirmed implementation plan.
5. If an upstream skill suggests another default path, override it with this repository rule.
6. If a document already exists for that feature, update the existing file immutably by writing the new complete version instead of creating dated duplicates.

## Execute

Use Superpowers TDD as the primary implementation discipline:

1. Write the failing test or regression case.
2. Run it and confirm the expected failure.
3. Implement the smallest change that passes.
4. Run the focused test.
5. Refactor only while tests stay green.

Use Agent-Skills `incremental-implementation` for thin vertical slices, feature flags, rollback-friendly changes, and production constraints.

When the plan contains independent tasks with disjoint write scopes, use Superpowers `subagent-driven-development` if the runtime allows subagents. Otherwise execute inline task by task.

### IMPORTANT: After Subagent-Driven Development

Subagent-driven development considers itself done when all plan tasks complete. **dev-workflow does NOT end there.** When subagent-driven-development finishes:

1. The Build phase is complete (all tasks done, tests pass).
2. **dev-workflow MUST now run Review, Simplify, and Verify.**
3. Subagent-driven-development's per-task reviews are partial input to Review — they do not replace the dev-workflow Review phase.
4. Aggregate all per-task review findings (spec compliance issues, code quality suggestions) into a single list. This list becomes the input for Simplify.

## Debug

This phase triggers only when builds fail, tests break, or unexpected behavior appears. For clean-path feature work, skip it.

For bugs, build failures, and unexpected behavior:

1. Use Superpowers `systematic-debugging` first.
2. Reproduce and locate root cause before any fix.
3. Add a regression test before implementation.
4. Use Agent-Skills `debugging-and-error-recovery` when build/test recovery or fallback design matters.

## Review (Phase 1 of 2)

After all Build tasks pass:

1. **Collect all per-task review findings.** If subagent-driven-development ran spec compliance and code quality reviews per task, aggregate every finding — CRITICAL, HIGH, IMPORTANT, MINOR — into one list. These are NOT resolved just because the subagent reviews said "approved." They are the raw material for the next phase.
2. Fix CRITICAL and HIGH findings now. Do not proceed to Simplify with open CRITICAL or HIGH issues.
3. Record IMPORTANT and MINOR findings. These feed directly into Simplify — do not drop them.
4. If security gate triggers (see Security Gate), run Agent-Skills `security-and-hardening` in parallel. Security findings that block shipping must be fixed before proceeding.
5. Mark the "Phase: Review" task `completed` only when: CRITICAL/HIGH findings are fixed, and remaining findings are documented for Simplify.

## Simplify (Phase 2 of 2 — MANDATORY)

**Simplify always follows Review. There is no skip condition.** Even if Review found zero issues, run a simplification pass over the new and modified code.

1. Read the aggregated review findings (IMPORTANT and MINOR issues from Review).
2. Run Agent-Skills `code-simplification` or `/code-simplify` on the changed files.
3. Address every finding:
   - **Fix it** — remove duplication, extract helpers, delete dead code, split long functions, improve names.
   - **Defer it** — add a `// TODO(simplify): ...` comment with a reason why now isn't the right time.
   - **Reject it** — only if the finding is factually wrong or the simplification would change behavior. Explain the rejection in the final report.
4. Re-run tests after simplification. All tests must still pass.
5. Mark the "Phase: Simplify" task `completed`.

**Simplify exit checklist (every item must be true):**

- [ ] Every IMPORTANT review finding has been addressed (fixed, deferred with TODO, or rejected with reason)
- [ ] Every MINOR review finding has been addressed or explicitly deferred
- [ ] No function over 50 lines added in this change (unless previously over 50 and explicitly deferred)
- [ ] No dead code or unused variables remain
- [ ] No duplicated logic blocks (same pattern appearing 2+ times) remain
- [ ] All tests pass after simplification

## Security Gate

Run Agent-Skills `security-and-hardening` when the change touches:

- Authentication, authorization, sessions, OAuth, password reset, or tokens.
- Payment, billing, webhooks, or financial data.
- PII, secrets, credentials, or user-generated content.
- Public API endpoints, database queries, file uploads, or data deletion.
- Dependency, CI, deployment, or permission changes.

High or critical security findings block shipping until fixed or explicitly accepted by the user with context.

## Verification

Before any completion claim:

1. Use Superpowers `verification-before-completion`.
2. Run fresh focused tests.
3. Run full tests, lint, typecheck, build, format, audit, or browser checks when relevant and available.
4. Read the output and report exact results.

No verification evidence means no completion claim.

## Completion Gate

**Before producing the final report**, check every phase task:

```
TodoWrite task status must be:
  Phase: Intake   — completed
  Phase: Define   — completed or deleted (with skip reason)
  Phase: Plan     — completed or deleted (with skip reason)
  Phase: Isolate  — completed or deleted (with skip reason)
  Phase: Build    — completed
  Phase: Debug    — completed or deleted (with skip reason)
  Phase: Review   — completed
  Phase: Security — completed or deleted (with skip reason)
  Phase: Simplify — completed (NEVER deleted)
  Phase: Verify   — completed
  Phase: Ship     — completed or deleted (with skip reason)
```

If any phase task is `pending`, stop. Do not produce the final report. Complete the pending phase first.

If a phase was genuinely not applicable (e.g., greenfield project with no worktree), mark it `deleted` with a short skip reason. But Simplify can never be deleted — it is always applicable.

## Approval Boundaries

Plan approval authorizes local execution only:

- local edits
- tests
- lint/typecheck/build
- review
- simplification
- local verification

Stop for explicit user approval before:

- creating commits that include unrelated or untracked user work
- pushing branches
- opening or updating PRs
- merging
- publishing packages or plugins
- deploying
- changing credentials or secrets
- modifying third-party resources
- destructive data changes

## Runtime Notes

### Claude Code

Prefer native slash commands and plugin skills:

- Superpowers: `superpowers@claude-plugins-official` or `superpowers@superpowers-marketplace`
- Agent-Skills: `agent-skills@addy-agent-skills`
- dev-workflow: standalone skill or local plugin

### Codex

Prefer installed Codex skills and plugins:

- Superpowers through the Codex plugin marketplace.
- Agent-Skills copied or installed into `.agents/skills` or `~/.agents/skills`.
- dev-workflow installed into `.agents/skills` or `~/.agents/skills`.

If the `$` skill selector shows prefixed names, use the names Codex exposes.

## Response Shape

Working update:

```text
Using dev-workflow. Agent-Skills will handle spec/plan/review/security gates; Superpowers will handle planning discipline, TDD, debugging, and verification.
```

Missing capability:

```text
Agent-Skills is available, but Superpowers is not. Install Superpowers first, or explicitly confirm degraded mode.
```

Phase progress (use sparingly — when switching between major phases):

```text
Phase status:
  ✅ Intake  ✅ Define  ✅ Plan  ✅ Build
  🔄 Review  ⏳ Simplify  ⏳ Verify  ⬚ Ship
```

Final report (only after Completion Gate passes):

```text
Changed: <files>
Agent-Skills gates: <spec/plan/review/simplify/security/ship status — each with resolution>
Superpowers gates: <TDD/debug/worktree/verification status — each with resolution>
Simplify: <summary of findings addressed, deferred, or rejected>
Verified: <commands and results>
Not done: <commit/push/PR/deploy if not performed>
```
