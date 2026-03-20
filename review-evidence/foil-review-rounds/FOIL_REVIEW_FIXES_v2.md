# Foil Review Fixes — Specs 23-25, Token Alignment, Marketing Site (v2)

**Date:** 2026-03-11 (v1), 2026-03-11 (v2)
**Reviewers:** Nolan (BE foil), Sloane (FE foil)
**Verdict:** v1 REVISE (41 issues). v2 addresses 11 revision items from `rfcs/001-foil-revision-brief.md`.
**Pattern:** Issues concentrate at integration boundaries, not within components.

---

## v2 Changelog (Revision Brief Items 1-7)

| # | Brief Item | What Changed |
|---|-----------|--------------|
| 1 | Issue 7 — Anonymous card auth **[DECIDED]** | Replaced `claim_token` in request body with `anonymous_session_id` cookie. `HttpOnly; Secure; SameSite=Lax`. Cookie set by `/analyze` response. |
| 2 | Issue 12 — claim_token delivery | Dropped `X-Claim-Token` header. SSE `session_init` is sole delivery. localStorage key aligned to `versed_anon_session` (was `versed_claim_token` in FE doc). |
| 3 | Issue 17 — TTL alignment | Changed from 48h/48h to 72h claim / 7d cleanup. Matches FE doc. |
| 4 | Issue 1 — agent_state enum | Verified against actual `src/soaria/models/agent.py:52-59`. Complete enum is 7 values. Faye's doc missing `applying`, `reviewing`, `returning` — flagged. |
| 5 | Issue 19 — max_translations cap | Changed from Pydantic `le=5` to runtime cap per subscription tier. |
| 6 | Issue 38 — IdentityNode type | Cross-checked against Spec 23E. Types match. Added to `contracts/types.ts` addendum. |
| 7 | Issue 11 — Paste endpoint path | Standardized on `POST /v1/paste` (not `/v1/paste/store`). Faye's doc to align. |

---

## Critical Blockers (1-16)

### Issue 1: PlannerInput.agent_state missing "anonymous"

**Spec:** 23 (Anonymous Sessions)
**Problem:** `PlannerInput.agent_state` is a `Literal` with 6 values. Spec 23 adds
`agent_state = 'anonymous'` to the DB CHECK constraint but never updates the Pydantic model.

**Fix — `src/soaria/models/agent.py`:**

```python
# Verified current values at lines 52-59:
# "onboarding", "searching", "optimizing", "applying", "reviewing", "returning"
#
# Fixed — 7 values (add "anonymous"):
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

**v2 note:** Faye's doc listed `"idle"` instead of `"applying"`, `"reviewing"`, `"returning"`.
`"idle"` does not exist in the codebase. The authoritative enum is the 7 values above.
Both docs must show this identical list. Faye to correct.

No FE impact — `agent_state` is not exposed to the frontend.

---

### Issue 2: ReframingPatternCreate.user_id not optional

**Spec:** 23 (Anonymous Sessions)
**Problem:** `ReframingPatternCreate.user_id: UUID` is required. Anonymous sessions create
reframing patterns with `user_id = NULL`. The model will reject anonymous pattern creation.

**Fix — `src/soaria/models/agent.py`:**

```python
class ReframingPatternCreate(SoariaBase):
    source_phrase: Annotated[str, StringConstraints(min_length=1)]
    target_phrase: Annotated[str, StringConstraints(min_length=1)]
    source_embedding: Annotated[list[float], Field(min_length=1024, max_length=1024)]
    target_embedding: Annotated[list[float], Field(min_length=1024, max_length=1024)]
    jd_keyword: str | None = None
    reframing_strategy: str | None = None
    job_id: UUID | None = None
    user_id: UUID | None = None       # Nullable for anonymous sessions
    session_id: UUID | None = None     # Set for anonymous sessions
    optimization_id: UUID | None = None

    @model_validator(mode="after")
    def at_least_one_owner(self) -> "ReframingPatternCreate":
        if self.user_id is None and self.session_id is None:
            raise ValueError("Either user_id or session_id must be set")
        return self
