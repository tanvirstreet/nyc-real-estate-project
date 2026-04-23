---
layout: page
title: Risk Register
---

# Risk Register — Analysis and Mitigation

**Project**: The Autonomous Real Estate Agent
**Version**: 1.0
**Last Updated**: 2026-04-23

---

## 1. Risk Assessment Framework

### Risk Scoring Matrix

| Severity | Probability | Risk Score | Action Required |
|---|---|---|---|
| **Critical** | High | **9-10** | Immediate mitigation, block launch |
| **High** | Medium/High | **7-8** | Mitigation plan, monitor closely |
| **Medium** | Medium | **5-6** | Monitor, have contingency plan |
| **Low** | Low/Medium | **1-4** | Accept, review periodically |

**Scoring Formula**: `Risk Score = Severity × Probability`
- Severity: 1 (Low) to 5 (Critical)
- Probability: 1 (Rare) to 5 (Certain)

---

## 2. Risk Register

### 2.1 Technology Risks

| ID | Risk | Severity | Probability | Score | Mitigation Strategy | Owner |
|---|---|---|---|---|---|---|
| **T1** | OpenAI API downtime breaks chatbot | Critical | Medium | 8 | ✅ Circuit breaker, graceful degradation, cached responses | Backend |
| **T2** | HubSpot API changes break CRM sync | High | Medium | 6 | ✅ Version锁定, webhook monitoring, automated testing | Backend |
| **T3** | Database connection pool exhausted | Critical | Low | 5 | ✅ Connection pooling (PgBouncer), auto-scaling, alerts | DevOps |
| **T4** | Third-party integration rate limits | High | High | 8 | ✅ Per-tenant quotas, exponential backoff, queue system | Backend |
| **T5** | AI video generation costs spiral | High | Medium | 6 | ✅ Per-tenant credit limits, cost alerts, approval workflow | Product |
| **T6** | Voice AI quality poor, leads annoyed | High | Medium | 6 | ✅ Pilot testing, transcript review, opt-out option | Product |
| **T7** | RAG hallucinations give wrong info | Medium | Medium | 4 | ✅ Source citations, confidence scores, human review | AI/ML |
| **T8** | pgvector performance degrades at scale | Medium | Low | 2 | ✅ HNSW indexing, benchmark at 100K vectors, migration plan | Backend |
| **T9** | Vercel cold starts slow API responses | Low | Medium | 2 | ✅ Serverless optimization, edge functions, caching | Backend |
| **T10** | Redis queue failure loses jobs | High | Low | 3 | ✅ BullMQ persistence, job retries, dead letter queue | Backend |

---

### 2.2 Security Risks

| ID | Risk | Severity | Probability | Score | Mitigation Strategy | Owner |
|---|---|---|---|---|---|---|
| **S1** | PII data breach (leads, contacts) | Critical | Low | 5 | ✅ Encryption at rest, encryption in transit, RBAC, audit logs | Security |
| **S2** | API keys exposed in code/repository | Critical | Low | 5 | ✅ Pre-commit hooks (secrets detection), env vars only, GitGuardian | DevOps |
| **S3** | SQL injection via user inputs | Critical | Low | 5 | ✅ Prisma ORM (parametrized queries), input validation (Zod), lints | Backend |
| **S4** | Cross-tenant data leakage | Critical | Medium | 8 | ✅ Row-level security, tenant isolation checks, integration tests | Security |
| **S5** | Webhook signature bypass | High | Low | 3 | ✅ HMAC verification, IP whitelisting, replay attack prevention | Security |
| **S6** | OAuth token theft | High | Low | 3 | ✅ Short-lived tokens, refresh token rotation, secure storage | Security |
| **S7** | Brute force login attacks | Medium | High | 6 | ✅ Rate limiting (Upstash), account lockout, MFA (future) | Security |
| **S8** | XSS in chatbot widget | Medium | Low | 2 | ✅ Content Security Policy, input sanitization, React escaping | Frontend |
| **S9** | CSRF on API routes | Medium | Low | 2 | ✅ SameSite cookies, CSRF tokens, origin checks | Backend |
| **S10** | Dependabot alerts for vulnerabilities | Medium | Medium | 4 | ✅ Dependabot enabled, weekly security updates, Snyk scanning | DevOps |

