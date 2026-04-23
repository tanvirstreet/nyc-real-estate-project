# Monitoring Strategy — Observability and Alerting

**Project**: The Autonomous Real Estate Agent
**Version**: 1.0
**Last Updated**: 2026-04-23

---

## 1. Monitoring Philosophy

This platform follows **observability best practices**:

1. **Proactive, Not Reactive**: Alerts before users notice
2. **Actionable Alerts**: Every alert requires immediate action
3. **SLO-Based Monitoring**: Measure against Service Level Objectives
4. **Golden Signals**: Latency, Traffic, Errors, Saturation
5. **Cost-Conscious**: Free-tier tools first, upgrade when needed

---

## 2. Monitoring Stack Overview

> See diagram: [monitoring-stack.mmd](diagrams/monitoring-stack.mmd)

**Core Tools**:
- **Vercel Analytics**: Web vitals, performance monitoring
- **Sentry**: Error tracking, performance monitoring
- **Upstash QStash**: Scheduled job monitoring
- **HubSpot Reports**: CRM lead/deal analytics
- **Custom Dashboard**: Business metrics (built in-app)

**Cost**: Free tiers only for MVP ($0/month)

---

## 3. Application Monitoring

### 3.1 Error Tracking (Sentry)

**Setup**:
```javascript
// sentry.client.config.ts
import * as Sentry from "@sentry/nextjs"

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1, // 10% of transactions for performance
  beforeSend(event) {
    // Filter out PII
    if (event.request) {
      delete event.request.cookies
      delete event.request.headers
    }
    return event
  }
})
```

**What to Track**:
- Unhandled errors (client + server)
- API route errors (4xx, 5xx)
- Database connection errors
- Third-party integration failures
- Performance transactions (slow endpoints)

**Error Scopes**:
```
Error: Lead capture form submission failed
  Scope:
    - userEmail: filtered
    - tenantId: {tenantId}
    - errorMessage: "HubSpot API rate limit exceeded"
    - endpoint: "/api/v1/leads"
    - statusCode: 503
```

**Alerting Rules**:
| Rule | Threshold | Action |
|---|---|---|
| Critical error spike | >10 errors in 5 minutes | Email + Slack to on-call |
| API error rate > 5% | 1-minute window | Page on-call engineer |
| Integration down | 3 consecutive failures | Email admin, auto-disable integration |

---

### 3.2 Performance Monitoring (Vercel Analytics)

**Metrics Tracked**:
- **Web Vitals**: LCP, FID, CLS
- **Page Views**: By page, by tenant
- **API Route Performance**: p50, p95, p99 latency
- **Build Times**: Deploy duration, build failures

**Key Dashboards**:

1. **Web Vitals Overview**
   - LCP (Largest Contentful Paint): Target < 2.5s
   - FID (First Input Delay): Target < 100ms
   - CLS (Cumulative Layout Shift): Target < 0.1
   - Route by route breakdown

2. **API Performance**
   - `/api/v1/leads`: Target < 500ms (p95)
   - `/api/v1/chatbot/message`: Target < 2s (p95, includes OpenAI call)
   - `/api/v1/content/generate`: Target < 10s (p95, async)

**Alerting Rules**:
| Rule | Threshold | Action |
|---|---|---|
| P95 latency > 2s | Any API route | Investigate slow queries |
| LCP > 3s | Landing page | Optimize images/code splitting |
| Build failure | Any deploy | Block merge, investigate |

---

### 3.3 Uptime Monitoring (UptimeRobot / Pingdom)

**Free Tier**:
- UptimeRobot: 50 monitors, 5-minute intervals
- Pingdom: One monitor, 1-minute intervals

**Monitors**:
1. **Landing Page**: `https://platform.com` (expected: 200)
2. **API Health**: `https://platform.com/api/health` (expected: 200)
3. **Chatbot API**: `https://platform.com/api/v1/chatbot/message` (expected: 200/400/422)
4. **Webhook Receiver**: `https://platform.com/api/webhooks/hubspot` (expected: 200)

**Alerting**:
- Down for >2 minutes: Email + Slack
- Down for >10 minutes: Page on-call

---

## 4. Business Metrics Monitoring

### 4.1 Lead Funnel Monitoring

**Metric**: Lead conversion rates by stage

**Dashboard**:
```
Leads (Last 7 Days)
├── Total Captured: 150
├── New: 80 (53%)
├── Contacted: 40 (27%)
├── Qualified: 20 (13%)
├── Hot: 8 (5%)
└── Converted: 2 (1.3%)

Conversion Rates
├── New → Contacted: 50%
├── Contacted → Qualified: 50%
├── Qualified → Hot: 40%
└── Hot → Converted: 25%
```

