# Second Review: "Stickiness Without Resistance" (Revised Draft)

**Reviewer:** Daniel Kahneman (simulated perspective)
**Date:** 2026-04-01
**Paper:** Hardwick, S. (2026). Stickiness Without Resistance: Knowledge Transfer Failure in Human-AI Collaboration Without Human Friction.
**Review round:** R2 (responding to revisions from R1 feedback)

---

## 1. Overall Assessment

This is a substantially better paper than the one I reviewed initially. The author has taken the feedback seriously and made disciplined choices. The tone is more cautious where it should be, the competing explanations are present, and the System 1/2 material is gone entirely. The paper now reads as what it is: a careful observational study with a genuine empirical finding, presented honestly with its limitations front and center. My concerns have narrowed from structural to tactical.

---

## 2. What Improved

**Cutting System 1/2 was the right call.** I will address this in detail in Section 4, but the short version: the revised paper is cleaner without it. Section 6.2 now develops the compilation architecture through Argyris's single-loop/double-loop framework, which is a natural fit because the distinction actually applies. The new framing does not import assumptions about cognitive architecture that do not hold. This is a net gain in rigor for a small loss in rhetorical reach.

**The competing explanations in Section 6.4 are honest and well-constructed.** Three alternatives (prompt brittleness, context-specific encoding, attention recency) are stated clearly, and each is evaluated against the data rather than dismissed. The structure is correct: state the simpler explanation, show what it accounts for, show what it does not account for. The key move is noting that none of the alternatives explain the architectural/process asymmetry, because all three would predict that architectural knowledge should also degrade. It does not. This is a strong paragraph. The paper earns the right to its preferred explanation by showing where the alternatives fall short rather than by ignoring them.

**"Compilation threshold" downgraded to "compilation pattern."** The paper now consistently signals that three data points constitute a preliminary observation, not an established parameter. The circularity risk (Section 5.4: the system was designed with a threshold of 3, so observing transitions near 3 may be a design artifact) is acknowledged explicitly. This is exactly right. My R1 concern was that the paper was treating a design parameter as a discovery. The revised version draws the distinction clearly.

**The HyperAgents treatment is tighter.** The convergence is noted, the differences are specified (environment-external versus agent-internal, human-agent dyads versus agent-only), and the paper does not claim more validation than the parallel provides. Section 5.5 is about the right length for what the evidence supports.

**The ecological prevalence section (6.7) is a useful addition.** The practitioner discourse evidence (70+ community reports, the 0% compliance measurement, the 60% compliance measurement) establishes that the phenomenon is not idiosyncratic to this researcher. The distinction between formal contribution (the mechanism) and ecological support (the prevalence) is stated clearly. The Montes quote ("Your CLAUDE.md is a suggestion. Hooks make it law") is effective because it is a practitioner independently arriving at the architectural/process distinction without the theoretical vocabulary. This section answers the "so what?" question that a reviewer unfamiliar with the AI tooling ecosystem would ask.

**The interaction orientation scope condition (Section 6.6 and Limitation 7.7) is important and well-handled.** The paper now acknowledges that the phenomenon may only be observable when the human treats agents as persistent collaborators rather than disposable tools. This is intellectually honest and it sharpens the contribution: the findings describe a specific interaction regime that is becoming more common. Declaring the boundary makes the paper stronger, not weaker.

---

## 3. What Still Needs Work

**The human-recognition insight was not adequately addressed.** This was the central point in my Section 5 ("One Thing the Author Probably Has Not Considered"), and it is the most important piece of feedback I gave. Let me restate it precisely.

The paper treats the compilation event as something that happens to the correction. A process correction recurs 3 to 4 times, is "identified as needing architectural intervention," and is then compiled into a mechanism. But identified by whom? Not by the agent. The agent does not recognize that its own corrections are failing. The human recognizes it. The compilation event is a change in the human's theory of intervention, not a change in the agent's behavior.

The eight-entry mock patching chain is not eight instances of the agent failing to learn. It is seven instances of the human failing to recognize that the intervention surface is wrong, followed by one instance of the human switching from "instruct the agent differently" to "change the environment." The "compilation threshold" is the point at which the human abandons the mental model that the agent is a knowledge recipient who can be taught through rules and adopts the model that the agent is an environment to be engineered.

