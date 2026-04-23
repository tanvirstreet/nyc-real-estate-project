---
layout: page
title: Sprint 2 - CRM & Leads
---

# Sprint 2 — Phase 2: CRM & Intelligent Lead Management

**Duration**: 7 working days  
**Team**: Full-Stack Developer + Backend Developer  
**Prerequisites**: Sprint 0 + Sprint 1 complete

---

## Day 1-2: HubSpot Integration

### Task 2.1: HubSpot API Client
- Install `@hubspot/api-client`
- Create authenticated client wrapper with:
  - Token refresh handling
  - Rate limit awareness (100 requests/10 seconds)
  - Retry with exponential backoff (429 responses)
  - Request/response logging
- **Files**: `src/lib/hubspot.ts`
- **Time**: 3 hours

### Task 2.2: Custom CRM Properties
- Create custom HubSpot contact properties via API:
  - `lead_source` (enumeration): WEBSITE_FORM, CHATBOT, VOICE_CALL, SOCIAL
  - `intent_type` (enumeration): BUYER, SELLER, BOTH, RENTER, INVESTOR
  - `property_interest` (text): specific property or area
  - `budget_range` (enumeration): <500K, 500K-1M, 1M-2M, 2M-5M, 5M+
  - `timeline` (enumeration): IMMEDIATE, 1_3_MONTHS, 3_6_MONTHS, 6_12_MONTHS, JUST_BROWSING
  - `lead_score` (number): 0-100
  - `lead_classification` (enumeration): HOT, WARM, COLD
  - `chatbot_summary` (multiline text): latest conversation summary
- Script to create properties: `scripts/setup-hubspot-properties.ts`
- **Time**: 2 hours

### Task 2.3: Contact Creation Pipeline
- On form submission (`/api/contact`):
  1. Create contact in local DB
  2. Search HubSpot for existing contact (by email)
  3. Create or update HubSpot contact with all custom properties
  4. Map form data to CRM properties
  5. Store HubSpot contact ID in local DB
- Handle deduplication: merge if existing contact found
- **Files**: Update `src/app/api/contact/route.ts`, `src/lib/hubspot.ts`
- **Time**: 4 hours

### Task 2.4: Chatbot → CRM Sync
- At conversation end (triggered by inactivity timeout or explicit close):
  1. Generate conversation summary via GPT-4o
  2. Extract structured data: intent, property interest, budget, timeline
  3. Search/create HubSpot contact (match by email if provided, otherwise create as "Anonymous Visitor")
  4. Update contact with conversation data
  5. Create HubSpot engagement (note) with full transcript
- **Files**: `src/lib/crm-sync.ts`, update chatbot route
- **Time**: 4 hours

---

## Day 3-4: Lead Scoring & Classification

### Task 2.5: Lead Scoring Engine
- Rule-based scoring algorithm (0-100 scale):

```
SCORING RULES:
┌─────────────────────────────────────────────┬────────┐
│ Signal                                      │ Points │
├─────────────────────────────────────────────┼────────┤
│ Form submitted (vs chatbot only)            │ +20    │
│ Phone number provided                       │ +10    │
│ Specific property interest mentioned        │ +15    │
│ Budget range specified                      │ +10    │
│ Timeline is IMMEDIATE                       │ +25    │
│ Timeline is 1-3 months                      │ +15    │
│ Timeline is 3-6 months                      │ +5     │
│ Intent is BUYER or SELLER (vs browsing)     │ +10    │
│ Multiple page views (>3 properties)         │ +5     │
│ Return visitor                              │ +10    │
│ Chatbot conversation (>5 messages)          │ +10    │
│ Requested human escalation                  │ +15    │
│ Clicked CTA                                 │ +5     │
│ ─────────────────────────────────────────── │ ────── │
│ CLASSIFICATION THRESHOLDS                   │        │
│ HOT:  score >= 70                           │ 🔴     │
│ WARM: score >= 35                           │ 🟡     │
│ COLD: score < 35                            │ 🔵     │
└─────────────────────────────────────────────┴────────┘
```

- Score recalculated on every new signal
- Score and classification synced to HubSpot
- **Files**: `src/lib/lead-scorer.ts`
- **Time**: 4 hours

### Task 2.6: HubSpot Pipeline Configuration
- Create sales pipeline via API:
  - **Stages**: New → Qualified → Contacted → Meeting Booked → Proposal Sent → Negotiation → Closed Won → Closed Lost
  - Auto-create deal when lead classified as HOT
  - Auto-advance stage based on events (meeting booked → "Meeting Booked" stage)
- **Files**: `scripts/setup-hubspot-pipeline.ts`, `src/lib/hubspot.ts`
- **Time**: 3 hours

