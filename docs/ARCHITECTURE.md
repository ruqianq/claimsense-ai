# ClaimSense AI Copilot Architecture

Runtime architecture for the five-week, synthetic-data proof of concept. Blue nodes are deterministic application code; the orange node is model judgment. The handoff between them is explicit: only authorized, prepared synthetic evidence and deterministic results cross into the model, and the model has no direct system access or authority to enforce access control.

```mermaid
flowchart TB
  subgraph PEOPLE[People]
    ADJ[Adjuster]
  end

  subgraph AWS[AWS non-production POC environment]
    subgraph APP[Python copilot service]
      UI[Adjuster UI and API]
      REQ[Request schema and identity token validation<br/>DETERMINISTIC]
      AUTH[Identity and claim entitlement gate<br/>DETERMINISTIC]
      DENY[Refuse request<br/>return no claim data]
      ORCH[Request orchestrator<br/>DETERMINISTIC]

      subgraph DET[DETERMINISTIC CODE BOUNDARY]
        CA[Typed Claims adapter<br/>read only]
        PA[Typed Policy adapter<br/>read only]
        DA[Typed Document adapter<br/>read only]
        CD[(Synthetic Claims data)]
        PD[(Synthetic Policy data<br/>and clause corpus)]
        DD[(Synthetic claim documents)]
        PREP[Evidence retrieval and preparation<br/>source IDs and loss-date selection]
        RULES[Deterministic checks<br/>coverage dates and monetary calculations]
        CTX[Prepared model context<br/>authorized evidence plus deterministic results]
        VALID[Model output and citation validator<br/>schema, evidence IDs, required uncertainty]
        COMPOSE[Response composer<br/>keeps deterministic results separate from model judgment]
        FALLBACK[Safe fallback<br/>route for human review]
      end

      subgraph MODEL[MODEL JUDGMENT BOUNDARY]
        AGENT[OpenAI Agents SDK workflow<br/>interpret clauses and assess evidence<br/>no client-system tools]
      end

      OUT[Draft response<br/>approve / flag-tag / route / deny<br/>rationale, citations, uncertainty, source type]
      REVIEW[Licensed adjuster review<br/>confirm, change, or escalate manually]
      AUDIT[(Audit and trace log)]
      METRICS[Operational metrics]
      DASH[Dashboard]
      ALARM[Alarm and recovery procedure]
    end

    subgraph DELIVERY[CI and release]
      CASES[20 synthetic evaluation cases<br/>four outcomes and adversarial cases]
      RUNNER[CI evaluation runner]
      GATE[Evaluation gate]
      IMAGE[Container image]
    end
  end

  ADJ -->|authenticated session, claim ID, request| UI
  UI -->|request payload and identity token| REQ
  REQ -->|validated principal and claim ID| AUTH
  AUTH -->|deny; audit refusal without claim payload| DENY
  DENY -->|generic refusal only| UI
  AUTH -->|allow plus authorized claim ID| ORCH

  ORCH -->|authorized claim read| CA
  ORCH -->|authorized policy read| PA
  ORCH -->|authorized document read| DA
  CA -->|read-only claim query| CD
  CD -->|synthetic claim facts and history| CA
  CA -->|typed claim facts| PREP
  PA -->|read-only policy query| PD
  PD -->|synthetic policy terms and clauses| PA
  PA -->|policy dates, clause text and source IDs| PREP
  DA -->|read-only document query| DD
  DD -->|synthetic document text and metadata| DA
  DA -->|typed document evidence and source IDs| PREP
  PREP -->|policy dates and monetary inputs for code checks| RULES
  PREP -->|authorized claim evidence and source IDs| CTX
  RULES -->|deterministic coverage-date and amount results| CTX
  CTX -->|MODEL BOUNDARY: authorized synthetic evidence, source IDs, deterministic results only| AGENT
  AGENT -->|typed draft judgment, rationale, cited IDs, uncertainty| VALID
  VALID -->|validated model judgment with citations and uncertainty| COMPOSE
  VALID -->|invalid or unsupported result; no forced judgment| FALLBACK
  FALLBACK -->|safe route-for-review judgment with reason| COMPOSE
  RULES -->|deterministic results retained as deterministic| COMPOSE
  COMPOSE -->|typed draft with source labels and evidence citations| OUT
  OUT -->|draft only; no final adjudication| UI
  UI -->|display draft for licensed review| REVIEW

  AUTH -->|request ID, principal, allow or deny, timestamp| AUDIT
  VALID -->|run ID, output source, outcome, citation validation status| AUDIT
  ORCH -->|latency, errors, run status| METRICS
  METRICS -->|service health and latency| DASH
  METRICS -->|threshold breach| ALARM

  CASES -->|synthetic test inputs and expected labels| RUNNER
  RUNNER -->|case results and safety checks| GATE
  GATE -->|pass only| IMAGE
  IMAGE -->|approved container deployment| ORCH

  classDef deterministic fill:#E7F1FF,stroke:#1F5E8C,color:#0E3550,stroke-width:1.5px;
  classDef judgment fill:#FFF0D8,stroke:#B86B00,color:#613800,stroke-width:2px;
  classDef data fill:#F3F5F7,stroke:#697681,color:#26323A;
  classDef ops fill:#E8F5E9,stroke:#4C7B53,color:#214329;
  class REQ,AUTH,ORCH,CA,PA,DA,PREP,RULES,CTX,VALID,COMPOSE,FALLBACK,UI,DENY,OUT deterministic;
  class AGENT judgment;
  class CD,PD,DD,AUDIT,CASES data;
  class METRICS,DASH,ALARM,RUNNER,GATE,IMAGE ops;
```

## Boundary and flow notes

- **Deterministic code owns** identity and claim authorization, policy-date checks, monetary calculations, data retrieval, request/output schema checks, and evidence-ID validation. Authorization occurs before any agent invocation.
- **Model judgment owns** interpretation of policy language and assessment of claim evidence, including inconsistencies. It receives a prepared evidence bundle and returns a draft with citations and uncertainty. It cannot read the synthetic stores directly, call client systems, authorize users, or finalize outcomes.
- **Uncertainty path:** missing, conflicting, or insufficient evidence produces a flagged/route-for-review draft that explains the gap. A licensed adjuster reviews and confirms every proposed outcome.
- **Data and write boundary:** only fictional synthetic data is used; adapters are read-only; no recommendation, confirmation, or claim change is written to client systems. Adjuster confirmation is manual in this POC.
- **Evaluation and operations:** the 20-case CI suite covers approve, flag/tag, route, and deny plus adversarial cases. Runtime telemetry feeds the dashboard and alarm; the runbook defines recovery.

## Discovery references

- [Architecture decisions](https://github.com/ruqianq/claimsense-ai/blob/main/docs/DECISIONS.md)
- [Engagement charter](https://github.com/ruqianq/claimsense-ai/blob/main/docs/Engagement%20Charter.md)
- [20-case evaluation suite](https://github.com/ruqianq/claimsense-ai/blob/main/docs/evaluation_cases.json)
