# Foil Review Fixes — Specs 26-30 (v3)

**Date:** 2026-03-12
**Reviewers:** Nolan (BE foil), Sloane (FE foil)
**Verdict:** REVISE (32 issues across Specs 27-30). Spec 26 approved by both foils.
**Pattern:** Integration boundary failures dominate again — RLS policies, column mismatches, cross-spec contract drift.

---

## Spencer's Decisions (Resolved During Review)

| # | Decision | Rationale |
|---|----------|-----------|
| 1 | Application tracking limit removed — unlimited for all tiers | Gating tracking adds enforcement complexity for a feature that feeds the data flywheel. More tracking = more outcome signal = better vocabulary mappings. Pro upgrade lever is proactive features only. |
| 2 | Recruiter try-before-buy anonymous path goes in Spec 28 | Endowment-before-capture must work for recruiters too. Anonymous JD analysis endpoint with lower rate limits. |
| 3 | PRD limits are source of truth for Spec 29 monthly API limits | 5/20/50/Custom for JD optimization. Can adjust in pilots. |
| 4 | Spec 29 response contracts: spec's richer models win where more comprehensive | Field names must be reconciled — update PRD to match spec where spec is more robust. Don't force spec to match PRD's simpler shapes. |

---

## Spec 26 — Canvas State Persistence: APPROVED

Both foils approve. No action needed. Locked.

---

## Spec 27 — Application Tracking v2

### Nolan (Backend Foil): REVISE — 4 blockers

---

### N27-1: `agent_messages.metadata` JSONB column doesn't exist

**Spec:** 27 (Application Tracking v2), chunk 27F
**Problem:** Spec says "Use the existing JSONB metadata field on agent_messages to tag
reminders." The column does not exist. The `agent_messages` table (migration 009) has:
`id`, `session_id`, `sequence`, `role`, `content`, `tool_name`, `tool_input`,
`tool_result`, `card_artifacts`, `token_count`, `created_at`. No `metadata`.

**Fix — migration 016:**

```sql
ALTER TABLE agent_messages ADD COLUMN IF NOT EXISTS metadata JSONB DEFAULT '{}';
```

**FE impact:** None — metadata is backend-internal for reminder tagging.

---

### N27-2: Outcome linking query uses wrong column name

**Spec:** 27 (Application Tracking v2), chunk 27E
**Problem:** SQL query uses `WHERE rod.resume_version_id = $1` but
`resume_optimization_deltas` has `from_version_id` and `to_version_id`, not
`resume_version_id`. Core reinforcement signal broken — outcome linking silently returns
zero rows.

**Fix:** Change `rod.resume_version_id` to `rod.to_version_id` in the outcome linking
query and chunk description.

**FE impact:** None — outcome linking is backend-internal.

---

### N27-3: `application_source` CHECK constraint missing `'merge_synced'`

**Spec:** 27 (Application Tracking v2), chunk 27C
**Problem:** Existing `applications` table CHECK constraint allows only `'soaria_direct'`
and `'self_reported'`. Merge-synced application INSERTs will fail with constraint
violation.

**Fix — migration 016:**

```sql
ALTER TABLE applications DROP CONSTRAINT applications_application_source_check;
ALTER TABLE applications ADD CONSTRAINT applications_application_source_check
  CHECK (application_source IN ('soaria_direct', 'self_reported', 'merge_synced'));
```

**FE impact:** See S27-4 — frontend `ApplicationSource` enum must also extend.

---

### N27-4: No `applied_at` derivation for Merge-synced applications

**Spec:** 27 (Application Tracking v2), chunk 27C
**Problem:** `applied_at DATE NOT NULL` has no default. Merge sync service must provide
a date but the field mapping isn't specified. INSERTs without explicit `applied_at` fail.

**Fix — add to chunk 27C:** "For new applications (INSERT), set `applied_at` from
Merge's `applied_at` field, falling back to `remote_created_at`, falling back to `NOW()`."

**FE impact:** None.

---

### Sloane (Frontend Foil): REVISE — 5 findings

---

### S27-1: `ApplicationResponse.resume_version_id` typed as non-optional

