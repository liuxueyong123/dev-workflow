# Dev Workflow

`dev-workflow` 是一个显式调用的开发流程 skill，把 **Agent-Skills** 的质量门禁、独立 plan 审查 skill（`grill-with-docs`）和 **Superpowers** 的执行纪律串成一条端到端工作流。Claude Code 和 Codex 均可运行。

它不会替代 Agent-Skills、独立 skills 或 Superpowers，也不是普通任务的默认流程——只有用户明确点名 `dev-workflow`、`/dev-workflow`、`dev-flow` 时才启用。

## 为什么需要 dev-workflow

Agent-Skills 提供了一组高质量的单步技能（interview、spec、plan、review、simplify、security、ship），`grill-with-docs`（依赖 `grilling`、`domain-modeling`）是独立 skill，用于在 plan 草稿产出后审查文档对齐、API 正确性和版本兼容性；Superpowers 提供严格的执行纪律（TDD、debugging、brainstorming、worktree、verification）。这些能力各自独立使用时缺少统一阶段编排——什么时候先澄清意图、什么时候写 spec、什么时候拷问 plan、什么时候 review、review 完是否必须 simplify，这些衔接逻辑正是 dev-workflow 承担的职责。

| 问题 | dev-workflow 如何解决 |
|------|----------------------|
| 写了 spec 不执行，或跳过 spec 直接写代码 | Interview → Define → Plan → Build 串行，每步有 Hard Gate |
| 需求意图不清晰就开始写 spec | Interview 阶段先用 `interview-me` 驱动置信度 ≥ 95%，产出 confirmed intent summary，再进入 spec |
| plan 缺少文档对齐和深度拷问 | Plan 中用 `grill-with-docs` 审查 plan 草稿，修订后再确认 |
| 写了代码不 review | Review 是必跑阶段，不可跳过 |
| review 发现问题不处理 | Simplify 紧随 Review，强制处理所有发现 |
| spec 确认后直接开始实现，不询问留档 | Gate 2（spec 确认）和 Gate 3（留档询问）分离 |
| 输出 spec/plan 后不等用户确认就开始写代码 | 五个 Hard Gate，每个都要求 STOP 等待 |
| Agent-Skills 和 Superpowers 各自为政 | 每个阶段指定主次 skill，按统一节奏推进 |

## 流程概览

```text
Intake    检查 Agent-Skills、独立 skills（grill-with-docs、grilling、domain-modeling）和 Superpowers 是否可用
Interview 用 interview-me 澄清意图，驱动置信度 ≥ 95% → 🔴 Gate 1 用户确认 intent summary
Define    基于确认的 intent summary 写 spec（纯行为，不含实现细节）→ 🔴 Gate 2 用户确认 → 🔴 Gate 3 询问留档
Plan      产出 plan 草稿 → 用 grill-with-docs 审查 → 修订 → 🔴 Gate 4 用户确认
Isolate   隔离工作区；greenfield 项目可跳过
Build     按已确认的 spec/plan 实现；默认 subagent 并行执行（每个 subagent 内部走 TDD）
Debug     仅在构建、测试或行为异常时触发
Review    代码审查，分析 correctness / readability / architecture / security / performance
Security  安全敏感改动时与 Review 并行运行
Simplify  必跑 — 处理 review 全部遗留问题，消除死代码和重复
Verify    fresh verification，无证据不报告完成
Ship      🔴 Gate 5 用户明确批准后 push / PR / merge / deploy
```

## 阶段追踪

每个阶段必须创建 TodoWrite 任务，工作流结束前全部标记为 `completed` 或给出跳过理由。以下是各阶段的跳过条件：

| 阶段 | 何时可跳过 |
|------|-----------|
| Intake | 不可跳过 |
| Interview | 仅限拼写/纯格式/元数据文本调整 |
| Define | 仅限拼写/纯格式/元数据文本调整 |
| Plan | 仅限拼写/纯格式/元数据文本调整 |
| Isolate | greenfield 项目无已有分支时 |
| Build | 不可跳过 |
| Debug | 未遇到失败时 |
| Review | 不可跳过 |
| Security | 无安全触发时 |
| Simplify | **不可跳过** — Review 后强制执行 |
| Verify | 不可跳过 |
| Ship | 用户未要求 merge/deploy/publish 时 |

## 硬门禁

