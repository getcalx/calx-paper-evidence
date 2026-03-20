# Foil Review Fixes — Specs 27-30 (v4)

**Date:** 2026-03-12
**Round:** Re-review after Bryce applied v3 fixes
**Reviewers:** Nolan (BE foil), Sloane (FE foil)
**Prior round:** All 32 v3 fixes verified as landed. New issues found during re-review.

---

## Summary

| Spec | Nolan New | Sloane New | Unique Issues | Blockers |
|------|-----------|------------|---------------|----------|
| 27 | 3 | 2 | 4 (1 overlap) | 1 |
| 28 | 4 | 4 | 6 (2 overlap) | 3 |
| 29 | 2 | 4 | 6 | 3 |
| 30 | 3 | 3 | 4 (2 overlap) | 2 |
| **Total** | **12** | **13** | **20** | **9** |

---

## Spec 27 — Application Tracking v2

### 27-F1: `models/applications.py` not in any chunk's file list (HIGH)

**Source:** Nolan (N27-R1)
**Problem:** S27-1 and S27-4 both require changes to `models/applications.py` — making
`resume_version_id` nullable and extending `ApplicationSource` with `"merge_synced"`.
These changes appear only as comments inside the chunk 27B code block for
`models/merge.py`. The chunk table lists only `models/merge.py (new), config.py` as
27B deliverables. `models/applications.py` isn't in any chunk's file list.

**Fix:** Add `models/applications.py` to chunk 27B's file list. Change description to:
"Merge.dev models + config + application model updates." Move the `ApplicationSource`
and `resume_version_id` changes from comments into explicit spec text in chunk 27B.

---

### 27-F2: Metadata `message_type` value mismatch — reminders silently fail (BLOCKER)

**Source:** Nolan (N27-R3) + Sloane (NEW-1) — both foils caught this
**Problem:** Chunk 27F metadata structure defines `"message_type": "follow_up_reminder"`.
But the planner match text says `metadata.message_type == 'reminder'`. Different values.
Planner never detects reminders. Feature silently fails.

**Fix:** Change the planner match text to `metadata.message_type == 'follow_up_reminder'`.
One-line fix.

---

### 27-F3: `job_id` mapping for Merge-synced apps unspecified (MEDIUM)

**Source:** Nolan (N27-R4)
**Problem:** Merge syncs applications from external ATSes. Those applications reference
jobs that don't exist in Versed's `jobs` table. The spec never specifies what `job_id`
is set to. The `UNIQUE(user_id, job_id)` constraint could block Merge syncs depending
on implementation.

**Fix:** Add to chunk 27C: "For Merge-synced applications, `job_id` is NULL unless a
matching Versed job can be identified by company + title. The `UNIQUE(user_id, job_id)`
constraint allows multiple NULL `job_id` entries per user (PostgreSQL treats NULLs as
distinct in unique constraints)."

---

### 27-F4: Response models missing 5 new Merge fields (MEDIUM)

**Source:** Sloane (NEW-2)
**Problem:** Migration adds `merge_application_id`, `vocabulary_translations_used`,
`outcome_linked_at`, `last_synced_at`, `sync_source` to the `applications` table.
None are added to `ApplicationResponse`/`ApplicationDetail`. Pydantic silently drops
them. FE components (`OutcomeInsight`, `ApplicationCard` origin indicator) render empty.

**Fix:** Add to chunk 27B model update instructions:

```python
# ApplicationResponse / ApplicationDetail — add:
merge_application_id: str | None = None
vocabulary_translations_used: list[dict] | None = None
outcome_linked_at: datetime | None = None
last_synced_at: datetime | None = None
sync_source: str = "manual"
```

---

### 27-F5: Migration comment for clarity (NON-BLOCKING)

**Source:** Nolan (N27-R2)
**Problem:** Migration 016 adds `outcome_validated BOOLEAN DEFAULT FALSE` to
`reframing_patterns` but this column already exists from migration 009. The
`IF NOT EXISTS` makes it safe (no-op) but could confuse Bryce during build.

