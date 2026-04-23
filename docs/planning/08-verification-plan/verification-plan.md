# Verification Plan — QA, Testing, and UAT

**Project**: The Autonomous Real Estate Agent
**Version**: 1.0
**Last Updated**: 2026-04-23

---

## 1. Testing Philosophy

This platform follows **testing best practices**:

1. **Test Pyramid**: Many unit tests, fewer integration tests, minimal E2E tests
2. **Shift Left**: Catch bugs early with unit tests and type checking
3. **Automated First**: Manual testing only for exploratory/UX validation
4. **Continuous Testing**: Tests run on every PR via CI/CD
5. **User-Centric**: UAT focuses on real-world workflows, not checkboxes

---

## 2. Testing Levels

```
           ┌──────────────┐
           │   Manual/UAT │  ← Real workflows, exploratory testing
           │    (10%)     │
           ├──────────────┤
           │   E2E Tests  │  ← Critical user journeys
           │    (20%)     │
           ├──────────────┤
           │ Integration  │  ← API contracts, database, integrations
           │    (40%)     │
           ├──────────────┤
           │  Unit Tests  │  ← Business logic, utilities, components
           │    (30%)     │
           └──────────────┘
```

**Target Coverage**: >80% for critical paths, >60% overall

---

## 3. Unit Testing

### 3.1 Tools
- **Framework**: Vitest (fast, native ESM, Jest-compatible)
- **Coverage**: c8 (built into Vitest)
- **Mocks**: Vitest mocking utilities

### 3.2 What to Unit Test

#### Business Logic
```typescript
// src/services/leadScorer.test.ts
describe('LeadScorer', () => {
  it('should score lead with budget match as HOT', () => {
    const lead = {
      budgetMatch: 0.9,
      timelineUrgency: 1.0,
      engagementLevel: 0.8,
      contactInfoCompleteness: 1.0
    }
    const result = scoreLead(lead)
    expect(result.score).toBeGreaterThan(70)
    expect(result.classification).toBe('HOT')
  })

  it('should score lead with low engagement as COLD', () => {
    const lead = {
      budgetMatch: 0.5,
      timelineUrgency: 0.3,
      engagementLevel: 0.2,
      contactInfoCompleteness: 0.6
    }
    const result = scoreLead(lead)
    expect(result.score).toBeLessThan(40)
    expect(result.classification).toBe('COLD')
  })
})
```

#### Utilities
```typescript
// src/utils/phone.test.ts
describe('formatPhoneNumber', () => {
  it('should format US phone number to E.164', () => {
    expect(formatPhoneNumber('(212) 555-1234')).toBe('+12125551234')
  })

  it('should handle international numbers', () => {
    expect(formatPhoneNumber('+44 20 7123 4567')).toBe('+442071234567')
  })

  it('should throw on invalid format', () => {
    expect(() => formatPhoneNumber('invalid')).toThrow()
  })
})
```

#### React Components
```typescript
// src/components/LeadForm.test.tsx
describe('LeadForm', () => {
  it('should render form fields', () => {
    render(<LeadForm />)
    expect(screen.getByLabelText('Email')).toBeInTheDocument()
    expect(screen.getByLabelText('Phone')).toBeInTheDocument()
  })

  it('should validate email format', async () => {
    const user = userEvent.setup()
    render(<LeadForm />)

    await user.type(screen.getByLabelText('Email'), 'invalid-email')
    await user.click(screen.getByRole('button', { name: 'Submit' }))

    expect(screen.getByText('Invalid email format')).toBeInTheDocument()
  })

  it('should submit valid form', async () => {
    const mockSubmit = vi.fn()
    const user = userEvent.setup()
    render(<LeadForm onSubmit={mockSubmit} />)

    await user.type(screen.getByLabelText('Email'), 'test@example.com')
    await user.type(screen.getByLabelText('Phone'), '(212) 555-1234')
    await user.click(screen.getByRole('button', { name: 'Submit' }))

    expect(mockSubmit).toHaveBeenCalledWith({
      email: 'test@example.com',
      phone: '+12125551234'
    })
  })
})
```

### 3.3 Unit Test Coverage Targets

| Module | Target Coverage | Rationale |
|---|---|---|
| **Lead Scoring Logic** | 100% | Critical business logic |
| **Data Validation (Zod schemas)** | 100% | Security-critical |
| **Utilities (formatting, dates)** | 90%+ | Easy to test, high reuse |
| **React Components** | 70%+ | Visual testing covers some |
| **API Route Handlers** | 80%+ | Integration tests cover more |

