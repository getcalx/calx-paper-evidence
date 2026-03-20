# db/ — Database Access Layer

## What Lives Here

Repository modules with typed async functions that wrap raw SQL queries via asyncpg.
Each domain gets its own module (e.g., `users.py`, `jobs.py`, `applications.py`).

Rule IDs: db-R001 through db-R020 | Last reviewed: 2026-03-15

## Conventions

### db-R001: Function Signatures
Source: Spec #10 | Added: 2026-03-02 | Status: active

- First argument is always `asyncpg.Connection` (not the pool).
- Callers acquire the connection and pass it in — this allows transaction grouping.
- All functions are `async def` with full type hints on params and return.

### db-R002: SQL Style
Source: Spec #10 | Added: 2026-03-02 | Status: active

- Use `$1, $2, ...` positional parameters (asyncpg native).
- Multi-line SQL in triple-quoted strings, indented at function level.
- Parameterized queries only — never f-string interpolation of user data.
- Use `RETURNING *` (or specific columns) for INSERT/UPDATE ops.

### db-R003: Naming Conventions
Source: Spec #10 | Added: 2026-03-02 | Status: active

- `get_<thing>` — single row fetch, returns `dict | None`
- `list_<things>` — multi-row fetch, returns `list[dict]`
- `create_<thing>` — INSERT, returns inserted row dict
- `update_<thing>` — UPDATE, returns updated row dict
- `delete_<thing>` — DELETE, returns count or None
- `upsert_<thing>` — INSERT ... ON CONFLICT, returns row dict

### Helpers
- `_helpers.py` has `fetch_one`, `fetch_all`, `paginate`, `search_filter`.
- Use these for common patterns; write raw SQL for complex queries (JOINs, CTEs).

### db-R004: Pool Management
Source: Spec #10 | Added: 2026-03-02 | Status: active

- `pool.py` has `DatabasePool` (app singleton) and `create_flow_pool()` (for Prefect flows).
- API endpoints get the pool via `get_db_pool()` dependency from `api/deps.py`.
- Flows create their own pool with `create_flow_pool()` and close it in a finally block.

### db-R005: Layer Boundaries
Source: Spec #10 | Added: 2026-03-02 | Status: active

- Business logic (stays in `services/`).
- Pydantic model definitions (stay in `models/`).
- HTTP/API concerns (stay in `api/`).
- supabase-py calls — this package replaces supabase-py for all DB access.

## Schema Migrations

### db-R006: Migration Runner
Source: Spec #12 | Added: 2026-03-02 | Status: active

- Forward-only SQL migration system. No down migrations.
- `run_migrations(pool, dir)` discovers `*.sql` files by numeric prefix, applies pending ones in order.
- Each migration runs in its own transaction — partial apply is recoverable.
- Tracking table: `schema_migrations (version TEXT PK, applied_at TIMESTAMPTZ, checksum TEXT)`.
- Checksum mismatch logs a warning but does NOT block execution.
- `migrate.py` is standalone tooling — does NOT use `_helpers.py`.

### db-R007: Migration CLI
Source: L029, Spec #12 | Added: 2026-03-02 | Status: active

- `python -m soaria.db apply` — apply pending migrations.
- `python -m soaria.db status` — show migration status table.
- Settings loaded lazily inside `main()` — never at import time.
- Uses raw `asyncpg.create_pool()`, not `DatabasePool` (standalone tooling).
- No `print()` — uses `sys.stdout.write()` (contract test enforced).

### db-R008: Migration File Standards
Source: Spec #12 | Added: 2026-03-02 | Status: active

- Files: `NNN_descriptive_name.sql` (e.g., `001_initial_schema.sql`).
- All DDL must be idempotent: `IF NOT EXISTS`, `CREATE OR REPLACE`, `DO $$ IF NOT EXISTS`.
- RLS policies wrapped in `DO $$ BEGIN IF NOT EXISTS (SELECT 1 FROM pg_policies ...) THEN ... END IF; END $$;`.
- Seed data uses `ON CONFLICT DO NOTHING`.
- Never modify an applied migration — create a new one instead.

### Makefile
- `make migrate` — apply pending migrations.
- `make migrate-status` — show current state.

### db-R009: supabase-py Migration Checklist
Source: Spec #10 | Added: 2026-03-03 | Status: active

When converting a service from supabase-py to asyncpg:
1. Create the corresponding `db/<domain>.py` with typed async functions.
2. Update the service to accept `asyncpg.Connection` instead of `supabase.Client`.
3. Remove `run_sync()` wrappers — asyncpg is natively async.
4. Remove `# type: ignore` comments that were needed for supabase-py.
5. Update API endpoints to pass `pool` (from `get_db_pool()`) instead of supabase client.
6. Verify: `rg "supabase" src/soaria/services/<file>.py` returns nothing.

### db-R010: Pool Acquisition Timeout Required
Source: L044 | Added: 2026-03-06 | Status: active

