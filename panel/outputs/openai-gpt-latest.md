## 1. Critical review of the inferred “typical” framework

Because no explicit framework was provided, I am assuming a common SaaS support sentiment framework:

> “Assign each support case a single sentiment label or score, usually Positive / Neutral / Negative or 1–5, based primarily on the language used by the customer.”

This design is simple, but it is not production-ready without stronger definitions, evidence rules, and edge-case handling.

---

## 2. Panel critique

### A. Customer Experience CX perspective

**Major flaw: sentiment is often confused with customer risk.**

A polite customer saying:

> “Hi, our production environment has been down for four hours. Please advise.”

may appear emotionally neutral, but from a CX standpoint this is highly negative because the customer is severely impacted.

Conversely, an angry customer saying:

> “This is ridiculous, why can’t I change the button color?”

may have strong negative emotion but low business impact.

A single undifferentiated score cannot reliably distinguish:

- Emotional tone
- Business impact
- Support experience satisfaction
- Resolution confidence
- Escalation or churn risk
- Customer effort

**Recommendation:** Use a final sentiment score, but support it with separate measurable dimensions.

---

### B. SaaS Technical Support perspective

**Major flaw: case context changes sentiment meaning.**

Technical support cases often involve:

- Known outages
- Bugs with no ETA
- Workarounds
- Multiple affected users
- Reopened tickets
- Delays caused by customer-side dependencies
- Multiple customer participants
- Internal notes that should not affect customer sentiment

A naive sentiment model may incorrectly treat an agent’s apology or friendly tone as evidence of positive customer sentiment.

**Recommendation:** Score only customer-authored content for sentiment. Use metadata and case status only as context, not as emotional evidence.

---

### C. Quality Assurance perspective

**Major flaw: labels are usually underdefined.**

Typical definitions like:

- 1 = Very negative
- 3 = Neutral
- 5 = Very positive

are too vague. Different evaluators will disagree on whether a frustrated but respectful customer is a 2 or 3, or whether a resolved case with a “thanks” is a 4 or 5.

**Recommendation:** Define observable triggers, caps, precedence rules, and examples. Require evidence quotes for every non-neutral score.

---

### D. Sentiment Analysis perspective

**Major flaw: lexical sentiment is brittle.**

Support conversations include:

- Sarcasm: “Great, another outage.”
- Polite negativity: “Thanks, but this still blocks our launch.”
- Template closings: “Thank you” even when dissatisfied
- Non-native phrasing
- Mixed sentiment: “Your agent was helpful, but the product is unusable.”
- Profanity directed at the product rather than the support agent

**Recommendation:** Analyze meaning and context, not isolated positive or negative words. Separate gratitude from satisfaction.

---

### E. Behavioral Psychology perspective

**Major flaw: politeness and anger are not reliable proxies for satisfaction.**

Customers vary culturally and personally in how they express dissatisfaction. Some customers remain calm while preparing to churn. Others express anger over minor irritations.

Bias risks include:

- Underestimating dissatisfaction from polite, formal, or non-native speakers
- Over-penalizing direct communication styles
- Treating profanity as inherently more severe than business impact
- Confusing urgency with hostility
- Assuming “thank you” means satisfaction

**Recommendation:** Anchor scoring to observable support-state signals: impact, effort, unresolved blockers, escalation language, and explicit satisfaction.

---

### F. Evaluation Framework Design perspective

**Major flaw: no auditability or reproducibility.**

A production scoring framework needs:

- Input boundaries
- Actor filtering
- Evidence requirements
- Deterministic scoring precedence
- Confidence ratings
- Handling of missing or ambiguous data
- Calibration examples
- Inter-rater agreement checks
- Versioning

Without these, the framework will produce inconsistent cross-model results.

**Recommendation:** Use a structured rubric and output schema that exposes the reasoning evidence without relying on opaque impressions.

---

## 3. Stress-test findings