---

## 4. Integration Testing

### 4.1 Tools
- **Framework**: Vitest (same as unit tests)
- **Database**: Test database (PostgreSQL via Supabase project)
- **External APIs**: MSW (Mock Service Worker) for mocking

### 4.2 What to Integration Test

#### API Routes
```typescript
// tests/integration/leads-api.test.ts
describe('POST /api/v1/leads', () => {
  beforeAll(async () => {
    await setupTestDatabase()
  })

  afterAll(async () => {
    await cleanupTestDatabase()
  })

  it('should create lead and sync to HubSpot', async () => {
    const response = await fetch('/api/v1/leads', {
      method: 'POST',
      body: JSON.stringify({
        source: 'landing_page',
        contact: {
          firstName: 'John',
          lastName: 'Doe',
          email: 'john@example.com',
          phone: '+12125551234',
          preferredContact: 'email'
        }
      })
    })

    expect(response.status).toBe(201)
    const data = await response.json()

    // Verify database record
    const lead = await db.lead.findUnique({ where: { id: data.id } })
    expect(lead).not.toBeNull()
    expect(lead.email).toBe('john@example.com')

    // Verify HubSpot sync (mocked or real test account)
    const hubSpotContact = await hubSpotClient.crm.contacts.getById(lead.crmSync.hubSpotContactId)
    expect(hubSpotContact.properties.email).toBe('john@example.com')
  })

  it('should reject duplicate lead', async () => {
    // Create first lead
    await createLead({ email: 'duplicate@example.com' })

    // Try to create duplicate
    const response = await createLead({ email: 'duplicate@example.com' })
    expect(response.status).toBe(409)
    expect(response.error.code).toBe('DUPLICATE_LEAD')
  })
})
```

#### Database Operations
```typescript
// tests/integration/database.test.ts
describe('Lead Database Operations', () => {
  it('should enforce tenant isolation', async () => {
    const tenant1 = await createTenant()
    const tenant2 = await createTenant()

    const lead1 = await createLead({ tenantId: tenant1.id, email: 'lead1@example.com' })
    const lead2 = await createLead({ tenantId: tenant2.id, email: 'lead2@example.com' })

    // Query as tenant1: should not see tenant2's leads
    const leads = await db.lead.findMany({ where: { tenantId: tenant1.id } })
    expect(leads).toHaveLength(1)
    expect(leads[0].id).toBe(lead1.id)
  })

  it('should create lead with embeddings', async () => {
    const property = await createProperty({
      description: 'Modern 2BR in Williamsburg with natural light'
    })

    await generateEmbeddings(property.id)

    const embeddings = await db.propertyEmbedding.findMany({
      where: { propertyId: property.id }
    })

    expect(embeddings.length).toBeGreaterThan(0)
    expect(embeddings[0].embedding).toBeDefined()
  })
})
```

#### Third-Party Integrations
```typescript
// tests/integration/hubspot.test.ts
describe('HubSpot Integration', () => {
  it('should create contact in HubSpot', async () => {
    const lead = await createLead({
      email: 'hubspot-test@example.com',
      firstName: 'Test',
      lastName: 'User'
    })

    await syncToHubSpot(lead.id)

    const updatedLead = await db.lead.findUnique({ where: { id: lead.id } })
    expect(updatedLead.crmSync.hubSpotContactId).toBeDefined()

    // Verify in HubSpot (use test account)
    const contact = await hubSpotClient.crm.contacts.getById(
      updatedLead.crmSync.hubSpotContactId
    )
    expect(contact.properties.firstname).toBe('Test')
  })

  it('should handle HubSpot rate limits gracefully', async () => {
    // Mock rate limit response
    mockHubSpotError(429, 'Rate limit exceeded')

    await expect(syncToHubSpot(leadId)).rejects.toThrow('Rate limit exceeded')

    // Verify retry was scheduled
    const job = await getJobFromQueue('hubspot-sync', leadId)
    expect(job).toBeDefined()
  })
})
```

### 4.3 Integration Test Checklist

- [ ] Lead capture → database → HubSpot sync
- [ ] Chatbot message → OpenAI → response streaming
- [ ] Lead scoring → classification → nurture enrollment
- [ ] Content generation → HeyGen → video download → R2 storage
- [ ] Email sending → SendGrid → webhook events
- [ ] WhatsApp sending → Twilio → delivery receipt
- [ ] Voice call → Bland.ai → transcript → lead update
- [ ] Webhook handling → queue → background job processing

