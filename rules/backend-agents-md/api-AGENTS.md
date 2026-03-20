# AGENTS.md — API Layer Conventions

Rules for working in `src/soaria/api/`. Distilled from Specs 5–10, Lessons L001–L002
L006–L009 L013 L018, and the cutover readiness session.

Rule IDs: api-R001 through api-R020 | Last reviewed: 2026-03-15

### api-R001: Router Mounting Required
Source: L002 | Added: 2026-03-02 | Status: active

Every new `api/<resource>.py` file MUST be imported and mounted in `router.py`
before the PR is opened. This is the #1 failure mode in this codebase. (L002)

Verify after adding any router:
```bash
python -c "from soaria.main import create_app; app = create_app(); print([r.path for r in app.routes])"
```

Currently mounted routers: health, users, feed, jobs, applications, admin (x2), resumes,
agent, canvas, billing, paste, recruiter, b2b-api, webhooks, catalog, auth_signup.

### api-R002: Auth Dependency Tiers
Source: L018, Spec #5 | Added: 2026-03-02 | Status: active

Three levels of access control in `deps.py`:

| Dependency | Use When |
|-----------|----------|
| `get_current_user` | Any authenticated endpoint. Decodes JWT, stores token on `request.state.access_token`. |
| `require_approved` | User must have `account_status = "approved"`. Allow-list: only "approved" passes. |
| `require_admin` | Admin-only endpoints. Checks `is_admin = true` on profile. |

Never create local auth helpers in API files. Import from `deps.py`. (L018)

### api-R003: User-Scoped DB Access
Source: L006, L018, Spec #10 | Added: 2026-03-02 | Status: active

**For asyncpg-migrated endpoints** (users, resumes, case_studies, embeddings):
Use `get_db_pool(request)` from `deps.py` to get the `DatabasePool`. Inject it
as a FastAPI dependency and pass it to service functions. These services are
`async def` and take `DatabasePool` as their first argument.

**For non-migrated endpoints** (feed, jobs, applications, admin):
Use `get_user_supabase(request)` from `deps.py`. This reads
`request.state.access_token` (set by `get_current_user`) and returns a Supabase
client with the user's JWT for RLS enforcement. (L006, L018)

`use_service_role=True` is ONLY for:
- Auto-profile creation (user has no profile yet)
- Prefect flows (run in their own process)
- Admin operations

If you're reaching for `service_role` in a user endpoint, stop. You're bypassing RLS.

### api-R004: Async Discipline
Source: L005, L016, Spec #10 | Added: 2026-03-02 | Status: active

**For asyncpg-migrated services** (users, resumes, case_studies, embeddings):
No `run_sync()` needed. asyncpg is natively async. Call service functions with
`await` directly from `async def` endpoints.

**For non-migrated supabase-py services** (admin, applications, feed, interactions,
optimization): supabase-py is synchronous. Every `.execute()` call inside an
`async def` MUST be wrapped in `await run_sync()` from `utils/async_compat.py`. (L005)

This applies to both direct `.execute()` calls AND sync helper functions that
internally call `.execute()` — wrap the helper call too. (L016)

Verify supabase-py discipline before committing:
```bash
rg "\.execute\(\)" src/soaria/services/ src/soaria/api/ -n
```

The ONLY exception: code in `flows/` (Prefect flows run in their own process).

### api-R005: Import-Time Safety
Source: L001, L007, L008 | Added: 2026-03-02 | Status: active

Nothing in this directory can call `get_settings()` at module level. (L001, L007, L008)

`create_app()` must succeed with zero environment variables:
```bash
python -c "from soaria.main import create_app"
```

If middleware needs settings (e.g., CORS origins), handle settings-unavailable
gracefully with a try/except fallback. (L008)

### api-R006: Error Response Shape
Source: Spec #8 | Added: 2026-03-02 | Status: active

All API errors use `ErrorResponse` from `api/errors.py`:
```python
{"detail": str, "error_code": str}
```

Catch `LLMBudgetExceededError` in endpoints and return 503 with
`error_code = "LLM_BUDGET_EXCEEDED"`.

### api-R007: Rate Limiting
Source: Spec #14 | Added: 2026-03-02 | Status: active

slowapi is configured in middleware. Disabled when `ENVIRONMENT=test` or
`ENVIRONMENT=development`. No per-endpoint configuration needed unless you want
a different limit than the default.

### api-R008: CORS Configuration
Source: L008 | Added: 2026-03-02 | Status: active

Explicit origin whitelist in middleware. No wildcards in production. Falls back to
`["http://localhost:5173"]` when settings aren't available (import-time safety).

### api-R009: CRUD Completeness
Source: L009 | Added: 2026-03-02 | Status: active

When building CRUD for any resource, implement all 5 operations (Create, Read,
Update, Delete, List) in the same commit. Check contract tests for completeness
assertions. (L009)

