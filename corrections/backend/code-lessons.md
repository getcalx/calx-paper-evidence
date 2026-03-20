# Lessons Learned — Soaria Backend

Update this file after every correction. Each entry is a mistake pattern
and the rule that prevents it. Review at session start.

---

## L001: Import-time settings loading crashes app
**Date:** 2026-02-28
**Spec:** #1
**Distilled to:** api-R005
**Reviewed:** 2026-03-09
**What happened:** `create_app()` called `get_settings()` in its body. App crashed
on Railway because env vars aren't available at import time in Docker containers.
Deploy showed zero output after "Starting Container" — no traceback, no uvicorn
startup, nothing. Silent death.
**Root cause:** Settings instantiation triggers Pydantic validation which throws
if required env vars are missing. This happens before logging is configured,
so the error is swallowed.
**Rule:** `create_app()` must NEVER call `get_settings()`. Settings load in lifespan
only. Test: `python -c "from soaria.main import app"` must succeed with zero env vars.
**Also check:** Any module imported by `create_app()` (router.py, middleware.py,
errors.py) and their transitive imports. If ANY file in the import chain has
`settings = get_settings()` at module level, the same crash happens.

## L002: Router files created but never mounted
**Date:** 2026-02-28
**Spec:** #5
**Distilled to:** api-R001
**Reviewed:** 2026-03-09
**What happened:** `api/users.py` existed with all endpoints defined, but was never
imported or mounted in `api/router.py`. All user endpoints returned 404.
**Root cause:** Specs are built in separate context windows. The spec that creates
users.py doesn't have router.py context. The spec that created router.py only
mounted health.
**Rule:** Every new `api/<resource>.py` MUST be imported and mounted in `api/router.py`
in the same commit. Verify with route listing.

## L003: Railway doesn't expand $PORT in startCommand
**Date:** 2026-02-28
**Spec:** #1
**Distilled to:** N/A (deployment)
**Reviewed:** 2026-03-09
**What happened:** `railway.toml` had `startCommand = "... --port $PORT"`. Railway
passes startCommand in exec form, not through a shell. `$PORT` was passed literally
to uvicorn, which threw "is not a valid integer."
**Root cause:** Railway's startCommand overrides Dockerfile CMD and runs in exec form
(no shell expansion).
**Rule:** Either (a) remove startCommand and let Dockerfile CMD handle it (shell form
expands vars), or (b) wrap in `sh -c`. Never put bare `$PORT` in startCommand.

## L004: Duplicate Pydantic class definitions silently overwrite
**Date:** 2026-02-28
**Spec:** #8
**Distilled to:** mdl-R001, mdl-R002
**Reviewed:** 2026-03-09
**What happened:** `models/resumes.py` had `class OptimizationDeltaResponse` defined
twice. The second definition (missing fields) silently replaced the first. Also
`BudgetStatus` had copy-pasted fields from a different model.
**Root cause:** Building models across multiple prompts/sessions. Later session
re-defined classes that already existed.
**Rule:** Before writing any model class, grep the file for existing definitions.
Never copy-paste model classes. Check: `rg "^class " src/soaria/models/<file>.py | sort | uniq -d`

## L005: supabase-py .execute() blocks async event loop
**Date:** 2026-02-28
**Spec:** #4, #5, #6, #7, #8
**Distilled to:** api-R004, svc-R001, svc-R002
**Reviewed:** 2026-03-09
**What happened:** Synchronous `.execute()` calls inside `async def` functions block
the entire FastAPI event loop. Under load, all concurrent requests stall.
**Root cause:** supabase-py is a sync library. No async equivalent available.
**Rule:** Every `.execute()` in services/ and api/ must be wrapped in
`await run_sync()`. Check: `rg "\.execute\(\)" src/soaria/services/ src/soaria/api/`

## L006: access_token not passed to Supabase client for RLS
**Date:** 2026-02-28
**Spec:** #5
**Distilled to:** api-R003, svc-R003
**Reviewed:** 2026-03-09
**What happened:** `get_supabase_client()` didn't accept an `access_token` param.
User-facing endpoints either used service_role (bypassing RLS = security hole) or
anon key without JWT (RLS blocks everything = silent failures).
**Root cause:** database.py was written before auth spec. Auth spec references JWT
passthrough but database.py was never updated.
**Rule:** User endpoints use `Depends(get_user_supabase)` which passes the JWT.
`use_service_role=True` is ONLY for Prefect flows and auto-profile creation.

## L007: Transitive imports can trigger settings loading
**Date:** 2026-03-02
**Distilled to:** api-R005
**Reviewed:** 2026-03-09
**What happened:** `create_app()` was fixed (no direct `get_settings()` call), but
the app still crashed on Railway with zero output. A module somewhere in the import
chain (router.py → some_api.py → some_service.py) had `get_settings()` at module level.
**Root cause:** `app = create_app()` runs at import time, triggering all lazy imports
in create_app's body. If any transitively imported module calls `get_settings()` at
module scope, the crash propagates silently.
**Rule:** Search entire import chain: `rg "get_settings\(\)" src/soaria/ -n` and
`rg "get_supabase_client\(" src/soaria/ -n`. Every hit must be inside a function body,
never at module level.

## L008: add_middleware() called get_settings() at app construction time
**Date:** 2026-03-02
**Spec:** #1
**Distilled to:** api-R005, api-R008
**Reviewed:** 2026-03-09
**What happened:** `middleware.py:add_middleware()` called `get_settings()` to read
CORS origins. `create_app()` calls `add_middleware(app)`, and `app = create_app()`
runs at module level. So importing `soaria.main` triggered settings validation,
failing with `ValidationError: supabase_jwt_secret Field required`.
**Root cause:** The docstring in `add_middleware` incorrectly claimed "settings are
loaded lazily here (middleware runs at request time)". In reality, `add_middleware`
runs at construction time, not request time. CORSMiddleware needs `allow_origins`
at construction.
**Fix:** Wrapped `get_settings()` in try/except inside `add_middleware()`. Falls back
to `["http://localhost:5173"]` for CORS origins when settings aren't available.
Production (env vars set) gets real origins; bare import gets safe default.
**Rule:** Any function called from `create_app()` must either (a) not call
`get_settings()`, or (b) handle settings-unavailable gracefully with a fallback.
Moving to lifespan doesn't work for CORSMiddleware because TestClient skips lifespan
and contract tests expect CORS to be registered at construction time.

## L009: Missing CRUD endpoints not caught until contract tests
**Date:** 2026-03-02
**Spec:** Audit Area 12
**Distilled to:** api-R009
**Reviewed:** 2026-03-09
**What happened:** Case studies had Create/Read/Update/List but no DELETE endpoint.
The service layer (`case_studies.py`) also lacked a `delete_case_study` function.
Contract test `test_case_studies_delete_endpoint` encoded this requirement.
**Root cause:** CRUD endpoints were built across multiple specs/sessions. DELETE was
specified in the audit but never implemented. The service and API layers were both
incomplete — not just a wiring issue.
**Rule:** When building CRUD for any resource, implement all five operations (Create,
Read, Update, Delete, List) in the same commit. Check contract tests for completeness
assertions before considering a resource "done."

## L010: Prefect deployment config needs more than just entrypoints
**Date:** 2026-03-02
**Spec:** Audit Area 12
**Distilled to:** flow-R005
**Reviewed:** 2026-03-09
**What happened:** `prefect.yaml` existed but was minimal — only entrypoints, work pool,
schedule, and concurrency_limit. Missing: descriptions, tags, timeout_seconds, retries,
retry_delay_seconds, parameters, and top-level pull/build/push steps.
**Root cause:** Initial `prefect.yaml` was a skeleton. Nobody went back to flesh it out
with the full deployment config that Prefect Cloud needs for proper scheduling, retry
behavior, and worker routing.
**Rule:** When creating deployment configs, match every parameter from the `@flow()`
decorator (timeout_seconds, retries) in the deployment YAML too. The YAML overrides
the decorator values in Prefect Cloud. Include pull steps for Railway containers
(`set_working_directory: /app`).

## L011: CLAUDE.md documentation drifts from reality
**Date:** 2026-03-02
**Spec:** N/A (meta)
**Distilled to:** N/A (process)
**Reviewed:** 2026-03-09
**What happened:** Multiple CLAUDE.md instructions were stale: prompts.py split threshold
said ~200 lines but the file was at 340+. Reference docs pointed to "V4 Data Contracts"
but specs dir had V5. LLM model names said "Claude Sonnet 4.5" rather than the actual
LiteLLM model IDs used in code. The `tasks/` directory existed but wasn't in the file
organization section. Commit scopes list was missing `llm` and `flows`.
**Root cause:** CLAUDE.md was written at project start and updated incrementally. Nobody
re-validated the full file against current project state after the fix passes.
**Rule:** After major milestones (spec completion, fix pass), re-read CLAUDE.md end-to-end
and verify every assertion against current code. Concrete checks:
- File size thresholds: `wc -l` the file and compare to stated limit
- Reference paths: `ls` the stated path and confirm it exists
- Model IDs: `rg "model.*=" src/soaria/llm/client.py` and compare to CLAUDE.md
- File organization: `ls src/soaria/` and confirm every directory is documented

## L012: `make check` silently modifies files before linting/testing
**Date:** 2026-03-02
**Spec:** N/A
**Distilled to:** N/A (process)
**Reviewed:** 2026-03-09
**What happened:** Running `make check` before commit auto-formatted files via `ruff format`
(the `fmt` target runs first). The formatted changes were unstaged, so `git diff` showed
changes that weren't in the commit. This could lead to committing unformatted code while
`make check` passes — because it fixed the files after the staging point.
**Root cause:** Makefile `check` target chains `fmt lint test`. The `fmt` step mutates
files in-place. If you stage files, then run `make check`, the formatter may change staged
files, creating a delta between the staged version and the working copy.
**Rule:** After running `make check`, always check `git diff` for format changes. Stage
them before committing. Or: run `make fmt` first, stage, then run `make lint test`.

## L013: Spec references to deleted modules cause confusion
**Date:** 2026-03-02
**Spec:** #5, #6
**Distilled to:** api-R001
**Reviewed:** 2026-03-09
**What happened:** Specs #5 and #6 reference `soaria.api.auth` for authentication
dependencies. The `auth.py` module was deleted during auth consolidation and replaced
by `soaria.api.deps`. New sessions following the spec literally would create a phantom
`auth.py` or fail trying to import from it.
**Root cause:** Specs are static documents written before the auth consolidation fix pass.
They were never updated to reflect the new canonical module.
**Rule:** When consolidating or renaming modules, add a note to the top of CLAUDE.md's
Current State section documenting the rename. Specs are treated as immutable (we don't
edit them), but CLAUDE.md overrides them. The Current State section is the source of truth
for module locations.