|Scenario|Failure of typical framework|Improved handling|
|---|---|---|
|Polite customer during outage|May score Neutral due to calm language|High impact unresolved case should cap sentiment at Negative or Very Negative|
|Churn threat stated politely|May miss risk because tone is calm|Explicit cancellation/churn language triggers Critical Risk|
|Angry customer with minor issue|May score Very Negative based only on anger|Emotion is negative, but low impact prevents automatic Critical unless churn/escalation/abuse is present|
|Long-running unresolved case|Last message may be “Thanks” and get Positive|Prior unresolved effort/friction limits score|
|Sarcasm|May misclassify “Great, another bug” as Positive|Sarcasm indicators treated as negative with medium confidence if ambiguous|
|Language barriers|May misread grammar as frustration or indifference|Score only explicit meaning; lower confidence when unclear|
|Multi-thread conversation|May average conflicting voices|Separate customer voices; final score reflects primary requester or highest active customer risk|
|Agent apologizes repeatedly|May infer customer dissatisfaction from agent text|Agent text is context only, not customer sentiment evidence|
|Customer praises agent but criticizes product|May score Positive|Mixed: support satisfaction positive, product/CX sentiment negative|
|Resolved after severe pain|May score Very Positive after “thank you”|Recovery can improve score, but severe prior effort prevents automatic 5|

---

## 4. Recommended production approach

The framework should produce:

1. **Final CX Sentiment Score**: 1–5.
2. **Supporting dimensions**:
    - Emotional tone
    - Business/user impact
    - Customer effort/friction
    - Resolution confidence
    - Escalation/churn risk
3. **Trajectory**: improving, worsening, stable, mixed, or unknown.
4. **Confidence level**.
5. **Evidence quotes**.
6. **Bias safeguards and exclusions**.

The final score should not be a vague average. It should be assigned through precedence rules, caps, and explicit triggers.

---

# Production-ready `SKILL.md`

````
# SKILL.md — SaaS Technical Support CX Sentiment Analyzer

## 1. Skill name

SaaS Technical Support CX Sentiment Analyzer

## 2. Version

Version: 1.0

## 3. Purpose

Analyze SaaS technical support interactions and assign a consistent, evidence-based customer sentiment score.

The goal is to measure the customer’s current CX sentiment and support-experience risk, not merely the emotional polarity of individual words.

This skill is designed for production use across technical support cases, including bugs, outages, configuration problems, billing-related technical issues, integrations, APIs, security incidents, long-running cases, and multi-thread customer conversations.

---

## 4. Core output

For each support case, produce:

1. A final `case_sentiment_score` from 1 to 5.
2. A final `case_sentiment_label`.
3. Supporting dimension scores:
   - `emotional_tone`
   - `business_impact`
   - `customer_effort`
   - `resolution_confidence`
   - `escalation_or_churn_risk`
4. Sentiment trajectory.
5. Confidence level.
6. Evidence quotes.
7. Short explanation.

---

## 5. Scope

### Include

Analyze:

- Customer-authored messages
- End-user-authored messages
- Authorized requester messages
- Customer survey comments, if present
- Explicit customer statements about satisfaction, frustration, urgency, impact, effort, escalation, cancellation, or resolution

Use the following as context only:

- Case metadata
- Priority
- SLA status
- Case age
- Product area
- Agent replies
- Internal escalation markers
- Known outage metadata

### Exclude

Do not treat the following as direct evidence of customer sentiment:

- Agent apologies
- Agent friendliness
- Internal notes
- Internal Slack or engineering comments
- Automated system messages
- Macro/template text from agents
- Sentiment of the support agent
- The model’s assumption about how the customer “probably feels” without evidence

---

## 6. Definitions

### 6.1 Final CX sentiment score

The final score represents the customer’s apparent support-experience sentiment and risk state at the case level.

It combines explicit emotional tone with observable support-context signals such as business impact, unresolved friction, escalation, and churn risk.

It is not a pure lexical sentiment score.

### 6.2 Score labels

| Score | Label | Meaning |
|---:|---|---|
| 1 | Very Negative / Critical Risk | Severe dissatisfaction, severe unresolved impact, churn/legal/escalation threat, production-down condition, data-loss/security concern, or extreme support breakdown |
| 2 | Negative | Clear frustration, dissatisfaction, unresolved moderate/high impact, repeated effort, failed workaround, delay concern, or elevated risk |
| 3 | Neutral / Mixed / Insufficient Evidence | Factual, ambiguous, low-emotion, balanced mixed signals, routine request, or insufficient customer evidence |
| 4 | Positive | Customer appears satisfied, cooperative, relieved, or accepting; issue is resolved or has a credible path forward; no significant unresolved negative signals |
| 5 | Very Positive | Strong explicit praise, delight, advocacy, or exceptional satisfaction; issue resolved or exceeded expectations; no material unresolved negative signals |

