# Agent-Skills + Superpowers Dev Workflow 实施计划

> **给自动化执行者：** 按任务逐项执行；如果运行时支持，优先使用 `superpowers:subagent-driven-development`，否则使用 `superpowers:executing-plans` 或内联执行。任务使用 checkbox (`- [ ]`) 追踪。

**目标：** 让 `dev-workflow` 重新以 Agent-Skills + Superpowers 为中心，并强制 spec/plan 文档使用中文和统一路径。

**架构：** 保持发布的 skill 小而明确：Agent-Skills 负责生命周期质量门禁，Superpowers 负责执行纪律，dev-workflow 负责编排两者交接。

**技术栈：** Markdown skills、JSON eval prompts、Claude Code plugin manifest、shell validation commands。

---

## 任务 1: 更新文档

- [ ] 用中文重写 `README.md`，突出 Agent-Skills + Superpowers 是主要依赖。
- [ ] 增加 Claude Code 安装 Superpowers 和 Agent-Skills 的命令。
- [ ] 增加 Codex 安装 Superpowers plugin，以及把 Agent-Skills 复制到 `.agents/skills` 或 `~/.agents/skills` 的说明。
- [ ] 移除把泛用兼容 fallback 作为主叙事的内容。
- [ ] 写明所有 spec/plan 文档必须使用中文，并写入 `docs/<feature>/spec.md` 和 `docs/<feature>/plan.md`。

## 任务 2: 更新 Skill

- [ ] 重写 `skills/dev-workflow/SKILL.md`，让缺失 Agent-Skills 或 Superpowers 时触发 setup gate。
- [ ] 定义 Agent-Skills 和 Superpowers 的 phase ownership。
- [ ] 增加 spec/plan 文档规则：中文正文、`docs/<feature>/spec.md`、`docs/<feature>/plan.md`。
- [ ] 保留外部动作必须显式审批的边界。

## 任务 3: 更新元数据和 Evals

- [ ] 更新 `plugin.json` 和 `.claude-plugin/plugin.json`。
- [ ] 更新 `evals/evals.json`，覆盖组合能力 workflow、缺失能力行为，以及中文 spec/plan 路径规则。
- [ ] 更新 `CLAUDE.md` 维护指南。

## 任务 4: 验证

- [ ] 运行 JSON 解析检查。
- [ ] 运行 skill frontmatter 检查。
- [ ] 运行 eval schema 检查。
- [ ] 运行 spec/plan 文档规则检查。
- [ ] 搜索过时的 generic-runtime 表述。
