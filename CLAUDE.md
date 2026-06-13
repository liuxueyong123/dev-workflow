# Dev Workflow

`dev-workflow` is a skill-only plugin that coordinates Agent-Skills quality gates with Superpowers execution discipline across Claude Code and Codex.

## Agent Rules

- Keep workflow logic in `skills/dev-workflow/SKILL.md`.
- Keep `dev-workflow` explicit-invocation only: `dev-workflow`, `$dev-workflow`, `/dev-workflow`, or `dev-flow`.
- Do not broaden docs or metadata so this skill auto-triggers for ordinary coding, bug fixes, refactors, cleanup, or sensitive changes.
- Require both Agent-Skills and Superpowers. If either is missing, ask for installation or explicit degraded-mode approval.
- Put new workflow behavior in `skills/` first. Update `commands/` only for legacy shim or cross-harness compatibility.
- Present spec and plan drafts in the conversation first. Archive only after user approval, in Chinese, under `docs/<feature>/spec.md` and `docs/<feature>/plan.md`.
- Preserve approval boundaries for push, merge, PR creation, publishing, deployment, credentials, paid jobs, and third-party resources.
- Add or update eval prompts when workflow behavior changes.

## Key Paths

```text
skills/dev-workflow/SKILL.md   - Core skill definition
evals/evals.json               - Behavior prompts for skill evaluation
docs/<feature>/spec.md         - Optional archived spec, Chinese
docs/<feature>/plan.md         - Optional archived plan, Chinese
.claude-plugin/plugin.json     - Claude Code plugin manifest
plugin.json                    - Plugin metadata
README.md                      - User-facing documentation
```

## Validation

Run relevant checks after edits:

```bash
node -e "JSON.parse(require('fs').readFileSync('plugin.json','utf8')); JSON.parse(require('fs').readFileSync('evals/evals.json','utf8')); console.log('json ok')"
node -e "const s=require('fs').readFileSync('skills/dev-workflow/SKILL.md','utf8'); if(!s.includes('Explicit Invocation Only')) throw new Error('missing explicit gate'); for (const c of ['Use for '+'feature work','bug '+'fixes','refac'+'tors','security-'+'sensitive']) { if(s.includes(c)) throw new Error('broad trigger remains: '+c); } console.log('explicit trigger gate ok')"
node -e "const s=require('fs').readFileSync('skills/dev-workflow/SKILL.md','utf8'); for (const c of ['spec draft','archive','Do not archive spec or plan documents by default']) { if(!s.includes(c)) throw new Error('missing '+c); } console.log('archive gates ok')"
```

Before handoff, review `git diff` and search changed user-facing docs for stale claims about unavailable slash commands, unrestricted automation, or default spec/plan archiving.
