# Create GET /v1/meetings/:id/summary API

> **One sentence outcome:** When this is done, an authorized user can fetch the current AI summary state (including TL;DR, decisions, action items, and status) for a meeting so the Workspace UI Sidebar can reliably display progress and results.

---

## Ticket Metadata (machine-friendly)
- **Type:** Feature
- **Priority:** P1
- **Owner:** @backend-sara
- **Reviewers:** @tech-lead-dave (Eng), @security-review (Security/Privacy), @infra-team (Infra)
- **Target Release/Milestone:** Project Echo - Dark Launch
- **Component(s):** API Gateway, Meetings Service, DB (meeting_summaries), Redis cache
- **Dependencies:** TDD §3.3 schema migration (meeting_summaries), RBAC permission check implementation, Redis cluster availability
- **Feature Flag:** Yes — `echo_summaries_api`
- **Risk Level:** Medium
- **Customer Impact:** Internal (Dark Launch), then Limited (Beta workspace)
- **Docs Needed:** Yes — API reference + runbook snippet for cache behavior

---

## Context
### Problem / Background
Project Echo generates meeting summaries asynchronously. The UI and other consumers need a single API to retrieve the **current state** of summarization (queued/transcribing/summarizing/completed/failed) plus the latest summary payload (TL;DR + structured data). Without a consistent endpoint, consumers will reimplement DB access patterns, risk missing RBAC enforcement, and cause inconsistent behavior across clients.

### Goal
Implement `GET /v1/meetings/:id/summary` as defined in the TDD so clients can fetch summary state and content with correct authorization, caching, and observability.

### Non-Goals
- Editing summaries (handled by `PATCH /v1/meetings/:id/summary`)
- Triggering transcription/summarization jobs
- Search endpoint implementation (`GET /v1/workspaces/:id/summaries/search`)

---

## Source of Truth (Traceability)
### PRD
- Link: [Project Echo - Smart Summaries](link-to-prd)
- Relevant PRD section(s): Goals → Standardized Output; Searchability; User Agency

### TDD
- Link: (this document)
- Relevant TDD section(s): `TDD §3.4 API Design`, `TDD §3.3 Data Model`, `TDD §4 Non-Functional Requirements (Caching, RBAC, Observability)`

### Requirement Mapping
- `REQ-API-3.4-GET`: Provide `GET /v1/meetings/:id/summary` to fetch current summary state and data — Source: `TDD §3.4`
- `REQ-RBAC-4.2`: Restrict access to users with `view` permissions on original meeting recording — Source: `TDD §4.2`
- `REQ-CACHE-4.1`: Cache results in Redis for 24 hours post-generation — Source: `TDD §4.1`
- `REQ-OBS-4.3`: Standardized logs for `job_id` tracing — Source: `TDD §4.3`

---

## Scope & Sizing (Vertical Slice)
### In Scope (this ticket delivers)
A fully functional API endpoint that:
- Enforces RBAC
- Reads latest summary state and payload from `meeting_summaries`
- Applies caching behavior (with rules below)
- Emits required logs/metrics
- Returns a stable, versioned response contract for the UI Sidebar

### Out of Scope (future work / separate tickets)
- DB migration creation (assumed dependency)
- Background worker job pipeline implementation
- PATCH endpoint for edits
- Search endpoint

### Interfaces / Boundaries
- Systems touched: API Gateway, Meetings Service, Postgres (`meeting_summaries`), Redis
- Components not touched: LLM Summarizer, Transcription Adapter, UI Sidebar
- Expected blast radius: Read path only; RBAC correctness is the primary risk

---

## Functional Requirements

### MUST
- Implement `GET /v1/meetings/:id/summary`.
- Enforce RBAC: requester must have `view` permission on the meeting recording.
- Return summary **status** and **data** per the table fields in `meeting_summaries`.
- Handle “not found” and “not ready yet” states cleanly and consistently.
- Include request correlation fields and `job_id` in logs when available.

### SHOULD
- Use Redis caching for 24 hours **after generation** (i.e., once status is `completed`).
- Avoid caching non-completed statuses longer than a short TTL (to keep UI progress fresh).
- Provide an ETag (or `updated_at`-based conditional response) to reduce client bandwidth (if supported in current API framework).

### COULD
- Include `Cache-Control` headers aligned to status (e.g., shorter for in-progress, longer for completed).

### Inputs / Outputs (Contracts)
**Input**
- Path param: `id` (UUID) — meeting ID
- Auth context: user identity + workspace context (as per existing auth middleware)

