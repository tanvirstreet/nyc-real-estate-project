# Integration Architecture — Third-Party Services

**Project**: The Autonomous Real Estate Agent
**Version**: 1.0
**Last Updated**: 2026-04-23

---

## 1. Integration Philosophy

This platform is **integration-first**. The autonomous real estate agent is only as powerful as the ecosystem it connects to. Every integration follows these principles:

1. **Idempotency**: All webhook handlers retry-safe
2. **Circuit Breakers**: Auto-disable failing integrations to prevent cascading failures
3. **Fallback Behavior**: Graceful degradation when external services are down
4. **Tenant Isolation**: Each tenant's API keys encrypted and stored separately
5. **Observability**: All integration calls logged with request/response IDs

---

## 2. Integration Map

> See diagram: [integration-map.mmd](diagrams/integration-map.mmd)

---

## 3. CRM Integration (HubSpot)

### 3.1 Purpose
- Central source of truth for contacts, deals, and pipelines
- Automate follow-up workflows via HubSpot workflows
- Bi-directional sync: Webhooks + API polling

### 3.2 Integration Type
**Bi-directional sync with webhooks**

### 3.3 Authentication
- OAuth2 for user-authorized access (recommended for multi-tenant)
- Private App Token for single-tenant MVP
- Store encrypted: `hubspot.accessToken`, `hubspot.refreshToken`

### 3.4 Key Operations

| Operation | Direction | Trigger | API Endpoint |
|---|---|---|---|
| Create contact | Platform → HubSpot | Lead capture form submit | `POST /crm/v3/objects/contacts` |
| Update contact | Platform → HubSpot | Lead profile updated | `PATCH /crm/v3/objects/contacts/:id` |
| Create deal | Platform → HubSpot | Lead classified as HOT | `POST /crm/v3/objects/deals` |
| Webhook receiver | HubSpot → Platform | Contact/deal updated in HubSpot | Webhook handler at `/api/webhooks/hubspot` |
| Sync workflows | Platform ← HubSpot | Periodic poll (15 min) | `GET /crm/v3/objects/workflows` |

### 3.5 Webhook Events
- `contact.created`: New lead from external source
- `contact.updated`: Lead data changed, sync to platform
- `deal.created`: New deal created
- `deal.stage_changed`: Pipeline stage changed, trigger nurture actions

### 3.6 Error Handling
- **429 Rate Limit**: Exponential backoff with jitter (max 30s)
- **5xx Errors**: Retry up to 3 times, then alert admin
- **Webhook Failures**: Queue for retry via BullMQ

### 3.7 Data Mapping

| Platform Field | HubSpot Field |
|---|---|
| `lead.email` | `email` |
| `lead.phone` | `phone` |
| `lead.firstName` | `firstname` |
| `lead.lastName` | `lastname` |
| `lead.score` | `lead_score` (custom property) |
| `lead.classification` | `lead_classification` (custom property) |
| `lead.source` | `lead_source` |

---

## 4. Messaging Integration (Twilio)

### 4.1 Purpose
- WhatsApp alerts for HOT leads
- SMS fallback for non-WhatsApp users
- Two-way messaging for lead qualification

### 4.2 Integration Type
**API-first with webhook callbacks**

### 4.3 Authentication
- Account SID + Auth Token stored as environment variables
- For multi-tenant: Per-tenant Twilio numbers (subaccounts)

### 4.4 Key Operations

| Operation | API Endpoint | Purpose |
|---|---|---|
| Send WhatsApp | `POST /messages.json` | Alert agent of HOT lead |
| Send SMS | `POST /messages.json` | SMS fallback for alerts |
| Receive message | Webhook at `/api/webhooks/twilio` | Process lead replies |

### 4.5 Message Templates
WhatsApp templates require pre-approval:

```
Template: hot_lead_alert
"🔥 HOT Lead Alert! New qualified lead from {source}. Name: {name}, Phone: {phone}. Reply YES to view details."
```

### 4.6 Rate Limits
- **WhatsApp**: 1 message/second per sender number
- **SMS**: 1 message/second per phone number (Twilio default)
- **Cost Budgeting**: Alert if monthly spend exceeds $50

---

## 5. Email Integration (SendGrid)

### 5.1 Purpose
- Transactional emails (lead notifications, password reset)
- Drip campaigns (nurture sequences)
- Weekly analytics reports

### 5.2 Integration Type
**API-first with webhook tracking**

### 5.3 Authentication
- API Key stored encrypted per tenant
- Single sender ID verified for platform

### 5.4 Key Operations

| Operation | API Endpoint | Purpose |
|---|---|---|
| Send email | `POST /mail/send` | Transactional emails |
| Batch send | `POST /mail/send` (batch) | Drip campaign blasts |
| Track events | Webhook at `/api/webhooks/sendgrid` | Opens, clicks, bounces |

