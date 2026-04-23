# Lead Capture Sequence Diagram

## Scenario: New Lead Submission from Landing Page

This sequence diagram shows the complete flow when a new lead submits their information through the landing page contact form.

> See diagram: [new-lead-submission-from-landing-page.md](diagrams/new-lead-submission-from-landing-page.md)
![new-lead-submission-from-landing-page](./diagrams/new-lead-submission-from-landing-page.png)

---

## Scenario: Chatbot Lead Capture

> See diagram: [chatbot-lead-capture.md](diagrams/chatbot-lead-capture.md)
![chatbot-lead-capture](./diagrams/chatbot-lead-capture.png)

---

## Scenario: Voice Call Qualification

> See diagram: [voice-call-qualification.md](diagrams/voice-call-qualification.md)
![voice-call-qualification](./diagrams/voice-call-qualification.png)

---

## Scenario: Content Generation and Publishing

> See diagram: [content-generation-and-publishing.md](diagrams/content-generation-and-publishing.md)
![content-generation-and-publishing](./diagrams/content-generation-and-publishing.png)

---

## Key Patterns

### 1. Async Processing with Queues
- All heavy operations (CRM sync, content gen, voice calls) are queued
- Redis Queue (BullMQ) ensures reliability with retry logic
- Webhook handlers return immediately, processing happens async

### 2. Circuit Breakers
- Each integration has a circuit breaker
- After 3 consecutive failures, integration auto-disables
- Background job retries with exponential backoff

### 3. Idempotency
- All webhook handlers are idempotent (safe to retry)
- Duplicate lead submissions return existing lead ID
- CRM sync jobs check if already synced

### 4. Event-Driven Updates
- Lead events trigger multiple downstream actions
- Hot lead → alert + nurture pause + CRM deal creation
- Engagement events → lead score recalculation + nurture adjustment
