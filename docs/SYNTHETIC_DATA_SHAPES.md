# Synthetic data shapes

All records below are fictional test fixtures. Use synthetic IDs and tokenized values only; do not place production or personally identifiable information in this POC. Keep the data shapes aligned to the system contracts in `SYSTEM_CONTRACTS.md`.

## Shape and relationship rules

- A claim references one synthetic policy and one or more synthetic vehicles.
- Policy coverage periods and clauses are date-bounded; compare them to the claim's loss date in deterministic code.
- Documents reference exactly one claim and have stable synthetic document IDs for citations.
- Use ISO `YYYY-MM-DD` dates, UTC RFC 3339 timestamps, and decimal strings plus currency for money.
- A missing document or unavailable endorsement is represented explicitly; do not create a placeholder that looks like evidence.
- Synthetic identifiers use `SYN-` for records and `DOC-` / `POL-` for evidence. No real names, addresses, phone numbers, account numbers, or VINs.

## Example bundle

```json
{
  "claims": [
    {
      "claim_id": "SYN-CLM-001",
      "policy_id": "SYN-POL-001",
      "status": "open",
      "loss": {
        "loss_date": "2026-11-10",
        "loss_type": "collision",
        "reported_summary": "A synthetic vehicle was struck from behind while stopped. No injury or conflicting account is reported.",
        "jurisdiction_code": "SYN-VA"
      },
      "vehicles": [
        {
          "vehicle_id": "SYN-VEH-001",
          "vin_token": "SYN-VIN-A"
        }
      ],
      "history": [
        {
          "event_id": "SYN-EVT-001",
          "occurred_at": "2026-11-10T15:20:00Z",
          "category": "reported",
          "summary": "Synthetic first notice of loss received."
        }
      ],
      "source": {
        "system": "claims",
        "record_id": "SYN-CLM-001",
        "version": "fixture-v1",
        "retrieved_at": "2026-11-24T12:00:00Z"
      }
    }
  ],
  "policies": [
    {
      "policy_id": "SYN-POL-001",
      "status": "active",
      "coverages": [
        {
          "kind": "collision",
          "effective_from": "2026-01-01",
          "effective_to": "2026-12-31",
          "vehicle_ids": ["SYN-VEH-001"],
          "limits": {
            "property_damage": "25000.00"
          },
          "deductible": "500.00",
          "currency": "USD"
        }
      ],
      "clauses": [
        {
          "clause_id": "POL-001-COL",
          "text": "Fictional clause: covered accidental collision damage is eligible while collision coverage is active, subject to listed exclusions.",
          "effective_from": "2026-01-01",
          "effective_to": "2026-12-31",
          "source": {
            "system": "policy",
            "record_id": "POL-001-COL",
            "version": "fixture-v1",
            "retrieved_at": "2026-11-24T12:00:00Z"
          }
        }
      ],
      "endorsements": [],
      "source": {
        "system": "policy",
        "record_id": "SYN-POL-001",
        "version": "fixture-v1",
        "retrieved_at": "2026-11-24T12:00:00Z"
      }
    }
  ],
  "documents": [
    {
      "document_id": "DOC-001-FNOL",
      "claim_id": "SYN-CLM-001",
      "kind": "claim_report",
      "created_at": "2026-11-10T15:20:00Z",
      "filename_token": "synthetic-fnol-001",
      "media_type": "text/plain",
      "text": "Synthetic report: loss date 2026-11-10; vehicle was stopped at a signal and struck from behind.",
      "extracted_text_status": "available",
      "source": {
        "system": "documents",
        "record_id": "DOC-001-FNOL",
        "version": "fixture-v1",
        "retrieved_at": "2026-11-24T12:00:00Z"
      }
    },
    {
      "document_id": "DOC-001-PHOTO",
      "claim_id": "SYN-CLM-001",
      "kind": "photo",
      "created_at": "2026-11-10T15:30:00Z",
      "filename_token": "synthetic-photo-001",
      "media_type": "image/jpeg",
      "text": null,
      "extracted_text_status": "not_applicable",
      "source": {
        "system": "documents",
        "record_id": "DOC-001-PHOTO",
        "version": "fixture-v1",
        "retrieved_at": "2026-11-24T12:00:00Z"
      }
    }
  ]
}
```

## Dataset extensions

The initial evaluation set contains 20 scenario records and should be mapped into these shapes before adapter-level tests. Add cases for each expected outcome (approve, flag, route, deny), plus unauthorized access, incomplete or conflicting evidence, missing policy terms, embedded prompt instructions, and cross-claim retrieval attempts. Store expected labels and evidence IDs separately from model inputs so the model cannot see the answer key. Keep all fixture records deterministic and version the dataset whenever expected outcomes or source evidence change.
