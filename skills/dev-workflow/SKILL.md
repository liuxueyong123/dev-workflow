---
name: dev-workflow
description: MUST use when the user explicitly asks to use, run, invoke, follow, or apply dev-workflow, $dev-workflow, /dev-workflow, or dev-flow by name, including routine code edits, defect repair, restructuring, cleanup, and sensitive work. Do not use when the user does not name dev-workflow/dev-flow, negates use of it, or only asks to explain or edit the skill itself.
---

# Dev Workflow

Coordinates **Agent-Skills** (intent, spec, plan, review, simplify, security, ship), standalone skills (`grill-with-docs`, `grilling`, `domain-modeling`), and **Superpowers** (brainstorming, TDD, debugging, worktrees, verification) as one production workflow.

## Explicit Invocation Only

Activate only on: `dev-workflow`, `$dev-workflow`, `/dev-workflow`, `dev-flow`, "use dev-workflow", "用 dev-workflow". Do NOT activate on: "do not use", "explain", "update dev-workflow skill", or ordinary coding requests. Once invoked, mandatory regardless of task type.

On positive invocation: stop any other path → send `Using dev-workflow.` → check Capability Gate → create Phase Tracking tasks → **respect every Hard Gate.**

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

| Gate | Phase     | Trigger                | Required Confirmation        |
| ---- | --------- | ---------------------- | ---------------------------- |
| 1    | Interview | Intent summary output  | User approves intent summary |
| 2    | Define    | Spec draft output      | User approves spec content   |
| 3    | Define    | Spec approved          | User chooses archive or not  |
| 4    | Plan      | Plan draft output      | User approves plan           |
| 5    | Ship      | Before external action | User explicitly requests it  |

**Protocol:** Output draft → ask confirmation → **STOP. Do NOT proceed.** → wait for user → continue. Common violations: coding after spec output; bundling approval+archive; skipping archive; entering Define before ≥95% confidence.

## Workflow

```text
intake → interview → define → plan → isolate → build → [debug] → review  ─┐
                                                                          ├── simplify → verify → ship
                                                                security ─┘
```

Review and security run in parallel when both are triggered. Never skip a phase without marking its task `deleted` with a reason.

## Phase Ownership

| Phase               | Primary                                                                                                                              | Secondary                                    | Exit gate                                                                                                                                  |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Intake              | Capability check (see Capability Gate)                                                                                               | —                                            | Agent-Skills, standalone skills, and Superpowers confirmed, or degraded mode explicitly approved                                           |
| Interview           | Agent-Skills `interview-me`                                                                                                          | Superpowers `brainstorming`                  | Confidence ≥ 95%; all GUESS markers resolved; intent summary output and approved by user (Gate 1)                                          |
| Define              | Agent-Skills `spec-driven-development`                                                                                               | Superpowers `brainstorming`                  | Spec from approved intent summary; behavior-only (no implementation); archive recorded                                                     |
| Plan                | Agent-Skills `planning-and-task-breakdown` or `/plan`; standalone `grill-with-docs`                                                  | Superpowers `writing-plans`                  | Plan drafted, grilled, revised; user-approved; archived if archive chosen at Gate 3                                                        |
| Isolate             | Superpowers `using-git-worktrees`                                                                                                    | Agent-Skills `git-workflow-and-versioning`   | Work area is safe                                                                                                                          |
| Build               | Superpowers `subagent-driven-development` (default, subagents do TDD); inline `test-driven-development` for single/overlapping tasks | Agent-Skills `incremental-implementation`    | Approved spec/plan implemented; focused tests pass                                                                                         |
| Debug (conditional) | Superpowers `systematic-debugging`                                                                                                   | Agent-Skills `debugging-and-error-recovery`  | Root cause fixed, regression test added                                                                                                    |
| Review              | Agent-Skills `code-review-and-quality` or `/review`                                                                                  | Superpowers `requesting-code-review`         | CRITICAL/HIGH fixed; IMPORTANT/MINOR captured for Simplify                                                                                 |
| Security            | Security: Agent-Skills `security-and-hardening`                                                                                      | Project audit commands                       | Security-sensitive findings addressed                                                                                                      |
| Simplify            | Agent-Skills `code-simplification` or `/code-simplify`                                                                               | Superpowers `verification-before-completion` | Every review finding addressed (fixed, deferred with comment, or rejected). No dupes, dead code, or >50-line functions; behavior unchanged |
| Verify              | Superpowers `verification-before-completion`                                                                                         | Agent-Skills `/test`                         | Fresh evidence collected                                                                                                                   |
| Ship                | Agent-Skills `shipping-and-launch` or `/ship`                                                                                        | Superpowers `finishing-a-development-branch` | User-approved external action                                                                                                              |

