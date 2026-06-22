# The AI Agent Council: How to Stress-Test an Idea Before You Build It

*Send your idea to a jury of models, let them deliberate, and have a foreman deliver the verdict. That disagreement is the whole point.*

---

A court case decided by a single judge in chambers arrives with the weight of one perspective — no deliberation, no cross-examination, no dissent. The same case heard by a jury of twelve, each bringing their own lens, and a foreman charged with weighing the disagreement into a single verdict? That's a different institution. Not because the judge is less smart, but because the judgment is *held to the test of multiple independent perspectives before it's allowed to become final*.

The same is true for ideas. The best single model in the world is still one perspective. The interesting question is what happens when you force the judgment to survive a council.

This is the fourth piece in my series on building with Agentic AI. The first three were about doing the work. This one's about pressure-testing the work *before* you commit to it. Every meaningful idea I build now starts with something I call a **Planner Panel** (my nickname for the pattern the industry is starting to call the **AI Agent Council** — a jury of models, a judge to weigh the deliberation, a verdict you can ship).

Here's what I built, what I ran, and what a weekend-shaped experiment on OpenRouter Fusion taught me about finding expensive mistakes three minutes into a prompt instead of three months into a rollout.

---

## The POC: an idea in front of a jury

The pattern I'm calling the AI Agent Council isn't new. It's the same instinct that gave us *multi-agent debate*, *Chain of Debates*, and *LLM-as-a-judge* in the research literature, now productised by platforms like OpenRouter under names like Fusion, and packaged as skills like the AI Agent Council on MCP Market.

What *is* new is how cheap it's gotten to actually run.

OpenRouter's Fusion API is the cleanest version of this I've used. You send one prompt. Fusion dispatches it **in parallel** to a configurable panel of models — in my case Claude Opus Latest, OpenAI GPT Latest, and Google Gemini Pro Latest. Each panel model gets web search and web fetch enabled, answers independently, and hands its output back. Then a **judge model** compares the panel's answers and produces a structured deliberation — *consensus, contradictions, coverage gaps, unique insights, blind spots*. A final **synthesizer** (in my POC, that's Claude Opus 4.8 — a larger model that can read all three outputs in full) takes that structured analysis and writes the final answer.

That's the whole architecture: **panel → judge → synthesizer → verdict.** It's a jury box with a foreman.

I tested it on the design of a `SKILL.md` for analyzing SaaS support cases — what every CX team needs to score customer sentiment consistently across thousands of tickets. The brief was the same one I sent to the panel: *act as a panel of senior experts in CX, SaaS Technical Support, Quality Assurance, Sentiment Analysis, Behavioral Psychology, and Evaluation Framework Design. Stress-test the proposed design. Find what would make it unshippable. Then produce a production-ready version that fixes what you found.*

Three models. Same brief. Three different `SKILL.md` files. Then a judge model. Then a final verdict.

The result is what the rest of this piece is about.

---

## What an AI Agent Council actually is

The pattern reduces to four moving parts, and the jury analogy maps to each one:

- **The brief.** What the case is, what the failure modes are, what a verdict looks like. *(This is the indictment, the instructions to the jury, and the verdict form.)*
- **The panel.** Multiple models, each playing the same expert cast, answering independently. *(The jurors. Each brings their own lens. None knows what the others said until the foreman reads it.)*
- **The judge.** Reads all panel outputs, surfaces *consensus, contradictions, unique insights, blind spots*. *(The foreman, calling the room to order.)*
- **The synthesizer.** Takes the structured deliberation and writes the final artifact. *(The verdict, delivered.)*

The cast — the expert roles the panel plays — is the load-bearing part. "Be a critical reviewer" is one generic voice with one blind spot. "Act as a panel of senior experts in CX, SaaS Support, QA, Sentiment Analysis, Behavioral Psychology, and Evaluation Framework Design" is six distinct angles, each pointing at a different way the artifact could be wrong. The same way a jury is selected for the case at hand, not at random.

Then the same brief goes to multiple models. Not one. Three. Maybe five if you're cost-conscious (more on that below).

---