## L014: Dockerfile CMD replaced by HEALTHCHECK CMD — silent death
**Date:** 2026-03-02
**Spec:** N/A (deploy fix)
**Distilled to:** N/A (deployment)
**Reviewed:** 2026-03-09
**What happened:** Commit `76ae94c` ("trying to fix healthcheck") replaced the
Dockerfile `CMD` line with `HEALTHCHECK CMD`. These are completely different
directives — `CMD` starts the main process, `HEALTHCHECK CMD` defines a periodic
probe. The container booted with no process to run and exited immediately. In the
same fix pass, commit `f73f0f4` removed `startCommand` from `railway.toml`,
eliminating the fallback. Combined: zero server start commands.
**Root cause:** Confusion between `CMD` and `HEALTHCHECK CMD` syntax. The intent
was to add a healthcheck, but it overwrote the startup command instead of adding
a separate line.
**Rule:** Dockerfile must always have BOTH:
1. `CMD python -m uvicorn soaria.main:app --host 0.0.0.0 --port ${PORT:-8000}`
2. `HEALTHCHECK CMD curl --fail http://localhost:${PORT:-8000}/health || exit 1`
After any Dockerfile change, verify with: `grep "^CMD " Dockerfile` — must return
a uvicorn command. If it doesn't, the deploy is broken.

## L015: HS256 JWT secret deprecated — migrate to JWKS/RS256
**Date:** 2026-03-02
**Spec:** #5 (auth)
**Distilled to:** api-R010
**Reviewed:** 2026-03-09
**What happened:** Supabase migrated to asymmetric JWT signing keys (RS256). The
`supabase_jwt_secret` setting was required in Settings but the corresponding env
var no longer exists in Supabase dashboard. Auth was using `python-jose` with HS256
which doesn't support JWKS.
**Root cause:** Supabase changed their JWT signing approach from symmetric (HS256)
to asymmetric (RS256). The old `SUPABASE_JWT_SECRET` env var is gone.
**Fix:** Created `api/auth.py` with `PyJWKClient` for JWKS-based RS256 verification.
Made `supabase_jwt_secret` optional in Settings (defaults to "") for HS256 fallback
in tests. Replaced `python-jose` with `PyJWT[crypto]`. The `_decode_jwt` function
reads the algorithm from the token header and routes to JWKS (RS256/ES256) or
legacy HS256 automatically.
**Rule:** JWT verification should use JWKS endpoint, not static secrets. The JWKS
client is cached at module level and lazy-initialized on first use (never at import
time). When Supabase changes auth infra, check their `/auth/v1/.well-known/jwks.json`
endpoint.

## L016: Sync helper functions called from async without run_sync wrapping
**Date:** 2026-03-02
**Spec:** #8 (optimization)
**Distilled to:** api-R004, svc-R002
**Reviewed:** 2026-03-09
**What happened:** `optimize_resume_for_job` in `services/optimization.py` is `async def`
and correctly wrapped its *direct* `.execute()` calls in `await run_sync(...)`. However,
it called 5 sync helper functions (`check_idempotency`, `_gather_context`,
`_get_next_version_number`, `_get_baseline_version_id`, `_fetch_deltas`) and 2
`BudgetTracker` methods (`check_budget`, `record_call`) directly without wrapping.
Each of these sync helpers internally calls `.execute()`, blocking the event loop for
the duration of every Supabase round-trip.
**Root cause:** The helpers were extracted as sync functions (correct for their internal
design), but the caller forgot to wrap them in `await run_sync()`. The direct
`.execute()` calls within the main function *were* wrapped — creating a false sense of
completeness. The audit rule ("grep for `.execute()` in services/") catches direct
calls but misses calls nested inside sync helpers invoked from async contexts.
**Rule:** When an `async def` function calls a sync helper that does `.execute()`,
the call to that helper must be wrapped in `await run_sync(helper, ...)`. The audit
check should be expanded: not just grep for `.execute()` in async functions, but also
check that sync functions *called from* async functions are wrapped. A simple heuristic:
any function in services/ that calls `.execute()` should either be (a) `async def` with
`run_sync` wrapping, or (b) only called via `await run_sync(fn, ...)` or
`await asyncio.to_thread(fn, ...)`.

## L017: Contract test expects specific export names — aliases required
**Date:** 2026-03-02
**Spec:** #7 (applications)
**Distilled to:** svc-R004
**Reviewed:** 2026-03-09
**What happened:** `services/applications.py` defined `VALID_TRANSITIONS` and
`validate_transition`, but the contract test imported `VALID_STATUS_TRANSITIONS`
and `validate_status_transition`. The test failed with `ImportError` / `AssertionError`.
**Root cause:** The contract test was written to expect the longer, more explicit names.
The implementation used shorter names. Neither was "wrong" — they just didn't agree.
**Rule:** Before implementing a service module, read the contract tests for it:
`rg "from soaria.services.<module> import" tests/test_contract_integration.py`.
Use the exact export names the tests expect. If you prefer shorter names internally,
create the test-expected name first and alias the short name from it.

---

## L018: Duplicate Supabase client helpers across API files
**Date:** 2026-03-02
**Spec:** N/A (refactor)
**Distilled to:** api-R002, api-R003
**Reviewed:** 2026-03-09
**What happened:** Every API file had its own local helper (`_get_user_supabase`,
`_user_client`, `_admin_supabase`) to create a user-scoped Supabase client.
**Root cause:** No canonical location existed for this dependency when the first
API files were built; each developer copy-pasted the pattern.
**Rule:** User-scoped Supabase client creation lives in `api/deps.py` as
`get_user_supabase(request: Request)`. Never create a local helper for this —
import from deps. When refactoring, update test mock targets from
`soaria.api.<module>.get_supabase_client` to `soaria.api.<module>.get_user_supabase`.

---

## L019: Test fixtures must match model constraints exactly
**Date:** 2026-03-02
**Spec:** #8A
**Distilled to:** mdl-R005
**Reviewed:** 2026-03-09
**What happened:** 7 case study tests failed because `_cs_row()` returned `"updated_at": None`, but `CaseStudyResponse.updated_at` is `datetime` (required, NOT NULL per V6 data contract).
**Root cause:** Fixture was written assuming optional field; model was later tightened to required. Nobody re-checked fixture data against the Pydantic model.
**Rule:** When writing test fixtures, verify every field against both the Pydantic model AND the DB schema. Required fields with NOT NULL + DEFAULT NOW() must have realistic timestamps, never None.

---

## L020: Test function signatures must match service signatures
**Date:** 2026-03-02
**Spec:** #8A
**Distilled to:** tst-R009
**Reviewed:** 2026-03-09
**What happened:** `test_create_resume_template_success` called `create_resume_template(client, user_id=..., data=data)` but the actual signature is `create_resume_template(supabase, *, user_id, template_name, job_focus, docx_content)`. The service was refactored to DOCX-first but the test was never updated.
**Root cause:** Service API changed during Spec #8 DOCX-first redesign; test was written for the old `data=ResumeTemplateCreate(...)` API.
**Rule:** After any service function signature change, grep for all callers including tests. `rg "create_resume_template" tests/` catches stale test calls.

---

## L021: PyJWT HS256 requires 32+ byte secrets
**Date:** 2026-03-02
**Spec:** N/A (test infra)
**Distilled to:** tst-R010
**Reviewed:** 2026-03-09
**What happened:** 65 warnings per test run because test JWT secret was 30 bytes ("test-jwt-secret-for-unit-tests"). PyJWT 2.11+ warns on HMAC keys < 32 bytes per RFC 7518 Section 3.2.
**Root cause:** Secret was chosen for readability without checking byte length.
**Rule:** Test JWT secrets must be >= 32 bytes. Current value: `"test-jwt-secret-for-unit-tests-XX"`. Update both `_helpers.py` and `conftest.py` together.

---

## L022: supabase-py result.data has overly broad union type
**Date:** 2026-03-02
**Spec:** N/A (type safety)
**Distilled to:** svc-R005
**Reviewed:** 2026-03-09
**What happened:** 564 mypy errors in `services/admin.py` because `result.data` is typed as `list[JSON]` where JSON is `bool | str | int | float | Sequence[JSON] | Mapping[str, JSON] | None`. Every `.get()` and `[]` access triggers union-attr errors.
**Root cause:** supabase-py stubs use a recursive JSON union type. In practice, table queries always return `list[dict[str, Any]]`.
**Rule:** Use a `_rows()` helper to type-narrow `result.data` once: `def _rows(data: Any) -> list[dict[str, Any]]: return data if data else []`. Also: `count="exact"` needs `# type: ignore[arg-type]` since the stubs expect `CountMethod | None` but the runtime accepts strings. Never use bare `dict` in type annotations — always `dict[str, Any]`.

---

## L023: supabase-py incompatible with new Supabase API keys (sb_secret_...)
**Date:** 2026-03-02
**Spec:** N/A (deploy/infra)
**Distilled to:** N/A (infra)
**Reviewed:** 2026-03-09
**What happened:** Prefect worker flows failed with 401 "Unregistered API key" when
using new-format Supabase secret keys (`sb_secret_...`). The API service appeared
unaffected because user-facing endpoints use the anon/publishable key + user JWT,
rarely hitting the service_role path.
**Root cause:** supabase-py v2.28.0 sends the key in **both** `apiKey` and
`Authorization: Bearer <key>` headers (see `Client._get_auth_headers`). The new
`sb_secret_...` keys are opaque tokens, not JWTs. Supabase's PostgREST accepts them
in the `apiKey` header but rejects them in `Authorization: Bearer` because they're
not valid JWTs. The legacy `eyJ...` keys are JWTs and work in both headers.
**Fix:** Reverted to legacy API keys (anon + service_role, both `eyJ...` format) on
both Railway services. The new keys can be re-adopted once supabase-py adds support.
**Rule:** Until supabase-py supports the new key format, use legacy JWT-based keys
(`eyJ...`) for both `SUPABASE_ANON_KEY` and `SUPABASE_SERVICE_ROLE_KEY`. Check
supabase-py releases for "new API key" or "sb_secret" support before switching.

---

## L024: Prefect task cache_policy=NONE required for non-serializable inputs
**Date:** 2026-03-02
**Spec:** N/A (Prefect flows)
**Distilled to:** flow-R003
**Reviewed:** 2026-03-09
**What happened:** All Prefect flow runs failed with `HashError: Unable to serialize
unknown type: <class 'supabase._sync.client.Client'>`. Prefect 3.x tries to hash
all task inputs for caching by default.
**Root cause:** Tasks in greenhouse_sync.py, embed_jobs.py, and rank_jobs.py accept
`supabase.Client`, `asyncpg.Pool`, `httpx.AsyncClient`, and `asyncio.Semaphore` as
arguments. These contain thread locks and connection state that can't be pickled or
JSON-serialized.
**Fix:** Added `cache_policy=NONE` (from `prefect.cache_policies`) to all `@task`
decorators across all 3 flows.
**Rule:** Any `@task` that receives non-serializable objects (DB clients, connection
pools, HTTP clients, semaphores) must use `cache_policy=NONE`. When creating new
Prefect tasks, check if any parameter is a client/pool/lock type and add the policy.