**Fix:** Add migration comment: `-- outcome_validated already exists from migration 009;
IF NOT EXISTS makes this a no-op. Adding outcome_validated_at and
outcome_application_id as new columns.`

---

## Spec 28 — Recruiter Backend

### 28-F1: Anonymous analysis storage blocked by NOT NULL constraint (BLOCKER)

**Source:** Nolan (N28-6) + Sloane (NEW-S28-1) — both foils caught this
**Problem:** Migration DDL defines `recruiter_id UUID NOT NULL REFERENCES
recruiter_profiles(id)`. The anonymous endpoint (S28-2 fix) stores analyses with
`recruiter_id = NULL`. Spec acknowledges the conflict in a parenthetical but never
resolves it. Anonymous endpoint INSERT fails with NOT NULL violation.

**Fix — Option A (recommended, matches candidate session claim pattern):**

```sql
-- Change line 153 from:
recruiter_id UUID NOT NULL REFERENCES recruiter_profiles(id) ON DELETE CASCADE,
-- To:
recruiter_id UUID REFERENCES recruiter_profiles(id) ON DELETE CASCADE,
-- NULL for anonymous analyses, populated on signup claim
```

Add partial index for anonymous analyses:
```sql
CREATE INDEX idx_jd_analyses_anonymous ON jd_analyses (created_at)
  WHERE recruiter_id IS NULL;
```

Update RLS: anonymous rows (NULL `recruiter_id`) are invisible to authenticated
recruiters until claimed. Anonymous INSERT uses service_role path.

---

### 28-F2: Anonymous claim pattern undocumented (BLOCKER)

**Source:** Sloane (NEW-S28-2)
**Problem:** Spec says "On signup, anonymous analyses can be claimed (same pattern as
candidate anonymous session claim)" but provides zero implementation detail. No claim
token, no linking mechanism. Recruiter does the work, signs up, and their analysis
is gone. Worse than no anonymous path — experienced loss at conversion gate.

**Fix — Add "Claim Pattern" subsection to 28G:**

1. Anonymous endpoint returns a `claim_token` (UUID) in the response alongside the
   `JDAnalysisResponse` fields.
2. Frontend stores `claim_token` in sessionStorage.
3. On recruiter profile creation (`POST /v1/recruiter/profile`), frontend sends
   accumulated `claim_tokens: string[]` in the request body.
4. Backend: `UPDATE jd_analyses SET recruiter_id = $new_recruiter_profile_id
   WHERE claim_token = ANY($tokens) AND recruiter_id IS NULL`.
5. If no tokens or no matches, skip silently.

Add `claim_token UUID DEFAULT gen_random_uuid()` to `jd_analyses` migration DDL.
Add unique index on `claim_token`.

---

### 28-F3: Rate limiting mechanism unspecified (BLOCKER)

**Source:** Nolan (N28-7) + Sloane (NEW-S28-4) — both foils caught this
**Problem:** Spec says "3/hour per IP" but provides no implementation. No IP-based rate
limiting exists in the codebase. Railway proxy means `request.client.host` gives proxy
IP, not user IP. Without this, anonymous endpoint is an LLM cost attack vector.

**Fix — Add to 28G:**

Implementation (DB-backed, no Redis dependency):
```sql
-- Add to migration:
CREATE TABLE IF NOT EXISTS anonymous_rate_limits (
  ip_address TEXT NOT NULL,
  endpoint TEXT NOT NULL,
  requested_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_rate_limits_lookup ON anonymous_rate_limits (ip_address, endpoint, requested_at);
```

```python
# IP extraction (Railway uses X-Forwarded-For):
client_ip = request.headers.get("x-forwarded-for", "").split(",")[0].strip()
            or request.client.host

# Check: SELECT COUNT(*) FROM anonymous_rate_limits
#   WHERE ip_address = $1 AND endpoint = $2
#   AND requested_at > NOW() - INTERVAL '1 hour'
# Cleanup: Prefect daily task or pg_cron: DELETE WHERE requested_at < NOW() - INTERVAL '24 hours'
```

Rate limit response contract:
```
HTTP 429
Body: {"detail": "Rate limit exceeded. Try again in X minutes.", "retry_after_seconds": N}
Header: Retry-After: N
```