---

## 7. Required dimension scores

Score each dimension independently before assigning the final case score.

### 7.1 Emotional tone

Measures the customer’s expressed affect.

| Score | Meaning | Observable signals |
|---:|---|---|
| -2 | Strong negative emotion | Angry, furious, panicked, hostile, “unacceptable,” “ridiculous,” abusive language, repeated emphatic complaint |
| -1 | Mild/moderate negative emotion | Frustrated, disappointed, concerned, worried, confused in a negative way |
| 0 | Neutral or unclear | Factual description, no affect, ambiguous language |
| +1 | Mild positive emotion | Polite appreciation, relief, cooperative tone, mild praise |
| +2 | Strong positive emotion | Explicit praise, delight, enthusiastic thanks, “excellent,” “amazing,” “you saved us” |

Important rules:

- Polite greetings or closings such as “thanks” or “please” do not automatically create positive sentiment.
- Profanity or strong language is negative evidence, but severity depends on context.
- Sarcasm should be interpreted by meaning, not literal positive words.

---

### 7.2 Business impact

Measures the operational or user impact described by the customer.

| Score | Meaning | Observable signals |
|---:|---|---|
| 0 | No impact stated | General inquiry, how-to question, no business effect described |
| 1 | Low impact | Cosmetic issue, minor inconvenience, single-user annoyance, workaround available |
| 2 | Moderate impact | Feature impaired, some users affected, degraded workflow, usable workaround exists |
| 3 | High impact | Team/customer workflow blocked, important process delayed, no acceptable workaround, deadline at risk |
| 4 | Critical impact | Production down, broad outage, revenue-impacting failure, data loss, security/privacy issue, regulatory exposure, launch blocked, executive/customer-facing incident |

Important rules:

- A calm tone does not reduce business impact.
- Do not infer critical impact unless stated or strongly supported by case metadata.
- If impact is unknown, use 0 and reflect uncertainty in confidence.

---

### 7.3 Customer effort / friction

Measures how much effort, repetition, delay, or friction the customer has experienced.

| Score | Meaning | Observable signals |
|---:|---|---|
| 0 | No notable effort | First contact, simple request, no friction stated |
| 1 | Low effort | One clarification, normal troubleshooting |
| 2 | Moderate effort | Multiple steps, repeated information, one failed attempt, moderate waiting |
| 3 | High effort | Reopened case, multiple failed fixes, repeated explanations, long delay, unclear ownership |
| 4 | Severe effort | Long-running unresolved case, circular troubleshooting, missed commitments/SLA, customer repeatedly escalates or performs excessive work |

Important rules:

- Case age alone is not enough; look for evidence of unresolved waiting or repeated effort.
- Repeated customer messages asking for updates are effort/friction evidence.
- If the issue is resolved after very high effort, do not automatically assign a Very Positive final score.

---

### 7.4 Resolution confidence

Measures whether the customer appears to believe the issue is resolved or on a credible path.

| Score | Meaning | Observable signals |
|---:|---|---|
| -2 | Failed or rejected resolution | “This did not work,” “still broken,” “not acceptable,” “we are back to square one” |
| -1 | Unresolved or doubtful | “Any update?”, “still seeing this,” “not sure this solves it,” workaround rejected |
| 0 | Unknown or pending | No clear resolution state |
| +1 | Credible path or acceptable workaround | Customer accepts next steps, confirms workaround is acceptable, expresses cautious relief |
| +2 | Explicitly resolved and satisfied | “That fixed it,” “resolved,” “works now,” especially with appreciation or praise |

Important rules:

- Agent saying “resolved” is not enough. Prefer customer confirmation.
- A workaround is not equivalent to full resolution unless the customer accepts it.
- “Thanks” without confirming resolution is not resolution evidence.

---

### 7.5 Escalation or churn risk

Measures explicit risk signals.

| Score | Meaning | Observable signals |
|---:|---|---|
| 0 | No risk stated | No escalation, cancellation, refund, legal, or executive concern |
| 1 | Mild urgency | “Please prioritize,” “we need this soon” |
| 2 | Elevated urgency/escalation | Deadline risk, request for escalation, important customer/account impact |
| 3 | Serious business risk | Refund request, SLA complaint, executive escalation, account impact, public complaint threat |
| 4 | Critical risk | Cancellation/churn threat, legal/regulatory threat, security incident, data-loss claim, severe public escalation, “we will leave,” “we are evaluating alternatives” |