**Output**
- `200 OK` with payload (see “Response Contract”)
- `401/403` when unauthorized
- `404` when meeting does not exist OR meeting exists but summary row not created (see error semantics)
- `500` for unexpected errors

#### Response Contract (proposed)
```json
{
  "meeting_id": "uuid",
  "summary": {
    "id": "uuid",
    "status": "queued|transcribing|summarizing|completed|failed",
    "tldr": "string|null",
    "structured_data": {
      "decisions": ["string", "..."],
      "action_items": [
        { "owner": "string", "task": "string" }
      ]
    },
    "is_edited": true,
    "transcript_url": "string|null",
    "last_updated_at": "RFC3339 timestamp"
  }
}
```

Note: transcript_url must only be included if policy allows the client to access it; it should be a time-bound signed URL generated server-side from transcript_s3_url and must respect data residency.

### Edge Cases & Error Handling
- **Summary row missing**
  - If meeting exists but no `meeting_summaries` row yet: return `404` with an error code indicating “summary_not_initialized” **or** return `200` with `status="queued"` and null fields.
  - **Decision for this ticket:** return `200` with `status="queued"` if meeting exists, to simplify UI polling and avoid ambiguous 404s.
- **Status `failed`**
  - Return `status="failed"` and omit `tldr/structured_data` if absent.
  - Include a safe, non-sensitive `error_code` if available *(optional, not required)*.
- **Data shape differences**
  - If `structured_data` is NULL in DB, return `{ "decisions": [], "action_items": [] }`.
- **Permission changes**
  - If user loses `view` permission, requests must immediately return `403` even if cached.

---

## Non-Functional Requirements (NFRs)
- [x] **Performance:** Endpoint p95 < 200ms in-region for cache hits; DB query should be indexed by `meeting_id`.
- [x] **Reliability:** Cache must not bypass RBAC; safe fallback to DB on cache errors.
- [x] **Security:** Enforce authz; ensure signed transcript URL TTL is short (e.g., 10–30 minutes) and never log raw URLs.
- [x] **Privacy/Data Handling:** Respect data residency when generating transcript URLs; do not return transcript URL if not allowed.
- [ ] **Compliance:** N/A for this ticket unless org policy requires.
- [ ] **Accessibility:** N/A (API only).
- [ ] **Internationalization:** N/A (API only).
- [x] **Cost:** Cache completed responses to reduce DB load on high view rates.
- [x] **Compatibility:** Response must be backward compatible once shipped; introduce new fields additively.

**NFR notes**
- Redis cache key should be scoped by `meeting_id` and **workspace/tenant** to avoid cross-tenant leakage.
- RBAC must be evaluated **before** serving cached content.

---

## UX / Product Notes (if applicable)
- UI Sidebar will poll this endpoint while status is in progress.
- Status strings must match exactly: `queued`, `transcribing`, `summarizing`, `completed`, `failed`.

---

## Acceptance Criteria (objective)
1. **Given** an authenticated user with `view` permission on meeting `M`, **when** they call `GET /v1/meetings/M/summary`, **then** they receive `200` with `meeting_id=M` and a `summary.status` value in the allowed enum.
2. **Given** a meeting `M` whose summary is `completed` with `tldr` length ≤ 300 and valid `structured_data`, **when** the endpoint is called, **then** the response includes that TL;DR and structured data and `is_edited` matches DB.
3. **Given** a meeting `M` that exists but has no summary content yet, **when** the endpoint is called, **then** it returns `200` with `status="queued"` and `tldr=null`, `structured_data.decisions=[]`, `structured_data.action_items=[]`.
4. **Given** a user without `view` permission on meeting `M`, **when** they call the endpoint, **then** they receive `403` and no summary content is returned.
5. **Given** status is `completed`, **when** the endpoint is called twice within 24 hours, **then** the second request is served from Redis (validated via metric/log) and still enforces RBAC.
6. **Given** Redis is unavailable, **when** the endpoint is called, **then** it falls back to DB and returns correct data without increased error rate beyond acceptable thresholds.
7. **Given** requests for the endpoint, **when** logs are emitted, **then** logs include `meeting_id`, `workspace_id`, and `job_id` (if present) and do not include PII or raw transcript URLs.

---

## Test Plan

### Automated Tests
- [ ] **Unit tests**
  - RBAC check invoked for every request (including cache hit path)
  - Response mapping: NULL `structured_data` → empty arrays
  - TL;DR max length enforcement: ensure API returns stored value and does not expand beyond 300 chars
