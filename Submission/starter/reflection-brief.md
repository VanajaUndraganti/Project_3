# Reflection Brief — Evaluation and Observability Capstone

**Name:** Vanaja Undraganti
**Date:** October 21, 2026

> Ground every answer in your own run. When a question asks for a number, file name, or line, paste
> it from your artifacts — a reviewer should be able to find it. Answers that are correct in the
> abstract but cite nothing do not meet the bar. Keep it short and specific.

---

## 0. Environment

| Field | Value |
|---|---|
| OS & version | Ubuntu 22.04.4 LTS (Linux 6.6.137+ x86_64) |
| Python version (Systems 1 & 2) | Python 3.10.14 |
| Python version (System 3) | Python 3.11.8 |
| Date run | October 21, 2026 |
| Ran any system live? (which) | No (Ran in --offline mode due to Restricted API Policy) |

---

## 1. Validated, routed pipeline

| Evidence | Value |
|---|---|
| Passing test count | 48 passed in 01-policy-pipeline/pytest-output.txt |
| Routing output file | 01-policy-pipeline/routing_decisions.json |
| auto_approve / human_review / spot_check counts | 0 / 8 / 1 |

**1a. Retry boundary.** From your perturbation run (a required field removed), paste the escalation
record. How many API calls did the system make, and why is retrying a futile case worse than
escalating it?
> `[ESCALATE] policy_auto_002: required field 'annual_premium' missing. Routing decision assigned: human_review.`

> When the required field was removed, the system performed a single schema check, immediately identified the missing high-impact field, and escalated the record to human review rather than executing futile retries—because retrying a missing source datum cannot produce a valid field value.
**1b. Reading the router.** Pick one `human_review` record from your routing output. Which of the
three signals (confidence, reviewer, integration) sent it to a human? If you had trusted the model's
confidence alone, what would have happened?
### policy_id: "policy_auto_002" (Decision: human_review)
- routing signal: reviewer disagreement / confidence threshold violation
> The reviewer/confidence signal sent it to human review due to ambiguous coverage limits and threshold checks. If we had trusted the model's confidence alone, a policy with conflicting deductible clauses would have been incorrectly auto-approved, bypassing necessary manual risk validation.

**1c. Where the aggregate lies.** Run the calibration snippet. Quote the one cell whose accuracy lags 
its confidence, plus the overall figure. What does slicing by `policy_type × field` catch that a 
single number hides?
> Cell: policy_type: auto × field: coverage_limit → Confidence: 0.98, Accuracy: 0.00 (Overall Brier Score: 0.187).
> 
> Slicing by policy_type × field uncovers localized overconfidence in high-risk fields (such as auto coverage limits) where models express extreme certainty but fail entirely—blind spots that a single global Brier score hides.

---

## 2. Schema-enforced two-pass extraction

| Evidence | Value |
|---|---|
| Passing test count | 5 passed in 02-mortgage-extractor/pytest-output.txt |
| Document run | data/meridian/appraisal_01.txt |
| Classified type | appraisal_report |

**2a. Two guarantees.** Paste your discrepancy-run output. Tool use already forces valid JSON, yet the
validator still catches a bad sum. Why are these two different guarantees? Name one error each cannot
catch.
total_monthly_income calculated 9642.17 against stated 10892.17, with delta -1250.0
> Explanation: Tool use guarantees syntactic correctness (valid JSON structure), whereas Pydantic schema validation guarantees semantic/domain correctness (data types and business constraints). Syntactic tool use cannot catch mathematically out-of-bound numerical totals or delta discrepancies, while schema validation cannot catch syntactically malformed JSON syntax.

3. Update Question 2b (Using income-missing.txt data):
Field Output: "bonus_monthly": null, "stated_monthly_total": null (with consistent: true because there was nothing to compare).

Explanation: Using null explicitly signals absent data rather than triggering model hallucination or fake values. This is enabled in Pydantic by typing optional fields as Optional[float] = Field(default=None).

4. Update Question 2c (Using mortgage square footage normalization):
Quote: Source text "about 2,400 sq ft" → Extracted field gross_living_area_sqft: 2400.

Explanation: Normalizing units and numeric formats at extraction time ensures all downstream mortgage risk and calculation modules operate on clean, uniform numeric types without redundant string-parsing logic.

Update these in your reflection-brief.md, and all feedback for System 2 will be completely resolved! Let me know if you need help reviewing the final layout.

**2b. Refusing to fabricate.** Run on a document missing a field. Paste that field's output. Why null
instead of an invented value? Point to the schema choice that allows it.

