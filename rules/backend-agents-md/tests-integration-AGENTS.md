# tests/integration/ — Integration Test Conventions

Rule IDs: tst-R001 through tst-R020 | Last reviewed: 2026-03-15

## What Lives Here

Integration tests that exercise real `service -> repository -> DB` paths against
a local Supabase PostgreSQL instance. These tests hit actual tables, constraints,
triggers, and pgvector operations -- no mocks.

### tst-R001: Scope Boundaries
Source: Spec #10 | Added: 2026-03-02 | Status: active

- **Unit tests** -- those live in `tests/` (top-level), use mocks for DB/HTTP.
- **Contract tests** -- `tests/test_contract_integration.py` validates spec compliance
  via TestClient against the FastAPI app. That is a separate concern.
- **Business logic tests** -- test complex logic in unit tests with mocked dependencies.

## Prerequisites

1. Local Supabase running: `supabase start`
2. Connection available at `postgresql://postgres:postgres@127.0.0.1:54322/postgres`
3. Schema must be up to date (run any pending migrations in Supabase SQL Editor first).

## Running

```bash
make test-integration              # runs with --integration flag
make test-integration-cov          # with coverage report
pytest tests/integration/ --integration -v   # direct invocation
```

Tests are skipped by default unless `--integration` is passed. The `conftest.py`
registers the CLI option and applies a skip marker to all `integration`-marked tests.

## Test Patterns

### tst-R002: Markers and Async
Source: Spec #10 | Added: 2026-03-02 | Status: active

Every test file has a module-level marker and every test is async:

```python
pytestmark = pytest.mark.integration

@pytest.mark.asyncio
async def test_something(int_db_pool: asyncpg.Pool) -> None:
    ...
```

### tst-R003: Connection Acquisition
Source: Spec #10 | Added: 2026-03-02 | Status: active

Use the `int_db_pool: asyncpg.Pool` session-scoped fixture. Acquire connections
inside each test:

```python
async with int_db_pool.acquire() as conn:
    result = await some_db_function(conn, ...)
```

### tst-R004: Import Convention
Source: Spec #10 | Added: 2026-03-02 | Status: active

Import db functions directly from `soaria.db.<module>` (not from services, unless
the test specifically exercises the service layer). Some older tests import from
`soaria.services.*` -- this is acceptable when the service function takes a pool
or connection directly.

### tst-R005: Test Isolation
Source: Spec #10 | Added: 2026-03-02 | Status: active

The `clean_tables` autouse fixture runs `TRUNCATE ... CASCADE` on all test-relevant
tables before each test. Order matters due to FK constraints. If you add a new table
that tests touch, add it to the TRUNCATE list in `conftest.py`.

### tst-R006: Naming Convention
Source: Spec #10 | Added: 2026-03-02 | Status: active

- Files: `test_<domain>_int.py` (the `_int` suffix distinguishes from unit tests)
- Tests: `test_<operation>` or `test_<operation>_<scenario>`
- Examples: `test_create_profile`, `test_create_profile_upsert`,
  `test_vector_search_excludes_inactive`

## Factory Helpers

Located in `conftest.py`. Each factory accepts `conn: asyncpg.Connection` plus
`**overrides` for any column, and returns `dict[str, Any]`:

| Factory | Table | Required args | Notes |
|---------|-------|---------------|-------|
| `user_factory` | `user_profiles` | `conn` | Generates unique email via uuid |
| `company_factory` | `companies` | `conn` | Generates unique domain via uuid |
| `job_factory` | `jobs` | `conn`, `company_id` | Requires existing company FK |
| `template_factory` | `resume_templates` | `conn`, `user_id` | Requires existing user FK, serializes `template_content` to JSON |

Import factories from conftest: `from tests.integration.conftest import user_factory`
or `from .conftest import user_factory`.

### tst-R007: pgvector Codec Registration
Source: Spec #10 | Added: 2026-03-02 | Status: active

Register the codec before any vector read/write operations:

```python
from tests.integration.conftest import _register_vector_codec

async with int_db_pool.acquire() as conn:
    await _register_vector_codec(conn)
    # Now vector columns can be read/written as Python lists
```

The production code equivalent is `register_vector_codec` from
`soaria.services.embeddings`. Either works in tests.

## Seed Fixtures

For tests that need a full data graph (user + resume + jobs + embeddings), use
the seed fixtures defined in `conftest.py`:

- `seed_company` -- inserts a companies row
- `seed_greenhouse_board` -- inserts a greenhouse_companies row (depends on `seed_company`)
- `seed_user_with_resume` -- inserts user_profiles + resume_templates with a 1024-dim fake embedding
- `seed_jobs_with_embeddings` -- inserts 10 jobs with embeddings (5 similar, 5 dissimilar)

These are request-scoped and depend on `clean_tables` for isolation.

### tst-R008: New Test Checklist
Source: Spec #10 | Added: 2026-03-02 | Status: active

