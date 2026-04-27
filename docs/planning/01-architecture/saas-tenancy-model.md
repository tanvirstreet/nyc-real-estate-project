---
layout: page
title: SaaS Multi-Tenancy Model
parent: Architecture
grand_parent: Project Planning
---

# SaaS Multi-Tenancy Model

**Project**: The Autonomous Real Estate Agent  
**Version**: 1.0  
**Last Updated**: 2026-04-22

---

## 1. Tenancy Strategy

We use a **shared-database, shared-schema** model with row-level tenant isolation. This is the most cost-effective approach for a SaaS MVP and scales well to thousands of tenants.

### Why Shared Database?
| Approach | Cost | Isolation | Complexity | Our Choice |
|---|---|---|---|---|
| Database-per-tenant | $$$$ | Maximum | High (migration management) | ✗ |
| Schema-per-tenant | $$$ | High | Medium | ✗ |
| **Shared DB + Row-Level** | **$** | **Medium** | **Low** | **✓** |

---

## 2. Tenant Model

```
Tenant
├── id (cuid)
├── name (string)              → "Smith Real Estate"
├── slug (string, unique)      → "smith-realty"
├── domain (string?, unique)   → "smith-realty.com" (custom domain)
├── subdomain (string, unique) → "smith" → smith.platform.com
├── plan (enum)                → FREE, STARTER, PRO, ENTERPRISE
├── settings (JSON)            → branding, feature flags, limits
│   ├── logo (url)
│   ├── primaryColor (#hex)
│   ├── agentName (string)
│   ├── agentPhoto (url)
│   ├── contactEmail (string)
│   ├── contactPhone (string)
│   └── featureFlags
│       ├── chatbotEnabled (bool)
│       ├── voiceAgentEnabled (bool)
│       ├── contentFactoryEnabled (bool)
│       └── socialPublishingEnabled (bool)
├── integrations (JSON, encrypted)
│   ├── hubspotApiKey
│   ├── openaiApiKey (or use platform key)
│   ├── twilioAccountSid
│   ├── twilioAuthToken
│   ├── heygenApiKey
│   └── bufferAccessToken
├── usage (computed)
│   ├── leadsThisMonth
│   ├── chatMessagesThisMonth
│   ├── contentGenerated
│   └── voiceCallMinutes
├── createdAt
└── updatedAt
```

---

## 3. Data Isolation Pattern

Every data table includes a `tenantId` foreign key:

```sql
-- Example: leads table with tenant isolation
CREATE TABLE leads (
    id         TEXT PRIMARY KEY,
    tenant_id  TEXT NOT NULL REFERENCES tenants(id),
    full_name  TEXT NOT NULL,
    email      TEXT NOT NULL,
    -- ... other fields ...
    created_at TIMESTAMPTZ DEFAULT NOW(),

    -- Composite unique: email unique per tenant, not globally
    UNIQUE(tenant_id, email)
);

-- Row-Level Security Policy (PostgreSQL)
CREATE POLICY tenant_isolation ON leads
    USING (tenant_id = current_setting('app.tenant_id')::TEXT);
```

### Application-Level Enforcement

```typescript
// middleware pattern — every DB query scoped to tenant
class TenantScopedRepository<T> {
  constructor(private tenantId: string) {}

  async findAll(where?: Partial<T>) {
    return prisma.model.findMany({
      where: { ...where, tenantId: this.tenantId }
    });
  }

  async create(data: Omit<T, 'tenantId'>) {
    return prisma.model.create({
      data: { ...data, tenantId: this.tenantId }
    });
  }
}
```

---

## 4. Tenant Resolution

Request flow: `Incoming Request → Edge Middleware → Resolve Tenant → Inject Context`

### Resolution Strategy (Priority Order)

1. **Custom Domain**: `smith-realty.com` → lookup `tenants.domain`
2. **Subdomain**: `smith.platform.com` → lookup `tenants.subdomain`
3. **Header**: `X-Tenant-ID` (for API consumers)
4. **JWT Claim**: `tenantId` embedded in auth token (authenticated routes)

```typescript
// Edge Middleware: tenant resolution
export async function middleware(request: NextRequest) {
  const hostname = request.headers.get('host') || '';
  
  let tenantSlug: string | null = null;

  // 1. Check custom domain mapping (cached in Redis)
  const customDomainTenant = await redis.get(`domain:${hostname}`);
  if (customDomainTenant) {
    tenantSlug = customDomainTenant;
  }

  // 2. Check subdomain
  if (!tenantSlug) {
    const subdomain = hostname.split('.')[0];
    if (subdomain !== 'www' && subdomain !== 'app') {
      tenantSlug = subdomain;
    }
  }

  // Inject tenant context into request headers
  if (tenantSlug) {
    const headers = new Headers(request.headers);
    headers.set('x-tenant-slug', tenantSlug);
    return NextResponse.next({ request: { headers } });
  }

  return NextResponse.next();
}
```

---

## 5. Feature Gating by Plan

| Feature | FREE | STARTER ($29/mo) | PRO ($79/mo) | ENTERPRISE |
|---|---|---|---|---|
| Landing page | ✓ | ✓ | ✓ | ✓ |
| Contact form + CRM sync | ✓ (50 leads/mo) | ✓ (500 leads/mo) | ✓ (Unlimited) | ✓ |
| AI Chatbot | ✗ | ✓ (500 msgs/mo) | ✓ (5000 msgs/mo) | ✓ (Unlimited) |
| Lead scoring | Basic | Advanced | Advanced | Custom |
| Drip campaigns | ✗ | ✓ (3 sequences) | ✓ (Unlimited) | ✓ |
| AI Voice calls | ✗ | ✗ | ✓ (50 calls/mo) | ✓ (Unlimited) |
| Content factory | ✗ | ✗ | ✓ (30 pieces/mo) | ✓ (Unlimited) |
| AI Video generation | ✗ | ✗ | ✓ (10 videos/mo) | ✓ (Unlimited) |
| Social publishing | ✗ | ✓ (3 platforms) | ✓ (5 platforms) | ✓ |
| White-label branding | ✗ | ✗ | ✗ | ✓ |
| Custom domain | ✗ | ✗ | ✓ | ✓ |
| API access | ✗ | ✗ | ✓ | ✓ |

---

## 6. Scaling Path

```
Phase 1 (MVP): Single tenant, shared everything
  → 1 Vercel project, 1 Supabase DB, 1 Redis instance
  → Cost: ~$0–20/mo

Phase 2 (Early SaaS): 10-50 tenants
  → Same infra, connection pooling, Redis caching
  → Cost: ~$50-100/mo

Phase 3 (Growth): 50-500 tenants
  → Supabase Pro, Upstash Pro, CDN for media
  → Cost: ~$200-500/mo

Phase 4 (Scale): 500+ tenants
  → Dedicated DB, managed Redis cluster, queue workers
  → Cost: ~$500-2000/mo
```
