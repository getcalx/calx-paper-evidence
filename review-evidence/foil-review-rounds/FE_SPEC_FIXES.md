# Frontend Spec Fixes — Compiled Reference

> Generated: 2026-03-12
> Source: Sloane (FE Foil) + Nolan (BE Foil) cross-review — 213 findings across 10 specs
> Wire contract reference: `WIRE_CONTRACTS.md` (same directory)

---

## Spencer Decisions (Override Foil Findings)

These decisions are FINAL. Where a correction file contradicts these, the decision wins.

| # | Decision | Impact |
|---|----------|--------|
| 1 | **App tracking unlimited all tiers** | subscription-trial gating table: applications are NOT tier-limited |
| 2 | **STAR story capture UNGATED, all tiers** | star-stories: remove ALL Pro-only gating. No `useFeatureGate`, no `GatedFeaturePrompt`. Core experience. PRD Key Decision 3 is wrong — spec should reflect this. |
| 3 | **Reattribution line ownership** | vocabulary-translation-ui owns the ReattributionLine component. Other specs (canvas-engine, job-search) reference/import it, don't redefine it. |
| 4 | **Anon recruiter path IN SCOPE** | recruiter-canvas-ui needs to account for anonymous JD analysis (pre-auth). Faye may need a spec addendum. |

---

## Cross-Cutting Fixes (Apply to ALL 10 Specs)

### CC-1: ARIA / Focus Management / Keyboard Navigation

Every overlay, drawer, dropdown, and modal must implement:

```
role="dialog" (or role="menu" for dropdowns)
aria-modal="true" (modals) or aria-modal="false" (non-blocking overlays)
aria-labelledby="{heading-id}"
Focus trap: on open, focus moves to first focusable element
Focus restore: on close, focus returns to trigger element
Escape key closes
```

**Per-spec affected components:**
- admin-panel: UserActions dropdown (`role="menu"`), CodeApplyModal
- application-tracking: ApplicationDrawer, StatusUpdatePicker
- auth-capture: SavePrompt (has `role="dialog"` but missing `aria-labelledby`, focus trap/restore)
- canvas-engine: ChatBubble expanded panel, MobileCarousel
- job-search: SavePassActions popover
- recruiter-canvas: JDOptimizeOverlay
- resume-optimization: OptimizationOverlay
- star-stories: StoryCapturePrompt
- subscription-trial: UpgradeModal (missing focus restore), PreCommitmentPrompt, TrialNotification (`aria-live`, not dialog)

**Every interactive card** (JobMatchCard, StarStoryCard, ApplicationCard):
- `tabIndex={0}`, `role="button"` or `role="article"`
- `aria-expanded` for expand/collapse
- Enter/Space toggles expansion
- Focus ring: `focus-visible:ring-2`

**Icon-only buttons** need `aria-label`.

### CC-2: Mobile Layouts

Every spec needs a "Mobile Behavior" section:
- Mobile (<768px): linear card stack, no canvas
- Drawers/overlays become full-screen on mobile
- Touch targets minimum 44x44px
- Popovers become bottom sheets on mobile
- Filter pills become horizontally scrollable
- Tablet (768-1024px): Canvas with reduced zoom (0.75), smaller card sizing

### CC-3: "Save" Language

Find and replace across all specs:
- "Create free account" → "Save with email"
- "Create an account" → "Save them with just your email"
- "Lock in" → "Join as a founding member" / "I'm in"
- "Sign up" → "Save"
- Any CTA that frames capture as commitment rather than preservation

### CC-4: Color Is Never the Only Signal

Every place color alone differentiates states needs a secondary indicator (icon, text label, pattern). See per-spec sections for specific components.

### CC-5: Performance Targets

Add a "Performance Targets" section to each spec with relevant subset:

| Target | Value |
|--------|-------|
| First card visible | <3s |
| Canvas drag | 60fps / <16ms |
| Page load LCP | <2s |
| SSE connection | <500ms |
| Card expand transition | <200ms |
| Overlay open | <300ms |
| List render (100 items) | <500ms |
| Component mount | <16ms |

### CC-6: Score Demotion

"Show the translation, not the score. Never lead with a number."
- job-search: Fit score badge → move to secondary position, smaller, below vocab mappings
- resume-optimization: Score delta → move to right side, 14px, secondary text
- recruiter-canvas: Alignment score → keep but ensure translations are visually dominant

### CC-7: Minimum Font Size

