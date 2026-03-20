# Foil Review Fixes — Specs 23-25, Token Alignment, Marketing Site

**Status:** Revision 1 — addressing 11 items from foil review round 1
**Date:** 2026-03-12 (Rev 1: 2026-03-12)
**Reviewers:** Nolan (BE foil), Sloane (FE foil)
**Process:** All 5 specs received REVISE verdicts. Round 1 review found 3 contract conflicts + 8 additional items. This revision addresses all 11. Pending final APPROVE/REVISE.

**Scope:** 41 issues total — 10 critical blockers, 6 cross-boundary integration gaps, 14 important non-blocking, 5 frontend marketing, 5 missing specs, 1 lessons section.

---

## Priority Tier 1: Critical Blockers (fix before any code ships)

### Backend — Anonymous Flow (Issues 1-4)

---

#### Issue 1: PlannerInput.agent_state missing 'anonymous'
**Spec:** 23 (Anonymous Sessions)
**Problem:** Migration adds `anonymous` to DB constraint, but `PlannerInput.agent_state` is a `Literal` type that doesn't include it. Every anonymous request crashes with Pydantic validation error.

**Proposed Fix (BE):**
```python
# models/agent.py (or wherever PlannerInput lives)
class PlannerInput(BaseModel):
    agent_state: Literal[
        "anonymous",
        "onboarding",
        "searching",
        "optimizing",
        "applying",
        "reviewing",
        "returning",
    ]
```

**FE impact:** None. Frontend doesn't send `agent_state` — it's internal to the planner pipeline.

---

#### Issue 2: ReframingPatternCreate.user_id required (not nullable), no session_id field
**Spec:** 23 (Anonymous Sessions)
**Problem:** Anonymous patterns use `session_id` instead of `user_id`, but the Pydantic model still has `user_id` as required with no `session_id` field. Zero patterns stored for anonymous sessions.

**Proposed Fix (BE):**
```python
class ReframingPatternCreate(BaseModel):
    user_id: UUID | None = None          # nullable for anonymous
    session_id: UUID | None = None       # new field for anonymous sessions
    pattern_type: str
    original_text: str
    reframed_text: str
    context: str | None = None

    @model_validator(mode="after")
    def require_owner(self) -> Self:
        if self.user_id is None and self.session_id is None:
            raise ValueError("Either user_id or session_id is required")
        return self
```

**FE impact:** None directly. Claim flow (Issue 16) transfers these to the authenticated user.

---

#### Issue 3: Rate limiting uses request.client.host behind Railway load balancer
**Spec:** 23 (Anonymous Sessions)
**Problem:** Railway LB forwards through a small set of IPs. `request.client.host` as rate limit key means all anonymous users share one 5 req/hr bucket.

**Proposed Fix (BE):**
```python
def get_client_ip(request: Request) -> str:
    """Extract real client IP, respecting trusted proxy headers."""
    forwarded_for = request.headers.get("X-Forwarded-For")
    if forwarded_for:
        # First IP in chain is the client; validate it's a plausible IP
        client_ip = forwarded_for.split(",")[0].strip()
        try:
            ipaddress.ip_address(client_ip)
            return client_ip
        except ValueError:
            pass
    return request.client.host if request.client else "unknown"
```

Use `get_client_ip(request)` as the rate limit key instead of `request.client.host`.

**FE impact:** None. Rate limiting is transparent to the frontend (until 429 — see Issue 40).

---

#### Issue 4: POST /v1/agent/analyze pipeline integration unspecified
**Spec:** 23 (Anonymous Sessions)
**Problem:** How `/analyze` connects to the planner→dispatcher→assembler pipeline is undefined. Risk of copy-pasting ~370 lines of SSE setup.

**Proposed Fix (BE):**
Extract shared pipeline logic into a reusable function:

```python
# services/agent_pipeline.py
async def run_agent_pipeline(
    *,
    message: str,
    session: AgentSession,
    user_id: UUID | None,
    agent_state: str,
    stream: bool = True,
) -> AsyncGenerator[ServerSentEvent, None]:
    """Shared pipeline entry point for both authenticated and anonymous flows.

    Handles: planner dispatch, tool execution, assembler, SSE streaming.
    Authenticated flow: passes user_id, loads memory/history.
    Anonymous flow: passes session_id only, skips memory, limited tools.
    """
    ...
```

Both `POST /v1/agent/chat` and `POST /v1/agent/analyze` call this function. The anonymous path restricts available tools (no `save_job`, no `optimize_resume`), skips memory injection, and uses `agent_state="anonymous"`.

**Spec addendum:** Add a section to Spec 23 defining which tools are available in anonymous mode vs. authenticated mode. Minimum anonymous tools: `analyze_vocabulary`, `search_jobs` (limited to 3 results).

**FE impact:** None structurally — SSE event format is identical. Frontend consumes the same stream regardless of auth state.

---

### Backend — Agent Alignment (Issues 5-7)

---

#### Issue 5: card_interactions.card_id has no referential integrity
**Spec:** 24 (Agent Alignment)
**Problem:** `card_id` is a UUID with no FK and no validation. Data flywheel built on unvalidated data.

**Proposed Fix (BE):**
FK to a cards table is impractical because cards are ephemeral SSE payloads, not persisted rows (yet). Instead:

1. Add validation in the card interaction endpoint — verify `card_id` exists in the session's card history:
```python
async def create_card_interaction(
    self, interaction: CardInteractionRequest, session_id: UUID
) -> CardInteraction:
    # Verify card_id was actually emitted in this session
    session = await self.get_session(session_id)
    emitted_card_ids = {msg.card_id for msg in session.messages if msg.card_id}
    if interaction.card_id not in emitted_card_ids:
        raise ValueError(f"card_id {interaction.card_id} not found in session {session_id}")
    ...
```

2. Add a CHECK constraint or application-level validation that `card_id` format is a valid UUIDv4.

**FE impact:** Frontend already sends `card_id` from rendered cards. No change needed — just ensures the ID came from a real card.

---

#### Issue 6: No error handling for malformed LLM output from analyze_vocabulary
**Spec:** 24 (Agent Alignment)
**Problem:** If the LLM returns malformed JSON from `analyze_vocabulary`, core value delivery crashes.

