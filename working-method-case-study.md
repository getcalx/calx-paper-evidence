# ACE Working Method — Case Study
**Date:** 2026-03-17
**Scope:** Methodology documentation for the Lattice/Parallax paper

## System Scale
- 13 distinct agent roles defined
- ~125+ lessons across all domains
- ~200+ rules across 8 domain files
- 35 thinker + ~25 investor voices in Prism
- 8 Prism clusters, routed dispatch
- 43 engineering specs, 28 DB migrations
- 7 FIX specs, 10 RFCs
- ~1.9B tokens of work across sessions
- 28 deep-dive agents in single audit session (this dispatch)

## Error Elimination Compounds
- Column drift: 3 errors/spec → 0 (checklist fix)
- Atomicity violations: 1/spec → 0 (rule fix)
- FE contract sync misses: dropped to 0 after mandatory spec section added
- Error categories tracked in ERROR_TRACKING.md — convergence is measurable

## Per-Spec Dispatch vs Batch
- Batch review (6 specs, 1 agent): 29 blocking issues found
- Per-spec dispatch (6 specs, 6 agents): 72 blocking issues found
- **2.5x improvement** — same specs, same review criteria, different dispatch pattern
- Lesson: agent context saturation is real. One spec per agent for deep review.

## Cross-Domain Foil Pattern
- Nolan (BE foil) reviewing FE specs: found 60 boundary issues
- Sloane (FE foil) reviewing same specs: found 0 of those boundary issues
- The foil pattern catches exactly what same-domain reviewers miss
- Pattern now codified: every spec gets both domain foil + cross-domain foil

## Context Window Split Pattern
- Problem: agent can't fit reference doc + output in single context
- Solution: Extract agent (full context, reads everything, produces structured summary) → Apply agent (narrow context, reads summary, writes output)
- Decouples verification from revision
- Used for: spec reviews, PRD reconciliation, coordination updates

## Correction Propagation
- lessons.md: append-only log (timestamped entries)
- rules.md: distilled operating rules (derived from lessons, with IDs)
- Pattern: mistake → lesson entry → rule extraction → template/checklist update → error eliminated
- Rules reference lessons for traceability
- Health tooling verifies rule coverage

## Key Methodology Insight
The system that Spencer built to develop Versed IS the same vocabulary translation pattern that Versed sells. Corrections compound. Context evolves. The method IS the thesis. This recursive application — using the product's own principle to build the product — is the strongest signal that the founder deeply understands the problem space.