```

**FE impact:** None — reframing pattern creation is backend-internal.

---

### Issue 3: IP extraction uses `request.client.host` — wrong behind proxy

**Spec:** 23 (Anonymous Sessions)
**Problem:** `request.client.host` returns the proxy IP on Railway, not the user's IP.

**Fix — `src/soaria/api/agent.py` (new helper):**

```python
def _get_client_ip(request: Request) -> str:
    """Extract real client IP. Railway passes X-Forwarded-For."""
    forwarded = request.headers.get("x-forwarded-for")
    if forwarded:
        return forwarded.split(",")[0].strip()
    return request.client.host if request.client else "unknown"
```

**FE impact:** None.

---

### Issue 4: No shared pipeline between `/chat` and `/analyze`

**Spec:** 23 (Anonymous Sessions), 24 (Agent Alignment)
**Problem:** `_event_stream()` is ~370 lines. Without extraction, `/analyze` either
duplicates the pipeline or imports a private function.

**Fix — new file `src/soaria/agent/pipeline.py`:**

```python
async def run_agent_turn(
    *,
    pool: DatabasePool,
    settings: Settings,
    session_id: UUID,
    message: str,
    user_id: UUID | None,         # None for anonymous
    skip_memory: bool = False,     # True for anonymous
    conversation_history: list[ConversationMessage],
    agent_state: str,
    user_profile_summary: str | None = None,
    has_resume: bool = False,
    has_prior_analysis: bool = False,
) -> AsyncGenerator[SSEEvent, None]:
    """Shared pipeline: planner -> dispatcher -> assembler -> SSE events."""
    ...
```

**Registry.execute() signature:**
```python
async def execute(self, tool_name: str, params: dict, user_id: UUID | None, ...) -> ToolResult
```

Tools requiring `user_id` check for `None` and return:
```python
ToolError(error_type="validation", message="Sign up to use this feature", recoverable=True)
```

This surfaces in the SSE stream as an `error` event:
```
event: error
data: {"event_type": "error", "sequence": N, "message": "Sign up to use this feature", "recoverable": true}
```

**FE handling:** On `error` event with `recoverable=true` and message containing "Sign up",
show signup prompt — not a generic error.

---

### Issue 5: No card_id validation against session

**Spec:** 24 (Agent Alignment)

**Fix — validation query + GIN index:**

```sql
-- Migration 013 addition:
CREATE INDEX IF NOT EXISTS idx_agent_messages_card_artifacts
ON agent_messages USING GIN (card_artifacts jsonb_path_ops)
WHERE card_artifacts IS NOT NULL;
```

```python
async def _validate_card_in_session(conn, session_id: UUID, card_id: UUID) -> bool:
    row = await conn.fetchval(
        """SELECT 1 FROM agent_messages
           WHERE session_id = $1
             AND card_artifacts @> $2::jsonb
           LIMIT 1""",
        session_id,
        json.dumps([{"card_id": str(card_id)}]),
    )
    return row is not None
```

Returns 404 `"Card not found in session"` on failure.

**FE impact:** None — FE sends card_id from rendered cards.

---

### Issue 6: LLM JSON parse failure crashes analyze_vocabulary

**Spec:** 24 (Agent Alignment)

**Fix — defensive parsing in `agent/tools/analysis.py`:**

```python
async def _analyze_vocabulary(...) -> AnalyzeVocabularyOutput:
    raw = await llm_call(...)
    try:
        parsed = _extract_json(raw)
        translations = [VocabularyTranslation(**t) for t in parsed["translations"]]
        gaps = parsed.get("gaps", [])
        summary = parsed.get("summary", "Analysis complete.")
    except (json.JSONDecodeError, KeyError, ValidationError) as exc:
        logger.warning("vocab_analysis_parse_failure", error=str(exc), raw_length=len(raw))
        return AnalyzeVocabularyOutput(
            translations=[],
            gaps=[],
            summary="I had trouble parsing the analysis. Let me try again with a simpler approach.",
        )
    return AnalyzeVocabularyOutput(translations=translations, gaps=gaps, summary=summary)
