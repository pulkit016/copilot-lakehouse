# Business Funnel Definition

## 1. Business Context

This project models a **sales and lead funnel** using real-time event data from the GitHub Public Events API.

Although GitHub is a developer platform, the underlying user behavior closely mirrors how users interact with products in real businesses:

- Repositories represent **products**
- Users represent **leads**
- GitHub events represent **user interactions**
- Increasing engagement represents **progress through a funnel**

In modern data platforms, sales funnels are rarely available as a direct data source.  
Instead, they are **derived from behavioral signals** such as clicks, signups, usage, and contributions.

This project intentionally follows that industry pattern by deriving a deterministic funnel from event-driven data.

---

## 2. Funnel Stages (Canonical Definitions)

The platform defines **four canonical funnel stages**.  
These stages are authoritative and must be used consistently across the lakehouse.

### 2.1 LEAD_CREATED
**Definition:**  
A user has become aware of a product and shown initial interest.

Typical real-world equivalents:
- Website visit
- Newsletter signup
- Ad click

---

### 2.2 LEAD_ENGAGED
**Definition:**  
A user is actively interacting with the product beyond passive interest.

Typical real-world equivalents:
- Product exploration
- Demo interaction
- Support conversation

---

### 2.3 LEAD_QUALIFIED
**Definition:**  
A user has demonstrated strong intent and is actively evaluating the product.

Typical real-world equivalents:
- Trial usage
- Proof-of-concept
- Sales-qualified lead (SQL)

---

### 2.4 LEAD_CONVERTED
**Definition:**  
A user has committed meaningful value to the product.

Typical real-world equivalents:
- Purchase
- Subscription activation
- Contract signing

---

## 3. Event to Funnel Mapping

The table below defines the **official and deterministic mapping** between GitHub event types and funnel stages.

| GitHub Event Type | Funnel Stage | Business Rationale |
|------------------|-------------|--------------------|
| WatchEvent | LEAD_CREATED | User expresses interest by starring a repository |
| ForkEvent | LEAD_CREATED | User copies the product for exploration |
| IssuesEvent (opened) | LEAD_ENGAGED | User initiates interaction |
| IssueCommentEvent | LEAD_ENGAGED | User actively participates |
| CreateEvent | LEAD_ENGAGED | User explores by creating branches or tags |
| PullRequestEvent (opened) | LEAD_QUALIFIED | User shows intent to contribute |
| PullRequestReviewEvent | LEAD_QUALIFIED | User performs deep evaluation |
| PushEvent | LEAD_CONVERTED | User actively contributes value |
| PullRequestEvent (merged) | LEAD_CONVERTED | Contribution is accepted into the product |

Business meaning is applied **only in the Silver layer**.  
Bronze data remains raw and uninterpreted.

---

## 4. Excluded Events

Certain GitHub events do not represent lead intent and are **explicitly excluded** from funnel calculations.

### 4.1 Excluded Event Types
- DeleteEvent
- ReleaseEvent
- Organizational maintenance events
- System-level operational events

Excluded events:
- Are ingested and stored in Bronze
- Do not contribute to funnel metrics
- Are retained for auditing and lineage

---

## 5. Human vs Bot Policy

GitHub event streams contain significant **automated bot activity**, which must not influence business metrics.

### 5.1 Bot Identification Rule

A user is classified as a bot if:



```text
actor.login contains “[bot]Z
```
### 5.2 Policy Rules
- Bot-generated events are excluded from all funnel stages
- Bot events remain stored in Bronze for traceability
- Only human-generated events contribute to Silver and Gold metrics

This policy ensures funnel metrics reflect **real human intent**.

---

## 6. Design Principles

This funnel definition follows these principles:

- **Deterministic** – Same input always yields the same funnel stage
- **Explainable** – Every mapping has a clear business rationale
- **Stable** – Funnel stages do not change casually
- **AI-safe** – Clear semantics reduce hallucination risk in AI systems

---

## 7. Downstream Usage

This document serves as the **semantic backbone** for:

- Silver-layer transformations
- Gold-layer KPIs and metrics
- Data quality validations
- AI readiness and BI Copilot behavior

Any change to this document requires:
- A versioned update
- An Architectural Decision Record (ADR)
- Reprocessing of downstream data where applicable



