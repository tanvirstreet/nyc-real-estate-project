# Resource Plan — Team, Skills, and Man-Hours

**Project**: The Autonomous Real Estate Agent
**Version**: 1.0
**Last Updated**: 2026-04-23

---

## 1. Resource Philosophy

This plan assumes a **lean, skilled team** building an MVP:

1. **Generalists over Specialists**: Full-stack devs who can handle frontend/backend/DevOps
2. **AI/ML Literacy**: At least one team member experienced with LLMs, RAG, embeddings
3. **Senior-First**: Prefer 1 senior dev over 2 juniors (velocity > headcount)
4. **Context Retention**: Minimize churn (knowledge loss is expensive)

---

## 2. Team Composition

### 2.1 MVP Team (Recommended)

| Role | Count | Seniority | Primary Responsibilities |
|---|---|---|---|
| **Full-Stack Developer** | 1-2 | Senior+ | Frontend, API, database, integrations |
| **AI/ML Engineer** | 1 | Mid-Senior | RAG, embeddings, prompt engineering, chatbot |
| **DevOps Engineer** | 0.5 | Mid | CI/CD, infrastructure, monitoring (part-time/contractor) |
| **Product Owner** | 1 | — | Requirements, priorities, stakeholder mgmt |
| **QA Engineer** | 0.5 | Mid | Testing strategy, UAT coordination (part-time/contractor) |

**Total Headcount**: 3.5-4.5 FTE (full-time equivalents)

**Minimum Viable Team** (if budget constrained):
- 1 Full-Stack Developer (senior)
- 1 Product Owner (part-time)
- Total: 1.5 FTE

**Risks of Minimum Team**:
- Slower velocity (2-3 month extension)
- Single point of failure (bus factor = 1)
- Limited AI/ML expertise (outsource chatbot/RAG)

---

### 2.2 Growth Team (Post-MVP, SaaS Launch)

| Role | Count | Seniority | When to Hire |
|---|---|---|---|
| **Full-Stack Developer** | +1-2 | Mid-Senior | 50+ tenants or $10K MRR |
| **AI/ML Engineer** | +1 | Mid-Senior | 100+ tenants or scale issues |
| **DevOps Engineer** | 1 | Senior | Production incidents or multi-region deployment |
| **Support Specialist** | 1 | Junior | 100+ tenants or customer support load |
| **Customer Success Manager** | 1 | Senior | 50+ tenants or churn concerns |

**Target Team Size**: 8-10 FTE at $50K MRR

---

## 3. Skill Requirements

### 3.1 Full-Stack Developer (Primary Role)

**Must-Have Skills**:
- TypeScript (5+ years experience)
- React/Next.js (App Router, SSR, ISR)
- PostgreSQL (Prisma or similar ORM)
- REST API design and implementation
- Git, GitHub (PR workflows, code review)
- Basic DevOps (Vercel, environment variables, CI/CD)

**Nice-to-Have Skills**:
- Serverless/edge functions
- Redis (caching, queues)
- Experience with real estate tech stack (MLS, RESO)
- Security best practices (OWASP Top 10)
- Performance optimization (Core Web Vitals)

**Daily Tasks**:
- Build React components (landing page, dashboard)
- Implement API routes (lead capture, chatbot)
- Database schema design and migrations
- Integrate third-party APIs (HubSpot, Twilio, SendGrid)
- Debug and fix bugs
- Code review and testing

---

### 3.2 AI/ML Engineer (Specialist Role)

**Must-Have Skills**:
- Python or TypeScript (language agnostic)
- LLM experience (OpenAI API, prompt engineering)
- Vector databases (Pinecone, pgvector, Weaviate)
- RAG implementation (retrieval-augmented generation)
- Embeddings and semantic search
- Natural language processing basics

**Nice-to-Have Skills**:
- LangChain or LlamaIndex experience
- Fine-tuning LLMs (for custom behavior)
- Computer vision (for property image analysis)
- Speech-to-text experience (Twilio, Bland.ai)

**Daily Tasks**:
- Build and optimize RAG pipeline
- Design prompts for chatbot, content generation
- Implement vector similarity search
- Monitor AI performance (latency, quality)
- Experiment with new AI models/techniques
- Debug AI-related issues (hallucinations, latency)

---

### 3.3 DevOps Engineer (Supporting Role)

