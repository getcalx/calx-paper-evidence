# Contract Corrections: Star Stories UI

Source: Nolan (Backend Foil) cross-review

---

## CRITICAL: SSE Wire Type Invented — No Backend Contract

`SSEStarStoryCard` fields don't match `CaseStudyResponse` from types.ts:

| Spec SSEStarStoryCard | Contract CaseStudyResponse |
|----------------------|---------------------------|
| `completeness_score: number` | `is_complete: boolean` |
| `section_scores: {...}` | `needs_more_detail: boolean` |
| `vocabulary_tags: string[]` | `skills_demonstrated: string[]` + `tags: string[]` |
| `linked_translations: string[]` | Does not exist |
| `category: string` | `functional_area: string` (maybe?) |

**Action:** File RFC defining exact `SSEStarStoryCard` payload. Must reconcile with `CaseStudyResponse`. Mark wire types as `/** PENDING RFC */`. Build against mocked SSE only.

PRD says `linked_translations` JSONB column and completeness scoring should be Spec 24 addendum — verify if that addendum exists.

## Card Type Coordination

- SSE `card_type: "star_story"` not coordinated with backend — may emit `"case_study"` instead
- `"story_prompt"` has NO entry in canvas `CardType` union — add it
- Document mapping: SSE `"star_story"` → canvas `"case_study"`, SSE `"story_prompt"` → canvas `"story_prompt"`
- File RFC or contract update for `SSECardEvent.card_type` union expansion

## CaseStudyData Conflict

Spec silently redefines `CaseStudyData` from canvas-engine spec. Changes should live in canvas-engine spec as a revision, not here. Also: `vocabTags` (canvas spec) vs `vocabularyTags` (this spec's mapping target) — pick one.

## Nullable Section Fields

```ts
// WRONG: sections typed as required string
situation: string; task: string; action: string; result: string;

// RIGHT: nullable during progressive capture
situation: string | null; task: string | null; action: string | null; result: string | null;
```

Mapper should default null → `""`. Component checks should use `content.length > 0` not truthiness.

## Pro Feature Gate MISSING

PRD Section 4 + Section 7 feature matrix: STAR story capture is Pro-only, gated behind reverse trial. Spec has ZERO gating logic.

**Add:**
- StoryCapturePrompt: wrap `onAccept` with `useFeatureGate("star_story_capture")`. Show `GatedFeaturePrompt` if gated.
- "Add more detail" button: same gate check.
- Document that agent-side gating handles free users who type story requests in chat.

## Other

- StoryConnectors are edges in all but name — note in canvas-engine spec that "no edges" claim is now false
- Reverse lookup (TranslationCard → StoryCards) unspecced — add `Map<string, string[]>` to CanvasContext
- `mapStoryCapturePrompt` drops `translation_card_id` — add `translationCardId` to props for canvas positioning
