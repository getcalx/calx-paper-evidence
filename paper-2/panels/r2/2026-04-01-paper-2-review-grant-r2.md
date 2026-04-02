# R2 Review: "Stickiness Without Resistance"

**Reviewer:** Adam Grant (organizational psychology lens)
**Date:** 2026-04-01
**Round:** 2 (responding to revised draft)
**Paper:** Hardwick, S. (2026). Stickiness Without Resistance: Knowledge Transfer Failure in Human-AI Collaboration Without Human Friction.

---

## 1. Overall Assessment

This is a substantially improved paper. The author took the structural feedback seriously and made the right cuts. The draft is leaner, the theoretical claims are more honest, and several of the decorative elements that would have drawn reviewer fire are gone. The absorptive capacity extension is now explicitly owned. The SECI model is cut. The System 1/System 2 mapping is cut. The autoethnographic framing is added and well-handled. The ecological prevalence section (6.7) is a smart addition that preempts the "N=1 so what" objection with practitioner convergence data.

The paper has moved from "interesting but theoretically underdisciplined" to "interesting and mostly disciplined, with two remaining gaps that need attention before submission." Those gaps: (1) causal ambiguity is still misidentified, and (2) the strongest theoretical implication of the findings (that resistance may enable transfer, not just impede it) remains unengaged.

I would now describe this as competitive at CSCW or a CHI workshop. With the two fixes below, it becomes a strong submission. Without them, it is still publishable but leaves its best contribution on the table.

---

## 2. What Was Fixed and How Well

**Absorptive capacity: Fixed, and well.** My R1 concern was that the paper slid between "lacks correction history" and "lacks experiential context" without acknowledging either as an extension of Cohen and Levinthal. The revision addresses this directly in Section 2.1:

> "We extend Cohen and Levinthal's (1990) original formulation here... the 'prior related knowledge' that matters for behavioral transfer is not domain expertise but the specific experience of having been wrong and corrected."

This is exactly the move I asked for. The author owns the extension rather than treating it as a straightforward application. The phrasing "This is an extension of the concept, not a direct application" is the right signal to reviewers. It says: I know what Cohen and Levinthal meant, I am going beyond it deliberately, and here is why the extension is warranted. A reviewer who wants to argue with the extension now has to argue with the reasoning, not catch the author in an unacknowledged redefinition.

One residual concern. In Section 6.1, the discussion of absorptive capacity reverts to slightly looser language: "the specific form that matters, the experiential context of having been wrong in the relevant way, is absent." This is the same idea but stated without the explicit "this is an extension" framing. A reader who jumps to the Discussion without reading Related Work carefully could miss that this is a deliberate conceptual move. Consider adding a single sentence in 6.1 that back-references the extension: "As argued in Section 2.1, this represents an extension of Cohen and Levinthal's formulation from domain knowledge to correction experience."

**SECI model: Cut entirely. Correct decision.** The forced mapping of socialization/externalization/combination/internalization onto human-agent dyads was the weakest theoretical move in the R1 draft. Cutting it removes the easiest target for Reviewer 2. No residual traces found in the text. Clean removal.

**System 1/System 2: Cut entirely. Correct decision.** The Kahneman mapping was decorative. Choice architecture (Thaler) was already doing the work. The revision no longer invokes dual-process theory at all. This is the right call. The paper now makes no claims about the cognitive architecture of LLMs, which means it cannot be attacked for misapplying cognitive theory to a system that does not have cognition.

**Autoethnographic framing: Added, well-handled.** Section 4.1 now explicitly names the methodology as autoethnographic and cites Ellis and Bochner (2000) and Adams et al. (2015). The framing is appropriately modest: "This approach trades external validity (a single practitioner's experience) for ecological validity (the corrections emerge from genuine production work, not experimental tasks)." This preempts the "this is just one person's experience" objection by naming the methodology and its tradeoffs. Reviewers in CSCW and CHI will recognize autoethnography as a legitimate approach. Reviewers in AMR or OrgSci will be more skeptical, but the paper was never going to AMR in its current form.

