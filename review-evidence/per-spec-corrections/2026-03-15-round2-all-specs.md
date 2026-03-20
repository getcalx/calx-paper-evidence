# Foil Re-Review Round 2: All FE Specs (A-E)

**Date:** 2026-03-15
**Reviewers:** Nolan (backend foil), Sloane (FE foil), Vivian (design), Reviewer-BH6 (affective), Reviewer-SY4 (ML), Reviewer-DS2 (performance)
**36 dispatches across 6 specs (Spec 32 + Specs A-E)**

---

## Spec A — Design System Migration

**Verdict: 3 APPROVE (Vivian, Reviewer-BH6, Reviewer-SY4), 3 REVISE (Nolan, Sloane, Reviewer-DS2)**
**All issues are in NavHome. Everything else is clean.**

### BLOCKING

**A-B1: `prefersReducedMotion` is dead code in NavHome** (Nolan, Sloane, Reviewer-DS2 — all three caught independently)
The variable is declared inside `useEffect` but never used in the JSX. The `transition` style prop always applies 300ms animation regardless of user preference. Accessibility failure.

Fix: Promote to `useState`, wire into style prop:
```tsx
transition: reducedMotion ? 'none' : dissolved ? 'opacity 300ms var(--ease-in)' : 'opacity 300ms var(--ease-out-expo)'
```

**A-B2: `aria-hidden` conflicts with `focus-within`** (Sloane)
When nav is dissolved, `aria-hidden={true}` is set. But `focus-within` CSS makes it visible and interactive. Screen reader users interact with content their AT says doesn't exist. WCAG violation.

Fix: Toggle `aria-hidden` to `false` when nav has focus-within. Requires JS — add focus/blur listener on nav ref.

### MUST-FIX

**A-M1: Nav dissolution scroll hysteresis missing** (Sloane)
Nav flickers on/off with minor scroll direction changes above 100px threshold. Touch scrolling on mobile will cause rapid flickering.

Fix: Add hysteresis — require 30px+ upward scroll before reappearing, or only reappear when scrollY returns below 100.

### SHOULD-FIX

**A-S1: Geist variable font loading** (Reviewer-DS2)
Explicit `weight: ['400', '500', '700', '800']` array may force 4 static font files instead of 1 variable font. Verify and remove if Geist resolves as variable font.

### ADVISORY

- Add `--duration-reveal` / `--duration-card-enter` timing tokens (Reviewer-BH6)
- Add terracotta AA contrast note — passes only at large text sizes (Reviewer-BH6)
- Design spec §0 shadow values stale: `0 4px 12px` should be `0 2px 8px` per brand-identity-v2 (Vivian, Sloane, Nolan — all flagged)

---

## Spec B — Homepage Rebuild

**Verdict: 3 APPROVE (Nolan, Reviewer-SY4, Reviewer-DS2), 3 REVISE (Sloane, Vivian, Reviewer-BH6)**

### BLOCKING

**B-B1: Hero font sizes undersized** (Vivian)
Spec: 40/48/56px. Design spec §2: 48/56/64px. The hero headline is the LCP element and the emotional hook.

Fix: Change to 48px mobile / 56px tablet / 64px desktop.

**B-B2: Hero h1 missing max-width** (Vivian)
Design spec says 600px max-width on the headline. Spec uses full 680px column. At 680px, line breaks change and visual tension weakens.

Fix: Add `max-width: 600px` on the h1 element itself.

**B-B3: Headline-to-input gap too tight** (Vivian)
Spec: 32px (`--space-8`). Design spec: 48px. The headline needs room to breathe before PasteInput.

Fix: Increase to 48px.

**B-B4: Block 2 source attribution missing** (Sloane, Reviewer-SY4)
PRD §13 explicitly requires "Source attribution: ResumeBuilder, 2025." Spec omits it. An unattributed stat on a brand-new homepage undermines credibility.

Fix: Add `<cite>` element below Block 2 context copy: "Source: ResumeBuilder, 2025" in `--text-ite` at `--text-xs`.

**B-B5: No heading hierarchy below the fold** (Sloane)
Page goes h1 → nothing. Screen reader users navigating by heading skip all below-fold content.

