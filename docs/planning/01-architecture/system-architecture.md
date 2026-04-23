---
layout: page
title: System Architecture
---

# System Architecture — SaaS Design

**Project**: The Autonomous Real Estate Agent  
**Version**: 1.0  
**Last Updated**: 2026-04-22

---

## 1. Architecture Philosophy

This platform is architected as a **SaaS-ready, multi-tenant system** from day one. Even though the initial deployment serves a single NYC real estate agent, every design decision is made to support:

- **Multi-tenancy**: Multiple agents/brokerages sharing the same infrastructure
- **Horizontal scaling**: Stateless services that scale independently
- **White-labeling**: Customizable branding per tenant
- **Usage-based billing**: Metered API calls, leads, content generation

---

## 2. High-Level Architecture

> See diagram: [system-overview.mmd](diagrams/system-overview.mmd)

### Layers

```
┌─────────────────────────────────────────────────────────┐
│                   CLIENT LAYER                          │
│  Landing Page │ Admin Dashboard │ Chatbot Widget        │
│  (Next.js SSR/SSG)                                      │
├─────────────────────────────────────────────────────────┤
│                   API GATEWAY                           │
│  Authentication │ Rate Limiting │ Tenant Resolution     │
│  (Next.js API Routes / Edge Middleware)                  │
├─────────────────────────────────────────────────────────┤
│                  SERVICE LAYER                          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌───────────┐  │
│  │ Lead     │ │ Chatbot  │ │ Content  │ │ Nurture   │  │
│  │ Service  │ │ Service  │ │ Factory  │ │ Engine    │  │
│  └──────────┘ └──────────┘ └──────────┘ └───────────┘  │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌───────────┐  │
│  │ CRM Sync │ │ Alert    │ │ Voice    │ │ Social    │  │
│  │ Service  │ │ Service  │ │ Agent    │ │ Publisher │  │
│  └──────────┘ └──────────┘ └──────────┘ └───────────┘  │
├─────────────────────────────────────────────────────────┤
│                INTEGRATION LAYER                        │
│  HubSpot │ OpenAI │ Twilio │ HeyGen │ Buffer │ Google  │
├─────────────────────────────────────────────────────────┤
│                   DATA LAYER                            │
│  PostgreSQL │ Redis (Cache/Queue) │ Pinecone (Vectors)  │
│  S3/R2 (Media Storage)                                  │
└─────────────────────────────────────────────────────────┘
```

---

## 3. Service Decomposition

Even though we deploy as a **modular monolith** initially (all services within one Next.js app), the code is structured so any service can be extracted into a standalone microservice later.

### 3.1 Lead Service
- **Responsibility**: Lead capture, validation, deduplication, scoring, classification
- **Input**: Contact form submissions, chatbot conversations, voice call outcomes
- **Output**: Scored & classified lead records in DB + CRM
- **Interfaces**: REST API (`/api/leads/*`)

### 3.2 Chatbot Service
- **Responsibility**: RAG-powered conversational AI, conversation management
- **Input**: User messages via chat widget
- **Output**: AI responses, conversation logs, intent classification
- **Interfaces**: REST API (`/api/chatbot/*`), WebSocket (future)
- **Dependencies**: OpenAI, Pinecone/pgvector

### 3.3 CRM Sync Service
- **Responsibility**: Bi-directional synchronization with HubSpot
- **Input**: Lead events, webhook payloads from HubSpot
- **Output**: Synced contact/deal records
- **Interfaces**: REST API (`/api/hubspot/*`), webhook receiver
- **Pattern**: Event-driven with retry queue

### 3.4 Alert Service
- **Responsibility**: Real-time notifications to agent
- **Input**: HOT lead classification events
- **Output**: WhatsApp messages, email alerts, push notifications (future)
- **Interfaces**: Internal event consumer
- **Dependencies**: Twilio, SendGrid

### 3.5 Nurture Engine
- **Responsibility**: Orchestrate drip campaigns, track engagement
- **Input**: Lead classification events, engagement signals
- **Output**: Scheduled emails, SMS, escalation triggers
- **Interfaces**: REST API (`/api/nurture/*`), cron jobs
- **Dependencies**: HubSpot Workflows or custom engine

### 3.6 Voice Agent Service
- **Responsibility**: AI voice qualification calls
- **Input**: Non-engaged HOT leads after Day 3
- **Output**: Call transcripts, qualification data, calendar bookings
- **Interfaces**: REST API (`/api/voice/*`)
- **Dependencies**: Bland.ai/Vapi, Google Calendar

