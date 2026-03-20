# Correction Timeline

Dates extracted from correction logs. Each entry shows when a lesson was recorded.

## Backend Code-Level Corrections (L001-L095)

| Lesson | Date | Spec | Phase |
|---|---|---|---|
| L001 | 2026-02-28 | #1 | Phase 1 |
| L002 | 2026-02-28 | #5 | Phase 1 |
| L003 | 2026-02-28 | #1 | Phase 1 |
| L004 | 2026-02-28 | #8 | Phase 1 |
| L005 | 2026-02-28 | #4, #5, #6, #7, #8 | Phase 1 |
| L006 | 2026-02-28 | #5 | Phase 1 |
| L007 | 2026-03-02 | ## L008: add_middleware() called get_settings() at app construction time | Phase 1 |
| 2026-03-02 | #1 | L009: Missing CRUD endpoints not caught until contract tests | Phase 1 |
| 2026-03-02 | Audit Area 12 | L010: Prefect deployment config needs more than just entrypoints | Phase 3 |
| 2026-03-02 | Audit Area 12 | L011: CLAUDE.md documentation drifts from reality | Phase 3 |
| 2026-03-02 | N/A (meta) | L012: `make check` silently modifies files before linting/testing | Phase 3 |
| 2026-03-02 | N/A | L013: Spec references to deleted modules cause confusion | Phase 3 |
| 2026-03-02 | #5, #6 | L014: Dockerfile CMD replaced by HEALTHCHECK CMD — silent death | Phase 1 |
| 2026-03-02 | N/A (deploy fix) | L015: HS256 JWT secret deprecated — migrate to JWKS/RS256 | Phase 3 |
| 2026-03-02 | #5 (auth) | L016: Sync helper functions called from async without run_sync wrapping | Phase 1 |
| 2026-03-02 | #8 (optimization) | L017: Contract test expects specific export names — aliases required | Phase 1 |
| 2026-03-02 | #7 (applications) | L018: Duplicate Supabase client helpers across API files | Phase 1 |
| 2026-03-02 | N/A (refactor) | L019: Test fixtures must match model constraints exactly | Phase 3 |
| 2026-03-02 | #8A | L020: Test function signatures must match service signatures | Phase 1 |
| 2026-03-02 | #8A | L021: PyJWT HS256 requires 32+ byte secrets | Phase 1 |
| 2026-03-02 | N/A (test infra) | L022: supabase-py result.data has overly broad union type | Phase 3 |
| 2026-03-02 | N/A (type safety) | L023: supabase-py incompatible with new Supabase API keys (sb_secret_...) | Phase 3 |
| 2026-03-02 | N/A (deploy/infra) | L024: Prefect task cache_policy=NONE required for non-serializable inputs | Phase 3 |
| 2026-03-02 | N/A (Prefect flows) | L025: Graduate content when specs are done | Phase 3 |
| 2026-03-02 | N/A (meta/process) | L026: Patch db module functions, not asyncpg conn methods, in service tests | Phase 3 |
| 2026-03-02 | #10C/D/E | L027: Flow callers must be updated when service signatures change | Phase 1 |
| 2026-03-02 | #10C/D/E | L028: Standard pool mock helper for asyncpg tests | Phase 1 |
| 2026-03-02 | #10C/D/E | L029: print() in src/ breaks contract test | Phase 1 |
| 2026-03-02 | #12 | L030: curl not available in python:3.12-slim — use urllib.request | Phase 1 |
| 2026-03-02 | #13 | L031: API test files must be updated when API modules drop supabase imports | Phase 1 |
| 2026-03-03 | #10K | L032: Gut re-export modules when migration completes — don't leave backward-compat shims | Phase 1 |
| 2026-03-03 | #10K | L033: `_mock_pool()` is the standard pattern — use AsyncMock for context manager | Phase 1 |
| 2026-03-03 | #10K | L034: LiteLLM load_dotenv poisons os.environ during test collection | Phase 1 |
| 2026-03-03 | Cross-cutting (test infrastructure) | L035: TestClient re-raises server exceptions by default | Phase 3 |
| 2026-03-03 | #15 | L036: Ruff RUF001 catches ambiguous Unicode characters | Phase 1 |
| 2026-03-03 | #16 | L037: Local migration files don't guarantee live DB state | Phase 1 |
| 2026-03-03 | N/A (schema drift) | L038: Lovable frontend lock-in creates dual-database architecture | Phase 3 |
| 2026-03-04 | #17 | L039: Ruff B905 requires explicit `strict=` on `zip()` | Phase 1 |
| 2026-03-06 | M2 alignment | L040: Circular import when prompt module imports from service module | Phase 3 |
| 2026-03-06 | M2 alignment (C2) | L041: SQL INSERT references non-existent column and violates CHECK constraint | Phase 3 |
| 2026-03-06 | M2 alignment (feed state tracking) | L042: LLM output parsed as UUID without error handling | Phase 3 |
| 2026-03-06 | M2 alignment (optimization) | L043: LLM call inside pool.acquire() exhausts connection pool | Phase 3 |
| 2026-03-06 | Pilot hardening | L044: asyncpg pool has no acquisition timeout by default | Phase 3 |
| 2026-03-06 | Pilot hardening | L045: Unbounded text fields enable LLM token burn and DB bloat | Phase 3 |
| 2026-03-06 | Pilot hardening | L046: Flows using app db_pool singleton instead of independent pools | Phase 3 |
| 2026-03-06 | FIX_07 | L047: Singleton access in dependencies blocks test override | Phase 3 |
| 2026-03-06 | FIX_07 | L048: Error messages leak internal details to API clients | Phase 3 |
| 2026-03-06 | Security audit | L049: LLM outputs must be type-converted defensively | Phase 3 |
| 2026-03-06 | Security audit | L050: Budget configure_budget() used wrong config field for warn_percent | Phase 3 |
| 2026-03-06 | Security audit | L051: LLM timeout fires but response access crashes before handler | Phase 3 |
| 2026-03-06 | Security audit | L052: 10s statement_timeout too aggressive for batch INSERT...ON CONFLICT | Phase 3 |
| 2026-03-06 | Security audit | L053: Pydantic Literal types must match DB CHECK constraints | Phase 3 |
| 2026-03-11 | #23 | L054: Cross-domain storage (sessionStorage/localStorage) doesn't work across origins | Phase 1 |
| 2026-03-11 | Marketing Site | L055: Integration seams need explicit contracts, not implicit assumptions | Phase 3 |
| 2026-03-11 | #23, #24, #25 | L056: Race conditions at status transitions need write-ordering semantics | Phase 1 |
| 2026-03-11 | #25 | L057: Anonymous paths require full audit of every user_id NOT NULL assumption | Phase 1 |
| 2026-03-11 | #23, #24, #25 | L058: Coordination bootstrap replaces manual reads at session start | Phase 1 |
| 2026-03-12 | N/A (infrastructure) | L059: Pydantic response models silently drop new DB columns | Phase 3 |
| 2026-03-12 | #27, #30 | L060: RLS policies systematically missing on new tables | Phase 1 |
| 2026-03-12 | #29, #30 | L061: Anonymous endpoint specs describe WHAT but not HOW | Phase 1 |
| 2026-03-12 | #28 | L063: Enum/Literal types must match DB CHECK constraints bidirectionally | Phase 1 |
| 2026-03-12 | #23, #27 | L064: IP extraction behind reverse proxies requires X-Forwarded-For | Phase 1 |
| 2026-03-12 | #23, #28 | L065: Metadata/event type values must be constants, not inline strings | Phase 1 |
| 2026-03-12 | #27 | L066: Spec column names must be verified against actual migration files | Phase 1 |
| 2026-03-12 | #27, #30 | L067: Sentinel values for "unlimited" tiers, not nullable integers | Phase 1 |
| 2026-03-12 | #29 | L068: Multi-step writes need explicit atomicity/failure semantics | Phase 1 |
| 2026-03-12 | #30 | L062: Lessons must be written during the work, not after | Phase 1 |
| 2026-03-12 | N/A (process) | L069: MagicMock row.get() returns MagicMock, not None | Phase 3 |
| 2026-03-12 | #20 | L070: AsyncMock pool requires proper context manager class | Phase 1 |
| 2026-03-12 | #20 | L071: Python str.format() treats literal braces as format specifiers | Phase 1 |
| 2026-03-13 | #21 | L072: Worktree agents write to main repo path via Bash heredocs | Phase 1 |
| 2026-03-13 | #23 | L073: Agent context grows past safe threshold on 7-chunk specs | Phase 1 |
| 2026-03-13 | #23 | L074: Spec SQL migrations can define duplicate columns | Phase 1 |
| 2026-03-13 | #24 | L075: ALTER TABLE migrations must check existing schema | Phase 1 |
| 2026-03-13 | #25 | L076: 4-chunk worktree dispatches stay within safe context bounds | Phase 1 |
| 2026-03-13 | #24, #25 | L077: Lazy imports require patching at source module, not caller | Phase 1 |
| 2026-03-13 | #28 | L078: SlowAPI limiter must be module-level singleton for decorator use | Phase 1 |
| 2026-03-13 | #28 | L079: Spec-defined endpoints must be built when the spec ships, not deferred | Phase 1 |
| 2026-03-13 | #23 | L080: RLS admin policies must use user_profiles.id, not user_profiles.user_id | Phase 1 |
| 2026-03-13 | #17 (migration 017) | L081: TS type definitions must match DDL enums, not aspirational values | Phase 1 |
| 2026-03-15 | #32 | L082: Foil review fixes must cascade to all representations of the same data | Phase 1 |
| 2026-03-15 | #32 | L083: Worktree agents fail silently on file edits | Phase 1 |
| 2026-03-15 | #32 | L084: Test agents can't mock implementations they haven't seen | Phase 1 |
| 2026-03-15 | #32 | L085: Single-session full-spec builds beat multi-session when chunks are independent | Phase 1 |
| 2026-03-15 | #32 | L086: Integration verification between dispatch rounds prevents compound failures | Phase 1 |
| 2026-03-15 | #32 | L087: Lazy imports inside async generators can't be patched at module level | Phase 1 |
| 2026-03-15 | #32A | L088: Memory must be updated at session start and end — not left stale | Phase 1 |
| 2026-03-15 | N/A (process) | L089: Global error handler stringifies HTTPException detail dicts | Phase 3 |
| 2026-03-15 | #33 | L090: Lazy imports inside functions make patch targets the source module | Phase 1 |
| 2026-03-15 | #34 | L091: Migrations must be applied to live DB via Supabase MCP, not just committed | Phase 1 |
| 2026-03-15 | #32, #33 | L092: Data contracts must be updated when populating previously-empty tables | Phase 1 |
| 2026-03-15 | #32 (cross-role markers) | L093: Bootstrap pipeline labeled JD vocabulary as candidate-side, breaking catalog grounding | Phase 1 |
| 2026-03-15 | #42 (bootstrap vocab sides) | L094: Refactoring endpoints to use new service layers breaks existing tests | Phase 1 |
| 2026-03-15 | #37 | L095: Handler functions with lazy imports are unpatchable — use module-level imports | Phase 1 |
| 2026-03-15 | #37 | LXXX: Short description | Phase 1 |
| YYYY-MM-DD | #N |  | Phase 1 |

## Phase Summary

| Phase | Period | Lessons | Specs Covered | Rate |
|---|---|---|---|---|
| Phase 1 | Feb 28 - Mar 2 | 28 | ~5 (Specs 1-10) | ~5.6/spec |
| Phase 2 | Mar 3 - Mar 6 | 24 | ~8 (Specs 10-16) | ~3.0/spec |
| Phase 3 | Mar 11 - Mar 15 | 43 | ~15 (Specs 17-42) | ~2.9/spec |

## Other Domains

Strategic corrections across other domains are dated in their respective `corrections/*/lessons.md` files.
Frontend-faye code-level corrections (L001-L044) span 2026-03-10 to 2026-03-15.
Frontend-frannie corrections (L001-L006) span 2026-03-05 to 2026-03-06.
