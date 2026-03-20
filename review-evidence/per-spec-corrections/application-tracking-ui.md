# Contract Corrections: Application Tracking UI

Source: Nolan (Backend Foil) cross-review

---

## Status Enum Fix

```ts
// WRONG (spec):
type ApplicationStatus = "applied" | "screening" | "interviewing" | "offer" | "accepted" | "rejected" | "withdrawn"

// RIGHT (contract):
type ApplicationStatus = "applied" | "screening" | "interviewing" | "offer" | "rejected" | "ghosted" | "withdrew"
```

- Remove `accepted` (doesn't exist — file RFC if needed)
- `withdrawn` → `withdrew`
- Add `ghosted` (terminal state, needs color/icon mapping)
- Update: state machine, transition table, constants, color map, timeline, filter logic, `TERMINAL_STATUSES`, `PROGRESSIVE_STATUSES`

## Field Renames

| Spec Says | Contract Says | Location |
|-----------|--------------|----------|
| `company` | `company_name` (nullable) | WireApplication, mapApplicationCard |
| `title` | `job_title` (nullable) | WireApplication, mapApplicationCard |
| `notes` | `user_notes` (nullable) | WireApplication, PUT body |
| `note` (PUT body) | `user_notes` | ApplicationStatusUpdateRequest |

## Type Shape Fix

**`WireApplication` → align to `ApplicationResponse`:**
```ts
// RIGHT (contract):
interface WireApplication {
  id: string;
  user_id: string;
  job_id: string | null;        // NULLABLE — spec has it required
  resume_version_id: string;
  resume_source: string | null;
  status: ApplicationStatus;
  application_source: string;
  applied_at: string;
  last_updated_at: string | null;
  user_confirmed: boolean;
  resume_used_unchanged: boolean | null;
  user_notes: string | null;
  created_at: string;
  updated_at: string;
  job_title: string | null;
  company_name: string | null;
}
```

**Fields that DON'T EXIST on backend (need RFC or graceful degradation):**
- `status_history` — no such field. Derive minimal timeline from `status` + `applied_at` + `updated_at`, or file RFC for status audit trail
- `optimization_score` — no such field anywhere
- `vocabulary_mappings_used` — Spec 27 dependency, not built. OutcomeInsight must degrade gracefully.

## Response Format Fix

```ts
// WRONG: spec treats response as raw array
const cards = response.map(mapApplicationCard)

// RIGHT: response is paginated object
interface ApplicationListResponse {
  applications: ApplicationResponse[];
  next_cursor: string | null;
}
const { applications, next_cursor } = response;
const cards = applications.map(mapApplicationCard);
```

Implement cursor-based pagination or infinite scroll.

## HTTP Status Fix

- `POST /v1/applications` returns **201**, not 200. Check `response.ok` or handle 201 explicitly.

## Other

- Free tier limit: spec says 3, PRD says 5 — resolve with Spencer
- `POST /v1/applications/connect-ats` (Merge.dev) doesn't exist — move to out-of-scope or file RFC
- Per-transition notes not supported — backend has single `user_notes` field, not per-status notes
