# Spec: 将 `interview-me` 纳入 `dev-workflow` 的 Define 阶段

## Objective
把 Agent-Skills 中的 `interview-me` 加入 `skills/dev-workflow/SKILL.md` 的工作流说明里，让 `dev-workflow` 在需求不清晰时先抽取用户真实意图，再进入 `spec-driven-development`。

## Assumptions
- 目标文件是 `skills/dev-workflow/SKILL.md`。
- 来源是全局 skill：`/Users/lxy/.codex/skills/interview-me/SKILL.md`。
- 不复制 `interview-me` 的完整 225 行内容；只把它作为 Define 阶段的前置/辅助 skill 集成进 `dev-workflow`。
- 本次属于插件更新，需要把版本从 `1.4.4` bump 到唯一的新版本，建议 `1.4.5`。
- 需要同步更新：
  - `plugin.json`
  - `.claude-plugin/plugin.json`
  - `.claude-plugin/marketplace.json`

## Recommended Approach
采用“轻量集成”方案：在 `dev-workflow` 的 Overview、Phase Ownership、Define 步骤中加入 `interview-me`，说明它用于“需求不明确、用户真实意图未确认、进入 spec 前需要 one-question-at-a-time intent extraction”的场景。

不整段复制 `interview-me`，因为目标 `SKILL.md` 当前正好 200 行，而项目约束要求不能超过 200 行。实现时需要压缩现有文本，保持 `SKILL.md <= 200` 行。

## Alternatives
1. **轻量集成到 Define 阶段**：推荐。最符合 `dev-workflow` 的编排职责，也满足 200 行限制。
2. **完整复制 `interview-me` 内容**：不推荐。会严重超出行数限制，并重复已有 skill。
3. **新增本地 `.agents/skills/interview-me/`**：暂不做。用户这次明确说加入到 `skill.md`，不是安装新 skill。

## Commands
- 查看行数：`wc -l skills/dev-workflow/SKILL.md`
- 验证命中：`rg -n "interview-me|Interview" skills/dev-workflow/SKILL.md`
- 验证版本一致：`rg -n '"version": "1.4.5"' plugin.json .claude-plugin/plugin.json .claude-plugin/marketplace.json`
- JSON 校验：`python3 -m json.tool plugin.json >/dev/null && python3 -m json.tool .claude-plugin/plugin.json >/dev/null && python3 -m json.tool .claude-plugin/marketplace.json >/dev/null`
- 查看变更：`git diff -- skills/dev-workflow/SKILL.md plugin.json .claude-plugin/plugin.json .claude-plugin/marketplace.json`

## Testing Strategy
先做“红灯”取证：当前 `skills/dev-workflow/SKILL.md` 不包含 `interview-me`，且版本仍是 `1.4.4`。

实现后做“绿灯”验证：`interview-me` 出现在 Define 编排中，目标 skill 不超过 200 行，三个版本文件都为 `1.4.5`，JSON 有效。

## Boundaries
- Always: 保持 `SKILL.md <= 200` 行；保持 YAML frontmatter 有效；同步三个版本号。
- Ask first: 如果要新增本地 `interview-me` skill 目录、改 README、或改 `.agents/` 未跟踪内容。
- Never: 不重置用户已有未跟踪文件；不提交、不 push、不发布插件，除非用户明确要求。

## Success Criteria
- `skills/dev-workflow/SKILL.md` 明确包含 `interview-me` 在 Define 阶段的使用规则。
- `skills/dev-workflow/SKILL.md` 行数不超过 200。
- `plugin.json`、`.claude-plugin/plugin.json`、`.claude-plugin/marketplace.json` 版本一致且唯一，建议为 `1.4.5`。
- 验证命令全部通过。
- 不触碰无关未跟踪内容：当前 `.agents/` 和 `skills-lock.json` 只在必要时读取，不清理。