---

## 5. End-to-End Testing

### 5.1 Tools
- **Framework**: Playwright (cross-browser, fast, reliable)
- **Visual Regression**: Percy (optional, for UI testing)

### 5.2 Critical User Journeys

#### Journey 1: Lead Discovery and Capture
```typescript
// tests/e2e/lead-capture.spec.ts
test('lead should find property and submit inquiry', async ({ page }) => {
  // 1. Navigate to landing page
  await page.goto('https://platform.com')
  await expect(page).toHaveTitle(/Find Your Dream Home in NYC/)

  // 2. Search for properties
  await page.fill('[data-testid="search-input"]', '2BR in Brooklyn')
  await page.click('[data-testid="search-button"]')

  // 3. View property details
  await page.click('[data-testid="property-card"]:first-child')
  await expect(page.locator('[data-testid="property-title"]')).toBeVisible()

  // 4. Schedule viewing (lead capture)
  await page.click('[data-testid="schedule-viewing-button"]')

  // 5. Fill lead form
  await page.fill('[name="firstName"]', 'John')
  await page.fill('[name="lastName"]', 'Doe')
  await page.fill('[name="email"]', 'john@example.com')
  await page.fill('[name="phone"]', '(212) 555-1234')
  await page.selectOption('[name="preferredContact"]', 'email')

  // 6. Submit form
  await page.click('[data-testid="submit-button"]')

  // 7. Verify success message
  await expect(page.locator('[data-testid="success-message"]')).toContainText(
    'Thank you! An agent will contact you shortly.'
  )

  // 8. Verify agent received notification (check admin dashboard)
  await page.goto('https://platform.com/admin/dashboard')
  await page.login('agent@platform.com', 'password')
  await expect(page.locator('[data-testid="new-lead-alert"]')).toContainText('John Doe')
})
```

#### Journey 2: Chatbot Conversation
```typescript
// tests/e2e/chatbot.spec.ts
test('lead should have conversation and get captured', async ({ page }) => {
  await page.goto('https://platform.com')

  // Open chatbot
  await page.click('[data-testid="chatbot-widget"]')
  await expect(page.locator('[data-testid="chatbot-container"]')).toBeVisible()

  // Send message
  await page.fill('[data-testid="chat-input"]', 'I am looking for a 2-bedroom apartment in Brooklyn under $3000')
  await page.click('[data-testid="send-button"]')

  // Wait for AI response
  await expect(page.locator('[data-testid="chat-message"]:last-child')).toContainText(
    /Brooklyn|apartment|2-bedroom/
  )

  // Continue conversation
  await page.fill('[data-testid="chat-input"]', 'What neighborhoods do you recommend?')
  await page.click('[data-testid="send-button"]')

  // Bot asks for contact info
  await expect(page.locator('[data-testid="chat-message"]:last-child')).toContainText(
    /email|phone|contact/
  )

  // Submit contact info
  await page.fill('[data-testid="chat-input"]', 'My email is test@example.com, phone is 212-555-1234')
  await page.click('[data-testid="send-button"]')

  // Verify lead captured
  await expect(page.locator('[data-testid="lead-captured-message"]')).toBeVisible()
})
```

#### Journey 3: Content Generation and Publishing
```typescript
// tests/e2e/content-factory.spec.ts
test('agent should generate and schedule content', async ({ page }) => {
  // Login as agent
  await page.goto('https://platform.com/admin/login')
  await page.fill('[name="email"]', 'agent@platform.com')
  await page.fill('[name="password"]', 'password')
  await page.click('[data-testid="login-button"]')

  // Navigate to content factory
  await page.click('[data-testid="content-factory-nav"]')
  await expect(page).toHaveURL(/\/admin\/content/)

  // Select property
  await page.selectOption('[data-testid="property-selector"]', '123 Main St')

  // Generate video script
  await page.click('[data-testid="generate-script-button"]')
  await expect(page.locator('[data-testid="generating-indicator"]')).toBeVisible()

  // Wait for generation (max 10s)
  await expect(page.locator('[data-testid="script-preview"]')).toBeVisible({ timeout: 10000 })

  // Approve and schedule
  await page.click('[data-testid="approve-button"]')
  await page.fill('[data-testid="schedule-date"]', '2026-04-30')
  await page.selectOption('[data-testid="platform-selector"]', ['instagram', 'facebook'])
  await page.click('[data-testid="schedule-button"]')

  // Verify scheduled
  await expect(page.locator('[data-testid="success-toast"]')).toContainText('Content scheduled')
  await expect(page.locator('[data-testid="scheduled-content-list"]')).toContainText('123 Main St')
})
```

