# 将 `interview-me` 纳入 `dev-workflow` Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use local execution with TDD-style shell checks. Do not use subagents unless the user later asks for parallel review.

**Goal:** 在 `dev-workflow` 的 Define 阶段加入 `interview-me` 作为需求不明确时的意图抽取前置 skill。

**Architecture:** 只修改工作流编排文本，不复制 `interview-me` 原文。保持 `skills/dev-workflow/SKILL.md` 作为 canonical workflow surface，并同步插件版本元数据。

**Tech Stack:** Markdown skill file, JSON plugin manifests, shell verification.

## Global Constraints
- `skills/dev-workflow/SKILL.md` 必须保持 `<= 200` 行。
- 插件版本号必须唯一，且同步到 `plugin.json`、`.claude-plugin/plugin.json`、`.claude-plugin/marketplace.json`。
- 不清理或重置已有未跟踪内容：`.agents/`、`skills-lock.json`。
- 不 commit、不 push、不发布插件，除非用户后续明确要求。

---

## Files
- Modify: `skills/dev-workflow/SKILL.md`
- Modify: `plugin.json`
- Modify: `.claude-plugin/plugin.json`
- Modify: `.claude-plugin/marketplace.json`
- Create after plan approval because archive was selected: `docs/add-interview-me-to-dev-workflow/plan.md`

## Task 1: Establish Red Checks

**Description:** 先证明当前目标还没满足，作为本次文本型 TDD 的 red baseline。

**Acceptance criteria:**
- `skills/dev-workflow/SKILL.md` 当前不包含 `interview-me`。
- 当前版本仍是 `1.4.4`。
- 当前 `SKILL.md` 行数已知，作为压缩约束依据。

**Steps:**
- Run: `rg -n "interview-me|Interview Me" skills/dev-workflow/SKILL.md`
  - Expected before implementation: no matches.
- Run: `rg -n '"version": "1.4.4"' plugin.json .claude-plugin/plugin.json .claude-plugin/marketplace.json`
  - Expected before implementation: matches in all three version surfaces.
- Run: `wc -l skills/dev-workflow/SKILL.md`
  - Expected before implementation: `200`.

**Dependencies:** None.

## Task 2: Add `interview-me` to Define-Phase Orchestration

**Description:** Edit `skills/dev-workflow/SKILL.md` so Define can use `interview-me` before spec-writing when intent is underspecified.

**Acceptance criteria:**
- Overview mentions `interview-me` in the Agent-Skills set.
- Phase Ownership Define row includes `interview-me` before `spec-driven-development`.
- Define instructions say to invoke `interview-me` when the ask is missing user, why, success, or binding constraint.
- The file remains `<= 200` lines.

**Exact content intent:**
- Change overview from:
  - `Agent-Skills (spec, plan, review, simplify, security, ship)`
- To include:
  - `Agent-Skills (intent, spec, plan, review, simplify, security, ship)`
- Change Define ownership to include:
  - `Agent-Skills interview-me, spec-driven-development or /spec`
- Add one concise Define rule:
  - `If intent is underspecified, run interview-me first and carry its confirmed intent into the spec.`

**Verification:**
- Run: `rg -n "interview-me|confirmed intent|underspecified" skills/dev-workflow/SKILL.md`
  - Expected: matches in Define-related lines.
- Run: `wc -l skills/dev-workflow/SKILL.md`
  - Expected: `200` or less.

**Dependencies:** Task 1.

## Task 3: Bump Plugin Version to `1.4.5`

**Description:** Update all required plugin metadata version fields from `1.4.4` to `1.4.5`.

**Acceptance criteria:**
- `plugin.json` contains `"version": "1.4.5"`.
- `.claude-plugin/plugin.json` contains `"version": "1.4.5"`.
- `.claude-plugin/marketplace.json` plugin entry contains `"version": "1.4.5"`.
- All JSON files remain valid.

**Verification:**
- Run: `rg -n '"version": "1.4.5"' plugin.json .claude-plugin/plugin.json .claude-plugin/marketplace.json`
  - Expected: three matches.
- Run: `python3 -m json.tool plugin.json >/dev/null && python3 -m json.tool .claude-plugin/plugin.json >/dev/null && python3 -m json.tool .claude-plugin/marketplace.json >/dev/null`
  - Expected: exit code `0`.

**Dependencies:** Task 1.

## Task 4: Review, Simplify, and Final Verification

**Description:** Inspect the resulting diff for overlong skill text, accidental unrelated changes, invalid JSON, and missed requirements.

**Acceptance criteria:**
- Diff touches only approved files plus archived docs.
- No full copy of the 225-line global `interview-me` skill appears in `dev-workflow`.
- `SKILL.md` remains focused and under the project line limit.
- All verification commands pass.

**Verification:**
- Run: `git diff -- skills/dev-workflow/SKILL.md plugin.json .claude-plugin/plugin.json .claude-plugin/marketplace.json docs/add-interview-me-to-dev-workflow/spec.md docs/add-interview-me-to-dev-workflow/plan.md`
- Run: `wc -l skills/dev-workflow/SKILL.md`
- Run: `rg -n "interview-me|confirmed intent|underspecified" skills/dev-workflow/SKILL.md`
- Run JSON validation command from Task 3.
- Run: `git status --short`

**Dependencies:** Tasks 2 and 3.

## Risks and Mitigations

| Risk | Impact | Mitigation |
|---|---:|---|
| Exceeding 200-line `SKILL.md` limit | High | Replace or compress existing wording while adding only concise `interview-me` rules. |
| Duplicating full `interview-me` logic | Medium | Reference it as an upstream skill and keep detailed behavior in the source skill. |
| Version drift across metadata files | Medium | Verify all three required version surfaces with one `rg` command. |
| Accidentally touching user untracked files | Medium | Restrict edits to explicit files only. |

## Self-Review
- Spec coverage: covered target skill integration, version bump, line limit, no unrelated file cleanup.
- Placeholder scan: no `TBD`, `TODO`, or unspecified implementation steps.
- Dependency order: red checks first, then skill text and version metadata, then review/verify.
- Task size: each task is small and touches no more than the approved file set.