**Proposed Fix (BE):**
```python
async def analyze_vocabulary(self, input: AnalyzeVocabularyInput) -> ToolResult:
    raw_output = await self.llm.generate(...)
    try:
        parsed = VocabularyAnalysisOutput.model_validate_json(raw_output)
    except (ValidationError, JSONDecodeError) as e:
        logger.warning(f"Malformed LLM output in analyze_vocabulary: {e}")
        # Retry once with stricter prompt
        raw_output = await self.llm.generate(..., retry_prompt=STRICT_JSON_PROMPT)
        try:
            parsed = VocabularyAnalysisOutput.model_validate_json(raw_output)
        except (ValidationError, JSONDecodeError):
            # Graceful degradation: return commentary card instead of crash
            return ToolResult(
                card_type="agent_commentary",
                data={"text": "I found some vocabulary patterns but had trouble formatting them. Let me try a different approach — could you tell me more about the specific role you're looking at?"},
                error=False,
            )
    return ToolResult(card_type="translation_card", data=parsed.model_dump())
```

**FE impact:** Frontend may receive an `agent_commentary` card where a `translation_card` was expected. This is fine — both card types are already renderable. No code change needed.

---

#### Issue 7: Anonymous card interaction auth undefined **[DECIDED — cookie approach]**
**Spec:** 24 (Agent Alignment)
**Problem:** Spec says interactions require "authentication OR valid anonymous claim_token" but defines no mechanism. No `claim_token` on `CardInteractionRequest`, no header, no middleware.

**Decision:** Cookie-based auth. Spencer signed off.

**Proposed Fix (BE):**
Anonymous card interactions use the `anonymous_session_id` cookie (set when `/analyze` creates the session):

```python
# In card interaction endpoint
async def record_interaction(
    request: Request,
    interaction: CardInteractionRequest,
    user: User | None = Depends(optional_current_user),  # None if no JWT
):
    if user:
        interaction.user_id = user.id
    else:
        session_id = request.cookies.get("anonymous_session_id")
        if not session_id:
            raise HTTPException(401, "Authentication or anonymous session required")
        # Validate session exists and is active
        session = await agent_service.get_anonymous_session(UUID(session_id))
        if not session:
            raise HTTPException(401, "Invalid or expired anonymous session")
        interaction.session_id = session.id
        interaction.user_id = None
```

**Contract (both sides):**

| Side | Detail |
|------|--------|
| BE | `POST /v1/agent/analyze` sets `Set-Cookie: anonymous_session_id=<uuid>; HttpOnly; Secure; SameSite=Lax; Max-Age=259200; Path=/` |
| FE | Cookie is automatic on same-origin. No explicit FE handling needed. |

**Cross-origin safety:** No cross-origin cookie issue exists. The paste relay (Issue 11) redirects the user's browser to `app.getversed.ai/translate`. The `/translate` page then calls `POST /v1/agent/analyze` — this request originates from the app domain, and the cookie is set on the app domain's backend. All subsequent card interaction requests are same-origin (app→backend). The marketing site never needs the cookie.

**FE:** No `claim_token` in `CardInteractionRequest`. Cookie is sent automatically by the browser. No code needed.

---

### Backend — Reverse Trial (Issues 8-10)

---

#### Issue 8: Race condition on first resume upload (subscription creation)
**Spec:** 25 (Reverse Trial)
**Problem:** Double-click on upload → two concurrent INSERTs → UNIQUE violation → 500.

**Proposed Fix (BE):**
```sql
-- Use INSERT ... ON CONFLICT instead of check-then-insert
INSERT INTO subscriptions (user_id, status, tier, founding_member, created_at)
VALUES ($1, 'trial', 'free', TRUE, NOW())
ON CONFLICT (user_id) DO NOTHING
RETURNING *;
```

If `DO NOTHING` fires (row already exists), fetch the existing subscription. Application code:

```python
async def ensure_subscription(self, user_id: UUID) -> Subscription:
    result = await self.db.execute(UPSERT_SUBSCRIPTION_SQL, user_id)
    if result:
        return Subscription.model_validate(result)
    # ON CONFLICT DO NOTHING fired — row exists
    return await self.get_subscription(user_id)
```

**FE impact:** None. Frontend just calls the upload endpoint. The 500 goes away.

---

#### Issue 9: usage_tracking.period_start calculation undefined
**Spec:** 25 (Reverse Trial)
**Problem:** `period_start DATE NOT NULL` with comment "First of the month" but no code. Timezone undefined. Free-tier limits reset inconsistently or never.

**Proposed Fix (BE):**
```python
# services/subscription.py
def current_period_start() -> date:
    """Period start is first of the current month in UTC."""
    return date.today().replace(day=1)  # UTC — server runs in UTC on Railway

async def get_or_create_usage(self, user_id: UUID, period_start: date | None = None) -> UsageTracking:
    period = period_start or current_period_start()
    result = await self.db.execute(
        """
        INSERT INTO usage_tracking (user_id, period_start, optimizations_used, searches_used)
        VALUES ($1, $2, 0, 0)
        ON CONFLICT (user_id, period_start) DO NOTHING
        RETURNING *
        """,
        user_id, period,
    )
    if result:
        return UsageTracking.model_validate(result)
    return await self.db.fetch_one(
        "SELECT * FROM usage_tracking WHERE user_id = $1 AND period_start = $2",
        user_id, period,
    )
```

Add to Spec 25:
- Period is always UTC month boundary (`date.today().replace(day=1)` where server time is UTC)
- Document this in the `usage_tracking` table comment
- Add `UNIQUE (user_id, period_start)` constraint to the migration

**FE impact:** Frontend receives `period_resets_at` in `SubscriptionSummary` (see Issue 24). Display as "Resets [date]" in usage UI.

---

#### Issue 10: Trial-to-Pro race condition
**Spec:** 25 (Reverse Trial)
**Problem:** Day 14 lifecycle flow sets `status='free'`. If Stripe webhook fires simultaneously, paying user gets downgraded.

**Proposed Fix (BE):**
The lifecycle flow must check for active Stripe subscription before downgrading:

```python
async def process_trial_expiry(self, user_id: UUID) -> None:
    sub = await self.get_subscription(user_id)
    if sub.stripe_subscription_id:
        # User has a Stripe subscription — check its status
        stripe_sub = await stripe.Subscription.retrieve(sub.stripe_subscription_id)
        if stripe_sub.status in ("active", "trialing"):
            # Stripe says they're paying. Don't downgrade.
            logger.info(f"Skipping trial expiry for {user_id}: active Stripe subscription")
            return
    # No active Stripe subscription — downgrade
    await self.update_subscription(user_id, status="free", tier="free")
```

