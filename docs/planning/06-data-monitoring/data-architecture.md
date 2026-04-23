---
layout: page
title: Data Architecture
---

# Data Architecture — Models and Strategy

**Project**: The Autonomous Real Estate Agent
**Version**: 1.0
**Last Updated**: 2026-04-23

---

## 1. Data Philosophy

This platform follows these data principles:

1. **Tenant Isolation**: All data scoped by `tenantId` with row-level security
2. **Immutable Audit Trail**: Critical changes logged with timestamps and user context
3. **Privacy-First**: PII encrypted, GDPR-compliant, right to deletion
4. **Analytics-Ready**: Structured for easy BI and reporting queries
5. **SaaS Scalable**: Schema supports 10K+ tenants without degradation

---

## 2. Database Schema Overview

> See diagram: [db-schema.mmd](diagrams/db-schema.mmd)

**Technology**: PostgreSQL 15+ (via Supabase)
**Extensions**: `pgvector` (vector similarity), `pg_trgm` (fuzzy text search)

---

## 3. Core Tables

### 3.1 Tenants

Multi-tenancy root table. All other tables reference this.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Unique tenant identifier |
| `name` | VARCHAR(255) | NOT NULL | Display name (agent/brokerage name) |
| `slug` | VARCHAR(50) | UNIQUE, NOT NULL | Subdomain slug (e.g., `agent1.platform.com`) |
| `status` | ENUM | NOT NULL | `ACTIVE`, `SUSPENDED`, `TRIAL`, `CANCELLED` |
| `plan` | ENUM | NOT NULL | `FREE`, `STARTER`, `PRO`, `ENTERPRISE` |
| `settings` | JSONB | DEFAULT {} | Tenant-specific settings (branding, preferences) |
| `billingEmail` | VARCHAR(255) | NOT NULL | Billing contact email |
| `createdAt` | TIMESTAMPTZ | DEFAULT NOW() | Tenant creation timestamp |
| `updatedAt` | TIMESTAMPTZ | DEFAULT NOW() | Last update timestamp |

**Indexes**:
- `idx_tenants_slug` (UNIQUE)
- `idx_tenants_status` (for querying active tenants)

---

### 3.2 Users

Platform user accounts (agents, admins).

| Column | Type | Constraints | Description |
|---|---|---|
| `id` | UUID | PK | User identifier |
| `tenantId` | UUID | FK → tenants.id, NOT NULL | Tenant association |
| `email` | VARCHAR(255) | UNIQUE, NOT NULL | Login email |
| `name` | VARCHAR(255) | NOT NULL | Display name |
| `role` | ENUM | NOT NULL | `SUPER_ADMIN`, `AGENT_ADMIN`, `AGENT` |
| `passwordHash` | VARCHAR(255) | NOT NULL | Bcrypt hash |
| `lastLoginAt` | TIMESTAMPTZ | NULLABLE | Last login timestamp |
| `isActive` | BOOLEAN | DEFAULT TRUE | Account active flag |
| `createdAt` | TIMESTAMPTZ | DEFAULT NOW() | Creation timestamp |
| `updatedAt` | TIMESTAMPTZ | DEFAULT NOW() | Last update timestamp |

**Indexes**:
- `idx_users_email` (UNIQUE)
- `idx_users_tenantId` (for tenant queries)
- `idx_users_role` (for role-based queries)

**Row-Level Security**:
- Policy: Users can only see users from their tenant
- Exception: `SUPER_ADMIN` can see all users

---

### 3.3 Leads

Core entity: captured leads from all sources.

| Column | Type | Constraints | Description |
|---|---|---|
| `id` | UUID | PK | Lead identifier |
| `tenantId` | UUID | FK → tenants.id, NOT NULL | Tenant association |
| `status` | ENUM | NOT NULL | `NEW`, `CONTACTED`, `QUALIFIED`, `HOT`, `CONVERTED`, `LOST` |
| `score` | INTEGER | CHECK (0-100), DEFAULT 0 | Lead score (0-100) |
| `classification` | ENUM | NOT NULL | `COLD`, `WARM`, `HOT` |
| `source` | ENUM | NOT NULL | `LANDING_PAGE`, `CHATBOT`, `VOICE_CALL`, `MANUAL`, `REFERRAL` |
| `firstName` | VARCHAR(100) | NOT NULL | Lead first name |
| `lastName` | VARCHAR(100) | NOT NULL | Lead last name |
| `email` | VARCHAR(255) | NOT NULL | Lead email (indexed for lookup) |
| `phone` | VARCHAR(20) | NOT NULL | Phone (E.164 format) |
| `preferredContact` | ENUM | NOT NULL | `EMAIL`, `WHATSAPP`, `SMS` |
| `propertyInterest` | JSONB | DEFAULT {} | Property preferences (budget, neighborhoods, etc.) |
| `utm` | JSONB | DEFAULT {} | UTM tracking parameters |
| `crmSync` | JSONB | DEFAULT {} | CRM integration data (HubSpot IDs, etc.) |
| `lastContactedAt` | TIMESTAMPTZ | NULLABLE | Last agent contact timestamp |
| `createdAt` | TIMESTAMPTZ | DEFAULT NOW() | Lead creation timestamp |
| `updatedAt` | TIMESTAMPTZ | DEFAULT NOW() | Last update timestamp |

