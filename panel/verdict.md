# Critical Review of a SaaS Support Sentiment Framework — and a Production-Ready Replacement

No explicit framework was attached, so I'll do what any competent reviewer would: reconstruct the **canonical design** that ~90% of CX QA programs and bolt-on "AI sentiment" helpdesk features actually use, then tear into it.

> **The framework under review:** A single sentiment label per support case — either **1–5** or **Positive / Neutral / Negative** — assigned (by a human reviewer or an LLM) primarily from the customer's tone and word choice across the conversation.

My verdict, and the panel's: **this design is not safe for production decision-making.** Here's why, followed by a complete rebuilt `SKILL.md`.

---

## Part 1 — What's Wrong With the Canonical Design

### Defect 1 (Critical): It measures the wrong construct by conflating four orthogonal things
A single score silently fuses **emotional valence**, **satisfaction**, **business impact**, and **churn risk**. These don't correlate cleanly, and collapsing them destroys decision value:

| Scenario | Valence | Satisfaction | Churn risk | Business impact |
|---|---|---|---|---|
| Polite customer, total prod outage | Calm | Neutral | **Critical** | **Critical** |
| Angry customer, cosmetic UI bug | Very negative | Low | Low | Trivial |
| Resolved but took 3 weeks | Relieved | Low | High | Medium |

A framework that scores row 1 as "positive 5/5" and row 2 as "negative 1/5" will route the *wrong* cases to retention and **systematically under-protect your most valuable, most stoic enterprise accounts** — the ones who churn quietly. This is the single most expensive flaw, and the entire panel agreed on it.

### Defect 2 (Critical): The scale is unanchored → low repeatability
Nothing defines the boundary between a 2 and a 3, or "Neutral" vs. "Negative." Unanchored scales are the #1 driver of low inter-rater reliability. Two competent reviewers — or two LLM runs at the same temperature — will disagree, and there's no rubric to say who's right. Naive scoring routinely drops below κ ≈ 0.60.

### Defect 3 (Critical): No mandatory evidence → unauditable
A score with no cited customer quote is unfalsifiable. You cannot QA the QA.

### Defect 4 (Critical): Fairness/bias risk from politeness and style proxies
Treating politeness as positivity and curtness/CAPS as anger **penalizes non-native speakers and direct communication styles**, and rewards verbose idiomatic native speakers. That's a discrimination defect, not just an accuracy one.

### Defect 5 (High): Temporal blindness
A case is a *trajectory*, not a point. A ticket that opens at fury and closes at gratitude is a support **win**; one label per case erases it. There's no open/close/trend signal.

### Defect 6 (High): Severity–sentiment confound
P1 vocabulary ("down," "production," "revenue") reads as "negative" even when the customer is courteous. The model learns to equate technical-severity language with negative emotion — a confound, not a signal.

### Defect 7 (High): Effort and resignation — the real churn predictors — are invisible
Customer effort (reopened tickets, "as I said in my last email," channel-switching) is a stronger churn predictor than raw emotion. And **resignation** ("whatever, we'll just find another tool") often reads *calmer* than active anger while being a far louder churn klaxon. "Negative" flattens these distinct states.

### Defect 8 (High): No determinism controls and no confidence/abstain path
Temperature, option ordering, reasoning leaking into labels, and forcing a label onto "ok thanks" all manufacture noise. There's no calibration set, no agreement target, no abstention rule.

