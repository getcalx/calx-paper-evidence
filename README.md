# Calx Research Program: Evidence Repository

Public evidence repository for three papers on knowledge transfer failure in human-AI collaboration.

**Researcher:** Spencer Hardwick
**Period:** February 24 through April 1, 2026 (43 days)
**Total corrections:** 151 across 8 agents, 2 products
**Total cost:** ~$450
**Repository assembled:** 2026-04-01

---

## Papers

| Paper | Title | Status | Evidence |
|---|---|---|---|
| 1 | The Behavioral Plane: Correction Dynamics in Multi-Agent Software Construction | Published (Zenodo) | Parent repo root |
| 2 | Stickiness Without Resistance: Knowledge Transfer Failure in Human-AI Collaboration Without Human Friction | Draft complete, panel-reviewed | `paper-2/` |
| 3 | The Compiler Gap (working title) | Planned | `paper-3/` (stub) |

## Structure

```
evidence/
  README.md              (this file)
  paper-2/
    README.md            (claim-to-evidence mapping, provenance)
    classification/      (codebook with worked boundary examples)
    statistical/         (Fisher's exact test, contingency table)
    ecological/          (practitioner evidence audit, 70+ sources)
    extended-dataset/    (Study 2 corrections, combined dataset)
    reconciliation/      (evidence reconciliation, verified numbers)
  paper-3/
    README.md            (stub: evidence inventory for synthesis paper)
  shared/
    README.md            (methodology, correction format, agent roster)
```

Paper 1's evidence is in the parent repo (`getcalx/calx-paper-evidence`).

## Key Artifacts

| Artifact | Location | Why it matters |
|---|---|---|
| Classification codebook | `paper-2/classification/codebook.md` | Operational decision procedure for replication |
| Fisher's exact test | `paper-2/statistical/fishers-exact-test.md` | p = 0.002 for the two-type split |
| Practitioner evidence audit | `paper-2/ecological/practitioner-evidence-audit.md` | 70+ sources across 6 platforms |
| Evidence reconciliation | `paper-2/reconciliation/evidence-reconciliation-v2.md` | Every number verified with sources |

## What's Here vs What's Not

**Here:** All evidence artifacts referenced in the papers. Correction logs, classification codebook, statistical tests, practitioner audit, panel reviews, reconciliation documents, methodology documentation.

**Not here:** Private repo access (commit hashes provided for verification), raw calx.db (exported as JSONL), thinker training files (competitive advantage), product strategy docs.

## Verification

Every empirical claim in both papers maps to one or more files in this repository. The claim-to-evidence tables in each paper's README provide the mapping. A reviewer can independently trace any finding to its source artifact.