**Indexes**:
- `idx_leads_tenantId_status` (composite, for filtering)
- `idx_leads_email` (for duplicate detection)
- `idx_leads_phone` (for duplicate detection)
- `idx_leads_score` (for sorting by score)
- `idx_leads_classification` (for filtering HOT/WARM/COLD)
- `idx_leads_createdAt` (for date-range queries)
- `idx_leads_propertyInterest` (GIN, for JSONB queries)

**Row-Level Security**:
- Policy: Tenant isolation enforced (users see only their tenant's leads)

**Data Retention**:
- Active leads: Retain forever
- Lost leads: Archive after 2 years
- Converted leads: Retain forever for attribution

---

### 3.4 LeadEngagements

Track all lead touchpoints and engagement signals.

| Column | Type | Constraints | Description |
|---|---|---|
| `id` | UUID | PK | Engagement identifier |
| `leadId` | UUID | FK → leads.id, NOT NULL | Associated lead |
| `type` | ENUM | NOT NULL | `EMAIL_SENT`, `EMAIL_OPENED`, `EMAIL_CLICKED`, `SMS_SENT`, `WHATSAPP_SENT`, `CHAT_MESSAGE`, `CALL_COMPLETED`, `PROPERTY_VIEWED`, `CONTENT_VIEWED` |
| `channel` | ENUM | NOT NULL | `EMAIL`, `SMS`, `WHATSAPP`, `CHAT`, `PHONE`, `WEB` |
| `metadata` | JSONB | DEFAULT {} | Engagement-specific data (link URL, video duration, etc.) |
| `createdAt` | TIMESTAMPTZ | DEFAULT NOW() | Engagement timestamp |

**Indexes**:
- `idx_engagements_leadId` (for lead engagement history)
- `idx_engagements_type` (for analytics aggregation)
- `idx_engagements_createdAt` (for time-series queries)

**Analytics Use Cases**:
- Lead health score (engagement frequency)
- Nurture sequence advancement
- Attribution modeling (which touchpoint led to conversion)

---

### 3.5 NurtureSequences

Drip campaign definitions.

| Column | Type | Constraints | Description |
|---|---|---|
| `id` | UUID | PK | Sequence identifier |
| `tenantId` | UUID | FK → tenants.id, NOT NULL | Tenant association |
| `name` | VARCHAR(255) | NOT NULL | Sequence name |
| `description` | TEXT | NULLABLE | Sequence description |
| `triggerClassification` | ENUM | NOT NULL | Which lead classification triggers this (`COLD`, `WARM`, `ALL`) |
| `isActive` | BOOLEAN | DEFAULT TRUE | Sequence active flag |
| `steps` | JSONB | NOT NULL | Array of step objects (see below) |
| `createdAt` | TIMESTAMPTZ | DEFAULT NOW() | Creation timestamp |
| `updatedAt` | TIMESTAMPTZ | DEFAULT NOW() | Last update timestamp |

**Steps Schema (JSONB)**:
```json
[
  {
    "order": 1,
    "delay": 0,
    "delayUnit": "hours",
    "type": "email",
    "templateId": "nurture-welcome",
    "subject": "Welcome to our community!"
  },
  {
    "order": 2,
    "delay": 3,
    "delayUnit": "days",
    "type": "email",
    "templateId": "nurture-educational-1",
    "subject": "5 things to know before buying in NYC"
  }
]
```

---

### 3.6 NurtureEnrollments

Track leads enrolled in nurture sequences.

| Column | Type | Constraints | Description |
|---|---|---|
| `id` | UUID | PK | Enrollment identifier |
| `leadId` | UUID | FK → leads.id, NOT NULL, UNIQUE | Associated lead |
| `sequenceId` | UUID | FK → nurtureSequences.id, NOT NULL | Sequence |
| `currentStep` | INTEGER | DEFAULT 1 | Current step in sequence |
| `pausedAt` | TIMESTAMPTZ | NULLABLE | Pause timestamp (NULL if active) |
| `completedAt` | TIMESTAMPTZ | NULLABLE | Completion timestamp |
| `enrolledAt` | TIMESTAMPTZ | DEFAULT NOW() | Enrollment timestamp |
| `nextStepAt` | TIMESTAMPTZ | NOT NULL | When to send next step |

**Indexes**:
- `idx_nurture_enrollments_leadId` (for lead status queries)
- `idx_nurture_enrollments_nextStepAt` (for cron job processing)
- `idx_nurture_enrollments_sequenceId` (for sequence analytics)

---

### 3.7 Properties

Real estate listings (synced from MLS or manually added).

| Column | Type | Constraints | Description |
|---|---|---|
| `id` | UUID | PK | Property identifier |
| `tenantId` | UUID | FK → tenants.id, NOT NULL | Tenant association |
| `mlsId` | VARCHAR(100) | UNIQUE, NULLABLE | MLS listing ID (if applicable) |
| `title` | VARCHAR(255) | NOT NULL | Property title |
| `address` | JSONB | NOT NULL | Full address object |
| `details` | JSONB | NOT NULL | Property details (price, beds, baths, sqft, type) |
| `description` | TEXT | NULLABLE | Property description |
| `features` | TEXT[] | DEFAULT [] | Property features array |
| `media` | JSONB | DEFAULT {} | Media URLs (images, videos, virtual tour) |
| `nearby` | JSONB | DEFAULT {} | Nearby amenities (schools, transit, parks) |
| `status` | ENUM | NOT NULL | `ACTIVE`, `PENDING`, `SOLD`, `RENTED` |
| `listedAt` | TIMESTAMPTZ | DEFAULT NOW() | Listing date |
| `updatedAt` | TIMESTAMPTZ | DEFAULT NOW() | Last update timestamp |

**Indexes**:
- `idx_properties_mlsId` (UNIQUE)
- `idx_properties_tenantId_status` (composite, for filtering)
- `idx_properties_details` (GIN, for JSONB queries like price range, bedrooms)
- `idx_properties_address` (GIN, for location search)

**Row-Level Security**:
- Policy: Tenants see only their properties
- Exception: Public-facing APIs expose properties without tenant ID

---

### 3.8 Content

Generated content (scripts, videos, social posts).

| Column | Type | Constraints | Description |
|---|---|---|
| `id` | UUID | PK | Content identifier |
| `tenantId` | UUID | FK → tenants.id, NOT NULL | Tenant association |
| `propertyId` | UUID | FK → properties.id, NULLABLE | Associated property (nullable for generic content) |
| `type` | ENUM | NOT NULL | `VIDEO_SCRIPT`, `SOCIAL_POST`, `PROPERTY_DESCRIPTION` |
| `status` | ENUM | NOT NULL | `GENERATING`, `COMPLETED`, `FAILED`, `APPROVED`, `SCHEDULED`, `PUBLISHED` |
| `content` | JSONB | DEFAULT {} | Content data (title, body, media URLs) |
| `generatedBy` | VARCHAR(100) | NULLABLE | Generation method (AI template ID, manual) |
| `approvedAt` | TIMESTAMPTZ | NULLABLE | Approval timestamp |
| `approvedBy` | UUID | FK → users.id, NULLABLE | User who approved |
| `scheduledAt` | TIMESTAMPTZ | NULLABLE | Scheduled publishing timestamp |
| `publishedAt` | TIMESTAMPTZ | NULLABLE | Actual publish timestamp |
| `platforms` | VARCHAR[] | DEFAULT [] | Target platforms (instagram, facebook, linkedin) |
| `engagement` | JSONB | DEFAULT {} | Post-engagement metrics (likes, comments, shares) |
| `createdAt` | TIMESTAMPTZ | DEFAULT NOW() | Creation timestamp |
| `updatedAt` | TIMESTAMPTZ | DEFAULT NOW() | Last update timestamp |

**Indexes**:
- `idx_content_tenantId_status` (composite, for filtering)
- `idx_content_propertyId` (for property-related content)
- `idx_content_scheduledAt` (for upcoming scheduled content)
- `idx_content_type` (for type filtering)

---

### 3.9 ChatbotSessions

Chatbot conversation sessions.

| Column | Type | Constraints | Description |
|---|---|---|
| `id` | UUID | PK | Session identifier |
| `tenantId` | UUID | FK → tenants.id, NOT NULL | Tenant association |
| `leadId` | UUID | FK → leads.id, NULLABLE | Associated lead (NULL until captured) |
| `messages` | JSONB | DEFAULT [] | Array of message objects |
| `propertyInterest` | JSONB | DEFAULT {} | Detected property preferences |
| `leadCaptured` | BOOLEAN | DEFAULT FALSE | Whether lead info was captured |
| `endedAt` | TIMESTAMPTZ | NULLABLE | Session end timestamp |
| `createdAt` | TIMESTAMPTZ | DEFAULT NOW() | Session start timestamp |

**Messages Schema (JSONB)**:
```json
[
  {
    "role": "user",
    "content": "I'm looking for a 2BR in Brooklyn",
    "timestamp": "2026-04-23T10:30:00Z"
  },
  {
    "role": "assistant",
    "content": "I found 5 matching properties...",
    "timestamp": "2026-04-23T10:30:02Z"
  }
]
```

**Indexes**:
- `idx_chatbot_sessions_tenantId` (for tenant queries)
- `idx_chatbot_sessions_leadId` (for lead conversation history)
- `idx_chatbot_sessions_createdAt` (for analytics)

**Data Retention**:
- Active sessions: Retain for 90 days
- Sessions with captured leads: Retain forever

---

### 3.10 Integrations

Third-party integration credentials and status.

| Column | Type | Constraints | Description |
|---|---|---|
| `id` | UUID | PK | Integration identifier |
| `tenantId` | UUID | FK → tenants.id, NOT NULL | Tenant association |
| `service` | ENUM | NOT NULL | `HUBSPOT`, `TWILIO`, `SENDGRID`, `BLAND_AI`, `HEYGEN`, `BUFFER`, `GOOGLE_CALENDAR` |
| `status` | ENUM | NOT NULL | `ACTIVE`, `DISABLED`, `ERROR` |
| `credentials` | BYTEA | NOT NULL | Encrypted API credentials |
| `lastHealthCheck` | TIMESTAMPTZ | NULLABLE | Last health check timestamp |
| `lastError` | TEXT | NULLABLE | Last error message |
| `configuration` | JSONB | DEFAULT {} | Service-specific config |
| `createdAt` | TIMESTAMPTZ | DEFAULT NOW() | Connection timestamp |
| `updatedAt` | TIMESTAMPTZ | DEFAULT NOW() | Last update timestamp |

**Security**:
- `credentials` column encrypted at rest using AES-256
- Encryption key stored in environment variable (never in DB)
- Only decrypted in-memory when needed for API calls

**Indexes**:
- `idx_integrations_tenantId_service` (composite UNIQUE)

---

### 3.11 AuditLog

Immutable audit trail for compliance and debugging.

| Column | Type | Constraints | Description |
|---|---|---|
| `id` | UUID | PK | Log entry identifier |
| `tenantId` | UUID | FK → tenants.id, NOT NULL | Tenant association |
| `userId` | UUID | FK → users.id, NULLABLE | Acting user (NULL for system actions) |
| `action` | VARCHAR(100) | NOT NULL | Action performed |
| `entityType` | VARCHAR(50) | NOT NULL | Entity type (lead, property, user, etc.) |
| `entityId` | UUID | NOT NULL | Entity identifier |
| `changes` | JSONB | DEFAULT {} | Before/after values |
| `ipAddress` | INET | NULLABLE | Client IP address |
| `userAgent` | TEXT | NULLABLE | Client user agent |
| `createdAt` | TIMESTAMPTZ | DEFAULT NOW() | Log timestamp |

**Indexes**:
- `idx_audit_tenantId` (for tenant audit queries)
- `idx_audit_userId` (for user activity tracking)
- `idx_audit_entity` (composite entityType + entityId)
- `idx_audit_createdAt` (for time-range queries)

**Data Retention**:
- General audit logs: Retain for 1 year
- Security-related logs: Retain for 7 years

---

## 4. Vector Data (RAG)

### 4.1 PropertyEmbeddings

Vector embeddings for semantic property search.

| Column | Type | Constraints | Description |
|---|---|---|
| `id` | UUID | PK | Embedding identifier |
| `propertyId` | UUID | FK → properties.id, NOT NULL | Associated property |
| `chunk` | TEXT | NOT NULL | Text chunk (description + features) |
| `embedding` | VECTOR(1536) | NOT NULL | OpenAI embedding vector (text-embedding-3-small) |
| `createdAt` | TIMESTAMPTZ | DEFAULT NOW() | Creation timestamp |

**Indexes**:
- `idx_embeddings_propertyId` (for property lookups)
- `idx_embeddings_vector` (HNSW index, for similarity search)

**Usage**:
- Semantic property search ("modern apartments with natural light")
- RAG context retrieval for chatbot

---

## 5. Data Relationships

```
tenants (1) ──< (N) users
tenants (1) ──< (N) leads
tenants (1) ──< (N) properties
tenants (1) ──< (N) content
tenants (1) ──< (N) nurtureSequences
tenants (1) ──< (N) integrations

leads (1) ──< (N) leadEngagements
leads (1) ──< (N) nurtureEnrollments
leads (1) ──< (N) chatbotSessions

properties (1) ──< (N) content
properties (1) ──< (N) propertyEmbeddings
```

---

## 6. Database Migrations Strategy

**Tool**: Prisma Migrate

**Migration Workflow**:
1. Create migration: `npx prisma migrate dev --name add_lead_source`
2. Review generated SQL
3. Apply to local database
4. Test with seed data
5. Commit migration file to Git
6. Deploy to production (via Supabase migrations or manual SQL)

**Rollback Strategy**:
- Keep migrations reversible (include `down` migration)
- Test rollback in staging before applying to production
- Never modify committed migrations (create new ones instead)

---

## 7. Data Backup Strategy

**Supabase Automated Backups**:
- Daily backups retained for 7 days (free tier)
- Point-in-time recovery available (Pro tier)

**Additional Backup (Recommended)**:
- Weekly full database export to R2
- Export format: SQL dump (compressed)
- Retention: 4 weeks
- Restoration tested quarterly

---

## 8. Data Privacy & Compliance

### 8.1 GDPR Compliance

**Right to Access**:
- API endpoint: `GET /api/v1/data-export`
- Returns: All data associated with user's email/phone

**Right to Deletion**:
- API endpoint: `DELETE /api/v1/data-delete`
- Action: Soft-delete (anonymize) all personal data
- Retain: Analytics data only (no PII)

**Consent Tracking**:
- `leadEngagements` table tracks consent timestamps
- Checkbox for marketing communication consent
- Respects unsubscribe requests

### 8.2 Data Encryption

**At Rest**:
- Database: Encrypted by Supabase (AES-256)
- API keys: Encrypted in `integrations` table (AES-256)

**In Transit**:
- All connections over TLS 1.3
- Vercel → Supabase: Encrypted
- API calls: HTTPS only

---

## 9. Data Analytics & BI

### 9.1 Key Metrics Stored

**Lead Metrics**:
- Leads by source (landing page, chatbot, voice)
- Lead score distribution (cold/warm/hot)
- Conversion funnel (new → contacted → qualified → hot → converted)

**Engagement Metrics**:
- Email open/click rates
- Chatbot session outcomes
- Content engagement (views, likes, shares)

**Agent Metrics**:
- Response time (lead capture → first contact)
- Appointment booking rate
- Content production volume

### 9.2 Analytics Queries

**Example: Lead Conversion Rate by Source**
```sql
SELECT
  source,
  COUNT(*) as total_leads,
  SUM(CASE WHEN status = 'CONVERTED' THEN 1 ELSE 0 END) as converted,
  ROUND(100.0 * SUM(CASE WHEN status = 'CONVERTED' THEN 1 ELSE 0 END) / COUNT(*), 2) as conversion_rate
FROM leads
WHERE tenantId = $1
  AND createdAt >= $2
GROUP BY source
ORDER BY conversion_rate DESC;
```

**Example: Average Engagement Score by Lead Classification**
```sql
SELECT
  classification,
  AVG(engagement_count) as avg_engagements,
  MAX(engagement_count) as max_engagements
FROM (
  SELECT
    l.classification,
    COUNT(e.id) as engagement_count
  FROM leads l
  LEFT JOIN leadEngagements e ON e.leadId = l.id
  WHERE l.tenantId = $1
  GROUP BY l.id, l.classification
) sub
GROUP BY classification;
```

---

## 10. Future Scalability Considerations

### 10.1 Partitioning

When tenant count > 1,000:
- Partition `leads` table by `tenantId`
- Partition `leadEngagements` by `createdAt` (time-series)
- Improves query performance and vacuum speed

### 10.2 Read Replicas

When read traffic increases:
- Promote Supabase read replica for analytics queries
- Direct all dashboard/reporting queries to replica
- Keep master for write operations

### 10.3 Connection Pooling

When concurrent users > 100:
- Use PgBouncer for connection pooling
- Supabase provides built-in connection pooling
- Reduces database connection overhead
