# Dev Workflow 确认与留档选择实施计划

> **给自动化执行者：** 按任务逐项执行；如果运行时支持，优先使用 `superpowers:subagent-driven-development`，否则使用 `superpowers:executing-plans` 或内联执行。任务使用 checkbox (`- [ ]`) 追踪。

**目标：** 让 `dev-workflow` 先在对话中确认 spec/plan 内容，再按用户选择决定是否写入 `docs/`，避免默认留档或自动进入实现。

**架构：** 保持 `skills/dev-workflow/SKILL.md` 为唯一 workflow 行为源。`README.md` 负责用户可见说明，`CLAUDE.md` 负责维护者约束，`evals/evals.json` 负责覆盖确认与留档选择的行为回归。

**技术栈：** Markdown skill、Markdown docs、JSON eval prompts、Node.js 本地验证命令。

---

## 文件映射

- `skills/dev-workflow/SKILL.md`：修改 Define/Plan 和文档规则，加入 spec 内容确认、留档选择、plan 内容确认的顺序约束。
- `README.md`：更新关键规则和示例流程，说明 spec/plan 是内容确认点，留档是单独选择。
- `CLAUDE.md`：更新维护约定和本地验证命令，避免继续要求所有 spec/plan 默认写入 `docs/`。
- `evals/evals.json`：新增或更新 eval，覆盖用户不留档时不得写入 `docs/`、用户留档时才写入固定路径、plan 前后确认边界。
- `docs/dev-workflow/spec.md`：保持已确认的 spec 作为留档，不再修改，除非执行中发现计划和 spec 冲突。
- `docs/dev-workflow/plan.md`：本实施计划，plan 获批后作为执行依据。

## 任务 1：更新 dev-workflow skill 的确认与留档门禁

**依赖：** 无

**范围：** 修改 `skills/dev-workflow/SKILL.md`

- [ ] 将 `Define And Plan` 改为三步顺序：输出 spec 草案并等待确认、询问是否留档、输出 plan 草案并等待确认。
- [ ] 修改 `Spec And Plan Document Rules`：只有用户选择留档时才写入 `docs/<feature>/spec.md` 和 `docs/<feature>/plan.md`。
- [ ] 明确如果 spec 选择不留档，本次 workflow 后续 plan 也不留档。
- [ ] 明确 spec 内容批准只授权询问留档并创建 plan，不授权实现。
- [ ] 明确 plan 内容批准才授权本地实现、测试、review、simplify 和 verify。
- [ ] 移除或改写“approved spec”“approved implementation plan”等可能暗示写入即批准的表述。

**验收标准：**

- [ ] `skills/dev-workflow/SKILL.md` 包含 `spec draft` 或等价规则。
- [ ] `skills/dev-workflow/SKILL.md` 包含 `archive` 或等价规则。
- [ ] `skills/dev-workflow/SKILL.md` 包含 spec 不留档时 plan 也不留档的规则。
- [ ] `skills/dev-workflow/SKILL.md` 没有要求所有 spec/plan 默认写入 `docs/` 的表述。

**验证命令：**

```bash
node -e 'const s=require("fs").readFileSync("skills/dev-workflow/SKILL.md","utf8"); for (const c of ["spec draft","archive","do not write the later plan"]) { if(!s.includes(c)) throw new Error("missing "+c); } console.log("skill gates ok")'
```

## 任务 2：更新用户文档和维护者说明

**依赖：** 任务 1

**范围：** 修改 `README.md`、`CLAUDE.md`

- [ ] 在 `README.md` 的关键规则中说明 spec 和 plan 是内容确认点，不是默认落盘步骤。
- [ ] 在 `README.md` 中说明 spec 内容确认后必须询问是否留档。
- [ ] 在 `README.md` 中说明不留档时 spec 和 plan 都不写入 `docs/`。
- [ ] 调整 `README.md` 中所有“Spec 和 Plan 必须中文落盘”的绝对表述，改为“选择留档时必须中文并使用固定路径”。
- [ ] 更新 `CLAUDE.md` 的 conventions，避免维护者继续把所有 spec/plan 默认写入 `docs/`。
- [ ] 更新 `CLAUDE.md` 的 local validation，使检查目标从默认路径变为确认与留档规则。

**验收标准：**

- [ ] `README.md` 清楚说明用户可以选择不留档。
- [ ] `README.md` 清楚说明 plan 是否留档继承 spec 的留档选择。
- [ ] `CLAUDE.md` 不再要求每个 spec/plan 无条件写入 `docs/<feature>/`。

**验证命令：**

