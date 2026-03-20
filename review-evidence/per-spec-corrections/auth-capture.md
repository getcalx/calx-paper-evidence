# Contract Corrections: Auth & Capture

Source: Nolan (Backend Foil) cross-review

---

## Claim Endpoint Fix

**Request body:**
```ts
// WRONG (spec):
POST /v1/agent/sessions/claim
{ session_id: "...", claim_token: "..." }

// RIGHT (contract):
POST /v1/agent/sessions/claim
{ claim_token: "..." }
// + Authorization: Bearer <jwt>
```

Remove `session_id` from claim request body in 4 locations: ClaimFlow state machine step 5, data flow step 8b, API integration section, ClaimFlow component behavior.

**Response body:**
```ts
// WRONG (spec):
{ session_id, messages_transferred, patterns_transferred }

// RIGHT (contract):
{ session_id: string, messages_transferred: number }
```

Remove `patterns_transferred` from `ClaimResponse` and `ClaimResult`. Update success message from "Transferred {n} messages and {m} vocabulary translations" to "Transferred {n} messages."

## Resume Upload Endpoint Fix

```
// WRONG (spec):
POST /v1/resumes/upload → ResumeMetadata { id, status, filename, uploaded_at }

// RIGHT (contract):
POST /v1/resumes (multipart/form-data) → ResumeTemplateResponse (201)
// Required form fields: file, template_name, job_focus
// Response includes: id, user_id, template_name, job_focus, template_content, is_active, baseline_version_id, etc.
```

Update ResumeUpload component to POST to correct path with correct form fields and handle 201 + correct response shape.

## Error Response Shape

```ts
// WRONG: { "error": "token_not_found", "message": "..." }
// RIGHT: { "detail": "...", "error_code": "NOT_FOUND" }
```

Update all error parsing.

## Other Fixes

- `interactionCount`: StreamContext doesn't have this field. Either add to StreamState or maintain a local `useRef<number>` in the page component. Document the source explicitly.
- `requiredRole` checks `user.is_recruiter` — this field doesn't exist on Supabase User. Role lives on backend profile (`GET /v1/me` → `role` field). Either call `/v1/me` for role check, or defer `requiredRole` for MVP (candidate-only).
- SavePrompt ARIA: `aria-modal="false"` is correct (non-blocking). Add note explaining why. Still needs `aria-labelledby` and focus management per CLAUDE.md.