Additionally, add optimistic locking with a `version` column or use `UPDATE ... WHERE status = 'trial'` to prevent the downgrade if the status already changed:

```sql
UPDATE subscriptions
SET status = 'free', tier = 'free', updated_at = NOW()
WHERE user_id = $1 AND status = 'trial'
RETURNING *;
```

If no rows returned, subscription was already changed (by webhook) — no-op.

**FE impact:** None. Subscription state is read-only from the frontend perspective.

---

## Priority Tier 2: Cross-Boundary Integration Gaps (Issues 11-16)

---

#### Issue 11: sessionStorage cross-origin handoff is broken
**Spec:** Marketing Site
**Problem:** `sessionStorage` is origin-scoped. `getversed.ai` can't share storage with `app.getversed.ai`. Primary conversion mechanism is dead on arrival.

**Proposed Fix:** Adopt Nolan's paste relay recommendation.

**BE contract:**
```
POST /v1/paste
  Auth: None (rate-limited by IP, 20/hr)
  Body: { "text": string }  // max 50KB
  Response: { "token": string }  // UUID (gen_random_uuid()), expires in 5 min
  Rate limit: 20 req/hr per IP
  CORS: Allow-Origin includes https://getversed.ai, https://www.getversed.ai
  CORS: Allow-Headers includes Content-Type (required for preflight)

GET /v1/paste/{token}
  Auth: None
  Response: { "text": string }
  Behavior: One-time read — token deleted after fetch. Also expires after 5 min if unfetched.
  404 if token expired or already consumed.
```

**Contract types (add to contracts/types.ts):**
```typescript
// === Paste Relay ===

export interface PasteStoreRequest {
  content: string;  // max 50KB — field name matches Bryce's PasteRequest.content
}

export interface PasteStoreResponse {
  token: string;  // UUID (gen_random_uuid()), not nanoid
}

export interface PasteRetrieveResponse {
  content: string;  // matches Bryce's response model
}
```

**BE implementation notes:**
- Store in Redis (or Supabase table with TTL cleanup) — not in-memory (Railway may restart)
- Token is a UUID generated by the database (`gen_random_uuid()`)
- CORS: `Access-Control-Allow-Origin` must include both `https://getversed.ai` and `https://www.getversed.ai`
- CORS: `Access-Control-Allow-Headers` must include `Content-Type` (POST sends JSON, triggering preflight)
- CORS: `OPTIONS /v1/paste` must return preflight response

**FE constant (add to lib/constants.ts):**
```typescript
export const API_URL = process.env.NEXT_PUBLIC_API_URL ?? "https://api.getversed.ai";
```

**FE contract (marketing site — PasteInput.tsx):**
```typescript
// On paste event:
async function handlePaste(text: string) {
  setStatus("processing");
  try {
    const res = await fetch(`${API_URL}/v1/paste`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ content: text.slice(0, 50_000) }),
    });
    if (res.status === 429) {
      const { retry_after_seconds } = await res.json();
      setRetryAfter(retry_after_seconds);
      setStatus("rate_limited");
      return;
    }
    if (!res.ok) throw new Error("Failed to store paste");
    const { token } = await res.json();
    window.location.href = `${APP_URL}/translate?paste_token=${token}`;
  } catch {
    setStatus("error");
    // Show: "Something went wrong. Try pasting directly in the app."
    // With link to APP_URL
  }
}
```

**FE contract (app — /translate page):**
```typescript
// On mount:
const token = searchParams.get("paste_token");
if (token) {
  const res = await fetch(`${API_URL}/v1/paste/${token}`);
  if (res.ok) {
    const { content } = await res.json();
    // Bootstrap anonymous session with this text
    await startAnonymousSession(content);
  } else {
    // Token expired or invalid — show empty state with paste prompt
  }
}
```

**Spec correction:** Marketing site spec line 21 — replace sessionStorage mechanism with paste relay.

---

#### Issue 12: No claim_token delivery mechanism
**Specs:** 23 + FE
**Problem:** Backend generates `claim_token` on anonymous session creation. No mechanism delivers it to the frontend.

**Proposed Fix:**

**Delivery mechanism:** SSE `session_init` event is the **sole** delivery mechanism. No response headers — `EventSource` API doesn't expose them, and even with `fetch()` + `ReadableStream`, relying on headers is fragile.

**BE:** `session_init` is always the **first** SSE event emitted by `/analyze`:

```
event: session_init
data: {"session_id": "<uuid>", "claim_token": "<token>"}
```

This is a new SSE event type. Add to the event taxonomy alongside existing types (`tool_status`, `card`, `text`, `error`, `done`).

**FE:** SSE consumed via `fetch()` + `ReadableStream` (not `EventSource`, which lacks headers and fine-grained control). On receiving `session_init`, store in `localStorage`:

```typescript
// In SSE stream reader:
if (event.type === "session_init") {
  const { session_id, claim_token } = JSON.parse(event.data);
  localStorage.setItem(
    "versed_anon_session",
    JSON.stringify({ sessionId: session_id, claimToken: claim_token, createdAt: new Date().toISOString() })
  );
}
```

**localStorage key:** `versed_anon_session` (not `versed_claim_token`). Stores structured object, not bare token.

On signup, read and pass to claim endpoint (Issue 13).

---

#### Issue 13: No frontend contract for claim flow
**Specs:** 23 + FE
**Problem:** Backend has `POST /v1/agent/sessions/claim`. Frontend has no spec for storage, passing, refresh recovery, or expiry handling.

**Proposed Fix:**

**Frontend claim flow spec:**

| Step | Action | Detail |
|------|--------|--------|
| 1 | Anonymous session starts | `session_init` SSE event → stored in `localStorage` key `versed_anon_session` as `{ sessionId, claimToken, createdAt }` |
| 2 | User signs up | After Supabase auth completes, read `versed_anon_session` from `localStorage` |
| 3 | Claim | `POST /v1/agent/sessions/claim` with `{ "claim_token": "<token>" }` and auth header |
| 4 | Success | Delete `versed_anon_session` from `localStorage`. Session history appears on canvas. |
| 5 | Failure (expired) | Delete `versed_anon_session`. Show: "Welcome! Your previous session expired, but we're ready to start fresh." |
| 6 | Tab refresh (pre-signup) | `localStorage` persists. Anonymous session cookie persists. User can continue interacting. |
| 7 | Token lost | If no `versed_anon_session` in `localStorage` on signup, skip claim. Fresh start. |

