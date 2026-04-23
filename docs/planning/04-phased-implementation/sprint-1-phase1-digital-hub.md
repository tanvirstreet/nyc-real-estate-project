# Sprint 1 — Phase 1: Digital Hub & AI Lead Capture

**Duration**: 10 working days  
**Team**: Full-Stack Developer + AI/ML Engineer  
**Prerequisites**: Sprint 0 complete

---

## Part A: Landing Page & Property Website (Days 1-5)

### Day 1-2: Hero & Property Showcase

#### Task 1.1: Hero Section
- Full-viewport hero with background image/video
- Gradient overlay (dark → transparent)
- Agent name and headline: "Your Real Estate Partner in NYC"
- Animated CTA button: "Get Your Free Home Valuation"
- Scroll-down indicator animation
- **Files**: `src/components/landing/HeroSection.tsx`, `src/components/landing/HeroSection.module.css`
- **Acceptance**: Hero loads with animation, CTA scrolls to contact form
- **Time**: 4 hours

#### Task 1.2: Property Showcase Grid
- Responsive grid: 3 columns desktop, 2 tablet, 1 mobile
- PropertyCard: image (with lazy loading), price (formatted), location, beds/baths, "View Details" link
- Hover effect: subtle lift + shadow + image zoom
- Data source: `properties.json` or database query
- **Files**: `src/components/landing/PropertyShowcase.tsx`, `src/components/landing/PropertyCard.tsx`
- **Acceptance**: 3-5 properties display, responsive, cards link to detail pages
- **Time**: 4 hours

#### Task 1.3: Property Detail Page
- Route: `/property/[id]`
- Image gallery (lightbox on click)
- Property details: price, address, beds, baths, sqft, description, features list
- Embedded Google Maps (location pin)
- Agent contact card (sidebar)
- "Schedule a Viewing" CTA
- Related properties section
- **Files**: `src/app/property/[id]/page.tsx`
- **Acceptance**: SSG-rendered, SEO meta tags, responsive
- **Time**: 6 hours

### Day 3: Contact Form & Trust Elements

#### Task 1.4: Contact Form
- Fields: Full Name (required), Email (required), Phone (required), Interest (Buyer/Seller/Both - radio), Property Preference (dropdown, optional), Message (textarea, optional)
- Client-side validation with Zod
- Loading state during submission
- Error states with field-level messages
- Auto-submit to `/api/contact` on valid submission
- **Files**: `src/components/landing/ContactForm.tsx`, `src/app/api/contact/route.ts`
- **Acceptance**: Form validates, submits, shows loading, handles errors
- **Time**: 4 hours

#### Task 1.5: Thank You Page
- Route: `/thank-you`
- Confirmation message: "Thank you! We'll contact you within 2 hours"
- Embedded calendar widget for scheduling (Calendly embed or custom)
- Secondary CTA: "Browse More Properties"
- **Files**: `src/app/thank-you/page.tsx`
- **Time**: 2 hours

#### Task 1.6: Trust & Social Proof Section
- Testimonials carousel (auto-scroll, 3 testimonials)
- "XXX Homes Sold" animated counter (count-up on scroll into view)
- Agent credentials badges (Licensed, Top Producer, etc.)
- **Files**: `src/components/landing/TestimonialSection.tsx`, `src/components/landing/TrustBadges.tsx`
- **Time**: 3 hours

### Day 4: Footer, Analytics & SEO

#### Task 1.7: Enhanced Footer
- Social media links (Instagram, LinkedIn, Facebook, TikTok) with icons
- Office address with Google Maps link
- WhatsApp direct chat button (click-to-chat URL)
- Phone number (click-to-call)
- Legal links (Privacy Policy, Terms)
- **Files**: `src/components/layout/Footer.tsx`
- **Time**: 2 hours

#### Task 1.8: Analytics Integration
- Google Analytics 4: `gtag.js` script in root layout
- HubSpot tracking pixel: script tag
- Custom events: form_submission, chatbot_opened, property_viewed, cta_clicked
- UTM parameter capture and forwarding to CRM
- **Files**: `src/lib/analytics.ts`, update `src/app/layout.tsx`
- **Time**: 2 hours

#### Task 1.9: SEO Optimization
- Dynamic meta tags per page (title, description, OG image)
- Structured data: `RealEstateAgent` schema (JSON-LD)
- Structured data: `RealEstateListing` per property page
- `sitemap.xml` generation (Next.js built-in)
- `robots.txt`
- **Files**: `src/app/sitemap.ts`, `src/app/robots.ts`, metadata in each `page.tsx`
- **Time**: 2 hours

### Day 5: Performance & Polish

#### Task 1.10: Performance Optimization
- Image optimization: `next/image` with WebP, responsive sizes
- Font optimization: `next/font` for Google Fonts (no layout shift)
- CSS optimization: critical CSS inline, defer non-critical
- Bundle analysis: `@next/bundle-analyzer`
- Target: Lighthouse Performance >90, LCP <2s
- **Time**: 3 hours

#### Task 1.11: Responsive Testing
- Test on: Mobile (375px), Tablet (768px), Desktop (1280px), Large (1440px)
- Fix any layout issues
- Test all interactive elements (form, carousel, navigation)
- **Time**: 2 hours

#### Task 1.12: Landing Page Integration Test
- Full flow: Visit → Browse properties → Fill form → Submission → Thank you page
- Verify API route receives correct data
- Verify analytics events fire
- **Time**: 1 hour

---

## Part B: AI Concierge Chatbot (Days 6-10)

### Day 6-7: Knowledge Base & RAG Engine

