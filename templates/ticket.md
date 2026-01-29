## How I’ll approach creating this ticket template

1. **Start from the job-to-be-done:** a ticket is a *reliable translation* of a PRD/TDD requirement into a **vertical slice** (or a small, fundamental unit of work) that an engineer can implement without guessing.
2. **Design for repeatability:** fixed section order, consistent wording, and checklists so tickets read the same across teams.
3. **Bake in traceability:** every ticket should point back to *exact* PRD/TDD sections and define success in measurable terms.
4. **Make “done” unambiguous:** acceptance criteria + test plan + rollout/observability + definition-of-done checklist.
5. **Include non-functional requirements by default:** performance, reliability, security/privacy, accessibility, cost, operability.
6. **Optimize for humans *and* agents:** structured fields, predictable headings, and explicit decisions/assumptions.

---

## Framework for evaluating a “good ticket”

| Category                                   | What “good” looks like                                                                               |
| ------------------------------------------ | ---------------------------------------------------------------------------------------------------- |
| 1) Clarity & Plain Language                | Understandable on first read; minimal jargon; definitions included.                                  |
| 2) Traceability to PRD/TDD                 | Links + specific section IDs; requirement mapping is explicit.                                       |
| 3) Right-Sized Scope                       | A vertical slice or small foundational unit; clear in-scope/out-of-scope; limited blast radius.      |
| 4) Completeness of Functional Requirements | Must/should/could captured; edge cases called out; inputs/outputs defined.                           |
| 5) Non-Functional Requirements Coverage    | Performance, reliability, security/privacy, accessibility, cost, compliance included where relevant. |
| 6) Testability & Acceptance Criteria       | AC is objective; includes test plan; “done” is measurable.                                           |
| 7) Operability (Observability + Runbooks)  | Metrics/logs/traces/alerts; rollout plan; operational notes and ownership.                           |
| 8) Dependency & Risk Management            | Dependencies listed; risks & mitigations; rollback strategy.                                         |
| 9) Collaboration & Reviewability           | Clear owners/reviewers; open questions; design decisions summarized.                                 |
| 10) Automation/Agent Friendliness          | Consistent structure; machine-parsable metadata; checklists and explicit signals.                    |

Scoring: **1–10** per category. Target is **8+** everywhere.

---

## Ticket template (v1)

*(Initial draft; I’ll evaluate it next and revise if anything lands under 8.)*

### Template (v1)

* Title
* Context + goal
* References (PRD/TDD)
* Scope (in/out)
* Requirements (functional + NFR)
* Acceptance criteria
* Test plan
* Observability + rollout
* Dependencies/risks
* Definition of done checklist

---

## Evaluation of v1 (1–10)

| Category                                | Score | Notes                                                       |
| --------------------------------------- | ----: | ----------------------------------------------------------- |
| Clarity & Plain Language                |     8 | Sections are straightforward.                               |
| Traceability to PRD/TDD                 |     8 | References exist, but not structured enough.                |
| Right-Sized Scope                       |     8 | Scope section exists, but lacks prompts to enforce sizing.  |
| Completeness of Functional Requirements |     8 | Present, but not detailed prompts.                          |
| Non-Functional Requirements Coverage    |     8 | Mentioned, but not enumerated.                              |
| Testability & Acceptance Criteria       |     8 | Included, but needs stronger structure (G/W/T).             |
| Operability                             |     8 | Included, but missing concrete prompts (dashboards/alerts). |
| Dependency & Risk Management            |     8 | Included, but could be more explicit.                       |
| Collaboration & Reviewability           |     8 | Implicit, but no explicit owner/reviewer/questions section. |
| Automation/Agent Friendliness           | **7** | Lacks consistent, machine-parsable metadata and checklists. |

At least one category is <8, so I’m revising.

---

## Revised ticket template (v2)

### Evaluation of v2 (1–10)

| Category                                | Score | Notes                                              |
| --------------------------------------- | ----: | -------------------------------------------------- |
| Clarity & Plain Language                |     9 | Clear prompts + definitions.                       |
| Traceability to PRD/TDD                 |     9 | Explicit requirement mapping + section anchors.    |
| Right-Sized Scope                       |     9 | Prompts enforce vertical slice + out-of-scope.     |
| Completeness of Functional Requirements |     9 | Inputs/outputs, edge cases, contracts included.    |
| Non-Functional Requirements Coverage    |     9 | Enumerated NFR checklist.                          |
| Testability & Acceptance Criteria       |     9 | G/W/T + test plan structure.                       |
| Operability                             |     9 | Observability + rollout + rollback prompts.        |
| Dependency & Risk Management            |     9 | Concrete dependency + risk + mitigation fields.    |
| Collaboration & Reviewability           |     9 | Owners/reviewers + questions + decision log.       |
| Automation/Agent Friendliness           |     9 | Metadata block + consistent headings + checklists. |

All categories are **8+**, so we’re done.

---

## Process summary (bulleted)

* Identified what a ticket must accomplish: translate PRD/TDD requirements into implementable, right-sized work.
* Defined an evaluation rubric emphasizing clarity, traceability, scope control, NFRs, testability, and operability.
* Drafted an initial template and scored it against the rubric.
* Revised the template to improve consistency and agent friendliness using structured metadata, explicit prompts, and checklists.
* Re-scored to confirm every category meets **8+**.

---

## Final ticket template (Markdown)