**BE contract:**
```
POST /v1/agent/sessions/claim
  Auth: Bearer <jwt> (required — must be authenticated)
  Body: { "claim_token": string }
  Success 200: { "session_id": "<uuid>", "messages_transferred": number }
  Error 404: Token not found or expired
  Error 409: Session already claimed
```

**TTL alignment with Issue 17:** `claim_token` validity must exceed cleanup TTL. See Issue 17.

---

#### Issue 14: SSE field naming mismatch
**Specs:** 24 + FE
**Problem:** Backend uses `source_phrase`/`target_phrase` (matches DB columns). Frontend uses `candidateText`/`recruiterText` (matches design system amber/purple semantics). Same concept, different names.

**Proposed Fix:** Keep both naming conventions where they belong. Add an explicit mapping layer on the frontend.

**BE:** No change. Backend keeps `source_phrase`/`target_phrase` — these are DB column names and changing them has migration impact.

**Contract types (add to contracts/types.ts):**

These types don't exist yet in contracts/types.ts. Bryce must publish them before FE can compile the mapper.

```typescript
// === SSE Card Payloads (from Spec 24) ===
// Backend field naming: source_phrase / target_phrase (matches DB columns)
// Frontend maps to: candidateText / recruiterText (design system semantics)

export interface SSETranslationCard {
  source_phrase: string;       // candidate's language
  target_phrase: string;       // recruiter's language
  category: string | null;     // e.g., "Engineering", "Marketing"
  confidence_score: number | null;
  context: string | null;
}

export interface SSEGapCard {
  target_phrase: string;       // recruiter term the candidate lacks
  importance: "required" | "preferred" | "nice_to_have";
  suggestion: string | null;   // how to address the gap
}

// === SSE Event Types ===
// Event taxonomy: session_init, tool_status, card, text, error, done

export interface SSESessionInitEvent {
  session_id: string;
  claim_token: string;
}

export interface SSECardEvent {
  card_type: "translation_card" | "gap_card" | "agent_commentary" | "resume_diff" | "download_link";
  data: SSETranslationCard | SSEGapCard | Record<string, unknown>;
}
```

**FE:** Add `src/lib/card-mappers.ts`:

```typescript
// src/lib/card-mappers.ts

import type { SSETranslationCard, SSEGapCard } from "@/types/api";
import type { TranslationCardProps } from "@/types/marketing";

/**
 * Maps backend SSE translation card data to frontend component props.
 * Backend uses source_phrase/target_phrase (DB schema naming).
 * Frontend uses candidateText/recruiterText (design system semantics).
 */
export function mapTranslationCard(sse: SSETranslationCard): TranslationCardProps {
  return {
    candidateText: sse.source_phrase,
    recruiterText: sse.target_phrase,
    category: sse.category ?? undefined,
  };
}

export function mapGapCard(sse: SSEGapCard) {
  return {
    recruiterTerm: sse.target_phrase,
    importance: sse.importance,
    suggestion: sse.suggestion ?? undefined,
  };
}
```

**Naming convention documentation:** Add comment to top of SSE types in `contracts/types.ts`:
```
// source_phrase → candidateText (amber panel, candidate's language)
// target_phrase → recruiterText (purple panel, recruiter's language)
```

---

#### Issue 15: versed_app_url defaults to marketing domain
**Spec:** 25 (Reverse Trial)
**Problem:** `config.py` has `versed_app_url: str = "https://getversed.ai"`. Stripe checkout redirects to marketing site instead of app.

**Proposed Fix (BE):**
```python
# config.py
versed_app_url: str = "https://app.getversed.ai"   # App, not marketing site
versed_marketing_url: str = "https://getversed.ai"  # Marketing site (if needed for emails)
```

Stripe checkout `success_url` and `cancel_url` use `versed_app_url` (the app where the user has a session). Marketing emails use `versed_marketing_url`.

**FE impact:** None directly — Stripe redirect goes to the right place.

---

#### Issue 16: card_interactions not included in claim transfer
**Specs:** 24 + 23
**Problem:** Claim transfer handles `agent_sessions` and `reframing_patterns` but not `card_interactions`. Post-signup, anonymous card interactions are orphaned.

**Proposed Fix (BE):**
Add `card_interactions` to the claim transfer function:

```python
async def claim_anonymous_session(
    self, claim_token: str, user_id: UUID
) -> ClaimResult:
    session = await self.validate_claim_token(claim_token)

    # Transfer session ownership
    await self.db.execute(
        "UPDATE agent_sessions SET user_id = $1 WHERE id = $2",
        user_id, session.id,
    )

    # Transfer reframing patterns
    await self.db.execute(
        "UPDATE reframing_patterns SET user_id = $1 WHERE session_id = $2 AND user_id IS NULL",
        user_id, session.id,
    )

    # Transfer card interactions (NEW)
    await self.db.execute(
        "UPDATE card_interactions SET user_id = $1 WHERE session_id = $2 AND user_id IS NULL",
        user_id, session.id,
    )

    # Invalidate claim token
    await self.invalidate_claim_token(claim_token)

    return ClaimResult(session_id=session.id, messages_transferred=len(session.messages))
```

**FE impact:** None. Claim is a single POST from the frontend. The backend handles all data transfer internally.

---

## Priority Tier 3: Important Non-Blocking (Issues 17-28)

### Backend (Issues 17-24)

---

#### Issue 17: Claim/cleanup TTL overlap at 24h
**Spec:** 23 (Anonymous Sessions)
**Problem:** Cleanup eligible after 24h inactivity. Claim tokens valid for 24h. Edge case: user returns at hour 23, session already cleaned up.

**Proposed Fix (BE):**
Stagger the TTLs:
- **Claim token validity:** 72 hours (3 days)
- **Session cleanup eligibility:** 7 days of inactivity
- **Cleanup job:** Runs daily, deletes sessions inactive for 7+ days

This gives users a generous window to return and sign up without the "save your results" promise being broken.

```python
# config.py
anonymous_claim_ttl_hours: int = 72
anonymous_cleanup_after_days: int = 7
```