---

## L025: Graduate content when specs are done
**Date:** 2026-03-02
**Spec:** N/A (meta/process)
**Distilled to:** N/A (process)
**Reviewed:** 2026-03-09
**What happened:** After all specs and fixes were implemented, actionable content
was scattered across 11 spec files, 6 fix files, and this lessons file. Reference
docs (ARCHITECT_REFERENCE.md, SPEC_DIGEST.md) tried to be both historical and
current, and drifted from reality as a result.
**Root cause:** No formal process for pulling actionable rules out of specs into
permanent, directory-scoped documentation. Specs are historical artifacts —
they preserve design context and rejected alternatives. But the rules they
establish (e.g., "always use get_user_supabase") need to live where agents
will find them when working in that directory.
**Fix:** Created AGENTS.md files in `api/`, `services/`, `flows/`, `llm/`, `models/`
with distilled rules from specs and lessons. Created `specs/SUMMARY.md` as the
canonical spec index. Created `how-we-document.md` defining what goes where.
Froze ARCHITECT_REFERENCE.md and SPEC_DIGEST.md as historical snapshots.
**Rule:** When a spec reaches "done," graduate its actionable content into
AGENTS.md files (directory-scoped rules) and CLAUDE.md (cross-cutting rules).
Leave the spec as a historical artifact — never modify it, but never rely on
it for current conventions either. The anti-pattern is letting reference docs
try to be both historical and current.

---

## L026: Patch db module functions, not asyncpg conn methods, in service tests
**Date:** 2026-03-02
**Spec:** #10C/D/E
**Distilled to:** svc-R008
**Reviewed:** 2026-03-09
**What happened:** When migrating tests from supabase mocks to asyncpg, two approaches
emerged: (A) mock `conn.fetchrow`/`conn.fetch` with `side_effect` to choreograph
return values across multiple sequential calls within a single `pool.acquire()` block,
or (B) patch the `db/<domain>.py` functions directly (e.g., `soaria.db.resumes.get_resume_template`).
**Root cause:** Services that do multiple db calls in one `async with pool.acquire() as conn:`
block (e.g., verify ownership then fetch version) require complex `side_effect` chains
if mocking at the conn level — fragile and hard to read.
**Rule:** For service-layer unit tests, patch at the db module level:
`patch("soaria.db.resumes.get_resume_template", new_callable=AsyncMock, return_value=row)`.
This works because the service imports the db module (`from soaria.db import resumes as db_resumes`)
and accesses functions on it — patching the module attribute is seen by all references.
Reserve conn-level mocks for db-layer tests only.

---

## L027: Flow callers must be updated when service signatures change
**Date:** 2026-03-02
**Spec:** #10C/D/E
**Distilled to:** svc-R008
**Reviewed:** 2026-03-09
**What happened:** After migrating `services/embeddings.py` to take `pool: DatabasePool`
instead of `supabase: Client`, `flows/embed_jobs.py` still passed `supabase` to the
service functions (`fetch_unembedded_jobs(supabase, limit=limit)`). The task also still
declared `supabase: Client` as a parameter and the main flow still called
`get_supabase_client()` to create it. Three-layer mismatch: flow → task → service.
**Root cause:** Service migration happened in a previous session; flow wiring was
deferred. The type mismatch would only surface at runtime (no static check since both
`Client` and `DatabasePool` are classes that Python wouldn't reject positionally).
**Rule:** When changing a service function signature, immediately grep for all callers:
`rg "fetch_unembedded_jobs" src/soaria/` — update every call site in the same chunk.
Flows are easy to miss because they live in a different directory from services.

---

## L028: Standard pool mock helper for asyncpg tests
**Date:** 2026-03-02
**Spec:** #10C/D/E
**Distilled to:** svc-R008
**Reviewed:** 2026-03-09
**What happened:** Three test files independently needed to mock `DatabasePool.acquire()`
as an async context manager. The pattern is non-obvious (requires a class implementing
`__aenter__`/`__aexit__`) and was duplicated across files.
**Rule:** Use this standard helper in any test that needs a pool mock:
```python
def _make_pool_mock() -> MagicMock:
    conn = AsyncMock()
    pool = MagicMock()
    class _Ctx:
        async def __aenter__(self): return conn
        async def __aexit__(self, *a): pass
    pool.acquire.return_value = _Ctx()
    return pool
```
Consider moving to `tests/conftest.py` as a shared fixture once 3+ test files use it.

---

## L029: print() in src/ breaks contract test
**Date:** 2026-03-02
**Spec:** #12
**Distilled to:** db-R007
**Reviewed:** 2026-03-09
**What happened:** `db/__main__.py` (CLI tool) used `print()` for user-facing
output. The contract test `test_no_print_statements_in_source` scans all files
under `src/soaria/` and fails if any contain `print(`.
**Root cause:** CLI tools naturally want `print()` for output, but the project
rule is "no print, use structlog." The contract test enforces this without
exceptions.
**Rule:** Never use `print()` in any file under `src/soaria/`. For CLI tools
that need stdout output, use `sys.stdout.write(msg + "\n")`. For stderr,
use `sys.stderr.write(msg + "\n")`. Create `_out()` and `_err()` helpers.

---

## L030: curl not available in python:3.12-slim — use urllib.request
**Date:** 2026-03-02
**Spec:** #13
**Distilled to:** N/A (deployment)
**Reviewed:** 2026-03-09
**What happened:** HEALTHCHECK used `curl --fail http://localhost:$PORT/health` but
`python:3.12-slim` doesn't include curl. The healthcheck silently failed.
**Root cause:** The Dockerfile was written assuming curl exists in the base image.
`python:3.12-slim` strips most system utilities including curl and wget.
**Rule:** For HEALTHCHECK in slim Python images, use Python's stdlib:
`python -c "import urllib.request; urllib.request.urlopen('http://localhost:${PORT:-8000}/health')"`
This works in any Python image without extra packages.

---

## L031: API test files must be updated when API modules drop supabase imports
**Date:** 2026-03-03
**Spec:** #10K
**Distilled to:** tst-R011
**Reviewed:** 2026-03-09
**What happened:** After migrating `api/admin.py` and `api/applications.py` to asyncpg
(removing `get_user_supabase` imports), 44 API-level tests failed because they patched
`soaria.api.admin.get_user_supabase` — an attribute that no longer existed on the module.
**Root cause:** Chunks 10H/10I migrated the API modules but left the API test files
unchanged. The service-layer tests were updated but the API-layer tests (which use
`TestClient` and patch at a different level) were not.
**Rule:** When removing an import from a module, immediately search for test patches
targeting that import: `rg "patch.*soaria.api.admin.get_user_supabase" tests/`. Every
patch targeting a removed attribute will cause `AttributeError` at test time. Migration
chunks must include test updates for all layers (db, service, AND api).

---

## L032: Gut re-export modules when migration completes — don't leave backward-compat shims
**Date:** 2026-03-03
**Spec:** #10K
**Distilled to:** svc-R011
**Reviewed:** 2026-03-09
**What happened:** `database.py` re-exported `DatabasePool` and `db_pool` from `soaria.db.pool`
for backward compatibility during the migration. Seven source files and three test files
still imported from the old path. This created two valid import paths for the same symbol,
making it unclear which was canonical.
**Root cause:** Re-exports are useful during migration but become tech debt if left
indefinitely. Nothing enforced cleaning them up.
**Rule:** When a migration completes, immediately gut the compatibility shim. Search for
all imports from the old module (`rg "from soaria.database import" src/ tests/`) and
update them to the canonical path. The old module should contain only symbols that
genuinely belong there (e.g., `get_supabase_client` stays in `database.py` because it
wraps `supabase-py`, not `asyncpg`).

---

## L033: `_mock_pool()` is the standard pattern — use AsyncMock for context manager
**Date:** 2026-03-03
**Spec:** #10K
**Distilled to:** tst-R012
**Reviewed:** 2026-03-09
**What happened:** The L028 pattern using a nested class for `__aenter__`/`__aexit__` was
replaced by a simpler approach in all migrated test files:
```python
def _mock_pool():
    pool = MagicMock()
    conn = AsyncMock()
    ctx = AsyncMock()
    ctx.__aenter__ = AsyncMock(return_value=conn)
    ctx.__aexit__ = AsyncMock(return_value=False)
    pool.acquire.return_value = ctx
    pool._conn = conn
    return pool
```
**Rule:** Use this `_mock_pool()` pattern (all AsyncMock, no custom class) for pool mocks.
The `pool._conn` attribute lets tests access the mock connection for setup. This is now
the de facto standard across `test_applications_service.py`, `test_optimization_service.py`,
`test_ranking_service.py`, `test_admin_service.py`, and `test_resumes_api.py`.

---

## L034: LiteLLM load_dotenv poisons os.environ during test collection
**Date:** 2026-03-03
**Spec:** Cross-cutting (test infrastructure)
**Distilled to:** tst-R013
**Reviewed:** 2026-03-09
**What happened:** `test_cors_middleware_present` passed in isolation but failed when
run as part of the full test suite (`pytest tests/`). The CORS middleware was configured
with `allow_origins=[""]` instead of `["http://localhost:5173"]`.
**Root cause:** LiteLLM calls `dotenv.load_dotenv()` at import time
(`litellm/__init__.py:82`), which reads `.env` and sets ALL values into `os.environ` —
including `CORS_ORIGINS=""` (empty in `.env`). This happens during test collection when
any test file transitively imports `litellm`. The contract test's `_set_test_env` fixture
used `os.environ.setdefault()`, which is a no-op when the key already exists (even with
empty value). So the empty string from `.env` persisted into the app's CORS config.
**Rule:** Never use `os.environ.setdefault()` in test fixtures — third-party libraries
(especially LiteLLM) may call `load_dotenv()` at import time, poisoning `os.environ`
with empty `.env` values during collection. Always use direct assignment
(`os.environ[key] = val`) or `monkeypatch.setenv()`. The conftest.py `set_test_env`
autouse fixture must include ALL env vars that appear in `.env`, including
`CORS_ORIGINS`.

---

## L035: TestClient re-raises server exceptions by default
**Date:** 2026-03-03
**Spec:** #15
**Distilled to:** tst-R014
**Reviewed:** 2026-03-09
**What happened:** Test `test_admin_can_soft_delete_users` failed because
FastAPI's TestClient (Starlette) has `raise_server_exceptions=True` by default.
When the endpoint raised `RuntimeError` (pool not initialized in unit tests),
the TestClient propagated the exception instead of returning HTTP 500.
**Root cause:** TestClient default behavior differs from real HTTP — exceptions
bubble up instead of being caught by ASGI error handling.
**Rule:** When testing that a route exists but may hit infrastructure errors
(like uninitialized pool), wrap the call in `try/except RuntimeError` or use
`TestClient(app, raise_server_exceptions=False)`.

---

## L036: Ruff RUF001 catches ambiguous Unicode characters
**Date:** 2026-03-03
**Spec:** #16
**Distilled to:** N/A (code standards)
**Reviewed:** 2026-03-09
**What happened:** Test strings using en-dash (`\u2013`) in date ranges like
`"2020 – Present"` triggered RUF001 lint errors.
**Root cause:** Ruff's RUF001 rule flags ambiguous Unicode characters that look
like ASCII but aren't (en-dash vs hyphen).
**Rule:** Use ASCII hyphen-minus (`-`) in test data strings, not en-dash (`\u2013`).
Reserve Unicode dashes only in production data that mirrors real documents.

---

## L037: Local migration files don't guarantee live DB state
**Date:** 2026-03-03
**Spec:** N/A (schema drift)
**Distilled to:** N/A (deployment)
**Reviewed:** 2026-03-09
**What happened:** Migration 001 defined `llm_call_log`, `admin_dashboard_stats()`,
`admin_llm_budget()`, and the `account_status` CHECK with 'deleted'. But the live
Supabase DB had none of these — the migration was never applied. Code referencing
these objects would fail at runtime despite the migration file existing locally.
**Root cause:** Migrations were consolidated into 001 from multiple historical SQL
files, but the consolidation happened after the DB was already provisioned. The local
migration runner (`make migrate`) was never run against production — schema changes
were applied ad-hoc via Supabase SQL Editor during early development.
**Rule:** After adding or modifying any migration file, verify the change exists in
the live DB using the Supabase MCP or SQL Editor. Local migration files are the
*intended* state; the live DB is the *actual* state. Use `/schema-check` to compare
them. The `TestSchemaAlignment` contract tests catch some drift at dev time, but
live DB verification requires MCP access.

---

## L038: Lovable frontend lock-in creates dual-database architecture
**Date:** 2026-03-04
**Spec:** #17
**Distilled to:** N/A (product context)
**Reviewed:** 2026-03-09
**What happened:** The Lovable-hosted frontend provisions its own Supabase project
that we cannot access or configure. The frontend queries 14 tables directly via
`supabase.from()`, bypassing the FastAPI API entirely. This means rate limiting,
budget tracking, audit logging, and business rules in the API are not enforced.
Four Edge Functions duplicate backend logic. Two separate databases exist with no
single source of truth.
**Root cause:** Lovable's deployment model couples the frontend to a Lovable-managed
Supabase instance. This was fine for prototyping but creates architectural debt:
direct DB queries bypass the API, auth is on a separate Supabase project from the
backend, and we have no control over RLS policies or schema on the frontend's DB.
**Rule:** When choosing a frontend platform, verify that it allows you to control
your own auth provider and database. If the platform provisions infrastructure you
cannot access, plan the migration path before building on it. For Soaria: the
frontend must use the backend API for all data access, with Supabase used only for
authentication (login, logout, session refresh).

---

## L039: Ruff B905 requires explicit `strict=` on `zip()`
**Date:** 2026-03-06
**Spec:** M2 alignment
**Distilled to:** N/A (code standards)
**Reviewed:** 2026-03-09
**What happened:** `zip(a, b)` without `strict=` parameter failed lint.
**Root cause:** Ruff B905 rule enforces explicit `strict=True` or `strict=False`
on all `zip()` calls to prevent silent data loss from unequal-length iterables.
**Rule:** Always pass `strict=True` (when lengths must match) or `strict=False`
(when truncation is intentional) to `zip()`.

---

## L040: Circular import when prompt module imports from service module
**Date:** 2026-03-06
**Spec:** M2 alignment (C2)
**Distilled to:** llm-R005
**Reviewed:** 2026-03-09
**What happened:** Moving `ROLE_CLUSTERS` constant to `services/ranking.py` and importing
it in `llm/prompts/ranking.py` created a circular import:
`services/ranking.py` → `llm/prompts/__init__` → `llm/prompts/ranking.py` → `services/ranking.py`
**Root cause:** The prompt module (`llm/prompts/ranking.py`) and the service module
(`services/ranking.py`) already have a dependency: service imports the prompt builder,
and the prompt `__init__` re-exports from ranking.py. Adding a reverse import (prompt → service)
closed the cycle.
**Rule:** Constants shared between prompts and services must live in the prompt module
(which is lower in the dependency graph), not in the service module. The service imports
from prompts; prompts must never import from services (except `services/embeddings.py`
for `strip_html`, which has no reverse dependency).

---

## L041: SQL INSERT references non-existent column and violates CHECK constraint
**Date:** 2026-03-06
**Spec:** M2 alignment (feed state tracking)
**Distilled to:** db-R012
**Reviewed:** 2026-03-09
**What happened:** `trigger_feed_refresh()` in `services/feed.py` inserted into
`feed_refresh_log` with a `requested_at` column that doesn't exist (table only has
`created_at` with DEFAULT NOW()). Additionally, it used `status='pending'` but the
CHECK constraint only allowed `'completed', 'failed', 'partial'`. Both bugs would
cause runtime errors but were invisible because the test mocked the pool.
**Root cause:** The INSERT was written from memory of the intended schema rather than
verified against the actual migration (001). Subagent-driven development across
multiple sessions means the author of the feed service didn't cross-check the
migration file.
**Rule:** Before writing any INSERT/UPDATE, verify column names and CHECK constraints
against the migration file: `rg "CREATE TABLE.*<table>" migrations/`. For CHECK
constraints, also run: `rg "CHECK.*<column>" migrations/`. Never assume a column or
enum value exists — verify it.

