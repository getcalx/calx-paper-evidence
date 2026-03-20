# Foil Review: Spec C — /for-employers Rewrite
**Date:** 2026-03-14
**Reviewers:** Sloane (foil), Vivian (design), Reviewer-DS1, Reviewer-DS2, Reviewer-DS3
**Verdict:** REVISE

---

## Blockers

### 1. Token names wrong — will cause build failures (Sloane + Vivian + Reviewer-DS2 — 4/5 consensus)
- `var(--color-terracotta)` → `var(--accent-terracotta)`
- `var(--color-verdigris)` → `var(--state-verdigris)`
- `var(--color-ochre)` → `var(--state-ochre)`
Match Spec A's actual token names.

### 2. Bridge seam gradient contradicts Oxidized (Vivian)
Bridge is always solid Terracotta, not a state-color-to-purple gradient. The bridge represents translation — it doesn't change per state. State color belongs on the left accent indicator.

### 3. Panel tinted backgrounds contradict Oxidized (Vivian)
Brand-identity-v2: "Both columns sit on the same Graphite card surface. Differentiation is entirely typographic." Remove the rgba tints on panels. **Spencer confirmed: Graphite-on-Graphite.**

### 4. No mobile layout for EmployerTranslationCard (Sloane + Reviewer-DS1)
Must specify: breakpoint (768px), stacking behavior, horizontal bridge seam, touch targets.

### 5. Nav section is dead code (Reviewer-DS2 + Sloane)
Spec A already defines Nav variants with props. Delete entire Nav section + Chunk 3. Page just renders `<Nav variant='for-employers' />`.

### 6. Capture funnel absent (Sloane)
Recruiter paste redirects to candidate-framed app. **Spencer resolved: capture funnel spec is authority.** Either scope paste as employer-framed inline results, or mark as known deviation with documentation.

---

## Must-Fix

### 7. EmployerTranslationCard must be server component (Reviewer-DS2)
Static cards, no state, no effects. Explicitly state no `'use client'`.

### 8. Extract CardShell primitive (Reviewer-DS1)
Both TranslationCard variants share the same structural skeleton (two-panel grid, bridge seam, bottom row). Extract `CardShell` before building EmployerTranslationCard. 30 min now vs refactor later.

### 9. Card copy length imbalance (Vivian)
Card 1 candidate reading is 26 words vs 9 on JD side. Trim candidate readings to 8-15 words for visual balance.

### 10. Product heading sizes too small (Vivian)
16px heading vs 15px body = invisible hierarchy on dark background. Bump to 18-20px.

### 11. Label font size 11px below dark-mode minimum (Vivian + Sloane)
Brand minimum is 14px on dark backgrounds. "YOUR JD SAYS" at 11px uppercase is too small. Use 12-13px minimum.

### 12. Spec A token gate (Reviewer-DS1)
Add explicit precondition to Chunk 1: verify required tokens exist in globals.css. List exact tokens. Fail loudly if missing.

### 13. Accessibility gaps (Sloane)
- Card needs `<article>` wrapper, ARIA labels
- Terracotta label text at 11px on Graphite fails WCAG AA (3.8:1). Increase size or use Limestone text with Terracotta accent indicator.

### 14. PasteInput "Analyzing..." text (Sloane)
Brand-identity-v2 anti-patterns: loading copy says "Translating..." never "Analyzing..." Fix or flag as separate cleanup.

---

## Design Notes

- **Reviewer-DS1:** Move static example cards ABOVE the paste input. Show the problem before handing the tool.
- **Reviewer-DS3:** Section 4 product descriptions actively hurt the page. Cut or radically redesign.
- **Reviewer-DS3:** EmployerTranslationCard is the single most brand-distinctive element. Make it the visual centerpiece, not three items in a list.
- **Reviewer-DS3:** "FILTERS" state is a category-defining UI pattern. Build marketing around it.
- **Reviewer-DS2:** PasteInput redirect 400ms setTimeout is arbitrary. Cut or reduce.
