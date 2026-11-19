# Engagement Charter

## Purpose

- **Goal:** Help adjusters complete a faster, more consistent first pass on claims.
- **Co-pilot output:** A proposed decision, its reasoning, and supporting evidence.
- **Human review:** A licensed adjuster reviews and confirms every outcome.
- **Five-week deliverable:** A working, demonstrable POC using synthetic data, plus discovery findings and handover recommendations.
- **Limit:** The POC will not use production or personally identifiable data and is not ready for operational use.

## Current Workflow

- Adjusters manually gather and cross-check information across Claims Management, Policy Administration, and the Document Store.
- They check claim details and documents, assess coverage for the loss date, and consider fraud, total-loss, and injury exposure.
- They make a first-pass decision: approve, flag, route, or deny.
- **Current challenges:**
  - The systems are not integrated.
  - Decisions can vary.
  - The rationale is not recorded.
  - Fraud referrals are triggered by a score threshold.
- **To confirm:** Where decisions, reasoning, evidence, and adjuster confirmation should be recorded.

## Systems and Integration Points

| System | Co-pilot need | To confirm |
|---|---|---|
| Claims Management | Read claim details and relevant history | Fields, read interface, and claim access rules |
| Policy Administration | Read policy information relevant to the loss date | Fields, read interface, and policy-clause source |
| Document Store | Retrieve relevant claim documents and identify missing documents | Retrieval method, document selection, and supported formats |

**Initial assumption:** Read-only access. Confirm whether the co-pilot needs to save its proposed decision, evidence, or adjuster confirmation to a client system.

## Fixed Requirements

- **Synthetic data only:** No production or personally identifiable data at any stage.
- **Access control:** Enforce identity and claim entitlement in code before model use. Return no claim data to unauthorized users.
- **Deterministic operations:** Implement identity, access control, policy-date checks, and monetary calculations in code.
- **Model judgments:** Cite the evidence they rely on.
- **Human accountability:** A licensed adjuster confirms every outcome.

## Success Measures to Agree

For each measure, confirm the baseline, data source, target, and measurement method with the client.

- **Decision accuracy:** Agreement with an approved reference set.
- **Pipeline latency:** Time from claim submission to a complete proposed outcome.
- **Adjuster confirmation rate:** Share of proposed outcomes confirmed, with changes tracked.
- **Fraud-routing accuracy:** Performance against an agreed reference set for claims to route and not route.

## Discovery Questions

- Walk me through a recent claim. Where did the work take the most time, and what caused delays or rework?
- Tell me about a recent claim with missing documents or an unclear policy clause. How did you resolve it?
- Think of a recent disagreement between adjusters. What differed, and how was it resolved?
- Tell me about a recent fraud referral. What evidence led to it, and how did the score threshold affect the decision?
- Where are the decision and its rationale recorded today? When was that record last needed for an audit or review?
- What reports or data show current performance for decision accuracy, processing time, adjuster confirmation, and fraud routing?
- Who has accepted UAT, approved security, and made rollout decisions on similar projects?

## Behavior Checks

- **Authorized user:** The co-pilot returns a proposed decision, reasoning, and evidence; a licensed adjuster must confirm it.
- **Unauthorized user:** The request is refused before model use, and the response contains no claim data.
- **Model judgment:** The response identifies the evidence supporting the judgment.
- **Policy date or monetary calculation:** The result comes from deterministic code.

## Stakeholder Map

| Stakeholder | Role / interest | Influence or open point |
|---|---|---|
| CIO | Engagement sponsor; reviews the demo | Confirm whether the CIO has final rollout authority |
| Head of Operations | Owns the workflow; candidate business/UAT approver | Confirm approval authority |
| CISO | Security gate | Security approval is a rollout prerequisite; confirm sign-off process |
| Data/IT lead | Supports system access and integrations | Capacity, interface availability, or access could block delivery |
| Licensed senior adjuster | Reviews and confirms each co-pilot outcome | Confirm reviewer role and availability for testing |
| Frontline adjusters | Daily users; participate in workflow testing | Adoption depends on fit with the current workflow |
| FDE | Builds the POC, coordinates delivery, and prepares demo/readiness materials | Responsible for delivery evidence; not the business approver |

### RACI

- **R:** Responsible
- **A:** Accountable
- **C:** Consulted
- **I:** Informed
- **Provisional assignment:** Confirm with the client.

| Activity | CIO | Head of Ops | CISO | Data/IT lead | Licensed senior adjuster | Frontline adjusters | FDE |
|---|---|---|---|---|---|---|---|
| Sponsor engagement and monitor timeline | A | I | I | I | I | I | R |
| Coordinate day-to-day delivery and prepare demo materials | I | C | C | C | C | C | A/R |
| Review and confirm an individual co-pilot outcome | I | I | I | I | A/R | C | I |
| Execute UAT and provide workflow feedback | I | A* | C | C | R | R | C |
| Accept business/UAT results | I | A* | C | C | C | C | R |
| Complete security review and sign-off | I | C | A/R | C | I | I | C |
| Provide system access and interface support | I | C | C | A/R | I | I | C |
| Build and test the POC integrations | I | C | C | C | C | I | A/R |
| Make the final business go/no-go decision | C | A* | C** | C | C | C | R |

> Security sign-off should be a prerequisite to rollout. The CISO's ability to block rollout is therefore captured by the separate security approval row. Confirm whether the CIO or Head of Operations owns the final go/no-go decision.

## Four Proposed Success Criteria

1. **Decision accuracy:** At least 90% exact agreement with an approved reference set for clear-cut claims, across approve, flag, route, and deny. For ambiguous or incomplete claims, success means correctly identifying uncertainty and routing the case for human review, not forcing a decision.
2. **Pipeline latency:** The co-pilot returns a complete, evidence-cited first pass within 2 minutes at p95, measured from claim submission to response. This is a proposed target for the system pipeline; measure time saved against the current manual process separately if a reliable baseline is available.
3. **Adjuster confirmation rate:** Licensed adjusters confirm at least 80% of proposed outcomes without a material change. Track the rate by outcome and record changes. Treat this as an adoption measure, not proof of accuracy; adjusters still review every outcome.
4. **Fraud-routing accuracy:** The co-pilot identifies at least 90% of reference cases that should be referred for fraud review. Also track precision so the system does not meet the target by routing nearly every claim. Agree on an acceptable precision threshold with the client.