**Must-Have Skills**:
- AWS or GCP experience (cloud infrastructure)
- Docker and containerization
- CI/CD pipelines (GitHub Actions, GitLab CI)
- Monitoring and observability (Sentry, Datadog)
- Database backups and restoration
- SSL/TLS certificates

**Nice-to-Have Skills**:
- Kubernetes (if moving to microservices)
- Terraform or Pulumi (IaC)
- Security hardening (penetration testing)
- Cost optimization (AWS/GCP billing)

**Daily Tasks**:
- Set up and maintain CI/CD pipelines
- Configure monitoring and alerting
- Manage infrastructure (Vercel, Supabase, Upstash)
- Respond to production incidents
- Optimize infrastructure costs
- Document runbooks and procedures

---

### 3.4 Product Owner (Business Role)

**Must-Have Skills**:
- Product management experience
- Stakeholder communication
- Prioritization frameworks (RICE, MoSCoW)
- Basic technical literacy (can discuss APIs, databases)
- User research and empathy

**Nice-to-Have Skills**:
- Real estate industry knowledge
- SaaS experience
- Marketing and growth
- Data analysis (SQL, analytics dashboards)
- UX/UI design sense

**Daily Tasks**:
- Gather and prioritize requirements
- Write user stories and acceptance criteria
- Manage sprint backlog and planning
- Communicate with stakeholders (agents, users)
- Review features and provide feedback
- Analyze metrics and make data-driven decisions

---

### 3.5 QA Engineer (Quality Assurance)

**Must-Have Skills**:
- Test automation (Vitest, Playwright, Cypress)
- API testing (Postman, REST Client)
- Bug reporting and documentation
- Test plan creation
- Basic SQL (for database validation)

**Nice-to-Have Skills**:
- Performance testing (k6, JMeter)
- Security testing (OWASP ZAP)
- Accessibility testing (axe-core)
- Mobile testing (iOS, Android)

**Daily Tasks**:
- Write and maintain automated tests
- Execute manual exploratory testing
- Document and report bugs
- Coordinate UAT with beta users
- Review test coverage and gaps
- Reproduce and verify bug fixes

---

## 4. Man-Hour Estimation

### 4.1 Sprint Breakdown

Based on the [Phased Implementation Plan](../04-phased-implementation/implementation-plan.md):

| Sprint | Duration | Tasks | Estimated Hours | Assumptions |
|---|---|---|---|---|
| **Sprint 0** | 3 days (24h) | Foundation setup | 24-32h | 1 senior dev |
| **Sprint 1** | 10 days (80h) | Digital Hub (landing page, chatbot) | 80-120h | 1 full-stack + 1 AI/ML |
| **Sprint 2** | 7 days (56h) | CRM & Leads (HubSpot, scoring) | 56-80h | 1 full-stack |
| **Sprint 3** | 8 days (64h) | Nurture (drip campaigns, voice) | 64-96h | 1 full-stack + 1 AI/ML |
| **Sprint 4** | 8 days (64h) | Content (factory, video, social) | 64-96h | 1 full-stack + 1 AI/ML |
| **Sprint 5** | 4 days (32h) | Polish & Launch (testing, fixes) | 32-48h | Full team |
| **TOTAL** | **40 days** | | **320-472 hours** | |

**Average**: ~8-12 hours per working day (1.5-2x FTE)

---

### 4.2 Task-Level Breakdown

#### Sprint 0: Foundation (24-32 hours)
| Task | Hours | Owner |
|---|---|---|
| Project setup (Next.js, TypeScript, ESLint) | 2h | DevOps/Full-Stack |
| Database schema design | 4h | Full-Stack |
| Supabase setup + migrations | 2h | DevOps |
| Design system + component library | 4h | Full-Stack |
| CI/CD pipeline (GitHub Actions) | 2h | DevOps |
| Authentication (NextAuth.js) | 4h | Full-Stack |
| Monitoring setup (Sentry, Vercel Analytics) | 2h | DevOps |
| README + CLAUDE.md documentation | 2h | Full-Stack |
| **Buffer** (unforeseen issues) | 2-4h | — |

