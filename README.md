# Calx Paper — Supporting Evidence

Public evidence repository for "The Behavioral Plane: Why Learned Corrections Don't Transfer Between Agents."

This repo contains the raw artifacts from a 20-day solo build of an 82K-line production system (42K Python, 40K TypeScript) across three codebases using Claude Code with a structured correction methodology. Every empirical claim in the paper maps to one or more files below.

## Claim-to-Evidence Map

### Core Finding: Corrections compound but don't transfer

| Claim | Evidence | Files |
|---|---|---|
| 134 corrections across 8 active domains | Correction logs with dates and traceability | `corrections/*/lessons.md` |
| ~306 rules derived from corrections | Rule files with lesson references | `corrections/*/rules.md`, `rules/backend-agents-md/` |
| Per-spec error rate drops ~50% (5.6 → 2.9/spec) | Phase breakdown in evidence reference | `paper-evidence-reference.md` §6 |
| Architectural fixes eliminate error classes permanently | Error recurrence data with named classes | `paper-evidence-reference.md` §6 |
| Process rules fail under cognitive load (8-entry test mock chain) | Backend code-level lessons L017→L095, each referencing prior | `corrections/backend/code-lessons.md` |
| Per-spec error rate decline over 3 phases | Timeline with dates per lesson | `timeline.md` |

### Natural Ablation: Rules don't transfer between agents

| Claim | Evidence | Files |
|---|---|---|
| Bryce generated seed docs for frontend agent | Spec 17 frontend migration spec | `ablation/SPEC_17_FRONTEND_MIGRATION.md` |
| Frannie built with seed docs: 6 lessons in 30 commits | Frannie's correction log | `corrections/frontend-frannie/lessons.md` |
| Faye built with same seed docs: 44 lessons despite transfer | Faye's correction log (281 lines) | `corrections/frontend-faye/code-lessons.md` |
| Seed docs are the rules Faye received | 8 AGENTS.md files (237 rules) from soaria-frontend | `ablation/seed-docs/` |
| Faye developed her own rules despite seed | 5 AGENTS.md files (103 rules), distinct from seed | `rules/faye-agents-md-*` |
| This was not a designed experiment — Spencer was genuinely trying to accelerate onboarding | Spec 17 written as a real engineering artifact, not a test | `ablation/SPEC_17_FRONTEND_MIGRATION.md` |

### Cross-Domain Review: Different expertise catches different failures

| Claim | Evidence | Files |
|---|---|---|
| Backend foil found 60+ boundary issues on FE specs | Contemporaneous lesson entry dated 2026-03-12 | `corrections/backend-foil/lessons.md` §2 |
| Frontend foil found zero boundary issues on same specs | Same lesson entry + frontend-foil lessons | `corrections/frontend-foil/lessons.md` |
| 17 per-spec correction files with exact field-level mismatches | Nolan's cross-domain review outputs | `review-evidence/per-spec-corrections/` |
| Sloane found 7 systematic patterns (ARIA, mobile, brand) across all 10 FE specs | Cross-cutting review output | `review-evidence/per-spec-corrections/CROSS_CUTTING.md` |
| Zero overlap between same-domain and cross-domain findings | 24-category confusion matrix (21 on shared FE spec set, 3 overlap) | `review-evidence/cross-domain-matrix.md` |

### Per-Spec vs Batch Review: Context compaction degrades quality

| Claim | Evidence | Files |
|---|---|---|
| Batch review (11 specs): 29 blocking issues | Contemporaneous lesson entry | `corrections/backend-foil/lessons.md` §1 |
| Per-spec review (same 11 specs): 72 blocking issues (2.48x) | Same lesson entry + per-spec review outputs | `corrections/backend-foil/lessons.md` §1 |
| Spec 23 flipped APPROVE → REVISE with 5 blockers | Lesson entry + FOIL_REVIEW_FIXES showing Issue 1-4 are Spec 23 | `review-evidence/foil-review-rounds/` |
| 4 rounds of foil review iteration (v1-v4) | Complete review round files | `review-evidence/foil-review-rounds/FOIL_REVIEW_FIXES_v*.md` |
| Per-spec reviews for Specs 32-34 with individual verdicts | Dated review files with blocking issues | `review-evidence/per-spec-reviews/` |

### Build Scale

| Claim | Evidence | Files |
|---|---|---|
| 40+ numbered specs | Spec summary index | `specs/SUMMARY.md` |
| Spec template showing methodology | Standard spec structure | `specs/SPEC_TEMPLATE.md` |
| 202 fixes across 10 specs in single FE revision | Git commit `5a404f0` in versed_fe repo (2026-03-12) | Cross-reference: `git log` |
| Backend AGENTS.md: 8 subdirectories of graduated rules | Complete rule files | `rules/backend-agents-md/` |

### Cost