Fix: Add visually-hidden `<h2>` to each block: "The Vocabulary Gap" / "62% rejection rate" / "Career translation platform" / "Try your translation".

**B-B6: Block 2 `aria-labelledby` points at "62%" — meaningless accessible name** (Sloane)
Screen reader user hearing "region: 62%" gets zero context.

Fix: Point `aria-labelledby` at a visually-hidden heading instead of the stat `<p>`.

### MUST-FIX

**B-M1: Add bridge line above PasteInput** (Reviewer-BH6)
The somatic gap between the provocation headline and the raw paste box is the spec's biggest affect failure. Even placeholder copy prevents arousal from dissipating.

Fix: Add interim text above PasteInput: "Paste a job description. See the gap." in Geist 400 at `--text-base`. Mark for replacement when capture funnel ships.

**B-M2: Increase vertical space before Block 2 to 120px** (Reviewer-BH6)
The emotional transition from recognition to betrayal needs a longer beat. 80px is too tight.

Fix: Use `var(--space-30)` (120px) before Block 2 specifically.

**B-M3: PasteInput "Create free account" copy** (Nolan)
Rate-limited state shows "Create free account" — brand violation. Will be newly prominent on rebuilt homepage.

Fix: Change to "Save with email" per cross-cutting §3. One-line change in PasteInput.tsx.

### ADVISORY

- VocabMoment line-height: 1.5 vs design spec 1.6 on recruiter text (Sloane, Vivian)
- "That's the gap" top margin could be 40px instead of 32px (Reviewer-BH6)
- Translation pair example could be more specific for stronger recognition (Reviewer-BH6)

---

## Spec C — /for-employers Rewrite

**Verdict: 5 APPROVE (Nolan, Vivian, Reviewer-BH6, Reviewer-SY4, Reviewer-DS2), 1 REVISE (Sloane)**
**Sloane caught real implementation bugs.**

### BLOCKING

**C-B1: `--transition-fast` token does not exist** (Sloane)
CTA button uses `duration-[var(--transition-fast)]` — not defined in Spec A or brand-identity-v2. Produces `transition-duration: ;` (instant, not smooth).

Fix: Replace with `duration-150` to match Spec A's pattern. Remove from token gate list.

**C-B2: CTA tag mismatch — `<h2>` opens, `</p>` closes** (Nolan, Sloane)
Will cause React hydration error and broken DOM.

Fix: Change `</p>` to `</h2>` at line 391.

### MUST-FIX

**C-M1: Card uses Warm Edge tokens** (Sloane)
`var(--text-primary)` and `var(--text-secondary)` are Warm Edge, not Oxidized. Marketing components must use `var(--text-limestone)` and `var(--text-ite)`.

**C-M2: Card border-radius 12px should be 16px** (Sloane)
`rounded-xl` = 12px. `brand-identity-v2` says 16px for marketing cards. Use `rounded-2xl` or `rounded-[var(--radius-card-marketing)]`.

**C-M3: Section 2 (cards) missing `<section>` wrapper and heading** (Sloane)
Every other section has `aria-labelledby`. Card group doesn't. Add `<section aria-labelledby="examples-heading">` with sr-only `<h2>`.

**C-M4: Paste section heading hierarchy broken** (Sloane)
Uses `<p>` as `aria-labelledby` target. Screen reader users skip it when navigating by heading.

Fix: Change to `<h2>` with same visual styling, or add separate sr-only `<h2>`.

### SHOULD-FIX

- Hardcoded font sizes instead of type scale tokens (Sloane)
- Shadow values hardcoded instead of `--shadow-card-ox` tokens (Sloane)

### ADVISORY

- Product descriptions (Section 4) cool the emotional arc between demo and CTA (Reviewer-BH6, Reviewer-DS3) — Spencer decides
- Bridge glow treatment not specified — Faye should include per design spec §6 (Vivian)
- Design spec §17 needs update: heading 16px→20px, CTA "analysis"→"translation" (Vivian)

---

## Spec D — /vocabulary-gap Page