---

## L042: LLM output parsed as UUID without error handling
**Date:** 2026-03-06
**Spec:** M2 alignment (optimization)
**Distilled to:** llm-R006
**Reviewed:** 2026-03-09
**What happened:** `UUID(cs_id)` was called on LLM-returned `caseStudiesUsed` array
without try/except. If the LLM returns a malformed string instead of a valid UUID,
the entire optimization call crashes with ValueError.
**Root cause:** LLM output was treated as trusted input. The list comprehension
assumed all values would be valid UUIDs.
**Rule:** All LLM-returned values that are parsed into typed objects (UUID, int,
datetime) must be wrapped in try/except. LLM output is untrusted external input —
validate it the same way you would user input.

---

## L043: LLM call inside pool.acquire() exhausts connection pool
**Date:** 2026-03-06
**Spec:** Pilot hardening
**Distilled to:** svc-R010
**Reviewed:** 2026-03-09
**What happened:** `optimization_chat.py:send_message()` held a DB connection for the
entire duration of an LLM `completion()` call (10-30s). With a 10-connection pool and
no acquisition timeout, 7+ concurrent chat users would exhaust the pool. All subsequent
requests — including non-chat endpoints — would hang indefinitely.
**Root cause:** The function used a single `async with pool.acquire()` block for the
entire read→LLM→write flow. The LLM call doesn't need a DB connection but was inside
the block for convenience.
**Rule:** Never hold a DB connection during an LLM call, HTTP call, or any operation
>100ms. Split into: acquire→read→release → long operation → acquire→write→release.
Verify with: `rg "await completion|await embed" src/soaria/` and check that none appear
inside a `pool.acquire()` block.

---

## L044: asyncpg pool has no acquisition timeout by default
**Date:** 2026-03-06
**Spec:** Pilot hardening
**Distilled to:** db-R010
**Reviewed:** 2026-03-09
**What happened:** `asyncpg.create_pool()` was called without a `timeout` parameter.
When the pool was exhausted, `pool.acquire()` would wait indefinitely — no error, no
timeout, just a silent hang. Combined with L043, this turned a pool exhaustion into a
total API outage.
**Root cause:** asyncpg defaults `timeout` to `None` (infinite wait). The pool was
created with only `min_size` and `max_size`.
**Rule:** Always pass `timeout=5.0` (or similar) to `asyncpg.create_pool()`. This
applies to both the app pool (`DatabasePool.connect()`) and flow pools
(`create_flow_pool()`). When timeout fires, asyncpg raises `asyncio.TimeoutError`
which surfaces as a 500 — much better than an infinite hang.

---

## L045: Unbounded text fields enable LLM token burn and DB bloat
**Date:** 2026-03-06
**Spec:** Pilot hardening
**Distilled to:** mdl-R006
**Reviewed:** 2026-03-09
**What happened:** Case study fields (situation, task, action, result) had no
`max_length` constraint. A user could POST 10MB of text per field (up to the 10MB
body limit). This text feeds into LLM prompts for ranking and optimization, burning
tokens and potentially hitting context window limits.
**Root cause:** Pydantic models were defined with bare `str` types and no `Field()`
constraints. The 10MB body size middleware is a blunt instrument — it doesn't protect
individual fields.
**Rule:** All user-facing text fields in request models must have `Field(max_length=N)`.
Guidelines: titles/names 500, reasons 1K, chat messages 5K, free-text fields 10K,
STAR story sections 50K. Response models are exempt (they reflect DB state).

---

## L046: Flows using app db_pool singleton instead of independent pools
**Date:** 2026-03-06
**Spec:** FIX_07
**Distilled to:** flow-R008
**Reviewed:** 2026-03-09
**What happened:** `embed_jobs.py` and `rank_jobs.py` used `pool = db_pool`
(the app singleton) instead of creating an independent pool. This works
in-process during development but would fail on Prefect Cloud workers where
the singleton was never initialized (`connect()` never called). Additionally,
`send_reminders.py` used `create_flow_pool()` which returns raw `asyncpg.Pool`,
but its task expected `DatabasePool` — a type mismatch that only worked via
duck typing.
**Root cause:** Flows were written when the pool architecture was still evolving.
`create_flow_pool()` was modeled after `greenhouse_sync.py` which uses raw
`asyncpg.Pool`, creating a type gap with services that expect `DatabasePool`.
**Rule:** Flows must create their own `DatabasePool` via `DatabasePool.from_dsn()`
with try/finally for cleanup. Never import `db_pool` in flows. Verify:
`grep "db_pool" src/soaria/flows/` should return zero hits.

---

