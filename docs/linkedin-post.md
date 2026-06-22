Single-model review is one opinion. Even when the model is the best in the world.

I've been running a pattern I call the AI Agent Council (Planner Panel is my nickname for it) — send the same expert-panel prompt to multiple top models in parallel, a judge model reads the disagreement, a synthesizer writes the final artifact. Same brief, three models, three different artifacts, then a verdict.

The polite customer during a total prod outage? Every model on the council flagged them as critical churn risk. The canonical single-sentiment design would have rated them a happy 5/5 and routed them away from retention. That's the polite-customer-during-an-outage problem — and a single model, even a brilliant one, would have missed it.

The load-bearing insight from running this for a week: the disagreement is the value, not the noise. If three top models gave you the same answer, you'd have paid for one slow single-model run. The fact that they disagree on architecture while agreeing on the obvious is the deliberation — and the deliberation is what earns the verdict.

The POC: built on OpenRouter Fusion — panel → judge → synthesizer → verdict. The cost-efficient variant works too: a few smaller models for parallel planning, one bigger model to fuse them, often beats a single flagship alone — at a fraction of the cost. OpenRouter's DRACO benchmarks back the pattern directionally (budget panel ~50% of Fable 5 cost, within 1% of its score). Not a guarantee for every task — see the repo disclaimers.

Infographic below shows the council at a glance 👇

Full breakdown with the test case, the tradeoffs, and the cost-conscious variant in the article and repo: https://github.com/admaduranga/agent-council-cx-sentiment-skill

What's the most expensive mistake a single-model review has ever let through for you?
