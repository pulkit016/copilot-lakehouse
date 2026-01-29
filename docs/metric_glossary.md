# Metric Glossary

## 1. Purpose

This document defines the **authoritative business metrics** for the AI-Ready Lakehouse.

It serves as the **single source of truth** for:
- Analytics and dashboards
- Gold-layer aggregations
- AI / BI Copilot responses

Metrics must not be redefined outside this document.

---

## 2. Metric Design Principles

All metrics defined here follow these principles:

- Single definition per metric
- Explicit aggregation grain
- Deterministic calculations
- Business explainability
- AI-safe semantics

---

## 3. Core Funnel Metrics

### 3.1 Leads Created

**Metric Name:** leads_created

**Definition:**  
Number of unique human users who became aware of a product during a given time period.

**Funnel Stage:** LEAD_CREATED

**Calculation Logic:**  
Count of distinct lead_id where funnel_stage = LEAD_CREATED

**Grain:**  
Product × Date

**Allowed Dimensions:**  
- product_id  
- date  

**Excluded:**  
- Bot users  
- Non-funnel events  

**Business Meaning:**  
Measures top-of-funnel awareness.

---

### 3.2 Leads Engaged

**Metric Name:** leads_engaged

**Definition:**  
Number of unique human users who actively interacted with a product.

**Funnel Stage:** LEAD_ENGAGED

**Calculation Logic:**  
Count of distinct lead_id where funnel_stage = LEAD_ENGAGED

**Grain:**  
Product × Date

**Allowed Dimensions:**  
- product_id  
- date  

**Business Meaning:**  
Indicates user interaction depth.

---

### 3.3 Leads Qualified

**Metric Name:** leads_qualified

**Definition:**  
Number of unique human users who demonstrated high intent.

**Funnel Stage:** LEAD_QUALIFIED

**Calculation Logic:**  
Count of distinct lead_id where funnel_stage = LEAD_QUALIFIED

**Grain:**  
Product × Date

**Allowed Dimensions:**  
- product_id  
- date  

**Business Meaning:**  
Represents users likely to convert.

---

### 3.4 Leads Converted

**Metric Name:** leads_converted

**Definition:**  
Number of unique human users who committed meaningful value to a product.

**Funnel Stage:** LEAD_CONVERTED

**Calculation Logic:**  
Count of distinct lead_id where funnel_stage = LEAD_CONVERTED

**Grain:**  
Product × Date

**Allowed Dimensions:**  
- product_id  
- date  

**Business Meaning:**  
Represents successful conversion.

---

## 4. Derived Metrics

### 4.1 Conversion Rate

**Metric Name:** conversion_rate

**Definition:**  
Proportion of created leads that converted.

**Calculation Logic:**  
leads_converted divided by leads_created

**Grain:**  
Product × Date

**Constraints:**  
- If leads_created equals zero, conversion_rate is NULL  
- Calculated only from Gold-layer tables  

**Business Meaning:**  
Measures overall funnel efficiency.

---

## 5. Time Semantics

All metrics use the following time rules:

- event_time is used for business calculations
- ingestion_time is used only for monitoring and freshness

Late-arriving events are allowed and may cause historical recalculations.

---

## 6. Metric Stability and Versioning

Metrics are considered **stable contracts**.

Breaking changes include:
- Changes in definition
- Changes in calculation logic
- Changes in aggregation grain

Breaking changes require:
- Metric versioning
- Documentation updates
- Controlled backfills

---

## 7. AI and BI Usage Contract

AI and BI consumers:
- May only reference metrics defined in this document
- Must not invent new metrics
- Must explain metrics using these definitions

This prevents ambiguity and hallucination.

---

## 8. Ownership and Governance

- Metrics are owned by the data platform
- Changes require review and approval
- Documentation updates are mandatory

---

## 9. Downstream Impact

This metric glossary directly powers:
- Gold-layer aggregation tables
- BI dashboards
- AI-driven explanations

Any change to this document must be treated as a platform-level change.