```bash
rg -n "必须中文落盘|必须写入 `docs/<feature>|默认写入|无条件" README.md CLAUDE.md
node -e 'const r=require("fs").readFileSync("README.md","utf8"); if(!r.includes("留档")) throw new Error("README missing archive wording"); const c=require("fs").readFileSync("CLAUDE.md","utf8"); if(!c.includes("archive")) throw new Error("CLAUDE missing archive wording"); console.log("docs wording ok")'
```

## 任务 3：更新 eval 行为覆盖

**依赖：** 任务 1

**范围：** 修改 `evals/evals.json`

- [ ] 新增 eval：用户要求使用 dev-workflow，spec 内容确认后选择不留档，期望不创建 `docs/<feature>/spec.md`，后续 plan 也只在对话中确认。
- [ ] 新增或更新 eval：用户选择留档时，spec 写入 `docs/<feature>/spec.md`，plan 获批后写入 `docs/<feature>/plan.md`。
- [ ] 更新已有 expected output 中“Any spec or plan documents are written...”这类默认留档表述。
- [ ] 保留缺失 Superpowers、security-sensitive change、trivial-doc-edit 等既有覆盖。

**验收标准：**

- [ ] `evals/evals.json` 是合法 JSON。
- [ ] eval 覆盖留档和不留档两个路径。
- [ ] eval 不再把生成 spec/plan 与写入 docs 绑定为默认行为。

**验证命令：**

```bash
node -e 'const e=JSON.parse(require("fs").readFileSync("evals/evals.json","utf8")); const text=JSON.stringify(e); if(!text.includes("no archive")) throw new Error("missing no-archive eval"); if(!text.includes("chooses archive")) throw new Error("missing archive eval"); console.log("eval archive coverage ok")'
```

## 任务 4：执行整体验证和简化检查

**依赖：** 任务 1、任务 2、任务 3

**范围：** 验证整个仓库的文档、JSON 和 workflow 表述一致性。

- [ ] 运行 JSON 解析检查。
- [ ] 运行 skill frontmatter 和关键短语检查。
- [ ] 搜索冲突表述：默认写入、必须落盘、写入即 approved、plan approval 被提前授权。
- [ ] 检查变更 diff，确认没有无关文件改动。
- [ ] 做一次简化 pass：删除重复解释，保留最小但明确的规则。

**验收标准：**

- [ ] 所有验证命令通过。
- [ ] `git diff --stat` 只包含本次 workflow 行为相关文件。
- [ ] 没有安全门禁触发项：未涉及认证、支付、PII、公开 API、权限、凭据、依赖或部署。

**验证命令：**

```bash
node -e 'JSON.parse(require("fs").readFileSync("plugin.json","utf8")); JSON.parse(require("fs").readFileSync("evals/evals.json","utf8")); console.log("json ok")'
node -e 'const s=require("fs").readFileSync("skills/dev-workflow/SKILL.md","utf8"); for (const c of ["spec draft","archive","plan draft"]) { if(!s.includes(c)) throw new Error("missing "+c); } console.log("skill wording ok")'
rg -n "Any spec or plan documents are written|必须中文落盘|必须写入 `docs/<feature>|approved spec|approved implementation plan" README.md CLAUDE.md skills/dev-workflow/SKILL.md evals/evals.json
git diff --stat
```

## 风险和缓解

| 风险 | 影响 | 缓解 |
|------|------|------|
| 上游 Superpowers brainstorming 默认要求写 spec 到自己的路径 | 中 | 在 `dev-workflow` 中声明仓库规则覆盖上游默认：先对话确认，再按用户选择留档 |
| README、CLAUDE、skill 之间措辞不一致 | 中 | 使用 `rg` 搜索旧表述，并用 Node 关键短语检查 |
| eval 仍然期待默认写入 docs | 中 | 更新 existing eval expected output，并新增留档/不留档路径 |
| 规则太松导致 plan 自动进入实现 | 高 | 明确 plan 内容确认前不得进入 Build/TDD/实现；plan 内容确认后才授权本地执行 |

## 执行顺序

1. 更新 `skills/dev-workflow/SKILL.md`。
2. 运行任务 1 的 skill gates 检查。
3. 更新 `README.md` 和 `CLAUDE.md`。
4. 运行任务 2 的文档措辞检查。
5. 更新 `evals/evals.json`。
6. 运行任务 3 的 eval 覆盖检查。
7. 运行任务 4 的整体验证。
8. 根据 diff 做简化 pass。
9. 再次运行整体验证。
10. 停下并汇报验证结果，不执行 push、发布或外部动作。
