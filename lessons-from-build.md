# Lessons From Build — Process Patterns
**Date:** 2026-03-17
**Source:** Full-system audit retrospective

## Pattern 1: Spec Boundary Problem
**What happened:** Spec 34 (Tier 2 classification) was scoped as backend-only. It implemented the classification engine and was reviewed by backend foils. The FE impact was noted but never turned into an FE engineering spec. Result: 40-65% of expected users get a 3-line handler instead of the full experience.

**Root cause:** The coordination layer tracked BE specs but didn't systematically derive FE specs from PRD sections with BE dependencies.

**Fix:** After every BE spec is locked, explicitly check: "Does this spec's output change what the user sees?" If yes, create FE spec before marking the feature as done.

**Applies to:** Reid (coordination), all future spec writing

## Pattern 2: Per-Spec Dispatch Superiority
**What happened:** Batch review (6 specs, 1 agent) found 29 issues. Per-spec dispatch (6 specs, 6 agents) found 72 issues on the same specs. 2.5x improvement.

**Root cause:** Agent context saturation. With 6 specs loaded, the reviewer can't hold the full detail of each one. With 1 spec, the reviewer goes deep.

**Fix:** Always dispatch one agent per spec for reviews. Never batch. Spend the tokens.

**Already codified:** feedback-per-spec-dispatch.md, feedback-one-agent-per-deep-dive.md

## Pattern 3: Cross-Domain Foil Catches Boundary Failures
**What happened:** Nolan (BE foil) reviewing FE specs found 60 boundary issues. Sloane (FE foil) reviewing the same specs found 0 of those issues.

**Root cause:** Same-domain reviewers share blind spots. Cross-domain reviewers ask "what happens when my system sends X and yours expects Y?"

**Fix:** Every spec gets both domain foil + cross-domain foil review.

**Already codified:** lesson-foil-cross-review.md

## Pattern 4: Context Window Split for Large Tasks
**What happened:** When an agent needs to read a large reference doc AND produce a large output, quality drops because context is saturated.

**Fix:** Split into Extract agent (reads everything, produces structured summary) and Apply agent (reads summary, produces output). Decouples verification from revision.

**Already codified:** lesson-context-window-splits.md

## Pattern 5: Document Contradictions Accumulate Silently
**What happened:** strategy.md still says STAR stories are Pro-gated and references "Oxidized palette" — both overridden by Spencer's corrections months ago. Footer says "Soaria, Inc." Code has [redacted-email].

**Root cause:** Corrections happen in conversation. Strategy docs don't get updated in the same session. Over time, the document drifts from reality.

**Fix:** When a decision overrides a strategy doc, update the strategy doc in the same session. Don't defer.

**New lesson for Reid.**

## Pattern 6: Copy Decisions Are Product Decisions
**What happened:** "Create account" vs "Save my translations" on the signup button. "Going live March 23" modal vs routing to /signup. These aren't cosmetic — they determine whether the capture funnel converts.

**Root cause:** Copy was treated as a polish task rather than a product decision. No spec covered CTA copy.

**Fix:** CTA copy and conversion-path copy belong in the PRD or FE spec, not as afterthoughts.

## Pattern 7: Building-Shipping Asymmetry
**What happened:** 43 specs, 28 migrations, ~1.9B tokens of work. 0 users. The rigor is real and will compound post-launch — but only if launch happens.

**Root cause:** The spec pipeline is well-oiled. The "just ship it" muscle is undertrained.

**Fix:** March 24 is the forcing function. After launch, measure spec-to-ship latency explicitly.
