# Monitoring Stack Architecture

## Complete Monitoring Ecosystem

```mermaid
graph TB
    subgraph "Application Layer"
        APP["Next.js App<br/>API Routes + Services"]
    end

    subgraph "Error Tracking"
        SENTRY["Sentry<br/>Error Tracking + Performance"]
    end

    subgraph "Web Analytics"
        VERCEL["Vercel Analytics<br/>Web Vitals + Page Views"]
        GA4["Google Analytics 4<br/>User Behavior"]
    end

    subgraph "Infrastructure Monitoring"
        SUPABASE["Supabase Dashboard<br/>DB Health + Metrics"]
        UPSTASH["Upstash Dashboard<br/>Redis Health + Usage"]
        R2["Cloudflare R2<br/>Storage Analytics"]
    end

    subgraph "Job Monitoring"
        QSTASH["Upstash QStash<br/>Scheduled Jobs"]
    end

    subgraph "Business Metrics"
        HUBSPOT["HubSpot Reports<br/>CRM Analytics"]
        CUSTOM["Custom Dashboard<br/>Built with Next.js"]
    end

    subgraph "Uptime Monitoring"
        UPTIME["UptimeRobot / Pingdom<br/>Uptime Checks"]
    end

    subgraph "Logging"
        VERCEL_LOGS["Vercel Logs<br/>Application Logs"]
        AUDIT["PostgreSQL<br/>Audit Log Table"]
    end

    subgraph "Alerting"
        EMAIL["Email Alerts<br/>On-call Rotation"]
        SLACK["Slack #alerts<br/>Team Notifications"]
        SMS["SMS/Pager<br/>Critical Alerts"]
    end

    %% Application Telemetry
    APP -->|"Errors + Unhandled Exceptions"| SENTRY
    APP -->|"Performance Transactions"| SENTRY
    APP -->|"Page Views + Web Vitals"| VERCEL
    APP -->|"User Events + Conversions"| GA4

    %% Infrastructure Telemetry
    APP -->|"Queries + Connections"| SUPABASE
    APP -->|"Commands + Queues"| UPSTASH
    APP -->|"Media Operations"| R2

    %% Job Monitoring
    APP -->|"Job Status + Failures"| QSTASH

    %% Business Metrics
    APP -->|"Lead/Deal Metrics"| HUBSPOT
    APP -->|"Custom Metrics"| CUSTOM

    %% Uptime Checks
    UPTIME -->|"HTTP Checks every 5min"| APP

    %% Logging
    APP -->|"Structured Logs"| VERCEL_LOGS
    APP -->|"Audit Events"| AUDIT

    %% Alert Routing
    SENTRY -->|"P0: Critical Errors"| EMAIL
    SENTRY -->|"P1: High Priority"| SLACK
    SENTRY -->|"P0 Only"| SMS

    UPTIME -->|"Site Down Alerts"| EMAIL
    UPTIME -->|"Site Down Alerts"| SMS

    SUPABASE -->|"Health Alerts"| SLACK
    UPSTASH -->|"Quota Warnings"| SLACK
    QSTASH -->|"Job Failures"| SLACK
    CUSTOM -->|"Business Alerts"| EMAIL

    %% Styling
    classDef app fill:#3b82f6,stroke:#1e40af,color:#fff
    classDef tracking fill:#ef4444,stroke:#b91c1c,color:#fff
    classDef analytics fill:#10b981,stroke:#047857,color:#fff
    classDef infra fill:#f59e0b,stroke:#d97706,color:#fff
    classDef business fill:#8b5cf6,stroke:#6d28d9,color:#fff
    classDef uptime fill:#ec4899,stroke:#be185d,color:#fff
    classDef logs fill:#6366f1,stroke:#4f46e5,color:#fff
    classDef alert fill:#dc2626,stroke:#991b1b,color:#fff

    class APP app
    class SENTRY tracking
    class VERCEL,GA4 analytics
    class SUPABASE,UPSTASH,R2,QSTASH infra
    class HUBSPOT,CUSTOM business
    class UPTIME uptime
    class VERCEL_LOGS,AUDIT logs
    class EMAIL,SLACK,SMS alert
```

---

## Monitoring Data Flow

### 1. Error Tracking Flow
```
Application Error
  ↓
Sentry SDK (Client + Server)
  ↓
Sentry Service
  ↓
Alert Evaluation (Severity)
  ↓
Route to Alert Channel (Email/Slack/SMS)
```

### 2. Performance Monitoring Flow
```
API Request / Page Load
  ↓
Vercel Edge / Sentry Traces
  ↓
Aggregate Metrics (p50, p95, p99)
  ↓
Dashboard Display
  ↓
Threshold Check
  ↓
Alert if P95 > Target
```