```

**FE impact:** FE receives empty translations list + explanatory summary. Render "No
translations found" state with retry option.

---

### Issue 7: Anonymous card interaction auth [DECIDED — Cookie]

**Spec:** 24 (Agent Alignment), 23 (Anonymous Sessions)
**Problem:** Card interaction endpoint uses `require_approved`. Anonymous users blocked.

**v2 fix — `anonymous_session_id` cookie (Spencer-approved):**

**BE sets cookie on `/analyze` response:**
```python
# api/agent.py — POST /v1/agent/analyze response
response = StreamingResponse(event_stream(), media_type="text/event-stream")
response.set_cookie(
    key="anonymous_session_id",
    value=str(session_id),
    httponly=True,
    secure=settings.is_production,
    samesite="lax",
    max_age=72 * 3600,  # 72h, matches claim TTL
    path="/",
)
return response
```

**Card interaction endpoint — dual auth via JWT or cookie:**
```python
@router.post("/sessions/{session_id}/cards/{card_id}/interact")
async def record_interaction(
    request: Request,
    session_id: UUID,
    card_id: UUID,
    body: CardInteractionRequest,  # No claim_token field — cookie handles anon auth
    pool: DatabasePool = Depends(get_db_pool),
    user: AuthenticatedUser | None = Depends(optional_current_user),
):
    if user:
        # Authenticated path: verify user owns the session
        await _verify_session_owner(pool, session_id, user.id)
        effective_user_id = user.id
    else:
        # Anonymous path: validate cookie matches session
        cookie_session_id = request.cookies.get("anonymous_session_id")
        if not cookie_session_id or UUID(cookie_session_id) != session_id:
            raise HTTPException(401, "Authentication or anonymous session required")
        await _verify_anonymous_session(pool, session_id)
        effective_user_id = None

    await _validate_card_in_session(pool, session_id, card_id)
    return await record_card_interaction(pool, session_id, card_id, body, effective_user_id)
```

**New dependency — `optional_current_user`:**
```python
# api/deps.py
async def optional_current_user(request: Request, pool: DatabasePool = Depends(get_db_pool)) -> AuthenticatedUser | None:
    """Returns user if JWT present, None if not. Never raises."""
    try:
        return await get_current_user(request, pool)
    except HTTPException:
        return None
```

**CardInteractionRequest — NO claim_token field:**
```python
class CardInteractionRequest(SoariaBase):
    card_id: UUID
    card_type: Literal["translation", "gap"]
    interaction_type: CardInteractionType
    user_input: Annotated[str, StringConstraints(max_length=2000)] | None = None
    # No claim_token — anonymous auth uses HttpOnly cookie
```

**Cross-origin note:** The cookie is set by `app.getversed.ai` (the `/analyze` response),
so subsequent requests from the app domain include it automatically. The marketing site
(`getversed.ai`) redirects to the app via paste relay — it never makes direct card
interaction requests.

**Contract (both sides must match):**

| Side | Detail |
|------|--------|
| BE | `POST /v1/agent/analyze` sets `Set-Cookie: anonymous_session_id=<uuid>; HttpOnly; Secure; SameSite=Lax; Max-Age=259200; Path=/` |
| FE | Cookie is automatic on same-origin (`app.getversed.ai`). No explicit handling. FE does NOT read or set this cookie — it's `HttpOnly`. |

---

### Issue 8: Subscription creation race condition

**Spec:** 25 (Reverse Trial)

**Fix — INSERT ... ON CONFLICT:**

```python
async def get_or_create_subscription(pool: DatabasePool, user_id: UUID) -> Subscription:
    async with pool.acquire() as conn:
        await conn.execute(
            """INSERT INTO subscriptions (user_id, status)
               VALUES ($1, 'free')
               ON CONFLICT (user_id) DO NOTHING""",
            user_id,
        )
        row = await conn.fetchrow(
            "SELECT * FROM subscriptions WHERE user_id = $1", user_id
        )
        return Subscription(**dict(row))
