# Technology Stack — SaaS Platform

**Project**: The Autonomous Real Estate Agent  
**Version**: 1.0  
**Last Updated**: 2026-04-22

---

## 1. Stack Summary

> See diagram: [tech-layers.mmd](diagrams/tech-layers.mmd)

This stack is optimized for a SaaS real estate platform: **rapid development**, **low operational cost**, **serverless-first**, and **easy scaling**. Every choice is made to minimize DevOps overhead while maximizing capability.

---

## 2. Frontend Stack

| Technology | Version | Purpose | SaaS Rationale |
|---|---|---|---|
| **Next.js** | 15.x (App Router) | Full-stack React framework | SSR for SEO (landing pages), SSG for performance, API routes eliminate separate backend, ISR for dynamic content freshness |
| **TypeScript** | 5.x | Type safety | Catches integration bugs early, self-documenting API contracts between services |
| **Vanilla CSS + CSS Modules** | — | Styling | Per-user requirement. CSS custom properties enable tenant-level theming (brandable) |
| **Zod** | 3.x | Validation | Shared schemas between client/server, type-safe form validation |
| **SWR / TanStack Query** | Latest | Data fetching | Optimistic updates, cache invalidation, stale-while-revalidate for dashboard |

### Why Next.js over alternatives?

| Framework | SSR/SEO | API Routes | Serverless | Edge | Verdict |
|---|---|---|---|---|---|
| **Next.js 15** | ✓ Full | ✓ Built-in | ✓ Vercel native | ✓ Edge middleware | **Selected** |
| Remix | ✓ | ✓ | Partial | ✗ | Good but less ecosystem |
| Nuxt (Vue) | ✓ | ✓ | ✓ | Partial | Team prefers React |
| SvelteKit | ✓ | ✓ | ✓ | Partial | Smaller ecosystem |
| Vite + Express | ✗ Manual | ✗ Separate | ✗ Manual | ✗ | Too much custom work |

---

## 3. Backend Stack

| Technology | Purpose | SaaS Rationale |
|---|---|---|
| **Next.js API Routes** | REST endpoints | Collocated with frontend, zero deployment complexity |
| **Edge Middleware** | Auth, rate limiting, tenant resolution | Sub-millisecond latency at CDN edge |
| **Prisma** (6.x) | ORM + migrations | Type-safe queries, automatic tenant scoping, migration management |
| **BullMQ** | Job queue | Reliable background processing for emails, video gen, CRM sync |
| **Upstash Ratelimit** | API rate limiting | Per-tenant rate limits, serverless-compatible |
| **NextAuth.js** (v5) | Authentication | Multi-provider auth, JWT with tenant claims |

---

## 4. Database & Storage

| Technology | Purpose | Tier | SaaS Rationale |
|---|---|---|---|
| **PostgreSQL** (via Supabase) | Primary datastore | Free → Pro | Row-level security for tenant isolation, pgvector for embeddings, proven at scale |
| **pgvector** extension | Vector similarity search | Included | Eliminates separate vector DB cost (Pinecone) for MVP |
| **Redis** (via Upstash) | Cache, queues, sessions | Free (10K cmds/day) | Serverless Redis, pay-per-request, no idle costs |
| **Cloudflare R2** | Media storage (images, videos) | Free (10GB) | S3-compatible, zero egress fees, CDN-ready |

### Database Decision: Supabase vs. Alternatives

| Database | Free Tier | pgvector | Auth | Realtime | Cost at Scale | Verdict |
|---|---|---|---|---|---|---|
| **Supabase** | 500MB, 2 projects | ✓ | ✓ Built-in | ✓ | $25/mo Pro | **Selected** |
| Neon | 512MB, 1 project | ✓ | ✗ | ✗ | $19/mo Pro | Good alternative |
| PlanetScale | Deprecated free | ✗ (MySQL) | ✗ | ✗ | $39/mo | MySQL, no vectors |
| Railway | $5 credit | ✓ | ✗ | ✗ | Usage-based | Less integrated |

---

## 5. AI / ML Stack

