# Evidence Reconciliation: "Stickiness Without Resistance"

**Date:** 2026-04-01
**From:** Reid
**Status:** Complete. Ready for paper spec.

---

## Verification Summary

| Claim | Status | Gaps |
|-------|--------|------|
| 1. Transfer stickiness without human friction | VERIFIED | Forgetting curve not built; no control condition without any correction system |
| 2. Two types with different transfer properties | VERIFIED | Classification criterion is non-circular and holds for 10/11 cases |
| 3. Instruction without correction = no change (control) | PARTIALLY VERIFIED | Post-hoc discovery (not designed). "No behavioral change" weakly operationalized. |
| 4. Compilation threshold (~3-4 recurrences) | VERIFIED | Based on few examples. Code uses 3; strongest example transitioned at 4. |
| 5. Capture surface associated with correction type | VERIFIED with caveat | Strong association, not causal. Severity confound acknowledged. |
| 6. HyperAgents convergence | VERIFIED as convergence | NOT validation. Different systems, different metrics, complementary not confirming. |
| 7. Recursive meta-evidence | VERIFIED | 7/17 corrections (41%) are about the correction system itself. |
| 8. Behavioral plane is dynamic | VERIFIED qualitatively | No quantitative retention metric. No correction-rate-over-time plot. |

---

## The Numbers (verified with sources)

| Number | Value | Source | Status |
|--------|-------|--------|--------|
| Rules transferred (Bryce to Faye) | 237 | paper-evidence-reference.md line 7; 6 independent sources | VERIFIED |
| New corrections (Faye) | 44 | paper-evidence-reference.md line 193; 5 sources | VERIFIED |
| In explicitly addressed categories | 13 (30%) | section-5-findings.md line 19; 4 sources | VERIFIED |
| Total corrections (original study) | 134 | paper-evidence-reference.md line 10 | VERIFIED |
| Total corrections (Calx build, DB) | 17 | calx.db query | VERIFIED |
| Combined total | 151 | Arithmetic | VERIFIED |
| Total agents (original) | 6 | paper-draft.md Section 3.1 | VERIFIED |
| Total agents (post-paper) | 3 (Reid overlap, Thane, Mara) | Memory files | VERIFIED |
| Unique agents total | 8 | 6 original + 2 new (Reid spans both) | VERIFIED |
| Total rules | ~306 | paper-evidence-reference.md line 38 | VERIFIED |
| Correction-to-rule ratio | 2.28 (306/134) | paper-evidence-reference.md line 70 | VERIFIED |
| Total domains | 12 (8 active, 4 inactive) | paper-evidence-reference.md Section 2 | VERIFIED |
| Confusion matrix categories | 21 in paper table, 26 in evidence ref | paper-draft.md Table 1 vs evidence-reference.md line 326 | DISCREPANCY |
| Zero-overlap categories | 18 of 21 (paper) | paper-draft.md line 287 | VERIFIED |
| Batch vs per-spec review | 29 vs 72 = 2.48x | paper-evidence-reference.md line 120 | VERIFIED |
| Cache read ratio | 94.7% across 2.72B tokens | paper-evidence-reference.md line 267 | VERIFIED (minor rounding) |
| Cost (original study) | ~$250 | paper-evidence-reference.md line 279 | VERIFIED |
| Cost (full 43-day run) | $400 (2 months Max sub) | Spencer, conversation | VERIFIED verbally |
| Study date range (original) | Feb 24 - Mar 16, 2026 | paper-draft.md line 101 | VERIFIED |
| Study date range (extended) | Feb 24 - Apr 1, 2026 (43 days) | Token usage table | VERIFIED |
| HyperAgents arXiv ID | 2603.19461 | reconciliation doc | VERIFIED |
| HyperAgents imp@50 | 0.630 (math transfer) | reconciliation doc | VERIFIED |
| 8-entry mock patching chain | L017-L095, self-referencing | paper-evidence-reference.md line 169 | VERIFIED |
| Auto-promotion threshold (code) | 3 | promotion.py line 12 | VERIFIED |
| Deactivate_rule recurrence count | 4 (across Sessions 1-3) | 4 foil review files with citations | VERIFIED |
| Meta-corrections in Calx build | 7 of 17 (41%) | calx.db content analysis | VERIFIED |

---

## Discrepancies to Resolve