```

**FE impact:** None.

---

### Issue 9: `period_start` timezone ambiguity

**Spec:** 25 (Reverse Trial)

**Fix — always UTC:**

```python
def _current_period_start() -> date:
    """First day of current month in UTC."""
    return date.today().replace(day=1)  # Railway runs in UTC
```

SQL: `date_trunc('month', NOW() AT TIME ZONE 'UTC')::date`

**FE impact:** Display "Resets monthly" without specifying exact time.

---

### Issue 10: Trial-to-Pro race condition

**Spec:** 25 (Reverse Trial)

**Fix — Stripe-wins semantics:**

```python
async def end_trial(pool, user_id):
    """Downgrade trial. No-op if user already has Stripe subscription."""
    row = await conn.fetchrow(
        """UPDATE subscriptions SET status = 'free', updated_at = NOW()
           WHERE user_id = $1 AND status = 'trialing'
             AND stripe_subscription_id IS NULL
           RETURNING *""",
        user_id,
    )
    return Subscription(**dict(row)) if row else None

async def activate_pro(pool, user_id, stripe_sub_id, stripe_price_id):
    """Unconditionally wins. Paying user never gets downgraded."""
    row = await conn.fetchrow(
        """UPDATE subscriptions
           SET status = 'pro', stripe_subscription_id = $2, stripe_price_id = $3, updated_at = NOW()
           WHERE user_id = $1
           RETURNING *""",
        user_id, stripe_sub_id, stripe_price_id,
    )
    return Subscription(**dict(row))
```

---

### Issue 11: sessionStorage cross-domain handoff broken — Paste Relay

**Spec:** Marketing Site

**v2 fix — endpoint path standardized to `POST /v1/paste` (not `/v1/paste/store`).**

**New table (migration 012 or separate):**
```sql
CREATE TABLE IF NOT EXISTS paste_tokens (
    token UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    content TEXT NOT NULL,
    ip_address INET,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    fetched_at TIMESTAMPTZ  -- NULL until consumed
);

CREATE INDEX IF NOT EXISTS idx_paste_tokens_cleanup
ON paste_tokens (created_at) WHERE fetched_at IS NULL;
```

**Endpoints:**

```python
# POST /v1/paste — no auth, rate-limited 10/hr per IP, max 50KB body
@router.post("/paste", status_code=201)
@limiter.limit("10/hour")
async def create_paste(
    request: Request,
    body: PasteRequest,  # content: Annotated[str, StringConstraints(max_length=51_200)]
    pool: DatabasePool = Depends(get_db_pool),
) -> PasteResponse:  # { token: UUID }

# GET /v1/paste/{token} — no auth, one-time atomic read
@router.get("/paste/{token}")
async def get_paste(
    token: UUID,
    pool: DatabasePool = Depends(get_db_pool),
) -> PasteContentResponse:  # { content: str }
```

**Models:**
```python
class PasteRequest(SoariaBase):
    content: Annotated[str, StringConstraints(min_length=1, max_length=51_200)]

class PasteResponse(SoariaBase):
    token: UUID

class PasteContentResponse(SoariaBase):
    content: str
```

**GET logic — one-time atomic read, 5-minute expiry:**
```sql
UPDATE paste_tokens
SET fetched_at = NOW()
WHERE token = $1
  AND fetched_at IS NULL
  AND created_at > NOW() - interval '5 minutes'