**Alerting Rules**:
| Rule | Threshold | Action |
|---|---|---|
| Lead capture drop | >30% decrease vs. last week | Investigate forms/chatbot |
| Conversion rate drop | >20% decrease vs. baseline | Review nurture sequences |

---

### 4.2 Integration Health Monitoring

**Metric**: Third-party service availability and latency

**Dashboard**:
```
Integration Status (Real-time)
├── HubSpot: ✓ (142ms)
├── Twilio: ✓ (89ms)
├── SendGrid: ✓ (234ms)
├── OpenAI: ✓ (1.2s)
├── Bland.ai: ✗ (timeout)
└── HeyGen: ✓ (4.1s)
```

**Health Check Logic**:
```typescript
// Background job runs every 5 minutes
async function checkIntegrationHealth(service: IntegrationService) {
  const start = Date.now()
  try {
    await service.ping()
    const latency = Date.now() - start

    await db.integration.update({
      where: { service },
      data: {
        status: 'ACTIVE',
        lastHealthCheck: new Date(),
        lastError: null
      }
    })

    if (latency > 5000) {
      // Slow but working
      logger.warn(`${service} latency high: ${latency}ms`)
    }
  } catch (error) {
    // Mark as degraded
    await db.integration.update({
      where: { service },
      data: {
        status: 'ERROR',
        lastHealthCheck: new Date(),
        lastError: error.message
      }
    })

    // Trigger alert if 3 consecutive failures
    const consecutiveFailures = await getConsecutiveFailures(service)
    if (consecutiveFailures >= 3) {
      await alertIntegrationDown(service, error)
      await disableIntegration(service)
    }
  }
}
```

---

### 4.3 Content Generation Monitoring

**Metric**: Content production volume and success rate

**Dashboard**:
```
Content Generation (Last 7 Days)
├── Video Scripts Generated: 12
├── AI Videos Completed: 8 (67% success)
├── Social Posts Scheduled: 24
├── Posts Published: 18
└── Average Generation Time: 4.2 minutes

Content Engagement
├── Total Views: 1,240
├── Total Likes: 89
├── Total Comments: 12
└── Avg. Engagement Rate: 3.2%
```

**Alerting Rules**:
| Rule | Threshold | Action |
|---|---|---|
| Video generation fail rate > 30% | 1-hour window | Check HeyGen status |
| No content generated for 7 days | Any tenant | Remind agent to create content |

---

### 4.4 Chatbot Performance Monitoring

**Metric**: Chatbot outcomes and satisfaction

**Dashboard**:
```
Chatbot Performance (Last 7 Days)
├── Total Sessions: 340
├── Leads Captured: 45 (13.2%)
├── Avg. Messages per Session: 4.2
├── Avg. Session Duration: 3:45
├── Handover to Human: 8 (2.3%)
└── User Satisfaction: 4.6/5.0

Top Intents
├── Property Search: 180 (52%)
├── Pricing Questions: 89 (26%)
├── Scheduling: 45 (13%)
└── General Questions: 26 (8%)
```

**Alerting Rules**:
| Rule | Threshold | Action |
|---|---|---|
| Lead capture rate < 5% | Daily average | Review chatbot prompts |
| Handover rate > 10% | Weekly average | Improve knowledge base |

---

## 5. Infrastructure Monitoring

### 5.1 Database Monitoring (Supabase)

**Metrics**:
- Connection pool usage
- Query performance (slow queries)
- Storage usage
- Row count per table

**Dashboard**:
```
Database Health
├── Connections: 12/100 (12%)
├── Storage Used: 245MB / 500MB (49%)
├── Slow Queries (Last 24h): 3
├── Backup Status: ✓ (last: 6 hours ago)
└── Replication Lag: N/A (single instance)

Slow Queries
├── SELECT * FROM leads WHERE ... (2.3s)
├── SELECT * FROM leadEngagements WHERE ... (1.8s)
└── SELECT * FROM properties WHERE details @> ... (1.5s)
```

**Alerting Rules**:
| Rule | Threshold | Action |
|---|---|---|
| Storage > 80% | Database size | Plan archival or upgrade |
| Slow query > 3s | Any query | Add index or optimize |
| Connection pool > 80% | Connection count | Increase pool size |

---

### 5.2 Redis Monitoring (Upstash)

**Metrics**:
- Command count (usage tracking)
- Memory usage
- Key count
- Queue backlog

