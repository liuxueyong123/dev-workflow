# Grill Spec Review Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在 `dev-workflow` 技能中加入独立 skills `grill-me` 和 `grill-with-docs` 作为 spec 文档审查流程，并保持技能文件长度、版本和验证要求一致。

**Architecture:** 这是一个技能文本和插件元数据更新。实现方式是压缩 `SKILL.md` 现有内容，加入 spec review 说明；同时增加 eval 覆盖，证明用户要求“审查 spec 文档”时 workflow 应引入两个 grill 流程。

**Tech Stack:** Markdown skill file, JSON plugin metadata, JSON eval definitions.

## Global Constraints

- `skills/dev-workflow/SKILL.md` 必须 `<= 200` 行。
- 插件版本号必须唯一，且同步更新 `plugin.json`、`.claude-plugin/plugin.json`、`.claude-plugin/marketplace.json`。
- 不修改无关文件。
- 不触发外部动作：不 commit、不 push、不发布。

---

## Task 1: Add Eval Coverage First

**Files:**
- Modify: `evals/evals.json`

**Interfaces:**
- Consumes: Existing `evals.evals[]` structure.
- Produces: New eval entry with id `spec-review-grill-flows`.

- [ ] **Step 1: Add the failing behavior expectation**

Add an eval entry whose prompt asks dev-workflow to review a spec document. The expected output must require `grill-me`, `grill-with-docs`, and preservation of `spec-driven-development` as the spec creation gate.

- [ ] **Step 2: Validate JSON**

Run: `python3 -m json.tool evals/evals.json`
Expected: exit 0.

## Task 2: Update `SKILL.md` Within 200 Lines

**Files:**
- Modify: `skills/dev-workflow/SKILL.md`

**Interfaces:**
- Consumes: Existing dev-workflow phase structure and hard gates.
- Produces: Define/spec review instructions naming standalone skills `grill-me` and `grill-with-docs`.

- [ ] **Step 1: Add spec review flow**

Add concise instructions that identify both grill flows as standalone skills and use:
- `grill-me` to pressure-test assumptions, boundaries, success criteria, counterexamples, and risks.
- `grill-with-docs` to align terminology, ADRs, context docs, and project knowledge; update project docs in the right place when decisions crystallize.

- [ ] **Step 2: Preserve line budget**

Compress existing wording so `wc -l skills/dev-workflow/SKILL.md` remains `<= 200`.

## Task 3: Bump Plugin Version

**Files:**
- Modify: `plugin.json`
- Modify: `.claude-plugin/plugin.json`
- Modify: `.claude-plugin/marketplace.json`

**Interfaces:**
- Consumes: Existing version `1.4.6`.
- Produces: Version `1.4.7` in all three locations.

- [ ] **Step 1: Update versions**

Change every plugin version field from `1.4.6` to `1.4.7`.

- [ ] **Step 2: Validate metadata**

Run:
- `python3 -m json.tool plugin.json`
- `python3 -m json.tool .claude-plugin/plugin.json`
- `python3 -m json.tool .claude-plugin/marketplace.json`

Expected: all exit 0.

## Task 4: Review, Simplify, Verify

**Files:**
- Inspect: full working tree diff.

**Interfaces:**
- Consumes: Tasks 1-3.
- Produces: Verified final diff without external action.

- [ ] **Step 1: Review diff**

Check that changes are scoped to the approved files, hard gates remain intact, and no security-sensitive behavior was introduced.

- [ ] **Step 2: Simplify**

Remove redundant wording and keep `SKILL.md` readable within the 200-line limit.

- [ ] **Step 3: Verify**

Run:
- `wc -l skills/dev-workflow/SKILL.md`
- `rg "grill-(me|with-docs)|version" skills/dev-workflow/SKILL.md plugin.json .claude-plugin/plugin.json .claude-plugin/marketplace.json`
- `python3 -m json.tool evals/evals.json`
- `python3 -m json.tool plugin.json`
- `python3 -m json.tool .claude-plugin/plugin.json`
- `python3 -m json.tool .claude-plugin/marketplace.json`
- `git diff --check`