RETURNING content
```

404 if not found, already consumed, or expired.

**Cleanup:** Delete paste_tokens older than 1 hour, piggybacked on anonymous session TTL flow.

**FE flow (marketing site → app):**
1. User pastes in PasteInput on `getversed.ai`
2. FE POSTs to `api.getversed.ai/v1/paste` → gets `{ token: UUID }`
3. FE redirects to `app.getversed.ai/translate?paste_token=<token>`
4. App page calls `GET api.getversed.ai/v1/paste/<token>` → gets `{ content }` → pipes to `POST /v1/agent/analyze`

**CORS requirement:** `getversed.ai` and `www.getversed.ai` must be in allowed origins
for the `POST /v1/paste` preflight. `Content-Type` must be in `Access-Control-Allow-Headers`.
(See Issue 41.)

---

### Issue 12: claim_token delivery — SSE `session_init` only

**Spec:** 23 (Anonymous Sessions)

**v2 fix — SSE `session_init` is the sole delivery mechanism. No response headers.**

The `EventSource` API doesn't expose response headers. FE uses `fetch()` + `ReadableStream`
for SSE (not `EventSource`), but the SSE-first approach is cleaner and avoids split delivery.

**BE model:**
```python
class SessionInitEvent(SSEEvent):
    """First event in every anonymous SSE stream."""
    event_type: Literal["session_init"] = "session_init"
    session_id: UUID
    claim_token: UUID | None = None  # Only set for anonymous sessions
```

**Pipeline change:** `run_agent_turn()` yields `SessionInitEvent` as sequence 0, before
any tool_status or card events. Only emitted for anonymous sessions (authenticated sessions
already have session_id from the request).

**Wire format:**
```
id: 0
event: session_init
data: {"event_type": "session_init", "sequence": 0, "session_id": "abc-123", "claim_token": "def-456"}
```

**FE contract (must match Faye's doc):**
- FE listens for `session_init` event in the SSE stream
- Stores in `localStorage` key `versed_anon_session` (NOT `versed_claim_token`):
```ts
interface AnonSession {
  sessionId: string;    // UUID
  claimToken: string;   // UUID
  createdAt: string;    // ISO 8601
}
localStorage.setItem("versed_anon_session", JSON.stringify({
  sessionId: event.session_id,
  claimToken: event.claim_token,
  createdAt: new Date().toISOString(),
}));
```

**FE consumption note:** SSE consumed via `fetch()` + `ReadableStream`, not `EventSource`.

---

### Issue 13: FE storage + claim flow contract

**Spec:** 23 (Anonymous Sessions)

**localStorage key:** `versed_anon_session`
**Shape:** `{ sessionId: string, claimToken: string, createdAt: string }`

**Lifecycle:**
| Step | Action | Detail |
|------|--------|--------|
| 1 | Session starts | `session_init` SSE → store in localStorage |
| 2 | Tab refresh | Read localStorage, resume session via `session_id` + `claim_token` in `AnonymousChatRequest` |
| 3 | Signup | After Supabase auth, read localStorage for `claimToken` |
| 4 | Claim | `POST /v1/agent/sessions/claim` with `{ session_id, claim_token }` + Bearer JWT |
| 5 | Success | Remove `versed_anon_session` from localStorage |
| 6 | Failure (expired) | Remove key. Show: "Welcome! Your previous session expired." |
| 7 | Token lost | Skip claim. Fresh start. |

**BE claim endpoint contract:**
```
POST /v1/agent/sessions/claim
  Auth: Bearer <jwt> (required)
  Body: { "session_id": UUID, "claim_token": UUID }
  200: { "session_id": UUID, "messages_transferred": int, "patterns_transferred": int }
  404: Token not found or expired (>72h)
  409: Session already claimed by different user
```

---

### Issue 14: SSE field naming — source_phrase vs candidateText

**Fix unchanged from v1.** Backend keeps `source_phrase` / `target_phrase`. FE maps:
- `source_phrase` → `candidateText`
- `target_phrase` → `recruiterText`

Mapping documented in `contracts/types.ts`.

---

### Issue 15: versed_app_url default wrong

**Fix — `config.py`:**
```python
versed_app_url: str = "https://app.getversed.ai"
```

---

### Issue 16: Session claim doesn't transfer card_interactions

**Fix — add to claim transaction:**

```python
# 3. Migrate card_interactions
await conn.execute(
    """UPDATE card_interactions SET user_id = $1
       WHERE session_id = $2 AND user_id IS NULL""",
    user_id, session_id,
)
```

---

## Important Issues (17-41)

### Issue 17: Anonymous session TTL — 72h claim / 7d cleanup

**Spec:** 23 (Anonymous Sessions)

**v2 fix — aligned with FE doc:**
- **Claim token validity:** 72 hours (3 days)
- **Session cleanup eligibility:** 7 days of inactivity
- **Cleanup job:** Runs daily, deletes sessions inactive for 7+ days

```python
# config.py
anonymous_claim_ttl_hours: int = 72    # Was 48
anonymous_cleanup_after_days: int = 7  # Was 2 (48h)
```

**Cleanup query:**
```sql
DELETE FROM agent_sessions
WHERE is_anonymous = TRUE AND user_id IS NULL
  AND last_message_at < NOW() - interval '7 days'
