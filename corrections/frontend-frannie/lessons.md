# Lessons Learned — Soaria Frontend

Update this file after every correction. Each entry is a mistake pattern
and the rule that prevents it. Review at session start.

---

## L001: Don't index specs in SUMMARY.md until implementation is done
- **Date:** 2026-03-05
- **Spec:** n/a (process)
- **What happened:** Added Spec 17 to specs/SUMMARY.md Active table immediately after saving the spec file, before any implementation work started.
- **Root cause:** Misread the graduation flow. SUMMARY.md is updated when a spec moves to "done" (merged), not when the spec is written.
- **Rule:** Specs exist in specs/ as files. They only appear in SUMMARY.md after implementation is complete and merged. The graduation checklist in how-we-document.md defines when.

## L002: React Hook Form watch() triggers React Compiler warnings
- **Date:** 2026-03-05
- **Spec:** Spec 21 (Profile & Onboarding)
- **What happened:** Every component using `watch()` from React Hook Form produces a React Compiler "incompatible-library" warning because `watch()` returns a function that can't be safely memoized.
- **Root cause:** React Compiler can't trace reactivity through RHF's `watch()` API. This is a known RHF + React Compiler friction point.
- **Rule:** These warnings are benign — React Compiler will skip memoizing the component, which is fine for form components. Do not suppress them. Accept 0 errors + N warnings as a clean lint result for form-heavy code.

## L003: App layout needs client hooks — use a client wrapper component
- **Date:** 2026-03-05
- **Spec:** Spec 21 (Profile & Onboarding)
- **What happened:** The app layout (`(app)/layout.tsx`) needed `useProfile()` for the onboarding gate, but layout.tsx should stay a server component for optimal rendering.
- **Root cause:** Next.js layouts are server components by default. Adding `"use client"` to a layout forces the entire subtree into client rendering.
- **Rule:** When a layout needs client-side hooks (auth checks, redirects, user context), extract the logic into a client wrapper component (e.g., `AppShell`) and render it from the server-side layout. Keep layout.tsx as a thin server component.

## L004: hookform/resolvers must be explicitly installed
- **Date:** 2026-03-05
- **Spec:** Spec 21 (Profile & Onboarding)
- **What happened:** `@hookform/resolvers` was in node_modules (transitive dep) but not in package.json. Build worked locally but would fail in CI.
- **Root cause:** Assumed transitive dependency was sufficient.
- **Rule:** Always add `@hookform/resolvers` to package.json explicitly when using `zodResolver`. Run `npm install @hookform/resolvers` — don't rely on transitive deps.

## L005: Never dispatch an entire spec as a single agent task
- **Date:** 2026-03-06
- **Spec:** Spec 23a + Spec 20
- **What happened:** Dispatched two agents, each tasked with building an entire spec (15-18 files). Both hit 175k+ tokens, compacted, and produced zero usable committed output. Complete waste of time and tokens.
- **Root cause:** Ignored the 80k token cap / ~15 tool calls / ~50k token rule documented in CROSS_INSTANCE.md. A full spec requires 40-60+ tool calls (read files, write files, fix errors, run build) — far beyond what fits in a single agent context.
- **Rule:** ALWAYS break specs into milestone-sized chunks of 3-6 files / ~15 tool calls BEFORE dispatching agents. Each chunk must be small enough to complete well under 80k tokens. Plan the chunking explicitly, then dispatch. Never shortcut this step.

## L006: Content graduation must happen immediately after each spec, not batched later
- **Date:** 2026-03-06
- **Spec:** Specs 23a, 20, 21a, 19a, 22
- **What happened:** Five specs were implemented in rapid succession across multiple sessions. Graduation (AGENTS.md creation, SUMMARY.md updates, lesson graduation) was skipped each time because the sessions focused on shipping code. The gap was discovered later and required a separate catchup session.
- **Root cause:** Graduation was treated as optional post-work rather than part of the definition of done. High-velocity sessions deprioritized documentation.
- **Rule:** Graduation is the last step of every spec, not a separate task. After the final commit of a spec, immediately: (1) create/update AGENTS.md in affected directories, (2) update specs/SUMMARY.md, (3) graduate any new lessons. Do this before moving to the next spec. If time-constrained, at minimum update SUMMARY.md and note "graduation pending" in the session log.