#### Sprint 1: Digital Hub (80-120 hours)
| Task | Hours | Owner |
|---|---|---|
| Landing page UI | 12h | Full-Stack |
| Property listing API + database | 8h | Full-Stack |
| Property detail pages | 8h | Full-Stack |
| Property embeddings (pgvector) | 8h | AI/ML |
| Chatbot UI (widget) | 8h | Full-Stack |
| Chatbot backend (OpenAI integration) | 12h | AI/ML |
| RAG pipeline implementation | 12h | AI/ML |
| Lead capture API + forms | 8h | Full-Stack |
| Integration testing | 4h | QA |
| **Buffer** | 8-12h | — |

#### Sprint 2: CRM & Leads (56-80 hours)
| Task | Hours | Owner |
|---|---|---|
| HubSpot integration setup | 8h | Full-Stack |
| Lead scoring algorithm | 8h | AI/ML |
| Lead classification logic | 4h | Full-Stack |
| Admin dashboard UI | 12h | Full-Stack |
| Alerts (Twilio WhatsApp) | 6h | Full-Stack |
| Alerts (SendGrid email) | 4h | Full-Stack |
| Nurture sequence database schema | 4h | Full-Stack |
| Nurture enrollment logic | 6h | Full-Stack |
| Integration testing | 4h | QA |
| **Buffer** | 6-10h | — |

#### Sprint 3: Nurture (64-96 hours)
| Task | Hours | Owner |
|---|---|---|
| Drip campaign engine | 12h | Full-Stack |
| Email templates + scheduling | 8h | Full-Stack |
| Bland.ai/Vapi integration | 12h | AI/ML |
| Voice call orchestration | 8h | Full-Stack |
| Google Calendar integration | 6h | Full-Stack |
| Appointment booking flow | 8h | Full-Stack |
| Transcript analysis (AI) | 6h | AI/ML |
| Testing + QA | 4h | QA |
| **Buffer** | 8-12h | — |

#### Sprint 4: Content (64-96 hours)
| Task | Hours | Owner |
|---|---|---|
| Content generation (OpenAI) | 8h | AI/ML |
| HeyGen video integration | 12h | AI/ML |
| Video storage (R2) + retrieval | 4h | Full-Stack |
| Buffer API integration | 8h | Full-Stack |
| Social media scheduling | 8h | Full-Stack |
| Content factory UI | 12h | Full-Stack |
| Engagement metrics tracking | 4h | Full-Stack |
| Testing + QA | 4h | QA |
| **Buffer** | 8-12h | — |

#### Sprint 5: Polish & Launch (32-48 hours)
| Task | Hours | Owner |
|---|---|---|
| E2E test suite (Playwright) | 8h | QA |
| Security audit + fixes | 6h | Full-Stack |
| Performance optimization | 6h | Full-Stack |
| UAT with beta agent | 8h | Product Owner |
| Bug fixes | 6h | Full-Stack |
| Documentation + onboarding guides | 4h | Product Owner |
| Launch day preparation | 2h | DevOps |
| **Buffer** | 4-8h | — |

---

## 5. Cost Estimation

### 5.1 Labor Costs (MVP Development)

**Assumptions**:
- Location: US-based (remote)
- Rates: Market rates for 2026
- Duration: 8 weeks (40 working days)

| Role | Hours | Hourly Rate | Total |
|---|---|---|---|
| Full-Stack Developer | 240-320h | $100-150/h | $24,000-48,000 |
| AI/ML Engineer | 80-120h | $120-180/h | $9,600-21,600 |
| DevOps Engineer | 40-60h | $120-180/h | $4,800-10,800 |
| QA Engineer | 20-40h | $80-120/h | $1,600-4,800 |
| Product Owner | 40-60h | $80-120/h | $3,200-7,200 |
| **TOTAL** | | | **$43,200-92,400** |

**Budget Estimate** (typical):
- **Lean MVP**: $50K-60K (1 senior dev, part-time others)
- **Balanced Team**: $70K-80K (2 senior devs, part-time others)
- **Premium Team**: $90K-100K (2-3 senior devs, full-time support)

---

### 5.2 Infrastructure Costs (First 6 Months)

| Service | Tier | Monthly | 6-Month Total |
|---|---|---|---|
| **Vercel** | Pro | $20 | $120 |
| **Supabase** | Pro | $25 | $150 |
| **Upstash Redis** | Free → Pro | $0-20 | $0-120 |
| **Cloudflare R2** | Free | $0 | $0 |
| **OpenAI API** | Usage | $50-100 | $300-600 |
| **Twilio** | Usage | $20-50 | $120-300 |
| **SendGrid** | Free → Pro | $0-20 | $0-120 |
| **Bland.ai** | Usage | $30-50 | $180-300 |
| **HeyGen** | Credits | $50-100 | $300-600 |
| **Buffer** | Free → Pro | $0-15 | $0-90 |
| **Sentry** | Free → Team | $0-26 | $0-156 |
| **Domain** | - | $1-2 | $12 |
| **TOTAL** | | **~$200-400/mo** | **$1,200-2,400** |