```

**Cookie Max-Age alignment:** `anonymous_session_id` cookie `Max-Age=259200` (72h) matches
claim TTL. Cookie expires at same time as claim eligibility.

**FE copy:** "Your results are saved" (no duration specified).

---

### Issue 18: Anonymous queries and RLS

**Fix:** Enumerate all service_role queries for anonymous paths:

| Query | Table | Why service_role |
|-------|-------|-----------------|
| INSERT session | agent_sessions | No user JWT available |
| INSERT message | agent_messages | Belongs to anonymous session |
| INSERT pattern | reframing_patterns | `user_id = NULL`, session-scoped |
| INSERT interaction | card_interactions | `user_id = NULL`, session-scoped |
| SELECT session by claim_token | agent_sessions | Resume anonymous session |
| INSERT/SELECT paste_tokens | paste_tokens | No auth on paste relay |

**Constraint:** Anonymous repository methods MUST NOT touch `user_profiles`, `resumes`,
`subscriptions`, `applications`, or any table with user PII.

Add test: verify authenticated operations still go through RLS and can't access other
users' data via anonymous paths.

---

### Issue 19: max_translations — runtime cap per tier, not Pydantic

**Spec:** 24 (Agent Alignment)

**v2 fix — runtime cap in tool handler, driven by subscription tier:**

```python
# Tier limits (config or constants)
TRANSLATION_LIMITS = {
    "anonymous": 5,
    "free": 10,
    "trialing": 25,
    "pro": 25,
    "founding_member": 25,
}

async def _analyze_vocabulary(input: AnalyzeVocabularyInput, context: ToolContext) -> AnalyzeVocabularyOutput:
    tier = context.subscription_tier or "anonymous"
    effective_max = min(input.max_translations, TRANSLATION_LIMITS.get(tier, 5))
    # Use effective_max in prompt and truncate response
    ...
```

**Pydantic model stays permissive:** `max_translations: int = Field(default=10, ge=3, le=25)`
— the `le=25` allows Pro values; the runtime cap enforces per-tier limits.

---

### Issue 20: Gap card importance classification criteria

**Fix — add to vocab analysis prompt:**
```
For each gap, classify importance:
- "required": JD uses "must have", "required", "X+ years", or lists in Requirements section
- "preferred": JD uses "preferred", "ideal", "bonus", in Preferred section
- "nice_to_have": Mentioned casually, in culture section, or as "or equivalent"
```

---

### Issue 21: Missing customer.subscription.updated webhook

**Fix — add handler in `api/billing.py`:**
```python
elif event_type == "customer.subscription.updated":
    sub = event_data["object"]
    await conn.execute(
        """UPDATE subscriptions
           SET current_period_start = to_timestamp($2),
               current_period_end = to_timestamp($3),
               status = CASE
                   WHEN $4 = 'past_due' THEN 'past_due'
                   WHEN $4 = 'canceled' THEN 'cancelled'
                   ELSE status
               END,
               updated_at = NOW()
           WHERE stripe_subscription_id = $1""",
        sub["id"], sub["current_period_start"], sub["current_period_end"], sub["status"],
    )
