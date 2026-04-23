# Infrastructure — Bare Minimum Setup

**Project**: The Autonomous Real Estate Agent  
**Version**: 1.0  
**Last Updated**: 2026-04-22

---

## 1. Infrastructure Philosophy

> **Principle**: Pay nothing until you have customers. Use free tiers aggressively. Scale only when metrics demand it.

The infrastructure is designed around **three constraints**:
1. **Zero idle cost** — serverless/pay-per-use only
2. **No DevOps overhead** — managed services, no servers to patch
3. **Production-ready** — despite being cheap, no compromises on reliability

---

## 2. Infrastructure Topology

> See diagram: [infra-diagram.mmd](diagrams/infra-diagram.mmd)

```
Internet
    │
    ▼
┌─────────────────────────────────────────────┐
│           Vercel Edge Network               │
│  ┌──────────┐  ┌─────────────┐  ┌────────┐ │
│  │ CDN      │  │ Edge        │  │ Cron   │ │
│  │ (Static) │  │ Middleware  │  │ Jobs   │ │
│  └──────────┘  └─────────────┘  └────────┘ │
│  ┌──────────────────────────────────────┐   │
│  │    Serverless Functions (API Routes) │   │
│  │    Node.js 20 Runtime                │   │
│  │    Max 10s (Hobby) / 60s (Pro)       │   │
│  └──────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
    │           │           │           │
    ▼           ▼           ▼           ▼
┌────────┐ ┌────────┐ ┌────────┐ ┌────────────┐
│Supabase│ │Upstash │ │ Cloud- │ │  External  │
│  PG    │ │ Redis  │ │flare R2│ │  APIs      │
│        │ │        │ │        │ │            │
│-500MB  │ │-10K/day│ │-10GB   │ │-HubSpot   │
│-pgvec  │ │-256MB  │ │-0 egr  │ │-OpenAI    │
│-2 proj │ │        │ │        │ │-Twilio    │
│-Free   │ │-Free   │ │-Free   │ │-etc.      │
└────────┘ └────────┘ └────────┘ └────────────┘
```

---

## 3. Service-by-Service Infrastructure

### 3.1 Compute — Vercel

| Parameter | Hobby (Free) | Pro ($20/mo) | Recommendation |
|---|---|---|---|
| Deployments | Unlimited | Unlimited | Start with **Hobby** |
| Serverless execution | 10s timeout | 60s timeout | Pro needed for video gen/voice calls |
| Bandwidth | 100GB/mo | 1TB/mo | Hobby OK for MVP |
| Edge Middleware | ✓ | ✓ | — |
| Cron Jobs | 2 per project | 40 per project | Hobby OK initially |
| Preview Deployments | ✓ | ✓ | — |
| Analytics | Basic | Advanced | — |
| Team members | 1 | Unlimited | Pro when team grows |
| **Verdict** | **Start here** | **Upgrade when needed** | |

**When to upgrade to Pro**: 
- Function timeout exceeds 10s (video generation, long AI calls)
- Multiple team members need access
- Need more than 2 cron jobs

### 3.2 Database — Supabase PostgreSQL

| Parameter | Free | Pro ($25/mo) | Recommendation |
|---|---|---|---|
| Database size | 500MB | 8GB | Free for MVP (~50K leads) |
| Bandwidth | 5GB/mo | 250GB/mo | Free OK |
| Concurrent connections | 30 (direct) | 300 (pooled) | Free for MVP |
| pgvector | ✓ | ✓ | Use for RAG vectors |
| Row-Level Security | ✓ | ✓ | Use for tenant isolation |
| Daily backups | ✓ (7 days) | ✓ (30 days) | — |
| Branching | ✗ | ✓ | Nice for staging |
| **Verdict** | **Start here** | **Upgrade at ~300 leads/mo** | |

**Storage estimate** (500MB budget):
```
leads table:          ~1KB per lead × 10,000 = 10MB
conversations:        ~5KB per session × 5,000 = 25MB
messages:             ~0.5KB per msg × 50,000 = 25MB
properties:           ~2KB per listing × 500 = 1MB
content:              ~3KB per piece × 2,000 = 6MB
vectors (pgvector):   ~6KB per embedding × 10,000 = 60MB
indexes + overhead:   ~100MB
─────────────────────────────────────────────────
Total estimate:       ~227MB (well within 500MB free)
```

### 3.3 Cache & Queues — Upstash Redis

| Parameter | Free | Pay-as-you-go | Recommendation |
|---|---|---|---|
| Commands/day | 10,000 | $0.2/100K commands | Free for MVP |
| Max data | 256MB | 1GB+ | Free OK |
| Persistence | ✓ | ✓ | — |
| Global replication | 1 region | Multi-region | Free OK |
| **Verdict** | **Start here** | **Upgrade at ~1K daily users** | |

**Usage estimate** (10K commands/day budget):
```
Session cache:     ~500 reads + 500 writes = 1,000
Rate limiting:     ~200 checks = 200
Chat context:      ~300 reads + 300 writes = 600
Job queue (BullMQ): ~500 enqueue + 500 dequeue = 1,000
CRM sync cache:    ~200 reads + 100 writes = 300
Misc caching:      ~500 = 500
─────────────────────────────────────
Total estimate:    ~3,600/day (well within 10K free)
```

### 3.4 Media Storage — Cloudflare R2

