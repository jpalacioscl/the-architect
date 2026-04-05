# Linkr

Minimalist SaaS for saving and organizing web bookmarks. Users paste a URL, get automatic metadata, search their library, and organize with tags.

## Commands

- `pnpm dev` — Start development server (http://localhost:3000)
- `pnpm build` — Production build
- `pnpm lint` — ESLint check
- `pnpm test` — Vitest unit tests
- `pnpm test:e2e` — Playwright E2E tests
- `pnpm db:push` — Push schema changes to Supabase
- `pnpm db:studio` — Open Drizzle Studio (local DB GUI)

## Tech Stack

Next.js 15 (App Router) + TypeScript + Tailwind CSS v4 + shadcn/ui + Supabase (Postgres) + Drizzle ORM + Clerk (auth) + Stripe (billing) + Vercel

## Architecture

### Directory Structure
- `src/app/(marketing)/` — Public pages (landing, pricing) — no auth required
- `src/app/(app)/` — Authenticated app — protected by Clerk middleware
- `src/app/api/` — API routes (bookmarks CRUD, search, webhooks)
- `src/components/app/` — Feature components (BookmarkCard, SearchBar, TagFilter)
- `src/components/ui/` — shadcn/ui primitives (never modify these directly)
- `src/lib/db/` — Drizzle client and schema (single source of truth)
- `src/lib/url-meta.ts` — URL metadata fetching utility

### Data Flow
- Server Components fetch initial data directly via Drizzle (no API hop)
- Client mutations go through API routes (`/api/bookmarks`, `/api/tags`)
- TanStack Query manages client-side cache and optimistic updates
- All DB queries are scoped by `userId` from Clerk's `auth()` — never trust client-provided user IDs

### Key Patterns
- **Server Components by default.** Add `"use client"` only when component needs interactivity (forms, search bar)
- **All DB queries in `src/lib/db/`** — no inline SQL elsewhere
- **Plan gate pattern:** check `user.plan === 'pro'` server-side before returning gated data; never gate on client only
- **Webhook idempotency:** all webhook handlers check if event was already processed before mutating

## Code Organization Rules

1. **One component per file.** Max 300 lines. Extract sub-components when approaching limit.
2. **Path alias:** Use `@/` for `src/` imports. Never use relative `../../` imports.
3. **No barrel exports.** Import directly: `import { BookmarkCard } from "@/components/app/BookmarkCard"`
4. **Server Components by default.** Only add `"use client"` when strictly required.
5. **Validate all API inputs with Zod.** No raw `req.body` access without schema validation.

## Design System

### Colors (CSS variables in globals.css)
- `--color-bg`: #0f0f11 (page background)
- `--color-surface`: #1a1a1f (cards, panels)
- `--color-surface-raised`: #242429 (hover states)
- `--color-border`: #2e2e35
- `--color-primary`: #6366f1
- `--color-text`: #f4f4f5
- `--color-muted`: #71717a

### Typography
- Font: Inter (variable, self-hosted via `next/font`)
- Code/URLs: JetBrains Mono
- Body: 15px / 1.6 line height

### Style
- Border radius: 6px default, 10px cards
- Dark mode only — no light mode toggle
- Spacing base: 4px
- Aesthetic: dense, information-rich, minimal decoration

## Environment Variables

| Variable | Description |
|----------|-------------|
| `DATABASE_URL` | Supabase PostgreSQL pooled connection string |
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Clerk frontend key |
| `CLERK_SECRET_KEY` | Clerk backend key |
| `CLERK_WEBHOOK_SECRET` | For verifying Clerk webhook signatures |
| `STRIPE_SECRET_KEY` | Stripe backend key |
| `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | Stripe frontend key |
| `STRIPE_WEBHOOK_SECRET` | For verifying Stripe webhook signatures |
| `STRIPE_PRO_PRICE_ID` | Price ID for the Pro subscription |
| `RESEND_API_KEY` | For transactional email |

## Non-Negotiable Rules

1. **Never trust client-provided user IDs.** Always get `userId` from `auth()` server-side.
2. **Never commit `.env.local`.** It is in `.gitignore` — verify before every commit.
3. **Verify webhook signatures.** Both Clerk and Stripe webhooks must be verified before processing.
4. **Scope every DB query by userId.** `WHERE user_id = $userId` on all bookmark/tag queries — no exceptions.
5. **Rate limit auth and write endpoints.** Add Upstash rate limiting to `/api/bookmarks` (POST) and `/api/search` from day one.
