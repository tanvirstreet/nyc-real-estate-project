# Sprint 4 — Phase 4: The Content Factory

**Duration**: 8 working days  
**Team**: Full-Stack Developer + AI/ML Engineer  
**Prerequisites**: Sprint 1 complete (landing page), Sprint 2 preferred (admin dashboard)

> Sprint 4 can run **in parallel** with Sprint 3 if a second developer is available.

---

## Part A: Content Generation Pipeline (Days 1-4)

### Day 1-2: Listing Data Ingestion

#### Task 4.1: Listing Input Interface
- Admin page: `/admin/content`
- Two input methods:
  1. **URL Input**: Paste property listing URL (Zillow, Realtor.com, StreetEasy)
     - Scrape listing data (title, price, beds/baths, sqft, description, images)
     - Use server-side fetch + HTML parsing (cheerio)
  2. **Manual Input**: Structured form
     - Address, price, beds, baths, sqft, description, features, images upload
- Preview extracted/entered data before generating content
- **Files**: `src/app/admin/content/page.tsx`, `src/lib/listing-scraper.ts`
- **Time**: 6 hours

#### Task 4.2: Content Generation Prompts
- Design LLM prompts for each content type:

**Video Script (30-60 seconds)**:
```
System: You are a real estate content creator for a top NYC agent.
Generate a 30-60 second video script for an AI avatar narration.

Property: {{propertyData}}
Tone: {{tone}} (luxury / family-friendly / investment-focused)

Format:
- Opening hook (5 seconds)
- Key selling points (20-35 seconds)
- Call to action with agent contact (10 seconds)
- Include stage directions [SHOW IMAGE], [PAN TO], etc.
```

**Instagram Caption (≤2200 chars)**:
```
Generate an Instagram carousel caption for this property listing.
Include: compelling hook, key details, neighborhood highlights,
relevant hashtags (15-20), emoji usage, CTA.
Max 2200 characters.
```

**LinkedIn Post**:
```
Generate a professional LinkedIn post about this property.
Position the agent as a market expert.
Tone: professional, insightful, not salesy.
Include market context and why this property represents value.
```

- **Files**: `src/lib/content-prompts.ts`
- **Time**: 3 hours

#### Task 4.3: LLM Content Pipeline
- API route: `POST /api/content/generate`
- Input: property data + content type + tone
- Process:
  1. Validate input
  2. Apply appropriate prompt template
  3. Call GPT-4o
  4. Post-process (character limits, hashtag formatting)
  5. Store generated content in DB (status: DRAFT)
- Generate all 3 formats in parallel (Promise.all)
- **Files**: `src/app/api/content/generate/route.ts`, `src/lib/content-generator.ts`
- **Time**: 4 hours

### Day 3-4: Tone Variations & Review UI

#### Task 4.4: Tone Variation System
- Preset tones with distinct characteristics:
  - **Luxury**: Sophisticated language, premium lifestyle, exclusivity, elegant descriptions
  - **Family-Friendly**: Warm, welcoming, community-focused, schools/parks/safety
  - **Investment**: ROI-focused, market data, rental yields, appreciation trends
- Auto-select tone based on property characteristics:
  - Price >$2M → Luxury
  - 3+ bedrooms, suburb area → Family-Friendly
  - Multi-unit or commercial → Investment
  - Manual override available
- **Files**: `src/lib/tone-selector.ts`, update content prompts
- **Time**: 2 hours

#### Task 4.5: Content Review & Edit UI
- Admin page: `/admin/content/review`
- Features:
  - Generated content preview (all 3 formats side by side)
  - Inline text editor (edit generated content)
  - Character count (especially for Instagram 2200 limit)
  - Regenerate button (per format, with option to adjust tone/prompt)
  - Approve / Reject buttons
  - Approved content moves to publishing queue
- **Files**: `src/app/admin/content/review/page.tsx`, `src/components/admin/ContentGenerator.tsx`, `src/components/admin/ContentEditor.tsx`
- **Time**: 6 hours

#### Task 4.6: Content History
- List all generated content with:
  - Property reference
  - Content type + tone
  - Status: DRAFT / APPROVED / PUBLISHED / REJECTED
  - Generated date, published date
  - Engagement metrics (after publishing)
- Filter by status, type, date
- **Files**: `src/app/admin/content/history/page.tsx`
- **Time**: 3 hours

---

## Part B: AI Video Generation (Days 5-6)

### Day 5: Video Pipeline

#### Task 4.7: HeyGen Integration
- HeyGen API client:
  - Create video from script
  - Select AI avatar (configure default for agent)
  - Set background (property images or branded template)
  - Add branding: agent logo overlay, contact info lower-third
  - Video format: 1080×1920 (vertical for social) + 1920×1080 (horizontal)
- Async flow:
  1. Submit video generation request
  2. Receive webhook on completion
  3. Download video → upload to R2
  4. Update content record with video URL
- **Files**: `src/lib/video-generator.ts`, `src/app/api/content/video/route.ts`, `src/app/api/webhooks/heygen/route.ts`
- **Time**: 5 hours

