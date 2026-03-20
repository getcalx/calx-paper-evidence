# Backend Lead (Bryce) — Lessons

Append-only log of corrections and learnings. Each entry captures what went wrong, the correction, and the takeaway.

Note: Code-level ACE lives in the backend repo at `src/soaria/tasks/lessons.md`. This file captures cross-session strategic and architectural learnings.

---

## L001: RLS policies are the #1 recurring foil review failure (2026-03-12)

**What happened:** Across v2, v3, and v4 foil reviews, RLS policy gaps appeared in every round. Wrong column references (N28-1: `auth.uid()` vs `recruiter_id`), missing admin policies (N28-5, 30-F1), missing service_role policies (29-F1). These are silent failures — queries return empty, no errors.

**Correction:** Added mandatory RLS section to spec template with three policy types (owner, admin SELECT, service_role ALL). Added rules 23-25 covering the checklist.

**Takeaway:** RLS is invisible when wrong. Make it structural (template-enforced) not procedural (remember to check).

---

## L002: Column name drift between PRD and migrations (2026-03-12)

**What happened:** v3 had 3 column name drift issues (N27-2: `resume_version_id` vs `to_version_id`, N30-2: `confidence` vs `confidence_score`, N29-5: multiple column name mismatches). Specs were written against PRD data models, not verified against actual migration DDL.

**Correction:** Added Column Verification checklist to spec template. Every SQL query must be verified against migration DDL before review. v4 had zero column name drift issues.

**Takeaway:** The fix worked — zero recurrence in v4. Keep the checklist.

---

## L003: FE contract drift is a new error class (2026-03-12)

**What happened:** B2B specs (27-30) have complex response models. Pydantic models gained fields that weren't reflected in TypeScript contracts. Faye builds against TS contracts. 27-F4 (5 missing fields), 30-F2 (incomplete CohortStatusPoll).

**Correction:** Added mandatory FE Contract Updates section to spec template. Every Pydantic response model change now requires a documented TS contract update. WIRE_CONTRACTS.md serves as the single source.

**Takeaway:** As specs get more complex, the Pydantic-to-TS boundary needs explicit tracking, not implicit assumption.

---

## L004: Anonymous endpoint specs need HOW, not just WHAT (2026-03-12)

**What happened:** v3 fix S28-2 correctly scoped the anonymous recruiter path but described only what it does, not how it works. v4 found three blockers (28-F1: NOT NULL constraint blocks anonymous storage, 28-F2: claim pattern undocumented, 28-F3: rate limiting unspecified). All from one feature.

**Correction:** Documented complete implementation: nullable FK, claim_token pattern, DB-backed rate limiting with IP extraction. Added rule 37 about multi-step atomicity.

**Takeaway:** "Add an anonymous endpoint" is not a complete spec. Anonymous paths need: storage (how to store without FK), claiming (how to link on signup), and rate limiting (how to prevent abuse). Spec all three.

---

## L005: Competing confidence mechanisms — pick one, not both (2026-03-14)

**What happened:** Spec 32 declares Beta distribution parameters (`confidence_alpha`, `confidence_beta`) on `cluster_vocabulary` but the `bayesian_confidence_update` function uses direct scalar math on the `confidence` float. The function never reads or writes alpha/beta. Additionally, alpha/beta were typed as INT (truncating fractional updates to zero), and initial confidence was set to 1.0 with a Beta(1,1) prior (which should = 0.5). Three reviewers caught the same contradiction independently.

**Correction:** Choose one mechanism. Either drop alpha/beta and use scalar (simpler), or implement real Beta distribution updates and derive confidence from `alpha / (alpha + beta)`.

**Takeaway:** When declaring a statistical model in a spec, verify the implementation function actually uses it. Dead schema columns that imply a model nobody implements are worse than no model — they mislead future developers about how the system works.

---

## L006: Obsidian schema ↔ DB schema must be verified bidirectionally (2026-03-14)

**What happened:** Spec 32 had `onet_soc_code` (singular TEXT) in Obsidian frontmatter but `onet_soc_codes` (plural JSONB array) in the database. The sync pipeline reads singular and writes to plural — type mismatch crash. Also, YAML boolean coercion (`yes` → `True`) was acknowledged but not closed in Pydantic validators.

**Correction:** Every field that flows from Obsidian frontmatter to Postgres must be verified in both directions: (1) Obsidian field name + type → (2) Pydantic ingestion model field + validator → (3) DB column name + type. All three must agree or have an explicit transformation step.

**Takeaway:** The Obsidian→Postgres seam is a three-layer boundary (YAML → Python → SQL). Verify each layer against the other two. Add explicit YAML-footgun validators (bool rejection on string fields, None handling on required fields) at the Pydantic layer since YAML is the most permissive format.

---
