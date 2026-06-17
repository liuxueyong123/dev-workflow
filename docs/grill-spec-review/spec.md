# Spec: 为 dev-workflow 增加 spec 审查拷问流程

## Objective

在 `skills/dev-workflow/SKILL.md` 中加入独立 skills `grill-with-docs` 和 `grill-me` 流程，用于在审查 spec 文档时更严格地拷问方案、术语、项目文档一致性和决策质量。

## Scope

- 修改 `skills/dev-workflow/SKILL.md`。
- 保持 `SKILL.md` 不超过 200 行；当前已是 200 行，所以新增内容必须通过压缩现有文字来腾出空间。
- 更新版本号到新的唯一版本，从 `1.4.6` 升到 `1.4.7`：
  - `plugin.json`
  - `.claude-plugin/plugin.json`
  - `.claude-plugin/marketplace.json`

## Expected Behavior

- `Define` 阶段的 spec 审查中，应明确引入：
  - 独立 skill `grill-me`：用于压力测试 spec、假设、边界、成功标准。
  - 独立 skill `grill-with-docs`：用于把 spec 与项目已有文档、术语、决策记录对齐，并在必要时更新文档。
- 这两个流程应作为 spec 审查和强化步骤，而不是替代 `spec-driven-development`。
- 仍保留 dev-workflow 的硬门禁：spec 审批、归档选择、plan 审批、外部动作审批。

## Success Criteria

- `SKILL.md` 明确说明 spec 文档审查时何时使用 `grill-me` 和 `grill-with-docs`，并说明二者是独立 skills，不属于 Agent-Skills。
- `SKILL.md` 行数 `<= 200`。
- 3 个版本字段一致且唯一。
- JSON 文件格式有效。
- 不引入安全敏感改动；无需触发 Security phase。

## Verification

- `wc -l skills/dev-workflow/SKILL.md`
- `rg "grill-(me|with-docs)|version" skills/dev-workflow/SKILL.md plugin.json .claude-plugin/plugin.json .claude-plugin/marketplace.json`
- `python3 -m json.tool plugin.json`
- `python3 -m json.tool .claude-plugin/plugin.json`
- `python3 -m json.tool .claude-plugin/marketplace.json`