#### Task 4.8: "Market Minute" Daily Automation
- Daily cron job (Vercel Cron, 7 AM EST):
  1. Fetch latest market data (median prices, inventory, days on market)
  2. Generate "Market Minute" script via GPT-4o
  3. Create 30-second video via HeyGen
  4. Auto-approve (or queue for review based on setting)
  5. Queue for social publishing
- Market data sources:
  - NYC OpenData API (free)
  - Cached market stats from knowledge base
  - Agent-provided weekly updates
- **Files**: `src/app/api/cron/market-minute/route.ts`, `src/lib/market-data.ts`
- **Time**: 4 hours

### Day 6: Video Management

#### Task 4.9: Video Management UI
- Admin page: `/admin/content/videos`
- Features:
  - Video gallery with thumbnails
  - Video preview player
  - Status: Generating / Ready / Published
  - Download button
  - Regenerate with different avatar/background
  - Publish to social (trigger social publisher)
- **Files**: `src/app/admin/content/videos/page.tsx`
- **Time**: 3 hours

---

## Part C: Social Media Publishing (Days 7-8)

### Day 7: Publishing Engine

#### Task 4.10: Buffer API Integration
- Buffer API client:
  - Connect social profiles (Instagram, LinkedIn, Facebook, TikTok)
  - OAuth2 flow for account connection
  - Create post with text + media (image or video)
  - Schedule post for future time
  - Fetch post analytics
- **Files**: `src/lib/social-publisher.ts`
- **Time**: 4 hours

#### Task 4.11: Platform-Specific Formatting
- Auto-adapt content per platform:
  - **Instagram**: Square/vertical image, caption with hashtags, first comment strategy for additional hashtags
  - **LinkedIn**: Horizontal image, professional tone, no hashtags in body, connection request CTA
  - **Facebook**: Flexible format, OpenGraph preview, community engagement focus
  - **TikTok**: Vertical video only, trending hashtags, short caption
- Character limits: Instagram (2200), LinkedIn (3000), Facebook (63,206), TikTok (2200)
- **Files**: `src/lib/platform-formatter.ts`
- **Time**: 3 hours

#### Task 4.12: Publishing Schedule
- Default schedule (configurable per tenant):
  - **Weekdays**: 10:00 AM, 2:00 PM, 6:00 PM EST
  - **Weekends**: 11:00 AM, 4:00 PM EST
- Queue management:
  - FIFO queue of approved content
  - One post per slot
  - Skip if queue empty
  - Manual override: post immediately
- Calendar view showing scheduled posts
- **Files**: `src/lib/publish-scheduler.ts`, `src/workers/publish-worker.ts`
- **Time**: 3 hours

### Day 8: Analytics & Testing

#### Task 4.13: Engagement Tracking
- Pull engagement metrics from Buffer API:
  - Likes, comments, shares, saves, reach, impressions
  - Poll every 6 hours via cron job
  - Store metrics per post in DB
- Feed engagement data back into lead scoring:
  - User comments on post → identify if existing lead → boost score
  - High engagement post → flag similar property listings for content generation
- **Files**: `src/lib/engagement-fetcher.ts`, `src/app/api/cron/engagement/route.ts`
- **Time**: 3 hours

#### Task 4.14: Content Calendar UI
- Admin page: `/admin/content/calendar`
- Features:
  - Monthly calendar view
  - Color-coded by platform
  - Status indicators (scheduled, published, failed)
  - Click to preview post
  - Drag to reschedule
  - Engagement metrics overlay (after publishing)
- **Files**: `src/app/admin/content/calendar/page.tsx`
- **Time**: 4 hours

#### Task 4.15: Sprint 4 Integration Testing
- E2E: Input listing → generate 3 content types → approve → schedule → publish
- E2E: Market Minute cron → auto-generate → video creation → publish
- Verify: Buffer post created with correct content and schedule
- Verify: Engagement metrics pulled and stored
- Verify: Video generated with branding
- **Time**: 3 hours

---

## Sprint 4 Deliverables Checklist

| # | Deliverable | Status |
|---|---|---|
| 4.1 | Listing input (URL scrape + manual) | ☐ |
| 4.2 | Content generation prompts (3 types) | ☐ |
| 4.3 | LLM content pipeline | ☐ |
| 4.4 | Tone variation system | ☐ |
| 4.5 | Content review & edit UI | ☐ |
| 4.6 | Content history page | ☐ |
| 4.7 | HeyGen video integration | ☐ |
| 4.8 | Market Minute daily automation | ☐ |
| 4.9 | Video management UI | ☐ |
| 4.10 | Buffer social publishing | ☐ |
| 4.11 | Platform-specific formatting | ☐ |
| 4.12 | Publishing schedule + queue | ☐ |
| 4.13 | Engagement tracking + lead scoring | ☐ |
| 4.14 | Content calendar UI | ☐ |
| 4.15 | Sprint 4 integration tests | ☐ |