**Dashboard**:
```
Redis Health
├── Commands Today: 8,432 / 10,000 (84%)
├── Memory Used: 45MB / 256MB (18%)
├── Total Keys: 1,240
├── Queue Backlog: 0
└── Uptime: 99.9%

Queue Status
├── CRM Sync: 0 pending
├── Email Send: 0 pending
├── Content Gen: 2 in progress
└── Voice Calls: 0 pending
```

**Alerting Rules**:
| Rule | Threshold | Action |
|---|---|---|
| Command quota > 90% | Daily usage | Alert admin, optimize caching |
| Queue backlog > 100 | Any queue | Scale workers |
| Redis down | Health check | Alert + failover to backup |

---

### 5.3 Storage Monitoring (Cloudflare R2)

**Metrics**:
- Storage usage
- Request count (Class A/B operations)
- Egress bandwidth

**Dashboard**:
```
R2 Storage
├── Storage Used: 2.4GB / 10GB (24%)
├── Total Objects: 842
├── PUT Requests: 245
├── GET Requests: 3,420
└── Egress This Month: 12.4GB

Cost Estimate
├── Storage: $0.024/GB × 2.4GB = $0.06
├── Class A Operations: $4.50/M × 245 = $1.10
├── Class B Operations: $0.36/M × 3,420 = $1.23
└── Egress: FREE (R2 benefit)
└── Total: ~$2.39/month
```

**Alerting Rules**:
| Rule | Threshold | Action |
|---|---|---|
| Storage > 80% | Free tier limit | Clean up old media or upgrade |
| Egress spikes > 10GB/day | Bandwidth usage | Investigate abuse |

---

## 6. Log Aggregation

### 6.1 Application Logs (Vercel Logs)

**Access**: Vercel Dashboard → Logs

**Log Levels**:
- `ERROR`: Application errors, integration failures
- `WARN`: Deprecation warnings, rate limits approaching
- `INFO`: Important events (lead created, content published)
- `DEBUG`: Detailed debugging (development only)

**Log Format**:
```json
{
  "timestamp": "2026-04-23T10:30:45.123Z",
  "level": "INFO",
  "tenantId": "uuid",
  "userId": "uuid",
  "action": "lead.created",
  "entityType": "lead",
  "entityId": "uuid",
  "metadata": {
    "source": "landing_page",
    "score": 85,
    "classification": "HOT"
  },
  "requestId": "uuid",
  "duration": 234
}
```

**Querying**:
```
# Find all lead creations in last hour
level:INFO action:lead.created @timestamp:[now-1h TO now]

# Find all errors for specific tenant
level:ERROR tenantId:{tenantId}

# Find slow API requests
duration:>3000 @timestamp:[now-24h TO now]
```

---

### 6.2 Audit Logs

**Storage**: Database table `auditLog`

**Retention**:
- General logs: 1 year
- Security logs: 7 years

**Export**:
- Monthly CSV export to R2
- Admin portal for ad-hoc exports

**Audit Trail for Compliance**:
```
User: john@agent.com (IP: 192.168.1.1)
Action: UPDATE lead status
Entity: lead/abc-123
Before: { "status": "NEW", "score": 50 }
After: { "status": "CONTACTED", "score": 65 }
Timestamp: 2026-04-23T10:30:45Z
```

---

## 7. Alerting Strategy

### 7.1 Alert Channels

**Primary Channels**:
1. **Email**: On-call engineer rotation
2. **Slack**: #alerts channel (for non-critical)
3. **SMS/Pager**: Critical alerts only (via Twilio)

**Channel Routing**:
| Severity | Channel | Response Time |
|---|---|---|
| **P0 - Critical** | Email + SMS + Slack | < 15 minutes |
| **P1 - High** | Email + Slack | < 1 hour |
| **P2 - Medium** | Slack only | < 4 hours |
| **P3 - Low** | Daily digest email | Next business day |

---

### 7.2 Alert Definitions

| Alert Name | Severity | Condition | Action |
|---|---|---|
| **API Error Rate Spike** | P0 | Error rate > 5% for 1 minute | Page on-call, investigate routes |
| **Database Connection Failed** | P0 | 3 consecutive connection failures | Page on-call, check Supabase status |
| **Integration Down** | P0 | 3 consecutive health check failures | Auto-disable, email admin |
| **Webhook Delivery Failed** | P1 | >10% failure rate for 10 minutes | Investigate webhook handler |
| **Lead Capture Drop** | P1 | >30% decrease vs. last week | Review forms/chatbot |
| **Slow API Response** | P2 | P95 latency > 3s for 5 minutes | Investigate slow queries |
| **Storage Quota Warning** | P2 | Any resource > 80% capacity | Plan cleanup or upgrade |
| **Queue Backlog** | P2 | >100 jobs pending for 15 minutes | Scale workers |
| **Low Lead Conversion** | P3 | Conversion rate < 1% for 7 days | Review nurture sequences |