**FE impact:** Frontend copy says "your results are saved" without specifying duration. No specific time commitment in UI copy — avoids future misalignment if TTLs change. The 72h/7d values are backend config, not user-facing.

---

#### Issue 18: No confirmation anonymous queries bypass RLS
**Spec:** 23 (Anonymous Sessions)
**Problem:** Anonymous requests have no JWT. Need to confirm service role usage is scoped.

**Proposed Fix (BE):**
Add to Spec 23 an explicit section:

> **RLS & Service Role Usage:**
> Anonymous endpoints (`/analyze`, `/paste`, `/paste/{token}`) use the service role for:
> - Creating anonymous sessions in `agent_sessions`
> - Storing/reading paste relay tokens (Redis or `paste_tokens` table)
> - Creating card interactions with `user_id = NULL`
>
> Service role queries MUST NOT touch: `users`, `resumes`, `subscriptions`, `applications`, or any table with user PII beyond the anonymous session itself.
>
> Enforcement: Each anonymous endpoint gets a dedicated repository method that only accesses the allowed tables. No shared `db.execute()` with raw SQL in the endpoint handler.

**FE impact:** None.

---

#### Issue 19: Progressive disclosure has no server-side enforcement
**Spec:** 24 (Agent Alignment)
**Problem:** Prompt says `max_translations=5` for Phase 1, but `AnalyzeVocabularyInput.max_translations` defaults to 10 with range 3-25.

**Proposed Fix (BE):**
Runtime cap in the tool handler, driven by config per subscription tier. Not a Pydantic constraint — a hard Pydantic `le=5` would reject legitimate values from Pro users in later phases.

```python
# In analyze_vocabulary tool handler:
tier_limits = {"anonymous": 3, "free": 5, "trialing": 5, "pro": 15}
effective_max = min(request.max_translations, tier_limits[subscription_tier])
```

Pydantic keeps a generous `le=25` for validation; the tool handler enforces the tier-specific cap.

**FE impact:** None. Frontend doesn't set `max_translations`.

---

#### Issue 20: Gap card importance classification has no prompt guidance
**Spec:** 24 (Agent Alignment)
**Problem:** `GapCard.importance` is `Literal["required", "preferred", "nice_to_have"]` but the LLM prompt doesn't define classification criteria.

**Proposed Fix (BE):**
Add to the `analyze_vocabulary` prompt:

```
Classify each gap's importance:
- "required": The job listing explicitly requires this skill/experience. Missing it likely means auto-rejection.
- "preferred": Listed as preferred/desired, or strongly implied by the role level. Competitive disadvantage if missing.
- "nice_to_have": Mentioned but clearly supplementary. Absence won't disqualify.

Base classification on the job description language: "must have" / "required" → required.
"Preferred" / "ideal" / "bonus" → preferred. Everything else → nice_to_have.
When uncertain, classify one level down (prefer false negatives over false positives).
```

**FE impact:** None. Frontend renders `importance` as-is — the display treatment (color, icon) is already keyed to these literal values.

---

#### Issue 21: Missing customer.subscription.updated webhook handler
**Spec:** 25 (Reverse Trial)
**Problem:** Handles `checkout.session.completed`, `invoice.paid`, `invoice.payment_failed`, `customer.subscription.deleted` but not `customer.subscription.updated`. Plan changes and billing cycle updates won't sync.

**Proposed Fix (BE):**
Add handler:

```python
async def handle_subscription_updated(self, event: stripe.Event) -> None:
    stripe_sub = event.data.object
    sub = await self.get_subscription_by_stripe_id(stripe_sub.id)
    if not sub:
        logger.warning(f"Subscription updated for unknown stripe_id: {stripe_sub.id}")
        return

    await self.update_subscription(
        sub.user_id,
        status=self._map_stripe_status(stripe_sub.status),
        current_period_start=date.fromtimestamp(stripe_sub.current_period_start),
        current_period_end=date.fromtimestamp(stripe_sub.current_period_end),
        cancel_at_period_end=stripe_sub.cancel_at_period_end,
    )
```

Register in webhook router alongside existing handlers.

**FE impact:** `SubscriptionSummary` will have accurate period dates. No frontend code change.

---

#### Issue 22: Empty Stripe key allows silent runtime failures
**Spec:** 25 (Reverse Trial)
**Problem:** `stripe_secret_key: str = ""` boots fine, fails on first API call.

**Proposed Fix (BE):**
```python
# config.py
stripe_secret_key: str = ""

# In app startup or billing service init:
def init_stripe(config: Config) -> None:
    if not config.stripe_secret_key:
        if config.environment == "production":
            raise RuntimeError("STRIPE_SECRET_KEY is required in production")
        logger.warning("Stripe key not configured — billing endpoints will return 503")
    else:
        stripe.api_key = config.stripe_secret_key
```

Billing endpoints return `503 Service Unavailable` with `{"detail": "Billing not configured"}` in dev/staging when no key is set.

**FE impact:** Frontend should handle 503 on billing endpoints gracefully. Show "Billing is currently unavailable" rather than a generic error.

---

#### Issue 23: Founding member flag permanence undocumented
**Spec:** 25 (Reverse Trial)
**Problem:** `founding_member` is set TRUE on activation but behavior through cancel/resubscribe is undefined.

**Proposed Fix (BE):**
Add to Spec 25:

> **Founding Member Permanence Rule:**
> `founding_member = TRUE` is set once and never unset. It persists through cancellation, expiry, and resubscription. A founding member who cancels and resubscribes retains the founding member price ($19/mo).
>
> Implementation: The `founding_member` column is never set to FALSE by any code path. The Stripe webhook handler checks `founding_member` when creating/updating subscriptions to apply the correct price ID.

```python
# In subscription update logic:
# NEVER do this:
# await self.update_subscription(user_id, founding_member=False)  # PROHIBITED
```

**FE impact:** `SubscriptionSummary.founding_member` is a permanent badge. Display it on the profile/billing page as a permanent status, not a temporary promotion.

---

#### Issue 24: No price info in SubscriptionSummary
**Spec:** 25 (Reverse Trial)
**Problem:** Faye needs `price_display` and `period_resets_at` to build trial/upgrade UI. Neither exists on `SubscriptionSummary`.

**Proposed Fix (BE):**
Add to `SubscriptionSummary` response model:

