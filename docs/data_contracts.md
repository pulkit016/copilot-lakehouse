# Data Contracts

## 1. Purpose of Data Contracts

This document defines **explicit data contracts** for each layer of the AI-Ready Lakehouse.

A data contract specifies:
- What data is guaranteed to exist
- What transformations are allowed
- What downstream consumers can rely on
- What constitutes a breaking change

These contracts ensure:
- Stability across Bronze, Silver, and Gold layers
- Predictable analytics and AI behavior
- Safe evolution of schemas and business logic

Once established, these contracts **must not be violated silently**.

---

## 2. Source Data Contract (GitHub Events API)

### 2.1 Source Description
- Source: GitHub Public Events API
- Data Type: Near-real-time, event-driven JSON
- Update Pattern: Continuous polling
- Schema Stability: **Not guaranteed**

### 2.2 Source Guarantees
The GitHub API guarantees:
- Each event has a globally unique `id`
- Each event has a `type`
- Each event has a `created_at` timestamp
- Event payload structure varies by event type

### 2.3 Source Limitations
- Schema may change without notice
- New event types may appear
- Bot activity is included
- Events may arrive late or out of order

### 2.4 Contractual Handling
- No assumptions are made about payload structure
- Raw payloads are preserved exactly
- Source data is never mutated or filtered at ingestion

---

## 3. Bronze Layer Contract (Raw & Immutable)

### 3.1 Purpose
The Bronze layer represents the **immutable system of record**.

It stores raw GitHub events exactly as received, enriched only with ingestion metadata.

### 3.2 Guarantees
The Bronze layer guarantees:
- Append-only writes
- No updates or deletes
- Preservation of full raw JSON payload
- Schema evolution without failure
- Event uniqueness preserved via `event_id`

### 3.3 Required Fields
Each Bronze record must include:

| Field | Description |
|-----|------------|
| event_id | Unique GitHub event ID |
| event_type | GitHub event type |
| raw_payload | Full raw JSON payload |
| event_time | Source `created_at` timestamp |
| ingestion_time | Time of ingestion |
| source | Data source identifier |
| batch_id | Ingestion batch identifier |

### 3.4 Prohibited Actions
- No deduplication
- No filtering
- No business logic
- No renaming of source fields

### 3.5 Breaking Changes
Any change that:
- Alters raw payload
- Drops events
- Mutates historical data

is considered **breaking** and forbidden.

---

## 4. Silver Layer Contract (Trusted & Interpreted)

### 4.1 Purpose
The Silver layer represents **trusted, business-interpretable data**.

It applies deterministic transformations to Bronze data while preserving traceability.

### 4.2 Guarantees
The Silver layer guarantees:
- Deduplicated events
- Deterministic funnel stage assignment
- Bot filtering applied
- Normalized schemas
- Referential integrity

### 4.3 Required Fields (Event-Level)
Each Silver event record must include:

| Field | Description |
|-----|------------|
| event_id | Deduplicated unique event |
| lead_id | Human user identifier |
| product_id | Repository name |
| raw_event_type | Original GitHub event type |
| funnel_stage | Derived funnel stage |
| event_time | Business event time |
| ingestion_time | Ingestion timestamp |
| is_bot | Bot classification flag |

### 4.4 Business Rules
- Funnel stage derivation follows `03_business_funnel_definition.md`
- Bot events (`is_bot = true`) are excluded from funnel metrics
- Deduplication is based on `event_id`

### 4.5 SCD Contracts
- Dimensions may use SCD Type 1 or Type 2
- Historical correctness must be preserved
- Changes to SCD logic require reprocessing

### 4.6 Breaking Changes
- Changing funnel mapping logic
- Changing bot identification rules
- Altering deduplication logic

All require:
- Versioned update
- ADR
- Controlled backfill

---

## 5. Gold Layer Contract (Business & AI-Ready)

### 5.1 Purpose
The Gold layer represents **authoritative business truth** and is the **only layer exposed to analytics and AI consumers**.

### 5.2 Guarantees
The Gold layer guarantees:
- Metric correctness
- Stable schemas
- Defined data grain
- AI-safe semantics
- Point-in-time correctness

### 5.3 Metric Contracts
All metrics must:
- Be defined in `05_metric_glossary.md`
- Have a single authoritative calculation
- Use Silver data as input
- Be reproducible via time travel

### 5.4 Required Tables
Examples:
- Daily funnel metrics
- Conversion rates
- Feature-ready aggregates
- Snapshot tables for ML

### 5.5 Prohibited Actions
- Redefining metrics in downstream tools
- Applying business logic outside Gold
- Exposing Silver data directly to AI

### 5.6 Breaking Changes
- Metric definition changes
- Grain changes
- Dimension cardinality changes

All breaking changes require:
- New metric version
- Backward compatibility where possible
- Explicit communication

---

## 6. Schema Evolution Policy

### 6.1 Allowed Changes
- Adding nullable columns
- Adding new event types
- Adding new dimensions

### 6.2 Disallowed Changes
- Removing required columns
- Changing column meanings
- Changing data grain

### 6.3 Versioning Strategy
- Schema versions tracked in metadata tables
- Changes documented via ADRs
- Reprocessing strategy defined before rollout

---

## 7. Time Semantics Contract

Two timestamps are mandatory and non-negotiable:

| Field | Meaning |
|-----|--------|
| event_time | When the event occurred |
| ingestion_time | When the event was ingested |

Rules:
- Business logic uses `event_time`
- Observability uses `ingestion_time`
- Late events are allowed and expected

This enables:
- Correct historical analysis
- AI reproducibility
- Time travel queries

---

## 8. Enforcement & Validation

Data contracts are enforced through:
- Schema validation
- Data quality checks
- Pipeline assertions
- Observability metrics

Violations must:
- Fail fast
- Be logged
- Trigger alerts

---

## 9. Downstream Impact

These data contracts ensure:
- Predictable analytics
- Trustworthy AI inputs
- Safe schema evolution
- Clear ownership boundaries

Any change to this document requires:
- Version control
- Architectural Decision Record (ADR)
- Validation of downstream impact