### 3.7 Content Factory
- **Responsibility**: AI content generation from listing data
- **Input**: Property URLs or structured listing data
- **Output**: Video scripts, social captions, AI videos
- **Interfaces**: REST API (`/api/content/*`)
- **Dependencies**: OpenAI, HeyGen/Synthesia

### 3.8 Social Publisher
- **Responsibility**: Multi-platform content scheduling and publishing
- **Input**: Approved content from Content Factory
- **Output**: Published posts, engagement metrics
- **Interfaces**: REST API (`/api/social/*`), cron jobs
- **Dependencies**: Buffer/Later API

---

## 4. Cross-Cutting Concerns

### 4.1 Authentication & Authorization
- **NextAuth.js** with Google OAuth + credentials provider
- Role-based access: `SUPER_ADMIN`, `AGENT_ADMIN`, `AGENT`
- JWT tokens with tenant ID embedded
- API routes protected via middleware

### 4.2 Multi-Tenancy Strategy
- **Database**: Shared database with `tenantId` column on all tables (Row-Level Security)
- **Application**: Tenant resolution via subdomain (`agent1.platform.com`) or path prefix
- **Storage**: Tenant-scoped media buckets (prefix-based on S3/R2)
- **Integrations**: Tenant-specific API keys stored encrypted in DB

> See detailed design: [saas-tenancy-model.md](saas-tenancy-model.md)

### 4.3 Event Bus (Internal)
- **Phase 1**: Simple in-process event emitter (Node.js EventEmitter pattern)
- **Phase 2**: Redis Pub/Sub for decoupled services
- **Future**: Dedicated message queue (BullMQ on Redis)

Key events:
```
lead.created       → CRM Sync, Lead Scorer, Alert Service
lead.scored        → Nurture Engine, Alert Service (if HOT)
lead.engaged       → Nurture Engine (pause/resume sequences)
chat.completed     → Lead Service, CRM Sync
content.approved   → Social Publisher
voice.completed    → Lead Service, Calendar Service
```

### 4.4 Background Jobs & Scheduling
- **BullMQ** on Redis for reliable job processing
- Job types:
  - CRM sync (retry with backoff)
  - Email/SMS sending
  - Content generation (long-running)
  - Video generation (async with webhooks)
  - Social publishing (scheduled)
  - Engagement metric fetching (periodic)

### 4.5 Caching Strategy
- **Redis** for:
  - API response caching (property data, market stats)
  - Session storage
  - Rate limiting counters
  - Chat context window (last N messages per session)
- **Next.js ISR** for:
  - Landing page (revalidate: 3600)
  - Property detail pages (revalidate: 1800)

---

## 5. Deployment Architecture

### Initial (Single-Tenant, Bare-Minimum)
```
Vercel (Free/Pro)
  ├── Next.js App (Frontend + API)
  ├── Edge Middleware (auth, rate limits)
  └── Cron Jobs (Vercel Cron)

Supabase (Free Tier)
  ├── PostgreSQL (with pgvector extension)
  └── Auth (backup option)

Upstash (Free Tier)
  └── Redis (caching, queues)

Cloudflare R2 (Free Tier)
  └── Media storage (images, videos)
```

### Growth (Multi-Tenant SaaS)
```
Vercel (Pro/Enterprise)
  ├── Next.js App
  ├── Edge Functions (tenant routing)
  └── Cron Jobs

AWS RDS / Supabase Pro
  └── PostgreSQL (with connection pooling)

Upstash Pro / AWS ElastiCache
  └── Redis (HA cluster)

Pinecone (Standard)
  └── Vector store (per-tenant namespaces)

AWS S3 / Cloudflare R2
  └── Media storage (tenant-scoped)
```

---

## 6. Security Architecture

| Layer | Measure |
|---|---|
| **Transport** | HTTPS everywhere (Vercel automatic TLS) |
| **Authentication** | NextAuth.js, JWT with short expiry (15min) + refresh tokens |
| **Authorization** | RBAC middleware, row-level security on DB queries |
| **API Security** | Rate limiting (Upstash Ratelimit), CORS, CSRF tokens |
| **Data Encryption** | API keys encrypted at rest (AES-256), PII fields encrypted |
| **Input Validation** | Zod schemas on all API inputs, SQL injection prevention via Prisma |
| **Secrets** | Environment variables, never committed to repo |
| **Audit** | Action logging for admin operations, CRM sync events |
| **Compliance** | GDPR-ready: data export, deletion APIs, consent tracking |
