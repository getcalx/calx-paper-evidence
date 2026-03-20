# Frontend & Design Lead — Lessons Log

This is an append-only log of corrections and learnings from Spencer. New entries are added at the bottom.

---

## 2026-03-12 — Wire Types Must Come From Contracts, Not Memory
**Correction:** All 10 FE specs had wire types (`Wire*`, `SSE*`) written against an imagined backend. Every spec got REVISE from backend foil cross-review. Field names, response shapes, enum values, endpoint paths, pagination formats — all wrong.
**Context:** Faye filled her context window reading backend specs + her own specs simultaneously and couldn't hold the contract details. She wrote types from memory/assumption instead of verifying against `contracts/types.ts` and `api-spec.json`.
**Takeaway:** Never author wire types without the contract open. If you can't fit both the contract and the spec in context, read the contract FIRST, extract the exact types/endpoints you need into a scratch section at the top of the spec, then write the spec referencing that scratch section. If a backend spec doesn't exist yet, mark wire types as "PROPOSED" and block build chunks on contract publication. Do NOT invent wire formats and treat them as authoritative.

## 2026-03-12 — Backend Specs Are the Contract, Not the Running Codebase
**Correction:** Spencer flagged that Faye generated a massive LOCKED table claiming specs "don't exist" and endpoints "don't exist" — when the backend specs were written and even foil-reviewed. She was checking the live codebase for endpoint existence instead of reading the specs.
**Context:** Faye grepped `src/soaria/api/` to check if endpoints existed, found they weren't implemented yet, and flagged them as blocked. But the specs at `~/PycharmProjects/soaria-backend/specs/` define the full contracts. Specs 26-30 were written, reviewed by both foils, and Spec 26 was approved. The endpoints just haven't been BUILT yet — that's expected, they're in the build queue.
**Takeaway:** The backend specs directory is the API contract source. If a numbered spec defines an endpoint with request/response models, that's the contract — write the frontend spec against it. LOCKED means "no spec exists," not "endpoint doesn't return 200." Never check the codebase for contract existence — check the specs.

## 2026-03-12 — Apply Cross-Cutting Fixes as Systematic Passes
**Correction:** Same 7 issues (ARIA, mobile, color accessibility, brand language, reattribution, performance targets, score demotion) appeared in all 10 specs independently because each spec was written in isolation.
**Context:** Foil review found the same gaps 10 times. More efficient to fix systematically.
**Takeaway:** After completing a batch of specs, do systematic passes: (1) ARIA/focus/keyboard, (2) mobile layouts, (3) brand language compliance, (4) color-never-alone, (5) performance targets. Don't treat each spec as independent for these concerns — they're cross-cutting and should be applied uniformly.

## 2026-03-14 — Specs Written Against Codebase AGAIN, Not Contracts (Recurrence)
**Correction:** Marketing site specs (A-E) used the existing PasteInput redirect flow from the codebase instead of the capture funnel spec. Specs B and C both send users through the candidate paste→redirect pipeline, but the capture funnel spec defines a distinct 3-step conversational flow. Spencer: "the capture funnel spec is the authority."
**Context:** This is the THIRD time this pattern has appeared. Lesson from 2026-03-12 said "backend specs are the contract." The same principle applies to product specs: the capture funnel spec IS the flow contract. The current codebase is the old implementation. Writing specs against "what's built" instead of "what's specced" produces specs that cement the old behavior instead of delivering the designed experience.
**Takeaway:** Before writing any spec that touches a user flow, read the flow authority doc (capture funnel spec, UX flow spec) FIRST. The codebase is never the source of truth for flow design. If the current code differs from the spec, that's a build gap — not a reason to spec around it.

## 2026-03-14 — Fabricated PRD Section References
**Correction:** Specs B, D, and E all cited PRD sections 13-20. The PRD has 11 sections. These references were fabricated — the sections don't exist. Spencer: "that means the traceability chain was fabricated, not verified."
**Context:** The PRD traceability is supposed to be the verification chain — "this spec implements THIS section of the PRD." When the references are wrong, nobody can verify product alignment. Foil review caught it, but this should never have made it to review.
**Takeaway:** After writing PRD traces in the spec header, VERIFY each section number exists in the actual PRD. Open the PRD, ctrl-F the section number, confirm it exists. If no PRD section covers this spec's scope, trace to the actual authority doc (capture funnel spec, brand identity, messaging architecture). Never invent section numbers.

## 2026-03-14 — Category Language Violation in LLM-Facing Content
**Correction:** The /vocabulary-gap spec (the page designed for LLM extraction) used "career intelligence platform" in the FAQ and JSON-LD. The locked category is "career translation platform" / "agentic career fluency." Spencer locked this months ago. Memory has it in feedback-career-fluency-category.md.
**Context:** "Career intelligence" is the old framing. The category shift to "career translation" / "career fluency" was a deliberate Prism-validated decision. Using the wrong term on the page specifically designed for LLMs to cite trains every LLM on the wrong category language.
**Takeaway:** Before writing any copy in a spec — especially FAQ, JSON-LD, meta descriptions, or any LLM-extractable content — verify category language against the locked decision: "agentic career fluency" (category), "career translation platform" (product descriptor), "translate" (category verb). "Optimize" and "intelligence" are enemy words.

## 2026-03-14 — Token Names Guessed Instead of Verified Against Spec A
**Correction:** Specs C, D, and E all used wrong token names (--color-terracotta instead of --accent-terracotta, --color-verdigris instead of --state-verdigris). Spec D mapped Oxidized token names to Warm Edge hex values — every color in the mapping table was wrong.
**Context:** Downstream specs were written in parallel with Spec A instead of after Spec A was finalized. Token names were guessed from the palette concept rather than read from Spec A's actual CSS custom property definitions.
**Takeaway:** Never guess token names. Before writing any spec that depends on Spec A, read Spec A's token layer section and extract exact variable names + hex values into a reference table at the top of your spec. If Spec A isn't finalized, block on it — don't write against assumed token names. Wrong token names cause silent CSS failures (undefined custom properties resolve to `initial`).

## 2026-03-14 — Living Docs With Changelogs, Not Competing Locked Docs
**Correction:** brand-review.md (locked, amber/purple panels) and brand-identity-v2.md (locked, Graphite-on-Graphite) contradicted each other. Both said "locked." Specs mixed and matched between them. Spencer: "every doc is a living thing with a changelog. I advocate for the latter so we can keep a decision thread focused by topic in one place."
**Context:** The v2 doc said "supersedes brand-review" but brand-review.md still had "locked" status. Agents reading brand-review.md had no way to know it was superseded without also reading v2. Two competing locked docs = guaranteed drift.
**Takeaway:** When a doc is superseded, update the OLD doc's status immediately. Add a changelog entry: "Superseded by [new doc] on [date]. Decisions in this doc are no longer authoritative." Better yet: make every strategy/brand doc a living doc with a changelog section. Don't create v2 files — update the original with dated changelog entries so the decision thread stays in one place.
