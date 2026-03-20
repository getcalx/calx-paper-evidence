# AGENTS.md — Service Layer Conventions

Rules for working in `src/soaria/services/`. Distilled from Specs 4–10, Lessons
L005 L016 L017 L022 L026 L027 L028, and the cutover readiness session.

Rule IDs: svc-R001 through svc-R013 | Last reviewed: 2026-03-15

### svc-R001: Async vs Sync Services
Source: L005, Spec #10 | Added: 2026-03-02 | Status: active

**Migrated to asyncpg (Spec 10):** `users.py`, `resumes.py`, `case_studies.py`,
`embeddings.py`. These are `async def` and take `DatabasePool` as their first
argument. SQL lives in `db/<domain>.py` repository modules. No `run_sync()`
needed — asyncpg is natively async.

**Still on supabase-py:** `admin.py`, `applications.py`, `feed.py`,
`interactions.py`, `optimization.py`. These remain sync. Callers in `async def`
endpoints wrap them in `await run_sync()` or `await asyncio.to_thread()`. (L005)

When migrating a remaining service to asyncpg:
1. Change the function to `async def`.
2. Replace the `supabase: Client` parameter with `pool: DatabasePool`.
3. Move raw SQL to a new `db/<domain>.py` repository module.
4. Grep for ALL callers including flows and update them (L027).

### svc-R002: Sync Helpers from Async
Source: L016, Spec #8 | Added: 2026-03-02 | Status: active

This section applies ONLY to the non-migrated supabase-py services (`admin.py`,
`applications.py`, `feed.py`, `interactions.py`, `optimization.py`).

When an `async def` function calls a sync helper that does `.execute()`, the call
to that helper must be wrapped in `await run_sync(helper, ...)`. (L016)

The audit check (`rg ".execute()"`) catches direct calls but misses calls nested
inside sync helpers. Check both.

For asyncpg-migrated services, this does not apply — they are natively async end
to end.

### svc-R003: Service Role Restrictions
Source: L006, Spec #5 | Added: 2026-03-02 | Status: active

`use_service_role=True` is ONLY for:
- Prefect flows (run in their own process)
- Auto-profile creation (user has no profile yet)
- Admin operations

For supabase-py services: user-facing functions receive the user-scoped Supabase
client from the API layer. For asyncpg services: functions receive `DatabasePool`
from the API layer via `get_db_pool()`. Never create a service-role client inside
a user-facing service function.

### svc-R004: Export Names Match Contract Tests
Source: L017 | Added: 2026-03-02 | Status: active

Contract tests import specific names from service modules. Before implementing,
check what the tests expect: (L017)

```bash
rg "from soaria.services.<module> import" tests/test_contract_integration.py
```

Use the exact names the tests expect. If you prefer shorter names internally,
create the test-expected name first and alias.

### svc-R005: Supabase Result Typing
Source: L022 | Added: 2026-03-02 | Status: active

This applies to non-migrated services that still use supabase-py.

`result.data` is typed as `list[JSON]` (a recursive union type). Use a `_rows()`
helper to type-narrow once: (L022)

```python
def _rows(data: Any) -> list[dict[str, Any]]:
    return data if data else []
```

For `count="exact"`, add `# type: ignore[arg-type]` since the stubs expect
`CountMethod | None` but the runtime accepts strings.

### svc-R006: asyncpg Result Typing
Source: Spec #10 | Added: 2026-03-03 | Status: active

For asyncpg-migrated services, query results are `asyncpg.Record` objects.
Convert to dict with `dict(row)`. The `db/<domain>.py` repository functions
typically handle this conversion and return plain dicts or domain objects.

### svc-R007: Upsert Semantics
Source: Spec #5 | Added: 2026-03-02 | Status: active

For auto-create paths (like profile creation on first login), use INSERT with
`ON CONFLICT DO NOTHING`. Never overwrite admin-modified data.

### svc-R008: Testing asyncpg Services
Source: L026, L027, L028, Spec #10 | Added: 2026-03-03 | Status: active

**Patch at the db module level, not the conn level.** (L026) For service-layer
unit tests, mock the repository functions in `db/<domain>.py`:

```python
@patch("soaria.db.resumes.get_resume_template", new_callable=AsyncMock, return_value=row)
async def test_get_template(self, mock_get):
    result = await get_resume_template(pool, template_id="abc", user_id="u1")
```

Do NOT mock `conn.fetchrow` / `conn.fetch` for service tests — that couples
tests to SQL implementation details and requires fragile `side_effect` chains
for multi-query services.

**Standard pool mock pattern** (L028):

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

**Grep all callers when changing signatures.** (L027) When migrating a service
function, run `rg "function_name" src/soaria/` to find every caller — including
flows, which live in a different directory and are easy to miss.

### svc-R009: Batch Operations
Source: Spec #10 | Added: 2026-03-03 | Status: active

For multi-row inserts, use a single INSERT with UNNEST or multi-row VALUES.
Never loop with individual INSERTs — that's N round-trips instead of 1.

### svc-R010: Never Hold DB Connection During LLM Calls
Source: L043 | Added: 2026-03-06 | Status: active

Never hold a DB connection during an LLM call, HTTP call, or any operation >100ms.
Split into: acquire->read->release -> long operation -> acquire->write->release.
Verify: `rg "await completion|await embed" src/soaria/` and check that none appear
inside a `pool.acquire()` block. (L043)

### svc-R011: Gut Compatibility Shims After Migration
Source: L032 | Added: 2026-03-03 | Status: active

When a migration completes, immediately remove backward-compat re-exports. Search
all imports from the old module (`rg "from soaria.<old> import" src/ tests/`) and
update to the canonical path. The old module should contain only symbols that
genuinely belong there. (L032)

### svc-R012: Enhanced Classification Wraps Original, Never Replaces
Source: Spec 34 | Added: 2026-03-15 | Status: active

When adding an enhanced version of an existing classifier (e.g., `classify_tier_with_context`
wrapping `classify_tier`), the enhanced version MUST reproduce all original checks
(Tier 1 gates) and only add new logic (Tier 2 detection) after them. The call site
should use the enhanced version when context is available (catalog_id) and fall back
to the original otherwise. See `analysis.py:_analyze_vocabulary` Tier classification
block and `services/tier2.py` for the pattern.

### svc-R013: Handler Registry Functions Use Module-Level Imports
Source: L095, Spec #37 | Added: 2026-03-15 | Status: active

Functions that serve as dispatch targets (handler registries, callback maps) MUST use
module-level imports for their dependencies. Lazy imports inside handler functions are
invisible to `unittest.mock.patch()` — the name never exists on the module namespace.

Pattern (`webhook_processor.py`):
```python
# Module level — patchable as soaria.services.webhook_processor.activate_pro
from soaria.services.subscription import activate_pro

async def handle_checkout(pool, payload):
    await activate_pro(pool, ...)
```

Anti-pattern:
```python
async def handle_checkout(pool, payload):
    from soaria.services.subscription import activate_pro  # NOT patchable on this module
    await activate_pro(pool, ...)
```
