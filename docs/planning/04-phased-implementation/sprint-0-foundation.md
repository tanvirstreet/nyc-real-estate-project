---
layout: page
title: Sprint 0 - Project Foundation
parent: Implementation Plan
grand_parent: Project Planning
nav_order: 2
---

# Sprint 0 — Project Foundation

**Duration**: 3 working days  
**Team**: Full-Stack Developer + DevOps  
**Prerequisites**: Node.js 20+, Git, GitHub account, Vercel account, Supabase account

---

## Day 1: Project Initialization

### Task 0.1: Initialize Next.js Project
- **Command**: `npx -y create-next-app@latest ./ --typescript --eslint --app --src-dir --import-alias "@/*" --use-npm`
- **Verify**: `npm run dev` starts successfully on localhost:3000
- **Time**: 30 min

### Task 0.2: Configure TypeScript Strictly
- Enable `strict: true`, `noUncheckedIndexedAccess: true` in `tsconfig.json`
- Add path aliases: `@/components/*`, `@/lib/*`, `@/types/*`
- **Time**: 15 min

### Task 0.3: Install Core Dependencies
```bash
# Production
npm install @prisma/client zod openai swr

# Development
npm install -D prisma vitest @testing-library/react prettier husky lint-staged
```
- **Time**: 15 min

### Task 0.4: Configure Code Quality
- Set up Prettier config (`.prettierrc`)
- Configure ESLint rules (`.eslintrc.json`)
- Set up Husky pre-commit hooks
- Configure lint-staged
- **Time**: 30 min

### Task 0.5: Environment Setup
- Create `.env.local` with all placeholder values
- Create `.env.example` (documented, no secrets)
- Add `.env.local` to `.gitignore`
- **Time**: 15 min

### Task 0.6: Git & GitHub Setup
- Initialize repo, create `.gitignore`
- Push to GitHub
- Set up branch protection rules (`main` requires PR)
- **Time**: 15 min

---

## Day 2: Database & Design System

### Task 0.7: Prisma + Supabase Setup
- `npx prisma init`
- Connect to Supabase PostgreSQL
- Create initial schema (all models from data architecture)
- Run `npx prisma db push`
- Create seed script with sample properties
- **Time**: 2 hours

### Task 0.8: Design System (globals.css)
```css
/* Core design tokens */
:root {
  /* Colors — NYC luxury real estate palette */
  --color-primary: #1a1a2e;          /* Deep navy */
  --color-secondary: #c9a96e;        /* Gold accent */
  --color-accent: #e8d5b7;           /* Warm cream */
  --color-bg: #0f0f1a;               /* Dark background */
  --color-surface: #1a1a2e;          /* Card surfaces */
  --color-text: #f5f5f5;             /* Primary text */
  --color-text-muted: #a0a0b0;       /* Secondary text */
  --color-success: #10b981;
  --color-warning: #f59e0b;
  --color-danger: #ef4444;
  --color-hot: #ef4444;
  --color-warm: #f59e0b;
  --color-cold: #3b82f6;

  /* Typography */
  --font-heading: 'Outfit', sans-serif;
  --font-body: 'Inter', sans-serif;
  --font-mono: 'JetBrains Mono', monospace;

  /* Spacing scale */
  --space-xs: 0.25rem;
  --space-sm: 0.5rem;
  --space-md: 1rem;
  --space-lg: 1.5rem;
  --space-xl: 2rem;
  --space-2xl: 3rem;
  --space-3xl: 4rem;

  /* Border radius */
  --radius-sm: 0.375rem;
  --radius-md: 0.75rem;
  --radius-lg: 1rem;
  --radius-xl: 1.5rem;
  --radius-full: 9999px;

  /* Shadows */
  --shadow-sm: 0 1px 3px rgba(0,0,0,0.3);
  --shadow-md: 0 4px 14px rgba(0,0,0,0.4);
  --shadow-lg: 0 10px 40px rgba(0,0,0,0.5);
  --shadow-glow: 0 0 30px rgba(201,169,110,0.15);

  /* Transitions */
  --transition-fast: 150ms ease;
  --transition-base: 250ms ease;
  --transition-slow: 400ms ease;
}
```
- Import Google Fonts (Inter, Outfit) in layout
- Create reusable CSS classes for buttons, cards, inputs, badges
- **Time**: 3 hours

### Task 0.9: Base UI Components
- `Button.tsx` — primary, secondary, outline, ghost variants + sizes
- `Input.tsx` — text, email, phone, select with validation states
- `Modal.tsx` — accessible dialog component
- `Toast.tsx` — success/error/info notifications
- `Spinner.tsx` — loading indicator
- **Time**: 2 hours

---

## Day 3: Layout, CI/CD & Documentation

### Task 0.10: App Layout
- Root `layout.tsx` — font imports, analytics scripts, metadata
- `Header.tsx` — agent logo, navigation, CTA button
- `Footer.tsx` — social links, office address, legal links
- Mobile responsive navigation (hamburger menu)
- **Time**: 2 hours

### Task 0.11: Vercel Deployment
- Connect GitHub repo to Vercel
- Configure environment variables in Vercel dashboard
- Verify automatic deploys on push to `main`
- Verify preview deploys on PRs
- **Time**: 30 min

### Task 0.12: CI/CD Pipeline
```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npx tsc --noEmit
      - run: npm run lint
      - run: npm run test
```
- **Time**: 30 min

### Task 0.13: README & Documentation
- Project overview, setup instructions
- Architecture summary with diagram links
- Contributing guidelines
- **Time**: 1 hour

### Task 0.14: Sprint 0 Verification
- [ ] `npm run dev` works
- [ ] `npm run build` succeeds
- [ ] `npm run test` passes
- [ ] Vercel deployment live
- [ ] Database connected and seeded
- [ ] Design system visible in browser
- **Time**: 30 min

---

## Sprint 0 Deliverables Checklist

| # | Deliverable | Status |
|---|---|---|
| 0.1 | Next.js 15 project initialized | ☐ |
| 0.2 | TypeScript strict mode | ☐ |
| 0.3 | Core packages installed | ☐ |
| 0.4 | ESLint + Prettier + Husky | ☐ |
| 0.5 | Environment variables template | ☐ |
| 0.6 | GitHub repo + branch protection | ☐ |
| 0.7 | Prisma schema + Supabase connection | ☐ |
| 0.8 | Design system (CSS tokens + classes) | ☐ |
| 0.9 | Base UI components (5 components) | ☐ |
| 0.10 | App layout (Header + Footer) | ☐ |
| 0.11 | Vercel deployment pipeline | ☐ |
| 0.12 | GitHub Actions CI | ☐ |
| 0.13 | README documentation | ☐ |
