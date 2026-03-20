# AGENTS.md — src/components/

Rules for agents working on React components in this directory.

## Structure

- `cards/` — Card components (JobCard, ResumeDiff, MatchExplanation, etc.)
- Top-level — Shared UI components (buttons, inputs, layout primitives)

## Rules

### comp-R001 — Cards are React components, not rendered markdown
Cards receive structured data as props and render structured UI.
Never use `dangerouslySetInnerHTML` or render raw markdown in cards.
- **Source:** CLAUDE.md critical rule #6
- **Added:** 2026-03-10
- **Status:** active

### comp-R002 — Vocabulary mappings are primary content
Every card that shows job/resume data must prominently display vocabulary mappings
("They say X → You have Y"). Never bury mappings below fold or behind a click.
- **Source:** CLAUDE.md critical rule #8
- **Added:** 2026-03-10
- **Status:** active

### comp-R003 — Use Meridian design tokens, never hardcode colors
All components use Meridian tokens from globals.css. The palette is cool navy backgrounds
with amber-gold (#E8A84C) as the sole warm accent. Use CSS custom properties
(`var(--bg-deep-ink)`, `var(--accent-signal)`, etc.) for all colors.
Never hardcode hex values or rgba in component styles — if a token doesn't exist, add it
to globals.css. The only exception is the OG image route which requires inline hex for
@vercel/og rendering.
- **Source:** CLAUDE.md design system, Meridian migration (2026-03-15)
- **Added:** 2026-03-10, updated 2026-03-15
- **Status:** active

### comp-R004 — Compact and expanded states
All cards must implement both compact (default) and expanded (on click) states.
Compact: company, title, score, 2-3 vocab mappings, CTA.
Expanded: full detail.
- **Source:** CLAUDE.md card component library
- **Added:** 2026-03-10
- **Status:** active

### comp-R005 — No `any` types
All component props must be fully typed. Use TypeScript interfaces in `@/types/`.
Never use `any`, `unknown` as a shortcut, or untyped event handlers.
- **Source:** CLAUDE.md critical rule #5
- **Added:** 2026-03-10
- **Status:** active

### comp-R006 — Every animation needs a reduced-motion counterpart
Every CSS animation or transition must include its `@media (prefers-reduced-motion: reduce)` counterpart in the same block. Don't add it as a follow-up — it ships with the animation or it doesn't ship.
- **Source:** L004
- **Added:** 2026-03-14
- **Status:** active

### comp-R007 — Server components: no event handlers, no hooks, CSS-only interactivity
Server components (no `"use client"`) cannot use `onX` event handler props, React hooks, or browser APIs. Hover/focus effects must be CSS-only (Tailwind `hover:`, `group-hover:`). Image fallbacks must use CSS `object-fit` + background color, not `onError`. When dispatching subagents for server components, include these constraints in the prompt.
- **Source:** L022, L025
- **Added:** 2026-03-14
- **Status:** active

### comp-R008 — Internal links use next/link
For any `href` starting with `/`, use `<Link>` from `next/link`, not `<a>`. HTML anchors are only for external URLs, `mailto:`, and `tel:` links. This is enforced by `@next/next/no-html-link-for-pages`.
- **Source:** L024
- **Added:** 2026-03-14
- **Status:** active

### comp-R009 — aria-expanded requires a role that supports it
`role="article"` does not support `aria-expanded` per ARIA spec. When a card needs expand/collapse, use `role="region"` (which supports `aria-expanded`) or place `aria-expanded` on a child `<button>`. Check role-supports-aria-props before combining any role with aria-state attributes.
- **Source:** L028, L034
- **Added:** 2026-03-14
- **Status:** active

### comp-R010 — CSS Grid column count must match visible children per breakpoint
When a grid has show/hide children across breakpoints (`hidden`, `md:block`), count the VISIBLE children at each breakpoint and set `grid-cols-N` accordingly. If a separator appears between two panels, use `grid-cols-[1fr_1px_1fr]` (explicit separator sizing) rather than `grid-cols-2` with a third element.
- **Source:** L040
- **Added:** 2026-03-14
- **Status:** active

### comp-R011 — Every named section needs a component entry
If a page layout references a named section, that section must appear in the component file structure — even if it's simple enough to inline. Consistency with the file list pattern matters more than saving a file.
- **Source:** L006
- **Added:** 2026-03-14
- **Status:** active

### comp-R012 — Locked design tokens are immutable
Before changing any token value, check the locked brand review. If the value is locked, use it exactly — no "warm shift" approximations, no 1-digit hex deviations. Deviations require explicit Spencer approval documented in the spec.
- **Source:** L003
- **Added:** 2026-03-14
- **Status:** active

### comp-R013 — Route groups separate auth from SSG
Auth providers go in `(app)/layout.tsx` with `export const dynamic = 'force-dynamic'`. Marketing layout has Nav + Footer only. Root layout stays minimal (fonts, globals, metadata). Never wrap SSG pages in providers that trigger client-side initialization.
- **Source:** L023
- **Added:** 2026-03-14
- **Status:** active

### comp-R014 — New integrations require immediate test updates
When adding a new import (hook, provider, module with side effects) to an existing component, immediately update that component's test file in the same step: add mocks for every new module in the import chain that has side effects or requires providers. Don't wait for test failures to discover cascade breakage.
- **Source:** L018
- **Added:** 2026-03-14
- **Status:** active

### comp-R015 — Use sed for bulk mechanical find-replace across many files
When doing the same string replacement across 10+ files (palette migrations, rename refactors), use `sed -i` via Bash instead of the Edit tool. Edit requires each file to be read first in the main context, which wastes tokens on mechanical work. Reserve Edit for targeted changes where understanding surrounding code matters.
- **Source:** L043
- **Added:** 2026-03-15
- **Status:** active

### comp-R016 — Palette migration: exhaustive negative grep after bulk replace
After bulk-replacing color values, run a comprehensive negative grep for ALL old RGB triplets — both spaced (`rgba(196, 101, 74`) and unspaced (`rgba(196,101,74`) variants. Include Tailwind class patterns. Iterate until zero results from source files (exclude specs/). Don't trust the migration plan's file list as exhaustive — grep finds what plans miss.
- **Source:** L044
- **Added:** 2026-03-15
- **Status:** active