FE renders: "You've analyzed 3 JDs this hour. Create an account for unlimited access."
with capture prompt.

---

### 28-F4: `error_message` not populated in failure path + missing from migration (MINOR)

**Source:** Nolan (N28-8)
**Problem:** `error_message` field exists on `JDAnalysisResponse` but the failure path
in 28E never populates it. Also, `error_message` column is not in the `jd_analyses`
migration DDL — field only exists on the Pydantic model.

**Fix:**
1. Add `error_message TEXT` to `jd_analyses` DDL in 28A migration.
2. Add to 28E failure path: set `error_message` to user-facing string on failure:
   - LLM timeout: "Analysis timed out. Please try again."
   - Parse failure: "Unable to analyze this job description. Please check the format."
   - API error: "Analysis service temporarily unavailable. Please try again shortly."

---

### 28-F5: `JDAnalysisSummary` missing `error_message` (MINOR)

**Source:** Sloane (NEW-S28-3)
**Problem:** List endpoint model `JDAnalysisSummary` includes `status` but not
`error_message`. Recruiter sees "failed" in list view with no explanation — must click
into detail for context.

**Fix:** Add `error_message: str | None = None` to `JDAnalysisSummary`.

---

### 28-F6: No tests for anonymous endpoint (MINOR)

**Source:** Nolan (N28-9)
**Problem:** 28H lists 12 tests. None cover the anonymous endpoint.

**Fix — Add to 28H:**
- `test_anonymous_jd_analysis` — returns `JDAnalysisResponse` without auth
- `test_anonymous_rate_limit` — 4th request within 1 hour returns 429
- `test_anonymous_analysis_claim` — after signup, anonymous analyses linked to profile

---

## Spec 29 — B2B API Products

### 29-F1: Missing service_role RLS on log tables (BLOCKER)

**Source:** Nolan (N29-NEW-2)
**Problem:** `api_subscriptions` and `persona_library` have service_role policies.
`jd_optimization_log` and `outreach_optimizations` do not. Application writes to
both tables via service_role pool (`get_db_pool()`). RLS is enabled but no
service_role policy exists. Every JD optimization and outbound optimization call
fails with permission error on log insertion.

**Fix:**

```sql
CREATE POLICY "Service role full access jd_optimization_log"
  ON jd_optimization_log FOR ALL
  USING (auth.role() = 'service_role');

CREATE POLICY "Service role full access outreach_optimizations"
  ON outreach_optimizations FOR ALL
  USING (auth.role() = 'service_role');
```

---

### 29-F2: Enterprise tier `monthly_limit: None` causes TypeError (BLOCKER)

**Source:** Nolan (N29-NEW-3)
**Problem:** `PLAN_TIER_CONFIG` sets enterprise `monthly_limit: None`. Auth middleware
does `if sub.usage_count >= sub.monthly_limit` — comparing `int >= None` raises
`TypeError` in Python. Enterprise tier is DOA.

Additionally, `monthly_limit` column is `INTEGER NOT NULL` in DDL. Enterprise INSERT
with `None` fails at DB level too.

**Fix — Option A (sentinel, simpler):**
Use `999999999` as the "unlimited" sentinel for enterprise tier. No nullable column,
no None guards needed anywhere.

**Fix — Option B (nullable):**
1. Make `monthly_limit` nullable: `INTEGER` (drop NOT NULL) in DDL
2. Update Pydantic model: `monthly_limit: int | None = None`
3. Add None guard in middleware:
```python
if sub.monthly_limit is not None and sub.usage_count >= sub.monthly_limit:
    raise HTTPException(429, "Monthly usage limit exceeded")
```
4. Apply same guard in `increment_api_usage` if it does a similar check.

Recommend Option A for simplicity.

---

### 29-F3: Orphaned `vocabulary_changes` field on `OutboundOptimizationResponse` (HIGH)

**Source:** Sloane (NEW-S29-1)
**Problem:** Response has both `changes_made: list[OutreachChange]` and
`vocabulary_changes: list[VocabularyChange]`. LLM prompt only produces `changes_made`.
`vocabulary_changes` is always empty. Two change lists with no documented distinction
is developer-hostile for a B2B API product.

