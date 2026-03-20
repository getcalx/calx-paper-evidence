# Spec Summary — Soaria Backend

Canonical index of all specs, fixes, and major work sessions. Each entry links to
its spec file (historical artifact) and lists key outputs.

Use this file as the current source of truth for what's been built.

---

## Archive

Completed specs (1–31), fixes (FIX_00–FIX_07), the security audit, and
superseded docs live in `specs/archive/`. They are historical artifacts —
design context, rejected alternatives, and implementation notes from the
build phase. For current conventions, see `AGENTS.md` files in each source
directory.

---

## System Overview

Soaria is an AI-powered job platform that translates between how candidates describe
their experience and the outcome-driven language recruiters use. The Python backend
(FastAPI on Railway) serves a Lovable (React/TypeScript) frontend. All data lives in
Supabase PostgreSQL with Row-Level Security enforced via JWT passthrough.

**Data pipeline.** Jobs enter the system through Greenhouse ATS board sync
(`flows/greenhouse_sync.py`), which fetches listings from the Greenhouse Boards API
in stalest-first order, upserts them into the `jobs` table, and deactivates missing
listings. Once ingested, the embedding flow (`flows/embed_jobs.py`) generates 1024-
dimension vectors via Voyage 3 Large and stores them in pgvector. The ranking flow
(`flows/rank_jobs.py`) then scores each candidate-job pair using vector similarity
to find candidates, followed by Claude Sonnet evaluation against a 100-point rubric.
Ranked matches populate the `job_matches` table, which the feed API serves as a
personalized job feed. Candidates can save, pass, or apply to jobs, and optionally
run Gemini-powered resume optimization before submitting.

**Auth model.** JWT verification uses the JWKS endpoint (`api/auth.py`) with RS256
as the primary algorithm and HS256 as a legacy fallback. The `get_current_user`
dependency decodes the token, auto-creates a profile on first login (ON CONFLICT DO
NOTHING for race safety), and returns an `AuthenticatedUser` with `account_status`.
Downstream dependencies (`require_approved`, `require_admin`) gate access based on
status and role. The raw JWT is stored on `request.state.access_token` for RLS
passthrough to Supabase when using the transitional supabase-py client.

**LLM integration.** Three models serve different purposes: `anthropic/claude-sonnet-4-5-20250514`
(ranking, temp 0.2, JSON output), `gemini/gemini-2.0-flash` (resume optimization,
temp 0.3), and `voyage/voyage-3-large` (embeddings, 1024 dims). All calls route
through `llm/client.py`, which enforces a configurable daily budget (default $25/day)
via in-memory tracking and persistent logging to `llm_call_log`. Budget exceeded
triggers an `LLMBudgetExceededError` that surfaces as HTTP 503.

**Database access.** The primary path is an asyncpg connection pool (`db/pool.py`)
with a repository layer in `db/` (domain modules: users, resumes, embeddings,
ranking, applications, case_studies, admin, optimization). The app pool is a singleton
initialized during FastAPI lifespan and injected via `get_db_pool()`. Prefect flows
use independent short-lived pools via `create_flow_pool()` to avoid sharing state
with the app process. A transitional supabase-py path (`get_user_supabase`) remains
but is being phased out.

**Orchestration.** Prefect Cloud runs three background flows: Greenhouse sync
(stalest-first board ingestion), embedding generation (batch Voyage calls for
unembedded jobs and resumes), and ranking (vector search + LLM scoring per user
focus area). Flows use tasks with retry policies for transient failures and
independent database pools for lifecycle isolation.

---

## Feature Guide

### 1. Greenhouse Sync

Prefect flow (`flows/greenhouse_sync.py`) that syncs job listings from Greenhouse
ATS boards. Fetches active boards ordered by staleness (`last_synced_at ASC NULLS
FIRST`), pulls jobs via the Boards API with concurrency-limited semaphores, upserts
into the `jobs` table, and deactivates listings that have disappeared. Location
parsing handles international formats with a country code lookup map. Dead boards
are marked inactive after repeated failures.

