# [Project Echo] AI-Powered Smart Meeting Summaries

**Status:** In-Review  
**Owner:** Jamie (Senior PM)  
**Engineers:** @tech-lead-dave / @backend-sara  
**Last Updated:** 2026-01-29

---

## 1. Executive Summary
Project Echo aims to reduce "meeting fatigue" by automatically generating concise, searchable summaries and action items from recorded video calls, directly integrated into the user’s workspace.

## 2. Context & Problem Statement

### The "Why"
* **Problem:** Users spend an average of 3 hours per week re-watching meeting recordings or manually typing notes to share with teammates. Information "leakage" occurs when action items aren't captured.
* **Evidence:** 65% of churned users in Q4 cited "lack of actionable insights from collaboration" as a primary reason for leaving.
* **Business Impact:** Increases "stickiness" of our platform as the primary source of truth for team decisions and project momentum.

### Target Audience
* **Product Managers & Team Leads:** Need to track decisions across multiple syncs without attending every call.
* **Individual Contributors:** Need to catch up on missed meetings quickly without watching 60-minute videos.

---

## 3. Goals & Success Metrics

### Primary Goals
* Automate the extraction of "Key Decisions" and "Action Items."
* Enable users to share a summary to Slack or Email with a single click.
* Provide a "Search across summaries" feature to find past decisions.

### Non-Goals (Out of Scope)
* **Real-time live transcription:** We are focusing on post-meeting processing for the MVP.
* **Video editing:** We will not provide tools to trim or cut the video files.

### Success Metrics
| Metric | Current Baseline | Target |
| :--- | :--- | :--- |
| **Feature Adoption** | 0% | 25% of all recorded meetings summarized |
| **Action Item Completion** | N/A | 15% increase in tasks marked "Done" within 48hrs |
| **Time to Summary** | Manual (15-30 mins) | Automated (< 3 mins post-upload) |

---

## 4. User Stories
| As a... [User Type] | I want to... [Action] | So that... [Value] |
| :--- | :--- | :--- |
| Busy Manager | See a 3-bullet summary of a meeting I missed | I can stay informed without losing an hour of my day. |
| Project Lead | Have action items automatically assigned to people | Nothing falls through the cracks after the call ends. |
| Privacy Officer | Ensure PII is redacted from AI summaries | We remain compliant with SOC2 and GDPR requirements. |

---

## 5. Functional Requirements

### [A] Transcription & AI Processing
* **REQ-1.1:** System must trigger an asynchronous job to transcribe MP4/WebM uploads upon "Meeting End" signal.
* **REQ-1.2:** Use LLM to generate:
    * A high-level **TL;DR** (max 300 characters).
    * Bulleted **Key Decisions**.
    * Bulleted **Action Items** with detected owners (e.g., "Dave to update the API docs").

### [B] UI / Workspace Integration
* **REQ-2.1:** Display the summary on the Video Playback page in a persistent right-hand sidebar.
* **REQ-2.2:** "Copy to Clipboard" button must format the output in clean Markdown for external use.
* **REQ-2.3:** Provide an "Edit" mode so users can manually correct AI-generated text.

---

## 6. Technical & Design Constraints
* **UI/UX:** Must follow the "Shadow" design system components for sidebars and typography.
* **Data Storage:** Summaries should be stored in the `meeting_metadata` table; do not re-run AI processing on every page load—cache the result.
* **Security/Privacy:** All transcripts must be encrypted at rest. Users must opt-in to "AI Analysis" at the Workspace level.
* **Non-Functional Requirements:** AI processing must complete within 20% of the total video duration (e.g., a 10-minute video should be summarized in < 2 minutes).

---

## 7. Edge Cases & Error Handling
* **Edge Case 1: No Audio Detected**
    * *Expected Behavior:* If the transcript is empty or < 50 words, show a "Summary unavailable: low audio quality" message instead of a blank box.
* **Edge Case 2: Multi-Language Audio**
    * *Expected Behavior:* Detect the primary language; if not English, provide a "Translate to English" toggle for the summary.
* **Edge Case 3: Token Limit (Long Meetings)**
    * *Expected Behavior:* For meetings > 3 hours, truncate transcript to the most recent 90 minutes and notify the user that only the latter half was summarized.

---

## 8. Rollout Plan
1.  **Internal Beta:** Feb 5 (Internal Product and Eng teams only).
2.  **Canary Release (5% of Pro Users):** Feb 12.
3.  **General Availability:** Feb 26.

---

## 9. FAQ / Appendices
* **How accurate is the action item detection?** Initial internal testing shows ~88% accuracy. This is why the "Edit" feature (REQ-2.3) is a P0 requirement.
* **Cost per summary?** Estimated $0.02 USD per hour of video processed via the current provider.