五个 Hard Gate 是工作流中最关键的机制——每个 Gate 要求输出草稿后 STOP，等待用户明确确认才能继续。跳过或合并任何 Gate 属于工作流违规。

| Gate | 阶段      | 何时触发          | 要求用户做什么            | 被阻塞直到                     |
|------|-----------|-------------------|---------------------------|--------------------------------|
| 1    | Interview | intent summary 输出后 | 确认意图总结准确完整       | 用户明确说"可以""没问题""ok"    |
| 2    | Define    | spec 草稿输出后      | 确认 spec 内容正确         | 用户明确说"可以""没问题""ok"    |
| 3    | Define    | spec 被确认后        | 选择是否留档到 docs/       | 用户明确说"留档"或"不留档"      |
| 4    | Plan      | plan 草稿输出后      | 确认 plan 可以执行         | 用户明确说"可以""执行吧""go"    |
| 5    | Ship      | 任何外部操作前       | 明确请求 push/merge/deploy | 用户明确说出操作名称            |

### Gate 协议

1. 输出完整草稿（或操作摘要）
2. 提出明确的确认问题
3. **STOP** — 不进入下一步，不写代码，不调用下一个 skill
4. **等待** — 用户的下一条消息就是 Gate 回应，不预判
5. 收到明确确认后才继续

### 常见违规

- 输出 spec 后立刻开始实现（正确做法：spec → STOP → 等确认 → 问留档）
- 一次消息同时要 spec 确认和留档（正确做法：先确认 spec，再单独问留档）
- 输出 plan 后立刻创建文件（正确做法：plan → STOP → 等确认 → 实现）
- 跳过留档询问直接进入 plan（正确做法：spec 确认 → 问留档 → plan）
- Interview 置信度不足就进入 Define（正确做法：interview-me 迭代到 ≥ 95%，Gate 1 确认后再写 spec）

## Agent-Skills、独立 Skills 与 Superpowers 分工

每个阶段的退出条件由对应的 skill 保证。Primary 负责主要产出，Secondary 负责辅助支撑。`grill-with-docs` 不是 Agent-Skills；它是独立 skill（依赖 `grilling` 和 `domain-modeling`），在 Intake 阶段一并检查可用性。

| 阶段     | Primary                                                    | Secondary                        | 退出条件                                        |
| -------- | ---------------------------------------------------------- | -------------------------------- | ----------------------------------------------- |
| Intake   | 能力检查（Agent-Skills、独立 skills、Superpowers）           | —                                | 所需 skills 均确认可用，或用户明确同意降级        |
| Interview | `interview-me`（Agent-Skills）                             | `brainstorming`（Superpowers）   | 置信度 ≥ 95%；GUESS markers 全部消除；Gate 1 确认 intent summary |
| Define   | `spec-driven-development`（Agent-Skills）                   | `brainstorming`（Superpowers）   | 基于确认的 intent summary 产出行为级 spec（无实现细节）；Gate 2 确认 + Gate 3 留档 |
| Plan     | `planning-and-task-breakdown` / `/plan`（Agent-Skills）；`grill-with-docs`（独立 skill） | `writing-plans`（Superpowers） | plan 草稿经 grill-with-docs 审查并修订；Gate 4 确认 plan |
| Isolate  | `using-git-worktrees`（Superpowers）                        | `git-workflow-and-versioning`（Agent-Skills） | 工作区安全                           |
| Build    | `subagent-driven-development`（Superpowers，默认，subagent 内部 TDD）；`test-driven-development`（单任务/写范围重叠时） | `incremental-implementation`（Agent-Skills） | 按已确认 spec/plan 实现；TDD 通过 |
| Debug    | `systematic-debugging`（Superpowers）                       | `debugging-and-error-recovery`（Agent-Skills） | 根因修复 + 回归测试                   |
| Review   | `code-review-and-quality` / `/review`（Agent-Skills）       | `requesting-code-review`（Superpowers） | CRITICAL/HIGH 修复；其余捕获给 Simplify   |
| Security | `security-and-hardening`（Agent-Skills）                    | 项目 audit 命令                   | 安全发现已处理或接受                             |
| Simplify | `code-simplification` / `/code-simplify`（Agent-Skills）    | `verification-before-completion`（Superpowers） | 全部发现已处理（修复/推迟/拒绝）；测试通过 |
| Verify   | `verification-before-completion`（Superpowers）             | `/test`（Agent-Skills）           | fresh evidence 已收集                            |
| Ship     | `shipping-and-launch` / `/ship`（Agent-Skills）             | `finishing-a-development-branch`（Superpowers） | 用户批准的外部操作                    |

