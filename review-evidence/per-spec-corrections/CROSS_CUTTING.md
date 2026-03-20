# Cross-Cutting Fixes (Sloane — All 10 Specs)

These issues repeat across most or all specs. Apply as systematic passes.

---

## 1. ARIA / Focus Management / Keyboard Navigation

Every overlay, drawer, dropdown, and modal must implement the CLAUDE.md Dialog ARIA Pattern:

```
role="dialog" (or role="menu" for dropdowns)
aria-modal="true" (modals) or aria-modal="false" (non-blocking overlays)
aria-labelledby="{heading-id}"
Focus trap: on open, focus moves to first focusable element
Focus restore: on close, focus returns to trigger element
Escape key closes
```

**Affected components (by spec):**
- admin-panel: UserActions dropdown (role="menu"), CodeApplyModal (partially done)
- application-tracking: ApplicationDrawer, StatusUpdatePicker
- auth-capture: SavePrompt (has role="dialog" but missing aria-labelledby, focus trap/restore)
- canvas-engine: ChatBubble expanded panel, MobileCarousel
- job-search: SavePassActions popover
- recruiter-canvas: JDOptimizeOverlay
- resume-optimization: OptimizationOverlay
- star-stories: StoryCapturePrompt
- subscription-trial: UpgradeModal (missing focus restore), PreCommitmentPrompt, TrialNotification (needs aria-live, not dialog)

**Every interactive card** (JobMatchCard, StarStoryCard, ApplicationCard) needs:
- `tabIndex={0}`, `role="button"` or `role="article"`
- `aria-expanded` for expand/collapse
- Enter/Space toggles expansion
- Focus ring: `focus-visible:ring-2`

**Icon-only buttons** need `aria-label` (CanvasControls zoom buttons, etc.)

---

## 2. Mobile Layouts

Every spec is desktop-only. Add a "Mobile Behavior" section to each:

- Mobile (<768px): linear card stack, no canvas
- Drawers/overlays become full-screen on mobile
- Touch targets minimum 44x44px (Faye's Rule 14)
- Popovers become bottom sheets on mobile
- Filter pills become horizontally scrollable
- Test: does every interactive element work at 375px width?

**Tablet (768-1024px):** Canvas with reduced zoom (0.75), smaller card sizing. Most specs collapse tablet into desktop — add tablet treatment.

---

## 3. "Save" Language

The locked brand rule: "Save" language, never "sign up" or "create account" (Reviewer-BH1).

**Find and replace across all specs:**
- "Create free account" → "Save with email"
- "Create an account" → "Save them with just your email"
- "Lock in" → "Join as a founding member" / "I'm in"
- "Sign up" → "Save"
- Any CTA that frames capture as a commitment rather than preservation

---

## 4. Color Is Never the Only Signal

Every place color alone differentiates states needs a secondary indicator (icon, text label, pattern).

**Affected:**
- application-tracking: Status left border colors + 6px status dots → add status icons (clock, eye, message-circle, star, check, x-circle, minus-circle)
- application-tracking: `--color-interviewing: #3B82F6` (blue) → remove blue entirely, use existing palette
- job-search: Strength dots (6px colored) → add icons (filled/half/hollow circle) + text labels
- job-search: Fit score badge → add aria-label
- recruiter-canvas: RepellentCard severity bar (red/amber/gray) → add text labels or icon variation
- resume-optimization: Diff change types (green/red/purple backgrounds) → add inline prefix markers (+, ~, ↕)
- star-stories: GapCard importance dots → add text labels + distinct icons
- subscription-trial: UsageMeter progress bar → add role="progressbar" + aria attributes

---

## 5. Reattribution Line

"You described [X] skills that are in demand. You just used different words."

This line is required by the brand review (Reviewer-BH6: "worth more than any feature") but appears in ZERO specs. Decide where it lives:

- **Primary home:** vocabulary-translation-ui (after full card reveal, before capture prompt)
- **Canvas entry:** canvas-engine (AgentCommentary card on first visit after signup)
- **Job search:** job-search-ui (after results, before save/pass)

The vocabulary-translation-ui spec should own the component. Other specs reference it.

---

## 6. Performance Targets

Most specs define zero performance benchmarks. Copy from UX flow spec and add spec-specific targets:

| Target | Value | Source |
|--------|-------|--------|
| First card visible | <3s | UX flow |
| Canvas drag | 60fps / <16ms | UX flow |
| Page load LCP | <2s | UX flow |
| SSE connection | <500ms | UX flow |
| Card expand transition | <200ms | Per spec |
| Overlay open | <300ms | Per spec |
| List render (100 items) | <500ms | Per spec |
| Component mount | <16ms (one frame) | General |

Add a "Performance Targets" section to each spec with relevant subset.

---

## 7. Score Demotion

"Show the translation, not the score. Never lead with a number." — UX flow Design Principle #1

**Affected:**
- job-search: Fit score badge (44px circle, top-right) → move to secondary position, smaller, below vocab mappings
- resume-optimization: Score delta (24px bold, center header) → move to right side, 14px, secondary text
- recruiter-canvas: Alignment score → keep but ensure translations are visually dominant

---

## 8. Minimum Font Size

Brand review: minimum 14px for body on dark backgrounds. Multiple specs use 11px for labels.

**Fix:** All label text minimum 12px (ideally 13px). For muted text on dark backgrounds, use `--text-secondary` (#A1A1AA, ~6.3:1 contrast) instead of `--text-muted` (#71717A, ~3.3:1 contrast — fails WCAG AA).

---

## 9. Missing States

Every component must handle: loading, empty, error, populated, overflow.

**Common gaps across specs:**
- Loading states for drawers/lists (show skeleton rows, not flash of empty state)
- Error states for computation components (OutcomeInsight, StoryConnector reverse lookup)
- Empty states for new users (MobileCarousel, Recent Activity, JDDashboard)