### 5.3 E2E Test Execution

**Local Development**:
```bash
# Run all E2E tests
npm run test:e2e

# Run specific test file
npm run test:e2e tests/e2e/lead-capture.spec.ts

# Run with UI (debug mode)
npm run test:e2e -- --ui

# Run in specific browser
npm run test:e2e -- --project=chromium
```

**CI/CD (GitHub Actions)**:
```yaml
- name: Run E2E tests
  run: npx playwright test
- name: Upload test results
  if: always()
  uses: actions/upload-artifact@v3
  with:
    name: playwright-report
    path: playwright-report/
```

---

## 6. User Acceptance Testing (UAT)

### 6.1 UAT Participants

| Role | Responsibilities | Count |
|---|---|---|
| **Beta Agent** | Real NYC real estate agent, daily usage | 1-2 |
| **Product Owner** | Feature validation, business goals | 1 |
| **QA Engineer** | Test plan execution, bug documentation | 1 |

### 6.2 UAT Environment

- **URL**: `https://uat.platform.com` (separate from production)
- **Data**: Seeded with realistic sample data (properties, leads)
- **Integrations**: Sandbox accounts (HubSpot, Twilio, SendGrid)
- **Access**: Shared credentials for beta testers

### 6.3 UAT Test Scenarios

#### Scenario 1: Lead Capture and Follow-Up
**Precondition**: Agent has 5 properties listed on platform

**Steps**:
1. **Agent**: Share landing page link with potential lead
2. **Lead**: Browse properties, find 2BR in Park Slope
3. **Lead**: Click "Schedule Viewing", fill form
4. **Agent**: Receive WhatsApp notification
5. **Agent**: View lead in admin dashboard
6. **Agent**: See lead scored as WARM (score: 65)
7. **Agent**: Send follow-up email via platform
8. **Lead**: Receive email, click link to view more properties

**Acceptance Criteria**:
- [ ] Lead appears in dashboard within 30 seconds
- [ ] WhatsApp notification received
- [ ] Lead score matches manual assessment
- [ ] Email sent successfully (check sent folder)
- [ ] Email content personalized (property address, lead name)

---

#### Scenario 2: Chatbot Handoff
**Precondition**: Chatbot enabled on landing page

**Steps**:
1. **Lead**: Open chatbot, ask "I need a 3BR in Manhattan under $5K"
2. **Chatbot**: Respond with 3 matching properties
3. **Lead**: Ask "What's the commute time to Grand Central?"
4. **Chatbot**: Provide transit info
5. **Lead**: Say "I want to schedule a viewing"
6. **Chatbot**: Ask for email/phone
7. **Lead**: Provide contact info
8. **Agent**: Receive notification, see chat transcript
9. **Agent**: Review lead, approve as HOT
10. **Agent**: Manual follow-up via WhatsApp

**Acceptance Criteria**:
- [ ] Chatbot answers property questions accurately
- [ ] Lead captured after contact info provided
- [ ] Chat transcript visible in dashboard
- [ ] Agent can review and approve lead
- [ ] WhatsApp message sent from platform

---

#### Scenario 3: Content Generation and Publishing
**Precondition**: Agent has 3 properties needing content

**Steps**:
1. **Agent**: Login to admin dashboard
2. **Agent**: Navigate to Content Factory
3. **Agent**: Select property "123 Main St, Brooklyn"
4. **Agent**: Click "Generate Video Script"
5. **System**: Generate script in < 2 minutes
6. **Agent**: Review script, make minor edits
7. **Agent**: Click "Generate AI Video"
8. **System**: Generate video in < 5 minutes
9. **Agent**: Preview video, approve
10. **Agent**: Schedule for Instagram and Facebook (tomorrow 10am)
11. **System**: Confirm scheduled
12. **Agent**: View calendar, see scheduled posts

**Acceptance Criteria**:
- [ ] Script generated within 2 minutes
- [ ] Script mentions property features (2BR, natural light)
- [ ] Video generation completes within 5 minutes
- [ ] Video quality acceptable for social media
- [ ] Scheduling works for both platforms
- [ ] Calendar shows scheduled posts

---

