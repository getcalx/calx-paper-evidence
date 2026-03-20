# Contract Corrections: Recruiter Canvas UI

Source: Nolan (Backend Foil) cross-review

---

## MAJOR: Entire Backend Is Spec 28 (Not Written Yet)

ALL 8 recruiter endpoints are unpublished:
- `POST /v1/recruiter/analyze-jd`
- `POST /v1/recruiter/jd/{id}/fix`
- `POST /v1/recruiter/jd/{id}/optimize`
- `POST /v1/recruiter/jd/{id}/optimize/approve`
- `GET /v1/recruiter/analyses`
- `GET /v1/recruiter/analyses/{id}`
- `GET /v1/recruiter/analyses/{id}/cards`
- `POST /v1/recruiter/cohort`

**All wire types are PROPOSED, not authoritative.** Bryce owns API contracts. Mark them as such.

**Action:** File RFCs for each endpoint cluster. Build chunks 1-8 with mock fixtures. Chunk 9 integration blocked on Spec 28 contract publication.

## Contract Gaps

1. **No "recruiter" role** — `UserRoleUpdate.role` is `"candidate" | "admin"` only. RFC needed to add `"recruiter"`.
2. **`PUT /v1/me` has no `perspective` field** — `UserProfileUpdate` doesn't include it. RFC needed.
3. **Recruiter chat endpoint unresolved** — separate `/v1/recruiter/agent/chat` vs shared `/v1/agent/chat` with context flag. Bryce must decide. File RFC.
4. **Analyze JD request body undefined** — spec says `{ content }` but backend likely expects `{ jd_text }` (matching DB column). Define in RFC.

## Path Fixes

```
// CONFLICT in spec:
POST /v1/recruiter/jd/optimize      (Backend Dependencies table)
POST /v1/recruiter/jd/{jd_id}/optimize  (JD Optimization flow)

// Standardize on:
POST /v1/recruiter/jd/{jd_id}/optimize
```

Cohort paths: singular `POST /v1/recruiter/cohort` vs plural `POST /v1/recruiter/cohorts/{id}/feedback`. Standardize pluralization.

## Missing Definitions

- Fix response shape (`POST .../fix`) undefined — what does success return? Need `updated_alignment_score` for optimistic UI refresh.
- SSE `card_type: "jd_center"` not in the card type union — add it or document that JDCenterNode populates from GET, not SSE.
- `CohortResults` response has no `cohortId` — needed for feedback endpoint path.

## Security

Add section noting:
- All recruiter endpoints must enforce RLS (`recruiter_id` matches JWT)
- Frontend never passes `recruiter_id` — backend derives from JWT
- 404 (not 403) for unauthorized access (prevent enumeration)
