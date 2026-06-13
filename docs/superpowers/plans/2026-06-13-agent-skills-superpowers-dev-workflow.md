# Agent-Skills + Superpowers Dev Workflow Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Recenter `dev-workflow` on Agent-Skills plus Superpowers.

**Architecture:** Keep the shipped skill small and explicit: Agent-Skills owns lifecycle quality gates, Superpowers owns execution discipline, and dev-workflow orchestrates their handoffs.

**Tech Stack:** Markdown skills, JSON eval prompts, Claude Code plugin manifest, shell validation commands.

---

## Task 1: Recenter Documentation

- [ ] Rewrite `README.md` in Chinese around Agent-Skills + Superpowers as the main dependencies.
- [ ] Include Claude Code install commands for Superpowers and Agent-Skills.
- [ ] Include Codex install guidance for Superpowers plugin and Agent-Skills copied into `.agents/skills` or `~/.agents/skills`.
- [ ] Remove broad compatibility fallback as the primary story.

## Task 2: Recenter The Skill

- [ ] Rewrite `skills/dev-workflow/SKILL.md` so missing Agent-Skills or Superpowers triggers a setup gate.
- [ ] Define phase ownership for Agent-Skills and Superpowers.
- [ ] Preserve explicit approval boundaries for external actions.

## Task 3: Update Metadata And Evals

- [ ] Update `plugin.json` and `.claude-plugin/plugin.json`.
- [ ] Update `evals/evals.json` to test the combined-capability workflow and missing-capability behavior.
- [ ] Update `CLAUDE.md` maintenance guidance.

## Task 4: Verify

- [ ] Run JSON parsing checks.
- [ ] Run skill frontmatter check.
- [ ] Run eval schema check.
- [ ] Search for stale generic-runtime language.
