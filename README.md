# Dev Workflow

`dev-workflow` 是一个显式调用的开发流程 skill，用来把 **Agent-Skills** 的质量门禁和 **Superpowers** 的执行纪律串成一条端到端工作流。

它不替代 Agent-Skills 或 Superpowers，也不是普通任务的默认流程。只有用户明确要求使用 `dev-workflow`、`$dev-workflow`、`/dev-workflow` 或 `dev-flow` 时才启用。

## 解决什么问题

- 避免只写 spec、不执行。
- 避免只写代码、不 review。
- 避免只跑测试、不做安全和发布门禁。
- 避免 Agent-Skills 与 Superpowers 各自为政，缺少统一阶段状态。

Claude Code 和 Codex 都可以运行本 skill；核心能力来自 Agent-Skills + Superpowers。

## 触发规则

正向显式调用时必须使用本 workflow，例如：

```text
/dev-workflow 用 Agent-Skills + Superpowers 实现 OAuth 登录
Use dev-workflow to fix a flaky unit test
按 dev-flow 处理这个多文件重构
```

没有显式点名时，不要自动启用：

```text
实现 OAuth 登录
修复这个测试
用 Agent-Skills 和 Superpowers 做一下
```

负向或说明性提及也不要当作执行调用：

```text
不要使用 dev-workflow
解释一下 dev-workflow 是什么
更新 dev-workflow skill 文档
```

## 流程概览

```text
Intake    检查 Agent-Skills 和 Superpowers 是否可用
Define    产出 spec 草案，用户确认
Archive   spec 确认后询问是否留档
Plan      产出 plan 草案，用户确认
Isolate   隔离工作区或确认当前工作区安全
Build     TDD / 小步实现
Debug     仅在构建、测试或行为异常时触发
Review    代码质量审查，修复 CRITICAL/HIGH
Security  安全敏感改动时触发
Simplify  必跑，处理 review 遗留问题和过度复杂度
Verify    fresh verification 后才能报告完成
Ship      仅在用户批准后 push / PR / 发布 / 部署
```

## 能力分工

| 阶段     | Agent-Skills                             | Superpowers                      |
| -------- | ---------------------------------------- | -------------------------------- |
| Define   | `spec-driven-development` / `/spec`      | `brainstorming`                  |
| Plan     | `planning-and-task-breakdown` / `/plan`  | `writing-plans`                  |
| Isolate  | `git-workflow-and-versioning`            | `using-git-worktrees`            |
| Build    | `incremental-implementation`             | `test-driven-development`        |
| Debug    | `debugging-and-error-recovery`           | `systematic-debugging`           |
| Review   | `code-review-and-quality` / `/review`    | `requesting-code-review`         |
| Security | `security-and-hardening`                 | verification support             |
| Simplify | `code-simplification` / `/code-simplify` | verification support             |
| Verify   | `/test` where available                  | `verification-before-completion` |
| Ship     | `shipping-and-launch` / `/ship`          | `finishing-a-development-branch` |

## 关键约束

- Agent-Skills 和 Superpowers 都是主依赖；缺失时不要静默降级成普通 checklist。
- 非平凡任务先在对话中确认 spec，再确认 plan。
- spec 确认后单独询问是否留档；只有用户选择留档才写入 `docs/<feature>/spec.md` 和 `docs/<feature>/plan.md`。
- 留档文档正文使用中文；技术标识、路径、命令、API 名称和引用文本可以保留原语言。
- 计划批准只授权本地编辑、测试、review、simplify 和 verify。
- push、merge、PR、发布、部署、凭据修改、付费任务、第三方资源变更和破坏性数据操作都必须再次获得明确批准。
- Review 后必须进入 Simplify；Simplify 没有跳过条件。
- 没有 fresh verification evidence，不允许报告完成。

## 安装到 Claude Code

安装 Superpowers：

```text
/plugin install superpowers@claude-plugins-official
```

安装 Agent-Skills：

```text
/plugin marketplace add addyosmani/agent-skills
/plugin install agent-skills@addy-agent-skills
```

安装 dev-workflow：

```text
/plugin marketplace add liuxueyong123/dev-workflow
/plugin install dev-workflow@dev-workflow
```

插件文件更新后，在 Claude Code 中执行：

```text
/reload-plugins
```

## 安装到 Codex

安装 Superpowers：

```text
/plugins
```

在插件界面搜索并安装 `Superpowers`。

安装 Agent-Skills：