| Technology | Purpose | Model | Pricing |
|---|---|---|---|
| **OpenAI API** | Chatbot RAG, content generation, summarization | GPT-4o | ~$2.50/1M input tokens, ~$10/1M output |
| **OpenAI Embeddings** | Document/query embedding for RAG | text-embedding-3-small | $0.02/1M tokens |
| **pgvector** | Vector similarity search | — | Free (PostgreSQL extension) |

### RAG Architecture

```
Knowledge Base (Markdown docs)
    ↓ Chunking (500 tokens, 100 overlap)
    ↓ Embedding (text-embedding-3-small)
    ↓ Store in pgvector

User Query
    ↓ Embed query
    ↓ Cosine similarity search (top 5 chunks)
    ↓ Construct prompt: system + context chunks + user query
    ↓ GPT-4o response
    ↓ Stream to chat widget
```

### Why pgvector over Pinecone?

| Factor | pgvector | Pinecone |
|---|---|---|
| Cost | Free (part of PostgreSQL) | Free up to 100K vectors, then $70/mo |
| Deployment | Same DB, no extra service | Separate managed service |
| Performance at <100K vectors | Excellent | Excellent |
| Performance at >1M vectors | Good (with HNSW index) | Better |
| **Verdict for MVP** | **Selected** | Add later if needed |

---

## 6. Third-Party Integrations

| Service | Purpose | SDK/API | Auth Method |
|---|---|---|---|
| **HubSpot** | CRM, contacts, deals, pipelines, workflows | `@hubspot/api-client` | OAuth2 or Private App Token |
| **Twilio** | WhatsApp messages, SMS | `twilio` | Account SID + Auth Token |
| **SendGrid** | Transactional email, drip campaigns | `@sendgrid/mail` | API Key |
| **Bland.ai** | AI voice qualification calls | REST API | API Key |
| **HeyGen** | AI avatar video generation | REST API | API Key |
| **Buffer** | Social media scheduling | REST API | OAuth2 |
| **Google Calendar** | Appointment booking | `googleapis` | OAuth2 (service account) |
| **Google Analytics 4** | Website analytics | `gtag.js` / Measurement Protocol | Measurement ID |

---

## 7. DevOps & Tooling

| Tool | Purpose |
|---|---|
| **Vercel** | Hosting, CI/CD, preview deployments, cron jobs |
| **GitHub** | Source control, issue tracking |
| **GitHub Actions** | CI: lint, type-check, test on every PR |
| **ESLint + Prettier** | Code quality and formatting |
| **Husky + lint-staged** | Pre-commit hooks |
| **Vitest** | Unit + integration testing |
| **Playwright** | E2E browser testing |

---

## 8. Monitoring & Observability

| Tool | Purpose | Tier |
|---|---|---|
| **Vercel Analytics** | Web vitals, performance | Included |
| **Sentry** | Error tracking, performance monitoring | Free (5K events/mo) |
| **Upstash QStash** | Scheduled jobs monitoring | Free tier |
| **HubSpot Reports** | CRM lead/deal analytics | Included |
| **Custom Dashboard** | Business metrics (leads, conversions, content) | Built in-app |

---

## 9. Key Package Dependencies

```json
{
  "dependencies": {
    "next": "^15.0.0",
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "@prisma/client": "^6.0.0",
    "prisma": "^6.0.0",
    "next-auth": "^5.0.0",
    "zod": "^3.23.0",
    "openai": "^4.60.0",
    "@hubspot/api-client": "^12.0.0",
    "twilio": "^5.0.0",
    "@sendgrid/mail": "^8.0.0",
    "googleapis": "^140.0.0",
    "bullmq": "^5.0.0",
    "@upstash/redis": "^1.34.0",
    "@upstash/ratelimit": "^2.0.0",
    "swr": "^2.2.0"
  },
  "devDependencies": {
    "typescript": "^5.6.0",
    "@types/react": "^19.0.0",
    "@types/node": "^22.0.0",
    "eslint": "^9.0.0",
    "prettier": "^3.3.0",
    "vitest": "^2.1.0",
    "@playwright/test": "^1.48.0",
    "husky": "^9.1.0",
    "lint-staged": "^15.2.0"
  }
}
```