## L047: Singleton access in dependencies blocks test override
**Date:** 2026-03-06
**Spec:** FIX_07
**Distilled to:** api-R011
**Reviewed:** 2026-03-09
**What happened:** `get_current_user` in `deps.py` used `pool = db_pool`
directly instead of `Depends(get_db_pool)`. `health.py` readiness probe and
`services/admin.py:force_sync_board` also accessed the singleton. Tests had to
patch module-level attributes instead of using FastAPI's dependency override.
**Root cause:** The singleton was convenient during initial development. When
`get_db_pool()` was added as a dependency wrapper, existing code wasn't updated.
**Rule:** API endpoints and dependencies must receive the pool via
`Depends(get_db_pool)`. Service functions must receive pool as a parameter.
Never import `db_pool` directly in API or service modules. The singleton is
only for: `deps.py:get_db_pool()` (the wrapper), `main.py` (lifespan), and
`llm/client.py` (deep infrastructure with lazy import).

---

## L048: Error messages leak internal details to API clients
**Date:** 2026-03-06
**Spec:** Security audit
**Distilled to:** api-R012
**Reviewed:** 2026-03-09
**What happened:** Multiple API endpoints passed `str(exc)` directly into HTTPException
`detail` fields: auth.py leaked PyJWT error messages ("Signature verification failed"),
health.py leaked DB connection errors, budget.py leaked spend amounts and env var names,
admin.py passed ValueError messages from services directly to clients.
**Root cause:** Exception handlers were written for developer convenience (show the real
error) rather than security (never leak internals). The `detail=str(exc)` pattern is
especially dangerous because it automatically leaks any future service error changes.
**Rule:** Never pass `str(exc)` to HTTPException detail. Use static messages for clients
("Invalid or expired token", "Resource not found", "Invalid request") and log the real
error server-side via structlog. Auth errors get generic messages + `from None` to suppress
exception chains. Health check errors return "unavailable" with no infrastructure details.

---

## L049: LLM outputs must be type-converted defensively
**Date:** 2026-03-06
**Spec:** Security audit
**Distilled to:** llm-R006
**Reviewed:** 2026-03-09
**What happened:** `parse_optimization_response()` called `float(d.get("confidence", 0.5))`
and `int(raw_response.get("qualityScore", 5))` directly. If the LLM returned `"high"` or
`"8 out of 10"`, these crash with ValueError before clamping is applied.
**Root cause:** LLM output was treated as well-typed after JSON parsing. But JSON values
can be strings even when numbers are expected — LLMs are unreliable about type consistency.
**Rule:** All type conversions on LLM output (`int()`, `float()`, `UUID()`) must be wrapped
in try/except with sensible defaults. Use helpers like `_safe_float(value, default)` and
`_safe_int(value, default)`. This extends L042 (UUID parsing) to all typed conversions.

---

## L050: Budget configure_budget() used wrong config field for warn_percent
**Date:** 2026-03-06
**Spec:** Security audit
**Distilled to:** llm-R007
**Reviewed:** 2026-03-09
**What happened:** `main.py` lifespan called `configure_budget(warn_percent=settings.llm_budget_kill_threshold_percent)` — passing the kill switch threshold (80%) instead of the
warning threshold (50%). Warnings didn't fire until 80% budget spent, defeating early alerting.
**Root cause:** Two similarly-named config fields (`llm_budget_warn_percent` and
`llm_budget_kill_threshold_percent`) and no test exercising the lifespan wiring.
**Rule:** When wiring config fields to function parameters, verify field names match
semantics, not just types. The kill switch auto-trigger works independently (reads settings
at runtime in `_check_auto_kill_switch`), but `warn_percent` for in-memory tracking
must use `llm_budget_warn_percent`.

---

## L051: LLM timeout fires but response access crashes before handler
**Date:** 2026-03-06
**Spec:** Security audit
**Distilled to:** llm-R008
**Reviewed:** 2026-03-09
**What happened:** `llm/client.py` added `timeout=60` to `litellm.acompletion()` but
didn't handle the timeout exception. When timeout fired, `litellm.Timeout` was raised,
`response` was never assigned, and `response.choices[0].message.content` on the next
line crashed with NameError/AttributeError. The generic `except Exception` at the bottom
only covered the call_log write, not the main call.
**Root cause:** Timeout was added as a parameter but the exception path wasn't considered.
The code assumed the litellm call would always return a response object.
**Rule:** When adding timeouts to external API calls, always add corresponding exception
handling. Wrap the call + response access in try/except, log the error, and re-raise.
The caller's existing error handling then works naturally. Same applies to embed().

---

## L052: 10s statement_timeout too aggressive for batch INSERT...ON CONFLICT
**Date:** 2026-03-06
**Spec:** Security audit
**Distilled to:** db-R011
**Reviewed:** 2026-03-09
**What happened:** `db/pool.py` set `statement_timeout: "10000"` (10s) at connection level.
`upsert_job_matches()` generates bulk `INSERT...ON CONFLICT` with 100+ rows and 18 columns
each. Under production load, this query can exceed 10s, causing `QueryCanceledError` and
failing the entire ranking flow despite the flow having a 600s timeout.
**Root cause:** Statement timeout was set conservatively without profiling batch operations.
Connection-level timeout applies uniformly — no per-query override.
**Fix:** Increased to 30s. This covers batch upserts while still protecting against runaway
queries. If specific queries need longer, use `SET LOCAL statement_timeout = '60000'` inside
a transaction.
**Rule:** When setting connection-level timeouts, profile the slowest legitimate query
(usually batch upserts or aggregations) and set the timeout above that. 30s is the current
baseline. Individual flows can override with SET LOCAL inside transactions.

---

## L053: Pydantic Literal types must match DB CHECK constraints
**Date:** 2026-03-11
**Spec:** #23
**Distilled to:** db-R013
**What happened:** Spec 23 adds `'anonymous'` to the `agent_sessions.agent_state` DB CHECK
constraint but doesn't update `PlannerInput.agent_state` Literal type. Planner would reject
anonymous sessions with a Pydantic validation error before any logic runs.
**Root cause:** DB schema and Pydantic models are separate concerns maintained in different
files. Adding a value to one without the other is easy to miss, especially when the CHECK
constraint is in a migration file and the Literal is in a models file.
**Rule:** When adding a value to a DB CHECK constraint, grep for the column name in models/
and update every Literal type that mirrors it. Verify with:
`rg "agent_state.*Literal" src/soaria/models/`

---

## L054: Cross-domain storage (sessionStorage/localStorage) doesn't work across origins
**Date:** 2026-03-11
**Spec:** Marketing Site
**Distilled to:** api-R019
**What happened:** Marketing site (getversed.ai) stores pasted text in sessionStorage and
redirects to app (app.getversed.ai). sessionStorage is origin-scoped — the app can't read
the marketing site's storage. Pasted text is silently lost.
**Root cause:** Web storage APIs are same-origin only. Different subdomains = different
origins. This is a fundamental browser security boundary, not a bug.
**Rule:** Never rely on sessionStorage or localStorage to transfer data between different
origins (including subdomains). Use a server-side relay (paste token pattern) or URL
parameters for cross-domain data transfer.

---

## L055: Integration seams need explicit contracts, not implicit assumptions
**Date:** 2026-03-11
**Spec:** #23, #24, #25
**Distilled to:** CLAUDE.md (Integration Rules / Cross-Boundary Contracts)
**What happened:** Foil review found 10+ issues at integration boundaries: SSE event types
not documented in FE types, field naming mismatches (source_phrase vs candidateText), session
binding info not delivered to FE, card interactions blocked for anonymous users. Every
individual component was correct — the wiring between them was wrong.
**Root cause:** Specs were written from each system's perspective. The BE spec says "emit
SSE events" and the FE spec says "render cards" but neither defines the contract between
them at field-level precision.
**Rule:** For every cross-boundary feature, write an explicit contract section in the spec:
exact field names, types, which side maps what, error cases. If the BE emits it and the FE
consumes it, the mapping must be documented before either side writes code. Check against
`docs/frontend-types.ts`.

---

## L056: Race conditions at status transitions need write-ordering semantics
**Date:** 2026-03-11
**Spec:** #25
**Distilled to:** db-R018
**What happened:** Trial end (sets status='free') and Stripe checkout completion (sets
status='pro') can race. If trial end runs after checkout, a paying user gets downgraded.
**Root cause:** Two independent systems (Prefect flow and Stripe webhook) write to the same
row without coordination. Last-write-wins means the slower system determines the outcome.
**Rule:** When two systems can write to the same status field concurrently, define which
one wins. Use WHERE guards (`AND stripe_subscription_id IS NULL`) so the lower-priority
write is a no-op when the higher-priority write has already happened. Document the semantics
explicitly: "Stripe-wins: paying user never gets downgraded."

---

## L057: Anonymous paths require full audit of every user_id NOT NULL assumption
**Date:** 2026-03-11
**Spec:** #23, #24, #25
**Distilled to:** api-R013
**What happened:** Foil review found 5 separate issues (2, 7, 12, 16, 18) all caused by the
same root: adding anonymous sessions means `user_id` can be NULL in places that assumed it
was always set. `ReframingPatternCreate.user_id` was required. `CardInteractionRequest` had
no anonymous auth path. `card_interactions` weren't transferred on claim. Service_role queries
weren't enumerated. Each was a different symptom of the same failure to audit.
**Root cause:** The anonymous path was treated as a single spec change (make user_id nullable)
rather than a cross-cutting concern that touches every table, model, service, and endpoint
referencing user_id.
**Rule:** When adding a new auth path (anonymous, API keys, service accounts), run a
systematic audit: `rg "user_id" src/soaria/models/ src/soaria/services/ src/soaria/api/`
and verify every NOT NULL assumption, every required field, and every auth dependency still
holds. The audit itself should be a section in the spec.

---

## L058: Coordination bootstrap replaces manual reads at session start
**Date:** 2026-03-12
**Spec:** N/A (infrastructure)
**Distilled to:** N/A (process)
**Reviewed:** 2026-03-12
**What happened:** Session-orient.sh upgraded to inject machine truth (timestamps, staleness, board summary, FE handoff). New coordination-gate.sh blocks writes to coordination dir without read marker and warns on stale reads (>30 min). SOURCES.md added as canonical truth index.
**Root cause:** Previously, reading coordination files was prose-enforced only — easy to skip or forget, leading to stale writes.
**Rule:** Session bootstrap handles coordination reads automatically. For long sessions, re-read coordination files before writing. Check SOURCES.md when unsure which file is authoritative.

---

