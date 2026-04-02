# Re-Review: "Stickiness Without Resistance"
## Reviewer: Chip Huyen (R2)
## Date: 2026-04-01

---

## 1. Overall Assessment

The revision is substantially stronger. The author addressed the most critical structural concern from my first review (the missing competing explanations) and added two sections that materially change the paper's defensibility: the ecological prevalence evidence and the interaction orientation scope condition. The paper now reads as a careful practitioner-researcher making bounded claims with appropriate caveats, rather than a system builder reaching for theoretical weight beyond what the data supports.

That said, three of my original six concerns are partially addressed rather than fully resolved, and the revision introduces one new issue worth flagging.

---

## 2. Issue-by-Issue Assessment

### 2.1 System Description Reproducibility: PARTIALLY ADDRESSED

My original concern: the system is not described precisely enough for another researcher to reconstruct it.

The revision adds detail about shell hooks, the correction pipeline, and the distillation process. Section 3.1 now specifies the orientation hook, pre-edit hook, and coordination gate by function. But the specification is still at the level of "what the hook does" rather than "what the hook checks." I said last time that a system specification appendix (exact hook scripts, rule injection format, correction log schema, pinned commit hash) would suffice. The paper still does not include one.

This matters less now than it did in the first draft, because the autoethnographic framing (Section 4.1) repositions the paper from "here is a system producing generalizable findings" to "here is a practitioner documenting an emergent phenomenon." Autoethnographic work has different reproducibility standards. The phenomenon needs to be characterizable by others, not necessarily reconstructable in exact form. The publicly available artifacts (correction logs, rule files, commit history) provide that.

**Verdict:** The concern is partially resolved by the methodological reframing. A technical appendix would still strengthen the paper, but it is no longer a blocking issue.

### 2.2 Correction Capture Operationalization: PARTIALLY ADDRESSED

My original concern: the definition of "correction" is never operationalized in a way another researcher could apply independently.

Section 4.1 now states: "An intervention qualified as a correction when the developer changed an agent's behavior: redirecting an implementation approach, identifying a missed constraint, correcting an architectural assumption, or flagging a process violation." This is better than the first draft but still defines a correction by its intent (to change behavior) rather than by observable criteria. Every prompt changes an agent's behavior. The question remains: where is the boundary between a correction and a normal instruction?

The autoethnographic framing helps here too, because it makes the researcher's judgment an explicit part of the methodology rather than a hidden confound. Section 7.1 and 7.6 both acknowledge single-rater classification as limitations. But I still think the paper would benefit from the specific decision rule I suggested in R1: an intervention was logged as a correction when it (a) referenced a specific failure in the agent's prior output, (b) prescribed a behavioral change for future sessions, and (c) was not a first-time instruction for a new task. Something at that level of specificity would give the "invite others to replicate" framing teeth. Right now, replicators would need to re-derive the classification standard.

**Verdict:** Improved by the autoethnographic framing and the expanded limitations, but the trigger criterion remains underspecified for replication purposes.

### 2.3 Competing Explanations: ADDRESSED

This was my most pointed concern. The first draft did not engage the simpler explanation that prompt-based instructions are just brittle.

Section 6.4 ("What Transfers and What Does Not") now confronts three competing explanations directly: prompt brittleness, context-specific encoding, and attention recency. Each is stated plainly, evaluated against the data, and shown to be partially explanatory but insufficient. The key argument is sound: none of the three alternatives accounts for the architectural/process asymmetry. If the problem were just prompt brittleness or attention recency, architectural knowledge should degrade too. It does not.

The Mara-to-Thane argument is deployed effectively: "Prompt brittleness predicts gradual degradation, not complete re-derivation." This is the right counterargument, stated precisely.

**Verdict:** Fully addressed. The paper now earns the behavioral plane concept by demonstrating that simpler explanations leave a residual the concept explains.

### 2.4 Szulanski Mapping: ADDRESSED

My original concern: causal ambiguity and absorptive capacity were being applied rather than extended, and the mapping was not clean.

Section 2.1 now explicitly states: "We extend Cohen and Levinthal's (1990) original formulation here" and explains that the "prior related knowledge" that matters is not domain expertise but experiential correction history. The paper no longer pretends this is a direct application. It says it is extending the concept, explains what the extension adds, and identifies where the original definition diverges from the observed mechanism.

This is how to use established theory: anchor your work in it, then say clearly where you depart.

**Verdict:** Fully addressed.

### 2.5 Agent Memory Architecture Critique: IMPROVED BUT STILL THIN

My original concern: the claim that "an agent that retrieves a correction artifact is not an agent that has been corrected" challenges an entire product category (Mem0, Zep, Letta) on two data points.

Section 2.5 remains brief. The Mara-to-Thane case in Study 2 is now positioned more carefully as a within-researcher replication, which helps. But the critique still rests on two transfer events (Bryce-to-Faye, Mara-to-Thane). The paper has not added thought experiments about what Mem0-style shared memory would and would not capture, and has not narrowed the claim to what the data supports.

The ecological prevalence section (6.7) indirectly strengthens this by showing that rule adherence failure is widespread across tools, which implies that retrieval-based memory would face the same problem at scale. But the connection is not made explicitly.

**Verdict:** Improved by the ecological evidence and the Study 2 replication, but the Section 2.5 claim still outruns the supporting data. One sentence scoping the claim ("Our evidence suggests this for correction-derived behavioral knowledge specifically; whether the pattern extends to other forms of experiential memory requires further study") would fix it.

### 2.6 Cold-Start / Data Flywheel Problem: NOT ADDRESSED

