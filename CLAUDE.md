# Dev Workflow

A skill-only plugin that coordinates Agent-Skills quality gates with Superpowers execution discipline across Claude Code and Codex.

## Project Structure

```text
skills/dev-workflow/SKILL.md   - Core skill definition
evals/evals.json               - Behavior prompts for skill evaluation
docs/<feature>/spec.md         - Chinese feature spec for workflow changes
docs/<feature>/plan.md         - Chinese implementation plan for workflow changes
.claude-plugin/plugin.json     - Claude Code plugin manifest
plugin.json                    - Plugin metadata
README.md                      - User-facing documentation
```

## Conventions

- Keep workflow logic in `skills/dev-workflow/SKILL.md`.
- Treat Agent-Skills and Superpowers as the primary capability dependencies.
- Do not silently downgrade missing Agent-Skills or Superpowers into a generic checklist; ask for install or explicit degraded-mode approval.
- Treat slash commands as optional wrappers for this repo, but document Agent-Skills lifecycle commands as expected upstream capabilities.
- Keep every Agent-Skills or Superpowers spec/plan document in Chinese under `docs/<feature>/spec.md` or `docs/<feature>/plan.md`.
- Preserve approval boundaries for push, merge, PR creation, publishing, deployment, credentials, paid jobs, and third-party resources.
- Add or update eval prompts when changing workflow behavior.

## Local Validation

Run these checks after editing:

```bash
node -e "JSON.parse(require('fs').readFileSync('plugin.json','utf8')); JSON.parse(require('fs').readFileSync('evals/evals.json','utf8')); console.log('json ok')"
node -e "const s=require('fs').readFileSync('skills/dev-workflow/SKILL.md','utf8'); for (const c of ['docs/<feature>/spec.md','docs/<feature>/plan.md','Chinese']) { if(!s.includes(c)) throw new Error('missing '+c); } console.log('doc routing ok')"
```

Also search changed user-facing docs for stale claims that imply unavailable slash commands or unrestricted automation.

## Publishing

1. Review `git diff`.
2. Run local validation.
3. Commit with a conventional commit message.
4. Push or publish only after explicit user approval.