Important rules:

- Do not treat a routine “how do I cancel a seat?” request as churn risk unless dissatisfaction is present.
- A polite churn threat is still critical risk.
- Escalation to engineering is not customer escalation unless the customer requests/escalates or risk is evident.

---

## 8. Final score decision procedure

Assign dimensions first. Then assign the final score using the following precedence rules.

Do not calculate the final score as a simple average.

### Step 1: Identify direct override signals

If an official customer satisfaction survey score is present, use it as strong evidence:

| CSAT or rating | Suggested final score |
|---|---:|
| 1 star / very dissatisfied | 1 |
| 2 stars / dissatisfied | 2 |
| 3 stars / neutral | 3 |
| 4 stars / satisfied | 4 |
| 5 stars / very satisfied | 5 |

If the survey conflicts with conversation evidence, keep the survey as strong evidence but explain the discrepancy.

Example:

- Customer says “Thanks, fixed” but leaves 1-star CSAT: score should generally be 1 or 2 with explanation.

---

### Step 2: Assign score 1 if any critical negative condition applies

Assign `case_sentiment_score = 1` if one or more of the following is present:

- Explicit cancellation, churn, legal, regulatory, or severe public escalation threat due to dissatisfaction
- Critical business impact unresolved: production down, data loss, security incident, revenue-critical outage, broad outage, blocked launch, or regulatory exposure
- Severe emotional negativity directed at the product, company, or support experience, especially with unresolved issue
- Long-running unresolved case with severe effort/friction and no credible path forward
- Repeated missed commitments or SLA failures with customer dissatisfaction
- Customer states the situation is “unacceptable,” “critical,” “we cannot operate,” “we are losing customers,” or equivalent
- Abuse or hostility combined with dissatisfaction or unresolved support failure

Use score 1 even if the customer is polite when the business impact or churn/legal risk is critical.

---

### Step 3: Assign score 2 if negative but not critical

Assign `case_sentiment_score = 2` if score 1 does not apply and one or more of the following is present:

- Customer expresses frustration, disappointment, concern, or dissatisfaction
- High business impact but not critical
- Moderate or high unresolved impact
- Repeated troubleshooting or customer effort
- Failed workaround or rejected solution
- Customer asks repeatedly for updates
- Customer questions support quality, response time, or ownership
- Escalation risk score is 2 or 3
- Resolution confidence is -1 or -2
- Mixed case where the customer appreciates the agent but remains materially blocked

---

### Step 4: Assign score 5 only when very positive conditions are met

Assign `case_sentiment_score = 5` only if all are true:

- Customer gives explicit strong praise, delight, or exceptional satisfaction
- Issue is resolved, avoided, or exceeded expectations
- No meaningful unresolved negative signal remains
- Business impact is 0 or 1 at the time of scoring
- Customer effort is 0 or 1, unless the customer explicitly praises the recovery despite prior effort
- Escalation/churn risk is 0
- Latest material customer sentiment is positive

Examples of score 5 evidence:

- “This solved it perfectly. Fantastic support.”
- “Amazing, thank you for the fast turnaround.”
- “You saved our launch. Really appreciate the excellent support.”

Do not assign score 5 for a generic “thanks” alone.

---

### Step 5: Assign score 4 for positive or resolved cases

Assign `case_sentiment_score = 4` if score 5 does not apply and the customer appears satisfied, relieved, cooperative, or accepting, and there are no significant unresolved negative signals.

Typical score 4 conditions:

- Customer confirms the issue is resolved
- Customer accepts a workaround or next step
- Customer expresses appreciation beyond a routine closing
- Customer appears cooperative and no material dissatisfaction is present
- Business impact is low or moderate and controlled
- Resolution confidence is +1 or +2
- Escalation/churn risk is 0 or 1

Examples:

- “That fixed it, thanks.”
- “The workaround is acceptable for now.”
- “Thanks for the quick help, we’re good.”

---

### Step 6: Assign score 3 as the default for neutral, mixed, or insufficient evidence

Assign `case_sentiment_score = 3` when:

- Customer language is factual or informational
- No clear sentiment is expressed
- The request is routine
- Impact is low or unknown
- Resolution state is pending or unclear
- Positive and negative evidence is balanced
- There is insufficient customer-authored evidence

Examples:

- “How do I configure SSO?”
- “I am seeing error code 403 when calling the API.”
- “Can you confirm whether this endpoint supports pagination?”
- “Thanks” as a greeting or closing with no resolution confirmation

---

## 9. Caps and guardrails

Use these caps to prevent inflated positive scores.

### 9.1 Unresolved impact cap

If `business_impact >= 3` and `resolution_confidence <= 0`, final score must be 1 or 2.

### 9.2 Churn/legal/security cap

If `escalation_or_churn_risk = 4`, final score must be 1 unless the risk is historical and explicitly resolved.

### 9.3 Generic gratitude cap

Generic politeness such as “thanks,” “thank you,” or “please” cannot raise a case above 3 unless accompanied by explicit resolution, satisfaction, relief, or praise.

### 9.4 Severe-effort cap

If `customer_effort = 4` and the issue remains unresolved, final score must be 1 or 2.

If the issue is resolved after severe effort, final score should usually be no higher than 4 unless the customer explicitly expresses exceptional recovery satisfaction.

### 9.5 Agent-text exclusion

Agent apologies, empathy, or positive phrasing must not increase or decrease the customer sentiment score unless the customer reacts to them.

### 9.6 Minor issue anger guardrail

Strong anger over a low-impact issue can justify score 2. Assign score 1 only if there is critical risk, abusive escalation, churn/legal threat, severe support breakdown, or repeated unresolved friction.

---

## 10. Multi-message and multi-thread handling

### 10.1 Chronology

Analyze the full conversation chronologically.

The final score should reflect the customer’s latest material state while retaining unresolved negative history.

A final “thanks” does not erase unresolved critical impact unless the customer confirms resolution or acceptance.

### 10.2 Multiple customer participants

If multiple customer-side participants are present:

- Score the primary requester if identifiable.
- If no primary requester is identifiable, score the highest active customer risk.
- Do not average away severe risk from one impacted stakeholder.
- Mention conflicting customer perspectives in the explanation.

### 10.3 Internal and agent messages

Do not score internal or agent messages as customer sentiment.

Use them only to understand context, such as whether the issue is known, whether a workaround was offered, or whether the case was escalated.

---

## 11. Sarcasm and indirect sentiment

Treat sarcasm as negative when contextual cues indicate the literal positive words are not sincere.

Examples:

| Customer statement | Interpretation |
|---|---|
| “Great, another outage.” | Negative |
| “Awesome, it broke again.” | Negative |
| “Thanks for nothing.” | Negative |
| “Perfect, now our entire team is blocked.” | Negative |

If sarcasm is plausible but uncertain, assign medium or low confidence and cite the ambiguous evidence.

---

## 12. Language, culture, and bias safeguards

Do not infer sentiment from:

- Customer name
- Geography
- Nationality
- Writing style
- Grammar quality
- Formality
- Directness
- Non-native language patterns
- Use or absence of pleasantries

Do not penalize customers for:

- Being concise
- Being direct
- Using imperfect grammar
- Not using “please” or “thank you”
- Communicating in a second language

When language is unclear:

- Score based only on explicit meaning.
- Use lower confidence.
- Avoid over-interpreting tone.

---

## 13. Confidence scoring

Assign one confidence level.

### High confidence

Use when:

- There are clear customer-authored statements.
- Sentiment and impact are explicit.
- Evidence is recent and unambiguous.
- Dimensions align consistently.

### Medium confidence

Use when:

- Evidence is present but mixed.
- Sarcasm or indirect language is likely but not certain.
- Resolution state is partially unclear.
- Multiple customer participants have different tones.

### Low confidence

Use when:

- Minimal customer text is available.
- Language is ambiguous or poorly translated.
- Only metadata is available.
- Customer sentiment must be inferred from weak evidence.
- The case contains mostly agent/internal messages.

Default to score 3 when confidence is low and no clear negative or positive evidence exists.

---

## 14. Evidence requirements

For every final score other than 3, provide at least one customer-authored evidence quote.

For score 1 or 2, cite the strongest negative evidence.

For score 4 or 5, cite explicit resolution, satisfaction, relief, or praise.

