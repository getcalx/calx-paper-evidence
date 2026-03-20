# Spec 43: Role Title Normalization — Review Panel

**Date:** 2026-03-16
**Reviewers:** Nolan (backend foil), Sloane (product/GTM), Reviewer-BH6 (strategy), Reviewer-SY4 (ML), Reviewer-SY5 (ML systems)
**Status:** REVISE — blockers identified, solutions provided

---

## Changelog

### R1 (2026-03-16)
- 5 reviewers dispatched in parallel
- Solutions round dispatched after initial findings
- Key decision: Part A ships before March 23, Part B decoupled to post-launch
- Representation mismatch identified as primary ML blocker
- Data patch (alias mining) recommended as Step 0 before any code

---

## Step 0: Data Patch Before Code (Sloane, Reviewer-BH6)

Mine `role_alias_misses` for top 50 failed inputs. Classify and add obvious ones as aliases via direct SQL or existing admin endpoint. Zero code, zero risk, fixes ~60-70% of failures immediately. **2-3 hours.**

---

## Part A Blockers + Solutions

### B1: Wrong Comparison Target (Reviewer-SY4)

**Problem:** Part A compares title embeddings against cluster embeddings (competency labels like "Stakeholder Management"). Titles and cluster labels live in different semantic neighborhoods — cosine similarity between them is near-meaningless.

**Solution:** Add `role_title_centroid VECTOR(1024)` column to `competency_catalogs`. Embed each canonical role title + its aliases via Voyage, store the mean as centroid. 13 embed calls, one ALTER TABLE, one backfill script. Part A then compares user input against these centroids — same semantic space, title-to-title.

```sql
ALTER TABLE competency_catalogs ADD COLUMN role_title_centroid VECTOR(1024);
```

Backfill: embed 13 canonical titles + ~60 aliases, compute mean per role, UPDATE.

### B2: Unvalidated Thresholds (Reviewer-SY4, Reviewer-SY5, Nolan)

**Problem:** 0.70 similarity and 0.05 gap are guesses. Short-text Voyage embeddings have tighter similarity distributions. "Product Manager" vs "Program Manager" might score above 0.70 against each other.

**Solution:** Run calibration script — embed all ~60 existing aliases, compute pairwise similarity matrix, read off same-role min vs cross-role max. Set threshold at midpoint. One Voyage API call, 10 min to write, 5 seconds to run.

```python
async def calibrate_thresholds():
    # Embed all aliases, compute same-role vs cross-role similarity distributions
    # Set FUZZY_MATCH_THRESHOLD from empirical data
    ...
```

Make thresholds named constants + config values (env var overridable):
```python
title_match_confident_threshold: float = 0.80  # set after calibration
title_match_review_threshold: float = 0.65      # set after calibration
```

### B3: Voyage on Critical Path — No Cache/Timeout/Circuit Breaker (Reviewer-SY5)

**Problem:** Role resolve is Step 0 of capture funnel. Adding ~100ms Voyage call (p99 could spike to 500ms+) with 30s default timeout and no cache is dangerous.

**Solution:**

**LRU Cache** (in-process, no Redis needed):
```python
class EmbeddingCache:
    def __init__(self, max_size: int = 2000):
        self._cache: OrderedDict[str, list[float]] = OrderedDict()
        self._max_size = max_size
```
2000 entries × ~4KB = ~8MB. Near 100% hit rate for repeated title inputs.

**Tight Timeout:** 30s → 2-3s for user-facing path.

**Circuit Breaker:** Trips after 3 consecutive Voyage failures, resets after 60s. No library needed — simple class with failure counter.

**Graceful Degradation:** If Voyage down or circuit open, fall back to `pg_trgm` fuzzy string matching against alias table (microseconds, no external API).

### B4: Batch Size 10 in Part B (Everyone)

**Problem:** 5-10K titles at batch 10 = 500-1,000 API calls. Voyage supports 128. Codebase already uses 50.

**Solution:** Change `embedding_batch_size` in config from 10 to 50. One config value. 5x speedup.

### B5: Miss Logging Race Condition (Nolan)

**Problem:** Current code logs miss BEFORE fuzzy fallback runs. Causes false positive misses.

**Solution:** Restructure `resolve_phase1` — try fuzzy match before logging. Only log to `role_alias_misses` when both exact and fuzzy paths fail.

```python
async def resolve_phase1(pool, raw_input, session_id=None):
    aliases = await catalog_db.get_aliases_for_input(pool, raw_input)
    if not aliases:
        fuzzy_aliases = await catalog_db.get_aliases_fuzzy(pool, raw_input, limit=5, min_similarity=0.4)
        if not fuzzy_aliases:
            await catalog_db.log_alias_miss(pool, raw_input, session_id)
            return _build_resolved(confidence=0.0, ...)
        aliases = fuzzy_aliases
    # ... rest unchanged
```

New DB function using pg_trgm:
```python
async def get_aliases_fuzzy(pool, raw_input, limit=5, min_similarity=0.4):
    # SELECT *, similarity(alias, $1) FROM role_aliases
    # WHERE similarity(alias, $1) >= $2 ORDER BY similarity DESC LIMIT $3
```

Requires GIN trigram index:
```sql
CREATE INDEX idx_role_aliases_alias_trgm ON role_aliases USING gin (alias gin_trgm_ops);
```

---

## Part A Concerns + Solutions

### C1: Silent Misclassification (Sloane)

**Problem:** If "DevOps Engineer" maps to "Software Engineer" via fallback, user gets wrong translations with no explanation. Worse than rejection.