**Fix:** Remove `vocabulary_changes` from `OutboundOptimizationResponse`. `changes_made`
covers the same concept with a superset of fields.

---

### 29-F4: `GET /v1/api/personas` returns `list[dict]` — untyped (HIGH)

**Source:** Sloane (NEW-S29-4)
**Problem:** Same untyped `dict` pattern S29-1 fixed everywhere else. SDK generation
produces `Record<string, unknown>[]`. Inconsistent with the typed approach.

**Fix:** Define `PersonaResponse` model:

```python
class PersonaResponse(SoariaBase):
    id: str
    role_type: str
    seniority: str
    industry: str | None = None
    communication_preferences: dict  # Flexible per persona
    vocabulary_profile: dict         # Flexible per persona
    description: str | None = None
```

Update endpoint return type + service functions.

---

### 29-F5: No error response contract for API consumers (MEDIUM)

**Source:** Sloane (NEW-S29-2)
**Problem:** B2B API product with no documented error shapes. Two different 429 cases
(rate limit vs monthly quota) need distinct error codes. SDK authors discover error
shapes empirically.

**Fix — Add "Error Response Contract" section:**

```python
class APIErrorResponse(SoariaBase):
    error: str           # Machine-readable error code
    message: str         # Human-readable description
    detail: dict | None = None
```

| HTTP Status | Error Code | When |
|-------------|-----------|------|
| 401 | `invalid_api_key` | Missing/invalid API key |
| 403 | `subscription_inactive` | Subscription suspended |
| 422 | `validation_error` | Request body fails validation |
| 429 | `rate_limit_exceeded` | Per-minute rate limit |
| 429 | `monthly_limit_exceeded` | Monthly quota exhausted |
| 500 | `internal_error` | LLM failure |

---

### 29-F6: No API versioning policy (LOW)

**Source:** Sloane (NEW-S29-3)
**Problem:** Endpoints at `/v1/api/` imply versioning commitment with no documented
policy. External consumers need a stability promise.

**Fix — Add brief "Versioning & Stability" section:**
- Additive changes (new optional fields): non-breaking, no version bump.
- Removal/rename of fields: breaking change, requires `/v2/api/` prefix.
- Deprecation: `Sunset` + `Deprecation` headers, 90-day minimum notice.

---

## Spec 30 — Cohort Delivery Pipeline

### 30-F1: Missing admin RLS on `cohort_requests` (BLOCKER)

**Source:** Nolan (N30-NEW-5)
**Problem:** `cohort_candidates` and `cohort_feedback_log` have admin SELECT policies.
`cohort_requests` does not. Admin dashboard endpoints (`GET /v1/admin/cohorts`) will
silently return empty results if any admin query path uses the user's JWT instead of
service role.

**Fix:**

```sql
DO $$ BEGIN
  IF NOT EXISTS (
    SELECT 1 FROM pg_policies
    WHERE tablename = 'cohort_requests' AND policyname = 'Admin read cohort_requests'
  ) THEN
    CREATE POLICY "Admin read cohort_requests" ON cohort_requests
      FOR SELECT USING (
        EXISTS (SELECT 1 FROM user_profiles WHERE id = auth.uid() AND role = 'admin')
      );
  END IF;
END $$;
```

---

### 30-F2: `CohortStatusPoll` TS contract incomplete (BLOCKER)

**Source:** Sloane (S30-R2-1)
**Problem:** Pydantic `CohortStatusResponse` has 8 fields. The `CohortStatusPoll`
TypeScript interface has 3. Missing: `cohort_id`, `candidates_matched`, `target_count`,
`error_message`, `created_at`, `updated_at`. Faye builds against incomplete contract.
`candidates_matched`/`target_count` needed for "Found 14 of 20 candidates" progress.
`error_message` needed for `failed` state.

**Fix:** Delete the separate `CohortStatusPoll` type. Tell Faye to use the full
`CohortStatusResponse` shape. One type, no drift. Update the FE contract:

