# Panel Brief — SaaS Support Sentiment SKILL.md Review

Use this prompt with OpenRouter Fusion (or any multi-model panel setup) to reproduce the AI Agent Council review described in this repository.

---

## Expert cast

Act as a panel of senior experts in:

- Customer Experience (CX)
- SaaS Technical Support
- Quality Assurance (QA)
- Sentiment Analysis
- Behavioral Psychology
- Evaluation Framework Design

---

## Task

Act as a panel of senior experts in CX, SaaS Technical Support, Quality Assurance, Sentiment Analysis, Behavioral Psychology, and Evaluation Framework Design.

Stress-test the proposed design for a `SKILL.md` that analyzes SaaS support cases — what every CX team needs to score customer sentiment consistently across thousands of tickets.

Find what would make it unshippable. Then produce a production-ready version that fixes what you found.

No explicit framework is attached. Reconstruct the **canonical design** that most CX QA programs and bolt-on "AI sentiment" helpdesk features actually use (a single 1–5 or Positive/Neutral/Negative label based primarily on customer tone), critique it rigorously, and replace it with a measurably better alternative.

---

## Failure modes to probe

Actively look for:

- Flaws and ambiguities
- Missing dimensions
- Bias risks
- Overlapping criteria
- Weak scoring definitions
- Edge cases
- Anything that could reduce consistency, repeatability, explainability, or cross-model agreement

Do not settle for generic criticism. The specificity of the failure list is what makes the critique load-bearing.

---

## Expected deliverable

1. A rigorous multi-expert critique of the canonical framework
2. Stress tests showing how the rebuilt framework behaves on hard cases (polite outage, polite churn threat, angry minor issue, resigned customer, sarcasm, ESL/terse, multi-stakeholder threads)
3. A complete, production-ready `SKILL.md` with anchored scales, evidence requirements, deterministic scoring rules, and strict JSON output schema

The synthesized skill from this POC lives at [`skills/cx-sentiment-analysis/SKILL.md`](../skills/cx-sentiment-analysis/SKILL.md).

---

## Follow-up prompts (optional)

After the council returns its verdict, run these as separate follow-up turns:

1. **Comparative review:** "From the council, who provided the best outcome?"
2. **Production roadmap:** "What are the next steps to improve this SKILL.md before using in production?"

---

## Models used in this POC

| Role | Model |
|---|---|
| Panel | Claude Opus Latest, OpenAI GPT Latest, Google Gemini Pro Latest |
| Judge | Fusion judge model (structured deliberation) |
| Synthesizer | Claude Opus 4.8 |

Panel models had web search and web fetch enabled. See [OpenRouter Fusion docs](https://openrouter.ai/docs/guides/routing/routers/fusion-router) for setup.

---

## Cost-efficient panel variant

This POC used frontier models on the panel. For lower cost, OpenRouter's [DRACO benchmarks](https://openrouter.ai/blog/announcements/fusion-beats-frontier/) show a **budget panel** (e.g. Gemini 3 Flash + Kimi K2.6 + DeepSeek V4 Pro) fused by a frontier synthesizer can approach solo frontier performance at ~50% of the cost. Same brief, same judge/synthesizer architecture — swap the panel tier.

See [README § Why fuse instead of one flagship model?](../README.md#why-fuse-instead-of-one-flagship-model) for economics, benchmark table, and disclaimers.
