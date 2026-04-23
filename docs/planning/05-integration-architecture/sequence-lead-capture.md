# Lead Capture Sequence Diagram

## Scenario: New Lead Submission from Landing Page

This sequence diagram shows the complete flow when a new lead submits their information through the landing page contact form.

> See diagram: [sequence-lead-capture.md](diagrams/new-lead-submission-from-landing-page.md)
![sequence-lead-capture](./diagrams/new-lead-submission-from-landing-page.png)

---

## Scenario: Chatbot Lead Capture

```mermaid
sequenceDiagram
    autonumber
    actor User as Website Visitor
    participant Chat as Chatbot Widget
    participant API as Chatbot API
    participant RAG as RAG Engine
    participant OpenAI as OpenAI API
    participant LS as Lead Service
    participant DB as PostgreSQL

    User->>Chat: "I'm looking for a 2BR in Brooklyn"
    Chat->>API: POST /api/v1/chatbot/message
    API->>RAG: Retrieve relevant context
    RAG->>DB: Search property listings
    DB-->>RAG: Matching properties
    RAG->>OpenAI: Construct prompt with context
    OpenAI-->>API: AI response (streaming)
    API-->>Chat: "I found 5 2BR apartments in Brooklyn..."

    %% Lead Qualification Questions
    User->>Chat: "What's the price range?"
    Chat->>API: Forward message
    API->>OpenAI: Get response
    OpenAI-->>API: "Prices range from $2,500-$4,500"
    API-->>Chat: Display answer

    %% Capture Lead Info
    User->>Chat: "I want to schedule a viewing"
    Chat->>API: Request contact info
    API-->>Chat: "Please provide your email and phone"

    User->>Chat: Email + phone
    Chat->>API: Submit with conversation context
    API->>LS: POST /api/v1/leads (from chatbot)
    LS->>DB: Create lead record
    LS->>LS: Score lead (HIGH intent due to viewing request)
    LS->>DB: Update as HOT lead
    LS-->>API: Lead created (ID)
    API-->>Chat: "Great! An agent will contact you shortly"

    %% Trigger Alert
    LS->>TWILIO: WhatsApp alert to agent
    LS->>CRM: Create HubSpot contact + deal
```

---

## Scenario: Voice Call Qualification

```mermaid
sequenceDiagram
    autonumber
    participant Nurture as Nurture Engine
    participant Queue as Redis Queue
    participant Voice as Voice Service
    participant Bland as Bland.ai API
    participant LS as Lead Service
    participant Calendar as Google Calendar
    participant DB as PostgreSQL

    %% Trigger: Non-engaged HOT lead after 3 days
    Nurture->>Queue: Find HOT leads with no engagement (Day 3)
    Queue-->>Nurture: List of leads
    Nurture->>Queue: Enqueue voice call jobs

    %% Process Voice Call
    Queue->>Voice: Dequeue voice job
    Voice->>Bland: Initiate outbound call
    Bland-->>Voice: Call in progress

    note over Bland: AI Agent calls lead<br/>and asks qualifying questions

    Bland->>Voice: Webhook: Call completed
    Voice->>Voice: Parse transcript + extracted data
    Voice->>LS: Update lead with qualification data

    alt Lead qualified + wants appointment
        Voice->>Calendar: Create calendar event
        Calendar-->>Voice: Event created
        Voice->>LS: Update lead status = "qualified"
        LS->>DB: Save appointment details
        LS->>SENDGRID: Send confirmation email
    else Lead requested callback
        Voice->>Queue: Schedule callback (24 hours)
        Voice->>LS: Update lead with callback time
    else Lead not interested
        Voice->>LS: Update lead status = "lost"
        LS->>DB: Mark as lost with reason
    end
```

---

## Scenario: Content Generation and Publishing

```mermaid
sequenceDiagram
    autonumber
    participant Agent as Real Estate Agent
    participant Dashboard as Admin Dashboard
    participant CF as Content Factory
    participant OpenAI as OpenAI API
    participant HeyGen as HeyGen API
    participant Buffer as Buffer API
    participant Social as Social Platforms
    participant R2 as Cloudflare R2
    participant DB as PostgreSQL

    %% Agent Initiates Content Generation
    Agent->>Dashboard: "Generate content for 123 Main St"
    Dashboard->>CF: POST /api/v1/content/generate
    CF->>DB: Fetch property details
    DB-->>CF: Property data

    %% Generate Script
    CF->>OpenAI: Generate video script
    OpenAI-->>CF: Script content
    CF->>OpenAI: Generate social media captions
    OpenAI-->>CF: Instagram, Facebook, LinkedIn captions

    %% Generate Video (Async)
    CF->>HeyGen: Submit video generation job
    HeyGen-->>CF: Job ID (async)
    CF->>DB: Save content record (status: generating)

    note over HeyGen: Video generation in progress<br/>(2-5 minutes)

    HeyGen->>CF: Webhook: Video ready
    CF->>HeyGen: Download MP4
    HeyGen-->>CF: Video file
    CF->>R2: Upload to storage
    R2-->>CF: Public URL
    CF->>DB: Update content record (status: completed)

    %% Agent Approval
    Dashboard->>Agent: Show preview + captions
    Agent->>Dashboard: Approve + schedule
    Dashboard->>CF: POST /api/v1/content/{id}/approve

    %% Schedule Social Posts
    CF->>Buffer: Create Instagram post
    CF->>Buffer: Create Facebook post
    CF->>Buffer: Create LinkedIn post
    Buffer-->>CF: Posts scheduled
    CF->>DB: Update content (status: scheduled)

    %% Publishing (at scheduled time)
    Buffer->>Social: Publish to Instagram
    Buffer->>Social: Publish to Facebook
    Buffer->>Social: Publish to LinkedIn
    Social-->>Buffer: Engagement metrics
    Buffer->>CF: Webhook: Post published
    CF->>DB: Store engagement data
```

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
