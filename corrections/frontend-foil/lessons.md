# Frontend Foil — Lessons Log

> Append-only. Each correction or learning from spec reviews gets logged here.
> Periodically distill patterns into rules.md.

## 2026-03-12 — Single Spec Per Agent Dispatch
**Observation:** Batch review (11 specs in one agent) found more wire format issues on backend specs because the prompt framed the task as "review all specs including frontend-facing elements." Per-spec agents classified most backend specs as "no frontend surface" and skipped wire format review.
**Context:** Sloane's per-spec prompt said "if no frontend surface, note it and move on." This is correct for frontend implementation specs but wrong for backend specs that define SSE event shapes and API response contracts the frontend consumes.
**Takeaway:** When reviewing backend specs, the prompt must explicitly say "review wire format contracts for frontend consumption — SSE event shapes, API response models, untyped dicts at rendering boundaries." Otherwise Sloane correctly identifies no frontend code and skips the boundary review that catches the most valuable issues.

## 2026-03-12 — "Previously Reviewed" Is Not "Correct"
**Observation:** Batch Nolan approved Spec 23 because it had "already been through a foil review cycle with 11+ fixes incorporated." Per-spec Nolan verified the fixes against the actual codebase and migration SQL, finding 5 blockers including a missing table, untransferred card_interactions, and a PRD contradiction on cookies.
**Context:** The batch agent trusted the spec's revision history as a proxy for correctness. The per-spec agent actually read the migration DDL and checked whether referenced tables/columns exist.
**Takeaway:** Never approve a spec based on revision history. Always verify fixes landed in the spec text. Check migration SQL against the actual database schema. A spec that says "fixed in v2" may have the fix described in a header comment but not in the actual chunk detail.

## 2026-03-12 — Systematic Patterns Across 10 FE Specs (R2 Review)
**Observation:** Reviewed all 10 frontend specs (post-Faye-revision). All 10 got REVISE. The same 7 issues repeated across most specs: (1) ARIA/focus management missing on every overlay/drawer/dropdown, (2) no mobile layouts anywhere, (3) "create account" instead of locked "save" language, (4) color as only signal for states/severity/change types, (5) reattribution line missing from all specs, (6) no performance targets, (7) fit scores as visual heroes over translations.
**Context:** Faye applied the FOIL_REVIEW_FIXES corrections from round 1 but didn't internalize the cross-cutting patterns. Each spec was written independently, so the same gaps propagated across all 10.
**Takeaway:** After a foil review round, compile cross-cutting findings into a single systematic fixes file (not per-spec). Faye should apply the systematic passes FIRST (ARIA, mobile, brand language, color accessibility, performance targets) then do per-spec fixes. This prevents the same issue appearing in 10 separate review outputs.

## 2026-03-12 — Reattribution Line Has No Owner
**Observation:** The brand review says "You described [X] skills that are in demand. You just used different words" is required post-reveal, pre-capture. Reviewer-BH6 called it "worth more than any feature." It appears in ZERO specs. Every spec that touches the capture funnel was flagged for it independently.
**Context:** The line sits at the boundary between vocabulary-translation-ui (where the reveal happens), auth-capture (where the capture happens), and canvas-engine (where the user first lands post-signup). Nobody owns it.
**Takeaway:** Cross-spec components that sit at boundaries between specs need explicit ownership assignment. Flag to Spencer: vocabulary-translation-ui should own the reattribution component, other specs should reference it.