Evidence must include, when available:

- Message ID
- Timestamp
- Speaker role
- Quote
- Dimension supported

Do not fabricate quotes.

If exact quotes are unavailable, summarize the evidence and mark it as `quote_unavailable`.

---

## 15. Output schema

Return a JSON object using the following structure:

```json
{
  "case_sentiment_score": 2,
  "case_sentiment_label": "Negative",
  "dimensions": {
    "emotional_tone": {
      "score": -1,
      "label": "Mild/moderate negative emotion"
    },
    "business_impact": {
      "score": 3,
      "label": "High impact"
    },
    "customer_effort": {
      "score": 2,
      "label": "Moderate effort"
    },
    "resolution_confidence": {
      "score": -1,
      "label": "Unresolved or doubtful"
    },
    "escalation_or_churn_risk": {
      "score": 1,
      "label": "Mild urgency"
    }
  },
  "trajectory": "worsening",
  "confidence": "high",
  "evidence": [
    {
      "message_id": "m123",
      "timestamp": "2026-01-15T10:22:00Z",
      "speaker_role": "customer",
      "quote": "This is blocking our finance team and we still do not have a workaround.",
      "supports": ["business_impact", "resolution_confidence", "case_sentiment_score"]
    }
  ],
  "explanation": "The customer remains materially blocked with no acceptable workaround. Although the language is professional, the unresolved high business impact supports a Negative score.",
  "scoring_rules_applied": [
    "Unresolved impact cap",
    "Score 2 negative-but-not-critical rule"
  ]
}
````

---

## 16. Trajectory definitions

Assign one trajectory value.

|Value|Meaning|
|---|---|
|`improving`|Customer sentiment or resolution confidence improves over time|
|`worsening`|Customer becomes more frustrated, impact grows, or delays accumulate|
|`stable`|Sentiment remains similar throughout|
|`mixed`|Some aspects improve while others remain negative|
|`unknown`|Not enough chronological evidence|

Trajectory should not replace the final score.

Example:

- A case can be `case_sentiment_score = 2` and `trajectory = improving` if the customer is still impacted but now has a credible workaround.
- A case can be `case_sentiment_score = 4` and `trajectory = worsening` only in rare cases where the latest message is positive but earlier messages showed a decline before recovery; explain clearly.

---

## 17. Examples

### Example 1: Polite but severely impacted customer

Customer:

> “Hi, our production login is down for all users. We have no workaround. Please help.”

Scoring:

```
{
  "case_sentiment_score": 1,
  "case_sentiment_label": "Very Negative / Critical Risk",
  "dimensions": {
    "emotional_tone": { "score": 0, "label": "Neutral or unclear" },
    "business_impact": { "score": 4, "label": "Critical impact" },
    "customer_effort": { "score": 0, "label": "No notable effort" },
    "resolution_confidence": { "score": -1, "label": "Unresolved or doubtful" },
    "escalation_or_churn_risk": { "score": 2, "label": "Elevated urgency/escalation" }
  },
  "trajectory": "unknown",
  "confidence": "high",
  "explanation": "The customer is polite, but the unresolved production-down condition for all users is critical."
}
```

---

### Example 2: Angry customer with minor issue

Customer:

> “This is ridiculous. I can’t believe I can’t change this icon color.”

Scoring:

```
{
  "case_sentiment_score": 2,
  "case_sentiment_label": "Negative",
  "dimensions": {
    "emotional_tone": { "score": -2, "label": "Strong negative emotion" },
    "business_impact": { "score": 1, "label": "Low impact" },
    "customer_effort": { "score": 0, "label": "No notable effort" },
    "resolution_confidence": { "score": 0, "label": "Unknown or pending" },
    "escalation_or_churn_risk": { "score": 0, "label": "No risk stated" }
  },
  "trajectory": "unknown",
  "confidence": "high",
  "explanation": "The customer expresses strong frustration, but the described impact is low and there is no churn, escalation, or critical business risk."
}
```

---

### Example 3: Generic thanks but unresolved

Customer:

> “Thanks. Please let me know when this is fixed because our team is still blocked.”

Scoring:

```
{
  "case_sentiment_score": 2,
  "case_sentiment_label": "Negative",
  "dimensions": {
    "emotional_tone": { "score": 0, "label": "Neutral or unclear" },
    "business_impact": { "score": 3, "label": "High impact" },
    "customer_effort": { "score": 1, "label": "Low effort" },
    "resolution_confidence": { "score": -1, "label": "Unresolved or doubtful" },
    "escalation_or_churn_risk": { "score": 1, "label": "Mild urgency" }
  },
  "trajectory": "stable",
  "confidence": "high",
  "explanation": "The word 'Thanks' is polite but does not indicate satisfaction. The team remains blocked."
}
```

---

### Example 4: Resolved with explicit praise

Customer:

> “That fixed it. Excellent support and very fast turnaround — thank you!”

Scoring:

```
{
  "case_sentiment_score": 5,
  "case_sentiment_label": "Very Positive",
  "dimensions": {
    "emotional_tone": { "score": 2, "label": "Strong positive emotion" },
    "business_impact": { "score": 0, "label": "No impact stated" },
    "customer_effort": { "score": 0, "label": "No notable effort" },
    "resolution_confidence": { "score": 2, "label": "Explicitly resolved and satisfied" },
    "escalation_or_churn_risk": { "score": 0, "label": "No risk stated" }
  },
  "trajectory": "improving",
  "confidence": "high",
  "explanation": "The customer confirms resolution and gives explicit strong praise."
}
```

---

### Example 5: Sarcasm

Customer:

> “Great, another outage. This is exactly what we needed before our launch.”

Scoring:

```
{
  "case_sentiment_score": 1,
  "case_sentiment_label": "Very Negative / Critical Risk",
  "dimensions": {
    "emotional_tone": { "score": -2, "label": "Strong negative emotion" },
    "business_impact": { "score": 4, "label": "Critical impact" },
    "customer_effort": { "score": 0, "label": "No notable effort" },
    "resolution_confidence": { "score": -1, "label": "Unresolved or doubtful" },
    "escalation_or_churn_risk": { "score": 2, "label": "Elevated urgency/escalation" }
  },
  "trajectory": "unknown",
  "confidence": "medium",
  "explanation": "The apparently positive word 'Great' is sarcastic in context. The outage affects a launch, indicating critical impact."
}
```

---

## 18. Production QA requirements

To maintain consistency:

1. Maintain a labeled calibration set of real anonymized cases.
2. Include edge cases:
    - Polite critical outages
    - Angry minor issues
    - Churn threats
    - Resolved after high effort
    - Multi-participant cases
    - Sarcasm
    - Non-native English
    - Agent-heavy conversations
3. Measure inter-rater agreement between human reviewers.
4. Review disagreement cases monthly.
5. Track score distributions by product, region, language, and account segment to detect bias.
6. Version this rubric and log rubric version with every score.
7. Require evidence quotes for auditability.
8. Do not use sentiment score alone for punitive agent evaluation.
9. Separate product dissatisfaction from support-agent performance where possible.
10. Recalibrate when support processes, products, or customer segments change.

---

## 19. Common failure modes to avoid

Avoid these errors:

- Scoring agent sentiment instead of customer sentiment
- Treating “thank you” as satisfaction
- Treating politeness as neutrality
- Ignoring business impact
- Ignoring unresolved status
- Averaging away a churn threat
- Overweighting profanity without considering impact
- Assigning 5 when the issue is merely acknowledged
- Assigning 1 for any complaint regardless of severity
- Inferring sentiment from demographics or writing style
- Using case priority as a substitute for customer sentiment
- Ignoring long-running effort
- Ignoring sarcasm
- Treating resolved-by-agent status as customer-confirmed resolution

---

## 20. Minimal deterministic checklist

Before finalizing the score, answer these questions:

1. Did I use only customer-authored text as sentiment evidence?
2. Is there explicit positive or negative language?
3. What business impact did the customer state?
4. Is the issue resolved according to the customer?
5. How much effort or friction has the customer experienced?
6. Is there any churn, legal, refund, executive, or public escalation risk?
7. Do any score-1 triggers apply?
8. Do any positive-score caps apply?
9. Did I provide evidence quotes?
10. Is the confidence level appropriate?

If uncertain and no strong evidence exists, use score 3 with low or medium confidence.

---

## 21. Final instruction

Always prioritize observable customer evidence, explicit impact, unresolved friction, and risk signals over superficial word polarity.

The final sentiment score must be explainable, auditable, and reproducible.

```
undefined
```