### Task 2.7: Score Update Integration
- Trigger re-scoring on events:
  - Form submission
  - Chatbot conversation end
  - Property page view (tracked via analytics)
  - Email opened / link clicked (Phase 3)
  - Voice call completed (Phase 3)
- Update both local DB and HubSpot on every score change
- **Files**: `src/lib/lead-scorer.ts`, event handlers
- **Time**: 2 hours

---

## Day 5: Alert System

### Task 2.8: WhatsApp Alert (Twilio)
- Configure Twilio WhatsApp Business API
- Template message: "🔥 HOT LEAD: {name} — {interest} — {intent} — Score: {score} — {phone} — {email}"
- Trigger: lead classified as HOT
- Fallback: if WhatsApp fails, send email alert
- **Files**: `src/lib/twilio.ts`, `src/app/api/alerts/route.ts`
- **Time**: 3 hours

### Task 2.9: Email Alert (SendGrid)
- Formatted HTML email alert to agent
- Subject: "🔥 New HOT Lead: {name} — {intent}"
- Body: lead details, conversation summary, direct link to HubSpot contact
- Always send as backup (even if WhatsApp succeeds)
- **Files**: `src/lib/sendgrid.ts`, update alert route
- **Time**: 2 hours

### Task 2.10: Alert Queue & Reliability
- Use BullMQ queue for alert delivery:
  - Prevents duplicate alerts
  - Retry on failure (3 attempts)
  - Dead letter queue for failed alerts
  - Alert delivery status logging
- **Files**: `src/lib/queue.ts`, `src/workers/alert-worker.ts`
- **Time**: 3 hours

---

## Day 6-7: Admin Dashboard

### Task 2.11: Admin Authentication
- NextAuth.js configuration:
  - Google OAuth provider (agent's Google account)
  - Credentials provider (email/password backup)
  - Session-based auth for dashboard pages
  - Protected route middleware for `/admin/*`
- **Files**: `src/app/api/auth/[...nextauth]/route.ts`, `src/middleware.ts`
- **Time**: 3 hours

### Task 2.12: Lead Management Dashboard
- Route: `/admin/leads`
- Features:
  - Lead table: name, email, phone, score, classification, source, date
  - Sort by: score, date, classification
  - Filter by: classification (HOT/WARM/COLD), source, date range
  - Search by name or email
  - Click row to expand lead detail
- Lead detail panel:
  - Full contact info
  - Scoring breakdown (which signals contributed)
  - Conversation history (from chatbot)
  - HubSpot link
  - Manual score override
  - Manual classification change
  - Notes field
- **Files**: `src/app/admin/leads/page.tsx`, `src/components/admin/LeadTable.tsx`, `src/components/admin/LeadScoreCard.tsx`
- **Time**: 6 hours

### Task 2.13: Dashboard Overview
- Route: `/admin`
- Metrics cards:
  - Total leads (this month)
  - HOT leads (this month)
  - Conversion rate
  - Average score
- Mini charts:
  - Leads by day (last 30 days)
  - Leads by source (pie chart)
  - Classification distribution (donut chart)
- Recent activity feed
- **Files**: `src/app/admin/page.tsx`, `src/components/admin/AnalyticsDashboard.tsx`
- **Time**: 4 hours

### Task 2.14: Sprint 2 Integration Testing
- E2E: Form → DB → HubSpot → Score → Classify → Alert
- E2E: Chatbot → DB → HubSpot sync → Score
- Verify: HubSpot contact has correct custom properties
- Verify: WhatsApp/Email alert received for HOT lead
- Admin: Lead table shows correct data, filters work
- **Time**: 3 hours

---

## Sprint 2 Deliverables Checklist

| # | Deliverable | Status |
|---|---|---|
| 2.1 | HubSpot API client with rate limiting | ☐ |
| 2.2 | Custom CRM properties created | ☐ |
| 2.3 | Contact creation pipeline (form → CRM) | ☐ |
| 2.4 | Chatbot → CRM sync | ☐ |
| 2.5 | Lead scoring engine (rule-based) | ☐ |
| 2.6 | HubSpot sales pipeline setup | ☐ |
| 2.7 | Score update on all events | ☐ |
| 2.8 | WhatsApp alert (Twilio) | ☐ |
| 2.9 | Email alert (SendGrid) | ☐ |
| 2.10 | Alert queue with retry | ☐ |
| 2.11 | Admin authentication (NextAuth) | ☐ |
| 2.12 | Lead management dashboard | ☐ |
| 2.13 | Dashboard overview with metrics | ☐ |
| 2.14 | Sprint 2 integration tests | ☐ |
