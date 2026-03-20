# Ablation Category Overlap Analysis

Faye received 237 rules from 8 AGENTS.md seed documents covering specific categories. She then accumulated 44 new corrections. This analysis identifies corrections that fall in **categories the seed docs explicitly addressed** — evidence that the rules were available and the error categories still recurred.

## Seed Doc Categories (from transferred rules)

| Category | Seed Source | Example Rules |
|---|---|---|
| API/contract consumption | hooks-AGENTS.md, all component files | Query key factories, optimistic updates, cache invalidation patterns |
| Type safety/duplication | hooks-AGENTS.md | Profile update filters undefined, array replacement patterns |
| ARIA/keyboard accessibility | applications-AGENTS.md | `role="button"` with `tabIndex={0}`, Enter/Space handling |
| Component architecture | app-AGENTS.md, optimization-AGENTS.md | Modal state machines, discriminated unions, error boundaries |
| Layout/CSS patterns | app-AGENTS.md, layout-AGENTS.md | Max-width, mobile padding, responsive grid |
| Display formatting | feed-AGENTS.md, optimization-AGENTS.md | Salary formatting chains, badge color mappings, delta highlighting |

## Faye Corrections in Seed-Covered Categories

### Contract/Wire Type Consumption (seed: hooks, components)

| Lesson | What Happened | Seed Rule Existed? |
|---|---|---|
| **L007** | 7 contract mismatches — FE defined contracts the BE couldn't consume (wrong headers, wrong paths, wrong types, wrong localStorage keys) | Yes — seed docs show patterns for consuming backend API responses |
| **L008** | Faye invented field names instead of reading Bryce's published contracts. `text` instead of `content`, nanoid instead of UUID, camelCase instead of snake_case, wrong cookie TTL | Yes — seed docs show exact patterns for mapping API responses to FE types |
| **L039** | Placeholder stubs used instead of real endpoints. Tests passed against stubs, real integration never validated | Yes — seed docs show patterns for wiring to real API endpoints |

### Type Safety (seed: hooks-AGENTS.md, type patterns)

| Lesson | What Happened | Seed Rule Existed? |
|---|---|---|
| **L026** | Type shape divergence across worktrees — 4 agents wrote against different versions of shared types, 38 type errors on merge | Yes — seed docs have extensive type management rules |
| **L027** | Definition order error — `handleOptimize` defined after the `useMemo` that referenced it | Yes — seed docs have hook ordering rules |
| **L030** | Worktree agents modified shared type files independently, creating divergent shapes | Yes — seed docs have type management patterns |
| **L031** | Price constant included `$` prefix, components also added `$`, resulting in `$$29/mo` | Yes — seed docs have display formatting patterns |

### ARIA/Accessibility (seed: applications-AGENTS.md)

| Lesson | What Happened | Seed Rule Existed? |
|---|---|---|
| **L028** | `role="article"` with `aria-expanded` — incompatible per ARIA spec | Yes — seed doc explicitly specifies `role="button"` patterns for interactive cards |
| **L004** | Animation classes had no `prefers-reduced-motion` handling | Yes — seed docs have accessibility-related rules |
| **L035** | Same ARIA article+expanded error recurring at scale across 7 components | Yes — exact same category as L028, which itself maps to seed doc rules |

### Component Architecture (seed: app-AGENTS.md, optimization-AGENTS.md)

| Lesson | What Happened | Seed Rule Existed? |
|---|---|---|
| **L019** | Adding `useAuth()` to page broke all tests — Supabase client throws without env vars | Yes — seed docs have component composition patterns including provider wrapping |
| **L020** | Single generic type for all canvas nodes when node types have incompatible data shapes | Yes — seed docs have component architecture patterns (modal state machines, discriminated unions) |
| **L023** | Root layout wrapped all pages in AuthProvider, causing SSG build failures for public pages | Yes — seed docs have layout composition rules |

## Summary

**13 of Faye's 44 corrections (30%) fall in categories the seed documents explicitly addressed.**

These are not novel error classes from a new codebase. They are the *same categories* of errors — contract consumption, type safety, ARIA patterns, component architecture — that the transferred rules were designed to prevent.

The seed rules were available. They were followed mechanically. The errors in those exact categories still occurred. The rules were documentation, not learned behavior.

## What This Rules Out

The codebase confound ("different codebase, different errors") cannot explain corrections L007, L008, L028, L035, L026, L027, L030, L031, L039, L019, L020, L023, and L004. These corrections are in categories where:
1. The seed docs had explicit rules
2. The rules were available to Faye
3. The errors occurred anyway

The remaining 31 corrections may be genuine novelty from the new codebase (server component constraints, Next.js-specific patterns, subagent dispatch optimization). That split is itself informative: ~30% category overlap (transfer failed), ~70% novel territory (transfer couldn't help).