**Verdict: 3 APPROVE (Nolan, Reviewer-DS2, Vivian-conditional), 2 REVISE (Reviewer-BH6, Reviewer-SY4), 1 pending (Sloane)**

### BLOCKING

**D-B1: 62% stat needs source attribution** (Reviewer-SY4)
An editorial reference page without citations is an opinion piece. Add "ResumeBuilder, 2025" wherever the stat appears.

**D-B2: "Optimization" in translation example** (Reviewer-SY4)
"Fixed the onboarding flow" → "User activation funnel optimization" — uses the enemy word as a positive translation output on the page that poisons that word. Brand logic error.

Fix: Replace with "User activation and retention engineering" or similar.

### MUST-FIX

**D-M1: Add emotional arc annotations to section placeholders** (Reviewer-BH6)
Sections have topics but not affective targets. Writers get an explainer, not recognition-evoking content.

Fix: Add emotional target per section (recognition, relief, anger, clarity, belonging, agency).

**D-M2: FAQ needs 2-3 second-person questions** (Reviewer-BH6)
All 6 questions are clinical third-person. "Why do qualified candidates get rejected?" → "Why do I keep getting rejected from jobs I'm qualified for?" The pronoun shift is the entire affective difference.

**D-M3: Section 3 needs "externalize blame" editorial constraint** (Reviewer-BH6)
Without it, copy writer may frame as "you shouldn't have optimized" instead of "the tools failed you."

**D-M4: CTA background — Shale should be Graphite** (Vivian)
Design spec §18 says Graphite. Spec says Shale. Use `var(--bg-graphite)` for consistency with FAQ section.

**D-M5: Reframe "overfitting detection"** (Reviewer-SY4)
Imprecise ML terminology presented as established mechanism. Either reframe as "pattern recognition" or define as Versed's term.

### ADVISORY

- Design spec §18 needs sync: Instrument Serif CTA, VocabMoment margin, pullquote glow (Vivian)
- Layout file contradiction in Build Chunks vs Route Setup — follow Route Setup (Nolan)
- FAQ Q6 "requires comprehension" → soften to "maps meaning across" (Reviewer-SY4)
- Pullquote emotional register guidance for copy writer (Reviewer-BH6)

---

## Spec E — SEO + OG Cleanup

**Verdict: 5 APPROVE (Nolan, Reviewer-BH6, Reviewer-SY4, Reviewer-DS2, Sloane), 1 APPROVE w/ conditions (Vivian)**
**Cleanest spec after review.**

### NO BLOCKING ISSUES

### MUST-FIX (design spec sync only)

**E-M1: Design spec §19 + §20 need updating** (Vivian, Sloane, Nolan)
Multiple foil-reviewed decisions (wordmark 20px Limestone, translation card visual in OGs, 1x rendering) not reflected in the design spec. The engineering spec is correct; the design spec is stale.

### SHOULD-FIX

- Clarify Geist 400 vs 500 for card share role title — Spencer decides (Vivian)
- Use exact version pin `0.6.4` not `^0.6.4` (Sloane)
- Change "Consider caching" fonts to "Cache fonts in module scope" (Reviewer-DS2, Sloane)
- Add redirect test for /about → / (Sloane)
- Wire fallback.png into root layout or remove from Chunk 2 (Sloane)
- Add Amazonbot to robots.ts, `speakable` on vocab-gap WebPage, `knowsAbout` on Organization schema (Reviewer-SY4)

---

## Cross-Cutting Issues (Hit Multiple Specs)

### CC-1: Design spec is stale in 6+ locations
- §0: Shadow values `0 4px 12px` → `0 2px 8px`
- §17: CTA "analysis" → "translation", heading 16px → 20px
- §18: VocabMoment margin `40px auto` → `40px 0`, Instrument Serif CTA
- §19: Wordmark 14px Ite Dim → 20px Limestone
- §20: Text-only OGs → translation card visual + terracotta accent

Reid should do a systematic design spec update pass.

### CC-2: PasteInput "Create free account" brand violation
Appears on both homepage and /for-employers in rate-limited state. Fix once in PasteInput.tsx.

### CC-3: NavHome is the only broken component
Every reviewer who touched it flagged the same bugs. Fix once, all pages benefit.
