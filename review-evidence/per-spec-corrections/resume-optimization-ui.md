# Contract Corrections: Resume Optimization UI

Source: Nolan (Backend Foil) cross-review

---

## CRITICAL: URL Path Parameters

```
// WRONG (spec):
/v1/resumes/{tid}/optimize
/v1/resumes/{tid}/versions/{vid}/download
/v1/resumes/{tid}/versions/{vid}/chat

// RIGHT (contract):
/v1/resumes/{template_id}/optimize
/v1/resumes/{template_id}/versions/{version_id}/download
/v1/resumes/{template_id}/versions/{version_id}/chat
```

Replace `{tid}` → `{template_id}` and `{vid}` → `{version_id}` everywhere.

## CRITICAL: Optimize Returns JSON, Not SSE

```ts
// WRONG: spec assumes SSE stream with tool_status + card events

// RIGHT (contract):
POST /v1/resumes/{template_id}/optimize
→ 200: OptimizationResult (JSON, cached)
→ 201: OptimizationResult (JSON, new)

interface OptimizationResult {
  version: ResumeVersionResponse;
  deltas: OptimizationDeltaResponse[];
  case_studies_used: UUID[];
  was_cached: boolean;
}

interface OptimizationDeltaResponse {
  section_name: string;
  original_text: string;
  optimized_text: string;
  reasoning: string;
  jd_keyword_targeted: string | null;
  confidence: number;
}
```

**Options:**
- A: File RFC for SSE extension. Define SSE event shapes as contract extension. Mark as "pending RFC."
- B: Rewrite data flow to consume `OptimizationResult` synchronously. Map `deltas[]` to display types. Replace OptimizationStatus stage progression with simple loading spinner.

**Option B is safer for shipping now.** The spec's `WireDiffSection`/`WireDiffChange` types don't match `OptimizationDeltaResponse` at all — `mapping`, `translation_applied`, `evidence_source`, `change_type` don't exist.

## CRITICAL: Chat Returns JSON, Not SSE

```ts
// WRONG: spec assumes SSE stream with updated diff

// RIGHT (contract):
POST /v1/resumes/{template_id}/versions/{version_id}/chat
→ 200: ChatResponse

interface ChatResponse {
  session_id: UUID;
  message: string;
  turn_count: number;
  max_turns: number;
  session_closed: boolean;
}
```

**Rewrite iteration flow:**
1. POST chat → get `ChatResponse`
2. Display `message` in ChatBubble
3. Fetch updated deltas separately (or re-trigger optimize)
4. Handle `turn_count / max_turns` — display in chat UI
5. Handle `session_closed: true` — disable further input

## Application Recording Fix

```ts
// WRONG:
{ applied_at: new Date().toISOString() }  // full datetime

// RIGHT:
{ applied_at: new Date().toISOString().split('T')[0] }  // date only: "2026-03-12"
// Also add:
{ application_source: "soaria_direct" }  // missing required field
```

## Other

- `POST .../approve` endpoint doesn't exist — implement client-side approval (just close overlay + show DownloadLink), or file RFC
- Add 502 (LLM call failed, retryable) and 503 (LLM budget exceeded, not retryable) to error paths
- Score delta (`original_score`/`optimized_score`) has no backend source — source pre-opt score from JobDetail, file RFC for post-opt score on `OptimizationResult`
