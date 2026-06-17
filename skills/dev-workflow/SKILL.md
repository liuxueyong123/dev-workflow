---
name: dev-workflow
description: MUST use when the user explicitly asks to use, run, invoke, follow, or apply dev-workflow, $dev-workflow, /dev-workflow, or dev-flow by name, including routine code edits, defect repair, restructuring, cleanup, and sensitive work. Do not use when the user does not name dev-workflow/dev-flow, negates use of it, or only asks to explain or edit the skill itself.
---

# Dev Workflow

Coordinates **Agent-Skills** (intent, spec, plan, review, simplify, security, ship), standalone skills (`grill-me`, `grill-with-docs`), and **Superpowers** (brainstorming, TDD, debugging, worktrees, verification) as one production workflow.

## Explicit Invocation Only

Activate only on explicit invocation: `dev-workflow`, `$dev-workflow`, `/dev-workflow`, `dev-flow`, "use dev-workflow", "用 dev-workflow", or "按 dev-flow 处理". Once invoked, this skill is mandatory regardless of task type.

Do NOT activate on: "do not use dev-workflow", "explain dev-workflow", "update the dev-workflow skill", or ordinary coding requests without naming dev-workflow/dev-flow.

## Invocation Compliance Gate

On positive invocation:
1. Stop any ordinary coding, debugging, or review path.
2. Send the working update: `Using dev-workflow.`
3. Check Capability Gate, then create Phase Tracking tasks before editing files.
4. Complete or explicitly skip every required phase before the final report.
5. **Respect every Hard Gate.** Stop and wait for user confirmation. Never skip or bundle gates.

## Capability Gate

Check that Agent-Skills, Superpowers, and required standalone skills (`grill-me`, `grill-with-docs` for spec review) are available. If any are missing, stop and tell the user. Continue in degraded mode only if the user explicitly approves it.

## Phase Tracking (MANDATORY)

Create a TodoWrite task for every phase. Each must be `completed` or `deleted` (with reason) before the final report.

| Task subject       | Skip condition                               |
| ------------------ | -------------------------------------------- |
| "Phase: Intake"    | Never                                        |
| "Phase: Interview" | Trivial change only (see Scope Rules)        |
| "Phase: Define"    | Trivial change only                          |
| "Phase: Plan"      | Trivial change only                          |
| "Phase: Isolate"   | Greenfield project with no existing branches |
| "Phase: Build"     | Never                                        |
| "Phase: Debug"     | No failures encountered                      |
| "Phase: Review"    | Never                                        |
| "Phase: Security"  | No security-gate triggers                    |
| "Phase: Simplify"  | Never — mandatory after Review               |
| "Phase: Verify"    | Never                                        |
| "Phase: Ship"      | User has not requested merge/deploy/publish  |

## Hard Gates (MANDATORY STOP POINTS)

Five hard gates require explicit user confirmation before proceeding. **Skipping a gate is a workflow violation.** Output the draft, ask for confirmation, then STOP — the user's next message IS the gate response.

| Gate | Phase     | Trigger                 | Required Confirmation              |
| ---- | --------- | ----------------------- | ---------------------------------- |
| 1    | Interview | Intent summary output   | User approves intent summary       |
| 2    | Define    | Spec draft output       | User approves spec content         |
| 3    | Define    | Spec approved           | User chooses archive or not        |
| 4    | Plan      | Plan draft output       | User approves plan                 |
| 5    | Ship      | Before external action  | User explicitly requests it        |

**Protocol:** (1) Output the complete draft. (2) Ask a clear confirmation question. (3) **STOP. Do NOT proceed**, do NOT write code, do NOT invoke the next skill. (4) Wait for user response. (5) Only then continue.
**Common violations:** coding right after spec output; bundling spec approval + archive in one message; skipping archive; entering Define before Interview confidence ≥ 95%.

## Workflow

```text
intake → interview → define → plan → isolate → build → [debug] → review  ─┐
                                                                          ├── simplify → verify → ship
                                                                security ─┘
```
Review and security run in parallel when both are triggered. Never skip a phase without marking its task `deleted` with a reason.

## Phase Ownership

