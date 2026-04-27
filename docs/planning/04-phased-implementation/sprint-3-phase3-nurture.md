---
layout: page
title: Sprint 3 - Nurture
parent: Implementation Plan
grand_parent: Project Planning
nav_order: 5
---

# Sprint 3 — Phase 3: Automated Nurture & Closing

**Duration**: 8 working days  
**Team**: Backend Developer + AI/ML Engineer  
**Prerequisites**: Sprint 2 complete (CRM + lead scoring operational)

---

## Part A: Drip Campaign Sequences (Days 1-4)

### Day 1-2: Email Templates & Content

#### Task 3.1: Email Template System
- Branded responsive HTML email template (base layout)
  - Agent logo header
  - Content area
  - Call-to-action button
  - Footer with contact info, social links, unsubscribe link
  - Dark mode compatible
- Use MJML or handcrafted HTML for email rendering
- Template variables: `{{firstName}}`, `{{agentName}}`, `{{ctaUrl}}`, etc.
- **Files**: `src/lib/email-templates/base.html`, `src/lib/email-renderer.ts`
- **Time**: 4 hours

#### Task 3.2: Buyer Drip Sequence Content
- **Day 1 — Welcome + Market Overview**
  - Subject: "Welcome, {{firstName}}! Your NYC property journey starts here"
  - Body: Agent intro, current market snapshot, what to expect next
  - CTA: "Browse Featured Properties"

- **Day 2 — Neighborhood Guide**
  - Subject: "Discover NYC's hidden gem neighborhoods"
  - Body: Top 5 neighborhoods for buyers, price ranges, lifestyle comparison
  - CTA: "Get My Personalized Neighborhood Guide"

- **Day 3 — Mortgage Resources**
  - Subject: "Financing your dream home: What you need to know"
  - Body: Pre-approval checklist, mortgage calculator link, partnered lenders
  - CTA: "Calculate Your Budget"

- **Day 4 — Success Story**
  - Subject: "How the Martinez family found their perfect home"
  - Body: Client success story, testimonial, before/after story
  - CTA: "Start Your Search"

- **Day 5 — Personal CTA**
  - Subject: "{{firstName}}, let's find your home this week"
  - Body: Direct, personal message from agent, limited availability urgency
  - CTA: "Book a Free Consultation"
- **Files**: `src/data/email-sequences/buyer-sequence.ts`
- **Time**: 3 hours

#### Task 3.3: Seller Drip Sequence Content
- **Day 1 — Free Valuation Offer**
  - Subject: "How much is your home really worth?"
  - CTA: "Get Your Free Home Valuation"

- **Day 2 — Market Trends**
  - Subject: "NYC seller's advantage: Why now is the time"
  - CTA: "See Market Data"

- **Day 3 — Staging Tips**
  - Subject: "5 staging secrets that add $50K+ to your sale price"
  - CTA: "Download Staging Guide"

- **Day 4 — Success Story**
  - Subject: "Sold in 5 days, 12% over asking: The Johnson story"
  - CTA: "Read More Success Stories"

- **Day 5 — Listing Appointment CTA**
  - Subject: "{{firstName}}, your home deserves a top listing agent"
  - CTA: "Schedule Your Listing Appointment"
- **Files**: `src/data/email-sequences/seller-sequence.ts`
- **Time**: 2 hours

### Day 3-4: Nurture Engine

#### Task 3.4: Drip Campaign Engine
- Campaign orchestrator:
  1. Enrollment trigger: lead classified with intent type
  2. Schedule emails at defined intervals (Day 1, 2, 3, 4, 5)
  3. Track delivery status (sent, opened, clicked, bounced)
  4. Suppression rules: don't send if lead manually contacted, or deal closed
  5. Goal: if meeting booked, exit sequence
- Use BullMQ delayed jobs for scheduling
- **Files**: `src/lib/nurture-engine.ts`, `src/workers/nurture-worker.ts`
- **Time**: 6 hours

