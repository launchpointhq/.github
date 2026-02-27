# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Launchpoint is a creator marketing platform connecting brands with creators for UGC campaigns. It consists of three projects:

- **launchpoint-backend**: Next.js dashboard (brands) + API server
- **launchpoint-expo**: Expo/React Native mobile app (creators)
- **launchpoint-imessage**: iMessage integration server (AppleScript + Express)

## Backend (launchpoint-backend)

**Stack**: Next.js 16, React 19, PostgreSQL + Drizzle ORM, Firebase Auth, Stripe, PayPal, Trigger.dev v4, Sentry, PostHog, Resend (email), S3 (media)

**Code quality**: oxlint + oxfmt for linting/formatting, knip for unused deps, Vitest for tests

### API Routes (`src/app/api/`)

- `/admin` - Admin-only routes (restricted to @launchpointhq.com)
- `/auth` - Authentication endpoints
- `/buy` - Brand dashboard API
- `/v2/sell` - Creator mobile app API
- `/v2/bio` - Bio links API
- `/v2/oauth` - OAuth integration
- `/v2/leaderboard` - Leaderboard API
- `/shopify` - Shopify integration
- `/webhook` & `/webhooks` - Webhook handlers (PayPal, Stripe, etc.)
- `/public` - Public analytics/products
- `/portfolio` - Portfolio video API
- `/referral` - Referral tracking
- `/slack` - Slack integration
- `/config` - App configuration

### Key Directories

- `src/app/(authenticated)/(dashboard)` - Brand dashboard UI pages
- `src/app/(public)` - Public pages (bio, login, portfolio, share)
- `src/components/ui` - shadcn/ui components (use `npx shadcn@latest add`)
- `src/db/schema/index.ts` - Drizzle schema (~2800 lines, 200+ tables)
- `src/db/migrations/` - 236+ migrations, `relations.ts` for relationships
- `src/lib/` - 60+ domain modules (see below)
- `src/trigger/` - Trigger.dev background tasks organized by domain
- `src/scripts/` - CLI maintenance utilities
- `src/hooks/` - Custom React hooks
- `src/contexts/` - React context providers
- `emails/` - React Email templates (at project root, not in src/)

### Domain Modules (`src/lib/`)

Major modules: `auth`, `offers`, `submissions`, `campaigns`, `paid-ads`, `creators`, `clients`, `media`, `stripe`, `paypal`, `shopify`, `ai`, `bot`, `fraud-detection`, `leaderboard`, `transcription`, `social-accounts`, `social-posts`, `social-providers`, `economics`, `wallet`, `teams`, `sms`, `slack`, `storage`, `targeting`, `properties`, `portfolio`, `referral`, `onboarding`, `kashi`, `imessage`, `contacts`, `todos`, `bi`, `analytics`, `metrics`

### Trigger.dev Tasks (`src/trigger/`)

Background job domains: `analyze`, `bi`, `bot`, `campaigns`, `migrations`, `notify`, `paid-ads`, `payouts`, `scraper`, `shopify`, `social`, `sounds`, `transcription`, `validate`

### Authentication

- Firebase Auth for user authentication
- `withAuth()` wrapper on API route handlers (no middleware.ts)
- Admin guard checks for `@launchpointhq.com` email
- Two DB connections: `db` (Neon serverless for API) and `triggerDb` (node-postgres TCP for Trigger tasks)

### Multi-Profile Users

- Primary profile: Firebase UID (e.g., `abc123`)
- Sub-profiles: `{uid}_{index}` (e.g., `abc123_1`)
- Wallet operations always use `parentUid`

### Database

- PostgreSQL with Drizzle ORM
- Core entities: users, offers, submissions, payments/payouts, campaigns, campaign_videos, socials, social_accounts, user_literals (creator properties), brand_imessage_contacts, paid-ads tables, targeting/properties tables

## Mobile App (launchpoint-expo)

**Stack**: Expo 54, React Native 0.81, React 19, expo-router v6, TanStack React Query v5, Firebase Auth, Sentry

**Platform SDKs**: TikTok SDK, Facebook SDK, Crisp Chat

**Native plugins**: FFmpeg, Aubio (audio analysis), whisper.rn (speech recognition)

**Config**: New Architecture enabled, React Compiler enabled, typed routes enabled

### App Structure (`app/`)

- `(tabs)/` - Main tab navigation: home, explore, create, chat
- `onboarding/` - Onboarding flow
- `profile/` - Profile screens, payouts, settings
- `deals/` - Deal browsing and details (`[id]/`)
- `deals-post.tsx` - Deal posting
- `create/` - Content creation (music, clips, teleprompt, custom)
- `bio-link/` - Bio link features
- `leaderboard.tsx` - Creator leaderboard
- `share-stats` - Share statistics
- `home-post/` - Home posts and deal details

### Key Directories

- `components/` - Reusable components (carousel-editor, core, create, deals, inputs, media, music, navigation, posts, sheets, social, stats, tutorials, ui, utilities)
- `contexts/` - Auth, user-info, home, deal-accept, review-overlays, toast
- `lib/` - 25+ domain modules mirroring backend structure
- `hooks/` - Custom React hooks
- `plugins/` - Expo plugins (FFmpeg, TikTok SDK, Aubio)
- `patches/` - Monkeypatches for dependencies

### State Management

- TanStack React Query for server state
- React Context for app-level state (auth, user info, toasts, overlays)

## iMessage Server (launchpoint-imessage)

**Stack**: Express 4, SQLite, ngrok

- `server.js` - REST API for message operations
- `message_db.js` - SQLite database interface
- AppleScript files: `send.scpt`, `quicksend.scpt`, `getcontact.scpt`, `addcontact.scpt`
- Managed with PM2 for production deployment