### Other defects
"Neutral" is an overloaded dumping ground (genuine / mixed / ambiguous / couldn't-tell); sarcasm and negation defeat lexical scoring; quoted/relayed third-party sentiment is undefined; multi-participant threads get averaged (drowning out one severely impacted stakeholder); and pasted logs, stack traces, agent apologies, and auto-replies pollute scoring.

---

## Part 2 — Stress Tests (how the *rebuilt* framework behaves)

| Scenario | Old framework | Rebuilt framework |
|---|---|---|
| **Outage, polite customer** | "Positive/Neutral" — ignored | Tone neutral, **Impact Critical, Churn High** → proactive outreach ✅ |
| **Polite churn threat** | Misses it (calm tone) | Churn-risk override fires Critical alert regardless of tone ✅ |
| **Angry customer, cosmetic bug** | "Very Negative" → wrongly to retention | Valence very negative, **Impact Low** → de-escalation, not retention ✅ |
| **3-week unresolved, now resigned** | Final "thanks" → Positive | Effort severe, resignation flagged, **Churn Critical** ✅ |
| **Sarcasm** ("Great, another outage") | "Positive" | Context-over-tokens → Negative + sarcasm flag + evidence ✅ |
| **ESL / terse customer** | Penalized as cold/angry | Bias guardrail: brevity ≠ frustration; abstain if ambiguous ✅ |
| **Multi-stakeholder thread** | Averaged away | Per-participant; worst active risk dominates routing ✅ |
| **"ok thanks"** | Forced "Positive" | Low confidence / low-signal, no fabricated sentiment ✅ |

One honest caveat the panel split on: whether to keep a final composite score at all. I've kept a final score **derived by deterministic precedence rules and caps — never a soft average** — because operations teams need one routable number, but the independent dimensions are the source of truth and must always travel with it.

---

## Part 3 — Production-Ready `SKILL.md`

```markdown
# SKILL.md — SaaS Technical Support CX Sentiment Analysis

**Version:** 1.0
**Construct measured:** Customer support-experience state and risk — NOT lexical word polarity.
**Determinism:** Run at temperature 0. Score dimensions before the final score. Never average; apply precedence rules. Log rubric version with every output.

---

## 1. Purpose & Non-Goals

Analyze SaaS technical support interactions to produce a consistent, evidence-based, auditable
assessment of customer sentiment and support risk.

**Non-goals (do NOT use this skill for):**
- Punitive agent performance evaluation (sentiment ≠ agent quality).
- Inferring customer demographics, mood, or personality.
- Scoring agent, internal, or automated text as customer sentiment.

---

## 2. Scope: What to Score

**SCORE ONLY (sentiment evidence):**
- Customer / end-user / authorized-requester authored messages.
- Official CSAT/survey scores and comments, if present.

**CONTEXT ONLY (never direct sentiment evidence):**
- Agent replies, apologies, empathy, macros.
- Internal notes, engineering comments, system/automated messages.
- Case metadata: priority, SLA status, case age, product area, known-outage flags.

**EXCLUDE ENTIRELY (do not score):**
- Pasted logs, stack traces, error messages, code, screenshots-as-text.
- Quoted third-party text UNLESS the customer endorses it (see §7.4).

---

## 3. The Six Independent Dimensions

Score each dimension on its own evidence BEFORE the final score. Dimensions must not
contaminate each other (a calm tone does not lower impact; high impact does not raise anger).

### 3.1 Emotional Valence  (−2 … +2)
| Score | Meaning | Signals |
|---:|---|---|
| −2 | Strong negative | Hostile, furious, abusive, "unacceptable", "ridiculous" |
| −1 | Mild/moderate negative | Frustrated, worried, disappointed, confused-negative |
| 0 | Neutral / factual | Descriptive, no affect |
| +1 | Mild positive | Cooperative, relieved, mild appreciation |
| +2 | Strong positive | Explicit praise, delight, advocacy |

### 3.2 Emotional Intensity / Arousal  (0 … 3)
0 = flat, 1 = mild, 2 = elevated, 3 = extreme. **Measured separately from valence**
(valence × arousal is a 2-axis model; resignation = negative valence + LOW arousal).

### 3.3 Business Impact  (0 … 4)
| Score | Meaning |
|---:|---|
| 0 | None stated (how-to/inquiry) |
| 1 | Cosmetic / single-user annoyance / workaround exists |
| 2 | Feature impaired, usable workaround |
| 3 | Workflow blocked, no acceptable workaround, deadline at risk |
| 4 | Production down, broad outage, data loss, security/privacy, regulatory, launch-blocking |
*Rule: do NOT infer level 4 unless stated or strongly supported by metadata. Calm tone never reduces impact.*

### 3.4 Customer Effort / Friction  (0 … 4)
0 = first contact; 1 = one clarification; 2 = repeated info / one failed attempt;
3 = reopened / multiple failed fixes / long delay; 4 = long-running unresolved, looping, missed SLAs.
*Rule: case age alone is insufficient — require evidence of repetition or waiting.*

### 3.5 Resolution Confidence  (−2 … +2)
−2 failed/rejected ("still broken") · −1 unresolved/doubtful ("any update?") · 0 pending/unknown ·
+1 credible path / accepted workaround · +2 customer-confirmed resolved.
*Rule: agent saying "resolved" is NOT resolution — prefer explicit customer confirmation.*

### 3.6 Escalation / Churn Risk  (0 … 4)
0 none · 1 mild urgency · 2 escalation/deadline · 3 refund/SLA complaint/exec escalation ·
4 churn/cancellation/legal/security-incident/competitor-evaluation threat.
*Rule: a POLITE churn threat is still level 4. A neutral "how do I cancel a seat?" is NOT, absent dissatisfaction.*

---

## 4. Affective-State Tag (required, single value)
Disambiguates "negative": `cooperative · transactional · anxious · frustrated · hostile · resigned · satisfied · delighted`.
**`resigned`** (calm + churning) MUST trigger a Critical alert even at low intensity.

---

## 5. Final Score — Deterministic Precedence (NEVER average)

Assign all dimensions, then walk these steps in order; first matching rule wins.

**Step 0 — Survey override:** If an official CSAT exists, map it (1★→1 … 5★→5). If it conflicts
with the conversation, keep the survey but record the discrepancy in `explanation`.

**Step 1 — Force score 1 (Very Negative / Critical Risk) if ANY:**
- `churn_risk = 4` (and not explicitly historical+resolved), OR
- `business_impact ≥ 3` AND `resolution_confidence ≤ 0`, OR
- `affective_state = resigned`, OR
- `customer_effort = 4` AND unresolved, OR
- severe sustained hostility tied to an unresolved support failure.

**Step 2 — Score 2 (Negative) if any:** valence ≤ −1; impact 2–3 unresolved; effort ≥ 2;
resolution_confidence ≤ −1; churn_risk 2–3; or mixed (praises agent but remains blocked).

**Step 3 — Score 5 (Very Positive) only if ALL:** valence = +2 with explicit praise;
resolution_confidence = +2; impact ≤ 1; effort ≤ 1 (unless customer explicitly praises recovery);
churn_risk = 0.

**Step 4 — Score 4 (Positive):** satisfied/relieved/cooperative, no material unresolved negatives,
resolution_confidence ≥ +1, churn_risk ≤ 1.

**Step 5 — Score 3 (Neutral / Mixed / Insufficient Evidence):** factual, balanced, routine,
or insufficient customer text. **Default here under low confidence.**

### Caps (always enforced)
- **Gratitude cap:** generic "thanks/please" cannot raise a case above 3 without explicit
  resolution, satisfaction, relief, or praise.
- **Unresolved-impact cap:** `impact ≥ 3` & `resolution_confidence ≤ 0` ⇒ max score 2.
- **Severe-effort cap:** `effort = 4` & resolved ⇒ max score 4.
- **Agent-text cap:** agent tone never moves the score unless the customer reacts to it.
- **Minor-anger guardrail:** anger over `impact ≤ 1` with no risk ⇒ floor at 2, not 1.

---

## 6. Trajectory (required)
`improving · worsening · stable · mixed · unknown`. Reported alongside, never replacing, the score.
Score reflects the customer's latest MATERIAL state while retaining unresolved negative history.

---

## 7. Linguistic & Edge-Case Rules

**7.1 Context outranks tokens.** Interpret meaning, not surface words.
**7.2 Sarcasm:** "Great, another outage" = negative. If sarcasm is plausible but uncertain → medium/low confidence, cite the ambiguous span.
**7.3 Negation:** handle "not bad", "no longer broken", "can't say I'm unhappy" by meaning.
**7.4 Quoted/relayed sentiment:** a customer calmly relaying a third party's fury counts as ACCOUNT-LEVEL risk if endorsed ("my CTO is furious"); attribute it explicitly.
**7.5 Multi-participant threads:** score each customer voice; for routing, **worst active risk dominates** (never average); for CSAT-proxy, weight the primary requester. Note conflicts.

---

## 8. Anti-Bias Guardrails (mandatory)
Do NOT infer sentiment from name, geography, nationality, writing style, grammar quality,
formality, directness, verbosity, capitalization, or punctuation alone.
- Brevity/curtness ≠ frustration (tag `transactional`).
- Missing "please/thank you" ≠ negativity.
- ALL CAPS / "!!!" are weak signals only, never sole evidence (accessibility/style confounds).
- ESL/translated text: score explicit meaning only; lower confidence when unclear.
- During a confirmed P1, default baseline tone to `anxious`; do not read terseness as hostility.

---

## 9. Confidence & Abstention
`high` (clear, recent, aligned customer evidence) · `medium` (mixed/likely-sarcasm/partial) ·
`low` (minimal text, ambiguity, translation noise, mostly agent/internal).
Under `low` confidence with no clear signal → score 3 (do not fabricate sentiment).

---

## 10. Evidence Requirements (auditability)
Every score other than a low-confidence 3, and EVERY non-zero `churn_risk`, `business_impact ≥ 3`,
sarcasm, or hostility tag, MUST include ≥1 verbatim customer quote with `message_id`, `timestamp`,
`speaker_role`, and the dimension it supports. Never fabricate quotes; if unavailable, mark
`quote_unavailable` and lower confidence. A `churn_risk = 4` without cancellation/competitor/
legal/refund language in the quote is an automatic QA failure.

---

## 11. Output Schema (strict JSON)
```json
{
  "case_id": "string",
  "rubric_version": "1.0",
  "analysis_window": "messages X–Y",
  "case_sentiment_score": 1,
  "case_sentiment_label": "Very Negative / Critical Risk",
  "affective_state": "resigned",
  "dimensions": {
    "valence": -1,
    "intensity": 0,
    "business_impact": 4,
    "customer_effort": 4,
    "resolution_confidence": -2,
    "escalation_or_churn_risk": 4
  },
  "trajectory": "worsening",
  "confidence": "high",
  "evidence": [
    {"message_id": "m41", "timestamp": "2026-01-15T10:22:00Z",
     "speaker_role": "customer",
     "quote": "Thanks for the update. We've been down 48h and are migrating to a competitor.",
     "supports": ["escalation_or_churn_risk", "business_impact", "case_sentiment_score"]}
  ],
  "explanation": "Polite tone but unresolved critical outage + explicit competitor migration. Resigned state forces Critical.",
  "scoring_rules_applied": ["Step 1: churn_risk=4", "Step 1: resigned state", "Gratitude cap"],
  "requires_intervention": true
}
```

---

## 12. Determinism Controls
Temperature 0 · fixed dimension/option ordering · score dimensions before final score ·
keep free-text reasoning out of label fields · emit the schema only · idempotency hash of
(input + rubric_version) so identical inputs yield identical outputs.

---

## 13. Governance, Calibration & Monitoring
1. Maintain a versioned, adjudicated **gold set** (start ~200 cases) covering all §2 stress cases.
2. **Inter-rater target: Krippendorff's α ≥ 0.80** (per dimension) before release; recalibrate below threshold.
3. Review disagreements monthly; adjudicate and feed back into the gold set.
4. Monitor score distributions by product, region, language, and segment to detect bias drift.
5. Watch rater/model biases: recency, severity-halo, leniency/severity drift, affect-as-information.
6. Re-tune `business_impact` and `churn_risk` anchors against your product's failure vocabulary.
7. Log rubric version with every score; version the gold set alongside the rubric.

---

## 14. Pre-Finalization Checklist
1. Used only customer-authored text as sentiment evidence?
2. Scored all six dimensions independently?
3. Applied precedence (Step 0→5), not an average?
4. Applied all relevant caps?
5. Checked sarcasm / negation / quoted-sentiment / multi-participant rules?
6. Applied anti-bias guardrails (no style/demographic inference)?
7. Cited verbatim evidence for every required field?
8. Set confidence honestly; abstained to 3 if low + no signal?
9. Emitted valid schema with rubric_version?
```

**Caveats before rollout:** the α ≥ 0.80 target and the 200-case gold set are starting recommendations — calibrate thresholds against your own adjudicated data, and expect the `business_impact` and `churn_risk` anchors to need a tuning pass against your product's specific failure vocabulary, since those are the dimensions most sensitive to domain context.