**Solution:** When normalization maps input to a different canonical title, show user:
> "Based on your input, we matched you to **Product Manager**. Is this right?"
> [Yes, that's me] [Not quite — let me clarify]

Store `raw_input` alongside `canonical_role` in session state. Frontend renders confirmation card when confidence < 0.95.

### C2: resolved_via Column Not Wired (Nolan)

**Problem:** Migration adds column but INSERT in `log_alias_miss()` never writes it.

**Solution:** Migration adds `resolved_via TEXT`, `resolved_at TIMESTAMPTZ`, `resolved_to TEXT` to `role_alias_misses`. New function `resolve_alias_miss()` does UPDATE when fuzzy/embedding resolves a previously logged miss.

### C3: Max-Across-Clusters Bias (Reviewer-SY4)

**Problem:** `max(cosine_similarity)` across clusters means roles with more clusters get more lottery tickets.

**Solution:** Use mean instead of max. Or use role_title_centroid directly (single comparison per role, zero bias).

### C4: No Observability (Reviewer-SY5)

**Problem:** Can't tune thresholds without seeing real similarity scores.

**Solution:** Structured log on every normalization: `raw_title`, `normalized_to`, `similarity`, `method` (cache_hit/voyage_embed/fuzzy_fallback), `elapsed_ms`, `circuit_breaker_open`. Extend admin dashboard with title normalization health stats.

### C5: Catalog Cluster Embedding Caching (Nolan)

**Problem:** Every fallback re-fetches all cluster embeddings from DB.

**Solution:** Simple in-memory TTL dict (1hr). Invalidate on catalog sync.

### C6: RLS Missing for New Tables (Nolan)

**Problem:** New tables need RLS policies.

**Solution:** `title_taxonomy` and any review queue tables get `service_role` only access.

### C7: No Feedback Loop from Disambiguation (Reviewer-SY4)

**Problem:** When users consistently pick candidate #2 over #1 in disambiguation, that signal should feed back.

**Solution:** Stub the feedback path — log disambiguation choices for post-launch analysis. Don't auto-update priors yet.

---

## Part B: Decouple to Post-Launch

**Unanimous:** Part B (title taxonomy from scraped data) is strategic gold but wrong week. Ship as separate spec the week after launch.

### Part B Solutions (for when it ships)

**Reviewer-SY4:** Use role_title_centroids as comparison targets, not single role-name embeddings. Add gap requirement to auto-approve threshold. Make weekly flow incremental (only embed new titles).

**Reviewer-SY5:** Dedup via `ON CONFLICT DO NOTHING`. Include monitoring. Define embedding lifecycle (cleanup, recomputation on model change).

**Nolan:** Idempotency guard via `alias_expansion_runs` table — skip if successful run exists in last 6 days.

**Reviewer-BH6 — Framework for New Canonical Roles vs Aliases:**
- Add ALIAS when: input maps to existing competency vocabulary, seniority variant, or company-culture title variant
- Add NEW ROLE when 3+ of: 10+ sessions requesting it in 7 days, 40%+ vocabulary divergence from nearest catalog, separate career paths, different recruiter-side vocabulary
- First expansion candidates: DevOps/SRE, UX Researcher, Technical Program Manager, Solutions Engineer

### Reviewer-BH6 — Pragmatic Part A Alternative

If embeddings feel like too much for this week: just add pg_trgm GIN index + fuzzy fallback query + seed 100+ high-frequency aliases in a migration. "Technical Product Manager" trigram-matches to "Product Manager" without any embedding call. **This alone probably solves the March 23 problem.**

---

## Recommended Build Sequence

| Day | Action |
|-----|--------|
| 1 | Mine alias misses, add top 50 as aliases (data patch) |
| 1 | Migration: `role_title_centroid` column, `resolved_via` columns, pg_trgm index |
| 2 | Backfill centroids (13 embed calls) + calibration script |
| 2 | LRU cache + circuit breaker + timeout fix |
| 3 | Fuzzy fallback in resolver + restructure miss logging |
| 3 | Tests + make check |
| 4 | FE: confirmation card when confidence < 0.95 |
| 5 | Integration test full capture flow |

---

## Migration Summary (Part A)

```sql
-- 028_role_title_normalization.sql

-- 1. Role title centroid for embedding comparison
ALTER TABLE competency_catalogs
    ADD COLUMN role_title_centroid VECTOR(1024);

-- 2. Resolution tracking on alias misses
ALTER TABLE role_alias_misses
    ADD COLUMN resolved_via TEXT
        CHECK (resolved_via IN ('exact', 'fuzzy', 'embedding', 'taxonomy', 'manual')),
    ADD COLUMN resolved_at TIMESTAMPTZ,
    ADD COLUMN resolved_to TEXT;

-- 3. Trigram index for fuzzy matching
CREATE INDEX IF NOT EXISTS idx_role_aliases_alias_trgm
    ON role_aliases USING gin (alias gin_trgm_ops);
```

---

## File Summary (Part A)

| File | Action | Source |
|------|--------|--------|
| migrations/028_role_title_normalization.sql | Create | Nolan, Reviewer-SY4 |
| src/soaria/services/role_resolver.py | Edit — fuzzy fallback, restructure miss logging | Nolan |
| src/soaria/db/catalog.py | Edit — add get_aliases_fuzzy(), get_all_catalog_centroids() | Nolan, Reviewer-SY4 |
| src/soaria/services/embedding_cache.py | Create — LRU cache + circuit breaker | Reviewer-SY5 |
| src/soaria/llm/client.py | Edit — tight timeout, circuit breaker integration | Reviewer-SY5 |
| scripts/backfill_title_centroids.py | Create — one-time centroid computation | Reviewer-SY4 |
| scripts/calibrate_thresholds.py | Create — pairwise similarity matrix | Reviewer-SY4 |
| tests/test_role_normalization.py | Create | All |