## 触发规则

### 正向显式调用（必须启用 dev-workflow）

```text
/dev-workflow 实现 OAuth 登录
Use dev-workflow to fix a flaky test
按 dev-flow 处理多文件重构
用 dev-workflow 修这个 bug
```

一旦用户正向调用，即使任务本身是日常编码、Bug 修复、重构或安全敏感改动，也必须走完整流程。

### 不应自动启用的情况

```text
实现 OAuth 登录              ← 普通开发请求
修复这个测试                  ← 普通 Bug 修复
用 Agent-Skills 做 code review ← 只点名了单个 skill
```

### 负向或说明性提及（不触发）

```text
不要使用 dev-workflow
解释 dev-workflow 是什么
更新 dev-workflow skill
为什么 dev-workflow 没运行
```

## 调用合规门禁

正向调用被识别后，必须执行以下步骤，不可跳过：

1. 停住任何普通编码、调试或 review 路径
2. 输出工作更新：`Using dev-workflow.`
3. 检查能力门禁（Agent-Skills 和 Superpowers 是否可用），然后创建阶段追踪任务，**之后才能编辑文件**
4. 完成或显式跳过每个必要阶段，最终报告前全部收束
5. **遵守每个 Hard Gate** — 每个 Gate 必须 STOP 等待用户确认，绝不跳过或合并

## 适用范围（Scope Rules）

不同的改动类型对应不同的工作流深度，非平凡任务不跳过 Define 和 Plan。

| 改动类型 | 工作流深度 |
|----------|-----------|
| 拼写错误、纯格式文档、元数据文本调整 | 直接编辑 + review + simplify + 验证 |
| 单文件行为变更 | 完整 spec + plan + TDD + review + simplify + 验证 |
| 多文件功能或重构 | 完整 spec + plan + TDD + review + simplify + 验证 |
| Bug 修复 | 完整 spec + plan + 系统调试 + 回归测试 + TDD 修复 + review + simplify + 验证 |
| 认证/支付/PII/密钥/权限/公开 API/数据删除 | 完整流程 + security-and-hardening |
| 发布/部署/上线 | shipping-and-launch + 分支完成 |

## 关键约束

- **无降级**：Agent-Skills、独立 skill（`grill-with-docs`、`grilling`、`domain-modeling`）或 Superpowers 缺失时停住，告知用户缺少什么。仅在用户明确要求时降级执行。
- **确认先行**：非平凡任务不跳过 Define 和 Plan；spec 和 plan 先在对话中确认再继续。
- **留档可选**：spec 确认后单独询问留档，不默认写入文件。用户选留档才写 `docs/<feature>/spec.md` 和 `docs/<feature>/plan.md`。若 `docs/<feature>/` 已存在，自动后缀 `-2`、`-3`，绝不静默覆盖已有目录。
- **中文正文**：留档文档正文使用中文。技术标识、文件路径、命令名、API 名和引用原文可保留原语言。
- **Review/Simplify 必跑**：即使是拼写、纯格式文档或元数据文本调整，也必须跑 Review 和 Simplify。Simplify 检查清单：所有 IMPORTANT 发现已处理；无超过 50 行的函数新增；无死代码；无重复逻辑；全部测试通过。
- **安全门禁触发**：认证、支付、PII、密钥、公开 API、数据库查询、文件上传、数据删除、CI/部署变更均触发安全审查。CRITICAL/HIGH 发现阻塞 Ship。
- **Subagent 后仍需完整流程**：Subagent 完成 ≠ 工作流完成。使用 subagent 后必须继续跑 Review、Simplify 和 Verify。
- **批准边界**：Plan 批准只授权本地编辑、测试、lint/typecheck/build、review、simplify 和 verify。push、merge、PR、发布、部署、凭据修改、第三方资源变更、破坏性数据操作均须再次批准。
- **验证证据**：无 fresh verification evidence 不报告完成。
- **完成门禁**：最终报告前每个阶段任务必须为 `completed` 或已给出跳过理由。Simplify 不可跳过。

## 依赖

dev-workflow 编排以下 skill，安装前需确保它们均可用（Intake 阶段会逐一检查）：