Always pass `timeout=5.0` to `asyncpg.create_pool()`. Both the app pool
(`DatabasePool.connect()`) and flow pools (`create_flow_pool()`). Without this,
pool exhaustion causes infinite hangs instead of errors. (L044)

### db-R011: Statement Timeout Sizing
Source: L052 | Added: 2026-03-06 | Status: active

Connection-level `statement_timeout` is 30s (current baseline). Profile the
slowest legitimate query (batch upserts) before changing. Individual queries
can override with `SET LOCAL statement_timeout` inside transactions. (L052)

### db-R012: Verify Columns and Constraints Before Writing SQL
Source: L041 | Added: 2026-03-06 | Status: active

Before writing INSERT/UPDATE, verify column names and CHECK constraints against
the migration file: `rg "CREATE TABLE.*<table>" migrations/`. For CHECK
constraints: `rg "CHECK.*<column>" migrations/`. Never assume a column or enum
value exists. (L041)

### db-R013: Pydantic Literal/Enum Types Must Mirror DB CHECK Constraints
Source: L053, L063 | Added: 2026-03-14 | Status: active

When adding a value to a DB CHECK constraint, immediately update the corresponding
Pydantic `Literal` or `StrEnum` type, and vice versa. These must stay in sync
bidirectionally — a CHECK value without a model value causes silent DB rejection;
a model value without a CHECK causes Pydantic 422 before the query runs.

Verify: `rg "CHECK.*column_name" migrations/` values match
`rg "Literal\[.*column_name" src/soaria/models/` values. (L053, L063)

### db-R014: Verify Column Names Against Migration Files, Not Specs
Source: L066 | Added: 2026-03-14 | Status: active

Before writing SQL that references existing columns, verify names against the actual
migration files: `rg "column_name" migrations/`. PRD and spec column names are
aspirational; migration DDL is the source of truth. Spec 27 referenced
`resume_version_id` but the actual column was `to_version_id`. (L066)

### db-R015: Scan Spec Migrations for Duplicate Column Definitions
Source: L074 | Added: 2026-03-14 | Status: active

Before dispatching agents with migration SQL from specs, check for columns defined
in both CREATE TABLE and a later ALTER TABLE ADD COLUMN in the same migration. The
ALTER will fail because the column already exists. Fix in the spec before building.
(L074)

### db-R016: ALTER TABLE Migrations Must Check Existing Schema
Source: L075 | Added: 2026-03-14 | Status: active

Before writing any migration, check existing migrations: `ls migrations/*.sql` and
`rg "table_name" migrations/`. If the table already exists from a prior migration,
use `ALTER TABLE ADD COLUMN IF NOT EXISTS`, not `CREATE TABLE`. Specs describe the
logical schema; migrations reflect what physically exists. (L075)

### db-R017: Multi-Step Writes Need Explicit Atomicity Semantics
Source: L068 | Added: 2026-03-14 | Status: active

Any multi-step write (generate data then insert rows) needs explicit failure
semantics in the spec: (a) atomic transaction — all or nothing, (b) exclude-on-failure
— skip broken items and log, or (c) rollback. If the response model assumes non-null
fields, the insert path must guarantee non-null. Never leave partial-failure behavior
implicit. (L068)

### db-R018: Concurrent Status Writers Need WHERE Guards
Source: L056 | Added: 2026-03-14 | Status: active

When two systems can write to the same status field concurrently (e.g., trial-end
flow and Stripe webhook), define which one wins. Use WHERE guards
(`AND stripe_subscription_id IS NULL`) so the lower-priority write becomes a no-op
when the higher-priority write has already occurred. Document the write-ordering
semantics explicitly. (L056)

### db-R019: RLS Admin Policies Use user_profiles.id, Not user_profiles.user_id
Source: L080 | Added: 2026-03-14 | Status: active

In RLS policies that check admin status, always use:
`EXISTS (SELECT 1 FROM user_profiles WHERE id = auth.uid() AND role = 'admin')`.
Never reference `user_profiles.user_id` — that column does not exist. The table's
PK `id` IS the auth user identity directly. Other tables (recruiter_profiles,
api_subscriptions) have a `user_id` FK, but `user_profiles` does not. (L080)

### db-R020: Webhook DLQ and Idempotency Tables (Spec 37)
Source: Spec #37 | Added: 2026-03-15 | Status: active

Two new modules in `db/`:
- **`webhooks.py`** — `check_idempotency`, `mark_processed`, `insert_failure`,
  `list_failures`, `get_failure`, `update_failure_status`, `generate_event_id`.
- **`analytics.py`** — `query_funnel`, `query_engagement`, `query_pipeline` (Spec 38).
  Pure SQL reads against existing tables, no new migrations.

`webhooks.py` follows a mixed pattern: `check_idempotency` and `mark_processed` take
`conn` (for transactional use with the handler), while DLQ functions (`insert_failure`,
`list_failures`, `get_failure`) take `pool` (independent operations). The `insert_failure`
function has a structlog fallback — if the DLQ insert itself fails, it logs ERROR rather
than crashing.
