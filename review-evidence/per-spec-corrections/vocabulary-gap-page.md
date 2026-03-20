# Foil Review: Spec D — /vocabulary-gap Page
**Date:** 2026-03-14
**Reviewers:** Sloane (foil), Vivian (design), Reviewer-DS1, Reviewer-DS2, Reviewer-DS3
**Verdict:** REVISE

---

## Blockers

### 1. "Career intelligence" in FAQ + JSON-LD (Sloane)
Wrong category language on the page designed for LLM extraction. Must be "career translation platform" / "agentic career fluency." Fix in:
- FAQ Question 5 answer
- FAQPage JSON-LD
- `articleSection` (change to "Career Fluency" or "Career Translation")
- Both `DefinedTermSet.name` entries

### 2. Token mapping table maps Oxidized names to Warm Edge hex values (Sloane)
Every color in the mapping table is wrong. `--text-limestone` mapped to `#F5F0EB` (Warm Edge) instead of `#E8E2DA` (Oxidized). `--accent-terracotta` mapped to `#D97706` (amber) instead of `#C4654A` (terracotta). Fix entire table to Spec A's actual values.

### 3. Background color conflict (Sloane)
Spec says "Basalt (`var(--bg-surface)` / `#1C1C1E`)" but Basalt is `#141316`. Use `var(--bg-basalt)` with correct hex.

---

## Must-Fix

### 4. VocabMoment prop: `variant` vs `size` (Reviewer-DS1 + Sloane)
Spec B defines `size?: 'default' | 'small'`. This spec uses `variant="small"`. Fix to `size="small"`.

### 5. CSS custom property fallbacks (Reviewer-DS1)
Every Spec A token reference must use fallback syntax: `var(--bridge, var(--color-candidate))`. Not comments. Comments don't prevent broken styles.

### 6. Say SSG, not SSR/SSG (Reviewer-DS2)
This is a static page. Zero dynamic data. Say "SSG" explicitly. Set `force-static` or note automatic static generation.

### 7. Single-source FAQ data (Reviewer-DS2 + Reviewer-DS1)
Hard requirement: `FAQ_DATA` constant drives both `<dl>` rendering AND FAQPage JSON-LD. Not a suggestion. Put in `src/lib/constants.ts`.

### 8. OG image is a launch gate (Reviewer-DS1)
Metadata references `vocabulary-gap.png` which doesn't exist. Either make OG image a blocking prerequisite or remove the image field until it exists. Shipping a 404 in structured data is an error.

### 9. Terracotta vs Amber color confusion (Vivian + Sloane)
Pullquotes and CTA use `var(--accent-terracotta)` mapped to `#D97706` (amber). Marketing pages use Oxidized: Terracotta is `#C4654A`. Resolve consistently.

### 10. PRD references fabricated (Sloane)
Sections 14.2, 15.2, 15.3, 17 don't exist. Fix traceability.

### 11. Performance targets missing (Sloane + Reviewer-DS2)
Add: LCP < 2s, CLS < 0.05, VocabMoment visible without JS.

---

## Design Fixes

### 12. Wall of text on dark (Vivian)
2000 words at 17px/24px margin-bottom is too tight for dark backgrounds. Increase paragraph spacing to 32px. Add section dividers (thin Terracotta accent marks above H2s). Minimum 3 pullquotes across 6 sections.

### 13. VocabMoment alignment (Vivian)
Left-align within the left-aligned column (`margin: 40px 0` not `40px auto`). Centered elements in left-aligned flow create visual wobble.

### 14. CTA too quiet (Vivian + Reviewer-DS3)
After 2000 words, the CTA whispers. Give "translate" a ghost button treatment (Terracotta border pill). Use Instrument Serif for CTA copy to create bookend effect.

### 15. FAQ needs visual separation (Vivian + Reviewer-DS3)
Different background treatment (full-bleed Graphite/Shale) or stronger divider. FAQ is functionally different content — it should look different.

### 16. H2 differentiation (Vivian)
28px Instrument Serif at same color as body text won't scan as section markers. Add letter-spacing shift or thin Terracotta underline.

### 17. Simplified nav should NOT be deferred (Reviewer-DS3)
Editorial page with full marketing nav reads as sub-page. Wordmark + "translate" reads as authoritative content. The nav frames the content's authority.

### 18. Mobile VocabMoment recruiter font too small (Vivian)
13px JetBrains Mono on dark violates 14px brand minimum. Raise to 14px.

---

## Design Notes

- **Reviewer-DS3:** Pullquote treatment is off-the-shelf. Use VocabMoment visual language (terracotta seam as left accent with glow). Make pullquotes feel like Versed pullquotes, not generic pullquotes.
- **Reviewer-DS2:** Left-aligned layout needs justification. If no plan for right column (TOC, sticky CTA), center the article. Empty asymmetric whitespace signals incompleteness.
- **Reviewer-DS1:** Left-alignment as editorial identity should be codified: editorial pages left-align, marketing pages center.