**Ecological prevalence section (6.7): New, and valuable.** This is the smartest addition in the revision. The 70+ community reports across six tools, the 0% compliance finding, the 60% compliance finding, and the practitioner convergence on "your CLAUDE.md is a suggestion, hooks make it law" all serve the same function: they establish that the phenomenon the paper formally characterizes is not idiosyncratic. The formal contribution is the mechanism (two types, the asymmetry, the Szulanski decomposition). The prevalence evidence establishes that the mechanism operates at ecosystem scale. This is the right division of labor between the formal study and the field evidence.

One caution. The section currently sits in the Discussion, which is fine for a workshop paper but may need to move to a separate section or an appendix for a full venue. At CSCW, a reviewer may ask whether the 70+ reports were systematically collected or cherry-picked. The reference to Hardwick (2026c) as "forthcoming" helps, but consider adding a sentence about the collection methodology (forums audited, date range, inclusion criteria) to preempt the question.

**Interaction orientation section (6.6): New, important, and adjacent to my strongest R1 suggestion.** This section names a real scope condition: the stickiness phenomenon documented here may only manifest when the human treats the agent as a persistent collaborator rather than a stateless tool. The insight that "the correction only gets captured as data when the human treats a repeated failure as a disappointment in an ongoing relationship rather than noise in a one-shot interaction" is sharp and honest. It is also doing some of the work that the "resistance as epistemic work" framing would do (more on this below).

**HyperAgents: Appropriately downweighted.** The abstract still mentions HyperAgents, contrary to my R1 suggestion to move it out of the abstract. However, the framing throughout the paper is now more careful. Section 5.5 explicitly states: "We do not claim that HyperAgents validates our findings." The weight is appropriate in the body. I would still prefer the abstract to focus on the core contribution and let HyperAgents appear in the findings, but this is a minor preference, not a structural concern.

---

## 3. What Was Not Fixed

**Kolb (1984): Still missing.** I flagged this in R1. The entire paper is about experiential learning: knowledge that forms through the experience of being corrected, not through receiving information. Kolb's experiential learning cycle (concrete experience, reflective observation, abstract conceptualization, active experimentation) maps directly onto the correction pipeline. The correction is the concrete experience. The rule distillation is abstract conceptualization. The next session is active experimentation. The recurrence is the cycle restarting because the abstract conceptualization (the rule) did not carry the concrete experience.

The absence is more conspicuous now that the SECI model is cut. In R1, SECI was at least gesturing at the knowledge conversion question, even if the gesture was wrong. With SECI gone, there is no framework in the paper for explaining the learning cycle itself. Argyris explains the single-loop/double-loop distinction. Szulanski explains why transfer fails. Lave and Wenger explain why situated participation matters. But nobody in the paper explains why experiential learning follows a cycle, which is Kolb's contribution. Add Kolb to Section 2.3 alongside Lave and Wenger.

**Argote (1999/2013): Still missing.** Linda Argote's work on organizational learning curves, knowledge depreciation, and the role of experience in knowledge retention is directly relevant. The behavioral plane degradation observation in Section 6.5 (the orchestrating agent's recall fidelity degraded over 43 days despite preserved artifacts) is a finding about knowledge depreciation. Argote documented the same phenomenon in manufacturing: organizations' production knowledge depreciates rapidly without reinforcing experience. The parallel is exact. Argote belongs in Section 2.1 alongside Szulanski, or in Section 6.5 where the degradation observation is discussed.

**Causal ambiguity: Still misidentified.** This was my second-most important R1 concern and it has not been addressed. The paper continues to use Szulanski's "causal ambiguity" to describe what is happening, but the mechanism in this system is not causal ambiguity in Szulanski's sense.

Szulanski's causal ambiguity means the source cannot articulate why the practice works. The knowledge is tacit at the source. The developer in this study can articulate exactly why each rule matters. He wrote them. He knows the failure that produced them. He knows the causal chain. The problem is not that the causal chain is unknown. The problem is that the causal chain does not survive compression into a rule.

