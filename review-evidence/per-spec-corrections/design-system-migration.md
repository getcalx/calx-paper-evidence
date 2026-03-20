# Foil Review: Spec A — Design System Migration
**Date:** 2026-03-14
**Reviewers:** Sloane (foil), Vivian (design), Reviewer-DS1, Reviewer-DS2, Reviewer-DS3
**Verdict:** REVISE

---

## Blockers

### 1. Contrast failure: `--text-ite-dim` on `--bg-basalt` (Sloane)
#6B6862 on #141316 = ~3.0:1. Fails WCAG AA for normal text. Footer signature uses this at `text-sm` (14px). Either lighten to ~#807C76 or restrict to large text (18px+/24px bold).

### 2. Nav dissolution inaccessible to keyboard users (Sloane)
Dissolved nav (opacity 0, pointer-events none) is permanently inaccessible via keyboard. No mechanism for keyboard-based reappearance. Add `focus-within` clause + `aria-hidden` toggle. Add skip-to-content link.

### 3. No `prefers-reduced-motion` on dissolution (Sloane + Reviewer-DS2)
Wrap dissolution transition in `@media (prefers-reduced-motion: reduce)` — disable entirely or instant show/hide.

---

## Must-Fix

### 4. Shadow tokens don't match brand-identity-v2 (Vivian + Sloane)
Spec: `0 4px 12px` / `0 8px 24px`. V2: `0 2px 8px` / `0 6px 20px`. Match v2 values exactly.

### 5. No spacing scale (Reviewer-DS1 + Vivian + Reviewer-DS2 + Sloane — 4/5 consensus)
Add spacing tokens from v2: `--space-unit: 8px`, `--space-card-padding: 24px`, `--radius-card-marketing: 16px`, `--space-section: 80px`, etc. Without these, every downstream spec hardcodes pixel values.

### 6. No type scale (Reviewer-DS1)
Define `--text-xs` through `--text-4xl` as CSS custom properties. Every font-size in B-E should reference scale steps, not pixel values.

### 7. Add Oxidized tokens to Tailwind theme (Reviewer-DS2)
`bg-[var(--bg-basalt)]` everywhere is DX friction. Add tokens to theme config: `bg-basalt`, `text-limestone`, etc. Cost: few lines. Payoff: hundreds of cleaner class names across B-E.

### 8. Split Nav into server/client components (Reviewer-DS2)
Home variant ships scroll listeners + useState to every page. `NavStatic` (server, zero JS) for non-home. `NavHome` (client) for home only.

### 9. Add performance budgets (Reviewer-DS2 + Sloane)
LCP < 1.5s, CLS = 0, total JS for marketing layout < 15KB gzipped.

### 10. Token cross-contamination enforcement (Reviewer-DS1)
Namespace Oxidized tokens under `.ox` class or CSS layer, or add ESLint rule preventing Warm Edge usage in marketing components and vice versa.

### 11. Font weight 900→800 audit (Sloane)
Verify which components use `font-black`. Document the visual delta.

### 12. `<section>` elements need aria-labels (Sloane)
Each replacement `<section>` should have `aria-labelledby` pointing to its heading.

---

## Design Notes (non-blocking)

- **Reviewer-DS3:** Geist is locked in v2. Noted concern about generic feel. Bridge should be visual signature, not color alias — consider bridge token group (width, curvature, animation, endpoint-radius).
- **Vivian:** Card stagger animation killed without replacement noted. Add out-of-scope note that stagger re-implemented in B-E.
- **Vivian:** Flag to Spencer — brand-review.md amber/purple panels vs v2 Graphite-on-Graphite. **RESOLVED: v2 supersedes. Graphite-on-Graphite.**
