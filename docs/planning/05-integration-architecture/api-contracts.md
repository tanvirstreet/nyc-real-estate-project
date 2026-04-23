---
layout: page
title: API Contracts
---

# API Contracts — Interface Definitions

**Project**: The Autonomous Real Estate Agent
**Version**: 1.0
**Last Updated**: 2026-04-23

---

## 1. Contract Philosophy

All API contracts in this platform follow these principles:

1. **OpenAPI-First**: Every endpoint has OpenAPI spec (future: code generation)
2. **Type-Safe**: TypeScript types derived from Zod schemas
3. **Versioned**: `/api/v1/` prefix for forward compatibility
4. **Tenant-Scoped**: All requests include `tenantId` header or derived from subdomain
5. **Idempotent**: Safe retry on network failures

---

## 2. API Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   CLIENT LAYER                          │
│  Browser │ Mobile App │ External Webhooks               │
├─────────────────────────────────────────────────────────┤
│                   API GATEWAY                           │
│  Edge Middleware: Auth, Tenant Resolution, Rate Limit   │
├─────────────────────────────────────────────────────────┤
│                   REST APIS                             │
│  /api/v1/leads    /api/v1/chatbot    /api/v1/content   │
│  /api/v1/crm      /api/v1/nurture    /api/v1/webhooks  │
├─────────────────────────────────────────────────────────┤
│                   SERVICE LAYER                         │
│  Lead Service │ Chatbot Service │ Content Factory       │
├─────────────────────────────────────────────────────────┤
│                   INTEGRATION LAYER                     │
│  HubSpot │ Twilio │ SendGrid │ Bland.ai │ HeyGen       │
└─────────────────────────────────────────────────────────┘
```

---

## 3. Public APIs (Client-Facing)

### 3.1 Lead Capture API

#### `POST /api/v1/leads`

Create a new lead from form submission or chatbot.

**Request:**
```json
{
  "source": "landing_page" | "chatbot" | "voice_call" | "manual",
  "propertyId": "string (uuid, optional)",
  "contact": {
    "firstName": "string",
    "lastName": "string",
    "email": "string (email)",
    "phone": "string (e164 format)",
    "preferredContact": "email" | "whatsapp" | "sms"
  },
  "interests": {
    "propertyType": "apartment" | "house" | "condo" | "townhouse",
    "budgetRange": {
      "min": "number",
      "max": "number"
    },
    "neighborhoods": ["string"],
    "bedrooms": "number (optional)",
    "timeline": "immediate" | "1-3 months" | "3-6 months" | "6+ months"
  },
  "utm": {
    "source": "string (optional)",
    "medium": "string (optional)",
    "campaign": "string (optional)"
  }
}
```

**Response (201):**
```json
{
  "id": "uuid",
  "status": "new" | "contacted" | "qualified" | "hot" | "converted" | "lost",
  "score": "number (0-100)",
  "classification": "cold" | "warm" | "hot",
  "createdAt": "iso8601",
  "contact": { /* ... */ },
  "interests": { /* ... */ }
}
```

**Error Responses:**
- `400 Bad Request`: Invalid email/phone, missing required fields
- `409 Conflict`: Duplicate lead (email + phone match)
- `422 Unprocessable Entity`: Validation failure

---

#### `GET /api/v1/leads/{id}`

Retrieve lead details (agent-only, authenticated).

**Response (200):**
```json
{
  "id": "uuid",
  "status": "string",
  "score": "number",
  "classification": "string",
  "createdAt": "iso8601",
  "updatedAt": "iso8601",
  "contact": { /* ... */ },
  "interests": { /* ... */ },
  "crm": {
    "hubSpotContactId": "string (optional)",
    "hubSpotDealId": "string (optional)"
  },
  "engagement": {
    "lastContactedAt": "iso8601 (nullable)",
    "emailsSent": "number",
    "smsSent": "number",
    "callsMade": "number",
    "chatSessions": "number"
  },
  "nurture": {
    "currentSequence": "string (nullable)",
    "sequenceStep": "number (nullable)",
    "pausedAt": "iso8601 (nullable)"
  }
}
```

**Error Responses:**
- `401 Unauthorized`: Missing/invalid auth token
- `403 Forbidden`: Lead belongs to different tenant
- `404 Not Found`: Lead ID doesn't exist

---

#### `GET /api/v1/leads`

List and filter leads (agent-only, authenticated).

**Query Params:**
- `status`: Filter by status (comma-separated)
- `classification`: Filter by classification
- `source`: Filter by lead source
- `sortBy`: `createdAt` | `score` | `updatedAt`
- `sortOrder`: `asc` | `desc`
- `page`: Page number (default: 1)
- `limit`: Results per page (default: 20, max: 100)

**Response (200):**
```json
{
  "leads": [ /* Lead objects */ ],
  "pagination": {
    "total": "number",
    "page": "number",
    "limit": "number",
    "totalPages": "number"
  }
}
```

---

### 3.2 Chatbot API

#### `POST /api/v1/chatbot/message`

Send a message to the chatbot and receive AI response.

**Request:**
```json
{
  "sessionId": "string (uuid, optional for new chat)",
  "message": "string",
  "context": {
    "propertyId": "string (uuid, optional)",
    "previousContext": "object (optional)"
  }
}
```

**Response (200):**
```json
{
  "sessionId": "uuid",
  "message": "string",
  "leadCaptured": "boolean",
  "leadId": "uuid (nullable)",
  "suggestedActions": [
    {
      "type": "property_request" | "contact_request" | "appointment_request",
      "label": "string",
      "data": "object"
    }
  ],
  "sources": [
    {
      "title": "string",
      "url": "string"
    }
  ]
}
```

**Streaming Response (future):**
Server-Sent Events (SSE) for real-time streaming.

---

#### `GET /api/v1/chatbot/history/{sessionId}`

Retrieve chat history (for context window reconstruction).

**Response (200):**
```json
{
  "sessionId": "uuid",
  "messages": [
    {
      "role": "user" | "assistant",
      "content": "string",
      "timestamp": "iso8601"
    }
  ],
  "leadCaptured": "boolean",
  "leadId": "uuid (nullable)"
}
```

---

### 3.3 Content API

#### `POST /api/v1/content/generate`

Generate content from property data or URL.

**Request:**
```json
{
  "type": "video_script" | "social_post" | "property_description",
  "source": {
    "propertyUrl": "string (optional)",
    "propertyData": "object (optional)"
  },
  "options": {
    "platform": "instagram" | "facebook" | "linkedin" | "twitter",
    "tone": "professional" | "casual" | "urgent",
    "includeCallToAction": "boolean",
    "hashtags": "boolean"
  }
}
```

**Response (202):**
```json
{
  "id": "uuid",
  "status": "generating",
  "estimatedTimeSeconds": "number",
  "webhookUrl": "string (for async completion)"
}
```

**Error Responses:**
- `400 Bad Request`: Invalid property URL or data
- `429 Too Many Requests`: Rate limit exceeded (10 generations/hour)

---

#### `GET /api/v1/content/{id}`

Retrieve generated content.

**Response (200):**
```json
{
  "id": "uuid",
  "type": "string",
  "status": "generating" | "completed" | "failed",
  "content": {
    "title": "string",
    "body": "string",
    "mediaUrls": ["string"],
    "metadata": "object"
  },
  "propertyId": "uuid (nullable)",
  "createdAt": "iso8601",
  "generatedAt": "iso8601 (nullable)"
}
```

---

#### `POST /api/v1/content/{id}/approve`

Approve content for publishing.

**Request:**
```json
{
  "scheduledAt": "iso8601 (optional, immediate if null)",
  "platforms": ["instagram", "facebook", "linkedin"]
}
```

**Response (200):**
```json
{
  "id": "uuid",
  "status": "scheduled" | "published",
  "scheduledAt": "iso8601",
  "publishedAt": "iso8601 (nullable)",
  "platforms": ["string"]
}
```

---

### 3.4 Property API

#### `GET /api/v1/properties`

List available properties.

**Query Params:**
- `neighborhood`: Filter by neighborhood
- `minPrice`, `maxPrice`: Price range
- `bedrooms`, `bathrooms`: Filter by count
- `propertyType`: Filter by type
- `page`, `limit`: Pagination

**Response (200):**
```json
{
  "properties": [
    {
      "id": "uuid",
      "title": "string",
      "address": {
        "street": "string",
        "neighborhood": "string",
        "city": "string",
        "state": "string",
        "zip": "string"
      },
      "details": {
        "price": "number",
        "bedrooms": "number",
        "bathrooms": "number",
        "sqft": "number",
        "propertyType": "string"
      },
      "media": {
        "images": ["string (url)"],
        "virtualTourUrl": "string (nullable)",
        "videoUrl": "string (nullable)"
      },
      "features": ["string"],
      "description": "string",
      "listedAt": "iso8601"
    }
  ],
  "pagination": { /* ... */ }
}
```

---

#### `GET /api/v1/properties/{id}`

Retrieve single property details.

**Response (200):**
```json
{
  "id": "uuid",
  "title": "string",
  "address": { /* ... */ },
  "details": { /* ... */ },
  "media": { /* ... */ },
  "features": ["string"],
  "description": "string",
  "nearby": {
    "schools": ["string"],
    "transit": ["string"],
    "parks": ["string"]
  },
  "analytics": {
    "views": "number",
    "inquiries": "number",
    "lastInquiryAt": "iso8601 (nullable)"
  },
  "listedAt": "iso8601",
  "updatedAt": "iso8601"
}
```

---

## 4. Webhook APIs (External Services)

### 4.1 HubSpot Webhook Handler

#### `POST /api/webhooks/hubspot`

Receive webhook events from HubSpot.

**Request Headers:**
- `X-HubSpot-Signature`: HMAC signature verification
- `X-HubSpot-Request-Timestamp`: Request timestamp

**Request Body:**
```json
{
  "eventId": "string",
  "subscriptionType": "contact.creation" | "contact.deletion" | "contact.propertyChange" | "deal.creation" | "deal.propertyChange",
  "objectId": "number",
  "propertyName": "string (for propertyChange events)",
  "changeSource": "string",
  "sourceId": "string",
  "occurredAt": "iso8601"
}
```

**Response (200):**
```json
{
  "received": "boolean"
}
```

**Event Processing:**
- Events added to queue for async processing
- Signature verified before queuing
- Response sent immediately (acknowledgment only)

---

### 4.2 Twilio Webhook Handler

#### `POST /api/webhooks/twilio/sms`

Receive incoming SMS messages.

**Request Body:** Form-encoded
- `From`: Sender phone number
- `To`: Receiver phone number (Twilio number)
- `Body`: Message content
- `MessageSid`: Message ID

**Response:** XML or JSON
```xml
<Response>
  <Message>Thank you! Your response has been recorded.</Message>