#### Task 3.5: Email Sending Service
- SendGrid integration for sending branded emails
- Track events via SendGrid webhooks:
  - delivered, opened, clicked, bounced, unsubscribed
- Update lead engagement score on each event
- **Files**: `src/lib/email-sender.ts`, `src/app/api/webhooks/sendgrid/route.ts`
- **Time**: 3 hours

#### Task 3.6: SMS Follow-ups
- Parallel SMS touchpoints via Twilio:
  - Day 1: "Hi {{firstName}}, thanks for reaching out! I'm {{agentName}}, your NYC real estate expert. Reply here or call me anytime at {{phone}}."
  - Day 3: "{{firstName}}, I noticed some perfect listings for you. Check them out: {{link}}. Any questions? Just reply!"
  - Day 5: "Last chance! I have some exclusive listings that match your criteria. Let's schedule a quick call: {{bookingLink}}"
- Opt-out handling ("STOP" keyword)
- **Files**: `src/lib/sms-sender.ts`, update nurture engine
- **Time**: 3 hours

#### Task 3.7: Engagement Tracking & Escalation
- Track engagement signals:
  - Email opened → engagement score +5
  - Email link clicked → engagement score +10
  - SMS reply → engagement score +20
  - No engagement after Day 3 → flag for escalation
- Escalation logic:
  - If lead is HOT and no engagement after Day 3 → trigger AI voice call (Task 3.8)
  - If lead is WARM and no engagement after Day 5 → notify agent for manual outreach
  - Update HubSpot contact with engagement data
- **Files**: `src/lib/engagement-tracker.ts`
- **Time**: 3 hours

---

## Part B: AI Voice Qualification Calls (Days 5-8)

### Day 5-6: Voice Agent Setup

#### Task 3.8: Voice Agent Configuration
- Create Bland.ai (or Vapi.ai) account and configure:
  - Provision phone number
  - Set up agent profile: name, voice, language
  - Configure webhook for call outcomes
- **Files**: `src/lib/voice-agent.ts`
- **Time**: 2 hours

#### Task 3.9: Call Script Design
- Structured qualification script:

```
SCRIPT FLOW:
1. GREETING
   "Hi, is this {{firstName}}? Great! This is {{agentVoiceName}} calling from 
    {{agentName}}'s office. You recently inquired about real estate in NYC. 
    Is now a good time to chat for about 2 minutes?"
   
   → If NO: "No problem! When would be a good time to call back?"
             → Schedule callback, end call
   → If YES: Continue

2. QUALIFY — Property Type
   "What type of property are you looking for? For example, are you interested 
    in a condo, co-op, townhouse, or single-family home?"
   → Capture: property_type

3. QUALIFY — Timeline
   "How soon are you looking to make a move?"
   → Capture: timeline (immediate / 1-3 months / 3-6 months / just exploring)

4. QUALIFY — Budget
   "Do you have a general budget range in mind? This helps me find the best 
    matches for you."
   → Capture: budget_range

5. QUALIFY — Decision Authority
   "Will you be making this decision on your own, or with a partner or family?"
   → Capture: decision_authority

6. BOOKING (if qualified)
   "Based on what you've told me, I think {{agentName}} would be a great fit 
    to help you. Would you like to schedule a free consultation? I have 
    openings on {{availableSlots}}."
   → If YES: Book appointment, send confirmation
   → If NO: "No problem! I'll have {{agentName}} send you some listings 
             that match what you're looking for."

7. CLOSING
   "Thank you for your time, {{firstName}}! You'll receive a confirmation 
    email shortly. Have a great day!"
```

- **Files**: `src/data/voice-scripts/qualification-script.ts`
- **Time**: 3 hours

#### Task 3.10: Voice Call Trigger
- Trigger conditions:
  - Lead is HOT classification
  - No email/SMS engagement after Day 3
  - Lead has phone number on file
  - Max 3 call attempts per lead
  - Call during business hours only (9 AM - 7 PM EST)
