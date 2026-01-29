# TDD: [Project Name or Feature Title]

**Author:** [Your Name]
**Status:** [Draft / In-Review / Approved / Superseded]
**Date:** YYYY-MM-DD
**Reviewers:** @eng-lead, @security-team, @infrastructure

---

## 1. Executive Summary
*One-paragraph summary of the problem and the proposed technical solution. What are we building, and why does it matter right now?*

## 2. Context & Requirements
### 2.1 Problem Statement
*Describe the pain point. Link to the PRD (Product Requirement Document) or Jira tickets.*

### 2.2 Goals (Must-Haves)
* Goal 1: Clear, measurable outcome.
* Goal 2: Performance or functional requirement.

### 2.3 Non-Goals (Out of Scope)
* *Crucial: What are we intentionally NOT doing to avoid scope creep?*

## 3. Proposed Design
### 3.1 High-Level Architecture
*Provide a conceptual overview. How does this fit into our existing ecosystem?*


[Image of System Architecture Diagram]


### 3.2 Detailed Component Design
*Break down the specific services, modules, or classes being created or modified.*

### 3.3 Data Model & Schema Changes
* **New Tables/Fields:** [Describe schema changes]
* **Data Flow:** How does data move through the system?
* **Consistency Model:** (e.g., Eventual vs. Strong consistency)

### 3.4 API Design
*Define new endpoints or modifications to existing contracts.*
* **Endpoint:** `POST /v1/resource`
* **Request/Response Payload:** (Use JSON blocks)

## 4. Non-Functional Requirements
### 4.1 Scalability & Performance
* Expected throughput (RPS).
* Latency targets (p95/p99).
* Caching strategy.

### 4.2 Security & Privacy
* How is data encrypted (at rest/in transit)?
* Authentication and Authorization (RBAC/Scopes).
* Compliance impact (GDPR, SOC2).

### 4.3 Observability
* **Metrics:** What are we tracking? (e.g., error rates, latency).
* **Logging:** Any specific PII we must avoid logging?
* **Alerting:** What conditions should trigger an on-call page?

## 5. Implementation Plan
### 5.1 Migration & Rollout Strategy
* How do we deploy this without downtime?
* Feature flags to be used?
* Backfill plan for legacy data?

### 5.2 Test Plan
* **Unit/Integration:** Key scenarios to cover.
* **Load Testing:** How will we verify performance under stress?

### 5.3 Milestones
1.  **Phase 1:** [Core API & Schema] - Date
2.  **Phase 2:** [UI Integration & Beta] - Date

## 6. Alternatives Considered
*Why didn't we use [Alternative X]? This prevents "Why didn't you just..." questions during the review.*

## 7. Risks & Unknowns
*What are the "known unknowns"? What could go wrong, and how will we mitigate it?*