```python
class SubscriptionSummary(BaseModel):
    # ... existing fields ...
    price_display: str | None = None            # e.g., "$29/mo", "$19/mo (founding member)", None for free
    period_resets_at: datetime | None = None     # When usage counters reset (first of next month, UTC)
    optimizations_used: int = 0
    optimizations_limit: int = 0                 # -1 for unlimited (pro)
    applications_used: int = 0                   # NOTE: "applications", not "searches" — matches Bryce's model
    applications_limit: int = 0                  # -1 for unlimited (pro)
```

Flat structure — no nested `UsageSummary` sub-model. Field names match Bryce's published contract (`applications_used`/`applications_limit`, not `searches_used`/`searches_limit`).

**FE consumption:**
```typescript
// Display in trial/upgrade UI:
// "3 of 5 optimizations used this month"
// "Resets March 1, 2026"
// "Upgrade to Pro — $29/mo" or "$19/mo (founding member)"
```

**Contract change:** Publish updated `SubscriptionSummary` to `contracts/types.ts`.

---

### Frontend — Token Alignment (Issues 25-28)

---

#### Issue 25: Missing Lato 500 weight
**Spec:** Token Alignment
**Problem:** Spec table (line 83) lists 400/500/700/900 weights but code example (line 91) only loads `["400", "700", "900"]`. Buttons and nav items need 500 (medium) — without it, browser synthesizes faux-bold.

**Proposed Fix:**
Correct the code example in `specs/token-alignment.md` line 91:

```tsx
// BEFORE (spec line 91):
const lato = Lato({ subsets: ["latin"], weight: ["400", "700", "900"], variable: "--font-body" });

// AFTER:
const lato = Lato({ subsets: ["latin"], weight: ["400", "500", "700", "900"], variable: "--font-body" });
```

Spec table at line 83 is already correct. Only the code example needs the fix.

---

#### Issue 26: Border token #2A2A2E deviates from locked brand review #2A2A2A
**Spec:** Token Alignment
**Problem:** Spec line 37 changes `--border-subtle` from `#2A2A2A` to `#2A2A2E` and `--border-hover` from `#3A3A3A` to `#3A3A3E`. The brand review is locked at `#2A2A2A` / `#3A3A3A`. Current `globals.css` (line 24-25) has the correct locked values.

**Proposed Fix:**
Correct the spec. Keep the current `globals.css` values:

```
// specs/token-alignment.md line 37:
// BEFORE: | `--border-subtle` | `#2A2A2E` | Warm-shifted border (was #2A2A2A) |
// AFTER:  | `--border-subtle` | `#2A2A2A` | Border color (per locked brand review) |

// specs/token-alignment.md line 38:
// BEFORE: | `--border-hover` | `#3A3A3E` | Warm-shifted hover border |
// AFTER:  | `--border-hover` | `#3A3A3A` | Hover border (per locked brand review) |
```

`globals.css` lines 24-25 remain unchanged — they already have the correct locked values.

---

#### Issue 27: No prefers-reduced-motion handling
**Spec:** Token Alignment
**Problem:** Animation utilities (`.animate-in`, `.animate-out`) don't respect `prefers-reduced-motion`. Accessibility gap.

**Proposed Fix:**
Add to `specs/token-alignment.md` after line 73 (after `.animate-out` block):

```css
@media (prefers-reduced-motion: reduce) {
  .animate-in,
  .animate-out {
    transition: none;
    opacity: 1;
    transform: none;
  }
}
```

Elements render immediately with no animation when the user has requested reduced motion.

---

#### Issue 28: --text-tertiary aliases --text-muted with no consumption rule
**Spec:** Token Alignment
**Problem:** `--text-tertiary: #71717A` duplicates `--text-muted: #71717A`. Two tokens, same value, no guidance on which to use.

**Proposed Fix:**
Change `--text-tertiary` to reference `--text-muted` via `var()` and add a consumption rule to the spec:

```css
--text-tertiary: var(--text-muted);
```

Add consumption rule to spec:
> **Token consumption rule:**
> - `--text-muted` — general muted text (captions, timestamps, metadata)
> - `--text-tertiary` — specifically for disabled states and placeholder text in inputs
>
> They share the same value today. If disabled states need to diverge from muted text in the future, only `--text-tertiary` changes. Always use the semantically correct token.

---

## Priority Tier 4: Frontend — Marketing Site (Issues 29-36)

---

#### Issue 29: Undocumented scope reduction (4 pages → 3)
**Spec:** Marketing Site
**Problem:** Marketing brief calls for 4 pages including LinkedIn Analysis Tool. Spec drops to 3 (correct — LinkedIn tool needs agent pipeline). Decision isn't documented.

**Proposed Fix:**
Add to `specs/marketing-site.md` after line 624 (end of Out of Scope section):

> **Scope Decision: 4 → 3 Pages**
> The marketing site brief specified 4 pages including a LinkedIn Analysis Tool page. This spec reduces to 3 pages (Home, For Employers, About). The LinkedIn Analysis Tool requires the agent pipeline and authenticated sessions — it belongs in the app (`app.getversed.ai`), not the static marketing site. The LinkedIn Analysis Teaser on the home page (Section 3) links users to the app for the full experience.

---

#### Issue 30: No focus management for mobile hamburger
**Spec:** Marketing Site
**Problem:** No focus trap when hamburger menu opens. Keyboard users get lost.

**Proposed Fix:**
Add to `specs/marketing-site.md` after line 551 (Nav accessibility section):

> **Focus Management (hamburger menu):**
> - On open: focus moves to first menu link (`For Employers`). `Escape` closes menu.
> - While open: `Tab` cycles through menu links + close button only (focus trap).
> - On close: focus returns to hamburger trigger button.
> - Implementation: `useEffect` that adds/removes `keydown` listener for `Escape` and `Tab` trap when menu state is open.

---

#### Issue 31: Mobile bridge seam unspecified
**Spec:** Marketing Site
**Problem:** TranslationCard bridge seam (vertical gradient between panels) has no mobile spec. 60%+ of traffic is mobile.

**Proposed Fix:**
Add to `specs/marketing-site.md` after line 184 (after bridge seam description):

> **Mobile bridge seam (< 768px):**
> When panels stack vertically, the bridge seam becomes a horizontal gradient line between the candidate (top) and recruiter (bottom) panels. Height: 2px. Gradient direction: left-to-right, amber to purple (matching the reading flow). On hover/tap: same subtle glow as desktop.

