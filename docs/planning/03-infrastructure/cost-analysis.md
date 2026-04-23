# Cost Analysis — Bare Minimum to Scale

**Project**: The Autonomous Real Estate Agent  
**Version**: 1.0  
**Last Updated**: 2026-04-22

---

## 1. Monthly Cost Summary

### Tier 1: MVP Launch (Single Agent, <100 leads/mo)

| Service | Plan | Monthly Cost |
|---|---|---|
| Vercel | Hobby (Free) | **$0** |
| Supabase PostgreSQL | Free (500MB) | **$0** |
| Upstash Redis | Free (10K cmd/day) | **$0** |
| Cloudflare R2 | Free (10GB) | **$0** |
| Cloudflare DNS | Free | **$0** |
| OpenAI API | Pay-as-you-go | **~$3** |
| SendGrid | Free (100/day) | **$0** |
| Buffer | Free (3 channels) | **$0** |
| Google Calendar API | Free | **$0** |
| HubSpot CRM | Free | **$0** |
| Sentry | Free (5K events) | **$0** |
| GitHub | Free | **$0** |
| Domain (.com) | Annual ÷ 12 | **~$1** |
| **TOTAL** | | **~$4/mo** |

> [!TIP]
> The MVP can launch at effectively **$4/month** — just domain + minimal OpenAI usage. All infrastructure runs on free tiers.

---

### Tier 2: Growth (Single Agent, 200-500 leads/mo, all features active)

| Service | Plan | Monthly Cost |
|---|---|---|
| Vercel | Pro | **$20** |
| Supabase PostgreSQL | Free → Pro borderline | **$0–25** |
| Upstash Redis | Pay-as-you-go | **~$2** |
| Cloudflare R2 | Free | **$0** |
| OpenAI API | Pay-as-you-go | **~$5** |
| Twilio (SMS + WhatsApp) | Pay-as-you-go | **~$15** |
| SendGrid | Free | **$0** |
| Bland.ai | Pay-per-call | **~$8** |
| HeyGen | Creator plan | **$29** |
| Buffer | Free | **$0** |
| HubSpot | Free / Starter | **$0–20** |
| Sentry | Free | **$0** |
| Domain | Annual ÷ 12 | **~$1** |
| **TOTAL** | | **~$80–125/mo** |

---

### Tier 3: SaaS (Multi-Tenant, 10-50 agents)

| Service | Plan | Monthly Cost |
|---|---|---|
| Vercel | Pro | **$20** |
| Supabase | Pro | **$25** |
| Upstash Redis | Pro | **$10** |
| Cloudflare R2 | Pay-as-you-go | **~$5** |
| OpenAI API | Pay-as-you-go (volume) | **~$50** |
| Twilio | Pay-as-you-go (volume) | **~$100** |
| SendGrid | Essentials ($20) | **$20** |
| Bland.ai | Volume | **~$60** |
| HeyGen | Business | **$89** |
| Buffer | Essentials ($6/channel) | **~$30** |
| HubSpot | Per tenant (pass-through) | **$0** (tenant pays) |
| Sentry | Team ($26/mo) | **$26** |
| Domain + SSL | | **~$2** |
| **TOTAL** | | **~$437/mo** |

> [!NOTE]
> At Tier 3, the platform generates revenue from 10-50 tenants paying $29-79/mo each = **$290-3,950/mo revenue** vs **~$437/mo cost**. Break-even at ~10-15 tenants on Starter plan.

---

## 2. Cost Scaling Curves

### Per-Lead Cost Breakdown

| Volume | Infra/Lead | AI/Lead | Comms/Lead | Total/Lead |
|---|---|---|---|---|
| 100 leads/mo | $0.04 | $0.03 | $0.00 | **$0.07** |
| 500 leads/mo | $0.16 | $0.01 | $0.03 | **$0.20** |
| 2,000 leads/mo | $0.05 | $0.01 | $0.03 | **$0.09** |
| 10,000 leads/mo | $0.03 | $0.01 | $0.03 | **$0.07** |

### Key Scaling Triggers

| Trigger | Threshold | Action | Added Cost |
|---|---|---|---|
| Function timeout | >10s responses | Vercel Hobby → Pro | +$20/mo |
| DB size | >500MB | Supabase Free → Pro | +$25/mo |
| Redis commands | >10K/day | Upstash Pay-as-you-go | +$2-10/mo |
| WhatsApp alerts needed | First alert | Twilio account + Meta approval | +$10-15/mo |
| Voice calls needed | First call | Bland.ai account | +$8/mo |
| Video generation needed | First video | HeyGen Creator plan | +$29/mo |
| Team access | >1 developer | Vercel Pro | Included |
| Production errors | >5K/mo | Sentry Team | +$26/mo |

---

## 3. One-Time Costs

| Item | Cost | Notes |
|---|---|---|
| Domain registration (.com) | $10-15/year | One-time + annual renewal |
| Meta Business verification (WhatsApp) | $0 | Free but takes 1-2 weeks |
| HubSpot Developer account | $0 | Free |
| Google Cloud project (Calendar API) | $0 | Free |
| SSL certificate | $0 | Vercel provides free |
| Development time (see resource plan) | See [resource-plan.md](../09-resources/resource-plan.md) | |

---

## 4. Cost Optimization Strategies

### Immediate (Day 1)
1. **Cache aggressively**: Cache HubSpot API responses (60s TTL) to reduce API calls
2. **Batch operations**: Batch CRM syncs instead of real-time per-event
3. **ISR pages**: Use Next.js ISR to serve cached pages, reducing function invocations
4. **Prompt optimization**: Keep chatbot prompts concise; use `gpt-4o-mini` for simple tasks

### Short-term (Month 2-3)
5. **pgvector over Pinecone**: Save $70/mo by using PostgreSQL extension
6. **SendGrid over Twilio SMS**: Use email for non-urgent nurture (free vs $0.0079/msg)
7. **Buffer free tier**: Limit to 3 social channels initially
8. **Video on-demand only**: Generate videos only when agent requests (skip daily automation)

### Long-term (SaaS Scale)
9. **OpenAI batch API**: 50% discount for non-real-time content generation
10. **Tenant BYOK**: Let tenants bring their own API keys (HubSpot, OpenAI) — zero cost to platform
11. **Usage-based pricing**: Charge per video/voice call above included limits
12. **Volume discounts**: Negotiate with Twilio/OpenAI at scale

---

## 5. Revenue Model (SaaS Phase)

| Plan | Price/mo | Target Segment | Included | Margin |
|---|---|---|---|---|
| **FREE** | $0 | Trial / landing page only | Landing page, 50 leads, basic scoring | — |
| **STARTER** | $29/mo | Solo agents | +Chatbot, +500 leads, +3 drip sequences | ~70% |
| **PRO** | $79/mo | Active agents | +Voice, +Content, +Videos, +Social | ~65% |
| **ENTERPRISE** | $199/mo | Brokerages | +White-label, +Custom domain, +API | ~75% |

### Break-Even Analysis

```
Fixed monthly costs (Tier 3):    ~$437
Average revenue per tenant:      ~$55 (mix of Starter + Pro)
Break-even tenants:              437 / 55 = ~8 tenants

Target Year 1:                   25 tenants
Year 1 MRR:                      25 × $55 = $1,375/mo
Year 1 margin:                   ($1,375 - $437) / $1,375 = 68%
```