### 5.5 Email Templates
Stored in SendGrid, referenced by template ID:

- `lead-notification`: Notify agent of new lead
- `nurture-welcome`: Welcome email for new leads
- `nurture-educational-1`: First nurture email
- `nurture-educational-2`: Second nurture email
- `weekly-report`: Weekly analytics summary

### 5.6 Bounce Handling
- **Hard Bounces**: Mark email invalid, stop sending
- **Soft Bounces**: Retry after 24 hours
- **Spam Complaints**: Immediate suppression

---

## 6. AI Voice Integration (Bland.ai / Vapi)

### 6.1 Purpose
- Automated outbound qualification calls
- Non-engaged HOT lead outreach (Day 3)
- Phone number qualification

### 6.2 Integration Type
**API-first with async webhooks**

### 6.3 Authentication
- API Key stored encrypted per tenant
- Custom voice persona per tenant (optional)

### 6.4 Key Operations

| Operation | API Endpoint | Purpose |
|---|---|---|
| Initiate call | `POST /calls` | Start outbound call |
| Get call status | `GET /calls/{callId}` | Check call outcome |
| Call completed | Webhook at `/api/webhooks/voice` | Retrieve transcript + data |

### 6.5 Call Flow
```
1. Identify: "Hi, this is [Agent Name]'s AI assistant calling about your property inquiry."
2. Qualify: Ask pre-configured questions (budget, timeline, pre-approval status)
3. Capture: Extract answers via speech-to-text
4. Route: Book calendar slot OR send text info OR request callback
5. Update: Sync qualification data to lead record
```

### 6.6 Cost Management
- Set per-tenant monthly call budget (default: $30)
- Alert when 80% budget consumed
- Auto-disable when limit reached

---

## 7. AI Video Integration (HeyGen / Synthesia)

### 7.1 Purpose
- Generate AI avatar videos from property scripts
- Scale video content production (10x vs. manual recording)
- Consistent branded content

### 7.2 Integration Type
**Async API with polling/webhooks**

### 7.3 Authentication
- API Key stored encrypted per tenant
- Custom avatar per tenant (optional add-on)

### 7.4 Key Operations

| Operation | API Endpoint | Purpose |
|---|---|---|
| Generate video | `POST /v1/videos` | Submit script for generation |
| Check status | `GET /v1/videos/{videoId}` | Poll for completion |
| Download | `GET /v1/videos/{videoId}/download` | Fetch MP4 |
| Webhook | `/api/webhooks/heygen` | Receive completion notification |

### 7.5 Generation Flow
```
1. Content Factory generates script (text)
2. Submit to HeyGen with script + avatar ID + voice ID
3. HeyGen processes (typically 2-5 minutes)
4. Webhook notifies platform: video ready
5. Download MP4 to R2 storage
6. Create content record in DB
7. Notify agent for approval
```

### 7.6 Cost Management
- Track video generation credits per tenant
- Alert at 80% credit consumption
- Prepaid credit model for SaaS tiers

---

## 8. Social Media Integration (Buffer)

### 8.1 Purpose
- Schedule and publish content to multiple platforms
- Support: Instagram, Facebook, LinkedIn, X (Twitter)
- Unified content calendar view

### 8.2 Integration Type
**OAuth2 + API**

### 8.3 Authentication
- OAuth2 flow for each platform
- Store tokens per tenant, per platform
- Refresh tokens automatically

### 8.4 Key Operations

| Operation | API Endpoint | Purpose |
|---|---|---|
| Create post | `POST /1/updates/create` | Schedule new post |
| Get profiles | `GET /1/profiles.json` | List connected accounts |
| Get scheduled | `GET /1/updates/scheduled.json` | Retrieve scheduled posts |
| Delete post | `DELETE /1/updates/{id}.json` | Cancel scheduled post |

### 8.5 Post Flow
```
1. Content Factory generates post (text + media)
2. Submit to Buffer with scheduled timestamp
3. Buffer posts to platform at scheduled time
4. Platform posts: fetch engagement metrics (likes, comments, shares)
5. Store metrics in DB for analytics
```

### 8.6 Platform-Specific Handling

| Platform | Image Aspect Ratio | Hashtag Support | Video Max Length |
|---|---|---|---|
| Instagram | 1:1 or 4:5 | ✓ | 90s |
| Facebook | 1.91:1 to 1:1 | ✓ | 240min |
| LinkedIn | 1.91:1 to 1:1 | Limited | 10min |
| X (Twitter) | 16:9 | ✓ | 140s |

---

## 9. Calendar Integration (Google Calendar)