### api-R010: JWKS/RS256 JWT Verification
Source: L015 | Added: 2026-03-02 | Status: active

JWT verification uses JWKS endpoint (RS256), not static secrets (HS256). The
`_decode_jwt` in `api/auth.py` reads the algorithm from the token header and
routes to JWKS (RS256/ES256) or legacy HS256 automatically. JWKS client is
cached at module level, lazy-initialized on first use (never at import time). (L015)

### api-R011: Pool via Dependency Injection Only
Source: L047 | Added: 2026-03-06 | Status: active

API endpoints and dependencies must receive the pool via `Depends(get_db_pool)`.
Never import `db_pool` directly. The singleton is only for: `deps.py:get_db_pool()`
(the wrapper), `main.py` (lifespan), and `llm/client.py` (deep infrastructure with
lazy import). (L047)

### api-R012: No Internal Details in Error Responses
Source: L048 | Added: 2026-03-06 | Status: active

Never pass `str(exc)` to HTTPException detail. Use static messages for clients
("Invalid or expired token", "Resource not found", "Invalid request") and log the
real error server-side via structlog. Auth errors get generic messages + `from None`
to suppress exception chains. Health check errors return "unavailable" with no
infrastructure details. (L048)

### api-R013: Anonymous Paths Require user_id NOT NULL Audit
Source: L057 | Added: 2026-03-14 | Status: active

When adding a new auth path (anonymous, API keys, service accounts), run a
systematic audit: `rg "user_id" src/soaria/models/ src/soaria/services/ src/soaria/api/`
and verify every NOT NULL assumption, every required field, and every auth dependency
still holds. The audit itself should be a section in the spec. (L057)

### api-R014: New Tables Need Three RLS Policies
Source: L060 | Added: 2026-03-14 | Status: active

Every new table with RLS enabled needs three policies: (1) owner access (`user_id`
match), (2) `service_role ALL`, (3) admin `SELECT`. When reviewing specs, grep for
`ENABLE ROW LEVEL SECURITY` and verify all three policy types exist for each table.
Missing `service_role` on log tables causes silent write failures. (L060)

### api-R015: Anonymous Endpoint Specs Must Resolve Schema/Auth/Infra
Source: L061 | Added: 2026-03-14 | Status: active

When adding an anonymous/unauthenticated path to an authenticated system, the spec
must explicitly address: (1) which NOT NULL FK constraints become nullable, (2) how
anonymous data links to accounts on signup (claim pattern), (3) rate limiting
mechanism (DB-backed, Redis, or middleware), (4) RLS policy for anonymous rows.
Don't just say "rate limited" — specify the implementation. (L061)

### api-R016: IP Extraction Requires X-Forwarded-For Behind Proxies
Source: L064 | Added: 2026-03-14 | Status: active

Always extract real client IP via X-Forwarded-For:
```python
request.headers.get("x-forwarded-for", "").split(",")[0].strip() or request.client.host
```
`request.client.host` returns the proxy IP on Railway/Cloudflare. Centralize in a
helper function, don't inline. (L064)

### api-R017: SlowAPI Limiter Must Be Module-Level Singleton
Source: L078 | Added: 2026-03-14 | Status: active

When using SlowAPI `@limiter.limit()` decorators in router files, the limiter
instance must be a module-level export from `middleware.py`. Create it at module
scope and reuse the same instance in `add_middleware()`. Decorators need import-time
access — `app.state.limiter` is not available at decoration time. (L078)

### api-R018: Spec-Defined Endpoints Must Ship with Router + Mount + Tests
Source: L079 | Added: 2026-03-14 | Status: active

Every endpoint defined in a spec must have: (1) a router file, (2) a mount in
`router.py`, (3) acceptance tests that hit the endpoint. Verify with:
```bash
python3 -c "from soaria.main import create_app; app = create_app(); print([r.path for r in app.routes])"
```
A migration without a router is not a shipped endpoint. (L079)

### api-R019: Cross-Origin Data Transfer Requires Server Relay
Source: L054 | Added: 2026-03-14 | Status: active

Never rely on sessionStorage or localStorage to transfer data between different
origins (including subdomains). Web storage is same-origin only. Use a server-side
relay (paste token pattern) or URL parameters for cross-domain data transfer. (L054)

### api-R020: Custom Error Shapes Require JSONResponse, Not HTTPException
Source: L089 | Added: 2026-03-15 | Status: active

The global `http_exception_handler` in `errors.py` calls `str(exc.detail)` and maps
status codes to standard error codes. If you pass a dict as `HTTPException.detail`,
it gets stringified into an unparseable repr. When an endpoint needs a custom error
shape (like `SignupErrorResponse` with a specific `error_code` field), return
`JSONResponse` directly instead of raising `HTTPException`. See
`api/auth_signup.py:_signup_error_response()` for the pattern. (L089)
