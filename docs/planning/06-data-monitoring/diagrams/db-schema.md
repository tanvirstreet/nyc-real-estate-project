---
nav_exclude: true
---
# Database Entity Relationship Diagram

## Complete Schema

```mermaid
erDiagram
    tenants ||--o{ users : "has"
    tenants ||--o{ leads : "owns"
    tenants ||--o{ properties : "lists"
    tenants ||--o{ content : "creates"
    tenants ||--o{ nurtureSequences : "defines"
    tenants ||--o{ integrations : "configures"
    tenants ||--o{ auditLog : "logs"

    leads ||--o{ leadEngagements : "tracks"
    leads ||--o{ nurtureEnrollments : "enrolled in"
    leads ||--o{ chatbotSessions : "has"

    nurtureSequences ||--o{ nurtureEnrollments : "executes"

    properties ||--o{ content : "generates"
    properties ||--o{ propertyEmbeddings : "embeds"

    tenants {
        uuid id PK
        string name
        string slug UK
        string status
        string plan
        jsonb settings
        string billingEmail
        timestamptz createdAt
        timestamptz updatedAt
    }

    users {
        uuid id PK
        uuid tenantId FK
        string email UK
        string name
        string role
        string passwordHash
        timestamptz lastLoginAt
        boolean isActive
        timestamptz createdAt
        timestamptz updatedAt
    }

    leads {
        uuid id PK
        uuid tenantId FK
        string status
        integer score
        string classification
        string source
        string firstName
        string lastName
        string email
        string phone
        string preferredContact
        jsonb propertyInterest
        jsonb utm
        jsonb crmSync
        timestamptz lastContactedAt
        timestamptz createdAt
        timestamptz updatedAt
    }

    leadEngagements {
        uuid id PK
        uuid leadId FK
        string type
        string channel
        jsonb metadata
        timestamptz createdAt
    }

    properties {
        uuid id PK
        uuid tenantId FK
        string mlsId UK
        string title
        jsonb address
        jsonb details
        text description
        text[] features
        jsonb media
        jsonb nearby
        string status
        timestamptz listedAt
        timestamptz updatedAt
    }

    propertyEmbeddings {
        uuid id PK
        uuid propertyId FK
        text chunk
        vector embedding
        timestamptz createdAt
    }

    content {
        uuid id PK
        uuid tenantId FK
        uuid propertyId FK
        string type
        string status
        jsonb content
        string generatedBy
        timestamptz approvedAt
        uuid approvedBy FK
        timestamptz scheduledAt
        timestamptz publishedAt
        string[] platforms
        jsonb engagement
        timestamptz createdAt
        timestamptz updatedAt
    }

    nurtureSequences {
        uuid id PK
        uuid tenantId FK
        string name
        text description
        string triggerClassification
        boolean isActive
        jsonb steps
        timestamptz createdAt
        timestamptz updatedAt
    }

    nurtureEnrollments {
        uuid id PK
        uuid leadId FK
        uuid sequenceId FK
        integer currentStep
        timestamptz pausedAt
        timestamptz completedAt
        timestamptz enrolledAt
        timestamptz nextStepAt
    }

    chatbotSessions {
        uuid id PK
        uuid tenantId FK
        uuid leadId FK
        jsonb messages
        jsonb propertyInterest
        boolean leadCaptured
        timestamptz endedAt
        timestamptz createdAt
    }

    integrations {
        uuid id PK
        uuid tenantId FK
        string service
        string status
        bytea credentials
        timestamptz lastHealthCheck
        text lastError
        jsonb configuration
        timestamptz createdAt
        timestamptz updatedAt
    }

    auditLog {
        uuid id PK
        uuid tenantId FK
        uuid userId FK
        string action
        string entityType
        uuid entityId
        jsonb changes
        inet ipAddress
        text userAgent
        timestamptz createdAt
    }
```

---

## Key Relationships

### Tenant Multi-Tenancy
```
tenants (1) ──< (N) users
tenants (1) ──< (N) leads
tenants (1) ──< (N) properties
tenants (1) ──< (N) content
tenants (1) ──< (N) nurtureSequences
tenants (1) ──< (N) integrations
tenants (1) ──< (N) auditLog
```

### Lead Funnel
```
leads (1) ──< (N) leadEngagements
leads (1) ──< (N) nurtureEnrollments
leads (1) ──< (N) chatbotSessions
```

### Content Pipeline
```
properties (1) ──< (N) content
properties (1) ──< (N) propertyEmbeddings
```

### Nurture Automation
```
nurtureSequences (1) ──< (N) nurtureEnrollments
```

---

## Indexes Summary

### Performance Indexes
- `leads`: tenantId + status, email, phone, score, classification, createdAt
- `leadEngagements`: leadId, type, createdAt
- `properties`: mlsId, tenantId + status, details (GIN), address (GIN)
- `content`: tenantId + status, propertyId, scheduledAt, type
- `chatbotSessions`: tenantId, leadId, createdAt
- `nurtureEnrollments`: leadId, nextStepAt, sequenceId
- `propertyEmbeddings`: propertyId, embedding (HNSW)

### Security Indexes
- `users`: email (unique)
- `tenants`: slug (unique)
- `integrations`: tenantId + service (unique)

### Analytics Indexes
- `auditLog`: tenantId, userId, entityType + entityId, createdAt
- `leadEngagements`: leadId + createdAt (time-series)

---

## Data Volume Estimates

### Per Tenant (Monthly)
- **Leads**: 50-100 new leads
- **LeadEngagements**: 500-1,000 records (10 per lead)
- **ChatbotSessions**: 200-300 sessions
- **Properties**: 10-20 active listings
- **Content**: 20-30 pieces generated

### Multi-Tenant Scale (100 Tenants, Monthly)
- **Leads**: 5,000-10,000 new rows
- **LeadEngagements**: 50,000-100,000 new rows
- **ChatbotSessions**: 20,000-30,000 new rows
- **Properties**: 1,000-2,000 rows
- **Content**: 2,000-3,000 rows

### Database Growth (Year 1)
- **Estimated total rows**: ~1.5M rows
- **Estimated storage**: ~5-10GB (includes indexes)
- **Recommended backup**: Weekly + daily incremental
