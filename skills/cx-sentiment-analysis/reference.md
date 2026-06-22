# CX Sentiment Analysis — Reference

## Status

**v1.0 candidate — not production-validated.** This rubric was synthesized by an AI Agent Council (OpenRouter Fusion) and has not been calibrated against real ticket data. Before production use, follow [panel/production-roadmap.md](../../panel/production-roadmap.md).

## Provenance

This skill was produced by merging three independent panel model outputs (Claude Opus, GPT Latest, Gemini Pro Latest) via OpenRouter Fusion panel → judge → synthesizer.

| Document | Location |
|---|---|
| Panel outputs | [panel/outputs/](../../panel/outputs/) |
| Fusion verdict | [panel/verdict.md](../../panel/verdict.md) |
| Comparative review | [panel/comparative-review.md](../../panel/comparative-review.md) |
| Production roadmap | [panel/production-roadmap.md](../../panel/production-roadmap.md) |
| Panel brief (reproduce) | [prompts/panel-brief.md](../../prompts/panel-brief.md) |

## Key stress-test scenarios

The rubric was designed to handle cases that break single-score sentiment:

| Scenario | Why canonical scoring fails |
|---|---|
| Polite customer, total prod outage | Calm tone masks critical impact and churn risk |
| Polite churn threat | Churn language without anger |
| Angry customer, cosmetic bug | High valence, low business impact |
| Resigned customer after long effort | Low arousal, high churn signal |
| Sarcasm ("Great, another outage") | Lexical positivity, negative meaning |
| ESL / terse customer | Style confound with frustration |

## Installation

Source files live at `skills/cx-sentiment-analysis/` in this repo. Copy the folder into your agent's skills directory:

| Agent | Project path | Global path |
|---|---|---|
| Cursor | `.cursor/skills/cx-sentiment-analysis/` | `~/.cursor/skills/cx-sentiment-analysis/` |
| Claude Code | `.claude/skills/cx-sentiment-analysis/` | `~/.claude/skills/cx-sentiment-analysis/` |

```bash
cp -r skills/cx-sentiment-analysis .cursor/skills/   # or .claude/skills/
```

See [skills/README.md](../README.md) for full install commands.
