# Spec 17: Frontend Migration — Lovable to Next.js

**Status:** In Progress
**Date:** 2026-03-04
**Depends on:** Specs 1–16 (all complete)

---

## Goal

Migrate the Lovable-hosted frontend to a single Next.js application deployed on
Vercel. Both `soaria.xyz` (marketing) and `app.soaria.xyz` (web app) point to the
same Vercel project. Consolidate to a single Supabase project (the backend's).
One repo, one deploy pipeline, shared design system.

## Background

### Why migrate

Lovable provides rapid prototyping but creates architectural problems at scale:

1. **Supabase lock-in.** Lovable provisions its own Supabase project that we cannot
   access or configure. Auth settings, RLS policies, and database schema are
   controlled by Lovable's deployment pipeline, not ours.

2. **Two databases.** The Lovable frontend queries its own Supabase directly for 14
   tables. The backend has the canonical data in a separate Supabase project. Data
   must be synchronized manually, and there is no single source of truth.

3. **Edge Functions duplicate backend logic.** Lovable auto-generates 4 Supabase
   Edge Functions that replicate business logic that already exists in our FastAPI
   backend (optimization, profile management). Changes must be made in two places.

4. **Direct DB queries bypass the API.** The Lovable frontend uses `supabase.from()`
   to query tables directly, bypassing the FastAPI API layer entirely. This means
   rate limiting, budget tracking, audit logging, and business rules in the API are
   not enforced.

5. **No build pipeline control.** We cannot configure Next.js settings, add custom
   middleware, or control the build/deploy pipeline. Performance optimization
   (code splitting, ISR, edge caching) is not possible.

### What migration achieves

- Single Supabase project for auth + data (the backend's)
- All data access through the FastAPI API (RLS, rate limits, budget tracking enforced)
- Full control over build pipeline, routing, SSR/SSG, middleware
- Vercel deployment with preview environments and custom domains
- Design system ownership (Tailwind + shadcn/ui, dark theme)

---

## Lovable Audit Summary

Audit of the current Lovable frontend (2026-03-04):

- **30 pages** across auth, feed, optimization, admin, and marketing sections
- **78 React components** (mix of page components and UI primitives)
- **14 Supabase tables** queried directly via `supabase.from()`
- **4 Edge Functions** (optimization, profile, sync, notifications)
- **Design system:** Tailwind CSS + shadcn/ui, dark theme, custom color tokens
- **Auth:** Supabase Auth with Google OAuth, email/password

### Pages by section

| Section | Pages | Key functionality |
|---------|-------|-------------------|
| Auth | 3 | Login, signup, callback |
| Feed | 4 | Job feed, job detail, saved jobs, feed stats |
| Optimization | 5 | Resume upload, optimization view, deltas, chat, download |
| Applications | 4 | Application list, detail, create, stats |
| Profile | 4 | Profile view/edit, onboarding, focus areas |
| Admin | 6 | User management, greenhouse, dashboard, LLM budget |
| Marketing | 4 | Landing, about, pricing, contact |

---

## Architecture

```
                    ┌──────────────┐
                    │   Vercel     │
                    │  Next.js 15  │
                    │  App Router  │
                    └──────┬───────┘
                           │ HTTPS
                           ▼
                    ┌──────────────┐
                    │   Railway    │
                    │   FastAPI    │
                    │   Backend    │
                    └──────┬───────┘
                           │ asyncpg
                           ▼
                    ┌──────────────┐
                    │   Supabase   │
                    │  PostgreSQL  │
                    │  (one project)│
                    └──────────────┘
```

**Auth flow:**
1. Next.js uses `@supabase/ssr` for cookie-based session management
2. Google OAuth configured on the backend Supabase project
3. On login, Supabase issues a JWT
4. Next.js middleware reads the session cookie and injects `Authorization: Bearer <jwt>`
   into all API requests to the FastAPI backend
5. FastAPI validates the JWT via JWKS (existing `api/auth.py`)

**Data flow:**
- All data access goes through the FastAPI API (`/v1/*` endpoints)
- No direct `supabase.from()` calls in the frontend
- Supabase client in the frontend is used ONLY for auth (login, logout, session refresh)

---

## Backend Preparation (this repo)

### Artifacts generated (Spec 17 deliverables)

| Artifact | Location | Purpose |
|----------|----------|---------|
| OpenAPI spec | `docs/openapi.json` | Machine-readable API contract for client codegen |
| API reference | `docs/API_REFERENCE.md` | Human-readable endpoint documentation for frontend devs |
| TypeScript types | `docs/frontend-types.ts` | Pydantic model → TS interface translation |
| Frontend CLAUDE.md | `docs/FRONTEND_CLAUDE_DRAFT.md` | Starter operating instructions for the frontend repo |

### CORS update

Added `http://localhost:3000` to `.env.example` CORS_ORIGINS for Next.js dev server.

### Endpoint gap analysis

| Capability | Status | Route | Notes |
|------------|--------|-------|-------|
| User feed state | Partial | `GET /v1/feed/stats` | Aggregate state only; per-job state included in feed cards |
| Optimization deltas | Complete | `GET /v1/resumes/{tid}/versions/{vid}/deltas` | Returns delta list + quality score |
| Resume DOCX upload | Complete | `POST /v1/resumes` | Multipart form: file + template_name + job_focus |
| Greenhouse sync logs | Complete | `GET /v1/admin/greenhouse/sync-logs` | Paginated, date-range filtered |
| Resume DOCX download | Complete | `GET /v1/resumes/{tid}/versions/{vid}/download` | Streams .docx file |

All five investigated capabilities exist and are functional. The feed state endpoint
returns aggregate data only (not a dedicated per-item state endpoint), but per-job
interaction state (is_saved, is_passed, is_viewed) is included in each `FeedJobCard`
in the feed response.

---

## Frontend Build (single repo: `soaria`)

Single Next.js repo, single Vercel project. Both `soaria.xyz` and `app.soaria.xyz`
point to the same deployment. No separate frontend repo.

### Tech stack

- **Framework:** Next.js 15 (App Router, Server Components)
- **Language:** TypeScript (strict mode)
- **Styling:** Tailwind CSS 4 + shadcn/ui
- **Auth:** @supabase/ssr (cookie-based sessions)
- **State:** React Query (TanStack Query) for server state
- **API client:** Single module with Bearer token injection

### Auth implementation

```
@supabase/ssr → createServerClient (middleware) → read session cookie
                                                → inject Authorization header
                                                → forward to FastAPI
```

- `middleware.ts` refreshes the session on every request
- Auth pages: `/login`, `/signup`, `/auth/callback`
- Protected routes: everything except auth pages and marketing pages
- Google OAuth: configured on the backend Supabase project

### API client pattern

Single `lib/api.ts` module:

```typescript
// All API calls go through this client
const api = {
  get: (path, options?) => fetch(`${API_URL}${path}`, { headers: authHeaders(), ...options }),
  post: (path, body, options?) => fetch(`${API_URL}${path}`, { method: 'POST', body: JSON.stringify(body), headers: authHeaders(), ...options }),
  // ... put, delete, patch
};
```

### Marketing pages — temporary placeholder

Marketing routes (`/`, `/how-it-works`, `/partnerships`, `/privacy-policy`, `/terms`)
ship with a temporary coming-soon page: a fun illustration and a "we're building
something" message. Zero traffic, zero users — no one cares. Real marketing pages
get ported later, after the app is live.

### Page port order

1. **Auth** — login, signup, callback (foundation)
2. **Marketing placeholder** — coming-soon page for `/`, `/how-it-works`, etc.
3. **Feed** — job feed, job detail, saved jobs (core experience)
4. **Optimization** — resume upload, optimization, chat, download (core value prop)
5. **Profile** — onboarding, profile, focus areas, preferences
6. **Applications** — list, detail, stats, interview tracking
7. **Admin** — user management, greenhouse, dashboard, LLM budget
8. **Marketing (real)** — port actual pages last, lowest priority

### Directory structure

```
src/
  app/
    (marketing)/            # Public marketing routes
      page.tsx              # / — coming-soon placeholder
      how-it-works/
      partnerships/
      privacy-policy/
      terms/
      layout.tsx            # Marketing layout (no sidebar)
    (auth)/                 # Auth pages (no sidebar)
      login/
      signup/
      auth/callback/
    (app)/                  # Authenticated app pages (sidebar layout)
      feed/
      jobs/[id]/
      resumes/
      applications/
      profile/
      admin/
      layout.tsx            # App layout (sidebar + nav)
  components/
    ui/                     # shadcn/ui components
    feed/                   # Feed-specific components
    resumes/                # Resume-specific components
    applications/           # Application-specific components
    admin/                  # Admin-specific components
    layout/                 # Sidebar, nav, header, footer
    marketing/              # Coming-soon page, illustrations
  lib/
    api.ts                  # API client (single module)
    supabase/
      client.ts             # Browser Supabase client
      server.ts             # Server Supabase client
      middleware.ts          # Middleware Supabase client
  types/                    # TypeScript interfaces (from backend)
  hooks/                    # Custom React hooks
  styles/                   # Global styles, Tailwind config
```

### Design system

From the Lovable audit:

- **Theme:** Dark mode primary, light mode secondary
- **Colors:** Dark backgrounds (#0a0a0a, #1a1a2e), blue accents (#3b82f6, #60a5fa),
  green for success, amber for warnings
- **Typography:** Inter (body), JetBrains Mono (code/data)
- **Components:** shadcn/ui as base, customized with Soaria tokens
- **Layout:** Sidebar navigation (desktop), bottom tabs (mobile)

---

## User Migration

### Steps

1. Export user data from Lovable's Supabase project
2. Map users to the backend Supabase project's `auth.users` table
3. Verify all user profiles exist in the backend's `user_profiles` table
4. Send password reset emails (or rely on Google OAuth for re-auth)

### Considerations

- Users who signed up via Google OAuth just need to re-authenticate
- Users with email/password need a password reset flow
- Activity history and application data already live in the backend DB
- No data loss expected — the backend is the source of truth for business data

---

## OAuth Setup

Configure Google OAuth on the backend Supabase project:

1. Google Cloud Console → Create OAuth 2.0 credentials
2. Authorized redirect URI: `https://<supabase-project>.supabase.co/auth/v1/callback`
3. Supabase Dashboard → Auth → Providers → Google → Enable, paste client ID + secret
4. Frontend: `supabase.auth.signInWithOAuth({ provider: 'google' })`

---

## Deployment

Single Vercel project, two domains pointing to the same deployment.

- **Platform:** Vercel
- **Domains:**
  - `soaria.xyz` — marketing pages (coming-soon initially, real pages later)
  - `app.soaria.xyz` — web application
  - Both are CNAMEs to the same Vercel project
- **Routing:** Next.js middleware distinguishes between domains if needed (e.g.,
  redirect `soaria.xyz/feed` → `app.soaria.xyz/feed`). Or use route groups to
  serve different layouts per domain.
- **Preview:** Automatic preview deployments on PRs
- **Environment variables:**
  - `NEXT_PUBLIC_SUPABASE_URL` — backend Supabase project URL
  - `NEXT_PUBLIC_SUPABASE_ANON_KEY` — backend Supabase anon key
  - `NEXT_PUBLIC_API_URL` — FastAPI backend URL (Railway)

### DNS cutover plan

1. Point `app.soaria.xyz` CNAME to Vercel (new — currently pointed at Lovable)
2. Update `soaria.xyz` CNAME to the same Vercel project (currently already on Vercel
   but serving the Lovable-synced marketing site)
3. Remove the old Lovable → GitHub → Vercel sync for the marketing site
4. Both domains now serve from the single Next.js project

---

## Verification Checklist

- [ ] Google OAuth works end-to-end (login → callback → session → API call)
- [ ] All 52 API endpoints reachable from the frontend
- [ ] Feed loads with ranked job cards
- [ ] Resume DOCX upload + optimization + download works
- [ ] Optimization chat (multi-turn) works
- [ ] Application creation and tracking works
- [ ] Admin dashboard loads with stats
- [ ] CORS allows requests from both Vercel domains
- [ ] Rate limiting works (100 req/min per IP)
- [ ] Error responses display correctly in the UI
- [ ] Mobile responsive layout works
- [ ] Lighthouse score > 90 (performance)
- [ ] Marketing coming-soon page renders on soaria.xyz
- [ ] App pages render on app.soaria.xyz
- [ ] soaria.xyz/feed redirects to app.soaria.xyz/feed (no app routes on marketing domain)
