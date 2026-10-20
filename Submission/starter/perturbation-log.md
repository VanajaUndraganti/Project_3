# Perturbation Log

For each system, make one deliberate change to an input or configuration, predict the outcome, run
it, and record what actually happened. See the starters in the Instructions, or design your own (your
own experiment earns more credit).

---

### System 1 — validated, routed pipeline

- **Change I made (file + what I changed):** Modified the intent routing confidence threshold from 0.80 to 0.99 in the router configuration file.
- **Command I ran:** pytest tests/test_pipeline.py
- **What I predicted:** High-confidence items will pass, but borderline or ambiguous insurance policy queries will fail the strict threshold and route directly to the manual review queue.
- **What actually happened (paste the key output line):** 
  [ROUTER] Policy ID #4829 confidence (0.91) below threshold (0.99) -> Route: MANUAL_REVIEW_QUEUE
- **How this differs from the unperturbed run:** In the unperturbed run (threshold 0.80), those borderline queries were processed automatically. Raising the threshold forced them into human review.
---

### System 2 — schema-enforced two-pass extraction

- **Change I made (file + what I changed):** Modified the mortgage extraction JSON schema to make an optional field (`borrower_middle_name`) required (`required: true`) with `additionalProperties: false`.
- **Command I ran:** python extract.py --sample mortgage_form.pdf
- **What I predicted:** Pass 1 extraction will omit the middle name when missing, causing a schema validation crash and triggering the Pass 2 self-healing schema-repair loop.
- **What actually happened (paste the key output line):** 
  [ValidationError] 'borrower_middle_name' is a required property. Triggering Pass 2 Schema Repair retry...
- **How this differs from the unperturbed run:** The unperturbed run passed validation cleanly on Pass 1 without needing the second-pass repair loop.
---

### System 3 — multi-source synthesis

- **Change I made (file + what I changed):** Restricted the max context tokens allocated to secondary supplier reports from 4096 down to 500 tokens in the synthesizer configuration.
- **Command I ran:** python synthesize.py --sources=supply_chain_docs
- **What I predicted:** Context truncation will cause the model to miss details from secondary supplier reports and rely solely on the primary vendor document.
- **What actually happened (paste the key output line):** 
  [Synthesis Warning] Truncated secondary sources. Risk score calculated using Primary Source only: HIGH_RISK
- **How this differs from the unperturbed run:** The baseline run successfully synthesized insights from multiple conflicting sources to form a balanced risk report.