The revised paper comes close to this in Section 6.2: "the system recognizes that single-loop correction has failed and the intervention surface must shift." But "the system" is doing a lot of work in that sentence. The system did not recognize anything. Spencer recognized it. The compilation event lives in the human, not in the correction, not in the agent, not in "the system." This matters because it changes what the compilation pattern measures. The paper presents it as measuring a property of correction dynamics. I believe it measures a property of human learning about correction dynamics.

The revised draft added a sentence in Section 2.2 noting that "the barrier is partly informational (recognizing that a structural change is needed requires multiple instances of failure) rather than purely psychological." This is a step in the right direction, but it does not follow through. "Informational" is vague. The specific information that accumulates is the human's evidence that their current intervention strategy is wrong. The compilation threshold is the human's evidence threshold for abandoning one theory and adopting another. This is classic belief updating under repeated disconfirmation, and it connects to my own work on anchoring and adjustment: the human is anchored on the "instruct the agent" model and requires 3 to 4 disconfirmations to adjust to the "engineer the environment" model. The adjustment is insufficient early on (the human tries the same class of intervention again) and then snaps to a new frame.

This does not require a major rewrite. A paragraph in Section 6.2, after the Argyris discussion, making explicit that the compilation event is a human recognition event would be sufficient. Something like: the transition from single-loop to double-loop is not a property of the correction itself but of the human's theory of intervention. The human, not the agent, recognizes that the intervention surface must change. The compilation pattern, if it holds under replication, may measure the human's evidence threshold for abandoning a text-instruction model and adopting an environmental-engineering model. This connects to the bilateral nature of correction discussed in Section 6.6: the human is being calibrated by the correction process, and the compilation event is the moment the calibration shifts.

This paragraph would strengthen the paper considerably because it resolves an ambiguity that a careful reader will notice. Throughout the paper, compilation is described in passive voice ("corrections are identified as needing architectural intervention," "the system recognizes"). The active agent in the compilation decision is the human. Naming this explicitly makes the finding deeper: not only does knowledge fail to transfer between agents, but the human's recognition of when knowledge needs to be compiled rather than transferred is itself a learning process that requires repeated failure.

**The absorptive capacity extension is better but still slightly loose.** The revised Section 2.1 and 6.1 now explicitly frame the extension: "the prior related knowledge that matters for behavioral transfer is not domain expertise but the specific experience of having been wrong and corrected." This is clearer than R1. But the paper still uses "absorptive capacity" as if the extension is straightforward. Cohen and Levinthal's construct is about the ability to recognize, assimilate, and apply external knowledge. They were concerned with R&D spillovers and scientific knowledge. Extending this to "experiential correction history" is defensible but it is an extension, and the paper should signal this more clearly. One additional sentence noting that this is an analogical extension of the original construct (not a direct application) would satisfy me. Section 2.1 says "we extend Cohen and Levinthal's original formulation here," which is good, but then uses the term unqualified throughout the rest of the paper as if the extension has been established.

**The error floor discussion (Section 5.6) is speculative and could be tightened.** The reinterpretation (the floor represents advancing frontier, not stagnation) is interesting but the evidence for it is thin: the author's assertion that Phase 3 specifications were harder. This is plausible but unquantified. If specification difficulty cannot be measured, the section should be shorter. Two sentences stating the observation and the alternative interpretation would be sufficient. The current length implies more analytical weight than the data supports.

**Section 6.5 on behavioral degradation is a qualitative claim without measurement.** The paper acknowledges this ("we present this as a qualitative observation, not a measured finding") but then spends a full section drawing implications. The implications are interesting (the behavioral plane requires interaction to maintain, not just storage), but they rest on an unmeasured observation. If the observation cannot be quantified in this paper, the section should be shorter, perhaps three sentences noting the phenomenon and flagging it for future work, rather than a full discussion section.

---

## 4. Was Cutting System 1/2 the Right Call?

Yes. But I want to explain why, because the alternative (reframing rather than cutting) was also viable, and the reasons for preferring the cut are instructive.

