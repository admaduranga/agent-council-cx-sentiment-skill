# Panel Run — OpenRouter Fusion POC

This directory contains the raw outputs and analysis from a single AI Agent Council run on June 22, 2025.

## Models used

| Role | Model |
|---|---|
| Panel | Claude Opus Latest |
| Panel | OpenAI GPT Latest |
| Panel | Google Gemini Pro Latest |
| Judge | Fusion judge model |
| Synthesizer | Claude Opus 4.8 |

Panel models had web search and web fetch enabled.

## Fusion workflow

1. **Panel** — the same expert brief ([`../prompts/panel-brief.md`](../prompts/panel-brief.md)) sent to all three models in parallel
2. **Judge** — compares panel outputs; surfaces consensus, contradictions, coverage gaps, unique insights, blind spots
3. **Synthesizer** — writes the final merged artifact

## Files

| File | Description |
|---|---|
| [`outputs/claude-opus-latest.md`](outputs/claude-opus-latest.md) | Claude's independent panel response |
| [`outputs/openai-gpt-latest.md`](outputs/openai-gpt-latest.md) | GPT's independent panel response |
| [`outputs/gemini-pro-latest.md`](outputs/gemini-pro-latest.md) | Gemini's independent panel response |
| [`verdict.md`](verdict.md) | Fusion synthesizer final verdict |
| [`comparative-review.md`](comparative-review.md) | Follow-up: which model produced the best outcome |
| [`production-roadmap.md`](production-roadmap.md) | Follow-up: phased plan for production readiness |

Panel outputs are **raw, unedited** responses as returned by each model.

## Further reading

- [OpenRouter Fusion blog post](https://openrouter.ai/blog/announcements/fusion-beats-frontier/)
- [OpenRouter Fusion Router docs](https://openrouter.ai/docs/guides/routing/routers/fusion-router)