### 3. Business Metrics Flow
```
Lead Created / Content Published
  ↓
Database Record
  ↓
Custom Analytics Query
  ↓
Dashboard Visualization
  ↓
Trend Analysis
  ↓
Alert if Anomaly Detected
```

---

## Monitoring Tool Costs

### MVP (Free Tiers Only)
| Tool | Tier | Cost | Limits |
|---|---|---|---|
| Sentry | Free | $0 | 5K events/mo, 1 user |
| Vercel Analytics | Free | $0 | Included with Vercel |
| Google Analytics 4 | Free | $0 | Unlimited |
| Supabase Dashboard | Free | $0 | Included with Supabase |
| Upstash Dashboard | Free | $0 | Included with Upstash |
| UptimeRobot | Free | $0 | 50 monitors, 5-min intervals |
| Custom Dashboard | Self-built | $0 | Built with Next.js |
| **Total** | | **$0/month** | |

### Growth Phase (Pro Tiers)
| Tool | Tier | Cost | Upgrade Trigger |
|---|---|---|---|
| Sentry | Team | $26/mo | >5K errors/mo or team expansion |
| Vercel Analytics | Pro | $20/mo | Vercel Pro plan |
| Supabase | Pro | $25/mo | >500MB DB or production deployment |
| Upstash | Pro | ~$20/mo | >10K Redis commands/day |
| Datadog/New Relic | Pro | ~$100/mo | Need full observability |
| PagerDuty | Standard | ~$25/user/mo | 24/7 on-call rotation |
| **Total** | | **~$200/month** | Multi-tenant SaaS launch |

---

## Alert Channel Configuration

### 1. Email Alerts (Primary)
- **Recipient**: On-call engineer rotation
- **Sender**: `noreply@platform.com`
- **Subject Format**: `[P0] Lead capture API errors spike - 23 errors in 1min`
- **Content**: Error summary, link to Sentry, affected tenants

### 2. Slack Alerts (Team Awareness)
- **Channel**: `#alerts`
- **Bot**: Platform Alert Bot
- **Format**:
  ```
  🚨 [P1] HubSpot Integration Down

  Service: HubSpot CRM Sync
  Status: ERROR (3 consecutive failures)
  Last Error: Rate limit exceeded (429)
  Started: 10 minutes ago

  Actions: Auto-disabled, investigating...
  ```

### 3. SMS/Pager Alerts (Critical Only)
- **Provider**: Twilio Programmable SMS
- **Trigger**: P0 alerts only
- **Recipients**: On-call engineer
- **Format**: `[P0] CRITICAL: Site down - https://platform.com returning 503. Respond immediately.`

---

## Dashboard Access Control

### Role-Based Access

| Role | Dashboards Access | Actions |
|---|---|---|
| **SUPER_ADMIN** | All dashboards + Sentry + Vercel | All actions |
| **AGENT_ADMIN** | Custom business dashboard only | View tenant metrics |
| **AGENT** | Custom business dashboard only | View-only (no filters) |

### Custom Dashboard (Built-in)
- **URL**: `/dashboard/analytics`
- **Authentication**: Required (NextAuth.js)
- **Tenant Scope**: Users see only their tenant's data
- **Metrics**: Lead funnel, engagement, content performance

---

## Monitoring Checklist

### Daily
- [ ] Check Sentry error dashboard
- [ ] Review failed job queue (QStash)
- [ ] Check integration health status
- [ ] Review lead capture volume vs. baseline

### Weekly
- [ ] Review performance trends (Vercel Analytics)
- [ ] Check SLO compliance (API success rate)
- [ ] Review storage usage (DB + R2)
- [ ] Audit log review (suspicious activity)

### Monthly
- [ ] SLO review meeting (missed targets?)
- [ ] Cost review (monitoring tools spend)
- [ ] Incident retrospective (if any SEV1+)
- [ ] Monitoring tool evaluation (upgrade needed?)

---

## Future Monitoring Enhancements

### Phase 2: Growth (Post-MVP)
- **Synthetics**: API uptime testing (Catch failures before users)
- **RUM (Real User Monitoring)**: Browser performance, device breakdown
- **Log Aggregation**: Centralized logs (Loki) for easier querying
- **Metrics**: Prometheus + Grafana for custom dashboards

### Phase 3: Scale (SaaS)
- **Distributed Tracing**: OpenTelemetry for request flow across services
- **Anomaly Detection**: ML-based alerting (reduce false positives)
- **Canary Monitoring**: Deploy to subset of users, monitor before full rollout
- **Churn Prediction**: Predict tenant churn based on usage patterns