## L059: Pydantic response models silently drop new DB columns
**Date:** 2026-03-12
**Spec:** #27, #30
**Distilled to:** mdl-R007
**Reviewed:** 2026-03-12
**What happened:** Migration added 5 columns to `applications` table (merge_application_id, vocabulary_translations_used, outcome_linked_at, last_synced_at, sync_source). None were added to `ApplicationResponse`/`ApplicationDetail`. Pydantic silently drops unknown fields — no error, no log. FE components render empty. Same pattern in Spec 30: `CohortStatusResponse` Pydantic model had 9 fields but the TS `CohortStatusPoll` contract had 3.
**Root cause:** Migration DDL and Pydantic models are edited in separate chunks/passes. No automated check ensures they stay in sync. Pydantic's default behavior (`model_config` without `extra = "forbid"`) silently ignores extra fields from DB rows, and silently omits fields the model doesn't declare.
**Rule:** Every migration column addition requires a corresponding Pydantic response model field AND a TS contract field update. Consider a build-time test that compares DB schema columns against response model fields for critical tables. Every spec chunk that touches migration DDL must explicitly list the Pydantic model changes in the same chunk.

---

## L060: RLS policies systematically missing on new tables
**Date:** 2026-03-12
**Spec:** #29, #30
**Distilled to:** api-R014
**Reviewed:** 2026-03-12
**What happened:** `jd_optimization_log` and `outreach_optimizations` (Spec 29) had RLS enabled but no service_role policy — every API write failed silently. `cohort_requests` (Spec 30) had owner and service_role policies but no admin SELECT policy — admin dashboard returned empty. Same class of bug as N28-5 in v3.
**Root cause:** RLS is enabled per-table but the required policies (owner, service_role ALL, admin SELECT) are added manually. Easy to forget one, especially service_role on log/audit tables that are only written by backend services.
**Rule:** Every new table with RLS needs three policies: (1) owner access (user_id match), (2) service_role ALL, (3) admin SELECT. Add an RLS checklist to the spec template. When reviewing specs, grep for `ENABLE ROW LEVEL SECURITY` and verify all three policy types exist for each table.

---

## L061: Anonymous endpoint specs describe WHAT but not HOW
**Date:** 2026-03-12
**Spec:** #28
**Distilled to:** api-R015
**Reviewed:** 2026-03-12
**What happened:** Spec 28 v3 added an anonymous JD analysis endpoint (S28-2) but left three critical implementation gaps: (1) `recruiter_id NOT NULL` blocked anonymous storage, (2) no claim pattern for linking anonymous work to new accounts, (3) no rate limiting mechanism specified. Three blockers from one feature addition.
**Root cause:** The v3 fix described the feature at the product level ("anonymous endpoint, 3/hr rate limit") without resolving the schema, auth, and infrastructure implications. Adding "anonymous" to an authenticated system touches NOT NULL constraints, RLS policies, rate limiting infra, and claim/linking flows.
**Rule:** When adding an anonymous/unauthenticated path to an authenticated system, the spec must explicitly address: (1) which NOT NULL FK constraints need to become nullable, (2) how anonymous data links to authenticated accounts on signup (claim pattern), (3) rate limiting mechanism (DB-backed, Redis, or middleware), (4) RLS policy for anonymous rows. Don't just say "rate limited" — specify the implementation.

---

## L063: Enum/Literal types must match DB CHECK constraints bidirectionally
**Date:** 2026-03-12
**Spec:** #23, #27
**Distilled to:** db-R013
**Reviewed:** 2026-03-12
**What happened:** Spec 23 added `'anonymous'` to `agent_state` DB CHECK constraint but never updated the Pydantic `Literal` type on `PlannerInput.agent_state`. Planner rejected anonymous states at the model validation layer — no DB error, just a Pydantic 422. Same class: Spec 27 extended `application_source` CHECK but the Pydantic enum wasn't updated in v1.
**Root cause:** DB schema and Pydantic models are edited in separate passes. Adding a value to one without the other creates silent rejection or acceptance of invalid data.
**Rule:** When adding a value to a CHECK constraint, immediately update the corresponding Pydantic Literal/StrEnum. Verify: `rg "CHECK.*column_name" migrations/` matches `rg "class.*Enum\|Literal" src/soaria/models/`.

---

## L064: IP extraction behind reverse proxies requires X-Forwarded-For
**Date:** 2026-03-12
**Spec:** #23, #28
**Distilled to:** api-R016
**Reviewed:** 2026-03-12
**What happened:** Rate limiting used `request.client.host`. On Railway (behind proxy), this returns the proxy IP, not the user's IP. Every user lands in the same rate limit bucket — either everyone is rate limited or nobody is.
**Root cause:** Assumption that `request.client` gives real client IP. True for direct connections, false behind reverse proxies (Railway, Cloudflare, etc.).
**Rule:** Always extract real IP via `X-Forwarded-For`: `request.headers.get("x-forwarded-for", "").split(",")[0].strip() or request.client.host`. Centralize in a helper function, don't inline.

---

## L065: Metadata/event type values must be constants, not inline strings
**Date:** 2026-03-12
**Spec:** #27
**Distilled to:** mdl-R008
**Reviewed:** 2026-03-12
**What happened:** Chunk 27F defined metadata `"message_type": "follow_up_reminder"`. The planner match logic checked `== 'reminder'`. Different string values. Planner never detected reminders. Feature failed silently — no error, just no trigger.
**Root cause:** Same conceptual value hardcoded as two different strings in two different places. No shared constant.
**Rule:** Values that cross module boundaries (metadata types, event names, status strings) must be defined once as a constant or enum and imported everywhere. Never hardcode matching strings in separate files.

---

## L066: Spec column names must be verified against actual migration files
**Date:** 2026-03-12
**Spec:** #27, #30
**Distilled to:** db-R014
**Reviewed:** 2026-03-12
**What happened:** Spec 27 outcome linking query referenced `resume_optimization_deltas.resume_version_id`. Actual migration column is `to_version_id`. Query returned zero rows silently. Spec 30 referenced `confidence` but migration 009 column is `confidence_score`.
**Root cause:** Spec authors wrote against the PRD data model or from memory instead of reading the actual migration files.
**Rule:** Before shipping any spec that references existing columns, verify with `rg "column_name" migrations/`. PRD column names are aspirational; migration column names are truth.

---

## L067: Sentinel values for "unlimited" tiers, not nullable integers
**Date:** 2026-03-12
**Spec:** #29
**Distilled to:** mdl-R009
**Reviewed:** 2026-03-12
**What happened:** `PLAN_TIER_CONFIG` set enterprise `monthly_limit: None`. Auth middleware compared `int >= None` — TypeError. Column was `INTEGER NOT NULL` — INSERT also failed. Enterprise tier DOA on two levels.
**Root cause:** Python None used to represent "unlimited" in a system that does integer comparison. No type checking caught `int >= None`.
**Rule:** Use sentinel value (999999999) for "unlimited" in quota/limit systems. Avoids nullable columns, None guards, and TypeError. The comparison `usage >= 999999999` just works. Document the sentinel in the constants alongside tier config.

---

## L068: Multi-step writes need explicit atomicity/failure semantics
**Date:** 2026-03-12
**Spec:** #30
**Distilled to:** db-R017
**Reviewed:** 2026-03-12
**What happened:** Cohort matching flow generates anonymized summaries then inserts `cohort_candidates` rows. If anonymization fails mid-batch, some candidates could be inserted with NULL `anonymized_summary`. The public response model assumes non-null — FE would crash or render blank cards.
**Root cause:** Spec didn't specify failure behavior for partial batch operations. Left implicit.
**Rule:** Any multi-step write (generate data + insert rows) needs explicit failure semantics in the spec: (a) atomic transaction (all or nothing), (b) exclude-on-failure (skip broken items, log), or (c) rollback. If the response model assumes non-null, the insert path must guarantee non-null.

---

## L062: Lessons must be written during the work, not after
**Date:** 2026-03-12
**Spec:** N/A (process)
**Distilled to:** N/A (process)
**Reviewed:** 2026-03-12
**What happened:** Applied 20 fixes across 4 specs without writing lessons between each write cycle. Spencer had to prompt for it.
**Root cause:** Treated lesson logging as a post-session task instead of an inline habit during each fix application.
**Rule:** Write lessons after each logical unit of work (per-spec fix pass, per-chunk implementation), not at session end. If a fix reveals a pattern, log the lesson immediately before moving to the next spec.

---

## L069: MagicMock row.get() returns MagicMock, not None
**Date:** 2026-03-12
**Spec:** #20
**Distilled to:** tst-R016
**Reviewed:** 2026-03-12
**What happened:** Added `url=row.get("url")` to VectorSearchResult construction. Existing tests used `MagicMock(**row_dict)` for DB rows, which auto-generates `.get()` as another MagicMock — not dict-like. Pydantic rejected the MagicMock as `str | None`.
**Root cause:** MagicMock doesn't emulate dict `.get()` semantics. Existing code used `row["key"]` (subscript) with an overridden `__getitem__`, but `.get()` bypasses that.
**Rule:** When accessing optional columns from mock DB rows, use `row["key"]` (subscript) not `row.get("key")`. Or add the column with explicit `None` to the mock row defaults. Match the access pattern already used in the test suite.

---

## L070: AsyncMock pool requires proper context manager class
**Date:** 2026-03-12
**Spec:** #20
**Distilled to:** tst-R017
**Reviewed:** 2026-03-12
**What happened:** Tests for save/pass/unsave tools failed with `'coroutine' object does not support the asynchronous context manager protocol`. Used `AsyncMock()` for pool with `__aenter__`/`__aexit__` assigned as attributes.
**Root cause:** `pool.acquire()` returns a context manager object, not a coroutine. Setting `__aenter__` on an AsyncMock's return value doesn't create a proper async CM. Need a real class with `async def __aenter__` / `async def __aexit__`.
**Rule:** Use the `_AcquireCtx` class pattern from `test_tools_search.py` for pool mocks. Never try to make AsyncMock act as an async context manager by assigning `__aenter__`/`__aexit__` — define a proper class.

---

## L071: Python str.format() treats literal braces as format specifiers
**Date:** 2026-03-13
**Spec:** #21
**Distilled to:** tst-R018
**Reviewed:** 2026-03-13
**What happened:** `PREFERENCE_CLASSIFICATION_PROMPT.format(feedback=...)` raised `KeyError: '"id"'`. The prompt contained JSON example `[{"id": "prefer_...", ...}]` with literal curly braces.
**Root cause:** Python's `str.format()` interprets `{` and `}` as format specifiers. Literal braces in prompt templates that use `.format()` must be escaped as `{{` and `}}`.
**Rule:** When writing LLM prompt templates that use `.format()` for variable interpolation, escape ALL literal curly braces as `{{` and `}}`. JSON examples in prompts are the #1 source of this bug. Consider using Template strings or f-strings with explicit variables instead.

---