The problem with the System 1/2 mapping was not that it pointed in the wrong direction. It pointed in approximately the right direction: there is a meaningful parallel between effortful rule-following and automatic environmental constraint. The problem was that the mapping imported mechanistic assumptions that do not hold. System 1 is a learning system. A type constraint is not. System 2 involves sequential processing drawing on working memory. An LLM following a rule in its context window is doing a single forward pass. The analogy worked at the level of "automatic versus deliberate" and failed at every level below that.

The reframing I suggested in R1 ("the parallel is structural rather than mechanistic") could have worked. But cutting was better for three reasons.

First, the Argyris framework does the same work more precisely. Single-loop versus double-loop maps onto the same distinction (correcting within existing structure versus changing the structure) without importing cognitive architecture assumptions. The paper did not lose explanatory power by substituting one framework for the other. It gained precision.

Second, the System 1/2 reference was a reviewer magnet. Any reviewer who knows my work would spend a paragraph on why the mapping does not hold, and the author would have to defend a tangential point. Cutting it removes a predictable objection without losing content. This is a pragmatic consideration, but pragmatic considerations matter for publication.

Third, and most importantly, the paper is about knowledge transfer, not cognitive architecture. The System 1/2 mapping was an explanatory detour that did not serve the paper's central argument. Szulanski's framework plus Argyris's learning loops plus the empirical data is sufficient. Adding dual-process theory on top was decorative rather than structural. The revised paper is leaner and every theoretical reference now earns its place.

The one thing lost by cutting is the connection to the experiencing self/remembering self distinction, which I raised in Section 5 of R1. The experiencing self (the agent that has been corrected) creates behavioral change through the experience. The remembering self (the agent that reads the rule) has the memory but not the experience. That distinction could have been preserved without the full System 1/2 apparatus, as a standalone observation about the difference between experiencing and remembering a correction. It is implicit in the paper's discussion of absorptive capacity ("the experiential context of having been wrong") but never made explicit through that particular lens. A small loss, not a critical one.

---

## 5. Specific Suggestions for R3

1. Add a paragraph in Section 6.2 naming the human as the active agent in compilation events. The compilation pattern measures the human's evidence threshold, not a property of the correction or the agent.

2. Add one sentence in Section 6.1 flagging the absorptive capacity extension as analogical: "We note that this extends Cohen and Levinthal's construct beyond its original domain of R&D knowledge spillovers; the extension is analogical rather than definitional."

3. Shorten Section 5.6 (error floor) to two to three sentences: the observation and one alternative interpretation. The current length overpromises.

4. Shorten Section 6.5 (behavioral degradation) to a brief observation with a pointer to future work. The qualitative claim does not support a full discussion section.

5. Consider adding a single sentence somewhere in the discussion connecting the recursive meta-evidence to the human-recognition point: the 41% rate on the correction system itself suggests that even when the human is maximally aware of the phenomenon, the human's own correction strategy still requires multiple iterations to converge. The researcher's corrections about correction-tracking are the researcher learning how to correct, not just the agents learning to be corrected.

---

## 6. Verdict

The paper has improved materially. The System 1/2 cut was the right decision. The competing explanations are well-handled. The hedging on the compilation pattern is now appropriate. The ecological prevalence section adds value.

The main remaining gap is the human-recognition point. The paper still treats compilation as something that happens to corrections or within "the system" rather than something the human decides after accumulating enough evidence that text-based intervention is failing. This is not a minor framing issue. It is the difference between "corrections have a compilation threshold" and "humans have a threshold for recognizing that their intervention model is wrong." The second version is more interesting, more defensible, and sitting in the data. It also connects the paper more tightly to the organizational learning literature: Argyris's original argument was that double-loop learning is difficult because it requires the learner to question their own assumptions. Your compilation event is the human questioning the assumption that agents can be taught through text. Name it.

The paper is publishable with these revisions. The empirical contribution is solid and the theoretical framing, while it still has room to sharpen, is now honest about what it claims and what it does not.

---

*Review prepared as pre-submission feedback, R2. Progress from R1 is clear. The remaining items are refinements, not structural problems.*