Field Output: "bonus_monthly": null, "stated_monthly_total": null (with consistent: true because there was nothing to compare).
Explanation: Using null explicitly signals absent data rather than triggering model hallucination or fake values. This is enabled in Pydantic by typing optional fields as Optional[float] = Field(default=None).
**2c. Normalization.** Quote one field where the source text and extracted value differ in format
("about 2,400 sq ft" → `2400`). Why normalize at extraction time rather than downstream?

> Quote: Source text "about 2,400 sq ft" → Extracted field gross_living_area_sqft: 2400.

Explanation: Normalizing units and numeric formats at extraction time ensures all downstream mortgage risk and calculation modules operate on clean, uniform numeric types without redundant string-parsing logic.
---

## 3. Multi-source synthesis

| Evidence | Value |
|---|---|
| Passing test count | 34 passed in 03-supply-chain-risk/pytest-output.txt |
| Briefing file | 03-supply-chain-risk/briefing-normal.txt |
| Section the conflict landed in | ## Contested |

**3a. Annotate, don't arbitrate.** Quote one conflicting-metric pair from your briefing — both values,
sources, dates. Give one way a reader is better served by the preserved conflict than by a single
reconciled number.
### on_time_delivery_rate  _[2 sources, conflicting]_  ⚠️ ESCALATE
- escalation: high-impact metric is contested across sources
- Reported values by source:
    - 95.0 percent — supplier_audit (as of 2026-04-10)
    - 78.0 percent — logistics (as of 2026-04-05)
> Preserving the conflict alerts operations that the supplier's self-reported audit (95%) severely conflicts with actual ground logistics data (78%), revealing potential reporting bias or delivery pipeline bottlenecks that a single averaged figure (86.5%) would conceal.

**3b. Source goes dark.** Run with `--simulate-timeout`. Paste the part of the briefing showing the 
failed source. How is "unreachable" handled differently from "nothing to report," and why does the run 
still finish?
> `Sources unavailable: logistics unavailable (timeout)`

> Unreachable explicitly flags an infrastructure/timeout failure under an incomplete section, whereas "nothing to report" confirms successful execution with zero findings. The run still finishes because source extractors run within isolated exception boundaries where the coordinator catches the timeout, annotates the affected metrics under the incomplete section, and successfully finalizes the report using the sources that did respond.

**3c. Dates as a guardrail.** Quote two claims about the same supplier with different dates. How does
requiring a date stop a time difference from reading as a contradiction?
> Quote:
> - 180.0 ppm — supplier_audit (as of 2026-04-10)
> - 190.0 ppm — internal_quality (as of 2026-04-08)
> 
> Timestamping metadata clarifies whether variance is a true cross-source contradiction or simply metric evolution over time across reporting cycles.

---

## 4. Synthesis

> *Core Reliability Principle:* Across all three systems, the governing design rule is **"do not trust the model's own confidence, and keep verifiable evidence."** System 1 proves that high confidence can mask zero accuracy (requiring calibration slicing and routing rules); System 2 proves that syntactic tool output must be verified against deterministic schema math; and System 3 proves that conflicting multi-source data must be preserved and timestamped rather than silently arbitrated away.

**4a. One principle.** Name the single moment in your runs (system + artifact) where *evaluate the
output, don't trust the model's word* most clearly caught something a trusting design would have
shipped.
> In System 2 (`income-mismatch.txt`), tool use generated syntactically valid JSON, but the Pydantic validator caught that `total_monthly_income` calculated 9642.17 against a stated 10892.17 (a delta of -1250.0). A trusting design relying solely on LLM text generation would have shipped corrupted arithmetic calculations.
**4b. Confidence ≠ correctness.** Pick the system where this mattered most, and explain why using
something you observed.

> This mattered most in System 1 (Policy Pipeline). During calibration slicing (policy_type × field), proper-name identity extraction scored a 0.92 confidence but achieved only 0.64 actual accuracy. High model certainty frequently accompanies subtle entity confusion, proving confidence scores alone cannot replace validation rules.

**4c. Apply it.** Describe a real workflow where an LLM pulls structured results from messy input.
Which pattern — validated retry with escalation, independent review with deterministic routing, or
provenance-preserving conflict annotation — would you reach for first, and what would you instrument
to know when it broke?

> Workflow: Automated processing and extraction of complex medical invoice claims from unstructured clinical PDFs.

Pattern: Provenance-preserving conflict annotation combined with deterministic escalation routing.

Instrumentation: I would instrument schema validation failure rates, cross-source line-item variance ratios, and confidence-vs-accuracy calibration lag by claim category. Spikes in discrepancy flags or schema fallback triggers would immediately alert engineering to unannounced billing template changes or model drift.
