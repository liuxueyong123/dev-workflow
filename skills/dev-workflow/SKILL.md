---
name: dev-workflow
description: MUST use when the user explicitly asks to use, run, invoke, follow, or apply dev-workflow, $dev-workflow, /dev-workflow, or dev-flow by name, including routine code edits, defect repair, restructuring, cleanup, and sensitive work. Do not use when the user does not name dev-workflow/dev-flow, negates use of it, or only asks to explain or edit the skill itself.
---

# Dev Workflow

Coordinates **Agent-Skills** (intent, spec, plan, review, simplify, security, ship), standalone skills (`grill-with-docs`, `grilling`, `domain-modeling`), and **Superpowers** (brainstorming, TDD, debugging, worktrees, verification) as one production workflow.

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

Check that Agent-Skills, Superpowers, and required standalone skills (`grill-with-docs`, `grilling`, `domain-modeling` for plan review) are available. If any are missing, stop and tell the user. Continue in degraded mode only if the user explicitly approves it.

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

**Protocol:** Output draft → ask confirmation → **STOP** → wait for user → continue. Common violations: coding after spec output; bundling approval+archive; skipping archive; entering Define before ≥95% confidence.

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
| Define              | Agent-Skills `spec-driven-development`                     | Superpowers `brainstorming`                  | Spec from approved intent summary; behavior-only (no implementation); archive recorded                                                                        |
| Plan                | Agent-Skills `planning-and-task-breakdown` or `/plan`; standalone `grill-with-docs` | Superpowers `writing-plans` | Plan drafted, grilled, revised; user-approved; archived if archive chosen at Gate 3 |
| Isolate             | Superpowers `using-git-worktrees`                           | Agent-Skills `git-workflow-and-versioning`   | Work area is safe                                                                                                                                                            |
| Build               | Superpowers `test-driven-development`; use `subagent-driven-development` when tasks are independent with disjoint write scopes | Agent-Skills `incremental-implementation` | Approved spec/plan implemented; focused tests pass |
| Debug (conditional) | Superpowers `systematic-debugging`                          | Agent-Skills `debugging-and-error-recovery`  | Root cause fixed, regression test added                                                                                                                                      |
| Review              | Agent-Skills `code-review-and-quality` or `/review` | Superpowers `requesting-code-review` | CRITICAL/HIGH fixed; IMPORTANT/MINOR captured for Simplify |
| Security            | Security: Agent-Skills `security-and-hardening`             | Project audit commands                       | Security-sensitive findings addressed                                                                                                                                        |
| Simplify            | Agent-Skills `code-simplification` or `/code-simplify` | Superpowers `verification-before-completion` | Every review finding addressed (fixed, deferred with comment, or rejected). No dupes, dead code, or >50-line functions; behavior unchanged |
| Verify              | Superpowers `verification-before-completion`                | Agent-Skills `/test`                         | Fresh evidence collected                                                                                                                                                     |
| Ship                | Agent-Skills `shipping-and-launch` or `/ship`               | Superpowers `finishing-a-development-branch` | User-approved external action                                                                                                                                                |

## Scope Rules

Trivial = typo, formatting-only docs, metadata copy only. Every behavior change and bug fix requires Interview + Define + Plan.

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

1. Invoke `interview-me`. Iterate until confidence ≥ 95% with no unresolved `GUESS` markers. **Do NOT accept a single-question interview as complete** — re-invoke `interview-me` if confidence is below threshold or any GUESS remains. Minimum 3+ question-answer exchanges unless the task is unambiguously trivial.
2. Output an **intent summary** covering: core goal, user scenarios, boundary constraints, non-goals. Code explorations that answer open questions are recorded as `resolved from code`.

**🔴 GATE 1 — Intent Summary Approval:**
- Ask: "Is this intent summary accurate and complete? May I proceed to write the spec based on it?"
- **STOP. Do NOT enter Define. Do NOT write a spec. Do NOT write code.**
- Wait for explicit user confirmation.

### Define (Spec Writing)

**Prerequisite:** Gate 1 passed, intent summary approved.

**Content boundary:** Spec = WHAT/WHY (behavior, scenarios, constraints, acceptance). Plan = HOW (code, architecture, APIs, DB, files). No implementation detail in spec — that is the Plan phase's job.

1. With the approved intent summary, run `spec-driven-development` to write the spec.
2. Spec draft header: intent recap; then the complete spec (behavior-only, no code, no architecture).

**🔴 GATE 2 — Spec Approval:**
- Ask: "Does this spec look correct? Shall I proceed to the plan phase?"
- **STOP.** Wait for explicit user confirmation.

**🔴 GATE 3 — Archive Decision:**
- Ask: "Archive intent summary and spec to `docs/<feature>/`? Plan will also be archived if chosen."
- **STOP until the user answers.**
- Archive: write `docs/<feature>/intent.md` + `docs/<feature>/spec.md`. Otherwise skip.

### Plan

**Input:** Approved spec.

1. Invoke `/plan` or `planning-and-task-breakdown`; use `writing-plans`. Resolve implementation decisions: frameworks, libraries, DB, API patterns, file structure, architecture.
2. Output the plan draft.
3. **MANDATORY — do NOT skip:** Invoke `grill-with-docs` against the plan draft to verify docs alignment, API correctness, version compatibility. This step is not optional. If `grill-with-docs` is unavailable, stop and report — do not proceed without it or explicit degraded-mode approval.
4. Revise the plan based on grill findings. Summarize changes before presenting the final plan.

**🔴 GATE 4 — Plan Approval:**
- Ask: "Does this plan look correct? Shall I proceed with implementation?"
- **STOP. Do NOT proceed to Build. Do NOT write code.** Wait for explicit user confirmation.

After confirmation: archive plan to `docs/<feature>/plan.md` only if archive was chosen at Gate 3. Then proceed to Build.

### Document Rules (when archiving)

- Body in Chinese; technical identifiers, paths, and quoted source text may remain in original language.
- Store under `docs/<feature>/` (kebab-case).
- `intent.md` for confirmed intent summary, `spec.md` for confirmed spec, `plan.md` for confirmed plan.
- If `docs/<feature>/` already exists, auto-suffix with `-2`, `-3`, etc. Never silently overwrite an existing directory.

**CRITICAL:** Never skip Interview→spec. Never output spec/plan then code. Spec approval → archive + plan only. Plan approval → local execution only.

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