---

#### Issue 32: No loading/error states for PasteInput
**Spec:** Marketing Site
**Problem:** No visual feedback during paste relay request. No error handling if relay fails.

**Proposed Fix:**
Add to `specs/marketing-site.md` after line 207 (after PasteInput props):

> **PasteInput states:**
>
> | State | Visual | Trigger |
> |-------|--------|---------|
> | `idle` | Pulsing cursor, placeholder text, `--border-subtle` border | Default |
> | `focused` | Amber border, cursor active | User focuses input |
> | `processing` | Text replaced with "Analyzing..." + subtle pulse animation. Input disabled. | After paste, during relay POST |
> | `error` | Red border (`--accent-warning`), message: "Something went wrong. [Try pasting in the app →]({APP_URL})" | Relay POST fails (network, 500) |
> | `rate_limited` | Amber border (not red — high-intent user), amber text. Message: "You've been busy! Sign up for unlimited translations." CTA: "Create free account" (primary). Muted: "Try again in [X] minutes" (parse `retry_after_seconds` from response body JSON). | Relay POST returns 429. First encounter point for Issue 40 (429 UX). |
> | `success` | Brief green flash (`--accent-positive`) before redirect | Relay POST succeeds |

---

#### Issue 33: LinkedIn Teaser component missing from file structure
**Spec:** Marketing Site
**Problem:** Home page Section 3 references a "LinkedIn Analysis Teaser" but there's no component in the file structure.

**Proposed Fix:**
Create `LinkedInTeaser.tsx` as a server component. Every other page section has its own component — consistency matters more than saving a file.

Add to `specs/marketing-site.md` file structure (after FounderSection.tsx):
```
      LinkedInTeaser.tsx              — LinkedIn analysis teaser (Section 3, home page)
```

Component spec:
```typescript
interface LinkedInTeaserProps {
  className?: string;
}
```

Server component. Wraps content in `SectionWrapper`. Contains header, 1-2 sentences, `CTAButton` ghost link to `{APP_URL}/linkedin`. Muted background variation (`--bg-card` at 50% opacity or similar).

---

#### Issue 34: No line-length constraint for dark-mode body copy
**Spec:** Marketing Site
**Problem:** About page body text on wide screens hits ~120 chars/line. Optimal is 60-80.

**Proposed Fix:**
Add to `specs/marketing-site.md` after line 539 (responsive breakpoints):

> **Body copy constraint:**
> All prose text sections (mission statement, founder story, pivot narrative, beliefs) apply `max-w-prose` (Tailwind: `max-width: 65ch`). This limits line length to ~65 characters for optimal readability on wide screens. Applied at the text container level, not the section wrapper.

---

#### Issue 35: Email inconsistency
**Spec:** Marketing Site
**Problem:** For Employers page uses `[redacted-email]` in some contexts and `hello@` in others.

**Proposed Fix:**
Standardize on `hello@getversed.ai` everywhere. The email is already defined in `lib/constants.ts` (line 366 of spec) as `hello@getversed.ai`.

Correct `specs/marketing-site.md` line 462: ensure the For Employers CTA section specifies `hello@getversed.ai` explicitly as the email fallback. Remove any reference to `[redacted-email]`.

---

#### Issue 36: Hero animation contradiction
**Spec:** Marketing Site
**Problem:** Line 378 says "all 7 sections wrapped in SectionWrapper." Line 383 says "no scroll animation on hero." Contradiction.

**Proposed Fix:**
Correct `specs/marketing-site.md` line 378:

```
// BEFORE: 7 sections, all wrapped in SectionWrapper for scroll animation.
// AFTER:  7 sections. Sections 2-7 wrapped in SectionWrapper for scroll animation. Hero renders immediately (above the fold).
```

Hero section renders without `SectionWrapper` — it's the first thing the user sees, no entrance animation needed.

---

## Priority Tier 5: Missing Specs (Issues 37-41)

These need full specs before implementation. Brief outlines below — full specs written separately.

---

#### Issue 37: No /translate page spec
**Problem:** PasteInput hands off to `app.getversed.ai/translate` but there's no spec for that page.

**Outline (full spec needed):**
- Route: `app.getversed.ai/translate`
- Query params: `?paste_token=<token>` (from paste relay)
- On mount: fetch paste text via `GET /v1/paste/{token}`, bootstrap anonymous session via `POST /v1/agent/analyze`
- States: loading (fetching paste), analyzing (SSE streaming), results (cards rendered), error (token expired / network)
- Layout: simplified canvas — chat disabled for anonymous, cards render in feed layout
- CTA: "Sign up to save your results and keep exploring" (persistent, non-blocking)
- If no `paste_token`: show empty state with paste-here prompt

**Depends on:** Issues 11 (paste relay), 12 (claim_token delivery), 4 (pipeline integration)

---

#### Issue 38: No IdentityNode frontend rendering spec
**Problem:** Backend builds IdentityNode (Spec 23E) but there's no frontend component.

**Outline (full spec needed):**
- Component: `IdentityNodeCard` — renders the user's parsed identity (skills, experience, vocabulary profile)
- Appears: after anonymous analysis completes, as the first card on canvas
- Content: key skills extracted, vocabulary gaps identified, suggested searches
- Interaction: expandable sections, "Sign up to build on this" CTA
- Post-signup: IdentityNode becomes the persistent profile card on canvas

**Depends on:** Backend IdentityNode schema (Spec 23E) being finalized

---

#### Issue 39: No ProactiveTeaserCard component spec
**Problem:** Backend Spec 25G defines teaser card data. No frontend component.

**Outline (full spec needed):**
- Component: `ProactiveTeaserCard` — teases a premium feature during free-tier usage
- Content: "Want to optimize for this role? [Upgrade to unlock]" with preview of what optimization would change
- Visual: `--bg-card` with dashed amber border (distinguishable from regular cards)
- Frequency: max 1 per session, appears after 3rd job card
- Dismissable: "Not now" ghost link

**Depends on:** Issue 24 (SubscriptionSummary with price info)

---

#### Issue 40: No 429 rate limit UX
**Problem:** Anonymous users hit 5 req/hr rate limit. No frontend handling for 429 response.

**Outline (full spec needed):**
- When any request returns 429:
  - Parse `retry_after_seconds` from 429 response body JSON (not `Retry-After` header)
  - Show inline message: "You've hit the free limit. Sign up to continue — it's free during early access."
  - CTA: "Sign up" (primary), "Try again in [X] minutes" (muted text)
