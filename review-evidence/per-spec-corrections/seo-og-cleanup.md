# Foil Review: Spec E — SEO + OG Cleanup
**Date:** 2026-03-14
**Reviewers:** Sloane (foil), Vivian (design), Reviewer-DS1, Reviewer-DS2, Reviewer-DS3
**Verdict:** REVISE

---

## Blockers

### 1. Font loading syntax error on line 111 (all reviewers)
```ts
// BROKEN:
fetch(new URL('/public/fonts/JetBrainsMono-Regular.woff', r => r.arrayBuffer())),
// FIXED:
fetch(new URL('/public/fonts/JetBrainsMono-Regular.woff', req.url)).then(r => r.arrayBuffer()),
```

### 2. 2x/1x render contradiction (Sloane + Reviewer-DS1 + Reviewer-DS2)
Layout section uses 2x values (2400px). ImageResponse config uses 1x (1200px). Pick one coordinate system. Recommendation: 1x values in JSX, `width: 1200` in config. Remove all 2x references.

### 3. Wrong dependency spec name (Sloane)
Header says "Depends on: Spec A (token-alignment)." Should be "Spec A (design-system-migration)."

### 4. Phantom PRD references (Sloane)
Sections 15, 16, 19, 20 don't exist. Fix traceability to actual source docs.

### 5. Geist font weight mismatch (Sloane)
Geist-Regular.woff registered at weight 500. Regular is 400. Either use Geist-Medium.woff or register at 400.

### 6. TypeScript return type error (Sloane)
Error paths return `new Response(...)` but function declares `Promise<ImageResponse>`. Fix to `Promise<ImageResponse | Response>`.

### 7. Missing Spencer Config section (Sloane)
List: `@vercel/og` (pin version), font file sources + versions, Edge Runtime requirement.

---

## Must-Fix

### 8. Static OG images are forgettable (Reviewer-DS3 + Vivian)
Text on dark rectangle. No translation visual, no terracotta. The card share image proves Versed has a visual identity — static OGs should use it. Add translation card element to homepage + employer OGs. Add terracotta accent to every OG surface.

### 9. Wordmark treatment (Vivian + Reviewer-DS3)
- Inconsistent position across OG surfaces (centered vs bottom-right vs absent). Standardize to bottom-right.
- Too small: 14px effective disappears at thumbnail scale. Bump to 20px.
- Use Limestone or Terracotta, not Ite Dim (#6B6862).

### 10. Cap dynamic height at 630px (Vivian)
LinkedIn/Twitter prefer 1200x630. Dynamic heights get cropped unpredictably. Truncate pairs beyond 3 with "+N more" indicator.

### 11. Return fallback OG image on errors (Reviewer-DS2)
400 text response = broken share preview. Return generic "Versed AI" fallback image on error instead.

### 12. Pin `@vercel/og` version (Reviewer-DS2)
Breaking changes between minor versions. Pin explicitly.

### 13. Use title template (Reviewer-DS2)
Per-page titles hardcode `| Versed AI` suffix. Use template, set just page-specific title.

### 14. Alt text is placeholder (Vivian)
Use headline text as alt: "Optimizing your resume is why you're getting rejected — Versed AI"

### 15. Font sourcing strategy (Sloane + Reviewer-DS2)
.woff files must match `next/font` versions. Add exact download URLs or version pins. Verify OG font rendering matches live site.

### 16. Card share image drops card state (Sloane)
Add optional `state` URL parameter. Render state indicator when provided.

---

## Design Notes

- **Reviewer-DS1:** Extract card share API (Section 3 + Chunk 4) into its own spec. It's a product feature masquerading as cleanup.
- **Reviewer-DS3:** The card share image is an untapped distribution tool. Needs a UI surface ("Share this translation" button) to drive traffic.
- **Reviewer-DS2:** `opengraph-image.tsx` convention could replace static PNGs for simpler maintenance. Tradeoff: more build-time cost.
- **Reviewer-DS2:** Structured data validation in CI would catch regressions automatically.
- **Vivian:** Verify terracotta seam legibility at 600px wide (social card thumbnail scale).