| Phase               | Primary                                                     | Secondary                                    | Exit gate                                                                                                                                                                    |
| ------------------- | ----------------------------------------------------------- | -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Intake              | Capability check (see Capability Gate)                      | —                                            | Agent-Skills, standalone skills, and Superpowers confirmed, or degraded mode explicitly approved                                                                             |
| Interview           | Agent-Skills `interview-me`                                 | Superpowers `brainstorming`                  | Confidence ≥ 95%; all GUESS markers resolved; intent summary output and approved by user (Gate 1)                                                                            |
| Define              | Agent-Skills `spec-driven-development`; standalone `grill-me`, `grill-with-docs`; `/spec` | Superpowers `brainstorming` | Spec written from approved intent summary; grill review passed; archive choice recorded                                                                                       |
| Plan                | Agent-Skills `planning-and-task-breakdown` or `/plan`       | Superpowers `writing-plans`                  | User approves the task plan; plan is archived only if archive was chosen after spec approval                                                                                 |
| Isolate             | Superpowers `using-git-worktrees`                           | Agent-Skills `git-workflow-and-versioning`   | Work area is safe                                                                                                                                                            |
| Build               | Superpowers `test-driven-development`; use `subagent-driven-development` when plan tasks are independent with disjoint write scopes | Agent-Skills `incremental-implementation` | Approved spec/plan implemented; focused tests pass                                                                                                                           |
| Debug (conditional) | Superpowers `systematic-debugging`                          | Agent-Skills `debugging-and-error-recovery`  | Root cause fixed, regression test added                                                                                                                                      |
| Review              | Review: Agent-Skills `code-review-and-quality` or `/review` | Superpowers `requesting-code-review`         | All CRITICAL and HIGH findings fixed; remaining IMPORTANT and MINOR findings captured for Simplify                                                                           |
| Security            | Security: Agent-Skills `security-and-hardening`             | Project audit commands                       | Security-sensitive findings addressed                                                                                                                                        |
| Simplify            | Agent-Skills `code-simplification` or `/code-simplify`      | Superpowers `verification-before-completion` | **Every review finding addressed** (fixed, deferred with comment, or rejected with reason). Duplication removed, dead code deleted, long functions split, behavior unchanged |
| Verify              | Superpowers `verification-before-completion`                | Agent-Skills `/test`                         | Fresh evidence collected                                                                                                                                                     |
| Ship                | Agent-Skills `shipping-and-launch` or `/ship`               | Superpowers `finishing-a-development-branch` | User-approved external action                                                                                                                                                |

## Scope Rules

Only typo, formatting-only docs, and metadata copy count as trivial changes. Every behavior change requires Interview + Define + Plan, even when the edit is expected to touch one file. Every bug fix requires Interview + Define + Plan before Debug/Build.

| Change type                                                    | Workflow                                               |
| -------------------------------------------------------------- | ------------------------------------------------------ |
| Typo, formatting-only docs, metadata copy                      | Direct edit, then Review + Simplify + verification          |
| Single-file behavior change                                    | Interview + spec + plan, TDD, review + simplify, verification |
| Multi-file feature or refactor                                 | Interview + spec + plan, TDD, review + simplify, verification |
| Bug fix                                                        | Interview + spec + plan, systematic debugging, regression test, TDD fix, review + simplify, verification |
| Auth, payment, PII, secrets, permissions, public API, deletion | Full workflow + security-and-hardening                 |
| Release, deployment, rollout                                   | Ship + branch completion                               |

## Interview → Define → Plan