### 9.1 Purpose
- Appointment scheduling for qualified leads
- Sync agent availability across platforms
- Prevent double-booking

### 9.2 Integration Type
**OAuth2 + API**

### 9.3 Authentication
- OAuth2 with offline access (refresh token)
- Service account for multi-tenant (recommended)

### 9.4 Key Operations

| Operation | API Endpoint | Purpose |
|---|---|---|
| Get availability | `GET /calendars/{calendarId}/events` | Find free slots |
| Create event | `POST /calendars/{calendarId}/events` | Book appointment |
| Update event | `PUT /calendars/{calendarId}/events/{eventId}` | Reschedule |
| Delete event | `DELETE /calendars/{calendarId}/events/{eventId}` | Cancel |

### 9.5 Scheduling Logic
```
1. Lead requests appointment via chat/voice
2. Fetch agent's Google Calendar for next 7 days
3. Identify available slots (exclude weekends, 9am-6pm only)
4. Present 3 options to lead
5. Lead selects slot
6. Create calendar event + send confirmation email
7. Sync event ID to lead record
```

---

## 10. Analytics Integration (Google Analytics 4)

### 10.1 Purpose
- Track website traffic and user behavior
- Measure lead conversion funnel
- Attribution analysis for marketing channels

### 10.2 Integration Type
**Client-side (gtag.js) + Server-side (Measurement Protocol)**

### 10.3 Key Events Tracked

| Event | Trigger | Parameters |
|---|---|---|
| `page_view` | Page navigation | page_title, page_location |
| `lead_submit` | Form submission | lead_source, property_id |
| `chat_initiated` | Chat widget opened | source_page |
| `chat_completed` | Chat conversation ended | message_count, lead_captured |
| `video_play` | Property video played | video_url, property_id |
| `appointment_booked` | Calendar event created | appointment_date |

### 10.4 Custom Conversions
- **Hot Lead Conversion**: Lead classified as HOT
- **Qualified Lead**: Lead completed qualification call
- **Appointment Set**: Calendar event created
- **Content Engagement**: User engaged with 3+ pieces of content

---

## 11. Integration Security

### 11.1 Credential Storage
- All API keys encrypted at rest using AES-256
- Encryption key stored in environment variable (never in code)
- Database: `tenantIntegrations` table with `encryptedCredentials` JSONB column

### 11.2 Webhook Security
- **Signature Verification**: Verify HMAC signatures for all webhooks
- **IP Whitelisting**: Only allow requests from known service IPs (where available)
- **Rate Limiting**: Per-IP rate limits on webhook endpoints

### 11.3 OAuth2 Token Management
- Access tokens stored with expiry timestamp
- Background job refreshes tokens before expiry
- Automatic token rotation on 401 Unauthorized responses

---

## 12. Integration Monitoring

### 12.1 Health Checks
Every integration runs a health check every 5 minutes:

```typescript
interface IntegrationHealth {
  service: 'hubspot' | 'twilio' | 'sendgrid' | 'blandai' | 'heygen' | 'buffer';
  status: 'healthy' | 'degraded' | 'down';
  lastCheck: Date;
  responseTime: number; // ms
  errorCount: number;
}
```

### 12.2 Alerts
- **Integration Down**: 3 consecutive failed health checks
- **Rate Limit Imminent**: 80% of quota consumed
- **Cost Overrun**: Monthly spend exceeds threshold
- **Webhook Failing**: >5% webhook delivery failure rate

### 12.3 Logging
All integration calls logged with:
- Request ID (UUID)
- Tenant ID
- Integration service
- Operation
- Request/response timestamp
- Status code
- Error details (if failed)

---

## 13. Integration Rollback Strategy

If an integration fails catastrophically:

1. **Circuit Breaker Trips**: Auto-disable integration
2. **Fallback Mode**: Platform continues with degraded functionality
   - HubSpot down: Store leads locally, sync when back
   - Twilio down: Queue messages, send when back
   - SendGrid down: Queue emails, retry in 1 hour
3. **Admin Alert**: Email + Slack notification
4. **Retry Strategy**: Exponential backoff with max 1 hour between attempts
5. **Manual Recovery**: Admin can force re-enable after issue resolved

---

## 14. Future Integrations (Post-MVP)

| Service | Purpose | Priority |
|---|---|---|
| **Zapier** | No-code integration for 5000+ apps | High (multi-tenant) |
| **Stripe** | Subscription billing for SaaS tiers | High (SaaS launch) |
| **Zillow/StreetEasy** | Direct MLS listing sync | Medium |
| **Intercom** | In-app chat and support | Medium |
| **Slack** | Team notifications and updates | Low |
| **Notion** | Documentation and knowledge base sync | Low |