All label text minimum 12px (ideally 13px). For muted text on dark backgrounds, use `--text-secondary` (#A1A1AA) instead of `--text-muted` (#71717A — fails WCAG AA on dark).

### CC-8: Missing States

Every component must handle: loading, empty, error, populated, overflow.
- Loading: skeleton rows, not flash of empty state
- Error: every computation component needs an error state
- Empty: new-user empty states for all list/dashboard views

### CC-9: Error Response Shape (Global)

```ts
// ALL specs use WRONG shape:
{ "error": "...", "message": "..." }

// RIGHT (backend contract):
{ "detail": "...", "error_code": "NOT_FOUND" | "AUTH_FORBIDDEN" | ... }
```

Update all error parsing everywhere.

---

## Per-Spec Fixes

### Spec 1: Admin Panel (`admin-panel.md`)

| ID | Severity | Fix |
|----|----------|-----|
| A1 | field | `role: "user"` → `role: "candidate"` everywhere |
| A2 | field | `per_page` → `page_size` query param |
| A3 | field | `AdminUser.name` → `full_name` (from `UserSummary`) |
| A4 | type | Align `AdminUser` to `UserSummary` shape (see WIRE_CONTRACTS.md). Remove `plan`, `founding_member` — don't exist on backend |
| A5 | endpoint | `GET /v1/admin/stats` → `GET /v1/admin/dashboard` (different response shape: `DashboardStats`) |
| A6 | endpoint | `POST /v1/admin/founding-codes` + GET + apply + pool → **DO NOT EXIST**. Block chunks 6-8, file RFC |
| A7 | endpoint | `GET /v1/admin/activity` → does not exist. Derive from user list or drop section |
| A8 | param | Backend `GET /v1/admin/users` does NOT support `sort`/`order`. Only: `page`, `page_size`, `status`, `search`. Remove sort dropdown or implement client-side sort |
| A9 | addition | Backend supports `DELETE /v1/admin/users/{id}` (soft delete) + `PUT /v1/admin/users/{id}/status` — consider adding to UserActions |
| A10 | enum | `account_status` values: `"pending_review" \| "approved" \| "suspended" \| "waitlisted"` |

### Spec 2: Application Tracking (`application-tracking-ui.md`)

| ID | Severity | Fix |
|----|----------|-----|
| AT1 | enum | Status: remove `accepted`, `withdrawn` → `withdrew`, add `ghosted`. Full enum: `"applied" \| "screening" \| "interviewing" \| "offer" \| "rejected" \| "ghosted" \| "withdrew"` |
| AT2 | field | `company` → `company_name` (nullable), `title` → `job_title` (nullable), `notes` → `user_notes` (nullable) |
| AT3 | type | Align `WireApplication` to `ApplicationResponse` (see WIRE_CONTRACTS.md). `job_id` is NULLABLE. |
| AT4 | field | `status_history` doesn't exist — derive timeline from `status` + `applied_at` + `updated_at`, or file RFC |
| AT5 | field | `optimization_score` doesn't exist anywhere |
| AT6 | field | `vocabulary_mappings_used` — Spec 27 dependency, not built. OutcomeInsight must degrade gracefully |
| AT7 | response | Response is paginated: `{ applications: [...], next_cursor: string \| null }`. Not raw array. Implement cursor pagination. |
| AT8 | http | `POST /v1/applications` returns **201**, not 200 |
| AT9 | decision | **Spencer decision: app tracking unlimited all tiers.** Remove any tier-based limits on application count. |
| AT10 | endpoint | `POST /v1/applications/connect-ats` doesn't exist — move to out-of-scope |
| AT11 | field | Per-transition notes not supported — backend has single `user_notes` field |

### Spec 3: Auth & Capture (`auth-capture.md`)

| ID | Severity | Fix |
|----|----------|-----|
| AC1 | CRITICAL | Claim request: remove `session_id` from body (4 locations). Only `{ claim_token }` + `Authorization: Bearer <jwt>` |
| AC2 | response | Claim response: remove `patterns_transferred`. Shape: `{ session_id, messages_transferred }`. Update success message. |
| AC3 | endpoint | Resume upload: `POST /v1/resumes/upload` → `POST /v1/resumes` (multipart). Required fields: `file`, `template_name`, `job_focus`. Response: `ResumeTemplateResponse` (201). |
| AC4 | field | `interactionCount` doesn't exist on StreamContext. Use `useRef<number>` locally or add to StreamState. |
| AC5 | field | `requiredRole` checks `user.is_recruiter` — field doesn't exist. Role lives on `GET /v1/me` → `role`. Defer for MVP (candidate-only). |
| AC6 | aria | SavePrompt: `aria-modal="false"` is correct (non-blocking). Add note explaining why. Still needs `aria-labelledby` + focus management. |

### Spec 4: Canvas Engine (`canvas-engine.md`)

| ID | Severity | Fix |
|----|----------|-----|
| CE1 | CRITICAL | `GET /v1/canvas` and `PUT /v1/canvas` do NOT exist (Spec 26). Block canvas persistence on Spec 26 contract. |
| CE2 | type | Consolidate `CanvasPersistedState` and `BackendCanvasState` into one shape. Recommend `BackendCanvasState`. |
| CE3 | enum | Card type naming: match backend `"translation_card" \| "gap_card" \| "agent_commentary" \| "resume_diff" \| "download_link"`. Add `"application_confirmation"` + `"application"` as stubs. |
| CE4 | type | `WireIdentityNode` — invented, no contract. Use `UserProfileResponse` from `GET /v1/me`. Block full IdentityNode on Spec 23E. |
| CE5 | type | `UserProfile` shadows contract. Map from `UserProfileResponse` (`full_name`, `current_job_title`, `current_company`, `career_stage`). |
| CE6 | field | `ChatMessage.role: "agent"` → `"assistant"`. Add `"system"` to union. |
| CE7 | field | `PlanTier` — no backend source. Hardcode "Free" for MVP or block on Spec 25 subscription contract. |
| CE8 | endpoint | `PUT /v1/me/discoverability` doesn't exist. File RFC or add to `PUT /v1/me`. |
| CE9 | layout | AppHeader layout inverted from PRD. PRD: Left = user name + plan badge + context. Right = admin + logout. |
| CE10 | session | No `session_id` management for authenticated canvas chat. Add: check `GET /v1/agent/sessions` on mount, use active or create on first message. |
| CE11 | redirect | Auth expired redirect goes to `/translate` (anonymous) — should go to re-auth flow. |
| CE12 | cross-ref | ReattributionLine: reference/import from vocabulary-translation-ui, don't redefine. |

### Spec 5: Job Search (`job-search-ui.md`)

| ID | Severity | Fix |
|----|----------|-----|
| JS1 | enum | Interaction types: `"save" → "saved"`, `"pass" → "passed"`, remove `"unsave"`. Add `"viewed"`, `"clicked_apply"`. |
| JS2 | field | Pass feedback: flatten `{ feedback: { reason } }` → `{ pass_reason: "wrong_level" }` on `InteractionRequest`. Drop `customText`. |
| JS3 | field | Optimize CTA: `target_job_id` → `job_id` |
| JS4 | type | `SSEJobMatchCard` does NOT exist in contracts — mark ALL wire types as "PROPOSED." File RFC. Key mismatches vs `FeedJobCard`: `company` (object not string), `title` → `job_title`, `remote` → `is_remote`, `fit_score` → `overall_fit_score`, salary is flat not nested, `posted_at` → `date_posted`. Add `job_id`, `url`, `is_hybrid`. |
| JS5 | type | Unify `JobCardData` (canvas-engine) and `JobMatchDisplay` (this spec). One type with `type: "job"` discriminator. |
| JS6 | component | `JobSearchEmpty` declared as server component but has `onQuerySelect` callback — must be client component. |
| JS7 | CC-4 | Strength dots: add icons (filled/half/hollow) + text labels. Fit score badge: add `aria-label`. |
| JS8 | CC-6 | Fit score → secondary position, below vocab mappings. |

### Spec 6: Recruiter Canvas (`recruiter-canvas-ui.md`)

| ID | Severity | Fix |
|----|----------|-----|
| RC1 | CRITICAL | ALL 8 recruiter endpoints are unpublished (Spec 28). All wire types are PROPOSED. Mark them as such. Build with mock fixtures. Integration blocked on Spec 28. |
| RC2 | role | No "recruiter" role exists — `UserRoleUpdate.role` is `"candidate" \| "admin"` only. File RFC. |
| RC3 | field | `PUT /v1/me` has no `perspective` field. File RFC. |
| RC4 | endpoint | Recruiter chat endpoint unresolved — separate vs shared. File RFC for Bryce to decide. |
| RC5 | field | Analyze JD request body: spec says `{ content }`, likely `{ jd_text }`. Define in RFC. |
| RC6 | path | Standardize: `POST /v1/recruiter/jd/{jd_id}/optimize` (not `/jd/optimize`). Standardize cohort pluralization. |
| RC7 | response | Fix response shape undefined. Need `updated_alignment_score` for optimistic UI. |
| RC8 | enum | `card_type: "jd_center"` not in union — add or document JDCenterNode populates from GET. |
| RC9 | field | `CohortResults` missing `cohortId` for feedback path. |
| RC10 | security | Add section: all endpoints enforce RLS via JWT, FE never passes IDs, 404 not 403 for unauthorized. |
| RC11 | decision | **Spencer decision: anon recruiter path IN SCOPE.** Add anonymous JD analysis flow (pre-auth). May need spec addendum. |
| RC12 | CC-4 | RepellentCard severity: add text labels or icon variation alongside color. |

### Spec 7: Resume Optimization (`resume-optimization-ui.md`)

| ID | Severity | Fix |
|----|----------|-----|
| RO1 | CRITICAL | URL params: `{tid}` → `{template_id}`, `{vid}` → `{version_id}` everywhere. |
| RO2 | CRITICAL | Optimize returns **JSON**, not SSE. `POST .../optimize` → `OptimizationResult { version, deltas[], case_studies_used, was_cached }`. Rewrite data flow to consume synchronously. |
| RO3 | CRITICAL | Chat returns **JSON**, not SSE. `POST .../chat` → `ChatResponse { session_id, message, turn_count, max_turns, session_closed }`. Rewrite iteration flow. |
| RO4 | type | `OptimizationDeltaResponse` fields: `section_name, original_text, optimized_text, reasoning, jd_keyword_targeted, confidence`. Spec's `WireDiffSection`/`WireDiffChange` types are completely wrong — `mapping`, `translation_applied`, `evidence_source`, `change_type` don't exist. |
| RO5 | field | Application recording: `applied_at` must be date-only (`"2026-03-12"`). Add missing `application_source: "soaria_direct"`. |
| RO6 | endpoint | `POST .../approve` doesn't exist — implement client-side approval or file RFC. |
| RO7 | error | Add 502 (LLM failed, retryable) and 503 (budget exceeded, not retryable) to error paths. |
| RO8 | field | Score delta has no backend source — file RFC for post-opt score. |
| RO9 | CC-4 | Diff change types: add inline prefix markers (+, ~, ↕) alongside color. |
| RO10 | CC-6 | Score delta → move to right side, 14px, secondary text. |

### Spec 8: Star Stories (`star-stories-ui.md`)

| ID | Severity | Fix |
|----|----------|-----|
| SS1 | CRITICAL | `SSEStarStoryCard` is invented — doesn't match `CaseStudyResponse`. Reconcile fields (see correction file for mapping table). Mark wire types as `/** PENDING RFC */`. |
| SS2 | enum | Card type: SSE may emit `"case_study"` not `"star_story"`. Add `"story_prompt"` to canvas `CardType` union. Document mapping. |
| SS3 | type | `CaseStudyData` silently redefined from canvas-engine — changes should live in canvas-engine as revision. `vocabTags` vs `vocabularyTags` — pick one. |
| SS4 | field | STAR section fields must be `string \| null` during progressive capture, not required `string`. Mapper defaults null → `""`. |
| SS5 | decision | **Spencer decision: STAR capture UNGATED, all tiers.** Remove ALL Pro-only gating. No `useFeatureGate("star_story_capture")`, no `GatedFeaturePrompt`. This is core experience. |
| SS6 | edge | StoryConnectors are edges — note in canvas-engine that "no edges" claim is now false. |
| SS7 | field | Reverse lookup (TranslationCard → StoryCards) unspecced — add `Map<string, string[]>` to CanvasContext. |
| SS8 | field | `mapStoryCapturePrompt` drops `translation_card_id` — add `translationCardId` to props. |
| SS9 | CC-4 | GapCard importance dots: add text labels + distinct icons. |

### Spec 9: Subscription & Trial (`subscription-trial-ui.md`)

| ID | Severity | Fix |
|----|----------|-----|
| ST1 | CRITICAL | Tier enum: `"free" \| "pro"` → `"free" \| "trial" \| "pro" \| "founding_member"`. Handle all 4. |
| ST2 | CRITICAL | Status enum: `"trialing" \| "active" \| "cancelled" \| "free"` → `"free" \| "trialing" \| "pro" \| "founding_member" \| "past_due" \| "cancelled"`. `"active"` does NOT exist. |
| ST3 | CRITICAL | Unlimited sentinel: `-1` → `null`. `optimizations_limit: number \| null`, `applications_limit: number \| null`. |
| ST4 | type | `SubscriptionSummary` not in published contracts — file RFC. Mark FE types as provisional. |
| ST5 | endpoint | Data source: `GET /v1/me` vs `GET /v1/billing/subscription` — pick one and document. |
| ST6 | field | Use backend flags `is_pro` and `show_pre_commitment` instead of re-implementing logic client-side. |
| ST7 | endpoint | PreCommitmentPrompt: `POST /v1/billing/checkout` → `POST /v1/billing/pre-commit` (applies 90-day coupon). Add `createPreCommitSession()`. |
| ST8 | redirect | `/settings?billing=success` → `/?billing=success` (no `/settings` route). Add query param handler to canvas shell. |
| ST9 | content | Day 12 message: remove `optimizationsUsed` reference (always -1). Use applications + proactive insight count. |
| ST10 | content | 90-day founding member free period absent from UI. Add to PreCommitmentPrompt + UpgradeModal. |
| ST11 | content | "Lock in" → "Join as a founding member" (agency framing). |
| ST12 | decision | **Spencer decision: STAR stories ungated, app tracking unlimited.** Update feature gating table accordingly. |
| ST13 | field | Add `founding_eligible` boolean for UpgradeModal pricing. |
| ST14 | endpoint | `GET /v1/feed/preview-count` doesn't exist. File RFC. Stub to return 0. |
| ST15 | content | Add `"past_due"` handling → PaymentIssueBanner with Stripe portal link. |

### Spec 10: Vocabulary Translation (`vocabulary-translation-ui.md`)

| ID | Severity | Fix |
|----|----------|-----|
| VT1 | endpoint | Card interaction: `POST /v1/agent/cards/{card_id}/interact` → `POST /v1/agent/sessions/{sessionId}/cards/{cardId}/interact`. Session-scoped. Pull sessionId from `versed_anon_session` localStorage. |
| VT2 | field | `card_state` and `outcome_validated` on `SSETranslationCard` — NOT in published contract. Use `inferCardState` from `confidence_score` only (Option B). Remove from FE type. Resolve internal contradiction (line 364 vs 466). |
| VT3 | CRITICAL | `card_id` missing from `SSECardEvent`. File RFC to add it — without it, FE can't reference cards back to backend. |
| VT4 | type | Interaction request shape mismatch: `"claim"` → `"submit_input"`, `updatedText` → `user_input`, add `card_type`. Add `mapCardInteraction()` to card-mappers.ts. |
| VT5 | field | Paste response: resolve `text` vs `content` field name. Recommend `content` (matches `PasteStoreRequest`). |
| VT6 | event | `voice` and `text` SSE events defined but never consumed. Render `text` events as text bubble (PRD requires). Log+skip `voice` for MVP. |
| VT7 | event | `tool_status` events have no render target — add status text element during `analyzing` state. |
| VT8 | guard | `agent_commentary` card type hits `addCard` without type guard — check `card_type` before adding. |
| VT9 | ttl | Stale localStorage session: add TTL check (72h) on page mount. |
| VT10 | boundary | `streamAnalysis` handles ONLY SSE (step 2). Add `fetchPasteContent()` helper for paste fetch (step 1). |
| VT11 | fallback | AgentTease: if `done` fires without `tease` but card count >4, render tease anyway. |
| VT12 | component | **Spencer decision: this spec owns ReattributionLine component.** Primary home after full card reveal, before capture prompt. |

---

## RFCs Required (Compile Before Dispatching)

Faye should note RFC needs in revised specs but NOT block revision on them. Mark affected sections with `<!-- RFC NEEDED: [description] -->`.

| RFC | Spec(s) | What |
|-----|---------|------|
| `card_id` on SSECardEvent | VT, CE, JS, SS | Without it, no card interactions possible |
| SSEJobMatchCard contract | JS | Proposed type needs Bryce to publish |
| SSEStarStoryCard contract | SS | Proposed type needs Bryce to publish |
| Canvas persistence endpoints | CE | `GET/PUT /v1/canvas` — Spec 26 |
| Recruiter endpoints (all 8) | RC | Spec 28 — entire recruiter backend |
| SubscriptionSummary publication | ST | Type not in published contracts |
| Founding codes endpoints | A | `POST/GET /v1/admin/founding-codes` |
| Status audit trail | AT | `status_history` for application timeline |
| Feed preview count | ST | `GET /v1/feed/preview-count` |

---

## Revision Checklist (Per Spec)

When revising each spec, Faye should:

1. Apply all per-spec fixes from the table above
2. Apply all relevant cross-cutting fixes (CC-1 through CC-9)
3. Align all wire types against `WIRE_CONTRACTS.md`
4. Apply Spencer decisions where they override foil findings
5. Mark RFC-dependent sections with `<!-- RFC NEEDED -->` comments
6. Mark PROPOSED wire types that lack backend contracts
7. Add "Mobile Behavior" section
8. Add "Performance Targets" section
9. Verify all error handling uses `{ detail, error_code }` shape
