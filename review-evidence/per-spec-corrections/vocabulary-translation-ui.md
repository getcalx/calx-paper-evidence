# Contract Corrections: Vocabulary Translation UI

Source: Nolan (Backend Foil) cross-review

---

## Card Interaction Endpoint — Two Paths, Pick One

```
// WRONG (3 locations):
POST /v1/agent/cards/{card_id}/interact

// RIGHT (FOIL_REVIEW_FIXES Issue 5/7):
POST /v1/agent/sessions/{sessionId}/cards/{cardId}/interact
```

Session-scoped path is correct — backend validates card was emitted in that session. Pull `sessionId` from `versed_anon_session` localStorage.

## SSE Contract Fields — Not Published

**`card_state` and `outcome_validated`** on `SSETranslationCard` — NOT in the published FOIL_REVIEW_FIXES Issue 14 contract:

```ts
// Published contract (FOIL_REVIEW_FIXES):
interface SSETranslationCard {
  source_phrase: string;
  target_phrase: string;
  category: string | null;
  confidence_score: number | null;
  context: string | null;
}

// Spec adds (not in contract):
card_state: "match" | "needs_input" | "gap"    // NOT PUBLISHED
outcome_validated: boolean                       // NOT PUBLISHED
```

**Options:**
- A: RFC to add both fields to contract. Keep `inferCardState` as fallback.
- B: Remove both from FE type. Use `inferCardState` from `confidence_score` only.

Spec also contradicts itself: line 364 computes state from `confidence_score`, line 466 says use backend-provided `card_state`. Pick one.

## `card_id` Missing from SSE Contract

`SSECardEvent` in FOIL_REVIEW_FIXES does NOT include `card_id`:

```ts
// Published:
interface SSECardEvent {
  card_type: "translation_card" | "gap_card" | "agent_commentary" | ...;
  data: SSETranslationCard | SSEGapCard | Record<string, unknown>;
}

// Needs:
card_id: string;  // Required for card interactions
```

**Action:** File RFC or update FOIL_REVIEW_FIXES to add `card_id` to `SSECardEvent`. Without it, frontend can't reference cards back to backend.

## Interaction Request Shape Mismatch

```ts
// Component-level (CardInteraction):
{ action: "accept" | "reject" | "edit" | "claim", cardId, updatedText? }

// Wire-level (CardInteractionRequest):
{ card_id, card_type, interaction_type: "accept" | "reject" | "edit" | "submit_input", user_input? }
```

Mismatches: `"claim"` vs `"submit_input"`, `updatedText` vs `user_input`, missing `card_type` on component level.

**Fix:** Add `mapCardInteraction()` to card-mappers.ts. Document canonical wire values. Specify where `card_type` comes from.

## Paste Response Field Name

FOIL_REVIEW_FIXES has internal contradiction:
- BE contract section: `GET /v1/paste/{token}` → `{ "text": string }`
- FE contract section: `const { content } = await res.json()`
- `PasteRetrieveResponse` type: `{ content: string }`

**Fix:** Resolve in FOIL_REVIEW_FIXES. Recommend `content` (matches `PasteStoreRequest.content`). Update BE contract description.

## Unhandled SSE Events

`voice` and `text` events defined in union but never consumed:
- `text` events (agent commentary) → should render as text bubble. PRD requires this.
- `voice` events → log and skip for MVP.

Add handling in SSE consumer. At minimum, render `text` events.

## Other

- `tool_status` events have no render target — add status text element during `analyzing` state
- `agent_commentary` card type hits `addCard` without type guard — check card_type before adding, skip unknown types
- Stale localStorage session: add TTL check (72h per FOIL_REVIEW_FIXES Issue 17) on page mount
- `streamAnalysis` boundary: explicitly note it handles ONLY SSE (step 2), not paste fetch (step 1). Add `fetchPasteContent()` helper.
- AgentTease should have engagement-based fallback: if `done` fires without `tease` but card count >4, render tease anyway