Section 6.1 says: "The rule is causally ambiguous in Szulanski's sense: the artifact exists, but the causal chain that gives it force is compressed beyond recovery." But this is not Szulanski's sense. In Szulanski's sense, causal ambiguity means the source cannot explain why it works. Here, the source can explain. What fails is the codification: the explanation is compressed into a heuristic that loses the contextual force.

This matters because the distinction is not just terminological. If this is genuine causal ambiguity, the implication is that the developer should try harder to articulate the causal chain. If this is codification compression loss (the term I suggested in R1), the implication is that no amount of articulation will solve it, because the loss occurs in the format translation from experiential knowledge to rule artifact. These are different diagnoses with different practical implications.

The author has two options, both defensible:

Option A: Reframe as codification compression loss, position it as a new mechanism adjacent to but distinct from Szulanski's causal ambiguity. This strengthens the contribution: you are identifying a mechanism Szulanski did not describe, not just confirming one he did. The source can explain. The explanation does not survive the format.

Option B: Argue that causal ambiguity is a property of the artifact, not just the source. This would be a genuine extension of Szulanski: the causal chain becomes ambiguous not because the source cannot articulate it but because the artifact cannot carry it. This is defensible but requires explicit argument. Szulanski's original formulation locates ambiguity in the source's knowledge. Relocating it to the artifact's expressive capacity is a conceptual move that needs to be named and defended.

Either option works. What does not work is treating the current usage as a straightforward application of Szulanski's concept. A reviewer familiar with the knowledge transfer literature will catch this.

---

## 4. The Unengaged Insight: Resistance as Epistemic Work

My R1 review's strongest contribution was the suggestion in Section 5: what if resistance is not just a barrier to transfer but a mechanism that enables it? The logic: a resistant human recipient pushes back, asks why, negotiates implementation, and in doing so builds the contextual understanding that makes the transferred knowledge load-bearing. AI agents skip all of that. They comply without engaging. The compliance may be precisely why the rules do not stick.

The revised paper does not engage this directly. Section 6.6 (Interaction Orientation) is adjacent. It identifies that the phenomenon requires a specific interaction style (treating agents as persistent collaborators), and it notes that "the correction only gets captured as data when the human treats a repeated failure as a disappointment in an ongoing relationship." This is close to the resistance-as-epistemic-work idea but approaches it from the human side (the human's interaction orientation determines whether the phenomenon is visible) rather than from the agent side (the agent's lack of resistance determines whether the knowledge transfers).

The gap matters because the "resistance as epistemic work" framing would reposition the paper's central contribution. Currently the paper says: "We removed human friction. Stickiness persists. Therefore stickiness has a structural component independent of social barriers." The resistance-as-epistemic-work framing says something more provocative: "We removed human friction. Stickiness got worse. Therefore some of what we call friction was actually doing epistemic work."