**Spec:** 27 (Application Tracking v2)
**Problem:** Migration makes `resume_version_id` nullable but the existing Pydantic
model at `models/applications.py` doesn't match. Merge-synced apps with NULL will
cause Pydantic validation errors, returning 500s on the list endpoint.

**Fix — `src/soaria/models/applications.py`:**

```python
# ApplicationResponse and ApplicationDetail:
resume_version_id: UUID | None = None  # Nullable for Merge-synced applications

# ApplicationCreate:
resume_version_id: UUID | None = None  # Optional for Merge-synced creation paths
```

**FE impact:** Frontend must handle `resume_version_id: null` in application cards.
No resume link for Merge-synced apps until user attaches one.

---

### S27-2: `vocabulary_mappings_used` vs PRD's `vocabulary_translations_used`

**Spec:** 27 (Application Tracking v2)
**Problem:** Spec uses `vocabulary_mappings_used` throughout. PRD, UX flow spec, and
brand all use "translations." Frontend will build against PRD naming.

**Fix:** Rename to `vocabulary_translations_used` in migration, service, tests, and AC-7.
Consistent with brand vocabulary.

**FE impact:** Frontend uses `vocabulary_translations_used` — no mapping needed once
spec is corrected.

---

### S27-3: No SSE event shape for real-time status sync

**Spec:** 27 (Application Tracking v2)
**Problem:** Webhook updates hit the DB but frontend has no push notification. The UX
flow spec defines an `app_update` SSE event but this spec doesn't emit it. Recruiter
status changes appear only on next page load.

**Fix — add to chunk 27C or 27D:** When `sync_application_status` successfully updates
status, emit an `app_update` SSE event.

**Payload model:**
```python
class ApplicationStatusData(SoariaBase):
    application_id: UUID
    status_before: str
    status_after: str
    sync_source: str
    updated_at: datetime
```

**FE contract — `contracts/types.ts`:**
```ts
interface ApplicationStatusEvent {
  event_type: "app_update";
  application_id: string;
  status_before: string;
  status_after: string;
  sync_source: string;
  updated_at: string; // ISO 8601
}
```

---

### S27-4: `ApplicationSource` enum not extended for Merge

**Spec:** 27 (Application Tracking v2)
**Problem:** Merge-synced apps have no valid source value for the frontend's
ApplicationCard origin indicator.

**Fix:** Extend `ApplicationSource` to include `"merge_synced"` (aligns with N27-3
adding `merge_synced` to CHECK constraint).

**FE contract:**
```ts
type ApplicationSource = "soaria_direct" | "self_reported" | "merge_synced";
```

---

### S27-5: Reminder message structure underspecified

**Spec:** 27 (Application Tracking v2), chunk 27F
**Problem:** Frontend gets a text string but `FollowUpPrompt` component needs structured
metadata with actions.

**Fix — define reminder message structure in 27F with metadata fields:**

```python
# agent_messages.metadata for reminder messages:
{
    "message_type": "follow_up_reminder",
    "reminder_source": "stale_application",
    "application_id": "<uuid>",
    "stale_days": 14,
    "job_title": "Senior Engineer",
    "company_name": "Acme Corp",
    "render_as": "follow_up_prompt",
    "actions": ["mark_updated", "snooze_7d", "dismiss"]
}
```

**FE contract:**
```ts
interface FollowUpReminderMetadata {
  message_type: "follow_up_reminder";
  reminder_source: "stale_application";
  application_id: string;
  stale_days: number;
  job_title: string;
  company_name: string;
  render_as: "follow_up_prompt";
  actions: ("mark_updated" | "snooze_7d" | "dismiss")[];
}
```

---

### Spencer Decision (now moot):