## Why even the best models will disagree — and why that's the point

This is the part the marketing never tells you.

I sent the same six-expert panel prompt to the three top-tier reasoning models from each major provider. I expected roughly aligned outputs with stylistic differences. I got something more useful.

**They agreed on the headline finding.** All three independently surfaced the same critical flaw in the canonical sentiment-analysis design: it conflates four completely different constructs (emotional tone, satisfaction, business impact, churn risk) into one number. That unanimous finding is the signal that it's *real*, not a model being weird. When three independent juries all return the same verdict on the obvious, the obvious is obvious.

**They disagreed on the architecture.** GPT built a mechanically tight scoring engine with ordered precedence rules and explicit numeric caps — the kind of thing you can ship tomorrow. Claude built a diagnostically sharper version with a named inter-rater reliability target (Krippendorff's α ≥ 0.80) and the observation that *resignation is a louder churn signal than anger* — genuinely the sharpest single idea in the whole exercise. Gemini built the most readable version with a clean three-dimensional state matrix (Tone × Friction × Commercial Risk), but the least production-ready.

**Each brought something the others missed.** GPT's hallucination-fail trigger (*churn_risk=4 with no cancellation language in evidence = automatic fail*). Claude's determinism mechanics (temperature 0, idempotency hashing). Gemini's vivid failure case ("the stoic churner — thank you for the update, unfortunately we're migrating to a competitor, best of luck").

**None of these was complete on its own.** Together, they were obvious. The disagreement wasn't a problem to resolve — it was the *deliberation*, the part of a jury's work that actually earns the verdict.

The pattern, then, isn't *one model plays the council*. It's *multiple models play the council independently, then a judge reads all outputs and surfaces the structure of the disagreement, then a synthesizer writes the final artifact by taking the best of each.* That middle step — the judge — is the part most people skip, and it's the part that turns a noisy multi-model run into a load-bearing review.

This works precisely *because* the models disagree. If they all gave identical outputs, you'd have paid for three slow single-model runs. The disagreement is what makes the council a council.

### How the three council members actually stacked up

When the same panel ran the same brief, the differences were sharp enough to score:

| Criterion | Claude Opus | GPT Latest | Gemini Pro |
|---|---|---|---|
| Critique depth / challenges assumptions | **Excellent** | Excellent | Good (vivid but lighter) |
| Multi-dimensional model | 6 dims, no composite | 5 dims **+ deterministic final score** | 3 dims + actionability |
| **Determinism (the hard part)** | Strong (temp 0, idempotency) | **Strongest** (full precedence + caps + checklist) | Weak (no precedence math) |
| Auditability / evidence rules | Strong | **Strongest** (per-field + QA-fail triggers) | Good |
| Worked examples with JSON | Few | **5 full examples** | Schema only |
| Edge-case coverage | Broad | **Broadest** | Selective |
| Brevity / elegance | Medium | Verbose | **Best** |

If you had to ship one artifact tomorrow, **GPT's version was the most production-ready** — mechanically tight, full of numeric caps and precedence rules, five worked JSON examples, and a hallucination-fail trigger that would catch confident nonsense before it shipped. **Claude's version was the most rigorous** — the diagnostics, the determinism mechanics, and the sharpest single observation of the whole exercise. **Gemini's version was the most quotable** — cleanest mental model, but the least ready to run.

Each model was best at *one thing*. That's exactly why you need all three.

---

## The polite customer your sentiment model is about to miss

The unanimous finding from my panel is worth pulling out, because it's the kind of expensive mistake that's invisible until it isn't.

The canonical design scores a single 1–5 sentiment label per case, mostly from tone. The flaw it creates:

| Scenario | Tone | Satisfaction | Churn risk | Business impact |
|---|---|---|---|---|
| Polite customer, total prod outage | Calm | Neutral | **Critical** | **Critical** |
| Angry customer, cosmetic UI bug | Very negative | Low | Low | Trivial |
| Resolved but took 3 weeks | Relieved | Low | High | Medium |

A framework that scores the polite outage customer a **5/5** and the angry-cosmetic-bug customer a **1/5** will route the *wrong* cases to retention. It will systematically under-protect your most valuable, most stoic enterprise accounts — the ones who churn quietly, in perfect grammar, without raising their voice. That's the **polite-customer-during-an-outage problem**, and every juror on the council caught it.

The same finding, if it had leaked into production and misrouted retention cases for a quarter, would have cost orders of magnitude more to find and fix than the three minutes it took to surface across a panel of three models. *That's* the verdict the council delivered that a single model wouldn't have.

---

## The follow-up question is the second trial

After the council returned and the three candidate artifacts landed, I asked two more questions — the second trial most people forget to schedule:

> *"From the council, who provided the best outcome?"*
>
> *"What are the next steps to improve this SKILL.md before using in production?"*

The first forced an explicit verdict on the council itself (GPT on shipping-readiness, Claude on governance, Gemini on clarity — each given credit for what it actually did best). The second turned the whole exercise from a design review into a productionization roadmap: close the open gaps, build a stratified gold set of ~200 real cases, calibrate inter-rater agreement per dimension, run a bias and fairness audit, do determinism and robustness testing, then shadow-deploy before guarded rollout.

That roadmap is the actual deliverable. The `SKILL.md` is the artifact. The AI Agent Council produced both — because asking *"what's next?"* after the verdict is the part that turns a review into a release.

---

## If you're cost-conscious: smaller models, more of them

Running three top-tier reasoning models on every idea gets expensive fast. The pattern I'm testing next is the cost-efficient variant — and I now have third-party evidence it works.

**OpenRouter's own Fusion benchmarks** show that lighter panel combinations can match a frontier model at a fraction of the cost. A panel of Gemini 3 Flash, Kimi K2.6, and DeepSeek V4 Pro — three *smaller* models, not flagship — ran at roughly **half the cost of Claude Fable 5** while landing within 1% of its benchmark score. The diversity gain came from having multiple independent perspectives, not from each perspective being the smartest possible model.

The shape of the cost-efficient council is the same:

1. Run the same expert-panel prompt across several smaller, cheaper models (5 is a good number, picked from different providers).
2. Use a single judge model — doesn't have to be the biggest, just a clean context window — to surface consensus, contradictions, and unique insights.
3. Use a synthesizer (typically a larger model like Opus 4.8, which has the reasoning budget to weigh all panel outputs) to write the final artifact.

I haven't validated this exact pattern in production yet. But the underlying mechanism — disagreement is information, and synthesis is where the value lives — is the same one Fusion's own numbers already validated. The frontier-model panel is the *premium* tier of the council, not the only tier.

---

## How to run your own

If you want to do this seriously:

**1. Pick the cast deliberately.** Six roles is the sweet spot for a design review. Fewer is shallow; more is theatre. For technical support work, mine are CX, SaaS Support, QA, Sentiment Analysis, Behavioral Psychology, and Evaluation Framework Design. For a different problem, change the cast to match — security, legal, finance, SRE, customer success, whatever the work actually touches.

**2. Define the failure modes in the brief.** Don't say *"be critical."* Say *"actively look for flaws, ambiguities, missing dimensions, bias risks, overlapping criteria, weak scoring definitions, edge cases, and anything that could reduce consistency, repeatability, explainability, or cross-model agreement."* The specificity of the failure list is what makes the critique load-bearing.

**3. Send the same brief to at least three models.** Don't average the outputs. Compare them. Findings all three agree on are robust. Findings only one surfaces are stylistic — worth noting, not worth betting on.

**4. Add a judge model.** This is the step most people skip. The judge reads all panel outputs and surfaces *consensus, contradictions, unique insights, blind spots* — the structured deliberation that lets the synthesizer reason about the disagreement rather than drown in it.

**5. Add a synthesizer.** A larger model (in my POC, Opus 4.8) takes the judge's structured deliberation and writes the final merged artifact, with explicit reasoning for what it took from each panel member.

**6. Ask the meta-question.** After the council returns, ask *"who gave the best outcome?"* and *"what are the next steps to actually use this?"* The first exposes the council's own blind spots. The second turns a verdict into a release.

**7. Don't trust the first verdict.** The council doesn't make the artifact right. It makes the next iteration cheaper.

---

## What the council is actually for

The leverage isn't the speed, and it isn't the multi-model theatrics. The leverage is finding the expensive mistake **three minutes into the prompt, not three months into the rollout.**

Most of the cost in any new system isn't the build. It's the redo — because the build was based on a single expert's opinion. Including yours.

An AI Agent Council is a forcing function for the kind of thinking you'd otherwise do too late: after the artifact is built, after the team has momentum, after the calendar has opinions. Done right, it makes sure the work is pointed at something that has earned the calendar.

A single judge in chambers is faster. A jury of peers is more reliable. The same is true for the ideas you ask a model to ship.

The next piece in this series will probably go deeper on the gold set and calibration step — that's where most agentic systems quietly fail in production, and it's where most of the interesting work is.

---

## Repository

The synthesized skill and full panel artifacts are in the companion repo: [`skills/cx-sentiment-analysis/SKILL.md`](../skills/cx-sentiment-analysis/SKILL.md). Copy to `.cursor/skills/` or `.claude/skills/` to use with Cursor or Claude Code. See [skills/README.md](../skills/README.md) for install commands.

---

## Infographic prompt (for NotebookLM)

> Generate a clean, modern, single-page infographic titled **"The AI Agent Council: How a Jury of Models Stress-Tests Your Idea."** Vertical top-to-bottom flow, 5 sections.
>
> **Header band:** A bold title and a one-line subtitle: *"Disagreement is information. The verdict is the artifact."* Style cue: a row of three model chips labelled Claude Opus, GPT, Gemini Pro feeding a single synthesizer node on the right.
>
> **Section 1 — THE CASE (icon: a gavel or single briefcase).** *"One model is one perspective. Even the best can miss critical blind spots."* Show a single figure standing alone, neutral background.
>
> **Section 2 — THE PANEL (icon: three stylised jurors at a long table).** *"Send the same expert brief to 3+ models in parallel. Each plays the same cast of expert personas."* Visually depict three independent model cards, each with a different provider logo, arrows flowing rightward.
>
> **Section 3 — THE DELIBERATION (icon: a balanced scale or a deliberation table).** *"Models agree on the obvious. They disagree on architecture. Each surfaces unique insights. Disagreement is the deliberation, not the noise."* Show a structured output with labelled slots: *Consensus · Contradictions · Unique Insights · Blind Spots*.
>
> **Section 4 — THE FOREMAN (icon: a larger figure at the head of the table).** *"A judge model reads all panel outputs and surfaces the structure. A synthesizer (typically a larger model) writes the final verdict."* Show a single synthesizer node consuming the deliberation and producing a single output card labelled "VERDICT."
>
> **Section 5 — THE RELEASE (icon: a ship sailing or a stamp on a document).** *"A v1 candidate that has survived multiple expert attacks. The follow-up question — 'what's next?' — turns the verdict into a release plan."*
>
> **Style:** Flat modern design, palette of deep navy + warm gold accent + neutral greys, clean sans-serif typography, generous whitespace, minimal ornamentation. Target audience: technical leaders and engineering managers on LinkedIn. Optimise for legibility on a phone screen.

---

## References

- AI Agent Council — https://mcpmarket.com/tools/skills/ai-agent-council
- OpenRouter Fusion: Surpassing Frontier Performance — https://openrouter.ai/blog/announcements/fusion-beats-frontier/
- OpenRouter Fusion Router docs — https://openrouter.ai/docs/guides/routing/routers/fusion-router
- Talk Isn't Always Cheap: Understanding Failure Modes in Multi-Agent Debate (arXiv 2509.05396)
- Agent Skills: What Makes an AI Agent Truly Useful — https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview
- Behaviorally Anchored Rating Scales (BARS) — Smith & Kendall (1963), a foundational reference for rubric design
- Krippendorff, K. (2011). *Computing Krippendorff's Alpha-Reliability* — the inter-rater reliability metric the council adopted as a production target