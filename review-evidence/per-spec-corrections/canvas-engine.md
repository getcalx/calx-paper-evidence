# Contract Corrections: Canvas Engine

Source: Nolan (Backend Foil) cross-review

---

## Blocked Endpoints (Spec 26 Required)

`GET /v1/canvas` and `PUT /v1/canvas` do NOT exist. These are Spec 26 (canvas state persistence), which hasn't been written yet.

**Action:** Block Chunk 2 (canvas-persistence.ts) and Chunk 9 on Spec 26 contract publication. File RFC with proposed wire shape.

**Wire shape conflict:** Spec has TWO different shapes (`CanvasPersistedState` and `BackendCanvasState`). Consolidate into one and propose via RFC. Recommend `BackendCanvasState` (has width/height/collapsed, layout_version).

## Card Type Naming Fix

```ts
// WRONG (spec):
type CardType = "translation" | "gap" | "job" | ...

// RIGHT (SSE contract from FOIL_REVIEW_FIXES):
card_type: "translation_card" | "gap_card" | "agent_commentary" | "resume_diff" | "download_link"
```

Either match backend naming (`"translation_card"`) or define explicit mapper from SSE `card_type` to frontend `CardType`. Document the mapping.

**Missing card types:** Add `"application_confirmation"` and `"application"` to `CardType` union as stubs for forward compatibility.

## Type Shape Fixes

**`WireIdentityNode` — invented, no contract:**
Don't define this. Use `UserProfileResponse` from `GET /v1/me` (fields that exist: `full_name`, `current_job_title`, `current_company`, `career_stage`). Block Chunk 4 on Spec 23E contract publication for the full IdentityNode.

**`UserProfile` — shadows contract:**
```ts
// WRONG (spec):
interface UserProfile { name, headline, skills, experienceSummary, avatarUrl }

// Available from UserProfileResponse:
{ full_name, current_job_title, current_company, career_stage }
// NOT available: headline, skills, experienceSummary, avatarUrl
```

Add mapper from `UserProfileResponse` → display type. File RFC for missing fields or derive what you can (`headline` = `current_job_title + " at " + current_company`).

**`PlanTier`** — no backend source. No subscription tier field in `UserProfileResponse`. Hardcode "Free" for MVP or block on subscription contract (Spec 25 → types.ts).

## Field Fixes

| Spec Says | Contract Says |
|-----------|--------------|
| `ChatMessage.role: "agent"` | `ChatMessageItem.role: "assistant"` |

Add `"system"` to role union too (contract includes it).

## Endpoint Fixes

- `PUT /v1/me/discoverability` doesn't exist. File RFC or add `discoverable_by_recruiters` to `PUT /v1/me` (also needs RFC since field isn't in `UserProfileUpdate`).

## Other

- AppHeader layout inverted from PRD. PRD says: Left = user name, plan badge, context indicator. Right = admin icon, logout. Spec has it reversed.
- No `session_id` management for authenticated canvas chat. Add section: check `GET /v1/agent/sessions` on mount, use active session or create on first message.
- Auth expired redirect goes to `/translate` (anonymous) — should go to re-auth flow, not anonymous page.
