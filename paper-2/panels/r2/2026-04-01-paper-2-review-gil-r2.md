# Re-Review: "Stickiness Without Resistance" (Revised Draft)

**Reviewer:** Yolanda Gil
**Angle:** AI research methodology and scientific rigor
**Date:** April 1, 2026
**Review round:** R2 (responding to revisions from R1 feedback)
**Target venues cited by author:** CHI 2027, CSCW

---

## 1. Summary of Revisions Assessed

The author addressed four of six major issues from R1:

| R1 Issue | Status | Assessment |
|---|---|---|
| Name method as autoethnography | Addressed | Partially sufficient |
| Demote accidental control group | Addressed | Well handled |
| Demote HyperAgents convergence | Partially addressed | Improved but still overclaimed |
| Add descriptive statistics | Not addressed | Critical gap remains |
| Formalize classification codebook | Not addressed | Critical gap remains |
| Engage knowledge representation literature | Not addressed | Serious gap remains |
| Add at least one figure | Not addressed | Still zero figures |

Two revisions not flagged in R1 were also made: the compilation threshold was downgraded to a "preliminary pattern" with circularity risk acknowledged (Section 5.4), and an ecological prevalence section (6.7) was added drawing on practitioner reports and academic mechanism papers. Both are positive additions.

---

## 2. What Improved

**The accidental control group is properly scoped.** The revision in Section 5.3 and Contribution 3 is exactly what I asked for. The language now reads "suggestive rather than conclusive," the alternative explanation (domains were simply unused) is stated plainly, and the contribution is framed as "a pattern warranting designed replication" rather than a standalone finding. This is honest scholarship. Reviewers will respect it.

**The autoethnographic framing exists.** Section 4.1 now names the method, cites Ellis and Bochner (2000) and Adams et al. (2015), and states the tradeoff between external and ecological validity. This is a meaningful improvement over the previous draft, which read as ad hoc practitioner journaling.

**The compilation threshold is properly hedged.** Section 5.4 now names the circularity risk explicitly: the system was designed with a threshold of 3, so observing transitions near 3 to 4 may reflect a design parameter rather than an emergent property. The contribution is reframed as "a testable hypothesis for future work." This is appropriately cautious for the strength of the evidence.

**The ecological prevalence section adds breadth.** Section 6.7 is new and useful. The practitioner discourse audit (70+ reports across six tools), the controlled compliance measurements (0% in one test, 60% in another), and the connection to positional attention bias research (Liu et al. 2023, Peysakhovich and Lerer 2023, Wang et al. 2026) all serve the paper well. This section does something the first draft could not: it positions the N=1 formal study within a broader ecosystem of converging observations. The "CLAUDE.md is a suggestion, hooks make it law" practitioner quote is a clean restatement of the paper's central distinction. This section earns its place.

---

## 3. What Remains Insufficient

### 3.1 The Autoethnographic Framing Is Thin

The revision names autoethnography and cites two sources. This clears the lowest bar. It does not clear the bar for CHI or CSCW, where autoethnographic methods are well-understood and reviewers will expect methodological depth.

Specific problems:

**The citation base is too narrow.** Ellis and Bochner (2000) is the foundational reference, appropriate as a starting point. Adams et al. (2015) is a general qualitative methods chapter, not an HCI autoethnography paper. The paper does not cite any of the HCI-specific autoethnographic work that would demonstrate familiarity with the tradition as practiced in the target venues. Desjardins and Ball (2018) on autobiographical design, Lucero (2018) on first-person research methods in HCI, Cunningham and Jones (2005) on autoethnographic approaches to technology use: these are the papers that CHI and CSCW reviewers will look for. Their absence signals that the autoethnographic framing was adopted as a label rather than engaged as a methodology.