Application tracking limit removed for all tiers. See [Cross-Spec: Application Tracking
Limit Removal](#cross-spec-application-tracking-limit-removal) at end of document.

---

## Spec 28 — Recruiter Backend

### Nolan (Backend Foil): REVISE — 5 blockers

---

### N28-1: RLS policy on `jd_analyses` uses wrong column

**Spec:** 28 (Recruiter Backend)
**Problem:** RLS policy compares `auth.uid()` to `recruiter_id` (auto-generated UUID from
`gen_random_uuid()`), not `user_id`. These are different UUIDs. Will block ALL recruiter
access to their own analyses.

**Fix — RLS policy must use subquery:**

```sql
CREATE POLICY jd_analyses_recruiter_select ON jd_analyses
  FOR SELECT USING (
    recruiter_id IN (
      SELECT id FROM recruiter_profiles WHERE user_id = auth.uid()
    )
  );
```

Apply same pattern for INSERT, UPDATE, DELETE policies on `jd_analyses`.

**FE impact:** Without this fix, every recruiter API call returns empty results.

---

### N28-2: `RecruiterContext` references `UserClaims` — type doesn't exist

**Spec:** 28 (Recruiter Backend)
**Problem:** Codebase uses `AuthenticatedUser`, not `UserClaims`. Build fails at import
time.

**Fix:** Change `UserClaims` to `AuthenticatedUser` in `RecruiterContext` and all
references in Spec 28.

**FE impact:** None — backend type only.

---

### N28-3: Migration number 017 is wrong

**Spec:** 28 (Recruiter Backend)
**Problem:** Highest existing migration is 010. Specs 19-27 claim 011-016. Actual number
depends on build order.

**Fix:** Note that migration number is tentative — use next available sequential number
at build time. Do not hardcode 017.

**FE impact:** None.

---

### N28-4: PRD says CHECK constraint role, spec uses boolean — no reconciliation note

**Spec:** 28 (Recruiter Backend)
**Problem:** PRD Section 8 says add `'recruiter'` to `user_profiles.role` CHECK. Spec
uses `is_recruiter BOOLEAN` (better design — additive, users can be both candidate and
recruiter). But no PRD divergence note.

**Fix — add PRD Divergence callout in Key Decisions:**

> **PRD Divergence:** PRD specifies adding 'recruiter' to user_profiles.role CHECK
> constraint. Spec uses `is_recruiter BOOLEAN` instead. Boolean approach is superior:
> a user can be both candidate and recruiter simultaneously. PRD should be updated to
> match. Frontend must check `is_recruiter`, not `role`.

**FE impact:** Frontend checks `is_recruiter` boolean, not role enum. Must be documented
in contracts.

---

### N28-5: No admin RLS policies for recruiter tables

**Spec:** 28 (Recruiter Backend)
**Problem:** Every existing table with RLS has admin read policies (migrations 004-005).
Spec omits admin access for `recruiter_profiles` and `jd_analyses`. Admin dashboard
can't read recruiter data.

**Fix — add admin SELECT policies matching existing pattern from migration 005:**

```sql
CREATE POLICY recruiter_profiles_admin_select ON recruiter_profiles
  FOR SELECT USING (
    auth.uid() IN (SELECT user_id FROM user_profiles WHERE role = 'admin')
  );

CREATE POLICY jd_analyses_admin_select ON jd_analyses
  FOR SELECT USING (
    auth.uid() IN (SELECT user_id FROM user_profiles WHERE role = 'admin')
  );
```

**FE impact:** Admin panel can read recruiter data. No candidate-facing impact.

---

### Sloane (Frontend Foil): REVISE — 2 findings

---

### S28-1: No `error_message` in `JDAnalysisResponse` for failed analyses

**Spec:** 28 (Recruiter Backend)
**Problem:** When analysis fails, frontend gets `status='failed'` with zero context.
Recruiter sees a dead state with no explanation.

**Fix — add field to `JDAnalysisResponse`:**

```python
class JDAnalysisResponse(SoariaBase):
    # ... existing fields
    error_message: str | None = None  # User-facing message on failure
```

Populate with user-facing message when analysis fails (e.g., "Unable to analyze this
job description. Please check the format and try again.").

**FE contract:**
```ts
interface JDAnalysisResponse {
  // ... existing fields
  error_message: string | null; // Non-null when status === "failed"
}
```

---

### S28-2: Recruiter try-before-buy anonymous path (SPENCER DECIDED: goes in Spec 28)

**Spec:** 28 (Recruiter Backend)
**Problem:** Capture funnel spec says recruiters can try with no auth. Spec 28 gates
`analyze-jd` behind `require_recruiter`. Endowment-before-capture pattern broken for
recruiter side.

**Fix — add anonymous JD analysis endpoint:**

- New endpoint: `POST /v1/recruiter/analyze-jd/anonymous`
- No auth required (no `require_recruiter` dependency)
- Lower rate limits: 3/hour per IP (vs authenticated recruiter limits)
- Returns same `JDAnalysisResponse` shape
- No recruiter profile created — analysis stored with `recruiter_id = NULL`
- On signup, anonymous analyses can be claimed (same pattern as candidate session claim)

**FE contract:** Same response shape as authenticated endpoint. Frontend routes to
anonymous endpoint when no JWT present.

---

## Spec 29 — B2B API Products

### Nolan (Backend Foil): REVISE — 6 blockers

---

### N29-1: Monthly limits unresolved (SPENCER DECIDED: PRD numbers)

**Spec:** 29 (B2B API Products)
**Problem:** DECISION NEEDED tag still open for monthly API limits.

**Fix:** Use PRD limits for JD optimization:

| Tier | Monthly JD Optimization Limit |
|------|-------------------------------|
| Starter | 5 |
| Growth | 20 |
| Scale | 50 |
| Enterprise | Custom |

Update `PLAN_TIER_CONFIG`. Remove the `DECISION NEEDED` tag.

**FE impact:** Dashboard shows correct limits per tier.

---

### N29-2: `require_recruiter` dependency doesn't exist

**Spec:** 29 (B2B API Products), chunk 29H
**Problem:** Spec 28 hasn't shipped. The auth dependency used in 29H endpoints
(`require_recruiter`) doesn't exist in `api/deps.py`. Current auth deps are
`get_current_user`, `require_approved`, `require_admin`.

**Fix:** Make the dependency explicit — Spec 28 must be built first. `require_recruiter`
is defined in Spec 28. Spec 29 endpoints using it are blocked until Spec 28 ships.
Build order: 28 → 29.

**FE impact:** None — sequencing issue only.

---

### N29-3: JD optimization response contract diverges from PRD (SPENCER DECIDED: spec wins)

**Spec:** 29 (B2B API Products)
**Problem:** Spec uses `suggested`/`context`. PRD uses `optimized`/`reason`. Spec's flat
`alignment_score` vs PRD's nested `{before, after}`.

**Fix:** Keep spec's richer structure. Reconcile field names:

| Concept | Spec Name | PRD Name | Resolution |
|---------|-----------|----------|------------|
| Replacement text | `suggested` | `optimized` | Keep `suggested` — more accurate (these are suggestions) |
| Explanation | `context` | `reason` | Keep `context` — broader than a single reason |
| Score shape | flat `alignment_score` | nested `{before, after}` | Keep spec's flat score |

Update PRD to match spec. Add PRD Divergence callout in spec.

**FE impact:** API consumers build against spec's field names.

---

### N29-4: Outbound response contract diverges from PRD (SPENCER DECIDED: spec wins)

**Spec:** 29 (B2B API Products)
**Problem:** Spec uses `changes_made`, PRD uses `vocabulary_changes`. Missing
`suggestions` field. Split resonance score vs single.

**Fix:** Same principle — keep spec's richer models. Reconcile naming:

| Concept | Spec Name | PRD Name | Resolution |
|---------|-----------|----------|------------|
| Change list | `changes_made` | `vocabulary_changes` | Keep `changes_made` — spec is more specific |
| Score shape | split resonance | single resonance | Keep spec's split (more granular signal) |

Update PRD to match spec. Add PRD Divergence callout.

**FE impact:** API consumers build against spec's field names.

---

### N29-5: `outreach_optimizations` column names diverge from PRD

**Spec:** 29 (B2B API Products)
**Problem:** `subscription_id` vs `api_subscription_id`, `persona_id` vs `persona` JSONB,
`int` vs `decimal` scores, renamed boolean columns.

**Fix:** Keep spec's approach (reference ID, integer scores 0-100). More practical than
PRD's decimal scores and inline JSONB. Note PRD data model update needed.

| Column | Spec | PRD | Resolution |
|--------|------|-----|------------|
| FK to subscriptions | `subscription_id` | `api_subscription_id` | Keep `subscription_id` (standard FK naming) |
| Persona reference | `persona_id` | `persona JSONB` | Keep `persona_id` (normalized, not inline) |
| Score type | `INTEGER` (0-100) | `DECIMAL` | Keep `INTEGER` (simpler, sufficient precision) |

**FE impact:** None — backend schema only.

---

### N29-6: Missing RLS on `jd_optimization_log` and `outreach_optimizations`

**Spec:** 29 (B2B API Products)
**Problem:** These tables contain customer-specific data but have no RLS policies. Only
`api_subscriptions` and `persona_library` got RLS.

**Fix — add RLS policies for both tables:**

```sql
-- jd_optimization_log
ALTER TABLE jd_optimization_log ENABLE ROW LEVEL SECURITY;
CREATE POLICY jd_optimization_log_owner ON jd_optimization_log
  FOR ALL USING (
    subscription_id IN (
      SELECT id FROM api_subscriptions WHERE user_id = auth.uid()
    )
  );

-- outreach_optimizations
ALTER TABLE outreach_optimizations ENABLE ROW LEVEL SECURITY;
CREATE POLICY outreach_optimizations_owner ON outreach_optimizations
  FOR ALL USING (
    subscription_id IN (
      SELECT id FROM api_subscriptions WHERE user_id = auth.uid()
    )
  );
```

Add admin SELECT policies for both following migration 005 pattern.

**FE impact:** None — RLS is transparent to frontend.

---

### Sloane (Frontend Foil): REVISE — 2 blockers

---

### S29-1: Three untyped `dict` fields in public API models

**Spec:** 29 (B2B API Products)
**Problem:** `repellent_phrases: list[dict]`, `changes_made: list[dict]`,
`candidate_context: dict | None`. These are developer-facing API products — the response
shapes ARE the product contract. Untyped dicts are unacceptable.

**Fix — define typed Pydantic models:**

```python
class RepellentPhrase(SoariaBase):
    phrase: str
    category: str
    severity: str          # "high", "medium", "low"
    suggestion: str

class OutreachChange(SoariaBase):
    original: str
    suggested: str
    reason: str

class CandidateContext(SoariaBase):
    role_type: str | None = None
    seniority: str | None = None
    industry: str | None = None
    skills: list[str] = []
    linkedin_about: str | None = None
```

Replace `dict` fields with typed model references in all response models.

**FE impact:** API consumers get typed contracts. SDK generation produces real types
instead of `Record<string, unknown>`.

---

### S29-2: Inconsistent field naming across sibling APIs

**Spec:** 29 (B2B API Products)
**Problem:** `VocabularyChange.suggested` vs outbound `changes_made[].optimized` for
the same concept (a suggested replacement).

**Fix:** Use `suggested` consistently across both products. "Suggested" is more accurate
— these are suggestions, not guaranteed improvements.

| Model | Current | Fixed |
|-------|---------|-------|
| `VocabularyChange` | `suggested` | `suggested` (no change) |
| `OutreachChange` | `optimized` | `suggested` |

**FE impact:** API consumers use one term for the same concept across products.

---

## Spec 30 — Cohort Delivery Pipeline

### Nolan (Backend Foil): REVISE — 4 blockers

---

### N30-1: RLS on `cohort_requests` references `user_id` but column is `recruiter_id`

**Spec:** 30 (Cohort Delivery Pipeline)
**Problem:** RLS policy uses `auth.uid() = user_id` but the column is `recruiter_id`.
Policy will silently match nothing. Recruiters can't read their own cohort requests.

**Fix:**

```sql
CREATE POLICY cohort_requests_recruiter ON cohort_requests
  FOR ALL USING (auth.uid() = recruiter_id);
```

**FE impact:** Without this fix, cohort request list returns empty for all recruiters.

---

### N30-2: Confidence adjustment SQL uses wrong column name

**Spec:** 30 (Cohort Delivery Pipeline)
**Problem:** Feedback adjustment SQL references `confidence` but `reframing_patterns`
table (migration 009) defines the column as `confidence_score`. Every feedback submission
that adjusts confidence fails at runtime.

**Fix:** Change all references from `confidence` to `confidence_score` in the feedback
adjustment SQL and safeguard logic.

```sql
-- Wrong:
UPDATE reframing_patterns SET confidence = confidence + $1 WHERE ...
-- Fixed:
UPDATE reframing_patterns SET confidence_score = confidence_score + $1 WHERE ...
```

**FE impact:** None — backend-internal.

---

### N30-3: `reveal_credits_remaining` ALTER missing from migration DDL

**Spec:** 30 (Cohort Delivery Pipeline)
**Problem:** Spec describes adding `reveal_credits_remaining` to `recruiter_profiles`
but the migration SQL block doesn't include the ALTER statement. Column doesn't exist
at runtime.

**Fix — add to migration DDL:**

```sql
ALTER TABLE recruiter_profiles
  ADD COLUMN IF NOT EXISTS reveal_credits_remaining INTEGER NOT NULL DEFAULT 0;
```

**FE impact:** Recruiter dashboard needs this field to display remaining reveal credits.

---

### N30-4: `CohortStatusResponse` and `CohortCandidateRevealed` models orphaned

**Spec:** 30 (Cohort Delivery Pipeline)
**Problem:** Models defined inline in endpoint chunks but not in `models/cohort.py`
where they belong per codebase conventions (all models live in `models/` package).

**Fix:** Add both models to 30B model listing in `models/cohort.py`. Update 30G and
30G2 to import from `models/cohort` instead of defining inline.

**FE impact:** None — same wire format either way.

---

### Sloane (Frontend Foil): REVISE — 3 blockers

---

### S30-1: Anonymized summary atomicity gap

**Spec:** 30 (Cohort Delivery Pipeline)
**Problem:** `CohortCandidatePublic` requires `anonymized_summary` (non-optional) but
`CohortCandidate` has it as optional. If pipeline fails mid-insert, recruiter sees
broken cohort at conversion gate — candidate rows with null summaries cause 500s when
serializing to `CohortCandidatePublic`.

**Fix — add explicit language in 30E:** Anonymized summary generation and
`cohort_candidates` insertion happen atomically. If anonymization fails for any
candidate, exclude that candidate from the cohort rather than inserting with null
summary. Log the exclusion for monitoring.

**FE impact:** Frontend always receives complete candidate entries. No null-summary
defensive handling needed.

---

### S30-2: No polling UX contract for processing state

**Spec:** 30 (Cohort Delivery Pipeline)
**Problem:** Recruiter submits JD, waits minutes. The spec has `progress_pct` and
`estimated_seconds_remaining` but no guidance on what UI states map to which status
values. Faye has no spec for what to render during processing.

**Fix — add "Frontend Polling UX Contract" subsection:**

| Status | UI State | Display |
|--------|----------|---------|
| `pending` | Submitted | "Request submitted" — spinner |
| `processing` | Analyzing | "Analyzing vocabulary..." — progress bar at `progress_pct` |
| `matching` | Matching | "Finding candidates..." — progress bar at `progress_pct` |
| `delivered` | Complete | Redirect to cohort view |
| `failed` | Error | "Something went wrong. Please try again." — retry button |

`progress_pct` renders as determinate progress bar.
`estimated_seconds_remaining` renders as "~X minutes remaining" (round up to nearest minute).

**Polling interval:** 3 seconds while `pending` or `processing` or `matching`.
Stop polling on `delivered` or `failed`.

**FE contract:**
```ts
interface CohortStatusPoll {
  status: "pending" | "processing" | "matching" | "delivered" | "failed";
  progress_pct: number;                    // 0-100
  estimated_seconds_remaining: number | null;
}
```

---

### S30-3: Feedback rating semantics undefined

**Spec:** 30 (Cohort Delivery Pipeline)
**Problem:** 1-5 scale has no labels. The rating is the flywheel signal — labels
determine feedback quality. Without labels, recruiters apply inconsistent scales.

**Fix — define rating semantics:**

| Rating | Label | Meaning |
|--------|-------|---------|
| 1 | Not relevant | Candidate does not match the role |
| 2 | Weak match | Some overlap but significant gaps |
| 3 | Reasonable match | Meets basic requirements, not standout |
| 4 | Strong match | Good fit, would interview |
| 5 | Excellent match | Ideal candidate for the role |

Labels must appear in the UI next to the rating selector.

**FE contract:**
```ts
const RATING_LABELS: Record<number, string> = {
  1: "Not relevant",
  2: "Weak match",
  3: "Reasonable match",
  4: "Strong match",
  5: "Excellent match",
};
```

---

## Cross-Spec: Application Tracking Limit Removal

Per Spencer's decision, the 5 tracked applications/month free-tier limit is removed
everywhere. Application tracking is unlimited for all tiers. The pro upgrade lever is
proactive features only (matching alerts, coaching, optimization suggestions).

### Spec 25 Changes

- Remove `applications_limit` from `UsageLimits` model
- Remove `check_application_access` function
- Remove `applications_limit` from `PLAN_TIER_CONFIG`
- Remove `test_check_application_free` and `test_check_application_pro` tests
- Remove AC-8 reference to 5 tracked applications
- Update AC-10 and any other references to application limits

### Spec 27 Changes

- Remove any reference to application tracking limits
- Merge-synced apps no longer have a limit concern

### PRD Changes

- Update feature comparison table: "5 applications tracked" → "Application tracking" (both tiers: unlimited, no distinction)
- Update AC-2: remove "Free tier: 5 tracked applications"
- Update AC-8: remove "Free tier enforces 5 tracked applications"
- Update Section 6 usage limits line
- Update UX flow `UsageMeter` component (remove "3/5" application counter)

---

## Cross-Cutting Observations

### Pattern: RLS policy failures
N28-1, N30-1 — RLS policies referencing wrong columns. Silent failures (empty results,
not errors). Same pattern as v2 Issue 18. Every new table with RLS needs a verify step.

### Pattern: Column name drift
N27-2, N30-2, N29-5 — specs reference column names that don't match actual migrations.
Caused by specs being written against PRD data models instead of verified against
migration files.

### Pattern: Response contract divergence
N29-3, N29-4, S29-1, S29-2 — spec and PRD disagree on response shapes. Root cause:
specs evolved richer models but PRD wasn't updated in parallel. Spencer's decision
(spec wins where richer) resolves this but PRD needs a reconciliation pass.

### Pattern: Missing atomicity guarantees
S30-1 — partial pipeline failure leaves invalid state. Same class as v2 Issue 10
(trial-to-pro race). Any multi-step write needs explicit atomicity language.

---

## Summary Table

| Spec | Nolan | Sloane | Total Fixes |
|------|-------|--------|-------------|
| 26 | APPROVE | APPROVE | 0 |
| 27 | REVISE (4) | REVISE (5) | 9 |
| 28 | REVISE (5) | REVISE (2) | 7 |
| 29 | REVISE (6) | REVISE (2) | 8 |
| 30 | REVISE (4) | REVISE (3) | 7 |
| Cross-spec | — | — | 1 (limit removal) |
| **Total** | | | **32** |

---

## Verification Checklist

- [ ] Nolan re-reviews all BE fixes (N27-1 through N30-4)
- [ ] Sloane re-reviews all FE-facing fixes (S27-1 through S30-3)
- [ ] Cross-check: every fix that touches both BE and FE has matching contracts
- [ ] All SQL changes have corresponding migration additions
- [ ] All new SSE event types documented in contracts/types.ts
- [ ] RLS policies verified against actual column names (not spec assumptions)
- [ ] PRD divergence callouts added where spec wins over PRD
- [ ] Application tracking limit removed from Specs 25, 27, and PRD
- [ ] No fix introduces a new untested code path
- [ ] Build order dependency documented: Spec 28 before Spec 29

All Spencer decisions are resolved. No open questions remain. Bryce can apply all fixes mechanically.
