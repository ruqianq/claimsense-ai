# Architecture Decision Log

These are the initial decisions for the five-week ClaimSense AI proof of concept (POC). They reflect the capstone constraints and current discovery assumptions. Client-owned interfaces, reviewer identities, and approval responsibilities remain subject to confirmation.

## ADR-001: Keep the POC integration read-only

- **Date:** 2026-11-19
- **Status:** Accepted for the initial POC
- **Decision:** The POC will use read-only interfaces over synthetic representations of Claims Management, Policy Administration, and the Document Store. It will not connect to production systems or write recommendations, decisions, or other changes back to the three client systems. Synthetic data only.
- **Reason:** Read-only access limits the POC's effect on client records while integration details, system permissions, and the official location for decision records are still being discovered. Synthetic data is a fixed capstone constraint.
- **Consequences:** The co-pilot's recommendation and evidence are returned for review in the POC interface. Any later production connection or write-back requires a separate decision covering the destination record, authorization, audit trail, and failure/retry behavior.

## ADR-002: Surface uncertainty and flag unsupported cases for review

- **Date:** 2026-11-19
- **Status:** Accepted for the initial POC
- **Decision:** When evidence is incomplete, conflicting, or insufficient to support a judgment, the co-pilot must identify the uncertainty and mark the claim for further human review in the POC. It must not present an unsupported judgment as a confident recommendation or imply that it has placed the claim in a client workflow queue.
- **Reason:** Claims may have missing documents, conflicting information, or policy language that does not support a clear first-pass conclusion. Exposing those gaps gives the reviewer a safer basis for deciding what to do next.
- **Consequences:** The response should state what is uncertain and cite the available evidence or identify the missing evidence. Evaluation cases must include incomplete, conflicting, and insufficient-evidence examples. The client must confirm the preferred review queue or handoff path for any future integrated workflow.

## ADR-003: Keep the co-pilot advisory; require licensed adjuster confirmation

- **Date:** 2026-11-19
- **Status:** Accepted as a capstone constraint; reviewer assignment to confirm
- **Decision:** The co-pilot provides a first-pass recommendation only. A client-designated licensed adjuster reviews and confirms every outcome. The POC presents this as a distinct review step; it does not make a final adjudication or update the client system of record.
- **Reason:** The capstone requires a licensed adjuster to remain accountable for every outcome. Human confirmation also provides a review point for model judgments and uncertainty.
- **Consequences:** The POC must make the recommendation distinguishable from the adjuster's confirmed outcome. The client must identify which licensed adjusters may review claims and confirm how the POC should represent their confirmation. Persisting confirmation to a client system is out of scope for the initial POC.

## ADR-004: Separate deterministic operations from model judgment

- **Date:** 2026-11-19
- **Status:** Accepted as a fixed capstone constraint
- **Decision:** Implement identity and access control, policy-date checks, and monetary calculations deterministically in code. Use model judgment for interpretation and assessment that require reading claim evidence, including policy-clause interpretation and fraud-related inconsistencies. Every model-based judgment must cite the evidence it relies on. Authorization must be checked before any model is invoked.
- **Reason:** Access decisions and calculations need predictable, testable behavior. Evidence-based judgment can use the model, but the supporting evidence must be visible to the adjuster.
- **Consequences:** Keep deterministic results and model judgments distinguishable in the response. Evaluation cases must verify that unauthorized requests are refused before model use, calculations and policy dates follow code-based rules, and model judgments cite supporting evidence.