| Parameter | Free | Paid |
|---|---|---|
| Storage | 10GB | $0.015/GB/mo |
| Class A ops (write) | 1M/mo | $4.50/1M |
| Class B ops (read) | 10M/mo | $0.36/1M |
| Egress | **Free forever** | **Free forever** |
| **Verdict** | **Start here** | |

**Storage estimate** (10GB budget):
```
Property images:    ~2MB × 5 images × 100 listings = 1GB
Agent avatars:      ~500KB × 10 = 5MB
Generated videos:   ~20MB × 50 = 1GB
Email templates:    ~100KB × 20 = 2MB
Branding assets:    ~5MB per tenant × 5 = 25MB
─────────────────────────────────────
Total estimate:     ~2GB (well within 10GB free)
```

### 3.5 Domain & DNS

| Service | Cost | Notes |
|---|---|---|
| **Cloudflare** (DNS) | Free | Free DNS, DDoS protection, SSL |
| **Domain** (.com) | ~$10/year | Via any registrar |
| **Vercel custom domain** | Free | Included with Vercel |

---

## 4. External API Usage Estimates (Monthly)

Based on a single-agent deployment handling ~200 leads/month:

| API | Operation | Est. Volume | Unit Cost | Monthly Cost |
|---|---|---|---|---|
| **OpenAI GPT-4o** | Chatbot responses | 1,000 msgs × ~500 tokens | $2.50/1M input | ~$1.25 |
| | Content generation | 50 pieces × ~2,000 tokens | $10/1M output | ~$1.00 |
| | Summarization | 100 calls × ~500 tokens | — | ~$0.50 |
| **OpenAI Embeddings** | RAG indexing | 500 chunks × ~200 tokens | $0.02/1M | ~$0.01 |
| | Query embedding | 1,000 queries × ~50 tokens | — | ~$0.01 |
| **Twilio SMS** | Lead alerts + nurture | 500 messages | $0.0079/msg | ~$3.95 |
| **Twilio WhatsApp** | Agent alerts | 200 messages | $0.005/msg + $0.05 template | ~$11.00 |
| **SendGrid** | Drip emails | 1,000 emails | Free (100/day) | $0.00 |
| **Bland.ai** | Voice calls | 30 calls × 3 min | $0.09/min | ~$8.10 |
| **HeyGen** | Video generation | 10 videos | Creator plan | ~$29.00 |
| **Buffer** | Social scheduling | 30 posts | Free (3 channels) | $0.00 |
| **Google Calendar** | Bookings | 50 events | Free | $0.00 |
| **HubSpot** | CRM operations | 500 API calls | Free tier | $0.00 |

---

## 5. Environment Configuration

### Required Environment Variables

```env
# ─── Application ───
NEXT_PUBLIC_APP_URL=https://your-domain.com
NEXTAUTH_SECRET=<generate-random-32-char>
NEXTAUTH_URL=https://your-domain.com

# ─── Database ───
DATABASE_URL=postgresql://user:pass@host:5432/db?sslmode=require

# ─── Redis ───
UPSTASH_REDIS_REST_URL=https://xxx.upstash.io
UPSTASH_REDIS_REST_TOKEN=xxx

# ─── AI ───
OPENAI_API_KEY=sk-xxx

# ─── CRM ───
HUBSPOT_ACCESS_TOKEN=pat-xxx
HUBSPOT_PORTAL_ID=xxx

# ─── Communication ───
TWILIO_ACCOUNT_SID=ACxxx
TWILIO_AUTH_TOKEN=xxx
TWILIO_PHONE_NUMBER=+1xxx
TWILIO_WHATSAPP_NUMBER=whatsapp:+1xxx
SENDGRID_API_KEY=SG.xxx
AGENT_PHONE_NUMBER=+1xxx
AGENT_EMAIL=agent@email.com

# ─── Voice ───
BLAND_AI_API_KEY=xxx

# ─── Video ───
HEYGEN_API_KEY=xxx

# ─── Social ───
BUFFER_ACCESS_TOKEN=xxx

# ─── Calendar ───
GOOGLE_CALENDAR_CLIENT_EMAIL=xxx
GOOGLE_CALENDAR_PRIVATE_KEY=xxx
GOOGLE_CALENDAR_ID=xxx

# ─── Storage ───
R2_ACCOUNT_ID=xxx
R2_ACCESS_KEY_ID=xxx
R2_SECRET_ACCESS_KEY=xxx
R2_BUCKET_NAME=xxx
R2_PUBLIC_URL=https://media.your-domain.com

# ─── Analytics ───
NEXT_PUBLIC_GA_MEASUREMENT_ID=G-xxx
HUBSPOT_TRACKING_ID=xxx

# ─── Monitoring ───
SENTRY_DSN=https://xxx@sentry.io/xxx
```

---

## 6. Security Hardening (Even on Free Tier)

| Concern | Solution | Cost |
|---|---|---|
| DDoS protection | Cloudflare free (DNS proxy mode) | $0 |
| Rate limiting | Upstash Ratelimit (API routes) | $0 |
| HTTPS/TLS | Vercel automatic SSL | $0 |
| Secrets management | Vercel environment variables (encrypted) | $0 |
| SQL injection | Prisma parameterized queries | $0 |
| XSS prevention | Next.js built-in escaping + CSP headers | $0 |
| CSRF | SameSite cookies + CSRF token | $0 |
| API key encryption | Encrypt at rest in DB (AES-256) | $0 |
| Dependency scanning | GitHub Dependabot (free) | $0 |
