# Skills

Portable agent skills from the AI Agent Council POC. Copy into your agent's skills directory — the folder structure is identical for Cursor and Claude Code.

## `cx-sentiment-analysis`

Six-dimension SaaS support sentiment and churn-risk rubric (v1.0 candidate). See [cx-sentiment-analysis/SKILL.md](cx-sentiment-analysis/SKILL.md).

## Install

Copy the skill folder into your project or global agent skills path:

```bash
# Cursor — project
mkdir -p .cursor/skills
cp -r skills/cx-sentiment-analysis .cursor/skills/

# Cursor — global
mkdir -p ~/.cursor/skills
cp -r skills/cx-sentiment-analysis ~/.cursor/skills/

# Claude Code — project
mkdir -p .claude/skills
cp -r skills/cx-sentiment-analysis .claude/skills/

# Claude Code — global
mkdir -p ~/.claude/skills
cp -r skills/cx-sentiment-analysis ~/.claude/skills/
```

Then invoke the skill by name (`cx-sentiment-analysis`) when analyzing support tickets.

> **Note:** Skills in this repo are source copies. Agents only load skills from `.cursor/skills/` or `.claude/skills/` (project or home directory) — not from `skills/` at the repo root directly.

## Provenance

This skill was synthesized via OpenRouter Fusion (panel → judge → synthesizer). See [panel/](../panel/) and [reference.md](cx-sentiment-analysis/reference.md).