- For anonymous `/analyze` specifically: disable PasteInput, show countdown
- Never show raw "rate limited" or "too many requests" — always frame as upgrade opportunity

---

#### Issue 41: CORS config needs updating
**Problem:** Marketing site on `getversed.ai` POSTs to backend. CORS must allow that origin. Paste relay introduces a cross-origin POST with `Content-Type: application/json`, which triggers a preflight `OPTIONS` request.

**Proposed Fix (BE):**
```python
# config.py
cors_origins: list[str] = [
    "https://getversed.ai",          # Marketing site
    "https://www.getversed.ai",      # Marketing site (www variant)
    "https://app.getversed.ai",      # App
    "http://localhost:3000",          # Local dev
]
```

CORS middleware configuration must include:
- `Access-Control-Allow-Origin`: from `cors_origins` list (match request `Origin` header)
- `Access-Control-Allow-Headers`: `Content-Type, Authorization` (Content-Type required for JSON POST preflight)
- `Access-Control-Allow-Methods`: `GET, POST, PUT, DELETE, OPTIONS`
- `OPTIONS` handler must return 204 with the above headers for preflight requests

Verify: `OPTIONS /v1/paste` returns correct preflight response. Without this, the marketing site's paste relay POST will be blocked by the browser before it reaches the server.

**FE impact:** None — CORS is server-side. Frontend just makes the request. But FE must NOT set non-standard headers on the paste relay request (would trigger additional preflight requirements).

---

## Spec Corrections Summary

After this document is approved, apply these corrections to the source specs:

### `specs/token-alignment.md`
| Line | Change |
|------|--------|
| 91 | Add `"500"` to Lato weight array: `["400", "500", "700", "900"]` |
| 37 | Change `#2A2A2E` → `#2A2A2A` in border-subtle row |
| 38 | Change `#3A3A3E` → `#3A3A3A` in border-hover row |
| 36 | Change `#71717A` → `var(--text-muted)` for `--text-tertiary` + add consumption rule |
| After 73 | Add `@media (prefers-reduced-motion: reduce)` block |

### `specs/marketing-site.md`
| Line | Change |
|------|--------|
| 21 | Replace sessionStorage mechanism with paste relay (`POST /v1/paste`, not `/v1/paste/store`) |
| 378 | Change "all 7 sections" → "Sections 2-7" (hero excluded) |
| After 49 | Add `LinkedInTeaser.tsx` to file structure (server component, `className` prop) |
| 462 | Specify `hello@getversed.ai` explicitly as email |
| After 184 | Add mobile bridge seam spec (horizontal gradient) |
| After 207 | Add loading/error/processing states for PasteInput |
| After 551 | Add focus trap spec for hamburger menu |
| After 539 | Add `max-w-prose` body copy constraint |
| After 624 | Add scope reduction documentation (4→3 pages rationale) |

---

## Lessons Logged

### Faye (Frontend)

**L001 — Spec code examples must match spec tables**
Code example for Lato omitted weight 500 even though the table listed it. Code examples are what gets copy-pasted into implementation — they must be the source of truth, not the prose.

**L002 — Cross-origin storage is never implicit**
Assumed sessionStorage would work across subdomains. It doesn't. Any cross-origin data transfer needs an explicit mechanism (API relay, postMessage, URL params). Verify storage scope before specifying it.

**L003 — Locked values are locked**
Spec introduced `#2A2A2E` border when the locked brand review says `#2A2A2A`. Even a 1-digit hex deviation from a locked decision needs explicit approval. When in doubt, match the locked value exactly.

**L004 — Animation specs need a reduced-motion counterpart**
Every animation definition should include its `prefers-reduced-motion` equivalent. This isn't optional accessibility polish — it's a baseline requirement.

**L005 — "All sections" language creates contradictions**
"All 7 sections wrapped in SectionWrapper" contradicts "hero has no scroll animation." Be precise: "Sections 2-7 wrapped..." avoids the contradiction entirely.

**L006 — File structure must account for every referenced section**
Home page Section 3 (LinkedIn Teaser) was referenced in page layout but missing from the component file structure. Every named section needs a component entry — consistency with the rest of the file structure pattern.

**L007 — Integration contracts require bilateral specification**
Seven of the 41 issues (7, 12, 13, 14, 16, 37, 41) exist because one side specified a contract without confirming the other side would consume it correctly. When a spec introduces a cross-boundary contract, BOTH the producer spec and the consumer spec must define the contract explicitly — field names, endpoint paths, localStorage keys, TTL values, type definitions. If only one side writes the contract, it will diverge.

**L008 — Always consume the producer's field names, not your own**
Four round-2 blockers had the same root cause: Faye specified field names, formats, and types independently instead of consuming Bryce's published contracts. `text` vs `content`, nanoid vs UUID, camelCase vs snake_case in SSE payloads, 24h vs 72h cookie TTL. When Bryce publishes a Pydantic model, the TypeScript interface must mirror it exactly. Build the mapper on your side, but the wire format is the producer's contract.

### Bryce (Backend) — proposed, for Bryce to log

**L001-BE — Pydantic models must include all DB enum values**
If a migration adds a value to a DB constraint, the corresponding Pydantic Literal must include it. Otherwise the application layer rejects what the database allows.

**L002-BE — Rate limiting behind a load balancer requires header inspection**
`request.client.host` behind a proxy is the proxy's IP, not the client's. Always use `X-Forwarded-For` with validation in load-balanced environments.

**L003-BE — Check-then-insert is always a race condition**
Use `INSERT ... ON CONFLICT` for idempotent creation. Never check existence then insert in separate queries.

**L004-BE — Webhook handlers need the full event set**
Missing `customer.subscription.updated` means plan changes don't sync. When integrating with any webhook-based API, enumerate ALL events that could affect local state.

**L005-BE — Config defaults must fail loud in production**
Empty string defaults for required secrets (`stripe_secret_key: str = ""`) boot silently and fail at runtime. Validate at startup or fail fast on first use.

**L006-BE — Anonymous paths require full audit of every user_id NOT NULL assumption**
Issues 2, 7, 12, 16, 18 all stem from the same root cause. When adding a new auth path (anonymous, API keys, service accounts), audit every table, model, service, and endpoint that touches `user_id`.
