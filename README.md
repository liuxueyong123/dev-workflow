# Dev Workflow

`dev-workflow` 是一个把 **Agent-Skills** 和 **Superpowers** 串起来的开发流程 skill。

它的定位很明确：

- **Agent-Skills** 提供生产级工程门禁：`/spec`、`/plan`、`/build`、`/test`、`/review`、`/code-simplify`、`/ship`，以及 spec、任务拆解、代码审查、安全、性能、发布等 lifecycle skills。
- **Superpowers** 提供执行纪律：brainstorming、writing-plans、TDD、系统化调试、worktree 隔离、subagent-driven development、verification-before-completion、finishing-a-development-branch。
- **dev-workflow** 负责把两者编排成一个连续流程，避免只规划不执行、只写代码不审查、只跑测试不做安全和发布门禁。

Claude Code 和 Codex 都可以参与，但它们只是运行入口；主能力来自 Agent-Skills + Superpowers。

## 总体流程

```text
Intake (能力可用性检查)
  -> Define        Agent-Skills /spec + Superpowers brainstorming
  -> Plan          Agent-Skills /plan + Superpowers writing-plans
  -> Isolate       Superpowers worktree / task isolation
  -> Build         Superpowers TDD + Agent-Skills incremental implementation
  -> [Debug]       Superpowers systematic debugging（仅在构建或测试失败时触发）
  -> Review ─┐     Agent-Skills code-review-and-quality
             ├───  （两者并行，互不依赖）
  -> Security ┘    Agent-Skills security-and-hardening（安全敏感改动触发）
  -> Simplify      Agent-Skills code-simplification
  -> Verify        Superpowers verification-before-completion
  -> Ship/Handoff  Agent-Skills shipping + Superpowers finishing branch
```

## 两套能力如何分工

| 阶段 | 主能力 | 辅助能力 | 目标 |
|------|--------|----------|------|
| Intake | 能力可用性检查 | — | 确认 Agent-Skills 和 Superpowers 均已安装，否则阻断 |
| Define | Agent-Skills `spec-driven-development` / `/spec` | Superpowers `brainstorming` | 明确范围、验收标准、边界 |
| Plan | Agent-Skills `planning-and-task-breakdown` / `/plan` | Superpowers `writing-plans` | 拆成小任务、文件路径、验证步骤 |
| Isolate | Superpowers `using-git-worktrees` | Agent-Skills `git-workflow-and-versioning` | 隔离改动，避免污染主工作区 |
| Build | Superpowers `test-driven-development` | Agent-Skills `incremental-implementation` | 红绿重构，小步提交 |
| Debug（条件） | Superpowers `systematic-debugging` | Agent-Skills `debugging-and-error-recovery` | 构建或测试失败时先找根因再修复 |
| Review ∥ Security | Review: Agent-Skills `code-review-and-quality` / `/review` | Superpowers `requesting-code-review` | 五维审查，修复 CRITICAL/HIGH |
| | Security: Agent-Skills `security-and-hardening` | 项目安全检查命令 | 认证、支付、PII、API、权限等必须过门 |
| Simplify | Agent-Skills `code-simplification` / `/code-simplify` | Superpowers verification loop | 删除重复和过度设计 |
| Verify | Superpowers `verification-before-completion` | Agent-Skills `/test` | 用最新证据证明完成 |
| Ship | Agent-Skills `shipping-and-launch` / `/ship` | Superpowers `finishing-a-development-branch` | PR、发布、回滚计划和交付说明 |

## 关键规则

1. **Agent-Skills 和 Superpowers 是主依赖**：如果其中一个缺失，不要静默降级成普通 checklist；先提示安装，或让用户明确同意降级执行。
2. **Spec 和 Plan 是手动确认点**：非平凡功能必须先明确 spec 和 plan，用户批准后再执行。
3. **计划批准只覆盖本地执行**：本地编辑、测试、review、simplify、verify 可以继续推进。
4. **外部动作必须再次确认**：push、merge、PR、发布、部署、凭据修改、第三方资源修改、破坏性数据操作都必须显式审批。
5. **TDD 优先采用 Superpowers**：Agent-Skills 也有测试能力，但红绿重构纪律以 Superpowers 为主。
6. **质量门禁优先采用 Agent-Skills**：代码审查、简化、安全、发布门禁以 Agent-Skills 为主。
7. **完成声明必须有验证证据**：最终报告前必须运行 fresh verification，并说明命令和结果。

