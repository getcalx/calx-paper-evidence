# Fundraising Lead — Lessons

Append-only log of corrections and learnings. Each entry captures what went wrong, the correction, and the takeaway.

---

## 2026-03-11 — "Says Pre-Seed, Gates on Traction" Pattern
**Correction:** Spencer flagged that both Fund A and Fund B passed on Versed when it was pre-product — despite both marketing as pre-seed/conviction-first investors. Fund A said "not enough market demand signal." Hustle Fund said "our sweet spot is when the product exists." Fund B partner never opened investor updates post-pass (confirmed via email engagement tracking).
**Context:** Spencer pitched both funds when Versed was a Replit loop and a presentation deck. He acknowledges it was very early and the company is far more compelling now. He considers both good sparring partners — no bitterness. But the pattern is real: funds that market as "people over product" or "pre-seed conviction" still evaluate on traction signals. Their marketing thesis and their actual evaluation criteria diverge.
**Takeaway:** When evaluating investor fit, don't trust stage marketing. Ask: "Walk me through a recent deal where you invested pre-product." If they can't name one, they're a Fund B partner/Fund A partner pattern — deprioritize for pre-product pitches, re-engage when traction exists. Use the Fund B partner archetype profile (`~/brain/training/fundraising/investor-deep-dives/nichols-hustlefund.md`) to pattern-match other investors who might behave this way. Important nuance: a prior pass doesn't mean a permanent pass — companies evolve, and pre-seed investors expect that. Re-engagement is valid when the story has genuinely changed.

## 2026-03-11 — Product Briefs Are Not Engineering Specs
**Correction:** Spencer discovered that the canvas had a product brief (`~/brain/docs/product/briefs/canvas.md`) and extensive UX flow documentation, but no engineering spec — neither backend (spatial persistence) nor frontend (build sequence, component contracts). Bryce explicitly flagged canvas as a gap in the spec series. The brief describes WHAT to build; it doesn't tell Bryce or Faye HOW to build it or in what order.
**Context:** The confusion arose because the product-level scoping was thorough (canvas brief, UX flow spec with 6,400+ lines, PRD sections). But none of this was translated into numbered engineering specs like Bryce's 20-25 series. Faye would look at the UX flow doc and see 6 card types, recruiter canvas, force-directed layout, progressive reveal across 9 phases — and not know which subset to build first or what APIs to call.
**Takeaway:** A product brief is an INPUT to engineering spec work, not a substitute for it. No feature enters a build queue without a numbered engineering spec from Bryce (backend) and/or Faye (frontend). The rule: brief → engineering spec → foil review → build. Never skip the middle step.