## Scope Rules

Trivial = typo, formatting-only docs, metadata copy only. Every behavior change and bug fix requires Interview + Define + Plan.

| Change type                                                    | Workflow                                                                                                 |
| -------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| Typo, formatting-only docs, metadata copy                      | Direct edit, then Review + Simplify + verification                                                       |
| Single-file behavior change                                    | Interview + spec + plan, TDD, review + simplify, verification                                            |
| Multi-file feature or refactor                                 | Interview + spec + plan, TDD, review + simplify, verification                                            |
| Bug fix                                                        | Interview + spec + plan, systematic debugging, regression test, TDD fix, review + simplify, verification |
| Auth, payment, PII, secrets, permissions, public API, deletion | Full workflow + security-and-hardening                                                                   |
| Release, deployment, rollout                                   | Ship + branch completion                                                                                 |

## Interview → Define → Plan

Each phase has hard gates. Stop at each gate and wait. See [Hard Gates](#hard-gates-mandatory-stop-points).

### Interview (Intent Clarification)

**Goal:** Use `interview-me` to drive confidence to ≥ 95%, eliminate all `GUESS` markers, and produce a user-confirmed intent summary BEFORE writing any spec. Define MUST consume this summary — never skip to spec.

1. Invoke `interview-me`. Iterate until confidence ≥ 95% with no unresolved `GUESS` markers. **Do NOT accept a single-question interview as complete** — re-invoke `interview-me` if confidence is below threshold or any GUESS remains. Minimum 3+ question-answer exchanges unless the task is unambiguously trivial.
2. Output an **intent summary** covering: core goal, user scenarios, boundary constraints, non-goals. Code explorations that answer open questions are recorded as `resolved from code`.

**🔴 GATE 1 — Intent Summary Approval:**
Ask: "Is this intent summary accurate and complete? May I proceed to write the spec based on it?" → **STOP. Do NOT enter Define.** Wait for explicit user confirmation.

### Define (Spec Writing)

**Prerequisite:** Gate 1 passed, intent summary approved.

1. With the approved intent summary, use `spec-driven-development` only to draft spec content for the conversation.
2. Override delegated skill file-save rules in this workflow: before Gate 2 approval and Gate 3 archive choice, do NOT create or modify any spec, intent, plan, task, code, or docs file.
3. Spec draft header: intent recap; then the complete spec (behavior-only, no code, no architecture). Preserve the full raw spec draft in conversation state; if the user requests changes, track each revision request and decision for archive fidelity.

**🔴 GATE 2 — Spec Approval:**
Ask: "Does this spec look correct? Shall I proceed to the plan phase?" → **STOP.** Wait for explicit user confirmation.

**🔴 GATE 3 — Archive Decision:**
Ask: "Archive intent summary and spec to `docs/<feature>/`?" → **STOP until the user answers.** Archive only after explicit archive approval: write `docs/<feature>/intent.md` + `docs/<feature>/spec.md`. Otherwise skip all intent/spec filesystem writes.

### Plan

**Input:** Approved spec. **Execute steps in strict order — do NOT interleave. Grill happens AFTER the draft is presented, never before or during.**

1. Invoke `/plan` or `planning-and-task-breakdown`; use `writing-plans`. Resolve implementation decisions: frameworks, libraries, DB, API patterns, file structure, architecture.
2. Output the raw plan draft in Chinese. Do NOT include grill findings here — the draft is un-reviewed.
3. **MANDATORY:** Now invoke `grill-with-docs` via the Skill tool against the raw plan draft. The skill works interactively — relay each question to the user one at a time, wait for the answer, then relay the next. Do NOT batch all output at once. Do NOT write your own assessment under a "grill" label. Fabricating output = workflow violation.
4. Revise the plan based on the actual grill Q&A. Present the final plan in Chinese with a summary of what changed. Preserve the full raw plan draft and actual grill Q&A/findings for archive fidelity.

**🔴 GATE 4 — Plan Approval:**
Ask: "Does this plan look correct? Shall I proceed?" → **STOP.** After confirmation: if Gate 3 chose archive, write the complete plan archive packet to `docs/<feature>/plan.md` and verify the write before Build. If the write fails, STOP and report it. If Gate 3 chose no archive, do not write plan docs. Then Build.

### Document Rules

- **All user-facing output** (intent summary, spec, plan draft, plan final) must be in Chinese. Technical identifiers, file paths, code symbols, and quoted source text may remain in original language.
- **Archiving** (when chosen at Gate 3): same language rules apply.
- Store under `docs/<feature>/` (kebab-case).
- `intent.md` for confirmed intent summary, `spec.md` for confirmed spec, `plan.md` for confirmed plan.
- If `docs/<feature>/` already exists, auto-suffix with `-2`, `-3`, etc. Never silently overwrite an existing directory.
- Archive fidelity is mandatory: never replace a detailed raw draft with only a short revised summary.
- `spec.md` must include raw spec draft, revision requests/decisions, final approved spec, and change summary when the spec changed after feedback.
- `plan.md` must include raw plan draft, actual grill Q&A/findings, final approved plan, and change summary.

**CRITICAL:** Never skip Interview→spec. Never output spec/plan then code. Spec approval → archive choice + plan only. Plan approval → archive-before-Build if Gate 3 chose archive; local execution only after that.

## Execute (Build → Debug → Review)

Build implements the approved spec/plan. **Default: `subagent-driven-development`** — fan out independent plan tasks in parallel (each subagent follows TDD internally — no trade-off). Fall back to inline **TDD** only when tasks share write scopes or the plan is a single linear task. Use `incremental-implementation` for vertical slices. **Subagent completion ≠ workflow completion** — still run Review, Simplify, Verify.

**Debug (conditional):** `systematic-debugging`: reproduce → root cause → regression test → fix.

**Review:** aggregate findings → fix CRITICAL/HIGH → record IMPORTANT/MINOR for Simplify. Run `security-and-hardening` in parallel if security gate triggers.

## Simplify (MANDATORY)

Always follows Review. For every finding: **fix it**, **defer it** (`// TODO(simplify):`), or **reject it** with reason. Re-run tests. Exit: all IMPORTANT addressed; no >50-line functions; no dead/duplicated code; tests pass.

## Security Gate

Trigger: auth, payment, PII, secrets, public API, DB queries, file uploads, data deletion, CI/deploy changes. High/critical findings block ship.

## Verification

Before claiming completion: fresh + full tests, lint, typecheck, build. Report results. No evidence = no completion claim.

## Completion Gate

Every phase task `completed` or `deleted` (with reason) before final report. Simplify can never be deleted.

## Approval Boundaries

Plan approval authorizes: required plan archive from Gate 4, local edits, tests, lint/typecheck/build, review, simplification, local verification. It does not authorize skipping the plan archive when Gate 3 chose archive. Stop for explicit approval before: commits (with unrelated user work), push, PR, merge, publish, deploy, credential changes, destructive data changes.

## Response Shape

- Working: `Using dev-workflow.` | Missing: name missing skills, ask to confirm degraded mode.
- Progress: `✅ Intake ✅ Interview ✅ Define ✅ Plan ✅ Build 🔄 Review ⏳ Simplify`
- Final report: changed files; all gate results; Simplify resolution; verification evidence; skipped commit/push/PR/deploy.