These are not contradictory claims. Both are supported by the data. But the second is the one that would get this paper cited. It challenges a thirty-year assumption in the knowledge transfer literature (that all four of Szulanski's barriers are impediments) by suggesting that at least one barrier (arduous relationship) may serve a dual function. It connects to the productive conflict literature, to the psychological safety research (Edmondson 1999: psychological safety enables learning partly through conflict, not the absence of it), and to the broader organizational learning finding that some friction is functional.

The data supports this reading. The Mara-to-Thane case is the clearest example. Thane did not resist the rule. Thane complied with the rule. Thane still needed three independent recurrences to arrive at the same behavioral understanding. If Thane had resisted (pushed back, asked why, demanded justification), the contextual understanding might have formed during the resistance rather than through three rounds of failure. The resistance would have been a one-time cost. The compliance was a three-time cost.

I am not asking the author to restructure the paper around this idea. I am asking for a substantive paragraph in the Discussion, probably in Section 6.1 or as a new 6.8, that engages this alternative interpretation. Something like:

"An alternative reading of these findings challenges the assumption that Szulanski's four barriers are uniformly impediments. In human teams, a resistant recipient engages with the transferred knowledge: questioning its applicability, negotiating its implementation, demanding justification. This friction forces contextual processing that may be absent in compliant AI agents. The agent reads the rule and follows it literally. The resistant human reads the rule and fights with it, and in fighting, builds the situated understanding that makes the rule load-bearing. If this interpretation holds, removing relational friction does not merely reveal structural barriers. It removes a mechanism that was, in human systems, partially compensating for those structural barriers. The implication is that some of what the knowledge transfer literature classifies as impediment may function simultaneously as enabler. This reframing warrants investigation in human-human transfer studies where relational friction can be experimentally varied."

This paragraph would cost the author 150 words and gain the paper its most citable claim.

---

## 5. Minor Issues

1. **The abstract is still dense.** It runs to a single paragraph containing the full evidence summary, both study descriptions, six contributions, and the HyperAgents convergence. For a workshop submission this is fine. For a full venue, consider breaking it into two paragraphs: one for the phenomenon and evidence, one for the mechanism and contributions.

2. **Section 2.1 is doing a lot of work.** It introduces Szulanski, extends absorptive capacity, maps the four barriers, and states the paper's core contribution. This is the section reviewers will read most carefully. Consider breaking it into 2.1 (Szulanski's framework) and 2.2 (Our extension), with the current 2.2 becoming 2.3 and so on. The section numbers would shift but the theoretical moves would be more legible.

3. **The "approximately" qualifiers are appropriate but become rhythmically heavy.** "Approximately 50%," "approximately 3 to 4 recurrences," "approximately 2.9 corrections per specification," "approximately 57 rules." The hedging is correct given the data. But when every number is approximate, the reader starts to wonder if any number is precise. Consider stating exact numbers where you have them (151 corrections, 237 rules, 44 new corrections, 13 in addressed categories) and reserving "approximately" for genuinely estimated values.

4. **Section 7.7 (Interaction Orientation) duplicates Section 6.6.** The limitation and the discussion section make the same point. The limitation section should reference the discussion rather than restating it. One sentence: "As discussed in Section 6.6, the findings describe a specific interaction regime. Whether the same dynamics emerge under transactional interaction styles is unknown."

5. **Reference list has formatting gaps.** There are blank lines between some references (after Hu et al. 2025, after Szulanski 2000) that suggest deleted entries. Clean up for submission.

---

## 6. Summary Verdict

The paper has improved meaningfully on four of my six R1 concerns. Absorptive capacity is now explicitly owned as an extension. SECI is cut. System 1/System 2 is cut. The autoethnographic framing and ecological prevalence section are valuable additions.

Two concerns remain unaddressed:

1. **Causal ambiguity is still mislabeled.** The mechanism in this system is codification compression loss, not causal ambiguity in Szulanski's sense. This is both a terminological issue and a missed opportunity: naming the new mechanism strengthens the contribution.

2. **Resistance as epistemic work is still unengaged.** The interaction orientation section (6.6) is adjacent but does not make the provocative claim that resistance may enable transfer. This is the paper's most citable potential insight and it costs one paragraph.

Two citations remain missing:

3. **Kolb (1984)** belongs alongside Lave and Wenger in Section 2.3. The experiential learning cycle maps directly onto the correction pipeline.

4. **Argote (1999/2013)** belongs alongside Szulanski in Section 2.1 or in Section 6.5 where behavioral plane degradation is discussed.

With these four fixes, this paper is a strong CSCW submission. The core contribution (stickiness persists without human friction, two types with different transfer properties) is real and well-evidenced. The theoretical grounding is now mostly solid. The remaining work is precision: naming the right mechanism, citing the right frameworks, and engaging the strongest alternative interpretation of your own findings.

The "resistance as epistemic work" paragraph is the single highest-value addition remaining. It turns a solid empirical contribution into a paper that challenges a foundational assumption in the knowledge transfer literature. That is the difference between a paper that gets published and a paper that gets cited.
