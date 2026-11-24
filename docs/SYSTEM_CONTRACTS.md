# Typed client-system contracts

These contracts describe the POC boundary for Claims Management, Policy Administration, and the Document Store. They are language-neutral shapes with Python-style type names. Implement adapters as read-only operations. The POC uses synthetic fixtures/stubs where client interfaces are not available.

## Shared types

```python
from datetime import date, datetime
from decimal import Decimal
from enum import StrEnum
from typing import Literal

SystemName = Literal["claims", "policy", "documents"]

class CoverageKind(StrEnum):
    LIABILITY = "liability"
    COLLISION = "collision"
    COMPREHENSIVE = "comprehensive"
    PERSONAL_INJURY = "personal_injury"
    UNINSURED_MOTORIST = "uninsured_motorist"
    OTHER = "other"

class DocumentKind(StrEnum):
    CLAIM_REPORT = "claim_report"
    POLICY_DECLARATIONS = "policy_declarations"
    POLICY_CLAUSE = "policy_clause"
    POLICY_ENDORSEMENT = "policy_endorsement"
    POLICE_REPORT = "police_report"
    PHOTO = "photo"
    ESTIMATE = "repair_estimate"
    INVOICE = "invoice"
    WITNESS_STATEMENT = "witness_statement"
    OTHER = "other"

class SourceRef:
    system: SystemName
    record_id: str
    version: str
    retrieved_at: datetime

class AdapterError:
    code: Literal[
        "NOT_FOUND", "FORBIDDEN", "INVALID_REQUEST",
        "UNAVAILABLE", "UPSTREAM_ERROR"
    ]
    retryable: bool
    message: str  # sanitized; never include another claim's data
    correlation_id: str

class EntitledReadContext:
    request_id: str
    principal_id: str
    authorized_claim_id: str
```

All IDs are opaque strings. Dates are ISO-8601 calendar dates; timestamps are UTC RFC 3339. Monetary values use decimal strings / Decimal with an explicit currency, never binary floating point. Adapters must enforce the supplied entitlement context and return only records belonging to `authorized_claim_id`. Unknown fields should be rejected at the adapter boundary; secrets, raw identity tokens, and unrelated claim records must not be returned.

## 1. Claims Management

```python
class ClaimReadRequest:
    context: EntitledReadContext
    claim_id: str

class LossFacts:
    loss_date: date | None
    loss_type: str | None
    reported_summary: str
    jurisdiction_code: str | None  # synthetic code in this POC

class VehicleRef:
    vehicle_id: str
    vin_token: str  # synthetic token; never a real VIN in the POC

class ClaimHistoryItem:
    event_id: str
    occurred_at: datetime
    category: str
    summary: str

class ClaimRecord:
    claim_id: str
    policy_id: str
    status: str
    loss: LossFacts
    vehicles: list[VehicleRef]
    history: list[ClaimHistoryItem]
    source: SourceRef

class ClaimReadResponse:
    claim: ClaimRecord
    errors: list[AdapterError]

class ClaimsAdapter:
    def get_claim(self, request: ClaimReadRequest) -> ClaimReadResponse: ...
```

**Input:** an entitled claim ID and request context. **Output:** typed claim facts/history and provenance. A missing claim is a `NOT_FOUND` error; authorization failure is `FORBIDDEN` and must not disclose whether another claim exists.

## 2. Policy Administration

```python
class PolicyReadRequest:
    context: EntitledReadContext
    policy_id: str
    loss_date: date

class CoveragePeriod:
    kind: CoverageKind
    effective_from: date
    effective_to: date
    vehicle_ids: list[str]
    limits: dict[str, Decimal]  # limit name -> amount
    deductible: Decimal | None
    currency: str

class PolicyClause:
    clause_id: str
    text: str
    effective_from: date
    effective_to: date
    source: SourceRef

class PolicyRecord:
    policy_id: str
    status: str
    coverages: list[CoveragePeriod]
    clauses: list[PolicyClause]
    endorsements: list[str]  # endorsement IDs; text may be absent
    source: SourceRef

class PolicyReadResponse:
    policy: PolicyRecord
    errors: list[AdapterError]

class PolicyAdapter:
    def get_policy_for_loss(self, request: PolicyReadRequest) -> PolicyReadResponse: ...
```

**Input:** an entitled policy ID and loss date. **Output:** only policy terms effective on or relevant to that date, with source IDs. The deterministic application layer—not the model—compares coverage dates and performs monetary calculations. Missing referenced endorsement text must be represented as missing, never inferred.

## 3. Document Store

```python
class DocumentSearchRequest:
    context: EntitledReadContext
    claim_id: str
    kinds: list[DocumentKind] | None
    updated_after: datetime | None
    limit: int  # 1..100

class DocumentRecord:
    document_id: str
    claim_id: str
    kind: DocumentKind
    created_at: datetime
    filename_token: str
    media_type: str
    text: str | None
    extracted_text_status: Literal["available", "missing", "failed", "not_applicable"]
    source: SourceRef

class DocumentSearchResponse:
    documents: list[DocumentRecord]
    next_cursor: str | None
    errors: list[AdapterError]

class DocumentStoreAdapter:
    def search_documents(self, request: DocumentSearchRequest) -> DocumentSearchResponse: ...
```

**Input:** an entitled claim ID, optional type/date filters, and bounded page size. **Output:** only that claim's matching documents, extraction status, and provenance. Documents are untrusted evidence; embedded instructions are data, not instructions to the copilot. Enforce the claim filter in deterministic adapter code, not in a model prompt.

## Error and retry behavior

- Validate the request before any upstream call; authorize before retrieving data or invoking a model.
- Return a stable `correlation_id` for diagnostics. Do not place claim content or tokens in errors or logs.
- Retry only `UNAVAILABLE` / retryable `UPSTREAM_ERROR`, with bounded exponential backoff and the same request ID. Do not retry `FORBIDDEN`, `NOT_FOUND`, or invalid requests.
- Responses may be cached only within the authorized claim scope and with a version / freshness policy agreed with the client.

## Writes and idempotency

**There are no client-system write operations in this five-week POC.** All three adapters expose reads only; the copilot returns a draft in the POC UI, and an adjuster confirms manually. No recommendation or confirmation is written to any client system.

If a separately approved future phase adds a write, it must use an explicit command contract with an `idempotency_key` (UUID) and a canonical payload hash. Enforce uniqueness on `(tenant_id, operation, idempotency_key)`: a retry with the same key and same payload returns the original result; reuse of the key with a different payload returns a conflict and performs no write. Persist the key and result atomically with the write; callers must reuse the same key after timeouts. Define retention at least as long as the target record's retry / replay window. Never claim exactly-once delivery across systems; provide idempotent effects and reconciliation. Any client write requires a new decision, authorization model, audit design, and client approval.
