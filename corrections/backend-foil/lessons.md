# Backend Foil — Lessons Log

> Append-only. Each correction or learning from spec reviews gets logged here.
> Periodically distill patterns into rules.md.

## 2026-03-12 — Single Spec Per Agent Dispatch
**Observation:** Batch review (11 specs in one agent) found 29 blocking issues. Per-spec review (one spec per agent) found 72 blocking issues on the same specs. Spec 23 flipped APPROVE → REVISE (5 blockers). Spec 26 flipped APPROVE → REVISE (3 blockers).
**Context:** Batch agents compact earlier spec context when reviewing later specs. By spec 8-11, the agent is working from compressed memory of specs 1-4. Review quality degrades proportionally to context consumed.
**Takeaway:** Always dispatch one spec per agent. Run all specs in parallel since they're independent within the same foil's pass. The token cost is negligible compared to the cost of missed blocking issues entering build.

## 2026-03-12 — Cross-Reviewing FE Specs Catches Boundary Failures
**Observation:** Ran Nolan on all 10 frontend specs (not just backend specs). Found 60+ blocking issues — nearly all wire format mismatches (wrong field names, wrong endpoint paths, wrong response shapes, wrong enum values, invented SSE contracts). Sloane's same-domain review found zero of these because she doesn't read backend contracts.
**Context:** Every single FE spec invented `Wire*` types against an imagined backend instead of the published contracts. Root cause: Faye filled her context window reading backend specs + her own specs and couldn't hold the contract details, so she wrote types from memory.
**Takeaway:** Cross-review is mandatory. Nolan must review FE specs for integration boundaries. Sloane must review BE specs for frontend consumption contracts. Same-domain foils catch design issues; cross-domain foils catch boundary failures. Run both in parallel.

## 2026-03-12 — Correction Files Beat Re-Reading Contracts
**Observation:** After identifying that Faye can't hold backend contracts + her own specs in context simultaneously, we compiled per-spec correction files (extracted from Nolan's reviews) that list exact field renames, type shape fixes, endpoint corrections, and blocked dependencies. Faye applies corrections mechanically without needing to re-read contracts.
**Context:** Spencer identified the root cause: "the problem I ran into was that Faye filled an entire context window just reading those backend specs and her front end specs." The correction file approach decouples contract verification (Nolan's job) from spec revision (Faye's job).
**Takeaway:** When an agent can't fit all necessary context, split the work: one agent extracts/verifies (produces corrections), another agent applies (reads corrections + own artifact). The extract step runs with full contract context; the apply step runs with only the corrections + the target file. This is a general pattern for context-limited agents working against large reference documents.

## 2026-03-12 — FE Specs Built Against Unwritten Backend Specs
**Observation:** Recruiter Canvas spec builds against 8 endpoints from Spec 28 (not written). Canvas Engine builds against GET/PUT /v1/canvas from Spec 26 (not written). Star Stories builds against SSE types from Spec 24 addendum (not written). Resume Optimization assumes SSE streaming when the backend is synchronous JSON.
**Context:** These specs were written before the backend specs they depend on. The wire types are frontend-authored, inverting authority (Bryce owns API contracts).
**Takeaway:** Flag any FE spec that references a backend spec number that doesn't exist yet. Wire types for unwritten backend specs must be marked "PROPOSED — awaiting backend contract" and build chunks that depend on them must be explicitly blocked. The build queue should not include chunks blocked on unwritten backend specs.
