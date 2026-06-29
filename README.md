# claude-hermes-connect

A Claude Code skill for configuring and operating a remote Hermes instance over SSH as a real MCP server.

## Repository contents

- `.claude/skills/claude-hermes-connect/SKILL.md` — the skill definition
- `.claude/skills/claude-hermes-connect/evals/evals.json` — starter eval prompts for iterating on the skill

## What the skill covers

This skill helps Claude Code:

- configure `remote-hermes` as a real MCP server entry
- guide passwordless SSH setup when needed
- maintain a durable Hermes role profile so Claude can infer when to use Hermes later
- support ongoing Hermes roles such as reminders, tech-news summaries, personal knowledge base work, and other long-term helper tasks

## Local-only files

This repository may contain local development files that should not be committed. In particular, keep these out of the published history:

- `.claude/settings.local.json`
- `.claude/skills/claude-hermes-connect-workspace/`
- `.claude/remote-hermes-profile.json`
- `.idea/`
