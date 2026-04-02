# Pre-Submission Review: "Stickiness Without Resistance"

**Reviewer:** Daniel Kahneman (simulated perspective)
**Date:** 2026-04-01
**Paper:** Hardwick, S. (2026). Stickiness Without Resistance: Knowledge Transfer Failure in Human-AI Collaboration Without Human Friction.

---

## 1. Overall Assessment

This is a well-constructed observational study that makes a genuine contribution by demonstrating knowledge transfer stickiness in a system where the standard human explanations have been removed by design. The two-type distinction (architectural vs. process corrections) is the paper's strongest empirical claim and it holds up: the data clearly shows the separation. The theoretical framing draws on the right literature but overreaches in places, particularly in Section 6.2 where my dual-process framework is applied to LLM agents in a way that is suggestive but not rigorous.

---

## 2. What Works

**The natural experiment structure is honest and compelling.** The author did not design this as an experiment. He tried to make transfer work, it failed, and he studied the failure. This is more convincing than a designed study would be at this stage, because the researcher was actively motivated to see the opposite result. Motivated reasoning should have pushed toward finding that transfer succeeded. It did not. This is good epistemic practice and the paper is transparent about it.

**The two-type distinction has empirical teeth.** Zero recurrence for architectural corrections across 43 days and two studies, versus approximately 50% persistence and chains up to 8 entries for process corrections. This is not a theoretical claim dressed up as data. The separation is sharp, it holds across both studies, and it is classifiable at capture time rather than retroactively. The prospective classification criterion is methodologically important and the author correctly emphasizes it.

**The recursive meta-evidence is worth more than the paper claims.** Building the correction-tracking tool while tracking corrections about correction-tracking, and finding that 41% of Study 2 corrections concern the system itself, is a strong robustness signal. The phenomenon persists when the researcher is maximally aware of it. In my framework, this is informative because awareness is precisely what should activate System 2 scrutiny. If knowing about the effect were sufficient to eliminate it, the rate should drop. It does not. The paper treats this as a methodological note. It could be a primary finding.

**The accidental control group is handled with appropriate caution.** The author correctly identifies that he cannot distinguish "rules without correction cycles are inert" from "those domains were unused." He presents it as suggestive. This is the right level of confidence for post-hoc discovery.

---

## 3. What Doesn't Work or Needs Strengthening

**The System 1/System 2 mapping in Section 6.2 is a loose analogy, not a mechanistic explanation.** This is my primary concern. The paper states that "text-based rules operate as System 2 knowledge" and "compiled mechanisms operate as System 1 analogs" with what it calls precision. But this is not precise. It is evocative.

System 1 and System 2 are descriptions of cognitive processing in biological minds with specific properties: System 1 is associative, operates on learned patterns, produces automatic responses shaped by extensive experience. System 2 is sequential, rule-following, and draws on working memory. When I say System 1 is "always running," I mean there is a continuous parallel processing stream that evaluates stimuli against learned patterns and produces outputs without conscious access.

LLM agents do not have two processing systems. They have one: a forward pass through a transformer that produces a token distribution. When an agent "follows a rule" from its context window, it is not engaging a separate deliberative system. The rule is part of the input that shapes the same single forward pass. When a type system prevents an invalid input, the agent is not being constrained by an "automatic" system operating below deliberation. The agent simply never encounters the error state because the environment has been modified.

The actual mechanism your data reveals is simpler and does not require dual-process theory to explain: environmental constraints are more reliable than instructional constraints because they remove the possibility of error rather than relying on correct recall and application of a rule within a context window. This is a finding about the difference between removing a degree of freedom and adding information to a decision process. It does not require System 1 and System 2 as explanatory constructs.

The analogy is not wrong in the sense that it points in the right direction. But it obscures more than it reveals because it imports assumptions about cognitive architecture that do not apply. A reader familiar with my work will notice immediately that the mapping lacks the core property: System 1 is a learning system that improves through exposure. A type system does not learn. It is a static constraint. Calling it a "System 1 analog" conflates automatic enforcement with automatic cognition. These are categorically different.

**The Szulanski mapping is slightly too neat.** You argue that removing motivation and relationship quality leaves causal ambiguity and absorptive capacity as the surviving barriers. This is clean and I understand why it is appealing for the paper's structure. But consider whether "absorptive capacity" is really the right construct for what you are observing. Cohen and Levinthal (1990) defined absorptive capacity as prior related knowledge that allows an organization to recognize and assimilate external knowledge. Your agents have extensive prior knowledge. What they lack is the specific experiential history of having been corrected. This is closer to what I would call the difference between knowing a rule and having a conditioned response. The absorptive capacity framing works at a high level but it may be doing some concept-stretching that weakens the argument for readers who know that literature well.

**The compilation threshold claim needs more hedging.** Three data points at 3-4 recurrences is a pattern, not a threshold. The paper acknowledges this ("the data points are few") but then uses "compilation threshold" throughout as if it were an established parameter. The language needs to consistently signal that this is an observed pattern in a small sample, not a measured constant.