- [ ] **Integration tests**
  - Seed meeting + meeting_summaries row; verify status/data response
  - “No summary row” case returns `queued` response
  - Permission denied returns 403
- [ ] **End-to-end tests**
  - API gateway → service → DB/Redis wiring validated in staging

### Manual / Verification Steps
- In staging, create a meeting with a seeded `meeting_summaries` record:
  - Verify 200 response and correct JSON shape
  - Flip permissions for test user; verify immediate 403 even if previously cached
- Simulate Redis down (or disable cache) and confirm DB fallback

### Regression Risks
- Accidentally exposing cross-tenant data via cache key misuse
- Returning transcript URLs in logs or to unauthorized users

---

## Observability & Operations

### Telemetry
- **Metrics**
  - `echo.summary_get.requests` (count) tagged by `status`, `cache_hit=true|false`, `http_status`
  - `echo.summary_get.latency_ms` (histogram)
  - `echo.summary_get.rbac_denied` (count)
- **Logs (structured)**
  - Include `request_id`, `meeting_id`, `workspace_id`, `user_id` (or anonymized subject), `job_id` (if available), `cache_hit`
- **Traces**
  - Span for RBAC check, Redis get/set, DB query

### Alerting / Dashboards
- Add dashboard panel for:
  - Error rate by status code
  - Cache hit ratio for completed summaries
  - Latency p95/p99
- Alerts:
  - Spike in `403` (possible permissions regression)
  - Spike in `500` or cache failures causing elevated latency

### Runbook Updates
- Add “Summary GET endpoint” section:
  - Cache key format, TTL rules
  - How to invalidate cache (if needed)
  - What to check when users report missing summaries

---

## Rollout Plan
- **Rollout strategy:** Feature flag `echo_summaries_api` enabled for internal workspaces only (Dark Launch).
- **Environments:** dev → stage → prod
- **Backward compatibility plan:** new endpoint; no client breakage
- **Rollback plan**
  - Disable feature flag
  - *(Optional)* revert routing at gateway if applicable
- **Success metrics during rollout**
  - p95 latency < 200ms (cache hit), < 400ms (cache miss)
  - 0 cross-tenant incidents
- **Kill-switch**
  - Turn off `echo_summaries_api`

---

## Dependencies, Risks, Decisions

### Dependencies / Blockers
- `meeting_summaries` table migration deployed (TDD §3.3)
- RBAC helper exists for “meeting recording view permission”
- Redis available in target regions

### Risks & Mitigations
- **Risk:** Cache serving data across tenants  
  - **Mitigation:** Include `workspace_id` in cache key; always RBAC-check before serving  
  - **Monitoring:** metric on cache key cardinality + security review of key schema
- **Risk:** Transcript URL leaks or violates data residency  
  - **Mitigation:** generate time-bound signed URL in-region; never log URL; omit when disallowed  
  - **Monitoring:** log field allowlist enforcement tests

### Decision Log
- **Decision:** Return `200` with `status="queued"` when meeting exists but summary row/content is absent (rather than 404).  
  - **Alternatives:** 404 with “summary_not_initialized” code  
  - **Rationale:** simplifies UI polling and reduces ambiguous “missing vs not ready” states

---

## Open Questions
- [ ] Do we already have `updated_at` / `last_updated_at` for `meeting_summaries`? If not, should we add it now or compute from existing fields?
- [ ] Should `transcript_url` be returned at all to the UI, or only used by backend services?

---

## Implementation Notes (optional, non-binding)
- **Suggested approach**
  1. Validate auth + load meeting to get `workspace_id`
  2. RBAC check *(must precede cache read of content)*
  3. If status is likely `completed`, attempt Redis read *(key includes workspace + meeting)*
  4. On miss, query DB: `SELECT ... FROM meeting_summaries WHERE meeting_id = $1 ORDER BY <version/updated_at> DESC LIMIT 1`
  5. If completed, write-through cache with TTL=24h
  6. Map DB fields to response contract with safe defaults

---

## Checklist (before closing)
- [ ] Acceptance criteria met and verified
- [ ] Tests added/updated and passing
- [ ] Observability in place (metrics/logs/traces as applicable)
- [ ] Rollout plan documented and feature flag wired
- [ ] Docs/runbook updated (if needed)
- [ ] Security/privacy review completed
- [ ] Follow-up tickets created/linked for open questions (if not resolved)
