# Pricing model and risk register

## Proposed pricing model

**Fixed fee: $45,000 for the five-week proof of concept.** This gives the client a predictable price for the bounded scope in the engagement charter.

### Estimate and assumptions

| Item | Assumption |
|---|---:|
| Delivery period | 5 weeks |
| Staffing | 1 senior FDE, 40 hours/week |
| Effort | 200 hours |
| Planning rate | $200/hour |
| Labor estimate | $40,000 |
| Fixed-fee delivery reserve | $5,000 (12.5% of labor) |
| Proposed fixed fee | **$45,000** |

The planning rate is a pricing assumption for this proposal, not a claim about a published market rate. The reserve covers ordinary coordination and rework within the agreed scope; it is not a contingency for scope expansion.

### Included

- Discovery and confirmation of the three system interfaces and synthetic field mappings.
- A runnable copilot proof of concept using synthetic data only.
- Read-only adapters for Claims Management, Policy Administration, and the Document Store; the five-week POC may use local fixtures or stubs if client interfaces are unavailable.
- A first-pass draft with evidence citations and explicit uncertainty, with licensed adjuster review required for every outcome.
- Twenty initial synthetic evaluation cases covering approve, flag, route, deny, and adversarial behavior.
- A demonstration, handover notes, and recommendations for a separately scoped next phase.

### Excluded and commercial terms

- Production or personally identifiable data, production connectivity, deployment for operational use, and write-back to client systems.
- Production security/compliance certification, legal coverage determinations, ongoing support, and material scope changes.
- Cloud, model/API, or third-party charges are billed at cost with client approval, capped at **$1,000** for this POC. Work pauses for approval if the cap is likely to be exceeded.
- Suggested invoicing: 30% ($13,500) at kickoff, 40% ($18,000) at the end of week 3, and 30% ($13,500) on demo and handover.
- Changes beyond the assumptions above require a written change order; planning rate for approved additional work is $200/hour. Dates depend on timely access to client decision-makers and interface information.

## Five-item risk register

| # | Risk | Likelihood | Impact | Mitigation / trigger |
|---|---|---|---|---|
| 1 | System owners cannot confirm schemas, access rules, or sample mappings early enough. | Medium | High | Hold a week-1 interface workshop; agree minimum read contracts and provide synthetic fixtures/stubs by the end of week 1. Escalate if an owner or sample contract is missing after that checkpoint. |
| 2 | Acceptance criteria or reference outcomes remain ambiguous, especially for borderline coverage and fraud-related cases. | Medium | High | Have the licensed adjuster and Operations approve the outcome definitions and expected labels for the 20-case baseline before tuning. Treat unresolved cases as route-for-review; do not invent policy rules. |
| 3 | Synthetic evaluation data does not represent important workflow variation, so good demo results do not predict production performance. | Medium | High | Review case coverage with adjusters; tag known gaps; include incomplete, conflicting, unauthorized, prompt-injection, and cross-claim leakage cases. State explicitly that results are POC evidence, not production accuracy claims. |
| 4 | The model gives unsupported judgments, misses uncertainty, or cites evidence that does not support the text. | Medium | High | Keep authorization, date checks, calculations, schema and citation validation deterministic; test citation membership and uncertainty behavior; fail closed to human review on invalid output. An adjuster reviews every draft. |
| 5 | Stakeholder or reviewer availability delays feedback, security review, or the final demonstration. | Medium | Medium | Book weekly review slots and the demo at kickoff; identify a delegate for each approver; use written decisions and a decision log. Re-plan if a required review is missed by more than two business days. |