- Queue via BullMQ with scheduling
- **Files**: `src/workers/voice-call-worker.ts`, update nurture engine
- **Time**: 3 hours

### Day 7: Calendar Integration & Recording

#### Task 3.11: Google Calendar Booking
- Google Calendar API setup (service account)
- Fetch available slots from agent's calendar
- Create calendar event with:
  - Attendees: agent + lead
  - Location: office address or Zoom link
  - Description: lead summary, property interest, qualification data
  - Reminder: 1 hour before
- Send confirmation email to lead
- **Files**: `src/lib/google-calendar.ts`, `src/app/api/calendar/route.ts`
- **Time**: 4 hours

#### Task 3.12: Booking Widget
- Embeddable calendar component for thank-you page and emails
- Show available time slots
- Lead selects slot → creates booking → confirmation
- **Files**: `src/components/calendar/BookingWidget.tsx`
- **Time**: 3 hours

#### Task 3.13: Call Recording & Summary
- Bland.ai webhook receives call completion:
  - Call recording URL
  - Call transcript (raw text)
  - Call duration, outcome
- Post-processing:
  - GPT-4o summarization of transcript
  - Extract: qualification answers, sentiment, next action
  - Store in DB with link to lead
  - Update HubSpot contact notes with summary
  - Update lead score based on call outcome
- **Files**: `src/app/api/webhooks/voice/route.ts`, `src/lib/call-processor.ts`
- **Time**: 4 hours

### Day 8: Outcome Routing & Testing

#### Task 3.14: Call Outcome Routing
- **Qualified + Meeting Booked**: 
  - Update lead status → "Meeting Booked"
  - Advance HubSpot deal stage
  - Send confirmation email
  - Pause drip sequence
- **Qualified + No Meeting**:
  - Update lead data with qualification answers
  - Continue drip sequence with updated personalization
  - Alert agent for manual follow-up
- **Not Qualified**:
  - Lower lead score by 20 points
  - Re-classify if needed (HOT → WARM)
  - Continue lighter nurture sequence
- **No Answer** (max 3 attempts):
  - Schedule retry: attempt 2 at +24h, attempt 3 at +48h
  - After 3 failed attempts: alert agent, continue email-only nurture
- **Files**: `src/lib/outcome-router.ts`
- **Time**: 3 hours

#### Task 3.15: Sprint 3 Integration Testing
- E2E: Lead created → drip emails scheduled → emails sent → engagement tracked
- E2E: No engagement → voice call triggered → call completed → calendar booked
- Verify: SendGrid webhook updates engagement
- Verify: Bland.ai webhook processes call summary
- Verify: Google Calendar event created with correct data
- Verify: HubSpot contact updated with call summary
- **Time**: 3 hours

---

## Sprint 3 Deliverables Checklist

| # | Deliverable | Status |
|---|---|---|
| 3.1 | Branded email template system | ☐ |
| 3.2 | Buyer drip sequence (5 emails) | ☐ |
| 3.3 | Seller drip sequence (5 emails) | ☐ |
| 3.4 | Drip campaign engine (BullMQ) | ☐ |
| 3.5 | Email sending via SendGrid + webhooks | ☐ |
| 3.6 | SMS follow-ups via Twilio | ☐ |
| 3.7 | Engagement tracking + escalation | ☐ |
| 3.8 | Voice agent configuration | ☐ |
| 3.9 | Qualification call script | ☐ |
| 3.10 | Voice call trigger logic | ☐ |
| 3.11 | Google Calendar integration | ☐ |
| 3.12 | Calendar booking widget | ☐ |
| 3.13 | Call recording processing + CRM sync | ☐ |
| 3.14 | Call outcome routing logic | ☐ |
| 3.15 | Sprint 3 integration tests | ☐ |
