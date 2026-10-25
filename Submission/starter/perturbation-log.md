# Perturbation Log

For each system, make one deliberate change to an input or configuration, predict the outcome, run
it, and record what actually happened. See the starters in the Instructions, or design your own (your
own experiment earns more credit).

---

### System 1 — validated, routed pipeline

=== SYSTEM 1: POLICY PIPELINE PERTURBATION ===
Change: Modified extraction values in 'data/policies/POL-2025-002.json' to induce divergences between the primary extractor and secondary review pass.
Command: PYTHONPATH=. python3 -m policy_extractor data/policies/ > evidence-pack/01-policy-pipeline/pipeline-run.txt
Prediction: The validation pipeline will detect divergences during secondary review, logging specific field disagreements and overriding standard approval to route the record to human review.
Result:
Policy ID: POL-2025-002
Decision: human_review
Reason: reviewer_disagreement=['coverage_limit', 'deductible', 'endorsements']
Reviewer Disagreements: ['coverage_limit', 'deductible', 'endorsements']
Contrast:
Baseline run (unperturbed): POL-2025-002 processed with uniform consensus across extracted fields.
Perturbed run: Reviewer disagreements across coverage_limit, deductible, and endorsements successfully forced an escalation into human_review, matching routing_decisions.json.

---

### System 2 — schema-enforced two-pass extraction

=== SYSTEM 2: MORTGAGE EXTRACTOR PERTURBATION ===
Change: Altered base monthly income from $8,392.17 to $7,142.17 in 'data/mortgage/income_doc_01.txt' while keeping stated_monthly_total at $10,892.17.
Command: python -m mortgage_extractor.main --input data/mortgage/income_doc_01.txt --output 02-mortgage-extractor/income-mismatch.txt
Prediction: Tool use will generate valid JSON, but Pydantic schema validation will report a sum consistency discrepancy.
Result:
"validation": { "consistent": false, "discrepancies": ["total_monthly_income calculated 9642.17 against stated 10892.17, with delta -1250.0"] }
Contrast:
Baseline run: Income line items summed up correctly to stated total with "consistent": true.
Perturbed run: Tool extraction succeeded as valid JSON, but validator flagged a delta discrepancy of -1250.0.

---

### System 3 — multi-source synthesis

=== SYSTEM 3: MULTI-SOURCE SYNTHESIS PERTURBATION ===
Change: Updated 'on_time_delivery_rate' in logistics source file 'data/supply_chain/logistics_01.json' from 78.0% to 95.0% to match supplier audit data.
Command: python -m supply_chain.synthesize --data-dir data/supply_chain --output 03-supply-chain-risk/briefing-perturbed.txt
Prediction: The metric 'on_time_delivery_rate' will move from ## Contested to ## Well-Established.
Result:
Under '## Well-Established': on_time_delivery_rate: 95.0% (Supported by supplier_audit, logistics)
Contrast:
Baseline run (briefing-normal.txt): Metric was flagged under '## Contested' due to 95% vs 78% discrepancy between audit and logistics.
Perturbed run (briefing-perturbed.txt): Metric moved to '## Well-Established' as both sources agreed on 95.0%.