**No discussion of autoethnographic validity criteria.** The autoethnographic tradition has its own standards for rigor, distinct from positivist reliability and validity. Ellis, Adams, and Bochner (2011) articulate these as: the work must be substantive (contribute to understanding of social life), aesthetic (engage the reader analytically and emotionally), reflexive (demonstrate awareness of the researcher's role in constructing the account), impactful (generate new questions or move the audience), and ethical (be faithful to lived experience). The current draft satisfies substantive, reflexive (partially), and ethical criteria. It makes no attempt to address the others. I am not suggesting the paper needs to satisfy aesthetic criteria in the literary sense. I am suggesting it needs to demonstrate awareness that autoethnographic validity is not the same as experimental validity, and that the tradition has articulated criteria the author should engage with.

**The known threats are listed but not analyzed.** Section 4.1 mentions "autoethnographic limitations" and compensates with "structured capture protocols, prospective classification criteria, and full public availability." This is fine as far as it goes. But the specific threats to autoethnographic research in this context are not named. Confirmation bias: the researcher who designed the correction framework is predisposed to find that corrections matter. Selective attention: what corrections were NOT logged? The paper never discusses the denominator problem: how many agent errors occurred that were not captured as corrections? What was the capture rate? Motivated reasoning: the researcher is building a product (Calx) that operationalizes correction dynamics. The commercial interest in finding that corrections are important and that they require tooling is a conflict of interest that should be disclosed, not merely gestured at.

**Recommendation:** Expand the autoethnography paragraph into a full subsection (4.1.1 or similar). Cite 2 to 3 HCI autoethnography papers. Name confirmation bias, selective attention (the denominator problem), and commercial interest as specific threats. State how each is mitigated or acknowledged. This is approximately 300 to 400 words and 3 to 4 additional citations.

### 3.2 Descriptive Statistics Are Still Missing

This was item 4.3 in my R1 review. The revision did not address it.

The paper still says "approximately 50% persistence" for process corrections without stating the count and denominator. How many total process corrections? How many recurred at least once? How many recurred 2+ times? The paper lists five recurrent error classes in Section 5.2 with entry counts (8, 5, 5, 3, 4 = 25 entries across 5 classes), but it does not state: how many total process corrections were there, and what proportion of them showed any recurrence? The 50% figure is ungrounded.

Similarly: of 151 total corrections, what is the split between architectural and process? The paper names 6 architectural error classes and 5 recurrent process error classes from Study 1, but that accounts for 11 error classes, not 134 corrections. Many corrections presumably fell into classes that did not recur. What happened to them? Were they all architectural? Were some process corrections that happened not to recur? The absence of a basic frequency table is a problem.

What is needed, at minimum:

1. A 2x2 contingency table: correction type (architectural vs. process) by recurrence (zero vs. one or more). Four cells, four counts, a Fisher's exact test or chi-square. This is the single most important addition the paper could make.

2. For process corrections: a distribution of recurrence counts. How many recurred once, twice, three times, four or more?

3. For Study 2: the same breakdown for 17 corrections.

This is not optional for a CHI or CSCW submission. Reviewers will reject without it. You have 151 data points. Report basic counts.

### 3.3 The Classification Codebook Is Still Missing

This was item 4.1 in my R1 review. Section 4.2 provides a narrative definition that is improved over the first draft: the architectural/process distinction is stated more crisply, the prospective criterion is explained, the borderline case is flagged. But there is still no operational codebook.

An operational codebook specifies: (1) the decision procedure (a series of questions or criteria applied in order), (2) boundary cases with worked examples (what about a configuration file change? what about a prompt template restructuring?), and (3) exclusion criteria (what does NOT count as a correction at all?).

Section 4.2 provides a definition and one borderline example. That is not a codebook. A reviewer at CHI who studies classification reliability will ask: could I hand this to a graduate student and get the same classifications? The current text does not support that.

I understand that producing inter-rater reliability data before submission may not be feasible with the author's resources. The codebook itself, however, is a writing task, not a data collection task. It can be included as an appendix. The publicly available correction logs are a start, but without the codebook a reader cannot independently verify the classifications.

**Recommendation:** Add an appendix with the operational codebook. Include 3 to 4 worked boundary examples from the actual correction log, showing how the criterion was applied. This signals to reviewers that the classification is reproducible even if inter-rater reliability has not yet been tested.

### 3.4 Knowledge Representation Literature: Still Absent

This was my "one thing the author probably hasn't considered" in R1, specifically flagged as a serious gap. The revision did not address it. No citations were added for the frame problem, Newell's knowledge level, or workflow constraint systems. The theoretical frame remains organizational psychology plus behavioral economics, with no engagement with the forty-year AI knowledge representation tradition that has studied precisely the phenomenon this paper documents.

I will not repeat the full argument from R1. The short version: the paper's central finding is that declarative behavioral knowledge fails to produce reliable behavioral change in computational agents while procedural/structural interventions succeed. This is a version of the declarative-procedural gap that knowledge engineering has characterized formally. Not engaging this literature leaves the paper theoretically incomplete and misses an opportunity to position the finding as convergent evidence from a new setting for a known architectural pattern.

The specific additions I recommended in R1 remain appropriate: McCarthy and Hayes (1969) on the frame problem, Newell (1982) on the knowledge level, and workflow constraint systems (Gil et al. 2011, Gil and Ratnakar 2013) as direct analogues to the architectural correction mechanism. A subsection in Related Work (2.8 or a new 2.9, renumbering the current summary) and a paragraph in Discussion would suffice. Approximately 500 words and 5 to 6 citations.

I flag this as the single largest theoretical gap remaining in the paper. A reviewer with a knowledge engineering background will notice the absence immediately. At CHI, which draws reviewers from HCI, AI, and cognitive science, the probability of getting such a reviewer is non-trivial.

### 3.5 Still Zero Figures

The paper has no visual representation of any finding. For a venue like CHI, this is almost disqualifying on its own. The architectural/process asymmetry, which is the paper's core empirical contribution, would be dramatically more legible as a figure.

Two figures would transform the paper:

**Figure 1: Correction type by recurrence.** A simple grouped bar chart or the 2x2 contingency table visualized. Architectural corrections on the left with a zero bar for recurrence. Process corrections on the right with bars showing the recurrence distribution. This makes the asymmetry visible at a glance.

**Figure 2: Correction timeline.** A timeline spanning the 43-day study period showing when corrections were logged, color-coded by type. Architectural corrections would appear as single points (one occurrence, then zero). Process corrections would appear as repeated marks at the same vertical position (showing the recurrence chains). The eight-entry mock patching chain would be immediately visible as a sawtooth pattern. This figure would replace several paragraphs of text with a single visual that reviewers could evaluate in seconds.

Either figure alone would be sufficient. Both together would meaningfully strengthen the paper.

### 3.6 HyperAgents Convergence: Improved But Still Overclaimed

The revision improved the HyperAgents treatment. Section 5.5 now states "We do not claim that HyperAgents validates our findings" and acknowledges that "the systems, metrics, and experimental conditions are too different for direct confirmation." Section 2.7 enumerates key differences. This is better.

However, it is still listed as Contribution 5 in the introduction: "Independent convergence with HyperAgents on the boundary between declarative and procedural behavioral learning." R1 recommended demoting this from a contribution to a discussion point. The body text was toned down, but the contributions list was not updated. Contribution 5 still claims "independent convergence," which overstates what the evidence supports. Two groups observing that some improvements transfer better than others, in fundamentally different systems with fundamentally different methods, is an interesting parallel. It is not a contribution of this paper. It is context.

**Recommendation:** Remove from the numbered contributions list. Keep the discussion in Sections 2.7 and 5.5, which are now reasonably hedged.

---

## 4. New Issues Identified in This Draft

### 4.1 Section 6.7 Needs Tighter Sourcing

The ecological prevalence section is a good addition, but the sourcing is informal. "nedcodes, 2026" and "Montes, 2026" appear to be practitioner blog posts or social media accounts. For a CHI submission, these need proper citations: author name, year, title, platform/publication, URL, date accessed. "Over 70 distinct community reports" should cite the forthcoming synthesis [Hardwick2026c] more precisely: is this a systematic review? What were the search terms? What platforms were included? What was the inclusion criterion for a "report"?

The academic citations in this section (Liu et al. 2023, Peysakhovich and Lerer 2023, Wang et al. 2026) are properly cited and strengthen the mechanistic argument. The practitioner citations need the same rigor.

### 4.2 The Contributions List Needs Pruning

The paper currently claims six contributions. R1 recommended demoting two (accidental control group and HyperAgents convergence). The revision demoted the accidental control group in substance (Contribution 3 is now hedged as "suggestive") but kept it in the list. HyperAgents remains Contribution 5.

For a CHI submission, four strong contributions are better than six where two require extensive qualification. I recommend:

1. Stickiness persists after removing human friction (strong, novel, well-supported).
2. Two types with predictably different transfer properties (strong, the core empirical finding, needs the contingency table to be fully convincing).
3. Recursive meta-evidence (current Contribution 6, interesting and well-scoped).
4. The compilation pattern as a testable hypothesis (current Contribution 4, appropriately hedged).

Contributions 3 (accidental control group) and 5 (HyperAgents convergence) become discussion points. The paper is tighter with four contributions that all stand on their own evidence.

---

## 5. Verdict: Is This Closer to Submission Readiness?

Yes, materially closer. The accidental control group handling, the compilation threshold hedging, the autoethnographic framing (even if thin), and the ecological prevalence section all represent real improvements. The paper reads as more self-aware and more methodologically honest than the first draft.

It is not yet ready to submit. Three gaps are blocking:

1. **Descriptive statistics and the contingency table.** No CHI or CSCW reviewer will accept "approximately 50% persistence" without counts, denominators, and a basic statistical test. This is the fastest fix and the highest-impact single addition. Estimated effort: 2 to 3 hours of data tabulation and one paragraph of text.

2. **The classification codebook.** Without it, the single-rater limitation is not just acknowledged but unaddressed. An appendix with the operational codebook and worked boundary examples signals that the classification is reproducible in principle. Estimated effort: half a day.

3. **At least one figure.** CHI is a venue that expects visual communication of empirical findings. A paper with 151 data points and zero figures will be flagged by every reviewer. Estimated effort: a few hours for a well-designed correction-type-by-recurrence chart.

Two gaps are important but not blocking:

4. **Knowledge representation literature.** The paper functions without it, but it is theoretically incomplete. A reviewer with a KR background will see the gap as a missed opportunity at best and ignorance of relevant prior art at worst. I continue to recommend addressing it.

5. **Deeper autoethnographic framing.** The current treatment clears the minimum bar. Expanding it with HCI-specific citations and explicit threat analysis would preempt reviewer objections. It is the difference between a reviewer noting "thin methodology" and a reviewer noting "methodology is acknowledged and bounded."

If the three blocking items are addressed and the contributions list is tightened to four, this paper is a credible CHI 2027 submission. The core finding (stickiness without human friction, two types, the asymmetry) is novel, the ecological prevalence section provides breadth, and the honest limitation handling builds trust. The remaining question is whether the methodological apparatus is strong enough to survive three reviewers, and right now it needs the statistics, the codebook, and the figure to do that.

---

*Yolanda Gil is a Research Professor of Computer Science and Spatial Sciences at the University of Southern California and a Fellow of the Association for the Advancement of Artificial Intelligence (AAAI). Her research focuses on intelligent interfaces for knowledge capture and discovery, including work on collaborative knowledge-based systems, scientific workflows, knowledge engineering, and provenance. She directed the Information Sciences Institute's Interactive Knowledge Capture group and has published extensively on how to make knowledge systems that are usable, reproducible, and scientifically rigorous.*
