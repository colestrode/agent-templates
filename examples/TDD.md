# TDD: [Project Echo] AI-Powered Smart Meeting Summaries

**Author:** Senior Staff Engineer
**Status:** In-Review
**Date:** 2026-01-29
**Reviewers:** @tech-lead-dave, @backend-sara, @security-review, @infra-team

---

## 1. Executive Summary
Project Echo introduces an automated pipeline to transcribe meeting recordings and generate structured summaries (TL;DR, Decisions, Action Items) using Large Language Models (LLMs). The system leverages an event-driven architecture to process video files asynchronously post-upload, ensuring meeting insights are delivered to the user's workspace within minutes of the call ending.

## 2. Context & Requirements
### 2.1 Problem Statement
Users currently lose ~3 hours/week manually summarizing meetings or re-watching recordings. Information "leakage" occurs when action items discussed in calls are not captured in the system of record.
* **PRD Reference:** [Project Echo - Smart Summaries](link-to-prd)

### 2.2 Goals (Must-Haves)
* **Async Extraction:** Automate transcription and summarization after "Meeting End".
* **Standardized Output:** Every summary must contain a TL;DR (< 300 chars), Key Decisions, and Action Items with detected owners.
* **Searchability:** Summaries must be indexed for global workspace search.
* **User Agency:** Users must be able to edit AI-generated text to correct hallucinations.

### 2.3 Non-Goals (Out of Scope)
* **Real-time Transcription:** Processing occurs only after the file is fully uploaded/saved.
* **Video Manipulation:** No clipping, trimming, or editing of the source MP4/WebM files.

## 3. Proposed Design

### 3.1 High-Level Architecture
The system follows an event-driven "Sidecar" pattern. We will decouple the core video storage logic from the AI processing pipeline using a message broker to ensure high availability and retry logic.



### 3.2 Detailed Component Design
1.  **Event Dispatcher:** Listens for `video.upload.completed` events from the Media Service.
2.  **Orchestration Worker (Echo-Worker):** A specialized service that manages the job state (Pending -> Transcribing -> Summarizing -> Completed).
3.  **Transcription Adapter:** Interacts with the external Speech-to-Text provider. Handles multi-language detection and returns a timestamped JSON transcript.
4.  **LLM Summarizer:** An internal service that constructs the prompt, handles PII redaction, and calls the LLM (e.g., GPT-4o or Claude 3.5).
5.  **Workspace UI Sidebar:** A new React component following the "Shadow" design system to display and edit results.

### 3.3 Data Model & Schema Changes
We will create a dedicated table for summaries to allow for versioning (tracking AI vs. Human edits) and to avoid bloating the primary `meeting_metadata` table.

**Table: `meeting_summaries`**
| Field | Type | Description |
| :--- | :--- | :--- |
| `id` | UUID (PK) | Unique identifier |
| `meeting_id` | UUID (FK) | Reference to `meetings.id` |
| `status` | ENUM | `queued`, `transcribing`, `summarizing`, `completed`, `failed` |
| `tldr` | VARCHAR(300) | Short executive summary |
| `structured_data` | JSONB | Contains `{decisions: [], action_items: [{owner: string, task: string}]}` |
| `is_edited` | BOOLEAN | `true` if a human has modified the AI output |
| `transcript_s3_url` | TEXT | Encrypted link to the raw transcript |

### 3.4 API Design
* **`GET /v1/meetings/:id/summary`**
    * Fetch the current summary state and data.
* **`PATCH /v1/meetings/:id/summary`**
    * **Payload:** `{ tldr: string, structured_data: JSON }`
    * Updates the summary and sets `is_edited = true`.
* **`GET /v1/workspaces/:id/summaries/search?q=query`**
    * Search across all meeting summaries within a workspace.

## 4. Non-Functional Requirements
### 4.1 Scalability & Performance
* **Throughput:** Must handle peak bursts of 500+ concurrent meeting completions.
* **Processing Latency:** Target $TotalTime \le 0.2 \times VideoDuration$.
* **Caching:** Results will be cached in Redis for 24 hours post-generation to handle high initial view rates.

### 4.2 Security & Privacy
* **PII Redaction:** The `LLM Summarizer` will use a Pre-processor to mask PII (Emails, Phone Numbers) before the transcript hits the external AI API.
* **Data Residency:** All transcripts must be stored in the region corresponding to the Workspace's data residency policy.
* **RBAC:** Summary access is restricted to users who have `view` permissions on the original meeting recording.

### 4.3 Observability
* **Logging:** Standardized logs for `job_id` to trace a request from upload to summary display.
* **SLA Monitoring:** Alerting if `processing_status == 'queued'` for $> 10$ minutes.
* **Accuracy Tracking:** Log a metric whenever a user clicks "Edit" to determine the LLM's "Edit Rate" (lower is better).

## 5. Implementation Plan
### 5.1 Migration & Rollout Strategy
1.  **Dark Launch:** Deploy the background workers to process internal meetings without showing the UI.
2.  **Beta Toggle:** Enable the "Echo Sidebar" for the `@product-team` workspace.
3.  **Phased Rollout:** Release to 5%, 25%, and 100% of Pro users via Feature Flags.

### 5.2 Test Plan
* **Unit Tests:** Verify the Markdown formatter correctly handles various special characters and bullet nesting.
* **Integration Tests:** End-to-end flow using a 60-second test video to verify the SQS -> Worker -> DB path.
* **Load Test:** Simulate 1,000 concurrent meeting endings to ensure the Orchestration Worker scales without hitting DB connection limits.

## 6. Alternatives Considered
* **Database-per-Summary:** Considered storing summaries in a NoSQL store (DynamoDB). Rejected because we need relational joins with `workspaces` for the search feature and RBAC checks.
* **On-the-fly Summarization:** Considered generating the summary only when the user opens the sidebar. Rejected due to the ~60s latency of LLMs; users expect the summary to be ready when they arrive.

## 7. Risks & Unknowns
* **LLM Hallucinations:** AI may assign action items to the wrong person.
    * *Mitigation:* UI will include a "Verify this summary" disclaimer and the Edit mode.
* **Transcription Quality:** Low-quality audio/heavy accents may lead to poor summaries.
    * *Mitigation:* Implement REQ-1.2 (Multi-language detection) and Edge Case 1 (Low audio detection).