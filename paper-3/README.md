# Paper 3 Evidence: "The Compiler Gap" (stub)

Supporting evidence for the synthesis paper tying formal studies, HyperAgents, academic mechanism research, and practitioner evidence together.

**Status:** Stub. Paper 3 not yet written.
**Assembled:** 2026-04-01

---

## Evidence Sources (to be organized)

### Formal Field Studies
- Hardwick 2026a ("The Behavioral Plane"): Parent repo root
- Hardwick 2026b ("Stickiness Without Resistance"): `../paper-2/`

### Controlled Experimental Evidence
- HyperAgents (Meta Superintelligence Labs, arXiv:2603.19461)

### Academic Mechanism Papers
| Paper | ArXiv | Key Finding |
|---|---|---|
| Lost in the Middle (Liu et al. 2023) | 2307.03172 | U-shaped attention, 30%+ degradation |
| Attention Sorting (Peysakhovich & Lerer 2023) | 2310.01427 | Attention priors deprioritize early tokens |
| Agent Drift (2026) | 2601.04170 | 42% reduction in task success, 3.2x human intervention |
| Instruction Density (2025) | 2507.11538 | 68% accuracy at 500 concurrent instructions |
| Refusals Fail (2025) | 2512.02445 | Safety rules degrade 50%+ past 100K tokens |
| Compliance Illusion (2025) | 2506.02357 | "Illusion of compliance" named |
| Intelligence Degradation (Wang et al. 2026) | 2601.15300 | Catastrophic collapse at 40-50% capacity |
| RoPE Decay (2024) | 2406.16008 | Positional encoding as architectural culprit |
| Instruction Following at Scale (2025) | 2510.14842 | Density-compliance relationship quantified |

### Practitioner Evidence (100+ sources)
- Structured audit: `../paper-2/ecological/practitioner-evidence-audit.md`
- Extended research: `~/Library/Application Support/Claude/.../ai_rules_ignored_research.md`
- Competitive audit: `~/calx/research/competitive-audit-2026-04-01.md`

### Platforms Documented
- Cursor (20+ forum threads, controlled 0/9 test)
- Claude Code (7+ GitHub issues, 8+ blog posts, ~60% compliance measured)
- GitHub Copilot (8+ issues/discussions)
- Windsurf (2+ issues)
- Aider (1 issue)
- Devin (community reports, ~45min decay)

### Community Tools Built Around the Limitation
- fcontext (persistent context layer)
- cursor2claude (rule sync)
- CodeRabbit (automatic rule detection)
- cursor-doctor (CI linting for rules)
- COLLAPSE.md (context collapse prevention standard)
- Agent Rules (community standard spec)

---

## Thesis

Not one researcher's observation. A structural property of how language models process instructions, documented from every direction: formal field study, controlled experiment, architectural analysis, and mass practitioner experience. The solution is compilation: converting text-based rules into structural enforcement mechanisms.

---

## TODO

- [ ] Organize academic papers with full citation info
- [ ] Extract key quotes from practitioner sources
- [ ] Build cross-platform comparison table
- [ ] Document community solutions mapped to architectural/process taxonomy
- [ ] Write claim-to-evidence mapping (once paper is drafted)