## L072: Worktree agents write to main repo path via Bash heredocs
**Date:** 2026-03-13
**Spec:** #23
**Distilled to:** CLAUDE.md (Parallel Execution)
**Reviewed:** 2026-03-13
**What happened:** Agent running in a git worktree used `cat > /absolute/main/repo/path/file.py << 'EOF'` heredocs, writing new files to the main repo instead of the worktree. When the worktree was force-removed, untracked files were lost. Had to recover file contents from the agent's JSONL output log.
**Root cause:** Bash heredocs with absolute paths bypass the worktree isolation. The Write/Edit tools correctly use worktree paths, but Bash commands with absolute paths go to whatever path is specified. The agent defaulted to main repo paths in its heredocs.
**Rule:** When dispatching worktree agents, include explicit instruction: "Use relative paths or worktree-prefixed paths for all Bash file creation. Never use absolute paths to the main repo." Better yet: prefer Write tool over Bash heredocs for file creation — it respects worktree isolation automatically.

---

## L073: Agent context grows past safe threshold on 7-chunk specs
**Date:** 2026-03-13
**Spec:** #23
**Distilled to:** CLAUDE.md (Token Discipline)
**Reviewed:** 2026-03-13
**What happened:** Spec 23 agent (7 chunks, 5 waves) reached 157k context tokens and got stuck debugging import path issues for 10+ minutes. Required manual intervention to recover and commit the work. Spec 27 agent (same size) finished at 113k.
**Root cause:** Worktree import path issues (editable install points to main repo, not worktree) caused a debugging spiral that consumed context. Combined with the 7-chunk spec, the agent exceeded safe operating range.
**Rule:** Cap worktree agent dispatches at ~4 chunks or ~100k expected context. For specs with 7+ chunks, split across 2 sequential agents: Agent A handles waves 1-3 and commits, Agent B picks up waves 4-5. This keeps each agent under the compaction danger zone. Also: include `uv pip install -e .` in the worktree setup instructions to fix editable install paths.

---

## L074: Spec SQL migrations can define duplicate columns
**Date:** 2026-03-13
**Spec:** #24
**Distilled to:** db-R015
**Reviewed:** 2026-03-13
**What happened:** Spec 24B migration defined `card_id UUID NOT NULL` in the CREATE TABLE for `card_interactions`, then later had `ALTER TABLE card_interactions ADD COLUMN card_id UUID REFERENCES cards(id)`. This would fail on execution — column already exists.
**Root cause:** The spec was written incrementally (section 2 defined the table, then a later note added the FK relationship) and the duplication wasn't caught during spec review.
**Rule:** Before dispatching agents with migration SQL from specs, scan the migration for duplicate column definitions: any column in CREATE TABLE should not appear in a later ALTER TABLE ADD COLUMN. Include this in the agent prompt as a spec bug to fix.

---

## L075: ALTER TABLE migrations must check existing schema
**Date:** 2026-03-13
**Spec:** #25
**Distilled to:** db-R016
**Reviewed:** 2026-03-13
**What happened:** Spec 25 called for `CREATE TABLE founding_member_codes` but migration 009/010 already created this table with a subset of columns. The agent correctly identified this and wrote ALTER TABLE instead, but only because it happened to check existing migrations.
**Root cause:** Specs are written against the logical schema (what should exist), not the physical schema (what already exists). When a spec says "create table X" but X already exists from a prior migration, the agent must adapt.
**Rule:** Before writing any migration, agents must `ls migrations/*.sql` and grep for the table name in existing migrations. If the table exists, use ALTER TABLE ADD COLUMN IF NOT EXISTS, not CREATE TABLE. Include this check explicitly in agent dispatch prompts.

---

## L076: 4-chunk worktree dispatches stay within safe context bounds
**Date:** 2026-03-13
**Spec:** #24, #25
**Distilled to:** CLAUDE.md (Token Discipline)
**Reviewed:** 2026-03-13
**What happened:** Both Wave 3 Session 1 agents completed 4-chunk dispatches within safe bounds — Spec 24 agent at 92k tokens (13 tests, 8 files), Spec 25 agent at 97k tokens (35 tests, 5 files). Both finished cleanly with no compaction or debugging spirals.
**Root cause:** N/A — this validates L073's rule. 4 chunks is the sweet spot for worktree agent dispatches.
**Rule:** Confirms: cap worktree agents at 4 chunks. Both agents in this session hit ~95k tokens, leaving comfortable headroom below the 120k compaction danger zone. For specs with more chunks, split across sequential sessions.

---

## L077: Lazy imports require patching at source module, not caller
**Date:** 2026-03-13
**Spec:** #28
**Distilled to:** tst-R015
**Reviewed:** 2026-03-13
**What happened:** Tests for `jd_analysis.analyze_jd()` patched `soaria.services.jd_analysis.completion` but `completion` is lazily imported inside the function body (`from soaria.llm.client import completion`). Patch target didn't exist on the module, causing `AttributeError`.
**Root cause:** Lazy imports mean the name isn't on the module's namespace until the function runs. Must patch at the source module (`soaria.llm.client.completion`), not at the import site.
**Rule:** For lazy (in-function) imports, always patch `source_module.func`, not `caller_module.func`. This is the same as L053 but for a different symptom — `AttributeError` instead of "patch doesn't take effect."

---

## L078: SlowAPI limiter must be module-level singleton for decorator use
**Date:** 2026-03-13
**Spec:** #28
**Distilled to:** api-R017
**Reviewed:** 2026-03-13
**What happened:** SlowAPI `@limiter.limit("3/hour")` decorator needs access to the limiter instance at import time. The limiter was created inside `add_middleware()` and only assigned to `app.state.limiter`. Router modules couldn't import it for decorator use.
**Root cause:** `_create_limiter()` was called inside `add_middleware()` with the result only stored on app state. Decorators need module-level access.
**Rule:** When using SlowAPI `@limiter.limit()` decorators in router files, the limiter instance must be a module-level export from middleware.py. Create it at module scope and reuse the same instance in `add_middleware()`.

---

