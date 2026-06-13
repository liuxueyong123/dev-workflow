# Dev Workflow 确认与留档选择设计

## 目标

让 `dev-workflow` 在 spec 和 plan 确认点尊重用户选择：先把内容输出给用户确认，确认后再询问是否需要留档；只有用户选择留档时才写入 `docs/`。

## 背景

`dev-workflow` 已经要求组合 Agent-Skills 和 Superpowers：

- Agent-Skills 提供生命周期质量门禁：spec、plan、增量实现、测试、review、简化、security、performance、documentation、CI/CD、git workflow、shipping。
- Superpowers 提供执行纪律：skill selection、brainstorming、written implementation plans、TDD、systematic debugging、worktree isolation、subagent-driven execution、verification before completion、development-branch finishing。

现有文档已经把 Define 和 Plan 的 exit gate 写成用户批准，但还不够显式。执行者可能把写入 `spec.md` 或 `plan.md` 误读为文档已经批准，或者把“写入 docs”当成固定步骤，导致用户不想留档时仍然产生文档。

## 需求

1. 先在对话中输出 spec 草案供用户确认，不默认写入 `docs/`。
2. 用户明确批准 spec 内容前，不得进入 Plan、Build、TDD、实现、review、verify 或 ship 阶段。
3. 用户批准 spec 内容后，必须询问用户是否需要留档。
4. 只有用户选择留档时，才把 spec 写入 `docs/<feature>/spec.md`。
5. 如果用户选择不留档，spec 不写入 `docs/`，并且本次 workflow 后续 plan 也不留档。
6. Spec 内容确认完成后，继续进入 Plan 阶段。
7. Plan 阶段先在对话中输出 plan 草案供用户确认。
8. 用户明确批准 plan 内容前，不得进入 Isolate、Build、TDD、实现、review、simplify、verify 或 ship 阶段。
9. 只有本次 workflow 已选择留档时，plan 才写入 `docs/<feature>/plan.md`；如果 spec 留档选择是不留档，plan 也不写入 `docs/`。
10. “用户要求继续”“继续吧”“确认”“批准”“按这个做”等明确回复可以视为内容批准；沉默、模糊反馈、未解决修改意见不能视为批准。
11. 如果用户要求修改 spec 或 plan，必须先输出修订版，再重新停下请求确认。
12. 只有没有生成 spec 或 plan 的 trivial 改动可以跳过这些确认与留档选择门禁。

## 文档规则

如果用户选择留档，Agent-Skills、Superpowers 或 dev-workflow 生成的 spec 和 plan 文档必须满足：

1. 文档正文使用中文；技术标识、文件路径、命令、API 名称和引用文本可以保留原语言。
2. 每个功能使用一个简短 kebab-case 名称作为 `<feature>`。
3. spec 固定写入 `docs/<feature>/spec.md`。
4. plan 固定写入 `docs/<feature>/plan.md`。
5. 如果上游 skill 给出其他默认路径，以本仓库规则覆盖。
6. 如果文档已经存在，使用新的完整版本覆盖更新，不创建 dated duplicates。

## 流程

1. Intake：检查 Agent-Skills 和 Superpowers 是否可用。
2. Define：使用 Agent-Skills `/spec` 和 Superpowers brainstorming，在对话中输出 spec 草案并等待用户确认。
3. Archive choice：spec 内容获批后，询问用户是否需要留档；需要时写入 `docs/<feature>/spec.md`，不需要时记录本次 workflow 不留档。
4. Plan：只有 spec 内容获批后，才使用 Agent-Skills `/plan` 和 Superpowers writing-plans，在对话中输出 plan 草案并等待用户确认；如果本次 workflow 已选择留档，则在 plan 获批后写入 `docs/<feature>/plan.md`。
5. Isolate：使用 Superpowers worktree 指南。
6. Build：使用 Superpowers TDD 和 Agent-Skills incremental implementation。
7. Debug：使用 Superpowers systematic debugging。
8. Review：使用 Agent-Skills code-review-and-quality。
9. Simplify：使用 Agent-Skills code-simplification。
10. Security：安全敏感改动运行 security-and-hardening。
11. Verify：使用 Superpowers verification-before-completion。
12. Ship 或 handoff：使用 Agent-Skills shipping 和 Superpowers branch completion。

## 审批边界

Spec 内容批准只授权询问是否留档并创建 plan，不授权实现。

Plan 批准只授权本地执行，包括本地编辑、测试、lint/typecheck/build、review、simplify 和 verify。push、merge、PR 创建、发布、部署、凭据修改、第三方资源修改、付费任务和破坏性数据操作仍然需要用户明确批准。

## 验收标准

1. `skills/dev-workflow/SKILL.md` 明确要求先在对话中输出 spec 草案并等待用户确认。
2. `skills/dev-workflow/SKILL.md` 明确要求 spec 内容获批后再询问是否留档。
3. `skills/dev-workflow/SKILL.md` 明确说明只有用户选择留档时才写入 `docs/<feature>/spec.md`。
4. `skills/dev-workflow/SKILL.md` 明确要求先在对话中输出 plan 草案并等待用户确认。
5. `skills/dev-workflow/SKILL.md` 明确说明 spec 选择不留档时，plan 也不写入 `docs/`。
6. `README.md` 的关键规则说明 spec 和 plan 是内容确认点，留档是单独选择。
7. `evals/evals.json` 至少包含一个覆盖 spec/plan 确认和留档选择的 eval。
8. 本地验证命令通过，且搜索不到和新规则冲突的默认留档或自动继续表述。

## 非目标

- 不把泛用兼容流程作为主要产品。
- 不把 Agent-Skills 或 Superpowers 当作可有可无的装饰。
- 不在任意能力包缺失时静默继续执行。
- 不要求 trivial 文档拼写修复必须创建 spec 或 plan。
