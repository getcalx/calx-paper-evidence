# Spec [NUMBER]: [Title]

**Status:** Draft | Review | Approved | Locked | Building | Complete
**Author:** Bryce (Backend Lead)
**Date:** YYYY-MM-DD
**PRD Section:** [Number and title from prd-mvp.md]

---

## Overview

[1-2 sentences: what this spec does and why.]

## Key Decisions

[Decisions made during spec writing. Include PRD Divergence callouts where the spec departs from PRD.]

> **PRD Divergence:** [If spec differs from PRD, explain what, why, and which is correct.]

---

## Migration

**Migration number:** [Next available — do not hardcode. Verify at build time.]

```sql
-- Migration: [description]
-- Depends on: [prior migrations]

[DDL here]
```

### RLS Policies (MANDATORY for every new table)

Every new table with RLS enabled MUST have all three policy types. Do not skip any.

```sql
-- 1. Owner policy (user access to own data)
CREATE POLICY "[table]_owner" ON [table]
  FOR ALL USING ([ownership_column] = auth.uid());

-- 2. Admin SELECT policy (admin dashboard access)
CREATE POLICY "[table]_admin_select" ON [table]
  FOR SELECT USING (
    auth.uid() IN (SELECT user_id FROM user_profiles WHERE role = 'admin')
  );

-- 3. Service role policy (backend service access)
CREATE POLICY "[table]_service_role" ON [table]
  FOR ALL USING (auth.role() = 'service_role');
```

**RLS Verification Checklist:**
- [ ] Every new table has `ENABLE ROW LEVEL SECURITY`
- [ ] Owner policy uses correct column (verify: is it `user_id`, `recruiter_id`, or a subquery?)
- [ ] Admin SELECT policy follows migration 005 pattern
- [ ] Service role ALL policy present if any backend service writes to this table
- [ ] If table has a FK to another table's ID (not `auth.uid()` directly), use a subquery in the owner policy

---

## Models

[Pydantic request/response models. Every field explicitly typed — no `dict` or `Any`.]

---

## Endpoints

[For each endpoint: method, path, auth requirement, request/response models, error responses.]

---

## Service Layer

[Business logic. SQL queries must reference column names verified against migration DDL above.]

---

## Column Verification (MANDATORY)

Every SQL query in this spec must be verified against the migration DDL. Fill in this checklist before review.

| Query Location | Column Referenced | Verified in DDL? | Notes |
|---------------|-------------------|-------------------|-------|
| [chunk/section] | [column_name] | [ ] | |

**Common failure modes:**
- Using `confidence` when the column is `confidence_score`
- Using `resume_version_id` when the column is `to_version_id` or `from_version_id`
- Using `user_id` in RLS when the table uses `recruiter_id` (requires subquery)
- Referencing columns from the PRD data model instead of the actual migration

---

## FE Contract Updates (MANDATORY if spec touches response models)

If this spec adds, removes, or renames any field on a Pydantic response model, document the corresponding TypeScript contract update here. Faye builds against these contracts, not the Pydantic models.

| Pydantic Model | Field Change | TS Contract Update |
|----------------|-------------|-------------------|
| [ModelName] | Added `field_name: type` | Add `field_name: tsType` to `contracts/types.ts` |

**Rules:**
- Every new response field must appear in the TS contract
- Every removed response field must be flagged for TS contract removal
- Every renamed field must show old → new in both Pydantic and TS
- If a new SSE event type is added, define the TS interface here
- Reference: `~/WebstormProjects/versed_fe/specs/WIRE_CONTRACTS.md` for current contracts

---

## Chunks

[Break into 1-3 file chunks. Each chunk lists exact files it creates/modifies.]

| Chunk | Files | Description | Dependencies |
|-------|-------|-------------|-------------|
| [A] | [file1, file2] | [what it does] | — |
| [B] | [file1] | [what it does] | A |

---

## Tests

[Test list. Include integration tests for all endpoints, edge cases for business logic, and specific tests for any anonymous/unauthenticated paths.]

---

## Acceptance Criteria

- AC-1: [Criteria]
- AC-2: [Criteria]

---

## Risks & Dependencies

- **Risk:** [risk description]
- **Dependency:** [dependency description]