```markdown
# <Ticket Title>
> **One sentence outcome:** “When this is done, <user/system> can <capability> so that <benefit>.”

---

## Ticket Metadata (machine-friendly)
- **Type:** Feature | Bug | Tech Debt | Spike
- **Priority:** P0 | P1 | P2 | P3
- **Owner:** @
- **Reviewers:** @ (Eng), @ (Product), @ (Design), @ (Security/Privacy) *(as needed)*
- **Target Release/Milestone:** 
- **Component(s):** 
- **Dependencies:** #issue, link, team
- **Feature Flag:** Yes/No — name: ``
- **Risk Level:** Low | Medium | High
- **Customer Impact:** None | Internal | Limited | Broad
- **Docs Needed:** Yes/No — where:

---

## Context
### Problem / Background
- What is happening today? What’s the pain or limitation?
- Who is impacted (user, internal team, system)?

### Goal
- What are we trying to achieve? (Be specific and outcome-focused.)

### Non-Goals (explicitly out of scope)
- Bullet list of things people might assume are included, but aren’t.

---

## Source of Truth (Traceability)
### PRD
- Link:
- Relevant PRD section(s): `PRD §<id/name>` (paste headings or anchor IDs)

### TDD
- Link:
- Relevant TDD section(s): `TDD §<id/name>` (paste headings or anchor IDs)

### Requirement Mapping
List the specific requirement(s) this ticket implements:
- `REQ-###`: <requirement statement> — Source: `PRD §... / TDD §...`
- `REQ-###`: <requirement statement> — Source: `PRD §... / TDD §...`

---

## Scope & Sizing (Vertical Slice)
### In Scope (this ticket delivers)
Describe the *smallest complete slice* that creates user/system value:
- 

### Out of Scope (future work / separate tickets)
- 

### Interfaces / Boundaries
- Systems touched:
- Components not touched:
- Expected blast radius:

> **Sizing check:** Can this be implemented, tested, and safely rolled out without a multi-week effort or broad refactor?
> - If not, split into smaller tickets by vertical slices (UI + API + data + rollout per slice).

---

## Functional Requirements
Use MUST/SHOULD/COULD to reduce ambiguity.

### MUST (required)
- 

### SHOULD (important, not required for initial merge/release)
- 

### COULD (nice-to-have)
- 

### Inputs / Outputs (Contracts)
- Inputs (events, API requests, UI actions):
- Outputs (API responses, side effects, UI states):
- API/Schema changes (if any):
  - Endpoint(s):
  - Request/response shape:
  - DB tables/fields:
  - Migration/backfill required: Yes/No

### Edge Cases & Error Handling
- Known edge cases:
- Failure modes (timeouts, partial failure, retries):
- Expected error messages / user messaging:

---

## Non-Functional Requirements (NFRs)
Check all that apply and fill details.

- [ ] **Performance:** latency/throughput targets, budgets, expected load
- [ ] **Reliability:** retries, idempotency, timeouts, circuit breakers, SLIs/SLOs
- [ ] **Security:** authz/authn, threat considerations, secrets, least privilege
- [ ] **Privacy/Data Handling:** PII, retention, encryption, access controls
- [ ] **Compliance:** SOC2, GDPR, HIPAA, etc. (if applicable)
- [ ] **Accessibility:** keyboard nav, ARIA, contrast, screen reader behavior
- [ ] **Internationalization:** locale/timezone, formatting, translations
- [ ] **Cost:** compute/storage/network impact, limits/quotas
- [ ] **Compatibility:** backward compatibility, versioning, graceful degradation

NFR notes:
- 

---

## UX / Product Notes (if applicable)
- UX behavior summary:
- Copy text (exact wording):
- Analytics events to emit:
- Screenshots / mocks / prototype links:

---

## Acceptance Criteria (objective)
Write as **Given / When / Then** and include measurable outcomes.

1. **Given** … **When** … **Then** …
2. **Given** … **When** … **Then** …

### Definition of “Done” (user-visible / system-visible)
- What will we observe that proves success?
- What is the expected behavior in steady state?

---

## Test Plan
### Automated Tests
- [ ] Unit tests:
- [ ] Integration tests:
- [ ] End-to-end tests:

### Manual / Verification Steps
- Steps to verify in dev/stage:
- Test data needed:
- Negative test cases:

### Regression Risks
- What might break?
- How will we know quickly?

---

## Observability & Operations
### Telemetry
- Metrics (name, type, dimensions/tags):
- Logs (key fields, correlation IDs):
- Traces (spans, propagation):

### Alerting / Dashboards
- Alerts to add/update:
- Dashboard(s) to add/update:
- Owner/on-call routing:

### Runbook Updates
- Link to runbook or add instructions here:
- Common failure scenarios + mitigations:

---

## Rollout Plan
- Rollout strategy: Dark launch | Feature flag | Canary | Phased rollout
- Environments: dev → stage → prod
- Backward compatibility plan:
- Migration/backfill steps (if any):
- Rollback plan (exact steps):
- Success metrics during rollout:
- Kill-switch / disable flag instructions:

---

## Dependencies, Risks, Decisions
### Dependencies / Blockers
- 

### Risks & Mitigations
- Risk:
  - Mitigation:
  - Monitoring:

### Decision Log (why we’re doing it this way)
- Decision:
  - Alternatives considered:
  - Rationale:

---

## Open Questions
- [ ] Question:
- [ ] Question:

---

## Implementation Notes (optional, non-binding)
- Suggested approach:
- Files/areas likely to change:
- Pseudocode/sequence (if helpful):

---

## Checklist (before closing)
- [ ] Acceptance criteria met and verified
- [ ] Tests added/updated and passing
- [ ] Observability in place (metrics/logs/alerts as applicable)
- [ ] Rollout plan executed or documented for release owner
- [ ] Docs/runbook updated (if needed)
- [ ] Security/privacy review completed (if applicable)
- [ ] No untracked follow-up work (or follow-ups ticketed and linked)
```
