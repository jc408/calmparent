# CalmParent (Phase 1 MVP)

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/jc408/calmparent)

CalmParent turns school-related Gmail messages into a daily “What Matters Today” list and a calm weekly digest. Phase 1 focuses only on Gmail ingestion, parsing, and in-app workflows with full transparency.

## Features
- Gmail OAuth connection (read-only)
- Incremental email sync (last 30 days on first sync)
- School-related filtering via keywords and sender domains
- LLM extraction with strict JSON schema + deterministic fallback
- Daily view and weekly digest (in-app, optional email)
- Settings for keywords, domains, timezone, digest schedule
- Data deletion flow

## Tech Stack
- Next.js 16 App Router + TypeScript
- Postgres + Prisma
- NextAuth Credentials (email/password)
- Gmail OAuth 2.0 (Google)
- Vercel-compatible cron endpoints

## Getting Started
1. Install dependencies
   ```bash
   npm install
   ```
2. Set environment variables (see below)
3. Run Prisma migrations
   ```bash
   npm run prisma:migrate
   ```
4. Start the dev server
   ```bash
   npm run dev
   ```

## Environment Variables
Create `.env` with:
```
DATABASE_URL=postgresql://USER:PASSWORD@HOST:5432/calmparent
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=your-random-secret
OAUTH_ENCRYPTION_KEY=base64-32-byte-key
CRON_SECRET=your-cron-secret
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
GMAIL_REDIRECT_URI=http://localhost:3000/api/gmail/callback
OPENAI_API_KEY=optional
OPENAI_MODEL=gpt-4o-mini
RESEND_API_KEY=optional
RESEND_FROM=CalmParent <hello@yourdomain.com>
```

Notes:
- `OAUTH_ENCRYPTION_KEY` must be 32 bytes, base64 encoded.
- Email sending only happens when `RESEND_API_KEY` and `RESEND_FROM` are set.

## Cron Endpoints
- `POST /api/cron/sync` (requires `x-cron-secret` header or `?secret=` query param)
- `POST /api/cron/digest` (requires `x-cron-secret` header or `?secret=` query param)

## Fixtures & Tests
Fixtures live in `fixtures/` with corresponding expected JSON. Run tests:
```bash
npm run test
```

## Assumptions
- Phase 1 uses NextAuth Credentials; Google OAuth is only for Gmail access (not sign-in).
- Weekly digest runs once per scheduled window and is stored in-app even if email is not configured.
- LLM output is accepted if it matches schema; if no key or validation fails, fallback parser is used.

## Non-Goals (Phase 1)
- No GPS, meal planning, behavior advice, calendar integration, or multi-caregiver sharing.
- No auto-sending emails or auto actions beyond internal items.

## Deploy to Vercel
1. Create a new Vercel project
   - Import this repo
   - Framework: Next.js

2. Provision Postgres
   - Option A: Vercel Postgres (recommended)
   - Option B: External Postgres (Supabase, RDS, etc.)
   - Copy the connection string into `DATABASE_URL`

3. Set environment variables (Production + Preview)
   Required:
   - `DATABASE_URL`
   - `NEXTAUTH_URL` (e.g. `https://your-app.vercel.app`)
   - `NEXTAUTH_SECRET`
   - `OAUTH_ENCRYPTION_KEY` (32 bytes base64)
   - `CRON_SECRET`
   - `GOOGLE_CLIENT_ID`
   - `GOOGLE_CLIENT_SECRET`
   - `GMAIL_REDIRECT_URI` (must be `https://your-app.vercel.app/api/gmail/callback`)

   Optional:
   - `OPENAI_API_KEY`
   - `OPENAI_MODEL`
   - `RESEND_API_KEY`
   - `RESEND_FROM`

4. Configure Prisma migrations on deploy
   - Recommended Build Command:
     `npm run vercel-build`
   - This runs `prisma migrate deploy` before `next build`.

5. Google Cloud OAuth settings
   - Authorized JavaScript origins:
     - `https://your-app.vercel.app`
     - (Preview) `https://your-app-git-branch.vercel.app`
   - Authorized redirect URIs:
     - `https://your-app.vercel.app/api/gmail/callback`
     - (Preview) `https://your-app-git-branch.vercel.app/api/gmail/callback`

6. Vercel Cron setup
   - `vercel.json` is included with:
     - `/api/cron/sync` every 15 minutes
     - `/api/cron/digest` hourly
   - Security:
     - Each cron call must include either:
       - Header `x-cron-secret: <CRON_SECRET>` OR
       - Query param `?secret=<CRON_SECRET>`
   - If Vercel Cron cannot send headers, use query param.