| 类别 | Skill | 来源 | 说明 |
|------|-------|------|------|
| Agent-Skills | `interview-me`, `spec-driven-development`, `planning-and-task-breakdown`, `code-review-and-quality`, `code-simplification`, `security-and-hardening`, `shipping-and-launch` 等 | [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) | 各阶段的主要执行 skill |
| Superpowers | `brainstorming`, `test-driven-development`, `subagent-driven-development`, `systematic-debugging`, `using-git-worktrees`, `verification-before-completion`, `writing-plans`, `finishing-a-development-branch` | [obra/superpowers](https://github.com/obra/superpowers) | 执行纪律与工程实践 |
| 独立 skill | `grill-with-docs` | 团队分发 / 本机 skill 目录 | Plan 草稿的文档对齐审查（Plan 阶段必跑） |
| 独立 skill | `grilling` | 团队分发 / 本机 skill 目录 | `grill-with-docs` 的依赖，提供交互式拷问能力 |
| 独立 skill | `domain-modeling` | 团队分发 / 本机 skill 目录 | `grill-with-docs` 的依赖，提供领域术语一致性检查 |

## 安装

### Claude Code

```text
# 1. 安装 Superpowers
/plugin install superpowers@claude-plugins-official

# 2. 安装 Agent-Skills
/plugin marketplace add addyosmani/agent-skills
/plugin install agent-skills@addy-agent-skills

# 3. 安装 dev-workflow
/plugin marketplace add liuxueyong123/dev-workflow
/plugin install dev-workflow@dev-workflow
```

`grill-with-docs`、`grilling`、`domain-modeling` 需手动放入 Claude Code 的 user skills 目录：

```bash
mkdir -p ~/.claude/skills
cp /path/to/grill-with-docs.md ~/.claude/skills/
cp /path/to/grilling.md ~/.claude/skills/
cp /path/to/domain-modeling.md ~/.claude/skills/
```

更新后执行 `/reload-plugins`。

### Codex

Superpowers 和 dev-workflow 在 `/plugins` 界面搜索安装。Agent-Skills 需手动安装：

```bash
git clone https://github.com/addyosmani/agent-skills.git /tmp/agent-skills
mkdir -p ~/.agents/skills
cp -R /tmp/agent-skills/skills/* ~/.agents/skills/
```

`grill-with-docs`、`grilling`、`domain-modeling` 同样放入 `~/.agents/skills/`。如 Codex 显示的 skill 名带插件前缀，以界面显示为准。

## 仓库结构

```text
skills/dev-workflow/SKILL.md   核心 skill 定义
evals/evals.json               行为回归用例
docs/dev-workflow/             已归档的 spec / plan
.claude-plugin/plugin.json     Claude Code 插件 manifest
.claude-plugin/marketplace.json 插件市场目录
plugin.json                    通用插件元数据
README.md                      用户说明（本文件）
CLAUDE.md                      仓库维护规则
```

## 验证

```bash
# JSON 格式
node -e "JSON.parse(require('fs').readFileSync('plugin.json','utf8')); JSON.parse(require('fs').readFileSync('.claude-plugin/plugin.json','utf8')); console.log('json ok')"

# Skill frontmatter
node -e "const s=require('fs').readFileSync('skills/dev-workflow/SKILL.md','utf8'); if(!/^---\nname: dev-workflow/m.test(s)) throw new Error('frontmatter missing'); console.log('frontmatter ok')"

# 硬门禁存在
node -e "const s=require('fs').readFileSync('skills/dev-workflow/SKILL.md','utf8'); for (const c of ['Hard Gates','GATE 1','GATE 2','GATE 3','STOP. Do NOT proceed']) { if(!s.includes(c)) throw new Error('missing '+c); } console.log('hard gates ok')"

# 显式触发规则
node -e "const s=require('fs').readFileSync('skills/dev-workflow/SKILL.md','utf8'); if(!s.includes('Explicit Invocation Only')) throw new Error('missing explicit gate'); console.log('explicit trigger gate ok')"
```

## 参考

- Superpowers: <https://github.com/obra/superpowers>
- Agent-Skills: <https://github.com/addyosmani/agent-skills>
- Claude Code skills: <https://code.claude.com/docs/en/skills>
- Claude Code plugins: <https://code.claude.com/docs/en/plugins>
- Codex Agent Skills: <https://developers.openai.com/codex/skills>

## License

MIT