### 2. Embeddings

Generates 1024-dimension vectors using Voyage 3 Large (`voyage/voyage-3-large`) for
both job descriptions and resume templates. Job text is prepared by stripping HTML
and concatenating title with description. Resume text combines summary, skills, and
experience sections. Vectors are stored in pgvector and used by the ranking flow for
semantic similarity search. Embedding is batched (50 items per call) and runs as a
Prefect flow (`flows/embed_jobs.py`) with budget-aware early termination.

### 3. Ranking

Scores candidate-job fit using a 100-point rubric across five dimensions: concept/skills
(40 points), role fit (20), experience (15), context (15), and source quality (10).
Source quality is deterministic (Greenhouse = 10, LinkedIn = 6, aggregator = 4).
The remaining 90 points come from Claude Sonnet LLM evaluation (`anthropic/claude-sonnet-4-5-20250514`,
temp 0.2, JSON response format). Vector search narrows candidates to semantically
relevant jobs before LLM scoring. Results upsert into `job_matches` with algorithm
versioning (`v4.0-voyage35-sonnet45`) and application priority classification
(high: 85-100, medium: 60-84, low: 0-59).

### 4. Feed

Serves ranked job matches as a personalized feed via `GET /v1/feed`. Uses offset-based
pagination (switched from cursor-based due to UUID ordering issues). Sort order:
unviewed jobs first, then by `overall_fit_score` DESC, then `date_posted` DESC. Feed
supports `focus_area` filtering, configurable score thresholds with sparse-feed
fallback (lowers threshold when too few results), and interaction tracking (save,
pass, view with dwell time and session context). An intervention threshold triggers
alerts after consecutive pass actions.

### 5. Applications

Tracks job applications through a status state machine: applied -> screening ->
interviewing -> offer, with rejection/ghosted/withdrawal exits at each stage.
Supports interview round tracking (create, update rounds per application) and
outcome recording. Provides conversion rate statistics and per-user application
stats. Admin endpoints expose cohort-level analytics and resume source tracking
(soaria_optimized vs user_original). The state machine enforces valid transitions
and prevents invalid status changes.

### 6. Resumes and Optimization

Resume management combines DOCX parsing (`python-docx`), structured storage, and
LLM-powered optimization. Users upload a resume template (fixed Soaria DOCX format),
which is parsed into structured sections (contact, summary, skills, experience,
education). Optimization uses Gemini Flash (`gemini/gemini-2.0-flash`, temp 0.3) to
tailor the resume for a specific job, producing a `system_optimized` version with
tracked deltas showing exactly what changed. Idempotent -- re-optimizing for the
same job returns the existing version. Budget-aware with daily spend limits.

### 7. Case Studies

STAR-format (Situation, Task, Action, Result) portfolio entries that candidates can
attach to their profile. Stored with metadata including quantified impact, skills
demonstrated, tags, time period, company context, and functional area. Tracks usage
count and completeness flags (`is_complete`, `needs_more_detail`). Case studies feed
into the resume optimization prompt as contextual evidence of the candidate's work.
CRUD operations via `GET/POST/PUT/DELETE /v1/case-studies`.

### 8. Admin

Admin-only endpoints (`/v1/admin/*`) gated by `require_admin` dependency. Covers
four domains: user management (list, detail, status/role updates with search and
filtering), Greenhouse board management (CRUD for board configs, manual sync trigger),
system dashboard (aggregate stats on users, jobs, matches, applications), and LLM
budget monitoring (usage breakdown by model, daily spend tracking, budget status).
Two-query pagination approach for user listing: fetch profiles, then batch-count
applications and merge in Python.

### 9. Auth and Users

JWT authentication via JWKS endpoint with RS256 (asymmetric) as the primary algorithm
and HS256 (symmetric) as a legacy fallback. `get_current_user` extracts the bearer
token, decodes via `_decode_jwt`, and auto-creates a profile on first login using
ON CONFLICT DO NOTHING for race safety. Account statuses default to `approved` on signup (waitlist retired per PRD v4.1).
Founding member codes (`POST /v1/me/redeem-code`) set pricing benefits ($19/mo locked
rate + 90 days free) — no longer an access gate. `require_approved` uses an allow-list pattern (only `approved`
passes). User endpoints at `/v1/me` support profile CRUD, code redemption, and focus
area management (the role types a candidate is interested in, which drive ranking and
feed content). Resume upload sets `onboarding_completed_at`.

