@"
# NovaDesk — AI Customer Support Automation

NovaDesk is a production-style AI customer support automation built with n8n.

It automates ticket intake, AI classification, confidence-based decisioning, team routing, escalation, human review, automated email replies, and audit logging.

## Core Features

- Webhook-based support ticket intake
- Input normalization and validation
- AI-powered ticket classification
- Structured output parsing
- Confidence-based routing
- Category-based team routing
- Critical escalation handling
- Human review and approval workflow
- Automated Gmail replies
- Slack notifications
- Ticket audit logging
- Integration logging
- Failure handling and fallback paths

## High-Level Architecture

```text
Support Form
    |
    v
Webhook
    |
    v
Normalize Ticket
    |
    v
Validate Required Fields
    |
    +--> Invalid Input
    |      |
    |      v
    |   Audit Log
    |
    v
AI Triage
    |
    +--> AI Failure
    |      |
    |      v
    |   Human Review Queue
    |
    v
Structured Output
    |
    v
Confidence Check
    |
    v
Business Decisioning
    |
    +--> Auto Reply
    |
    +--> Team Routing
    |
    +--> Escalation
    |
    +--> Human Review
            |
            v
       Approve / Reject
            |
            v
       Audit Update

Workflow Structure
The project is split into modular n8n workflows:
- main-workflow.json
- Category Routing & Decisioning.json
- human-review.json
- human-review-approval.json
- team-routing-notification.json
- Auto Reply.json
AI Decision Model
The AI layer interprets the incoming support ticket and produces structured fields such as:
- category
- priority
- sentiment
- business impact
- confidence
- summary
Business rules then determine what happens next.
Design Principle
AI interprets ambiguity.
Rules decide.
Humans handle uncertainty.
Confidence Logic
Typical confidence logic:
>= 0.85
Safe for automated response where business rules allow it.

>= 0.65 and < 0.85
Routing allowed, but no automatic reply.

< 0.65
Human review required.

Main Ticket Categories
- general_question
- technical_issue
- billing_issue
- account_issue
- refund_request
- complaint
- cancellation
- sales_inquiry
- other
Priority Levels
- low
- medium
- high
- critical
Human Review
Tickets requiring manual review are sent to Slack using an approval workflow.
Reviewer metadata is captured:
- review status
- reviewer name
- reviewer ID
- review source
- review timestamp
Possible review outcomes:
approved
rejected

Escalation
Critical tickets can be escalated to dedicated teams.
Example:
technical_issue
+
priority = critical
+
businessImpact = critical
=
technical_lead escalation

Escalation events are also written to the integration log.
Auto Reply
High-confidence tickets that are safe for automation can trigger an automated Gmail reply.
The workflow tracks:
- response type
- reply preparation time
- email delivery status
- first response time
Audit Model
NovaDesk uses two audit layers.
support_ticket_audit
Tracks business state and ticket lifecycle.
Examples:
- ticket metadata
- AI triage output
- route
- status
- decision reason
- escalation
- auto-reply state
- human review state
- reviewer metadata
- final business status
integration_log
Tracks external integration attempts.
Fields include:
- ticketId
- integration
- integrationAction
- integrationTarget
- integrationStatus
- attemptedAt
- completedAt
- errorMessage
Examples:
gmail / auto_reply
slack / team_notification
slack / escalation_notification
slack / human_review_notification

Failure Handling
The workflow includes controlled failure paths for cases such as:
- invalid ticket input
- AI provider failure
- Slack notification failure
- Gmail delivery failure
- low-confidence AI output
AI failures are routed to human review instead of terminating the entire business process.
Example Input
{
  "name": "Omar Hassan",
  "email": "customer@example.com",
  "subject": "Production system is completely down",
  "message": "Our production system is unavailable for all users and business operations have stopped completely."
}

Tested Scenarios
- General question → Auto reply
- Critical technical issue → Escalation
- Human review → Approve
- Human review → Reject
- Invalid input → Rejected before AI
- AI failure → Human review fallback
- Slack notification
- Gmail delivery
- Ticket audit updates
- Integration logging
Security Notes
Sensitive credentials are not stored in the repository.
n8n credentials should be configured inside the n8n credential manager.
Do not hard-code:
- API keys
- OAuth tokens
- passwords
- webhook secrets
- private customer data
Local Development
NovaDesk was developed using:
Windows
Docker Desktop
n8n
Docker Compose
ngrok
Slack
Gmail

Typical local access:
http://localhost:5678

ngrok can be used during development when external services need to reach local webhooks.
Production Deployment
A recommended production setup would be:
Ubuntu VPS
    |
    v
Docker Compose
    |
    +--> n8n
    |
    +--> PostgreSQL
    |
    +--> Reverse Proxy
    |
    +--> HTTPS

Production deployments should also include:
- backups
- monitoring
- retry policies
- secure secret management
- rate-limit handling
- environment separation
Project Goal
This project demonstrates how n8n can be used as an automation orchestration layer for a production-style AI customer support workflow with deterministic business logic, human-in-the-loop controls, auditability, and external integrations.
