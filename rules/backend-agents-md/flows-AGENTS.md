# AGENTS.md — Prefect Flow Conventions

Rules for working in `src/soaria/flows/`. Distilled from Specs 2–4, Lessons L010
L024, and FIX_06.

Rule IDs: flow-R001 through flow-R009 | Last reviewed: 2026-03-15

### flow-R001: Use Flow Template
Source: Spec #2 | Added: 2026-03-02 | Status: active

Copy `_template.py` as the starting point for every new flow. It has the standard
structure: flow function orchestrates tasks, tasks contain business logic.

### flow-R002: Service Role Permitted
Source: Spec #4 | Added: 2026-03-02 | Status: active

Flows run in their own process (Prefect worker). They use `use_service_role=True`
for Supabase access. This is one of the three approved service-role use cases.

### flow-R003: cache_policy=NONE for Non-Serializable Inputs
Source: L024 | Added: 2026-03-02 | Status: active

Any `@task` that receives non-serializable objects (DB clients, connection pools,
HTTP clients, semaphores) MUST use `cache_policy=NONE`. (L024)

Prefect 3.x tries to hash all task inputs for caching by default. Clients and
pools contain thread locks that can't be serialized.

```python
from prefect.cache_policies import NONE

@task(cache_policy=NONE)
def my_task(supabase: Client, ...):
    ...
```

### flow-R004: Concurrency Limit
Source: Spec #2 | Added: 2026-03-02 | Status: active

All flows use `concurrency_limit=1` to prevent parallel runs of the same flow.

### flow-R005: Deployment Config Completeness
Source: L010 | Added: 2026-03-02 | Status: active

`prefect.yaml` at repo root defines all deployments. Every parameter from the
`@flow()` decorator (timeout_seconds, retries) must also appear in the YAML.
The YAML overrides decorator values in Prefect Cloud. (L010)

Include pull steps for Railway containers: `set_working_directory: /app`.

### flow-R006: Error Isolation Per Item
Source: Spec #2 | Added: 2026-03-02 | Status: active

Single-item failures (one board, one job, one resume) must not crash the entire
flow run. Catch exceptions per-item and log them. Report counts at flow end.

## Current Flows

| Flow | Schedule | Timeout |
|------|----------|---------|
| `greenhouse-sync` | Every 5 min | 300s |
| `embed-jobs-and-resumes` | Every 5 min | 300s |
| `rank-new-jobs` | Every 15 min | 900s |

### flow-R007: Structured Logging
Source: Spec #2 | Added: 2026-03-02 | Status: active

Log flow start, completion, and key metrics (items processed, duration) using
structlog. Prefect's built-in logging supplements but doesn't replace structured
application logs.

### flow-R008: Independent Pool per Flow
Source: L046 | Added: 2026-03-06 | Status: active

Flows must create their own `DatabasePool` via `DatabasePool.from_dsn()` with
try/finally for cleanup. Never import `db_pool` singleton in flows. Verify:
`grep "db_pool" src/soaria/flows/` returns zero hits. (L046)

### flow-R009: Match Vocabulary Side to Source
Source: L093 | Added: 2026-03-15 | Status: active

When inserting vocabulary entries, the `side` label must match the actual source:
JD-extracted phrases are `"recruiter"`, resume-extracted phrases are `"candidate"`.
Verify downstream consumers (prompt builders, lookup functions) can find the data
by checking their filter conditions against the side you're inserting. (L093)