</Response>
```

**Processing:**
- Match sender phone number to lead record
- Parse message content
- Extract intent (yes/no/question)
- Update lead record
- Trigger appropriate follow-up action

---

#### `POST /api/webhooks/twilio/voice-status`

Receive voice call status updates.

**Request Body:** Form-encoded
- `CallSid`: Call ID
- `CallStatus`: `queued` | `ringing` | `in-progress` | `completed` | `failed` | `busy` | `no-answer`
- `RecordingUrl`: URL to call recording (if completed)

**Response:** 200 OK

---

### 4.3 SendGrid Webhook Handler

#### `POST /api/webhooks/sendgrid/events`

Receive email engagement events.

**Request Body:**
```json
[
  {
    "email": "string",
    "event": "processed" | "delivered" | "open" | "click" | "bounce" | "dropped" | "spamreport",
    "timestamp": "number (unix)",
    "sg_message_id": "string",
    "url": "string (for click events)"
  }
]
```

**Response (200):**
```json
{
  "received": "boolean",
  "processedCount": "number"
}
```

**Event Processing:**
- Update lead engagement metrics
- Increment open/click counts
- Mark email invalid on bounce
- Trigger nurture sequence step advancement

---

### 4.4 Voice AI Webhook Handler

#### `POST /api/webhooks/voice/completed`

Receive completed call data from Bland.ai/Vapi.

**Request Body:**
```json
{
  "callId": "string",
  "status": "completed" | "failed" | "no-answer",
  "duration": "number (seconds)",
  "transcript": "string",
  "recordingUrl": "string (url)",
  "extractedData": {
    "budget": "string (nullable)",
    "timeline": "string (nullable)",
    "preApproved": "boolean (nullable)",
    "preferredContact": "string (nullable)"
  },
  "outcome": "booked_appointment" | "requested_info" | "requested_callback" | "not_interested"
}
```

**Response (200):**
```json
{
  "received": "boolean"
}
```

**Processing:**
- Update lead qualification data
- Create calendar event if outcome is `booked_appointment`
- Send follow-up SMS/email if outcome is `requested_info`
- Schedule callback if outcome is `requested_callback`
- Mark as lost if outcome is `not_interested`

---

### 4.5 AI Video Webhook Handler

#### `POST /api/webhooks/video/completed`

Receive video generation completion from HeyGen.

**Request Body:**
```json
{
  "videoId": "string",
  "status": "completed" | "failed",
  "videoUrl": "string (url)",
  "thumbnailUrl": "string (url)",
  "duration": "number (seconds)",
  "error": "string (if failed)"
}
```

**Response (200):**
```json
{
  "received": "boolean"
}
```

**Processing:**
- Download video to R2 storage
- Update content record status
- Notify agent for approval
- Schedule social media posts (if pre-configured)

---

## 5. Internal APIs (Service-to-Service)

### 5.1 Lead Scoring Service

#### `POST /internal/score-lead`

Calculate lead score based on attributes and engagement.

**Request:**
```json
{
  "leadId": "uuid",
  "attributes": {
    "budgetMatch": "number (0-1)",
    "timelineUrgency": "number (0-1)",
    "engagementLevel": "number (0-1)",
    "contactInfoCompleteness": "number (0-1)"
  }
}
```

**Response (200):**
```json
{
  "leadId": "uuid",
  "score": "number (0-100)",
  "classification": "cold" | "warm" | "hot",
  "reasons": ["string"],
  "nextAction": "string"
}
```

---

### 5.2 CRM Sync Service

#### `POST /internal/crm/sync-contact`

Sync contact to HubSpot.

**Request:**
```json
{
  "leadId": "uuid",
  "operation": "create" | "update",
  "contact": {
    "email": "string",
    "phone": "string",
    "firstName": "string",
    "lastName": "string"
  }
}
```

**Response (202):**
```json
{
  "jobId": "uuid",
  "status": "queued"
}
```

---

## 6. Common Response Patterns

### 6.1 Standard Error Response

```json
{
  "error": {
    "code": "LEAD_NOT_FOUND" | "VALIDATION_ERROR" | "RATE_LIMIT_EXCEEDED" | "INTERNAL_ERROR",
    "message": "Human-readable error message",
    "details": "object (optional, additional context)",
    "requestId": "uuid (for support)"
  }
}
```

### 6.2 Validation Error Response

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": {
      "fields": [
        {
          "field": "contact.email",
          "message": "Invalid email format"
        },
        {
          "field": "interests.budgetRange.min",
          "message": "Must be greater than 0"
        }
      ]
    }
  }
}
```