Each phase has hard gates. Stop at each gate and wait. See [Hard Gates](#hard-gates-mandatory-stop-points).

### Interview (Intent Clarification)

**Goal:** Use `interview-me` to drive confidence to ≥ 95%, eliminate all `GUESS` markers, and produce a user-confirmed intent summary BEFORE writing any spec. Define MUST consume this summary — never skip to spec.

1. Invoke `interview-me`. Iterate until confidence ≥ 95% with no unresolved `GUESS` markers.
2. Output an **intent summary** covering: core goal, user scenarios, boundary constraints, non-goals. Code explorations that answer open questions are recorded as `resolved from code`.

**🔴 GATE 1 — Intent Summary Approval:**
- Ask: "Is this intent summary accurate and complete? May I proceed to write the spec based on it?"
- **STOP. Do NOT enter Define. Do NOT write a spec. Do NOT write code.**
- Wait for explicit user confirmation.

### Define (Spec Writing)

**Prerequisite:** Gate 1 passed, intent summary approved.

1. With the approved intent summary as input, run `spec-driven-development`, `grill-me`, `grill-with-docs`.
2. `grill-me` and `grill-with-docs` must output conclusions: assumptions checked, edge cases resolved, docs/terminology impact assessed.
3. Spec draft must start with: Intent summary recap, Grill findings, Docs decision; then the complete spec.

**🔴 GATE 2 — Spec Approval:**
- Ask: "Does this spec look correct? Shall I proceed to the plan phase?"
- **STOP. Do NOT proceed to plan. Do NOT ask about archiving. Do NOT write code.** Wait for explicit user confirmation.

**🔴 GATE 3 — Archive Decision:**
- Ask: "Archive intent summary and spec to `docs/<feature>/`? Plan will also be archived if you choose yes."
- **STOP until the user answers.**
- Archive: write `docs/<feature>/intent.md` + `docs/<feature>/spec.md`. Don't archive: skip files, note the choice.

### Plan

1. Invoke `/plan` or `planning-and-task-breakdown`; use `writing-plans`.
2. Output the complete plan draft.

**🔴 GATE 4 — Plan Approval:**
- Ask: "Does this plan look correct? Shall I proceed with implementation?"
- **STOP. Do NOT proceed to Build. Do NOT write code.** Wait for explicit user confirmation.

After confirmation: archive plan to `docs/<feature>/plan.md` only if archive was chosen at Gate 3. Then proceed to Build.

### Document Rules (when archiving)

- Body in Chinese; technical identifiers, paths, and quoted source text may remain in original language.
- Store under `docs/<feature>/` (kebab-case).
- `intent.md` for confirmed intent summary, `spec.md` for confirmed spec, `plan.md` for confirmed plan.
- If `docs/<feature>/` already exists, auto-suffix with `-2`, `-3`, etc. Never silently overwrite an existing directory.

**CRITICAL:** Never skip Interview and go straight to spec. Never output a spec/plan and immediately start implementing. Spec approval authorizes only the archive question and plan. Plan approval authorizes only local execution (see Approval Boundaries).

## Execute

Build implements the approved spec according to the approved plan.

TDD remains the primary discipline: write failing test → confirm failure → implement → pass → refactor. Use `incremental-implementation` for vertical slices and feature flags. Use `subagent-driven-development` only when plan tasks are independent with disjoint write scopes.

### After Subagent-Driven Development

Subagent completion ≠ workflow completion. dev-workflow MUST still run Review, Simplify, and Verify. Aggregate all per-task findings into one list for Simplify.

## Debug (conditional)

Triggered on failures. Use `systematic-debugging`: reproduce → locate root cause → add regression test → fix. Use `debugging-and-error-recovery` for build/test recovery.

## Review

After Build passes: aggregate all findings, fix CRITICAL/HIGH immediately, record IMPORTANT/MINOR for Simplify. Run `security-and-hardening` in parallel if security gate triggers.

## Simplify (MANDATORY)

Always follows Review. For every finding: **fix it**, **defer it** with `// TODO(simplify):` comment, or **reject it** with explanation. Re-run tests afterward.
**Exit checklist:** all IMPORTANT findings addressed; no function >50 lines added; no dead code; no duplicated logic; all tests pass.

## Security Gate

Trigger: auth, payment, PII, secrets, public API, DB queries, file uploads, data deletion, CI/deploy changes. High/critical findings block shipping.

## Verification

Before claiming completion: run fresh tests, full tests, lint, typecheck, build. Report exact results. No evidence = no completion claim.

## Completion Gate

Before the final report, every phase task must be `completed` or `deleted` (with reason). If any phase is `pending`, finish it first. Simplify can never be deleted.

## Approval Boundaries

Plan approval authorizes: local edits, tests, lint/typecheck/build, review, simplification, local verification.
Stop for explicit user approval before: commits (if including unrelated user work), push, PR, merge, publish, deploy, credential changes, destructive data changes.

## Response Shape

- Working update: `Using dev-workflow.`
- Missing capability: `Agent-Skills is available, but Superpowers is not. Install Superpowers or confirm degraded mode.`
- Phase progress at major transitions only: `✅ Intake ✅ Interview ✅ Define ✅ Plan ✅ Build 🔄 Review ⏳ Simplify`
- Final report after Completion Gate: changed files; Agent-Skills/standalone gates; Superpowers gates; Simplify resolution; exact verification results; skipped commit/push/PR/deploy.