#### Task 1.13: Knowledge Base Preparation
- Create markdown docs covering:
  - NYC real estate market overview (pricing, trends, neighborhoods)
  - Staten Island market data (average prices, top neighborhoods)
  - Buying process guide (steps, timeline, costs)
  - Selling process guide (steps, staging, pricing strategy)
  - FAQs (financing, closing costs, inspections, etc.)
  - Agent bio and services
- **Files**: `src/data/knowledge-base/*.md` (6-8 documents)
- **Time**: 4 hours

#### Task 1.14: Embedding Pipeline
- Document loader: read markdown files
- Chunking: RecursiveCharacterTextSplitter (500 tokens, 100 overlap)
- Embedding: OpenAI `text-embedding-3-small`
- Storage: pgvector table in Supabase
- Seeding script: `prisma/seed-vectors.ts`
- **Files**: `src/lib/rag-engine.ts`, `prisma/seed-vectors.ts`
- **Time**: 4 hours

#### Task 1.15: RAG Retrieval Pipeline
```typescript
// Pseudocode
async function chatWithRAG(query: string, conversationHistory: Message[]) {
  const queryEmbedding = await embed(query);
  const relevantChunks = await pgvector.search(queryEmbedding, topK=5);
  
  const systemPrompt = `You are a knowledgeable NYC real estate assistant. 
    Use the following context to answer questions. 
    If you don't know, say so and offer to connect with the agent.
    Context: ${relevantChunks.join('\n')}`;
  
  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [
      { role: 'system', content: systemPrompt },
      ...conversationHistory,
      { role: 'user', content: query }
    ],
    stream: true
  });
  
  return response;
}
```
- **Files**: `src/lib/rag-engine.ts`, `src/lib/openai.ts`
- **Time**: 4 hours

### Day 8-9: Chat Widget UI

#### Task 1.16: Chat Widget Component
- Floating button: bottom-right corner, animated pulse
- Expand to chat panel (350px × 500px)
- Chat header: agent avatar, name, "Online" status
- Message area: scrollable, auto-scroll to bottom
- User messages: right-aligned, colored
- Bot messages: left-aligned, with avatar, markdown rendering
- Typing indicator: animated dots
- Minimize/close button
- Persistent across page navigation (state in context)
- **Files**: `src/components/chatbot/ChatWidget.tsx`, `ChatBubble.tsx`, `ChatInput.tsx`, `ChatMessage.tsx`
- **Time**: 6 hours

#### Task 1.17: Chat API Route
- `POST /api/chatbot`
- Input: `{ message, sessionId, conversationHistory }`
- Output: Streaming response (Server-Sent Events)
- Store conversation in DB (Conversation + Message models)
- Generate session ID for anonymous visitors (cookie-based)
- **Files**: `src/app/api/chatbot/route.ts`
- **Time**: 3 hours

#### Task 1.18: Chat Hook
- `useChat` custom hook managing:
  - Message state (optimistic UI)
  - Streaming response handling
  - Session persistence (localStorage)
  - Error handling and retry
  - Connection status
- **Files**: `src/hooks/useChat.ts`
- **Time**: 2 hours

### Day 10: Intelligence & Integration

#### Task 1.19: Conversation Logging
- Store all conversations in DB with:
  - Visitor ID (from cookie)
  - All messages with timestamps
  - Auto-generated conversation summary (GPT-4o at session end)
  - Intent classification captured during conversation
- **Files**: Update `src/app/api/chatbot/route.ts`
- **Time**: 2 hours

#### Task 1.20: Intent Classification
- After each AI response, classify visitor intent:
  - `BUYER` — looking to purchase
  - `SELLER` — looking to sell
  - `RENTER` — looking to rent
  - `INVESTOR` — investment inquiry
  - `GENERAL` — general question
- Store intent in conversation record
- Use for downstream lead routing (Sprint 2)
- **Files**: `src/lib/intent-classifier.ts`
- **Time**: 2 hours

#### Task 1.21: Human Escalation
- "Talk to a Human" button in chat widget
- Creates escalation record in DB
- Sends email alert to agent with conversation summary
- Displays: "An agent will be in touch shortly. You can also call us at [phone]"
- **Files**: Update `src/components/chatbot/ChatWidget.tsx`, `src/app/api/chatbot/escalate/route.ts`
- **Time**: 2 hours

#### Task 1.22: Sprint 1 Integration Testing
- E2E test: full chatbot conversation flow
- E2E test: form submission → API → DB
- Performance: chatbot first response <3s
- Mobile: chat widget usable on 375px screen
- **Time**: 2 hours

---

## Sprint 1 Deliverables Checklist

| # | Deliverable | Status |
|---|---|---|
| 1.1 | Hero section with CTA | ☐ |
| 1.2 | Property showcase grid | ☐ |
| 1.3 | Property detail page `/property/[id]` | ☐ |
| 1.4 | Contact form with validation | ☐ |
| 1.5 | Thank you page + calendar widget | ☐ |
| 1.6 | Testimonials + trust badges | ☐ |
| 1.7 | Footer with social + WhatsApp | ☐ |
| 1.8 | GA4 + HubSpot analytics | ☐ |
| 1.9 | SEO (meta, schema, sitemap) | ☐ |
| 1.10 | Performance (<2s LCP) | ☐ |
| 1.11 | Responsive design verified | ☐ |
| 1.12 | Landing page integration test | ☐ |
| 1.13 | Knowledge base documents | ☐ |
| 1.14 | Embedding pipeline + pgvector | ☐ |
| 1.15 | RAG retrieval pipeline | ☐ |
| 1.16 | Chat widget UI | ☐ |
| 1.17 | Chat API (streaming) | ☐ |
| 1.18 | useChat hook | ☐ |
| 1.19 | Conversation logging | ☐ |
| 1.20 | Intent classification | ☐ |
| 1.21 | Human escalation flow | ☐ |
| 1.22 | Sprint 1 integration tests | ☐ |