| Claim | Evidence | Files |
|---|---|---|
| $250 actual cost ($200/mo Max + ~$50 overage) | Daily token breakdown table | `paper-evidence-reference.md` §8 |
| 2.72B tokens, $2,116 API equivalent | Same section with per-day granularity | `paper-evidence-reference.md` §8 |
| 94.7% cache read ratio | Token breakdown | `paper-evidence-reference.md` §8 |

### Methodology Case Study

| Claim | Evidence | Files |
|---|---|---|
| Working method description from build period | Case study written during Mar 17 audit | `working-method-case-study.md` |
| Lessons and patterns from the build | Contemporaneous reflection | `lessons-from-build.md` |
| Compiled evidence reference with verified numbers | Point-in-time audit (2026-03-17) | `paper-evidence-reference.md` |

## Provenance

All correction logs, rules files, and review outputs were generated during the build period (Feb 24 - Mar 19, 2026). The primary evidence chain is:

1. **Private repos** (`soaria-backend`, `versed_fe`, `soaria-frontend`) contain the full git history with commit timestamps showing append-over-time accumulation
2. **Brain vault** (`~/brain/docs/agents/`) contains strategic-level corrections across 12 domains
3. **This repo** extracts the evidence artifacts without modification

Git history in the source repos is the strongest authenticity signal. Commit hashes and timestamps for key evidence points:

| Evidence | Repo | Commit | Date |
|---|---|---|---|
| Foil review files (specs 23-30) | soaria-backend | `1821dae` | 2026-03-12 |
| Foil reviews archived | soaria-backend | `aaf2666` | 2026-03-14 |
| Per-spec reviews (specs 32-34) | soaria-backend | `359e391` | 2026-03-15 |
| FE spec revision (202 fixes) | versed_fe | `5a404f0` | 2026-03-12 |
| Faye lessons append history | versed_fe | 17 commits | 2026-03-10 to 2026-03-15 |
| Lessons L072-L076 | soaria-backend | `9e9f0ef` | 2026-03-13 |
| Rule graduation (27 lessons) | soaria-backend | `1c176fe` | 2026-03-14 |

## What's Not Here

- **Original batch review session output** — The 29-issue batch review was a conversation session, not a saved file. The number is documented contemporaneously in `corrections/backend-foil/lessons.md`.
- **Identity-based persona profiles** — Thinker dispatch mechanism is competitive advantage (paper two).
- **Commercial strategy** — Calx ecosystem thesis, fundraising docs, investor framing.
- **Product business logic** — Spec implementation details ("what" sections). Review artifacts show "how it was reviewed," not what was built.
- **API keys, secrets, credentials** — Scanned and verified clean before publishing.

## Repository Structure

```
calx-paper-evidence/
├── README.md                           # This file — claim-to-evidence map
├── paper-evidence-reference.md         # Compiled evidence audit (2026-03-17)
├── working-method-case-study.md        # Methodology description
├── lessons-from-build.md               # Build reflections
├── timeline.md                        # Correction timeline with dates and phases
├── corrections/                        # Per-domain correction logs
│   ├── backend/                        # Bryce: strategic lessons + code-lessons.md (L001-L095)
│   ├── backend-foil/                   # Nolan: cross-domain + batch findings
│   ├── frontend-faye/                  # Faye: strategic + 44 code-level lessons
│   ├── frontend-frannie/               # Frannie: 6 lessons (ablation control)
│   ├── frontend-foil/                  # Sloane: systematic UX patterns
│   ├── brand/, design/, marketing/,    # Inactive domains (top-down rules, zero corrections)
│   │   systems-architect/              #   Included for domain scope, not correction activity
│   ├── content/, fundraising/,         # Active domains with corrections
│   │   lens-rotator/                   #
│   └── root/                           # Reid (orchestrator): 91 lines
├── rules/
│   ├── backend-agents-md/              # 8 AGENTS.md files from soaria-backend
│   └── faye-agents-md-*               # 5 AGENTS.md files Faye developed
├── ablation/
│   ├── SPEC_17_FRONTEND_MIGRATION.md   # Origin spec (Bryce → frontend seed)
│   └── seed-docs/                      # 8 AGENTS.md files given to Faye (237 rules)
├── review-evidence/
│   ├── cross-domain-matrix.md          # 26-category detection matrix (zero overlap)
│   ├── foil-review-rounds/             # FOIL_REVIEW_FIXES v1-v4
│   ├── per-spec-corrections/           # 17 correction files + CROSS_CUTTING.md
│   └── per-spec-reviews/               # Individual review files (Specs 32-34, 39, 43)
├── specs/
│   ├── SUMMARY.md                      # Spec index (40+ specs)
│   ├── SPEC_TEMPLATE.md                # Standard spec structure
│   └── SPEC_17_FRONTEND_MIGRATION.md   # Ablation origin spec
└── cost/                               # (Token data in paper-evidence-reference.md §8)
```