```

---

### Issue 22: Empty Stripe key not caught

**Fix — validate on first billing operation:**
```python
def _get_stripe_client(settings: Settings) -> None:
    if not settings.stripe_secret_key:
        raise ConfigError("Stripe not configured: STRIPE_SECRET_KEY is empty")
    stripe.api_key = settings.stripe_secret_key
```

---

### Issue 23: founding_member is append-only

**Rule:** Once `founding_member = TRUE`, it never reverts. No code path sets it to FALSE.
Resubscription always uses founding price. Document in spec 25B and `AGENTS.md`.

---

### Issue 24: SubscriptionSummary missing price + period

**Fix — add fields:**
```python
class SubscriptionSummary(SoariaBase):
    # ... existing fields
    price_display: str | None = None         # "$19/mo", "$29/mo", or None
    period_resets_at: datetime | None = None  # First of next UTC month
```

**FE contract — `contracts/types.ts`:**
```ts
interface SubscriptionSummary {
  status: SubscriptionStatus;
  is_pro: boolean;
  trial_days_remaining: number | null;
  founding_member: boolean;
  optimizations_used: number;
  optimizations_limit: number | null;
  applications_used: number;
  applications_limit: number | null;
  price_display: string | null;
  period_resets_at: string | null;  // ISO 8601
}
```

---

### Issue 25: Lato weight 500 missing

**FE fix.** Add `"500"` to Lato import weights.

---

### Issue 26: Border token value

**FE fix.** `--border-subtle: #2A2A2A` per locked brand review.

---

### Issue 27: Reduced-motion support

**FE fix.** Add `@media (prefers-reduced-motion: reduce)` rule to `globals.css`.

---

### Issue 28: --text-tertiary [DECIDED — Keep with alias]

**v2 fix (Spencer-approved):** Keep `--text-tertiary` as `var(--text-muted)` alias.

```css
--text-tertiary: var(--text-muted);
```

**Consumption rule:**
- `--text-muted`: timestamps, captions, secondary info
- `--text-tertiary`: disabled states, placeholder text

Same value today. Semantic distinction preserved for future differentiation.

---

### Issue 29: LinkedIn page deferral

**FE fix.** Add "Scope Decision" section documenting LinkedIn company page deferral.

---

### Issue 30: Hamburger focus trap

**FE fix.** Add focus trap to Nav.tsx hamburger menu.

---

### Issue 31: Mobile bridge seam

**FE fix.** Horizontal 1px gradient on `<768px`.

---

### Issue 32: PasteInput loading/error/timeout states + 429

**FE fix.** Add states: `idle`, `loading`, `error`, `timeout`, `rate_limited`.

Rate-limited state (connects to Issue 40):
- Amber border (not red), amber text
- Message: "You've been busy! Sign up for unlimited translations."
- CTA: "Create free account"
- Muted: "Try again in [X] minutes" (parse `Retry-After` header)

---

### Issue 33: LinkedInTeaser component

**FE fix.** Create `LinkedInTeaser.tsx` as server component.

---

### Issue 34: About page max-width

**FE fix.** `max-w-[65ch]` on body text containers.

---

### Issue 35: Email address standardization

**BE fix — `config.py`:**
```python
email_from_address: str = "Versed AI <hello@getversed.ai>"
```

---

### Issue 36: Hero SectionWrapper animate prop

**FE fix.** `animate={false}` prop on SectionWrapper. Hero renders immediately.

---

### Issue 37: /translate page contract

**Cross-boundary contract:**

| Step | System | Action |
|------|--------|--------|
| 1 | App page | Read `paste_token` from query params |
| 2 | App → BE | `GET /v1/paste/{token}` → `{ content }` |
| 3 | App → BE | `POST /v1/agent/analyze` with content as message |
| 4 | BE → App | SSE stream: `session_init` → tool_status → cards → commentary → done |
| 5 | App | On `session_init`: store session in localStorage |
| 6 | App | Render cards as they arrive |

**Error states:**
- Paste 404 (expired): show PasteInput fallback, "This link has expired. Paste your text again."
- SSE failure: "Something went wrong. Try again."
- No `paste_token` param: show empty state with paste-here prompt