#### Scenario 4: Nurture Campaign
**Precondition**: Lead classified as WARM, enrolled in nurture sequence

**Steps**:
1. **Lead**: Captured via landing page (score: 55)
2. **System**: Enroll in "Warm Lead Nurture" sequence
3. **Day 0**: Lead receives welcome email
4. **Day 2**: Lead receives educational email "5 Things to Know Before Buying in NYC"
5. **Lead**: Clicks link in email, visits platform
6. **System**: Track engagement, bump lead score to 62
7. **Day 5**: Lead receives third email "Top 10 Neighborhoods for First-Time Buyers"
8. **Lead**: Replies "I'm interested in Park Slope"
9. **Agent**: Receives notification, replies via email
10. **System**: Pause nurture sequence (agent engaged)

**Acceptance Criteria**:
- [ ] Welcome email sent immediately
- [ ] Subsequent emails sent on schedule
- [ ] Email links work (tracking, landing pages)
- [ ] Lead score updates on engagement
- [ ] Agent receives reply notification
- [ ] Nurture sequence pauses on manual engagement

---

### 6.4 UAT Bug Reporting

**Bug Report Template**:
```markdown
## Bug Title
[Short description]

**Steps to Reproduce**:
1. Step 1
2. Step 2
3. Step 3

**Expected Behavior**:
[What should happen]

**Actual Behavior**:
[What actually happened]

**Environment**:
- Browser: [Chrome/Firefox/Safari]
- Device: [Desktop/Mobile]
- User Role: [Agent/Admin]
- Screenshots: [Attach if applicable]

**Severity**:
- [ ] Critical (blocks workflow)
- [ ] High (workaround exists)
- [ ] Medium (minor issue)
- [ ] Low (cosmetic)
```

---

## 7. Performance Testing

### 7.1 Load Testing
**Tool**: k6 (open-source, scriptable)

**Test Scenario**: 100 concurrent users browsing properties

