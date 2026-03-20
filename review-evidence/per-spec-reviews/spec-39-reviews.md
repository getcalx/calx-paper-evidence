# Spec 39: Production Hardening — Reviews

## Changelog
- **R1 (2026-03-15):** 8 blockers across 5 chunks. Wrong index (39A), RLS policy mismatch (39D), missing webhook handler + suppression scope + auth model (39E).
- **R2 (2026-03-15):** 39D and 39E fully resolved. **39A:** 2 duplicate indexes to remove (already exist under different names from migrations 004/014). **39B:** Reviewer-SY5 wants structured logging on Redis fallback. **39E observation:** HMAC should include `expires` in signed payload. **Decision: BUILD-READY** with 39A index cleanup and 39B logging note.

---

## R1 (2026-03-15)

## Nolan (Backend Foil) — Per-Chunk Verdicts

### 39A: REVISE (1 blocker)
- `idx_llm_log_created` wrong for budget enforcement. Hot path is `WHERE user_id = $1 AND created_at >= $2`. Need `(user_id, created_at DESC)` composite. Check if `idx_llm_call_log_daily` and `idx_llm_call_log_user` already exist from data contracts.

### 39B: APPROVE
- Narrow scope, correct pattern. Observation: fallback behavior (in-memory vs pass-all) worth a comment.

### 39C: APPROVE
- Correct dual-write architecture. Observation: Redis cold-start reconciliation not specified but acceptable at this scale.

### 39D: REVISE (1 blocker)
- RLS policy "Admin read audit_log" has body `auth.role() = 'service_role'`. Name says admin, body blocks admins. Replace with standard three-policy pattern. Also: nullable `actor_id` needs `actor_type` convention.

### 39E: REVISE (2 blockers)
- Resend webhook handler is acceptance criterion with no file in change table. Missing: endpoint, signature verification, payload parsing.
- `is_suppressed()` doesn't distinguish transactional vs marketing. Unsubscribed user can't receive password reset.
- No RLS on `email_suppressions` table.

---

## Sloane (Frontend Foil) — REVISE (3 blockers)

### S39-1: Audit log actor_id not resolvable
UUID in actor column is unrenderable. Response model must include `actor_email` resolved at query time. Pagination shape missing.

### S39-2: Unsubscribe auth model undefined
`POST /v1/me/unsubscribe` with JWT won't work for email link clicks (no session). Need signed token path or dual auth.

### S39-3: No `Retry-After` header on 429
FE has no signal for backoff duration. Add header + extend `ApiError` TS type with `retry_after_seconds`.

---

## Reviewer-SY4 (Systems/ML)

Indexes serve admin listing patterns, not enforcement patterns. Missing `user_id` coverage on `llm_call_log` and `subscriptions`. Redis paradigm fit is correct for rate limiting and budget tracking. Audit log `details JSONB` is right for heterogeneous payloads. Email suppression schema encodes permanence where the domain requires optionality — no unsuppress path for soft bounces.

---

## Reviewer-SY5 (Production Systems)

Per-chunk: 39A solid after fixing the index. 39B "graceful degradation" to in-memory on multi-instance = no rate limiting. 39C correct dual-write but missing cold-start reconciliation. 39D functional but unconstrained `action TEXT` will make incident queries useless in 6 months. 39E most incomplete — missing Resend webhook endpoint, HMAC verification, complaint handling, re-subscription path.

---

## Reviewer-BH6 (Affective Computing)

Rate limits, audit visibility, and email control are affective user experiences, not neutral plumbing. Rate limit at 5/hour during translation reveal is a trust rupture — should surface signup prompt, not 429. Audit log is a panopticon without transparency — users can't see what's been done to their accounts. Email suppression is a one-way autonomy violation with no re-subscribe path.

Opportunity: Rate limit event is the highest-intent conversion moment in the anonymous funnel.

---

## Vivian (Design Lead)

4 gaps: (1) Audit log JSONB needs rendering contract and actor resolution. (2) Unsubscribe needs landing page design — single confirmation screen, brand-consistent. (3) Rate limit errors need defined 429 response body and user-facing copy. (4) No admin health surface for LLM budget, rate limit metrics, or audit log.

---

## R2 (2026-03-15) — Post-Revision

### Nolan — Per-Chunk
- **39A REVISE (1):** `idx_llm_log_user_created` duplicates existing `idx_llm_call_log_user` from migration 004 (different name, same columns). `idx_subscriptions_status` duplicates migration 014. Remove both, add comments.
- **39B APPROVE**
- **39C APPROVE**
- **39D APPROVE:** RLS fixed. `actor_email` resolved via JOIN. Pagination shape present. Observation: define action string constants before first write.
- **39E APPROVE:** All 3 R1 blockers resolved. Observation: HMAC should include `expires` in signed payload to prevent expiry extension attacks.

### Reviewer-SY5 — Per-Chunk
- **39A APPROVE:** Composite index correct.
- **39B REVISE:** Silent per-instance in-memory fallback on multi-instance = bypass, not degradation. Need `structlog.warning("rate_limit_degraded")` on Redis failure.
- **39C APPROVE:** Dual-write architecture correct. Cold-start acceptable at scale.
- **39D APPROVE:** RLS fixed. `actor_type` and unconstrained `action` are operational debt, not blockers.
- **39E APPROVE:** All 3 blockers resolved. Webhook handler, suppression scope, RLS all present.

### Reviewer-SY4 — APPROVE all chunks
Index representations now match query patterns. Redis paradigm fit correct. Suppression scope model correctly distinguishes reversible vs permanent.

### Reviewer-BH6 — APPROVE
Suppression scope resolves the autonomy concern. Signed token unsubscribe preserves one-click. Audit log panopticon remains (users can't see their own audit entries) — tracked debt, not blocker.

### Sloane — APPROVE
`actor_email` resolved. Signed token path defined. `Retry-After` header is FE-side contract, correctly out of scope for this BE spec.

### Vivian — APPROVE (with holds)
39A/C/D clear. 39B needs `Retry-After` header and 429 response body before build. 39E needs unsubscribe landing page design (Vivian's queue) before going live to users.
