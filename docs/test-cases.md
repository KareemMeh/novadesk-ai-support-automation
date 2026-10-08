# NovaDesk — Test Cases

This document summarizes the main scenarios I used to verify the NovaDesk workflow during development.

## 1. General Question → Auto Reply

**Goal:** Verify that a safe, high-confidence general question can be answered automatically.

**Expected result:**
- Ticket is accepted and normalized.
- AI classification succeeds.
- Confidence is high enough for auto reply.
- Business rules allow automated response.
- Gmail reply is sent.
- Ticket audit is updated.
- Integration log records the email action.

**Result:** Passed.

---

## 2. Critical Technical Issue → Escalation

**Goal:** Verify that a critical technical issue is escalated correctly.

**Expected result:**
- Ticket is classified as a technical issue.
- Priority/business impact triggers escalation.
- Slack escalation notification is sent.
- Escalation state is stored in the ticket audit.
- Integration log records the Slack escalation action.

**Result:** Passed.

---

## 3. Human Review → Approve

**Goal:** Verify the human-in-the-loop approval path.

**Expected result:**
- Ticket is sent to the human review queue.
- Slack approval request is delivered.
- Reviewer selects Approve.
- Reviewer metadata is captured.
- Ticket audit is updated with the approved state.

**Result:** Passed.

---

## 4. Human Review → Reject

**Goal:** Verify the human rejection path.

**Expected result:**
- Ticket is sent to the human review queue.
- Slack approval request is delivered.
- Reviewer selects Reject.
- Reviewer metadata is captured.
- Ticket audit is updated with the rejected state.

**Result:** Passed.

---

## 5. Invalid Input

**Goal:** Verify that incomplete requests are rejected before AI processing.

**Expected result:**
- Required-field validation fails.
- AI triage is not called.
- Route becomes `rejected_input`.
- Status becomes `invalid`.
- Decision reason explains that required fields are missing.
- Ticket audit is saved.

**Result:** Passed.

---

## 6. AI Provider Failure → Human Review Fallback

**Goal:** Verify that an AI failure does not stop the whole business process.

**Expected result:**
- AI triage fails.
- Ticket is routed to `human_review_queue`.
- Status becomes `waiting_human_review`.
- `humanReviewRequired` is enabled.
- Automatic reply is disabled.
- Ticket continues through the human review path.

**Result:** Passed.

---

## 7. Human Review Notification

**Goal:** Verify Slack notification delivery for human review.

**Expected result:**
- Human review notification is sent to Slack.
- `humanReviewNotificationStatus` is updated to `sent`.
- Notification timestamp is stored.
- Human decision fields are not overwritten by the notification update.

**Result:** Passed.

---

## 8. Gmail Delivery

**Goal:** Verify the auto-reply integration.

**Expected result:**
- Reply content is prepared.
- Gmail sends the customer response.
- `autoReplied` is updated.
- `firstResponseAt` is stored.
- Integration log records the email action.

**Result:** Passed.

---

## 9. Ticket Audit Update Isolation

**Goal:** Verify that each update step only changes the fields it owns.

**Expected result:**
- Notification updates do not overwrite human-review decisions.
- Human-review updates do not overwrite notification state.
- Email updates do not overwrite unrelated routing or escalation fields.

**Result:** Passed.

---

## 10. Integration Logging

**Goal:** Verify that external integration attempts are logged separately from business-state audit data.

**Expected result:**
- Gmail actions are recorded.
- Slack team notifications are recorded.
- Slack escalation notifications are recorded.
- Human-review notifications can be recorded.
- Integration status and timestamps are stored.

**Result:** Passed.

---

## Notes

These tests were performed during local development using n8n, Docker, Slack, Gmail, Gemini, and local webhook access.

The project is a portfolio and learning implementation, not a production SLA-certified system.