**Note**: Usage-based services (OpenAI, Twilio) scale with leads/content volume.

---

### 5.3 Total MVP Cost Estimate (6 Months)

| Category | Cost Range |
|---|---|
| **Labor** (development) | $50,000-100,000 |
| **Infrastructure** (6 months) | $1,200-2,400 |
| **Third-Party Services** (6 months) | $1,000-3,000 |
| **Contingency** (15% buffer) | $7,800-15,600 |
| **TOTAL** | **$60,000-120,000** |

**Breakdown by Phase**:
- **Development (8 weeks)**: $50K-100K
- **Testing & Polish (2 weeks)**: $5K-15K
- **Launch & Support (4 weeks)**: $5K-10K

---

## 6. Hiring Plan

### 6.1 MVP Phase (Immediate Hire)

**Priority 1** (hire immediately):
1. **Full-Stack Developer** (1-2 senior devs)
   - Start date: Week 0
   - Role: Core development, architecture
   - Interview focus: TypeScript, React, PostgreSQL, APIs

**Priority 2** (hire within 2 weeks):
2. **AI/ML Engineer** (1 mid-senior)
   - Start date: Week 2
   - Role: Chatbot, RAG, content generation
   - Interview focus: LLM experience, vector DBs, embeddings

**Priority 3** (contractor/part-time):
3. **DevOps Engineer** (part-time, 10h/week)
   - Start date: Week 0
   - Role: CI/CD, infrastructure setup
   - Interview focus: Vercel, GitHub Actions, monitoring

4. **QA Engineer** (part-time, 10h/week)
   - Start date: Week 4 (before Sprint 2)
   - Role: Testing strategy, UAT coordination
   - Interview focus: Test automation, API testing

---

### 6.2 Growth Phase (Post-MVP, $10K MRR)

**Hire when**:
- 50+ paying tenants OR
- $10K+ monthly recurring revenue OR
- Technical debt blocks velocity

**Roles to hire**:
1. **Full-Stack Developer** (1 mid-level)
   - Focus: Feature development, bug fixes
   - Relieve senior devs for architecture work

2. **Customer Success Manager** (1 senior)
   - Focus: Onboarding, retention, upsell
   - Reduce churn, improve satisfaction

---

### 6.3 Scale Phase (SaaS, $50K MRR)

**Hire when**:
- 200+ paying tenants OR
- $50K+ monthly recurring revenue OR
- Preparing for Series A

**Roles to hire**:
1. **DevOps Engineer** (1 senior, full-time)
   - Focus: Infrastructure scalability, security
   - Prepare for multi-region deployment

2. **Support Specialist** (1-2 junior)
   - Focus: Tier 1 support, documentation
   - Reduce support burden on devs

3. **Marketing Manager** (1 mid-senior)
   - Focus: Growth, content marketing, SEO
   - Drive acquisition and reduce CAC

---

## 7. Training and Onboarding

### 7.1 New Developer Onboarding (Week 1)

**Day 1-2**:
- [ ] Set up development environment (Node.js, PostgreSQL, Git)
- [ ] Clone repo, install dependencies
- [ ] Review architecture documentation (docs/planning/)
- [ ] Run application locally (dev server)
- [ ] Create first PR (documentation fix)

**Day 3-4**:
- [ ] Database schema walkthrough (Prisma schema)
- [ ] API architecture review (API routes, contracts)
- [ ] Integration overview (HubSpot, Twilio, SendGrid)
- [ ] Complete small feature task (with code review)

**Day 5**:
- [ ] Join sprint planning meeting
- [ ] Pick up first real task
- [ ] Pair programming with senior dev
- [ ] Review testing strategy (unit, integration, E2E)

---

### 7.2 Cross-Training (Ongoing)

**Full-Stack Dev → AI/ML**:
- Learn RAG fundamentals (retrieval-augmented generation)
- Understand embeddings and vector similarity
- Practice prompt engineering (OpenAI playground)
- Build simple chatbot using LangChain