---

## 7. Rate Limiting

| Endpoint | Limit | Window |
|---|---|---|
| `POST /api/v1/leads` | 10 | 1 hour per IP |
| `POST /api/v1/chatbot/message` | 60 | 1 minute per session |
| `POST /api/v1/content/generate` | 10 | 1 hour per tenant |
| `GET /api/v1/*` | 100 | 1 minute per tenant |

**Rate Limit Response (429):**
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests",
    "details": {
      "limit": 10,
      "remaining": 0,
      "resetAt": "iso8601"
    }
  }
}
```

---

## 8. API Versioning Strategy

- **URL Versioning**: `/api/v1/`, `/api/v2/` (future)
- **Backward Compatibility**: v1 remains functional for 6 months after v2 launch
- **Deprecation Warnings**: Include `X-API-Deprecation` header for deprecated endpoints
- **Sunset Timeline**: Announced 90 days before removal

---

## 9. Authentication

### 9.1 Agent Authentication (Session-Based)

**Request:**
```
POST /api/auth/signin
```

**Request Body:**
```json
{
  "email": "string",
  "password": "string"
}
```

**Response (200):**
```json
{
  "sessionToken": "string (jwt)",
  "expiresAt": "iso8601",
  "user": {
    "id": "uuid",
    "name": "string",
    "email": "string",
    "role": "SUPER_ADMIN" | "AGENT_ADMIN" | "AGENT"
  }
}
```

**Subsequent Requests:**
Include header: `Authorization: Bearer {sessionToken}`

---

### 9.2 Public Client Authentication (API Key)

For public clients (landing page, chatbot widget):

**Request:**
```
GET /api/v1/properties
X-API-Key: {public_api_key}
```

API keys are scoped per tenant and rate-limited separately.

---

## 10. OpenAPI Spec Location

Full OpenAPI 3.0 specification available at:

**Production**: `https://api.platform.com/api/openapi.json`
**Development**: `http://localhost:3000/api/openapi.json`

Interactive documentation (Swagger UI): `/api/docs`