---

### 7.3 On-Call Rotation

**Initial (Single Dev)**:
- On-call: Solo founder/dev
- Response time: Best effort
- Escalation: Self-remediate

**Growth (2-3 Devs)**:
- Weekly rotation
- Primary + secondary on-call
- Escalation path: Primary → Secondary → CTO

**Mature (Team)**:
- 24/7 rotation (if 24/7 support offered)
- PagerDuty for automated paging
- Runbook for each alert type

---

## 8. Service Level Objectives (SLOs)

### 8.1 SLO Definitions

| Service | Metric | Target | Measurement Window |
|---|---|---|---|
| **Landing Page** | Availability | 99.9% | Monthly (43.2 min downtime/month allowed) |
| **API Routes** | Success Rate | 99.5% | Monthly (3.6 hours downtime/month allowed) |
| **Lead Capture** | Uptime | 99% | Weekly (1.68 hours downtime/week allowed) |
| **Chatbot** | Response Time | P95 < 3s | Daily |
| **Content Generation** | Success Rate | 95% | Weekly |
| **CRM Sync** | Data Freshness | < 5 min lag | Real-time |

---

### 8.2 SLO Dashboards

**Example: API Success Rate**
```
API Success Rate (Last 30 Days)
├── Target: 99.5%
├── Actual: 99.72%
├── Error Budget Remaining: +0.22%
└── Status: ✓ ON TRACK

Breakdown by Endpoint
├── POST /api/v1/leads: 99.9%
├── POST /api/v1/chatbot/message: 99.5%
├── GET /api/v1/properties: 99.95%
└── POST /api/v1/content/generate: 98.2% ⚠️
```

---

### 8.3 Error Budget Policy

**Over Budget**: If SLO missed:
- **Pause non-critical work**: No new features until SLO recovered
- **Blameless postmortem**: Document root cause, not blame
- **Action items**: Specific tasks to prevent recurrence

**Under Budget**: If SLO exceeded:
- **Velocity increases**: Can take on riskier features
- **Experimentation**: A/B tests, new integrations

---

## 9. Incident Response

### 9.1 Incident Triage

**Severity Levels**:
- **SEV0**: Complete outage (all users affected)
- **SEV1**: Critical feature down (lead capture, chatbot)
- **SEV2**: Partial outage (some users, some features)
- **SEV3**: Minor issue (workaround available)

**Response SLAs**:
| Severity | Initial Response | Update Frequency | Resolution Target |
|---|---|---|---|
| SEV0 | 15 minutes | Every 15 minutes | < 2 hours |
| SEV1 | 30 minutes | Every 30 minutes | < 4 hours |
| SEV2 | 1 hour | Every 2 hours | < 24 hours |
| SEV3 | Next business day | Daily | < 1 week |

---

### 9.2 Incident Runbook Template

```markdown
# Incident: [Title]

## Meta
- **Severity**: SEV0/SEV1/SEV2/SEV3
- **Start Time**: [timestamp]
- **Detected By**: [monitoring alert / user report]
- **On-Call Engineer**: [name]
- **Status**: INVESTIGATING / IDENTIFIED / MONITORING / RESOLVED

## Impact
- **Users Affected**: [all / specific tenant / region]
- **Features Affected**: [lead capture / chatbot / etc.]
- **Current State**: [down / degraded / workaround available]

## Timeline
- [10:30] Alert triggered: [alert name]
- [10:35] Started investigation
- [10:40] Identified root cause
- [10:50] Implemented fix
- [11:00] Monitoring recovery
- [11:15] Resolved

## Root Cause
[What happened and why]

## Resolution
[What was done to fix it]

## Prevention
[What will be done to prevent recurrence]
```

---

## 10. Monitoring Roadmap

### Phase 1: MVP (Current)
- ✅ Vercel Analytics (web vitals)
- ✅ Sentry (error tracking)
- ✅ Upstash QStash (job monitoring)
- ✅ Custom dashboard (business metrics)

### Phase 2: Growth (Post-MVP, < 100 tenants)
- ⬜ Grafana + Loki (centralized logs)
- ⬜ Prometheus + OpenTelemetry (metrics)
- ⬜ PagerDuty (on-call management)
- ⬜ Synthetics (API uptime testing)

### Phase 3: Scale (100+ tenants)
- ⬜ Datadog/New Relic (full observability)
- ⬜ Real User Monitoring (RUM)
- ⬜ Network performance monitoring
- ⬜ ML anomaly detection