**The "stickiness without friction" framing, while catchy, may be less surprising than the paper implies.** Consider: if I give you a written procedure for tying a specific knot, you can read it and attempt to follow it. If I instead give you a jig that physically guides the rope into the correct configuration, you cannot tie the knot wrong. Is anyone surprised that the jig transfers better than the written procedure? The paper's contribution is demonstrating this in the AI agent context and measuring the separation, but the discussion sometimes reads as if the finding itself is deeply counterintuitive. It is counterintuitive relative to the implicit assumption in the agent memory literature that all knowledge is transferable as text. It is not counterintuitive relative to what we know about the difference between declarative and procedural knowledge, or between informing and constraining. The paper should own this more directly: the finding is surprising only given the specific assumption the AI engineering community has been making.

---

## 4. Suggested Fixes

**Rewrite Section 6.2's dual-process passage.** Replace the claim that the mapping is precise with an acknowledgment that it is analogical. The useful insight is that the distinction between effortful-rule-following and automatic-environmental-constraint has a structural parallel to the distinction between deliberative and automatic processing in human cognition. Frame it as a parallel, not an isomorphism. Something like: "The distinction between process and architectural corrections parallels the dual-process distinction in cognitive psychology (Kahneman, 2011), but the parallel is structural rather than mechanistic. LLM agents do not possess separate processing systems. The parallel holds at the level of system design: reliance on explicit recall is fragile in both human cognition and agent behavior, while environmental constraints that remove the possibility of error are robust in both." Two sentences. That is what the data supports.

**Reframe absorptive capacity more carefully.** Either (a) argue explicitly that the experiential correction history constitutes a form of prior related knowledge in Cohen and Levinthal's sense, which is defensible but requires a paragraph making the case, or (b) introduce a more precise term for what the receiving agent lacks. "Correction history" or "experiential calibration" would be more accurate than absorptive capacity for what you are actually observing. You can still cite Cohen and Levinthal for the general principle while being more precise about your specific instantiation.

**Add a "why this is not obvious" paragraph.** Directly address the objection that environmental constraints outperforming text instructions is unsurprising. The answer is in your data: the agent memory ecosystem (Mem0, Zep, Letta) and the practitioner correction systems (MindMeld, Compound Engineering) are built on the assumption that captured text is transferable behavioral infrastructure. Your finding is surprising relative to that specific and widely held assumption. Make this the explicit foil. The finding is not that jigs work better than written procedures. The finding is that an entire product category is being built on the assumption that written procedures are sufficient, and your data says they are not.

**Downgrade "compilation threshold" to "compilation pattern" until you have more data.** Use the current term when describing the system's implementation (which uses 3 as a threshold parameter) but use "compilation pattern" when making the empirical claim. The distinction between what you built and what you observed matters.

**Tighten the HyperAgents convergence claim.** The paper correctly notes that the two systems are too different for direct confirmation. But it then spends considerable space drawing the parallel. The convergence is suggestive and worth noting, but at this length it reads as if HyperAgents is providing more validation than it actually does. Two independent programs finding "a boundary between text-level and mechanism-level improvements" is interesting. But the boundaries are not identical. Your architectural corrections are environment-external. Their meta-level improvements are agent-internal compiled mechanisms. A paragraph noting the convergence and the differences is sufficient. The current treatment is about twice as long as the evidence warrants.

---

## 5. One Thing the Author Probably Has Not Considered

The paper frames the two correction types as if they live on a single axis from fragile to durable, with compilation as the mechanism that converts one into the other. But there is a third category your data hints at that you have not named: corrections that change the human's behavior rather than the agent's.

Consider the recursive meta-evidence. 41% of Study 2 corrections are about the correction system itself. The researcher is correcting agents on how to do corrections. But corrections on a system the researcher is building are, at some level, corrections of the researcher's own design. The researcher specified the correction system, the agents built it wrong (or differently than intended), and the researcher corrected them. At what point is the correction actually about the researcher updating his own specification rather than the agent updating its behavior?

Your model has two participants in the dyad: human corrector and agent corrected. But correction is bilateral even in human-AI systems. The human is also being calibrated by the correction process. The eight-entry mock patching chain is not only evidence that the agent fails to learn. It is evidence that the human failed to recognize, for seven iterations, that the intervention surface was wrong. The compilation event (recognizing that the fix must be architectural) is a correction of the human's theory of intervention, not a correction of the agent's behavior.

This matters because it connects to a deeper point about the experiencing self and the remembering self. Your paper treats the human as a fixed corrector and the agent as a variable correctee. But your own data shows the human's correction strategy evolving over the study period. The compilation threshold is not a property of the agent. It is a property of the human recognizing that their approach to correction needs to change. This is the human's double-loop learning moment, not the agent's. The agent never learns to compile. The human learns to stop trying to fix the agent and starts fixing the environment instead.

If this is right, "stickiness without resistance" has a more radical interpretation than the one you present. The stickiness is not in the agent. The stickiness is in the human's theory of how agents learn. The human keeps trying to transfer knowledge as text because the human's System 1 treats the agent as a knowledge recipient (analogous to a junior developer who can be instructed). The compilation event is the moment the human's System 2 overrides this default model and recognizes that the agent is better understood as an environment to be engineered than a mind to be instructed.

That reframing would be the strongest possible version of this paper, and it is sitting in your data.

---

*Review prepared as pre-submission feedback. The paper is publishable with the revisions noted above. The core empirical contribution (two types, sharp separation, transfer failure with friction removed) is solid. The theoretical framing needs tightening, not replacement.*