```bash
git clone https://github.com/addyosmani/agent-skills.git /tmp/agent-skills
mkdir -p ~/.agents/skills
cp -R /tmp/agent-skills/skills/* ~/.agents/skills/
```

安装 dev-workflow：

1. 在 Codex CLI 执行 `/plugins`，或在 Codex App 侧边栏打开 `Plugins`。
2. 添加 marketplace：`liuxueyong123/dev-workflow`。
3. 安装 `dev-workflow`。

如果 Codex 显示的 skill 名带插件前缀，以界面显示为准。

## 推荐 AGENTS.md 片段

```markdown
## Development Workflow

- 只有用户明确点名 `dev-workflow`、`$dev-workflow`、`/dev-workflow` 或 `dev-flow` 并要求使用该流程时，才启用本 skill。
- 一旦用户正向显式调用，必须先切入 dev-workflow，再执行任务本身。
- 普通功能开发、Bug 修复、重构和安全敏感改动不能自动触发 dev-workflow。
- `dev-workflow` 必须组合 Agent-Skills 和 Superpowers，而不是退化成普通 checklist。
- Spec 和 plan 先在对话中确认；spec 确认后询问用户是否留档。
- 只有用户选择留档时，spec/plan 才写入 `docs/<feature>/spec.md` 与 `docs/<feature>/plan.md`，并且正文必须使用中文。
- 如果 Agent-Skills 或 Superpowers 缺失，先提示安装；只有用户明确同意才允许降级执行。
- push、merge、PR、发布、部署、凭据或第三方资源变更前必须先询问用户。
```

## 使用示例

Claude Code：

```text
/dev-workflow 实现 OAuth 登录
```

Codex：

```text
$dev-workflow 实现用户分页接口
```

自然语言：

```text
使用 dev-workflow 修复这个 flaky test
```

## 仓库结构

```text
skills/dev-workflow/SKILL.md   核心 skill
evals/evals.json               行为回归用例
docs/dev-workflow/             已归档的 spec / plan
.claude-plugin/plugin.json     Claude Code 插件 manifest
plugin.json                    通用插件元数据
README.md                      用户说明
CLAUDE.md                      仓库内 agent 维护规则
```

## 验证

检查 JSON：

```bash
node -e "JSON.parse(require('fs').readFileSync('plugin.json','utf8')); JSON.parse(require('fs').readFileSync('.claude-plugin/plugin.json','utf8')); JSON.parse(require('fs').readFileSync('evals/evals.json','utf8')); console.log('json ok')"
```

检查 skill frontmatter：

```bash
node -e "const s=require('fs').readFileSync('skills/dev-workflow/SKILL.md','utf8'); if(!/^---\nname: dev-workflow\ndescription: /m.test(s)) throw new Error('frontmatter missing'); console.log('frontmatter ok')"
```

检查 eval 基本字段：

```bash
node -e "const e=JSON.parse(require('fs').readFileSync('evals/evals.json','utf8')); const ids=new Set(); for (const x of e.evals) { if(!x.id||!x.prompt||!x.expected_output||!Array.isArray(x.files)) throw new Error('bad eval '+(x.id||'<missing>')); if(ids.has(x.id)) throw new Error('duplicate eval '+x.id); ids.add(x.id); } console.log('eval schema ok')"
```

检查显式触发规则：

```bash
node -e "const s=require('fs').readFileSync('skills/dev-workflow/SKILL.md','utf8'); if(!s.includes('Explicit Invocation Only')) throw new Error('missing explicit gate'); if(!s.includes('Invocation Compliance Gate')) throw new Error('missing invocation compliance gate'); for (const c of ['Use for '+'feature work','bug '+'fixes','refac'+'tors','security-'+'sensitive']) { if(s.includes(c)) throw new Error('broad trigger remains: '+c); } console.log('explicit trigger gate ok')"
```

检查 spec/plan 确认与留档规则：

```bash
node -e "const s=require('fs').readFileSync('skills/dev-workflow/SKILL.md','utf8'); for (const c of ['spec draft','archive','Do not archive spec or plan documents by default']) { if(!s.includes(c)) throw new Error('missing '+c); } console.log('archive gates ok')"
```

## 参考

- Superpowers: <https://github.com/obra/superpowers>
- Superpowers marketplace: <https://github.com/obra/superpowers-marketplace>
- Agent-Skills: <https://github.com/addyosmani/agent-skills>
- Claude Code skills: <https://code.claude.com/docs/en/skills>
- Claude Code plugins: <https://code.claude.com/docs/en/plugins>
- Codex Agent Skills: <https://developers.openai.com/codex/skills>

## License

MIT
