# Foil Review: Spec B — Homepage Rebuild
**Date:** 2026-03-14
**Reviewers:** Sloane (foil), Vivian (design), Reviewer-DS1, Reviewer-DS2, Reviewer-DS3
**Verdict:** REVISE

---

## Blockers

### 1. Capture funnel conflict (Sloane)
Hero uses existing PasteInput (paste → redirect). Capture funnel spec defines a 3-step conversational flow (role title → JD paste → resume paste). **Spencer resolved: capture funnel spec is the authority.** Align the homepage to the capture funnel or explicitly mark PasteInput as placeholder pending capture funnel build.

### 2. VocabMoment has zero screen reader semantics (Sloane)
No ARIA labels distinguishing candidate from recruiter side. Typography-as-label works visually but not for assistive tech. Add `aria-label="Candidate language"` / `aria-label="Recruiter language"` on containers, or wrap in `role="figure"` with `aria-label="Vocabulary translation example"`.

### 3. PRD traceability broken (Sloane)
Traces to PRD Sections 13, 15.3, 16, 17. **PRD has 11 sections. These don't exist.** Fix references.

---

## Must-Fix

### 4. Font loading strategy missing (Reviewer-DS2 + Sloane)
Three custom fonts, no preload strategy. Hero headline in Instrument Serif IS the LCP element. If fonts FOIT, VocabMoment renders as two identical text blocks — destroys the typographic differentiation. Ensure `next/font` with `display: 'swap'` and preload for serif font.

### 5. No performance budgets (Reviewer-DS2 + Sloane + Reviewer-DS1)
Add: LCP < 1.0s, CLS = 0, TBT < 50ms. Page is SSG with near-zero JS — targets should be aggressive.

### 6. VocabMoment font sizes too small (Vivian)
20px/16px reads as body copy. Bump to 28px/20px desktop, 20px/16px mobile. The VocabMoment is the hero moment — it needs scale.

### 7. `prefers-reduced-motion` on smooth scroll (Sloane)
Wrap `html { scroll-behavior: smooth; }` in `@media (prefers-reduced-motion: no-preference)`.

### 8. "translate" link below 44px touch target (Sloane)
15px text with no padding = ~18px height. Add padding to reach 44px minimum.

### 9. Mobile font minimum (Sloane + Vivian)
VocabMoment small variant: 13px JetBrains Mono on dark background is below 14px brand minimum. Raise to 14-15px.

### 10. 680px not tokenized (Reviewer-DS1)
Content width appears in 3+ places across specs. Add `--content-width-reading` token in Spec A.

### 11. Smooth scroll is global (Reviewer-DS2)
Scope to homepage or specific container. Global `scroll-behavior: smooth` can interfere with App Router scroll restoration.

### 12. Reattribution line unowned (Sloane)
Add Out of Scope note documenting that the reattribution line belongs to the capture funnel / translate page spec.

---

## Design Notes

- **Vivian:** Block 3 category declaration (Instrument Serif 28px) competes with hero (Instrument Serif 56px). Consider switching Block 3 to Geist 500 for tonal separation. Spencer's call.
- **Vivian:** 62% stat needs more vertical isolation — 120px gap before Block 2 instead of 80px.
- **Reviewer-DS2:** Page should explicitly declare `force-static` or note SSG. Use container queries on VocabMoment instead of media queries for reuse portability.
- **Reviewer-DS3:** Terracotta seam should become Versed's signature element across every touchpoint. Font pairing is category-defining if committed to fully.
- **Reviewer-DS1:** Extract `TranslationPair` as shared type. Use existing `StatBlock` component or kill it.
