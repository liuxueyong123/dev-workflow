# Agent-Skills + Superpowers Dev Workflow 设计

## 目标

让 `dev-workflow` 明确组合 Agent-Skills 和 Superpowers，而不是退化成泛用兼容流程。

## 修正方向

核心价值来自两套能力包的组合：

- Agent-Skills 提供生命周期质量门禁：spec、plan、增量实现、测试、review、简化、security、performance、documentation、CI/CD、git workflow、shipping。
- Superpowers 提供执行纪律：skill selection、brainstorming、written implementation plans、TDD、systematic debugging、worktree isolation、subagent-driven execution、verification before completion、development-branch finishing。

Claude Code 和 Codex 都是支持的入口，但它们不是 workflow 的能力来源。workflow 应优先使用这两套能力包；如果任意一套缺失，必须停止并提示，除非用户明确批准降级执行。

## 文档规则

Agent-Skills、Superpowers 或 dev-workflow 生成的 spec 和 plan 文档必须满足：

1. 文档正文使用中文；技术标识、文件路径、命令、API 名称和引用文本可以保留原语言。
2. 每个功能使用一个简短 kebab-case 名称作为 `<feature>`。
3. spec 固定写入 `docs/<feature>/spec.md`。
4. plan 固定写入 `docs/<feature>/plan.md`。
5. 如果上游 skill 给出其他默认路径，以本仓库规则覆盖。

## 流程

1. Intake：检查 Agent-Skills 和 Superpowers 是否可用。
2. Define：使用 Agent-Skills `/spec` 和 Superpowers brainstorming。
3. Plan：使用 Agent-Skills `/plan` 和 Superpowers writing-plans。
4. Isolate：使用 Superpowers worktree 指南。
5. Build：使用 Superpowers TDD 和 Agent-Skills incremental implementation。
6. Debug：使用 Superpowers systematic debugging。
7. Review：使用 Agent-Skills code-review-and-quality。
8. Simplify：使用 Agent-Skills code-simplification。
9. Security：安全敏感改动运行 security-and-hardening。
10. Verify：使用 Superpowers verification-before-completion。
11. Ship 或 handoff：使用 Agent-Skills shipping 和 Superpowers branch completion。

## 审批边界

计划批准只授权本地执行。push、merge、PR 创建、发布、部署、凭据修改、第三方资源修改、付费任务和破坏性数据操作仍然需要用户明确批准。

## 非目标

- 不把泛用兼容流程作为主要产品。
- 不把 Agent-Skills 或 Superpowers 当作可有可无的装饰。
- 不在任意能力包缺失时静默继续执行。