1. Create `test_<domain>_int.py` with `pytestmark = pytest.mark.integration`.
2. Import the db functions you are testing from `soaria.db.<module>`.
3. Use `int_db_pool` fixture and acquire connections.
4. Use factory helpers for test data setup.
5. If the domain introduces new tables, add them to `clean_tables` TRUNCATE list.
6. Run: `make test-integration` to verify.

### tst-R009: Test Signatures Must Match Service Signatures
Source: L020 | Added: 2026-03-02 | Status: active

After any service function signature change, grep for all callers including tests:
`rg "function_name" tests/`. Stale test calls will fail with TypeError at runtime
but not at import time. (L020)

### tst-R010: JWT Test Secret Requirements
Source: L021 | Added: 2026-03-02 | Status: active

Test JWT secrets must be >= 32 bytes per RFC 7518. Current value:
`test-jwt-secret-for-unit-tests-XX`. Update both `_helpers.py` and `conftest.py`
together when changing. (L021)

### tst-R011: Update Test Patches When Removing Module Imports
Source: L031 | Added: 2026-03-03 | Status: active

When removing an import from a module, search for test patches targeting that
import: `rg "patch.*soaria.api.<module>.<removed_name>" tests/`. Migration chunks
must include test updates for all layers (db, service, AND api). (L031)

### tst-R012: Standard Pool Mock Pattern
Source: L033, L028 | Added: 2026-03-03 | Status: active

Use the standard `_mock_pool()` pattern for asyncpg pool mocks. See svc-R008 for
the full pattern. (L033)

### tst-R013: LiteLLM load_dotenv Poisoning
Source: L034 | Added: 2026-03-03 | Status: active

Never use `os.environ.setdefault()` in test fixtures -- LiteLLM calls
`load_dotenv()` at import time, poisoning `os.environ`. Always use direct
assignment (`os.environ[key] = val`) or `monkeypatch.setenv()`. (L034)

### tst-R014: TestClient Server Exception Behavior
Source: L035 | Added: 2026-03-03 | Status: active

FastAPI's TestClient has `raise_server_exceptions=True` by default. Use
`TestClient(app, raise_server_exceptions=False)` or wrap in try/except when
testing routes that may hit infrastructure errors. (L035)

### tst-R015: Lazy Imports Require Patching at Source Module
Source: L077 | Added: 2026-03-14 | Status: active

For lazy (in-function) imports like `from soaria.llm.client import completion`,
always patch `soaria.llm.client.completion`, not `soaria.services.module.completion`.
The name isn't on the caller module's namespace until the function runs, so patching
the caller raises `AttributeError`. Patch the source module where the symbol is
defined. (L077)

### tst-R016: Mock DB Rows — Use Subscript Access, Not .get()
Source: L069 | Added: 2026-03-14 | Status: active

When accessing optional columns from `MagicMock` DB rows, use `row["key"]`
(subscript) not `row.get("key")`. MagicMock auto-generates `.get()` as another
MagicMock — not dict-like behavior. Either use subscript access or add the column
with explicit `None` in the mock row defaults. (L069)

### tst-R017: Async Pool Mocks Need Real Context Manager Class
Source: L070 | Added: 2026-03-14 | Status: active

Use a proper `_AcquireCtx` class with `async def __aenter__` / `async def __aexit__`
for pool mocks. Never try to make `AsyncMock` act as an async context manager by
assigning `__aenter__`/`__aexit__` attributes — that produces `'coroutine' object
does not support the asynchronous context manager protocol`. See
`test_tools_search.py` for the reference pattern. (L070)

### tst-R018: Escape Literal Braces in .format() Prompt Templates
Source: L071 | Added: 2026-03-14 | Status: active

When testing LLM prompt templates that use `.format()` for variable interpolation,
verify that literal curly braces (especially JSON examples) are escaped as `{{` and
`}}`. Unescaped braces raise `KeyError` at runtime. JSON examples in prompts are
the #1 source of this bug. (L071)

### tst-R019: Lazy Imports — Patch at Source Module, Not Caller
Source: L090 | Added: 2026-03-15 | Status: active

When a function uses a lazy import (`from soaria.llm.client import embed` inside the
function body), the imported name only exists in the function's local scope, NOT as a
module attribute. `patch("soaria.services.tier2.embed")` will raise `AttributeError`.
Patch at the source: `patch("soaria.llm.client.embed")`. This refines tst-R015 (L053):
lazy imports are even more invisible than top-level imports because there's no module
attribute to mislead you. (L090)

### tst-R020: Refactored Endpoints Need Updated Test Mocks
Source: L094 | Added: 2026-03-15 | Status: active

When refactoring an endpoint to delegate to a new service layer (e.g., wrapping
`stripe_webhook` logic in `process_webhook_event()`), ALL existing tests for that
endpoint must be updated to mock the new dependency. The old test may have only
mocked the previous call chain — the new service function introduces pool.acquire()
or other operations that weren't in the original test scope. Search: `rg "endpoint_path"
tests/` to find every test that hits the refactored endpoint. (L094)