```ts
interface CohortStatusResponse {
  cohort_id: string;
  status: "pending" | "processing" | "matching" | "delivered" | "expired" | "failed";
  candidates_matched: number;
  target_count: number;
  progress_pct: number;
  estimated_seconds_remaining: number | null;
  error_message: string | null;
  created_at: string;
  updated_at: string;
}
```

---

### 30-F3: `expired` status missing from UX Contract + polling interval inconsistency (MINOR)

**Source:** Nolan (N30-NEW-4) + Sloane (S30-R2-2) — both foils caught `expired`.
Nolan (N30-NEW-1) + Sloane (S30-R2-3) — both foils caught polling interval.

**Fix (two items, same pass):**

1. Add `expired` row to Frontend Polling UX Contract table:
   `expired` → Expired → "This cohort has expired. Create a new one." + "New Cohort" CTA.
   Add `"expired"` to status union in TS type.

2. Change Async Status Contract section from "polls every 5 seconds" to "polls every
   3 seconds" to match the UX Contract subsection.

---

### 30-F4: Reattribution line at reveal moment — cross-spec flag (NON-BLOCKING)

**Source:** Sloane (S30-R2-5)
**Observation:** The reveal moment (`POST .../reveal`) is the highest-value point in
the recruiter funnel. Reattribution line ("This candidate described [4 skills you need].
They just used different words.") belongs here but isn't referenced. Cross-spec gap —
vocabulary-translation-ui owns the component (Spencer decision). Not blocking Spec 30
but flagging for Faye's recruiter-canvas spec revision.

---

## Cross-Cutting Patterns (v4)

### Pattern: Anonymous endpoint implementation gaps
28-F1, 28-F2, 28-F3 — Spec 28's anonymous endpoint was correctly scoped per Spencer's
decision but the implementation details (storage, claim, rate limiting) were left
unresolved. Three blockers from one feature. Root cause: the S28-2 fix in v3 described
the WHAT but not the HOW.

### Pattern: RLS policy gaps continue
29-F1, 30-F1 — service_role and admin policies missing on new tables. Same class as
N28-5 (v3). Every new table with RLS needs: owner policy, admin SELECT policy,
service_role ALL policy. Consider a checklist in the spec template.

### Pattern: FE contract drift from Pydantic models
27-F4, 30-F2 — response models gain fields in migration/Pydantic but the TS contracts
aren't updated to match. Faye builds against the TS contract, not the Pydantic model.
Every Pydantic response model change needs a corresponding TS contract update.

---

## Verification Checklist

```
Spec 27:
- [x] 27-F1: models/applications.py in chunk 27B file list
- [x] 27-F2: planner match string → "follow_up_reminder"
- [x] 27-F3: job_id mapping language in chunk 27C
- [x] 27-F4: 5 new fields on ApplicationResponse/ApplicationDetail
- [x] 27-F5: migration comment for outcome_validated no-op

Spec 28:
- [x] 28-F1: recruiter_id nullable in DDL + partial index + RLS update
- [x] 28-F2: claim pattern subsection with claim_token
- [x] 28-F3: rate limit table + IP extraction + response contract
- [x] 28-F4: error_message column in DDL + failure path population
- [x] 28-F5: error_message on JDAnalysisSummary
- [x] 28-F6: 3 anonymous endpoint tests in 28H

Spec 29:
- [x] 29-F1: service_role RLS on jd_optimization_log + outreach_optimizations
- [x] 29-F2: enterprise monthly_limit — sentinel (999999999), no nullable column
- [x] 29-F3: remove vocabulary_changes from OutboundOptimizationResponse
- [x] 29-F4: PersonaResponse typed model + endpoint return type
- [x] 29-F5: error response contract section
- [x] 29-F6: versioning policy section

Spec 30:
- [x] 30-F1: admin SELECT RLS on cohort_requests
- [x] 30-F2: delete CohortStatusPoll, expand CohortStatusResponse TS contract
- [x] 30-F3: add expired to UX table + fix polling interval to 3s
```

All 20 fixes applied. Decision: 29-F2 uses sentinel value (999999999) for enterprise unlimited.