**1. Confusion matrix: RESOLVED.**
Evidence reference says 26 because it includes 5 backend-only categories from non-shared specs. Paper draft says 21 because Table 1 is scoped to the shared frontend spec set (both reviewer types operated on the same specs). 21 is correct and more rigorous. 18 of 21 show zero overlap. 3 boundary-adjacent detected by both. The 5 backend-only categories were correctly excluded from the comparison table.

**2. methodology.md says ratio is 1.87:1.**
The math from its own numbers (306 rules / 134 corrections) gives 2.28:1, not 1.87. The evidence reference correctly says ~2.3. methodology.md is wrong.

**3. JSONL vs DB correction count: 9 vs 17.**
The JSONL has 9 corrections (C001-C009). The DB has 17 (C001-C017). 8 corrections were logged directly to DB but not to JSONL. The two stores are not synchronized. Use DB count (17) as the authoritative number.

**4. C008 ID mismatch.**
C008 in corrections.jsonl = "Did not log corrections in real time." C008 in engineering specs = "spec-skipping, 3 recurrences." Different numbering schemes. The spec references appear to use an internal numbering separate from the JSONL. Traceability gap.

**5. $400 cost: DOCUMENTED.**
$450 = 2 months x $200/mo Anthropic Max subscription + $50 overage in the first month. Covers the full 43-day extended period (Feb 18 - Apr 1, 2026). The $250 figure for Study 1 is a subset ($200/mo + ~$50 overage). The second month is $200 flat. Total: $450 actual cost for 4.12B tokens. API-equivalent would be ~$3,185.

**6. HyperAgents timeline: DOCUMENTED.**
arXiv:2603.19461 submitted March 19, 2026. Our field study ended March 16 (3 days before). Spencer discovered HyperAgents on March 30 (11 days after submission). Independent discovery is clean and documentable.

---

## Domingos's Critical Question: Answered

**"How do you classify architectural vs process BEFORE knowing transfer outcome?"**

**Answer: VERIFIED.** The paper (Section 4.3) defines the criterion prospectively:
- **Architectural:** changes code, configuration, infrastructure, or hooks. Makes an error class structurally impossible.
- **Process:** adds a text instruction the agent must remember and execute.

The criterion is: does the correction modify the environment or add a text rule? This is determinable at capture time. Holds for 10 of 11 error classes. One borderline case (duplicate model definitions classified as architectural despite being enforced by a grep check).

The recurrence data then validates the classification without defining it. Architectural = zero recurrence. Process = recurring chains. But the classification was applied at capture time based on intervention type, not retroactively from outcomes.

---

## What's Airtight

1. The 237/44/13 transfer numbers (6+ independent sources)
2. The Mara-to-Thane non-transfer (Mara corrected Mar 22, Thane acquired independently Apr 1, exact files cited)
3. The two-type taxonomy with prospective classification criterion
4. The 8-entry mock patching chain (self-referencing confirmed)
5. The 4 inactive domains with zero corrections and authored rules
6. The deactivate_rule 4-strike chain with full provenance
7. The 2.48x reviewer specificity (29 vs 72, independently verified)
8. All 9 JSONL corrections are process-type; all foil review architectural findings are structural
9. HyperAgents timeline: our study (Feb 24 - Mar 16) predates their arXiv submission (March 2026)
10. The recursive meta-evidence: 41% of Calx-build corrections are about the correction system itself

## What Needs Work Before Paper

1. **Confusion matrix discrepancy** (21 vs 26): reconcile and state precisely
2. **"No behavioral change" in inactive domains**: define operationally. Safest: "zero corrections logged and no rule file modifications observed"
3. **Compilation threshold**: present as "approximately 3-4 recurrences" not a fixed number. Acknowledge limited data points.
4. **Capture surface claim**: soften from "determines" to "strongly associated with." Acknowledge severity confound.
5. **HyperAgents framing**: "independently converged on a similar boundary" not "validates"
6. **Correction-rate-over-time plot**: Silver asked twice, never built. Should build it from the original 134 corrections if timestamps exist in the evidence repo.
7. **$400 cost**: add to evidence reference as documented figure

## What We Honestly Don't Have

1. **A second researcher.** N=1 on the human side. Biggest limitation.
2. **A designed control condition.** The inactive domains are post-hoc, not pre-planned. Useful but not a designed experiment.
3. **Cross-model evidence.** All on Claude family models.
4. **A forgetting curve.** No retention-by-session-distance data.
5. **A formal learning curve plot.** Phase-level data exists (5.6 -> 3.0 -> 2.9) but no daily/weekly time series.
6. **Inter-rater reliability on the two-type classification.** One rater (Spencer).
