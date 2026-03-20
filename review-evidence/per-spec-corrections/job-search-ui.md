# Contract Corrections: Job Search UI

Source: Nolan (Backend Foil) cross-review

---

## Interaction Type Enum Fix

```ts
// WRONG (spec):
interaction_type: "save" | "pass" | "unsave"

// RIGHT (contract):
interaction_type: "viewed" | "saved" | "passed" | "clicked_apply"
```

- `"save"` → `"saved"`
- `"pass"` → `"passed"`
- `"unsave"` → does NOT exist. File RFC or remove toggle-unsave UI.

## Pass Feedback Shape Fix

```ts
// WRONG (spec):
{ interaction_type: "pass", feedback: { reason: "wrong_level" } }

// RIGHT (contract):
{ interaction_type: "passed", pass_reason: "wrong_level" }
```

Flatten `PassFeedback` to use `pass_reason?: string | null` directly on `InteractionRequest`. Drop `customText` (not in contract) or file RFC for structured feedback.

## Optimize CTA Fix

```ts
// WRONG: { target_job_id: job.id }
// RIGHT: { job_id: job.id }
```

## SSE Wire Type — PROPOSED, NOT AUTHORITATIVE

`SSEJobMatchCard` does not exist in contracts. Field mismatches with existing `FeedJobCard`:

| Spec SSEJobMatchCard | Contract FeedJobCard |
|---------------------|---------------------|
| `company: string` | `company: FeedCompany \| null` (object) |
| `title: string` | `job_title: string` |
| `remote: boolean` | `is_remote: boolean` |
| `fit_score: number` | `overall_fit_score: number` |
| `vocabulary_mappings` | Does not exist |
| `salary_range: { min, max, currency }` | `min_annual_salary`, `max_annual_salary`, `salary_string` (flat) |
| `posted_at` | `date_posted` |
| missing | `is_hybrid: boolean` |
| missing | `url: string` |
| missing | `job_id: string` (needed for interactions!) |

**Action:** File RFC for Bryce to publish `SSEJobMatchCard` type. Mark all wire types as "PROPOSED — awaiting backend contract." Add `job_id` and `url` to the proposed type.

## Canvas Type Conflict

`JobCardData` (canvas-engine spec) and `JobMatchDisplay` (this spec) have incompatible property names:
- `vocabMappings[].candidatePhrase` vs `vocabularyMappings[].candidateText`
- `JobCardData` missing many `JobMatchDisplay` fields

**Fix:** Unify into one type. `JobCardData` in canvas.ts should BE `JobMatchDisplay` with a `type: "job"` discriminator.

## Other

- `JobSearchEmpty` declared as server component but has `onQuerySelect` callback. Must be client component.
- Add `is_hybrid` field from contract → render "Hybrid" badge.
