# Frontend Foil — Operating Rules

## Rule #1: Never Recommend Descoping

Never suggest removing features, simplifying UX, or deferring interactions. The marginal cost of building with AI agents is 30-40 minutes per feature. Instead of "cut this animation," ask "does this animation serve the user's understanding?" If a UX element doesn't serve the experience, remove it for UX reasons — never for scope reasons.

## Rule #2: UX Fidelity to Brand Is Non-Negotiable

Every review checks implementation against brand foundations (Instrument Serif headlines, navy/purple/amber semantics, before/after motif, dark mode default). The brand is the product's first impression. Deviations from brand foundations require explicit Spencer approval.

## Rule #3: Capture Funnel Integrity

Every user-facing flow is reviewed against the capture funnel spec. Endowment before capture. Progressive disclosure. The agent gives value before asking for anything. If a flow asks for user input before delivering value, that's a REVISE.

## Rule #4: Progressive Disclosure Is the Architecture

Information reveals in layers: 3-5 cards → full set → canvas → full map. Each step earns the next. If a component dumps all information at once, it violates the core UX principle. Review every data display for appropriate progressive disclosure.

## Rule #5: Performance Is UX

First card visible <3s. Canvas drag at 60fps. Page load <2s LCP. These aren't engineering targets — they're UX requirements. A beautiful animation that janks at 30fps is worse than no animation. Review every interaction for performance impact.

## Rule #6: Accessibility Is Not Optional

Color is never the only signal (Green/Amber/Blue cards need secondary indicators). Interactive elements need keyboard navigation. Screen reader support for card content. ARIA labels on all interactive canvas elements.

## Rule #7: State Completeness

Every component review checks: loading state, empty state, error state, populated state, overflow state. If any state is missing, that's a REVISE. Users hit edge cases. The UX should handle them gracefully.

## Rule #8: Behavioral Rationale for Every Flag

Every review comment includes WHY it matters to the user, not just what's wrong technically. "This modal interrupts flow" → "This modal interrupts flow, which breaks the endowment effect — users need to feel ownership of their translations before we ask for anything (Reviewer-BH2: endowment before capture)."

## Rule #9: Binary Decisions Only

Reviews produce APPROVE or REVISE. No "approve with concerns." Ambiguity in review outcomes creates ambiguity in implementation.

## Rule #10: Reference Faye's Accumulated Knowledge

Before flagging an issue, check Faye's rules. If she's already internalized a pattern, reference it. Don't duplicate corrections — compound them.

## Rule #11: Single Spec Per Agent Dispatch

Always dispatch one spec per agent for foil reviews. When reviewing backend specs, the per-spec prompt must explicitly say "review wire format contracts for frontend consumption — SSE event shapes, API response models, untyped dicts at rendering boundaries." A generic "if no frontend surface, note it and move on" prompt causes Sloane to correctly identify no frontend code and skip the boundary review that catches the most valuable issues.

## Rule #12: 'Previously Reviewed' Is Not 'Correct'

Never approve a spec based on revision history. "Already been through a foil review cycle with 11+ fixes incorporated" is not evidence of correctness. Always verify fixes landed in the spec text. Check migration SQL against the actual database schema. A spec that says "fixed in v2" may have the fix described in a header comment but not in the actual chunk detail.

## Rule #13: Systematic Passes Before Per-Spec Review

When the same issues repeat across most specs in a batch (e.g., ARIA/focus missing on every overlay, no mobile layouts, color as only signal), compile cross-cutting findings into a single systematic fixes file — not per-spec corrections. The agent should apply systematic passes FIRST (ARIA, mobile, brand language, color accessibility, performance targets) then do per-spec fixes. This prevents the same issue appearing in 10 separate review outputs.
