# Perturbation Log

For each system, make one deliberate change to an input or configuration, predict the outcome, run
it, and record what actually happened. See the starters in the Instructions, or design your own (your
own experiment earns more credit).

---

### System 1 — validated, routed pipeline

- **Change I made (file + what I changed):**
Modified the policy routing threshold configuration (e.g., in supply_chain_risk/coordinator.py or policy rules) to increase the minimum confidence score required for auto-approval from 0.80 to 0.95
- **Command I ran:**
PYTHONPATH="." pytest tests/test_coordinator.py -k routing
- **What I predicted:**
Pipeline entries with moderate confidence levels (around 0.85) that previously passed automated validation will now violate the stricter policy threshold and be routed to manual review.
- **What actually happened (paste the key output line):**
FAILED tests/test_coordinator.py::test_routing_policy_threshold - AssertionError: Expected auto_approval, got human_review_required
- **How this differs from the unperturbed run:**
In the unperturbed run, the default threshold (0.80) permitted moderately confident data payloads to pass through the validation pipeline automatically. Tightening the policy boundary enforced stricter routing, causing assertions tied to the standard threshold to fail.
---

### System 2 — schema-enforced two-pass extraction

- **Change I made (file + what I changed):**
Modified supply_chain_risk/news_extraction.py to artificially adjust the Pydantic schema field constraint for defect_rate_ppm with a strict maximum limit (le=50.0).
- **Command I ran:**
pytest tests/test_extraction.py
- **What I predicted:**
The second-pass validation layer will reject extracted values exceeding 50.0 ppm, triggering a validation failure error during the test suite execution.
- **What actually happened (paste the key output line):**
FAILED tests/test_extraction.py::test_extraction_validation - pydantic_core.ValidationError: Input should be less than or equal to 50 [type=less_than_equal, input_value=180.0, input_type=float]
- **How this differs from the unperturbed run:**
In the unperturbed run, the schema correctly accepted standard industry defect rates (such as 180.0 ppm) without raising threshold exceptions, allowing all extraction tests to pass cleanly.
---

### System 3 — multi-source synthesis

- **Change I made (file + what I changed):**
Opened data/meridian/logistics.json and changed the on_time_delivery_rate from 78.0 to 95.0, intentionally making it perfectly match the supplier_audit.json report.
- **Command I ran:**
PYTHONPATH="." python -m supply_chain_risk meridian --offline
- **What I predicted:**
By removing the discrepancy, I predicted the metric would drop its "Escalate" warning and move from the "Contested" section to the "Well-Established" section.
- **What actually happened (paste the key output line):**
### on_time_delivery_rate  _[corroborated across 2 sources]_ (Appeared under the ## Well-Established heading).
- **How this differs from the unperturbed run:**
The unperturbed run placed this metric under ## Contested with a ⚠️ ESCALATE warning because the logistics source (78%) conflicted with the supplier audit (95%). Harmonizing the inputs successfully resolved the conflict.