My "one thing the author probably has not considered" from R1: the system optimizes itself toward fewer corrections, which degrades the signal quality available for future compilation threshold estimation. The compilation threshold is calibrated on an immature system generating abundant corrections, and will become noisier as the system matures.

This does not appear in the revised Discussion or Future Work. Section 5.6 discusses the error rate floor but does not connect it to the signal-sparsity problem. The 2.9 corrections/specification plateau is discussed as a possible frontier effect (the system is handling harder territory at the same rate), which is a reasonable interpretation. But the data flywheel concern is complementary, not competing: even if the frontier interpretation is correct, the compilation threshold estimate becomes less reliable as the correction rate drops.

**Verdict:** Not addressed. Worth a paragraph in Future Work at minimum.

---

## 3. New Additions: Assessment

### 3.1 Autoethnographic Methodology (Section 4.1): STRONG

The Ellis and Bochner (2000) and Adams et al. (2015) citations anchor the work in an established qualitative tradition. The framing is honest: "The approach trades external validity for ecological validity." This is the right move. The first draft was caught between wanting to be an empirical systems paper and being a practitioner case study. Committing to the autoethnographic frame resolves that tension.

### 3.2 Ecological Prevalence (Section 6.7): STRONG WITH A CAVEAT

The 70+ practitioner reports across six tools is exactly the kind of convergent evidence the paper needed. It transforms the N=1 limitation from a fatal weakness to a tradeoff: one deeply documented case of a widely experienced phenomenon.

The academic mechanism citations (Liu et al. "Lost in the Middle," Wang et al. threshold collapse) give the process correction failure a neuroanatomical explanation at the transformer level, which complements the organizational psychology framing. The practitioner quote ("Your CLAUDE.md is a suggestion. Hooks make it law") is a near-perfect restatement of the architectural/process distinction in practitioner language.

**Caveat:** The section promises a forthcoming synthesis paper [Hardwick2026c]. This is fine for a working paper but creates a dependency for submission. At submission time, either the ecological evidence needs to be self-contained in this paper or the synthesis needs to be available. Do not submit with a "forthcoming" citation as the primary support for a major section.

### 3.3 Interaction Orientation (Section 6.6): VALUABLE

This is the most important new addition for the paper's intellectual honesty. Acknowledging that the phenomenon only manifests when the human treats the agent as a persistent collaborator (rather than a stateless tool) is a genuine scope condition that most authors would hide. Stating it openly, and noting that the relevant interaction regime is becoming more common, is exactly the right framing: "This is a scope condition, not a weakness."

### 3.4 Section 6.3 (Context-Variance in a Kind Domain): CLEAN

The Epstein framing is well deployed. Observing transfer failure in a domain with fast, accurate feedback isolates the structural mechanism from feedback-quality confounds. The narrowing from "manufacturing kind environments" (which would have been too strong) to "removing context-dependence within a kind domain" is appropriately scoped.

---

## 4. One New Issue

**The paper has a citation hygiene problem.** Several references are to practitioner blog posts and community forum reports (nedcodes 2026, Montes 2026) that appear in Section 6.7 but are not in the reference list. The ecological prevalence evidence is strong, but if it is going to anchor a major section, the sources need proper citation. At minimum: URLs, access dates, and a note that these are community reports rather than peer-reviewed work. The reference list currently has gaps (Hu et al. 2025 has an "et al." with no other authors; the ACE framework citation in Section 2.4 uses a bracket citation [ACE2025] that does not resolve in the references section; [CompoundEng2026], [Anthropic2025], [MindMeld2025], and [Hardwick2026c] similarly lack full entries).

For a paper that stakes its credibility on transparency and public auditability, incomplete citations undermine the signal. Fix this before submission.

---

## 5. Summary Scorecard

| Concern from R1 | Status | Notes |
|---|---|---|
| System description reproducibility | Partially addressed | Autoethnographic reframe reduces the burden; appendix still recommended |
| Correction capture operationalization | Partially addressed | Better but trigger criterion still underspecified for replication |
| Competing explanations | Fully addressed | Section 6.4 is well argued |
| Szulanski mapping | Fully addressed | Extension explicitly acknowledged |
| Agent memory critique | Improved | Still needs tighter scoping |
| Cold-start / data flywheel | Not addressed | Missing from Discussion and Future Work |
| NEW: Citation hygiene | New issue | Multiple bracket citations unresolved; practitioner sources missing from references |

---

## 6. Bottom Line

The paper moved from "interesting phenomenon, underdeveloped argument" to "defensible contribution with clear scope." The competing explanations section is the single biggest improvement: the behavioral plane concept now earns its place by explaining what simpler alternatives cannot. The ecological prevalence evidence transforms the N=1 limitation from existential to contextual. The autoethnographic framing resolves the identity crisis between empirical systems paper and practitioner case study.

Three things to do before submission:

1. Fix the citation hygiene. This is mechanical but non-negotiable. Every bracket citation needs a reference entry. Every practitioner source needs a URL and access date.

2. Add a paragraph in Future Work on the data flywheel / cold-start problem. The compilation threshold is calibrated on an immature system. As the system matures and corrections become rarer, the threshold signal becomes noisier. This is a known dynamic in production ML and it connects the work to a well-understood challenge. One paragraph.

3. Scope the Section 2.5 agent memory critique to what the data supports. Two transfer events support a claim about correction-derived behavioral knowledge specifically. They do not support a general claim about all experiential memory in shared-memory architectures.

After those three fixes, the paper is ready for a serious venue. The contribution (two types with different transfer properties, the Szulanski barrier decomposition, the compilation pattern, and the ecological prevalence evidence) is real and actionable. The scope is honest. The limitations are stated rather than hidden. That is a publishable paper.
