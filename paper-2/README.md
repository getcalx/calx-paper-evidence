# Paper 2 Evidence: "Stickiness Without Resistance"

Supporting evidence for "Stickiness Without Resistance: Knowledge Transfer Failure in Human-AI Collaboration Without Human Friction" (Hardwick 2026b).

**Assembled:** 2026-04-01
**Paper status:** Draft complete, panel-reviewed (2 rounds, 14 thinkers), revisions applied
**Relationship to Paper 1:** This paper extends Paper 1 ("The Behavioral Plane") with a second study, theoretical engagement, and ecological prevalence evidence. Paper 1's evidence is in the parent repo root.

---

## Claim-to-Evidence Map

### Primary Contribution 1: Stickiness persists without human friction

| Claim | Evidence | Files |
|---|---|---|
| 237 rules transferred, 44 new corrections, 13 in addressed categories (Study 1) | Correction logs, seed docs, rule files | Parent repo: `ablation/`, `corrections/frontend-faye/` |
| Mara corrected Mar 22 on "spec before building" | Mara's correction store | `extended-dataset/mara-corrections.jsonl` |
| Thane acquired same correction independently, 3 recurrences, Apr 1 | Thane's correction store, C007 | `extended-dataset/thane-corrections.jsonl` |
| Written rule available in shared memory when Thane started | Reid's memory files | `extended-dataset/shared-memory-snapshot.md` |
| Codification compression loss (not Szulanski's original causal ambiguity) | Analysis in paper Section 2.1 | `reconciliation/evidence-reconciliation-v2.md` |

### Primary Contribution 2: Two types with statistically significant separation

| Claim | Evidence | Files |
|---|---|---|
| 6/6 architectural error classes: zero recurrence | Error class inventory with recurrence data | `statistical/error-class-inventory.md` |
| 5/5 process error classes: recurrence (chains of 3-8) | Same inventory, chain lengths documented | `statistical/error-class-inventory.md` |
| Fisher's exact test p = 0.002 | Computation with worked example | `statistical/fishers-exact-test.md` |
| Classification criterion is prospective (not post-hoc) | Codebook with decision procedure | `classification/codebook.md` |
| 1 borderline case (duplicate model definitions) | Boundary analysis | `classification/codebook.md` Section 3 |
| 8-entry mock patching chain (L017-L095) | Backend code-level lessons | Parent repo: `corrections/backend/code-lessons.md` |

### Primary Contribution 3: HyperAgents convergence

| Claim | Evidence | Files |
|---|---|---|
| Study 1 ended Mar 16, HyperAgents submitted Mar 19 | ArXiv submission date | `reconciliation/hyperagents-timeline.md` |
| Prompt-level vs meta-level maps to process vs architectural | Analysis | Paper Section 5.5 |
| imp@50 = 0.630 in math transfer | HyperAgents paper, arXiv:2603.19461 | External |

### Primary Contribution 4: Recursive meta-evidence

| Claim | Evidence | Files |
|---|---|---|
| 17 corrections captured during Study 2 | Correction database | `extended-dataset/calx-corrections.jsonl` |
| 7/17 (41%) about the correction system itself | Classification of each | `extended-dataset/meta-correction-classification.md` |

### Preliminary Observation: Inactive domains

| Claim | Evidence | Files |
|---|---|---|
| 4 domains with ~57 authored rules, zero corrections | Rule files from inactive domains | Parent repo: domain directories |
| 8 domains with active work and corrections | Same | Parent repo: domain directories |

### Preliminary Observation: Compilation pattern

| Claim | Evidence | Files |
|---|---|---|
| deactivate_rule: 4 recurrences across Sessions 1-3 | Foil review files with citations | `extended-dataset/compilation-chains.md` |
| C006 (annotations): 3 recurrences | Same | Same |
| Clean-exit marker: 3 recurrences | Same | Same |
| System code implements threshold = 3 | Source code | `extended-dataset/promotion-threshold-source.py` |
| Circularity risk acknowledged | Paper Section 5.4 | N/A |

### Ecological Prevalence (Section 6.8)

| Claim | Evidence | Files |
|---|---|---|
| 70+ distinct community reports across 6 tools | Structured audit | `ecological/practitioner-evidence-audit.md` |
| 0/9 compliance in controlled .cursorrules test | nedcodes DEV.to article | `ecological/source-index.md` |
| ~60% CLAUDE.md compliance | Montes Medium article | `ecological/source-index.md` |
| Community independently named the distinction | "CLAUDE.md is a suggestion, hooks make it law" | `ecological/source-index.md` |
| Academic mechanism papers | Liu et al. 2023, Wang et al. 2026, etc. | `ecological/academic-mechanisms.md` |

### Extended Dataset

| Claim | Evidence | Files |
|---|---|---|
| 151 total corrections (134 + 17) | Combined dataset | `extended-dataset/combined-corrections.md` |
| 43 days (Feb 24 - Apr 1, 2026) | Timestamps on all entries | Same |
| 8 unique agents (6 + 2, Reid spans both) | Agent roster | `extended-dataset/agent-roster.md` |
| $450 total cost ($200/mo x 2 + $50 overage) | Subscription records | `extended-dataset/cost-breakdown.md` |
| 4.12B tokens across both studies | Usage data | `extended-dataset/token-usage.md` |

---

## Provenance

### Study 1 (Feb 24 - Mar 16, 2026)
All provenance documented in parent repo README. Commit hashes from soaria-backend, versed_fe, soaria-frontend.

### Study 2 (Mar 27 - Apr 1, 2026)

| Evidence | Source | Timestamp |
|---|---|---|
| Mara correction (spec before building) | oss/corrections.jsonl | 2026-03-22 |
| Thane C007 (same correction, independent) | calx/corrections.jsonl | 2026-04-01 |
| 17 Study 2 corrections | calx.db | 2026-03-27 to 2026-04-01 |
| Foil review files (deactivate_rule chain) | calx/evaluations/ | Sessions 1-3 |
| getcalx v0.6.0 release | PyPI | 2026-04-01 |

### Ecological Evidence (Jan 2025 - Apr 2026)
All URLs verified 2026-04-01. Source index includes platform, URL, date, engagement metrics.

---

## What's Not Here

- **Private repo access** -- soaria-backend, versed_fe, soaria-frontend are private. Commit hashes provided for verification if access is granted.
- **Raw calx.db** -- Contains the 17 Study 2 corrections. Exported as JSONL in this repo. DB available on request.
- **Full panel prompts** -- The thinker training files used to dispatch panel reviewers are competitive advantage. The reviews themselves (the output) are included in full.
- **Product strategy docs** -- Calx company strategy, fundraising materials, and competitive intelligence are not evidence and are excluded.