---

## Active Specs

| Spec | Name | Status | Key Outputs |
|------|------|--------|-------------|
| 39 | Production Hardening | Partial (39A merged) | 39A: DB indexes (merged PR #116). 39B-E blocked on Redis provisioning. |

## Recently Completed (archived → `specs/archive/`)

| Spec | Name | Key Outputs |
|------|------|-------------|
| 32 | Tier 0 Competency Catalog | 11 roles, two-tier Obsidian+Postgres, Bayesian confidence, HDBSCAN clustering, SSE contract. PR #107. |
| 33 | OAuth + Signup Flow | `POST /v1/auth/complete-signup`, profile enrichment, reverse trial, session claiming. Migration 024. PR #109. |
| 34 | Tier 2 Experience Path | `classify_tier_with_context`, seniority detection, better-fit role matching. PR #110. |
| 35 | Contract Sync + Rate Limits | types.ts alignment, slowapi rate limiting. PR #110. |
| 36 | Prefect Deployment | All 11 flows deployed, prefect.yaml, verify_deployments.py. PR #111. |
| 37 | Webhook Resilience | DLQ, idempotency, handler registry, admin endpoints. Migrations 026-027. PR #116. |
| 38 | Funnel Analytics | 3 admin analytics endpoints (funnel, engagement, pipeline). No migrations. PR #116. |
| 40 | Commentary Suppression | `suppress_commentary` flag, prompt modification, summary enforcement. |
| 41 | Stress Test Harness | 50-pair JD/resume eval, LLM judge, 65% win rate threshold. |
| 42 | Bootstrap Vocab Sides | Fix `side="candidate"` → `side="recruiter"` in bootstrap pipeline. |
| Context Engineering | ACE Hardening | Structured rule IDs, bidirectional traceability (95 lessons ↔ 106 rules), context health scripts, MCP coordination. |

## Completed Specs (archived → `specs/archive/`)

| Spec | Name | Key Outputs |
|------|------|-------------|
| 1 | Repo Scaffold | `main.py`, `config.py`, `database.py`, `api/router.py`, `api/health.py`, Dockerfile, railway.toml, Makefile |
| 1.5 | Schema Setup | 27 Supabase tables, RLS policies, pgvector, seeded industries + Greenhouse companies (SQL only, no Python) |
| 1.7 | Integration Tests | `supabase/config.toml`, integration test fixtures, `make test-integration` target |
| 2 | Greenhouse Sync | `services/greenhouse.py`, `flows/greenhouse_sync.py`, `models/jobs.py` — stalest-first board sync, upsert, dead-board marking |
| 3 | Embeddings Pipeline | `services/embeddings.py`, `flows/embed_jobs.py`, `models/embeddings.py`, `utils/async_compat.py` — Voyage 3 Large 1024d vectors via asyncpg |
| 4 | Ranking Service | `services/ranking.py`, `flows/rank_jobs.py`, `models/ranking.py`, `llm/prompts/ranking.py` — 100-point scoring (40 concept + 20 role + 15 exp + 15 context + 10 source) |
| 5 | Auth + Users + Onboarding | `api/auth.py`, `api/deps.py`, `api/users.py`, `services/users.py`, `models/users.py` — JWKS/RS256, auto-profile, onboarding, focus areas |
| 6 | Job Feed API | `api/feed.py`, `api/jobs.py`, `services/feed.py`, `services/interactions.py`, `models/feed.py` — offset pagination, visibility tiers, save/pass/view interactions |
| 7 | Application Tracking | `api/applications.py`, `services/applications.py`, `models/applications.py` — status state machine, interview rounds, conversion stats |
| 8 | Resume Optimization | `api/resumes.py`, `services/optimization.py`, `services/case_studies.py`, `services/docx_parser.py`, `models/resumes.py`, `llm/prompts/optimization.py`, `llm/budget.py` — DOCX upload, Gemini optimization, delta tracking, $25/day budget |
| 9 | Admin API | `api/admin.py`, `services/admin.py`, `models/admin.py` — user management, board management, system dashboard, LLM budget endpoints |
| 10 | asyncpg Migration | `db/` package (`pool.py`, `_helpers.py`, `users.py`, `resumes.py`, `case_studies.py`, `embeddings.py`), migrated services + API endpoints to asyncpg |
| 11 | Integration Tests | `tests/integration/` — 6 test files (47 tests total), factory fixtures, CI workflow, coverage config |
| 12 | SQL Migrations | `db/migrate.py`, `db/__main__.py`, `migrations/001_initial_schema.sql` — forward-only migration runner, CLI, consolidated baseline schema, `make migrate`/`make migrate-status` |
| 13 | Service Split | Multi-stage Dockerfile (uv + hatchling), PEP 621 pyproject.toml, Railway buildTarget, `scripts/check_imports.py` import audit |
| 14 | Email + Circuit Breaker | `services/email.py` (Resend API), per-user LLM cap (10/day default), global kill switch with auto-trigger, `flows/send_reminders.py` (Prefect), `migrations/002_system_settings.sql` |
| 15 | Admin On-Behalf | Admin resume/case-study/focus-area upload on behalf, soft-delete user with PII anonymization, relaxed activity event validator, onboarding re-submit fix |
| 16 | Optimization Chat + DOCX | `GET /download` renders structured_data to .docx, `POST/GET/DELETE /chat` stateful multi-turn optimization refinement, `services/optimization_chat.py`, `db/optimization_chat.py`, `llm/prompts/optimization_chat.py`, `migrations/003_optimization_sessions.sql` |
| 17 | Frontend Migration | Next.js migration plan, OpenAPI export, API reference, TypeScript types, frontend CLAUDE.md draft. Frontend repo (Frannie) built 7 specs from this plan. |
| 18 | Agent Core | Agent planner, dispatcher, assembler, voice streaming, SSE endpoint, session management, tool registry (13 tools), memory handoff, reframing patterns, resume template adaptation, observability traces, agent state machine. Migration 009 (6 tables). |
| 19 | Self-Serve Onboarding | Waitlist default, founding member code redemption, embed-on-upload, analysis prompt fix, proactive analysis trigger, career memory seeding, case study guidance tool, onboarding form removal, email rebrand (Soaria→Versed). |
| 20 | Conversational Job Search | Vocab mappings in ranking prompt, search tool passthrough, search refinement via session state, save/pass/unsave tools, REST feed deprecation, configurable batch size. |
| 21 | Agent Optimization Flow | translation_applied in assembler, iterate_optimization tool, planner approval/iteration routing, optimization state tracking, preference carryover, STAR story integration. |
| 22 | Session Memory | SessionState model + JSONB column, state read/write service, planner prompt injection, state update after dispatch, context summarization, session recovery + first-visit greeting. Migration 011. |
| 23 | Anonymous Sessions & Identity Node | Nullable user_id on agent_sessions, anonymous session creation, claim_token binding, session claiming endpoint, data migration on claim, Identity Node JSONB, anonymous rate limiting, paste relay. Migration 012. |
| 24 | Agent Alignment | TranslationCard/GapCard models, CardState enum, analyze_vocabulary tool, progressive disclosure, opinionated advisor prompt, card interactions, gap→translation conversion, STAR linking. Migration 013. |
| 25 | Reverse Trial & Subscription | Subscriptions + usage_tracking tables, Stripe Checkout, trial lifecycle, feature gating, founding member pricing ($19/mo locked, 90 days free), billing router + webhook. Migration 014. |
| 26 | Canvas State Persistence | canvas_state table, PUT/GET /v1/canvas endpoints, JSONB positions + viewport, upsert semantics. Migration 015. |
| 27 | Application Tracking v2 | Merge.dev integration (webhook + polling), outcome-to-translation linking, follow-up reminders via Prefect, merge_sync_log audit trail. Migration 016. |
| 28 | Recruiter Backend | recruiter_profiles + jd_analyses tables, recruiter auth role, JD vocabulary analysis, candidate-repellent scoring, JD optimization flow, multi-JD dashboard. Migration 017. |
| 29 | B2B API Products | api_subscriptions table, JD Optimization API, Outbound Optimization API, API key auth, rate limiting per tier, persona library, usage tracking. Migration 018. |
| 30 | Cohort Delivery Pipeline | cohort_requests + cohort_candidates tables, candidate consent, Prefect matching flow, recruiter feedback → confidence loop, ops dashboard, anonymized summaries. Migration 019. |
| 31 | Security Hardening | 5 B2B API boundary fixes: API key hashing, request signing, audit logging, input sanitization, rate limit tightening. |
| Security Audit | Security & Quality Audit | 8 commits: input validation, error sanitization, LLM output safety, timeout discipline, concurrency safety, budget wiring. Lessons L048-L052. |

## Fixes (archived → `specs/archive/`)

| Fix | Name | What It Fixed |
|-----|------|---------------|
| FIX_00 | Master Runbook | Index and execution order for FIX_01–FIX_06 |
| FIX_01 | Boot + Router + Port | Import-time settings crash, unmounted routers, hardcoded port |
| FIX_02 | Auth Token Passthrough | JWT not passed to Supabase for RLS enforcement |
| FIX_03 | Resume Model Cleanup | Duplicate Pydantic class definitions silently overwriting |
| FIX_04 | Async Event Loop | Sync .execute() blocking FastAPI event loop |
| FIX_05 | Schema Verification | Missing columns and wrong column names |
| FIX_06 | Prefect Deployment | Missing deployment config, missing DELETE endpoint |
| FIX_07 | Flow Pool Isolation | Flows sharing app singleton pool → `DatabasePool.from_dsn()` for independent lifecycle. flow-R008. |

## Cutover Readiness Session — Done

Landed as 11 commits of post-spec hardening. Key changes:

- Feed pagination: cursor-based (broken with UUID) → offset-based
- Activity batch insert: loop → single multi-row INSERT
- Upsert semantics: ON CONFLICT DO NOTHING for auto-profile-create
- Rate limiting: slowapi on all user-facing endpoints
- CORS lockdown: explicit origin whitelist, no wildcards
- .dockerignore: excludes tests, specs, docs from container
- Request size limits: 10MB body size middleware
- Prompts package split: `llm/prompts.py` → `llm/prompts/` package
- `get_user_supabase` centralized in `api/deps.py` (no more local helpers)
- Lessons L019–L024 captured

## M2 Alignment Session — Done (2026-03-05)

Product alignment with Spencer: documented core product loop, agentic workflow
expectations, frontend strategy decision (keep Next.js port, defer deployment).
Outputs: `product-context.md` in memory, SPENCER.md updates.

## ACE Hardening — Done (2026-03-14)

Context engineering formalization: structured rule IDs in all 8 AGENTS.md files,
bidirectional traceability (80 lessons ↔ 92 rules), `scripts/context_health.py`
(coverage/staleness/deduplication), context collapse prevention hooks, MCP
coordination server (8 tools).

## Content Graduation — Done (2026-03-02, updated 2026-03-14)

All actionable content from specs, fixes, and lessons graduated into:

- **AGENTS.md files** — `api/`, `services/`, `flows/`, `llm/`, `models/`, `db/`, `tests/integration/`
- **CLAUDE.md** — Updated with current state, structured session protocol
- **how-we-document.md** — Documentation framework defining what goes where
- **This file** — Canonical spec index

Graduation passes: Specs 1–9 and FIX_01–06 (2026-03-02), Specs 10–13 (2026-03-03),
Specs 14–16 (2026-03-03), Security Audit and Spec 17 (2026-03-09), Specs 19–31
and L053–L080 (2026-03-14).