**AI/ML Engineer → Full-Stack**:
- Learn React/Next.js basics
- Understand REST API design
- Practice database queries (Prisma)
- Build simple API endpoint

**All Devs → DevOps**:
- Understand CI/CD pipeline (GitHub Actions)
- Learn monitoring tools (Sentry, Vercel Analytics)
- Practice basic infrastructure tasks (env vars, deployments)
- Review runbooks and incident response

---

## 8. Productivity Tips

### 8.1 Maximize Velocity

**Reduce Context Switching**:
- dedicate "focus days" (no meetings)
- batch similar tasks (API work in mornings, UI in afternoons)
- limit work-in-progress (WIP) to 2-3 tasks per dev

**Leverage AI Tools**:
- Use GitHub Copilot for boilerplate code
- Use ChatGPT/Claude for debugging and documentation
- Use AI code review tools (e.g., CodeRabbit)

**Eliminate Friction**:
- Automate repetitive tasks (scripts, CI/CD)
- Use hot reload for fast development iteration
- Pre-configure development environments (Docker)

---

### 8.2 Quality vs. Speed Trade-offs

**MVP Phase** (favor speed):
- Accept technical debt (pay down later)
- Minimize test coverage (test critical paths only)
- Use third-party services over custom solutions
- Skip non-essential features (nice-to-haves)

**Growth Phase** (balance):
- Pay down technical debt
- Increase test coverage (80%+ target)
- Optimize performance and scalability
- Refactor for multi-tenancy

**Scale Phase** (favor quality):
- SLO-based development (99.9% uptime)
- Comprehensive testing (E2E, performance, security)
- Architecture review and refactoring
- Documentation and knowledge sharing

---

## 9. Retention and Morale

### 9.1 Keep Developers Happy

**Autonomy**:
- Trust devs to make technical decisions
- Avoid micromanagement (focus on outcomes)
- Allow flexible work hours (remote-first)

**Mastery**:
- Provide learning opportunities (conferences, courses)
- Encourage experimentation (20% time for R&D)
- Share knowledge (tech talks, code reviews)

**Purpose**:
- Connect work to business impact (tenant success stories)
- Celebrate wins (ship parties, shout-outs)
- Transparent communication (company goals, metrics)

---

### 9.2 Avoid Burnout

**Realistic Sprint Goals**:
- Leave 15% buffer for unforeseen issues
- Don't overcommit (under-promise, over-deliver)
- Track velocity and adjust accordingly

**Work-Life Balance**:
- Respect evenings and weekends (no crunch >1 week)
- Encourage time off (vacation, sick days)
- Monitor for signs of burnout (irritability, disengagement)

**Recognize and Reward**:
- Bonus for launching on time
- Equity/stake in company success
- Career growth opportunities (promotion path)

---

## 10. Outsourcing Considerations

### 10.1 When to Outsource

**Good Candidates for Outsourcing**:
- UI/UX design (if no in-house designer)
- QA testing (if no in-house QA)
- Content writing (if no in-house copywriter)
- DevOps setup (one-time infrastructure setup)

**Poor Candidates for Outsourcing**:
- Core business logic (lead scoring, RAG)
- Security-sensitive features (authentication, payments)
- Performance-critical code (chatbot, AI processing)

### 10.2 Outsourcing Risks

| Risk | Mitigation |
|---|---||
| Communication delays | Overlap working hours, daily standups |
| Quality issues | Code review, acceptance criteria, testing |
| Knowledge loss | Documentation, knowledge transfer sessions |
| Dependency on vendor | Maintain ownership of code, avoid vendor lock-in |

---

## 11. Conclusion

**Recommended Team for MVP**:
- 2 senior full-stack developers (or 1 senior + 1 mid-level)
- 1 AI/ML engineer (part-time or contractor)
- 1 product owner (part-time)
- 1 DevOps/QA (shared contractor, 10h/week each)

**Total Investment**:
- **Time**: 8 weeks (40 working days)
- **Cost**: $60K-120K (labor + infrastructure + contingency)
- **Team**: 3-4 FTE (full-time equivalents)

**Success Factors**:
1. **Senior developers** (velocity > headcount)
2. **Clear requirements** (product owner role)
3. **Regular testing** (QA involvement from Sprint 2)
4. **Agile iteration** (sprint planning, retrospectives)
5. **Open communication** (daily standups, async updates)