## L079: Spec-defined endpoints must be built when the spec ships, not deferred
**Date:** 2026-03-13
**Spec:** #23
**Distilled to:** api-R018
**Reviewed:** 2026-03-13
**What happened:** Spec 23 defined the paste relay endpoints (POST /v1/paste, GET /v1/paste/{token}) and migration 012 created the paste_tokens table. But the actual router, models, and endpoint code were never written. Both endpoints 404'd in production. The frontend's PasteInput component on the marketing site and the /translate page's paste_token consumption path were completely broken — the entire marketing-to-app cross-domain handoff was dead.
**Root cause:** Migration 012 created the table (with the wrong schema — session_id FK that shouldn't exist, VARCHAR token instead of UUID PK, expires_at instead of fetched_at). The table existing gave a false sense of completeness. No router file was created, no mount was added to router.py, and no tests existed to catch the gap. The spec was marked "shipped" based on the migration and the anonymous session work, but the paste relay endpoints — a distinct deliverable within the same spec — were silently dropped.
**Rule:** Every endpoint defined in a spec must have: (1) a router file, (2) a mount in router.py, (3) acceptance tests that hit the endpoint. Verify with `python3 -c "from soaria.main import create_app; app = create_app(); print([r.path for r in app.routes])"` and confirm every spec-defined path appears. A migration without a router is not a shipped endpoint.

---

## Template for new entries

## L080: RLS admin policies must use user_profiles.id, not user_profiles.user_id
**Date:** 2026-03-13
**Spec:** #17 (migration 017)
**Distilled to:** db-R019
**Reviewed:** 2026-03-13
**What happened:** Migration 017 (recruiter_backend) failed to apply in production. The admin RLS policies referenced `SELECT user_id FROM user_profiles WHERE role = 'admin'` but `user_profiles` has no `user_id` column — the PK `id` is the auth user ID (matches `auth.uid()`).
**Root cause:** Copy-paste from a different table pattern. Other tables (recruiter_profiles, api_subscriptions) have a `user_id` FK column, but `user_profiles.id` IS the user ID directly. The migration passed locally because the Supabase MCP `apply_migration` runs the whole migration transactionally — the failure on the last policy rolled back all table creation, making it appear nothing was created.
**Rule:** In RLS policies that check admin status against `user_profiles`, always use `EXISTS (SELECT 1 FROM user_profiles WHERE id = auth.uid() AND role = 'admin')`. Never reference `user_profiles.user_id` — that column does not exist. The table's PK `id` is the auth user identity.

## L081: TS type definitions must match DDL enums, not aspirational values
**Date:** 2026-03-15
**Spec:** #32
**Distilled to:** spec-R (pending)
**Reviewed:** 2026-03-15
**What happened:** Round 2 foil review (B6) required adding 5 TS interfaces to the FE Contract section. The `LevelSignal.seniority_level` TS union was written with 8 values (`junior | mid | senior | staff | principal | director | vp | c_level`) but the DDL CHECK constraint and Pydantic `SeniorityLevel` enum both define only 4 values (`ic | manager | director | vp`). Caught in build notes before shipping.
**Root cause:** TS types were written from conceptual understanding of seniority levels rather than derived field-for-field from the Pydantic models as the plan specified.
**Rule:** When deriving TS interfaces from Pydantic models, copy enum values exactly from the `StrEnum` definition — never infer or expand them. Cross-check against DDL CHECK constraints when they exist.

## L082: Foil review fixes must cascade to all representations of the same data
**Date:** 2026-03-15
**Spec:** #32
**Distilled to:** spec-R (pending)
**Reviewed:** 2026-03-15
**What happened:** Round 2 blocking fixes (B5, B8) added fields to card models in the Pydantic section but initially missed the inline TS example (lines ~150), the FE Contract bullet descriptions, and the acceptance criteria. 11 build notes caught 10 additional cascade points across env var tables, config sections, test lists, and inline examples.
**Root cause:** Spec has multiple representations of the same data (Pydantic models, TS interfaces, inline TS examples, FE Contract bullets, AC lines, env var table, config.py section). Editing one without scanning for all representations leaves inconsistencies.
**Rule:** After any spec edit, grep for the field/type/config name across the entire spec to find all representations. A spec edit is not complete until every occurrence is consistent.

---

## L083: Worktree agents fail silently on file edits
**Date:** 2026-03-15
**Spec:** #32
**Distilled to:** dispatch-R010
**Reviewed:** 2026-03-15
**What happened:** 32J agent was dispatched to edit `llm/prompts/analysis.py` and `agent/tools/analysis.py` in a worktree. Agent copied the original files without modifications, returning "done" with no error.
**Root cause:** Worktree agents are reliable for *creating new files* but unreliable for *editing existing files*. The agent may read the file, plan edits, but fail to apply them (context pressure, Edit tool errors, or silent fallback to Write with original content). The main window has no way to detect this without verifying diffs.
**Rule:** Reserve worktree agents for new file creation only. All edits to existing files (config.py, router.py, __init__.py, existing tools/prompts) must happen in the main window, where the orchestrator can verify changes directly.

---

## L084: Test agents can't mock implementations they haven't seen
**Date:** 2026-03-15
**Spec:** #32
**Distilled to:** dispatch-R011
**Reviewed:** 2026-03-15
**What happened:** Test agent wrote mocks patching `soaria.services.catalog_corrections.db` (a lazy import alias) instead of `soaria.db.catalog_corrections` (the actual module). 18 tests failed with AttributeError. Also patched `soaria.services.role_resolver.pin_catalog_version` when the function lives in `soaria.services.catalog`.
**Root cause:** Test agent was dispatched with spec-level function signatures but not the actual implementation code. It guessed mock targets based on assumed import patterns. Lazy imports (`from soaria.db import catalog_corrections as db` inside function bodies) are invisible from signatures alone.
**Rule:** Test agents must receive the actual implementation source (or at minimum the import statements) for every module they're mocking. Include the first 30 lines of each implementation file in the test agent prompt.

---

## L085: Single-session full-spec builds beat multi-session when chunks are independent
**Date:** 2026-03-15
**Spec:** #32
**Distilled to:** dispatch-R012
**Reviewed:** 2026-03-15
**What happened:** Spec 32 (9 chunks, 8 waves) was planned as 3 sessions but completed in 1. The 3-session plan existed for token discipline, but worktree agents keep the main window lean (~100k). Parallel dispatch of independent chunks (Rounds 4+5 had 4 agents simultaneous) compressed wall clock to ~30 min.
**Root cause:** Multi-session plans add handoff overhead (orientation, coordination reads, branch management) that's unnecessary when the main window does orchestration only. Context pressure comes from *doing code work* in the main window, not from *dispatching agents*.
**Rule:** When chunk dependency graph allows 3+ parallel dispatches and chunks are file-disjoint, prefer single-session worktree dispatch over multi-session sequential builds. Reserve multi-session for specs where chunks share files or require deep main-window code review.

---

## L086: Integration verification between dispatch rounds prevents compound failures
**Date:** 2026-03-15
**Spec:** #32
**Distilled to:** dispatch-R013
**Reviewed:** 2026-03-15
**What happened:** Between each round of agent dispatches, main window ran import verification (`python -c "from soaria.models.catalog import *"`), router mounting checks, and model field assertions. This caught the 32J agent failure immediately, the missing `python-frontmatter` dep, and the numpy top-level import in bootstrap_catalog.py — all before they could compound.
**Root cause:** Worktree agents operate in isolation. Without verification between rounds, errors propagate forward and test failures become harder to diagnose.
**Rule:** After copying files from each agent round, verify: (1) all new modules import cleanly, (2) router mounts are registered (for API chunks), (3) no duplicate class names in models. Fix issues before dispatching the next round.

---

## L087: Lazy imports inside async generators can't be patched at module level
**Date:** 2026-03-15
**Spec:** #32A
**Distilled to:** tst-R016
**Reviewed:** pending
**What happened:** Test for commentary kill tried `patch("soaria.agent.pipeline.generate_commentary")` and `patch("soaria.agent.pipeline.planner_plan")`. Both failed with `AttributeError` because these are imported inside the `run_agent_turn` function body, not at module level.
**Root cause:** `from soaria.agent.planner import plan as planner_plan` inside a function means the name never exists on the module. Must patch at the source: `soaria.agent.planner.plan` and `soaria.agent.voice.generate_commentary`.
**Rule:** When patching lazily-imported functions, patch the source module (`soaria.agent.planner.plan`) not the consumer module (`soaria.agent.pipeline.planner_plan`). Check import location before writing the patch target.

---

## L088: Memory must be updated at session start and end — not left stale
**Date:** 2026-03-15
**Spec:** N/A (process)
**Distilled to:** session-R001
**Reviewed:** pending
**What happened:** Memory files (spec-status.md, MEMORY.md) still listed Spec 31 as "not built" and infra as "not wired" when both had been shipped in prior sessions. Spencer had to correct stale claims about project state.
**Root cause:** Memory updates were skipped at session end. Next session read stale memory and presented it as fact.
**Rule:** Every session start: read and verify memory against actual state (git log, branch status). Every session end: update memory files to reflect what was built, merged, or changed. Spencer should never have to track state for you.

## L089: Global error handler stringifies HTTPException detail dicts
**Date:** 2026-03-15
**Spec:** #33
**Distilled to:** api-R020
**Reviewed:** pending
**What happened:** `_raise_signup_error` raised `HTTPException(detail={"detail": msg, "error_code": code})`. The global `http_exception_handler` in `errors.py` calls `str(exc.detail)`, converting the dict to a Python repr string. FE received `{"detail": "{'detail': 'Claim token has expired', 'error_code': 'claim_expired'}"}` — unparse-able.
**Root cause:** `errors.py` enforces a flat `ErrorResponse(detail=str, error_code=str)` shape. Passing a dict as `detail` to HTTPException gets stringified.
**Rule:** When an endpoint needs a custom error shape that differs from the global `ErrorResponse` format, return `JSONResponse` directly instead of raising `HTTPException`. This bypasses the global handler. See `api/auth_signup.py` `_signup_error_response()` for the pattern.

## L090: Lazy imports inside functions make patch targets the source module
**Date:** 2026-03-15
**Spec:** #34
**Distilled to:** tst-R019
**Reviewed:** pending
**What happened:** Tests patched `soaria.services.tier2.embed` but `embed` was imported lazily inside `classify_tier_with_context` via `from soaria.llm.client import embed`. The module-level attribute never existed, so `patch()` raised `AttributeError: module does not have the attribute 'embed'`.
**Root cause:** `patch()` requires the target attribute to exist on the module. Lazy imports (`from x import y` inside a function body) only create the name in the function's local scope, not on the module.
**Rule:** For lazy imports, patch at the source: `soaria.llm.client.embed`, not `soaria.services.tier2.embed`. This is a refinement of L053/tst-R015: lazy imports are even more invisible than top-level imports because there's no module attribute to mislead you.

---

## L091: Migrations must be applied to live DB via Supabase MCP, not just committed
**Date:** 2026-03-15
**Spec:** #32, #33
**Distilled to:** session-R002
**Reviewed:** 2026-03-15
**What happened:** Migrations 023 (competency catalog) and 024 (signup flow) were committed and merged to main but never applied to the live Supabase database. Bootstrap flows were triggered against a DB that didn't have the required tables. Spencer had to catch this — Bryce should have applied them as part of the spec build.
**Root cause:** Treated migration files as "done" once committed. The Supabase MCP tool exists specifically for applying migrations to the live DB, and the schema-check skill exists for drift detection. Neither was used.
**Rule:** Every spec that includes a migration file MUST end with: (1) apply the migration via `mcp__supabase__apply_migration`, (2) verify with `mcp__supabase__list_migrations` or `mcp__supabase__list_tables`. Committing a .sql file is not deploying it. The live DB is the only DB that matters.

---

## L092: Data contracts must be updated when populating previously-empty tables
**Date:** 2026-03-15
**Spec:** #32 (cross-role markers)
**Distilled to:** session-R003
**Reviewed:** 2026-03-15
**What happened:** Built cross-role discriminative markers flow that populates the `discriminative_markers` table for the first time. Did not update `types.ts` in the coordination repo. Spencer caught it. The table existed but was always empty — the new flow makes it a live data source for the first time, which means the FE contract needs to know about it.
**Root cause:** Treated "existing table, no schema change" as "no contract change." But a table going from empty to populated is a functional change — downstream consumers need to know the shape of data they'll now receive.
**Rule:** When a flow or service populates a table for the first time (or changes the semantics of what's in it), update `~/soaria-coordination/contracts/types.ts` even if the SQL schema is unchanged. The contract reflects what data is _available_, not just what columns exist.

---

## L093: Bootstrap pipeline labeled JD vocabulary as candidate-side, breaking catalog grounding
**Date:** 2026-03-15
**Spec:** #42 (bootstrap vocab sides)
**Distilled to:** flow-R009
**Reviewed:** 2026-03-15
**What happened:** The catalog grounding prompt builder (`_build_catalog_block`) filters for `side="recruiter"` vocabulary, but the bootstrap pipeline inserted all JD-extracted phrases with `side="candidate"`. This meant the catalog grounding block was always empty — the feature was silently broken since Spec 32 shipped.
**Root cause:** JD competency sections contain recruiter-side vocabulary (the language employers use in job descriptions), but the `sync_to_postgres` task hardcoded `side="candidate"`.
**Rule:** When inserting vocabulary, match the `side` label to the actual source: JD-extracted phrases are `"recruiter"`, resume-extracted phrases are `"candidate"`. Verify that downstream consumers (prompt builders, lookup functions) can actually find the data you're inserting by checking their filter conditions.

---

## L094: Refactoring endpoints to use new service layers breaks existing tests
**Date:** 2026-03-15
**Spec:** #37
**Distilled to:** tst-R020
**Reviewed:** pending
**What happened:** Refactored `merge_webhook` endpoint to delegate to `process_webhook_event()` (Spec 37B). Existing `test_webhook_valid_signature` test broke because the endpoint now calls pool.acquire() via the processor, but the test only mocked signature verification — not the pool or processor. The endpoint returned 500 instead of 200.
**Root cause:** Wrapping an endpoint's logic in a new service layer changes the dependency chain. Existing tests that only mock the old dependencies fail when the endpoint now reaches deeper into the stack.
**Rule:** When refactoring an endpoint to delegate to a new service layer, update ALL existing tests for that endpoint to mock the new service function. Search for the endpoint path in tests/ and verify each test still works with the new call chain.

---

## L095: Handler functions with lazy imports are unpatchable — use module-level imports
**Date:** 2026-03-15
**Spec:** #37
**Distilled to:** svc-R (pending)
**Reviewed:** pending
**What happened:** Webhook handler functions in `webhook_processor.py` used lazy imports (`from soaria.services.subscription import activate_pro` inside the function body). Tests patching `soaria.services.webhook_processor.activate_pro` failed with `AttributeError` because the name only exists in the function's local scope during execution.
**Root cause:** Lazy imports create names in the function's local scope, not on the module. `unittest.mock.patch()` modifies module-level attributes.
**Rule:** Functions that are dispatch targets (handler registries, callback maps) should use module-level imports for their dependencies. This makes them patchable and testable. Lazy imports are fine for rarely-called code paths, but handler functions invoked by a registry need clean mock targets. Reinforces L077/L087/L090.

---

## LXXX: Short description
**Date:** YYYY-MM-DD
**Spec:** #N
**Distilled to:** <rule-ID or pending>
**Reviewed:** YYYY-MM-DD
**What happened:** What broke.
**Root cause:** Why it broke.
**Rule:** The rule that prevents this.