---

### 2.3 Operational Risks

| ID | Risk | Severity | Probability | Score | Mitigation Strategy | Owner |
|---|---|---|---|---|---|---|
| **O1** | Lead data sync fails, agents double-book | High | Medium | 6 | ✅ Idempotent sync operations, duplicate detection, alerts | Backend |
| **O2** | Email/SPAM filters block nurture emails | Medium | High | 6 | ✅ SPF/DKIM/DMARC setup, warm-up sending domain, monitor bounce rates | Ops |
| **O3** | WhatsApp business account banned | High | Low | 3 | ✅ Follow WhatsApp ToS, opt-in only, easy opt-out | Ops |
| **O4** | Cloudflare R2 egress fees spike | Medium | Low | 2 | ✅ Cost monitoring, usage alerts, CDN caching | DevOps |
| **O5** | Supabase hits free tier limits | Medium | Medium | 4 | ✅ Monitoring alerts, auto-scaling plan, backup provider ready | DevOps |
| **O6** | Vercel build time exceeds limit | Low | Medium | 2 | ✅ Optimize build, parallel builds, cache dependencies | DevOps |
| **O7** | Domain expires or DNS issues | High | Low | 3 | ✅ Auto-renew, multiple DNS providers, health checks | Ops |
| **O8** | SSL certificate expires | High | Low | 3 | ✅ Auto-renew (Let's Encrypt), monitoring alerts | Ops |
| **O9** | Backup restoration fails | Critical | Low | 5 | ✅ Quarterly restore drills, backup validation, multiple locations | Ops |
| **O10** | On-call engineer unavailable | Medium | Medium | 4 | ✅ Primary + secondary on-call, escalation policy, runbooks | Ops |

---

### 2.4 Business Risks

| ID | Risk | Severity | Probability | Score | Mitigation Strategy | Owner |
|---|---|---|---|---|---|---|
| **B1** | Agent loses leads due to platform bugs | Critical | Medium | 8 | ✅ Extensive testing, gradual rollout, insurance/fund | Product |
| **B2** | Low lead conversion rates → churn | High | Medium | 6 | ✅ Onboarding support, nurture optimization, best practices | Product |
| **B3** | Competing platform launches first | High | High | 8 | ✅ Speed to market, unique features (AI voice, video) | Product |
| **B4** | OpenAI pricing makes platform unprofitable | High | Low | 3 | ✅ Usage-based pricing, pass-through costs, alternative models | Business |
| **B5** | Third-party integration pricing changes | Medium | Medium | 4 | ✅ Fallback providers, contract negotiation, price alerts | Business |
| **B6** | Legal/regulatory changes (AI, data privacy) | High | Low | 3 | ✅ GDPR compliance, legal review, adaptable architecture | Legal |
| **B7** | Agent provides negative feedback on chatbot | Medium | High | 6 | ✅ Continuous training, feedback loop, human handover | Product |
| **B8** | Platform scales poorly (performance issues) | High | Medium | 6 | ✅ Load testing, architecture review, scaling plan | Backend |
| **B9** | Tenant onboarding is too complex | Medium | High | 6 | ✅ Simplified signup, tutorials, templates, support | Product |
| **B10** | Customer support demand overwhelms solo dev | Medium | Medium | 4 | ✅ Self-service docs, chatbot support, prioritize bugs | Support |

---

### 2.5 Compliance & Legal Risks

| ID | Risk | Severity | Probability | Score | Mitigation Strategy | Owner |
|---|---|---|---|---|---|---|
| **C1** | GDPR violation (data retention/deletion) | Critical | Low | 5 | ✅ Data export API, right to deletion, audit logs, consent tracking | Legal |
| **C2** | TCPA violation (SMS/WhatsApp consent) | High | Medium | 6 | ✅ Explicit opt-in, easy opt-out, consent logging, legal review | Legal |
| **C3** | Fair Housing Act violations (AI bias) | High | Low | 3 | ✅ Regular bias audits, transparency, human review | Legal |
| **C4** | MLS data scraping violation | Medium | Medium | 4 | ✅ Use official MLS API, terms compliance, data licensing | Legal |
| **C5** | AI-generated content copyright issues | Medium | Low | 2 | ✅ Platform owns content, indemnify agents, ToS clarification | Legal |
| **C6** | Email marketing law violations (CAN-SPAM) | Medium | Low | 2 | ✅ Physical address, unsubscribe link, consent tracking | Legal |
| **C7** | Accessibility non-compliance (ADA) | Medium | Medium | 4 | ✅ WCAG 2.1 AA compliance, accessibility audit | Frontend |
| **C8** | Data residency requirements (EU, etc.) | Low | Low | 1 | ✅ Supabase regions available, tenant region selection | DevOps |

---

## 3. Risk Mitigation Status

### ✅ Mitigated (Implemented)
- **T1**: Circuit breaker implemented for OpenAI API
- **T4**: Per-tenant rate limiting via Upstash
- **S2**: GitGuardian scanning enabled
- **S3**: Prisma ORM with parametrized queries
- **S4**: Row-level security implemented
- **O2**: SPF/DKIM/DMARC configured
- **O9**: Quarterly backup restore drills scheduled
- **C1**: GDPR data export/deletion APIs implemented

### 🔄 In Progress (Partially Mitigated)
- **T2**: HubSpot API version锁定 (in testing)
- **T3**: Connection pooling (PgBouncer evaluation)
- **B1**: Insurance fund (researching options)
- **C2**: TCPA consent review (legal consultation)

### ⏳ Planned (Not Started)
- **T8**: pgvector performance benchmarking
- **B4**: Alternative AI model evaluation (Claude, Llama)
- **C3**: Fair Housing bias audit framework
- **C7**: WCAG 2.1 AA accessibility audit

---

## 4. Risk Response Strategies

### 4.1 Avoid (Eliminate Risk)
- **S2**: Never commit API keys to Git (pre-commit hooks)
- **S3**: Never build raw SQL queries (Prisma only)
- **O7**: Auto-renew domains (manual payment oversight)

### 4.2 Transfer (Insurance/SLAs)
- **B1**: Professional liability insurance
- **O9**: Rely on Supabase/Vercel SLAs for infrastructure
- **T5**: Pass through AI video generation costs to tenants

### 4.3 Mitigate (Reduce Probability/Impact)
- **T1**: Circuit breaker reduces impact of OpenAI downtime
- **S4**: Row-level security reduces cross-tenant leakage probability
- **B2**: Onboarding support reduces churn probability

### 4.4 Accept (Acknowledge and Monitor)
- **T9**: Accept cold starts for cost savings (monitor user impact)
- **C5**: Accept low probability of copyright issues (monitor legal landscape)
- **C8**: Accept data residency compliance (monitor new regulations)

---

## 5. Key Performance Indicators (Risk Metrics)

### 5.1 Technology Health
- **API Success Rate**: Target >99.5%
- **Integration Uptime**: Target >99%
- **Database Connection Pool Usage**: Target <80%
- **Error Rate**: Target <0.5%

### 5.2 Security Posture
- **Vulnerability Remediation Time**: Target <7 days
- **Failed Security Audits**: Target 0
- **Unauthorized Access Attempts**: Monitor and alert

### 5.3 Operational Efficiency
- **Lead Sync Failure Rate**: Target <0.1%
- **Email Bounce Rate**: Target <2%
- **Backup Restoration Success**: Target 100%
- **On-Call Response Time**: Target <15 min (P0)

### 5.4 Business Viability
- **Lead Conversion Rate**: Monitor trend (target >5%)
- **Customer Churn Rate**: Target <5% monthly
- **NPS Score**: Target >50
- **Platform Uptime**: Target >99.9%

---

## 6. Risk Review Process

### 6.1 Frequency
- **Weekly**: Review new critical/high risks (standup)
- **Monthly**: Full risk register review (team meeting)
- **Quarterly**: Risk assessment refresh (planning session)
- **Annually**: External security audit (if budget allows)

### 6.2 Risk Review Agenda
1. Review new risks identified since last meeting
2. Update mitigation status for existing risks
3. Re-score risks based on new information
4. Assign action items and owners
5. Schedule follow-up review

### 6.3 Risk Escalation Path
```
Individual Contributor (identifies risk)
  ↓
Team Lead (assesses, assigns severity)
  ↓
Engineering Manager (approves mitigation plan)
  ↓
CTO/Founder (approves budget for mitigation)
  ↓
Execute Mitigation
```

---

## 7. Crisis Management Playbooks

### 7.1 Data Breach Response
1. **Identify**: Confirm scope (which tenants affected)
2. **Contain**: Disable compromised accounts, revoke API keys
3. **Notify**: Legal counsel, affected tenants, authorities (if required)
4. **Investigate**: Forensic analysis, root cause identification
5. **Remediate**: Patch vulnerability, restore from backup if needed
6. **Review**: Post-incident review, update prevention measures

### 7.2 Platform Outage Response
1. **Detect**: Alert triggered (Sentry/UptimeRobot)
2. **Assess**: Determine scope (partial/total outage)
3. **Communicate**: Status page update, tenant notification
4. **Fix**: Rollback deployment, scale infrastructure, fix bug
5. **Verify**: Confirm fix, monitor for recurrence
6. **Postmortem**: Document incident, blameless analysis

### 7.3 Integration Failure Response
1. **Detect**: Health check fails (3 consecutive failures)
2. **Disable**: Auto-disable integration via circuit breaker
3. **Communicate**: Alert admin, notify affected tenants
4. **Debug**: Investigate logs, contact third-party support
5. **Fix**: Update API version, fix integration bug
6. **Re-enable**: Gradual rollout, monitor health

---

## 8. Insurance and Liability Coverage

### 8.1 Recommended Insurance
| Type | Coverage | Estimated Cost | Priority |
|---|---|---|---|
| **General Liability** | Third-party bodily injury, property damage | $300-500/year | Medium |
| **Professional Liability** | Errors/omissions in professional services | $500-1,000/year | High |
| **Cyber Liability** | Data breach response, notification costs | $800-1,500/year | High |
| **Business Interruption** | Lost revenue due to downtime | Varies | Low (for MVP) |

### 8.2 Service Agreements
- **SLA with Tenants**: Define uptime guarantees, credit for downtime
- **Third-Party SLAs**: Rely on Supabase/Vercel SLAs for infrastructure
- **Limitation of Liability**: Cap liability in ToS (legal review required)

---

## 9. Risk Dependencies

### Critical Path Risks (Block Launch)
- **S4**: Cross-tenant data leakage
- **S1**: PII data breach protection
- **T1**: OpenAI API downtime mitigation
- **B1**: Agent loses leads due to platform bugs

### Non-Critical Risks (Can Defer)
- **T8**: pgvector performance (monitor at scale)
- **C7**: Accessibility compliance (Phase 2)
- **C8**: Data residency (only if EU tenants)

---

## 10. Lessons Learned (Post-Launch)

### Track and Document
- Every incident postmortem
- Near-miss scenarios
- False alarm alerts (refine thresholds)
- Successful risk mitigations (repeat for similar risks)

### Continuous Improvement
- Update risk register after each incident
- Refine severity/probability scores based on actual data
- Share lessons learned with team
- Improve playbooks based on real incidents