## 适用场景

| 任务 | 推荐流程 |
|------|----------|
| 拼写、格式、轻量文档 | 直接编辑 + Superpowers verification |
| 单文件行为变更 | 简短 spec + Superpowers TDD + focused Agent-Skills review |
| 多文件功能 | `/spec` -> `/plan` -> Superpowers TDD execution -> `/review` -> `/code-simplify` -> verify |
| Bug 修复 | Superpowers systematic debugging -> regression test -> fix -> Agent-Skills review |
| 认证、支付、PII、公开 API、权限 | 完整流程 + Agent-Skills security-and-hardening |
| 发布或上线 | Agent-Skills shipping-and-launch + Superpowers finishing-a-development-branch |

## 安装到 Claude Code

### 1. 安装 Superpowers

优先使用 Claude Code 官方 marketplace：

```text
/plugin install superpowers@claude-plugins-official
```

如果需要使用 Superpowers 自己的 marketplace：

```text
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

### 2. 安装 Agent-Skills

```text
/plugin marketplace add addyosmani/agent-skills
/plugin install agent-skills@addy-agent-skills
```

如果 SSH clone 有问题，可以用 HTTPS：

```text
/plugin marketplace add https://github.com/addyosmani/agent-skills.git
/plugin install agent-skills@addy-agent-skills
```

安装后应能使用这些入口：

```text
/spec
/plan
/build
/test
/review
/code-simplify
/ship
```

### 3. 安装 dev-workflow

个人全局安装：

```bash
mkdir -p ~/.claude/skills
cp -R skills/dev-workflow ~/.claude/skills/dev-workflow
```

项目内安装：

```bash
mkdir -p .claude/skills
cp -R /path/to/dev-workflow/skills/dev-workflow .claude/skills/dev-workflow
```

本地插件调试：

```bash
claude --plugin-dir /path/to/dev-workflow
```

如果用插件方式加载，修改后在 Claude Code 内执行：

```text
/reload-plugins
```

### 4. Claude Code 使用方式

推荐直接调用：

```text
/dev-workflow 用 Agent-Skills + Superpowers 实现 OAuth 登录
```

或者从 Agent-Skills lifecycle command 开始：

```text
/spec 构建 OAuth 登录；后续用 dev-workflow 串联 Superpowers 执行和 Agent-Skills 门禁
```

## 安装到 Codex

Codex 也能使用 SKILL.md，但安装路径和 Claude Code 不同。

### 1. 安装 Superpowers

在 Codex CLI 中打开插件界面：

```text
/plugins
```

搜索：

```text
superpowers
```

选择 `Install Plugin`。

Codex App 中可以在侧边栏 `Plugins` 里找到 `Superpowers` 并安装。

### 2. 安装 Agent-Skills

Agent-Skills 的 skills 是普通 `SKILL.md` 目录，Codex 可从 `.agents/skills` 或 `~/.agents/skills` 读取。

个人全局安装：

```bash
git clone https://github.com/addyosmani/agent-skills.git /tmp/agent-skills
mkdir -p ~/.agents/skills
cp -R /tmp/agent-skills/skills/* ~/.agents/skills/
```

项目内安装：

```bash
git clone https://github.com/addyosmani/agent-skills.git /tmp/agent-skills
mkdir -p .agents/skills
cp -R /tmp/agent-skills/skills/* .agents/skills/
```

### 3. 安装 dev-workflow

个人全局安装：

```bash
mkdir -p ~/.agents/skills
cp -R skills/dev-workflow ~/.agents/skills/dev-workflow
```

项目内安装：

```bash
mkdir -p .agents/skills
cp -R /path/to/dev-workflow/skills/dev-workflow .agents/skills/dev-workflow
```

重启 Codex，或等待 Codex 自动检测 skill 变化。

### 4. 推荐 AGENTS.md 配置

在项目 `AGENTS.md` 中加入：

```markdown
## Development Workflow

- 功能开发、Bug 修复、重构和安全敏感改动优先使用 `$dev-workflow`。
- `$dev-workflow` 必须组合 Agent-Skills 和 Superpowers，而不是退化成普通 checklist。
- Agent-Skills 负责 spec、plan、review、code-simplify、security、ship 门禁。
- Superpowers 负责 brainstorming、writing-plans、TDD、systematic-debugging、worktree、verification。
- 如果 Agent-Skills 或 Superpowers 缺失，先提示安装；只有用户明确同意才允许降级执行。
- push、merge、PR、发布、部署、凭据或第三方资源变更前必须先询问用户。
```

### 5. Codex 使用方式

显式调用：

```text
$dev-workflow 用 Agent-Skills + Superpowers 实现用户分页接口
```

自然语言调用：

```text
使用 dev-workflow。先用 Agent-Skills 做 spec/plan，再用 Superpowers TDD 执行，最后用 Agent-Skills review/security/ship 门禁。
```

如果 `$` 菜单里 skill 名带插件前缀，以 Codex 显示为准。

## 推荐执行顺序

### 普通功能

```text
Agent-Skills /spec
-> Agent-Skills /plan
-> Superpowers writing-plans
-> Superpowers TDD / subagent-driven execution
-> Agent-Skills /review + /security（并行，安全门禁仅在敏感改动时触发）
-> Agent-Skills /code-simplify
-> Superpowers verification-before-completion
-> handoff or Agent-Skills /ship
```

### Bug 修复

```text
Superpowers systematic-debugging
-> regression test
-> Superpowers TDD fix
-> Agent-Skills debugging-and-error-recovery if needed
-> Agent-Skills /review
-> Superpowers verification-before-completion
```

### 安全敏感改动

```text
Agent-Skills /spec
-> Agent-Skills /plan
-> Superpowers TDD execution
-> Agent-Skills /review  ──┐
                           ├──（并行执行）
-> Agent-Skills /security ─┘
-> Agent-Skills /code-simplify
-> Superpowers verification-before-completion
-> explicit approval before ship/deploy
```

## 验证这个仓库

检查 JSON：

```bash
node -e "JSON.parse(require('fs').readFileSync('plugin.json','utf8')); JSON.parse(require('fs').readFileSync('.claude-plugin/plugin.json','utf8')); JSON.parse(require('fs').readFileSync('evals/evals.json','utf8')); console.log('json ok')"
```

检查 skill frontmatter：

```bash
node -e "const s=require('fs').readFileSync('skills/dev-workflow/SKILL.md','utf8'); if(!/^---\nname: dev-workflow\ndescription: /m.test(s)) throw new Error('frontmatter missing'); console.log('frontmatter ok')"
```

检查 eval 数量和基本字段：

```bash
node -e "const e=JSON.parse(require('fs').readFileSync('evals/evals.json','utf8')); if(e.evals.length!==5) throw new Error('expected 5 evals'); for (const x of e.evals) { if(!x.id||!x.prompt||!x.expected_output||!Array.isArray(x.files)) throw new Error('bad eval '+(x.id||'<missing>')); } console.log('eval schema ok')"
```

## 官方参考

- Superpowers：<https://github.com/obra/superpowers>
- Superpowers marketplace：<https://github.com/obra/superpowers-marketplace>
- Agent-Skills：<https://github.com/addyosmani/agent-skills>
- Claude Code skills：<https://code.claude.com/docs/en/skills>
- Claude Code plugins：<https://code.claude.com/docs/en/plugins>
- Codex Agent Skills：<https://developers.openai.com/codex/skills>

## License

MIT