---

### Issue 38: identity_node in /me response

**BE fix — add to GET /v1/users/me response:**
```python
profile["identity_node"] = row.get("identity_node")
```

**Verified against Spec 23E — IdentityNode type for `contracts/types.ts`:**
```ts
interface IdentityNode {
  display_name: string | null;
  current_role: string | null;
  current_company: string | null;
  career_stage: string | null;
  key_skills: string[];         // max 20
  industries: string[];         // max 10
  years_experience: number | null;
  source: "resume" | "linkedin" | "conversation" | "manual";
  updated_at: string | null;    // ISO 8601
}
```

**UserProfileResponse addition:**
```ts
interface UserProfileResponse {
  // ... existing fields
  identity_node: IdentityNode | null;
}
```

---

### Issue 39: proactive_teaser card type in FE types

**Fix — add to `contracts/types.ts`:**
```ts
interface ProactiveTeaserCard {
  card_type: "proactive_teaser";
  card_id: string;
  message: string;
  upgrade_cta: boolean;
  insight_count: number;
}
```

FE renders as muted card with upgrade CTA. CTA triggers `POST /v1/billing/checkout` →
get `{ checkout_url }` → redirect to Stripe.

---

### Issue 40: 429 response contract

**Fix — response headers + body:**

```
HTTP/1.1 429 Too Many Requests
Retry-After: 3600
Content-Type: application/json
```

```json
{
  "detail": "Rate limit exceeded. Sign up for unlimited access.",
  "retry_after_seconds": 3600
}
```

**BE implementation:**
```python
@app.exception_handler(RateLimitExceeded)
async def rate_limit_handler(request: Request, exc: RateLimitExceeded):
    # slowapi provides retry_after on the exception
    retry_after = getattr(exc, "retry_after", 3600)
    return JSONResponse(
        status_code=429,
        content={
            "detail": "Rate limit exceeded. Sign up for unlimited access.",
            "retry_after_seconds": int(retry_after),
        },
        headers={"Retry-After": str(int(retry_after))},
    )
```

**FE handling:** Parse `retry_after_seconds` from response body. Show upgrade prompt
(PasteInput `rate_limited` state, Issue 32).

---

### Issue 41: CORS origins for production + paste relay

**BE fix — `config.py` documentation:**

```python
# CORS origins (comma-separated). Production must include:
#   https://getversed.ai,https://app.getversed.ai,https://www.getversed.ai
# Set via CORS_ORIGINS env var on Railway.
cors_origins: str = "http://localhost:5173"
```

**CORS middleware configuration must allow:**
- Origins: `getversed.ai`, `app.getversed.ai`, `www.getversed.ai`
- Headers: `Content-Type`, `Authorization` in `Access-Control-Allow-Headers`
- Methods: `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`

**v2 note:** Paste relay (`POST /v1/paste`) specifically needs `getversed.ai` in allowed
origins for the preflight `OPTIONS` request. Verify `OPTIONS` handling for this endpoint.

---

## Cross-Cutting Observations

### Pattern: Integration boundary failures
Issues 4, 7, 11, 12, 13, 14, 16, 37, 38, 39 are all integration seam failures.

### Pattern: LLM output trust
Issues 6, 19, 20 trust LLM output without server-side enforcement.

### Pattern: Auth dual-path
Issues 3, 7, 12, 13, 16, 18 stem from the anonymous path being additive to authenticated.

### Pattern: Cross-domain handoff
Issues 11, 32, 37 form a chain: marketing → paste relay → app.

---

## Verification Checklist

- [ ] Nolan re-reviews all 16 critical fixes (BE-focused)
- [ ] Sloane re-reviews all FE-facing fixes (Issues 11-14, 25-36, 37-41)
- [ ] Cross-check: every fix that touches both BE and FE has matching contracts
- [ ] No fix introduces a new untested code path
- [ ] All SQL changes have corresponding migration additions
- [ ] All new SSE event types documented in contracts/types.ts
- [ ] v2 items 1-7 verified against FE doc