```javascript
// tests/performance/property-browse.js
import http from 'k6/http'
import { check } from 'k6'

export let options = {
  stages: [
    { duration: '1m', target: 50 },   // Ramp up to 50 users
    { duration: '2m', target: 100 },  // Ramp up to 100 users
    { duration: '2m', target: 100 },  // Stay at 100 users
    { duration: '1m', target: 0 },    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% of requests under 500ms
    http_req_failed: ['rate<0.01'],    // Error rate < 1%
  },
}

export default function () {
  // Browse property listing
  let listRes = http.get('https://platform.com/api/v1/properties')
  check(listRes, {
    'status is 200': (r) => r.status === 200,
    'has properties': (r) => JSON.parse(r.body).properties.length > 0,
  })

  // View property details
  let propertyId = JSON.parse(listRes.body).properties[0].id
  let detailRes = http.get(`https://platform.com/api/v1/properties/${propertyId}`)
  check(detailRes, {
    'status is 200': (r) => r.status === 200,
    'has details': (r) => JSON.parse(r.body).id === propertyId,
  })

  sleep(1)
}
```

### 7.2 Performance Targets

| Endpoint | Metric | Target | Measurement |
|---|---|---|---|
| **GET /api/v1/properties** | P95 latency | < 500ms | k6 load test |
| **GET /api/v1/properties/:id** | P95 latency | < 300ms | k6 load test |
| **POST /api/v1/leads** | P95 latency | < 1000ms | k6 load test |
| **POST /api/v1/chatbot/message** | P95 latency | < 3000ms | k6 load test (includes OpenAI) |
| **Landing page** | LCP | < 2.5s | PageSpeed Insights |
| **Admin dashboard** | LCP | < 1.5s | PageSpeed Insights |

---

## 8. Security Testing

### 8.1 Automated Security Scans
- **Dependency Scanning**: Snyk or Dependabot (weekly)
- **Secret Scanning**: GitGuardian or TruffleHog (pre-commit)
- **SAST (Static Analysis)**: ESLint security plugins, CodeQL
- **Container Scanning**: Trivy (if using Docker)

### 8.2 Manual Security Testing Checklist

#### Authentication & Authorization
- [ ] Test user enumeration (login form)
- [ ] Test session hijacking (token theft)
- [ ] Test privilege escalation (agent accessing admin)
- [ ] Test password strength (weak passwords)
- [ ] Test account lockout (brute force protection)

#### Input Validation
- [ ] Test SQL injection (form fields)
- [ ] Test XSS (chatbot, comments)
- [ ] Test CSRF (API routes)
- [ ] Test file upload (malicious files)
- [ ] Test API rate limiting (exceed limits)

#### Data Security
- [ ] Test PII encryption (check database)
- [ ] Test API key exposure (check source code)
- [ ] Test tenant isolation (cross-tenant data access)
- [ ] Test webhook signature bypass
- [ ] Test access control (unauthorized endpoints)

#### Integration Security
- [ ] Test OAuth token theft
- [ ] Test API key exposure in logs
- [ ] Test third-party rate limits (DoS)
- [ ] Test webhook replay attacks
- [ ] Test integration abuse (spam)

---

## 9. Accessibility Testing

### 9.1 Tools
- **Automated**: axe-core (DevTools extension, CI/CD)
- **Manual**: Screen reader (NVDA/JAWS), keyboard navigation

### 9.2 WCAG 2.1 AA Checklist

#### Perceivable
- [ ] All images have alt text
- [ ] Color contrast ratio ≥ 4.5:1 for text
- [ ] Forms have visible labels
- [ ] Error messages are descriptive
- [ ] Content is scalable (200% zoom)

#### Operable
- [ ] All functionality accessible via keyboard
- [ ] Focus order is logical
- [ ] Skip links for navigation
- [ ] No keyboard traps
- [ ] Sufficient time for timed responses

#### Understandable
- [ ] Language declaration in HTML
- [ ] Consistent navigation
- [ ] Error identification and suggestions
- [ ] Labels and instructions clear
- [ ] Error prevention (important actions)

#### Robust
- [ ] Valid HTML
- [ ] ARIA roles correct
- [ ] Compatible with assistive technologies
- [ ] Name, role, value defined

---

## 10. Testing Execution Schedule

### Sprint 0 (Foundation)
- [ ] Set up testing infrastructure (Vitest, Playwright, CI/CD)
- [ ] Write unit tests for utilities and business logic
- [ ] Set up integration test database

### Sprint 1 (Digital Hub)
- [ ] Unit tests for React components (forms, property cards)
- [ ] Integration tests for lead capture API
- [ ] E2E tests for property browsing and lead submission

### Sprint 2 (CRM & Leads)
- [ ] Integration tests for HubSpot sync
- [ ] Integration tests for lead scoring
- [ ] E2E tests for lead dashboard and alerts

### Sprint 3 (Nurture)
- [ ] Integration tests for nurture sequences
- [ ] Integration tests for email/SMS sending
- [ ] E2E tests for nurture enrollment and engagement

### Sprint 4 (Content)
- [ ] Integration tests for content generation
- [ ] Integration tests for social publishing
- [ ] E2E tests for content factory workflow

### Sprint 5 (Polish & Launch)
- [ ] Full UAT with beta agent
- [ ] Performance testing (load tests)
- [ ] Security testing (vulnerability scan)
- [ ] Accessibility audit
- [ ] Bug fixes and refinements

---

## 11. Go-Live Checklist

### Pre-Launch (Must Pass)
- [ ] All unit tests passing (>80% coverage)
- [ ] All integration tests passing
- [ ] All E2E tests passing (critical journeys)
- [ ] Security scan shows 0 critical/high vulnerabilities
- [ ] Performance benchmarks met (all endpoints)
- [ ] UAT sign-off from beta agent
- [ ] Backup and restore tested successfully
- [ ] Monitoring and alerts configured
- [ ] Error tracking (Sentry) configured
- [ ] Rate limiting configured
- [ ] SSL certificate valid
- [ ] DNS propagated
- [ ] Domain accessible from production URL

### Launch Day
- [ ] Database migrations run successfully
- [ ] Environment variables configured
- [ ] Third-party integrations connected (HubSpot, Twilio, SendGrid)
- [ ] Webhook endpoints registered with third parties
- [ ] Seed data loaded (properties, templates)
- [ ] Admin user account created
- [ ] Smoke test (create lead, send email, test chatbot)
- [ ] On-call engineer ready

### Post-Launch (First 24 Hours)
- [ ] Monitor error rates (Sentry)
- [ ] Monitor API performance (Vercel Analytics)
- [ ] Monitor integration health (HubSpot, Twilio, SendGrid)
- [ ] Review new leads (verify capture working)
- [ ] Check email deliverability (open rates, bounces)
- [ ] Address any critical bugs immediately
- [ ] Collect feedback from beta agent
- [ ] Prepare for customer support inquiries
