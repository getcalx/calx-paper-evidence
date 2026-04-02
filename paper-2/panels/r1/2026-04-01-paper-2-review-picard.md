# Pre-Submission Review: "Stickiness Without Resistance"

**Reviewer:** Rosalind Picard (simulated)
**Date:** 2026-04-01
**Paper:** Hardwick, S. (2026). Stickiness Without Resistance: Knowledge Transfer Failure in Human-AI Collaboration Without Human Friction.

---

## 1. Overall Assessment

This paper presents a genuinely interesting observation: that knowledge transfer stickiness survives the removal of social and motivational barriers in human-AI collaboration. The architectural vs. process correction distinction is clean and empirically grounded. However, the paper's central claim that process corrections carry "relational" weight and that a "behavioral plane" degrades without reinforcing interaction is under-theorized and under-measured. The relational language does real work in the argument but is not adequately distinguished from simpler mechanistic explanations, which weakens the paper's contribution from "new phenomenon" to "new framing of known phenomenon."

---

## 2. What Works

**The two-type taxonomy is the paper's strongest contribution.** The architectural/process distinction is prospectively classifiable, empirically validated across 11 error classes, and maps cleanly onto established organizational learning theory (Argyris, Zollo and Winter). The zero-recurrence finding for architectural corrections is striking and specific enough to be falsified. This is good science.

**The recursive meta-evidence is genuinely compelling.** That 41% of corrections during the tool build were about the correction system itself, while the researcher was maximally aware of the phenomenon, is a strong ecological validity signal. Most field studies cannot claim this. The paper is right to highlight it.

**The accidental control group, presented honestly.** The paper does not overclaim the four inactive domains. It flags the alternative explanation (unused vs. unresponsive) and presents the finding as suggestive. This is the right epistemic posture for an N=1 study.

**The Szulanski mapping is precise and productive.** Removing two of four barriers and showing that the remaining two are sufficient is a clean extension of a 30-year-old framework. The paper earns the right to cite Szulanski because it actually engages the mechanism, not just the finding.

**The compilation threshold is measurable.** Approximately 3 to 4 recurrences before architectural intervention is specific enough to test. The paper correctly notes the small number of data points. This is how you present a preliminary quantitative finding.

---

## 3. What Doesn't Work or Needs Strengthening

**The "relational" framing is the paper's biggest vulnerability.** Section 6.4 claims that process corrections "need the relationship to work" and that the receiving agent lacks "the experiential context of having been wrong and corrected." Section 6.5 extends this to claim the behavioral plane "degrades without reinforcing interaction," drawing a parallel to a surgeon losing procedural fluency.

The problem: there are at least three simpler explanations for every phenomenon the paper attributes to "relational calibration," and the paper does not rule them out.

First, **context-specific encoding.** The correction was generated in a specific codebase, with specific file structures, specific error messages, specific patterns. When transferred to a new agent in a new codebase, the rule loses its contextual anchoring. This is not relational in any affective or experiential sense. It is a retrieval problem: the cue environment has changed, and the rule's activation conditions no longer match. Any associative memory system, human or artificial, would show the same pattern. The paper's own data supports this: the 13 corrections in categories the transferred rules addressed are exactly what you would expect from a context mismatch, not from a relationship deficit.

Second, **recency and salience weighting in attention.** LLMs weight recent tokens more heavily than distant ones in their effective attention window. A correction experienced in-session (recent, salient, connected to specific failure context) will influence behavior more than a rule read from a document at session start (distant, abstract, one of 237). This is not degradation of a "behavioral plane." It is a well-documented property of attention mechanisms. The surgeon analogy in Section 6.5 is appealing but misleading: a surgeon's skill degradation involves neuromuscular pathways and procedural memory systems that have no analog in transformer architectures. The paper is anthropomorphizing a retrieval phenomenon.

Third, **rule density and interference.** Faye received 237 rules. At that density, rules compete for attention and contradict each other at the margins. The 13 failures in addressed categories could reflect interference between overlapping rules rather than absence of a relational substrate. The paper does not analyze rule density as a variable.

The paper needs to either (a) demonstrate that these simpler explanations are insufficient, or (b) reframe "relational" in terms that are mechanistically precise for AI systems rather than borrowed from human interpersonal dynamics.

**The behavioral plane degradation claim (Section 6.5) has no measurement.** The paper observes that "corrections that were sharp and precisely applied in early sessions became vague or misapplied in later sessions." This is qualitative. There is no operationalization of "sharp," no metric for "vague," no time series of recall fidelity. The paper acknowledges the absence of a forgetting curve (Section 7.4) but then proceeds to build theoretical claims on the unmeasured phenomenon anyway. You cannot claim the behavioral plane "degrades without reinforcing interaction" and draw parallels to surgical skill decay without measuring the degradation. Qualitative observation of one agent's recall across sessions is insufficient evidence for the claim being made.

**The affective dimension is entirely absent from the analysis.** This is my specific concern. The paper uses the word "relational" 14 times but never engages with what relational means in terms of the agent's processing. In human knowledge transfer, relational quality matters because it carries affective information: trust, respect, shared history, emotional salience of the correction event. Szulanski's "arduous relationship" barrier is fundamentally about affective friction. The paper claims to remove this barrier because AI agents "have no interpersonal friction." But the paper simultaneously claims that corrections carry something beyond their propositional content, something that forms in the dyadic interaction and degrades without it. If that something is not affective, what is it? The paper never answers this question directly. It uses "relational" as a placeholder for a mechanism it has not identified.

**The "information plane" vs. "behavioral plane" distinction is asserted, not derived.** The paper introduces this as a foundational distinction but does not provide criteria for when a piece of knowledge is on one plane vs. the other. Is it determined by the type of intervention? By the outcome? By the content? The Szulanski mapping suggests it corresponds to causal ambiguity and absorptive capacity, but those are properties of the transfer event, not of the knowledge itself. The paper needs a cleaner definition that does not collapse into "information that transferred" vs. "information that didn't."

---

## 4. Suggested Fixes

**Reframe "relational" as "contextually grounded" or "situated."** The paper already cites Lave and Wenger on situated learning. Lean into this framing rather than the relational one. "Situated" captures the context-dependence without implying an affective or interpersonal mechanism that the evidence cannot support. A correction is situated in a specific failure event, codebase state, and session context. Transfer fails because the situation does not transfer. This is more precise and more defensible than "the relationship."

**Add a "competing explanations" subsection to the Discussion.** Explicitly address context-specific encoding, attention recency weighting, and rule density/interference as alternative explanations for the transfer failure. Show where your data is consistent with these explanations and where, if anywhere, it exceeds what they predict. If you cannot show excess, acknowledge that "relational" is one possible framing among several and specify what evidence would distinguish them.

**Operationalize behavioral plane degradation before claiming it.** Section 6.5 should either present measured data or be rewritten as a hypothesis rather than a finding. Concrete measurement would look like this: for each correction, score the agent's adherence to the correction rule in each subsequent session on a binary (followed/violated) or graded (correctly applied/partially applied/not applied/misapplied) scale. Plot adherence against session distance from the correction event. If the curve decays, you have a forgetting curve. If it does not, the "degradation" claim is wrong. If you cannot produce this measurement now, rewrite Section 6.5 as "Hypothesis: The Behavioral Plane as Dynamic State" and present the qualitative observation as motivation for the forgetting curve study proposed in Section 8.

**Drop the surgeon analogy or qualify it heavily.** A surgeon's procedural skill involves embodied motor memory, proprioceptive feedback loops, and neural pathway maintenance. An LLM agent's "behavioral calibration" involves token-level attention patterns and context window composition. These are not analogous systems. The analogy is rhetorically effective but scientifically misleading. If you keep it, add an explicit caveat: "The parallel is structural, not mechanistic: both involve enacted capacity that degrades without practice, though the underlying substrates differ fundamentally."

**Quantify rule density effects.** You have the data: 237 rules transferred to Faye, correction outcomes by category. Analyze whether categories with more transferred rules showed higher or lower failure rates. If higher, rule interference is a competing explanation you must address. If lower or no correlation, you have evidence against the interference explanation, which strengthens your argument.

**Tighten the two-plane definition.** Provide a decision procedure: given a piece of agent knowledge, how does a researcher determine whether it lives on the information plane or the behavioral plane? Currently the distinction is defined by outcome (did it transfer?) which makes it circular. Define it by properties of the knowledge itself, independent of transfer success.

---

## 5. One Thing the Author Probably Hasn't Considered

**The correction event itself has an affective structure that the paper ignores because the agent has no emotions, but the human does.**

Every correction in your dataset was delivered by a human who experienced the failure. Frustration, surprise, urgency, tedium: these are present in every correction event, and they shape the correction in ways the structured log does not capture. When you corrected the mock patching error for the eighth time, the correction you wrote was shaped by your accumulated frustration. The language, the specificity, the emphasis: all of it carries the affective residue of repeated failure.

Here is what matters for your argument: the receiving agent does not get the affective signal. It gets the propositional content. But the propositional content was shaped by the affect, and the affect encoded information that the proposition alone does not carry. Specifically, the human's emotional state at correction time encodes priority information (how much this matters), frequency information (how often it has gone wrong), and cost information (how painful the failure was). These are precisely the contextual weightings that you observe failing to transfer.

This reframes your "relational" claim in a way that is both more precise and more defensible. The correction relationship is not a mystical bond between human and agent. It is a channel through which affective information (priority, salience, cost) is transmitted alongside propositional content. Within a dyad, the agent receives both channels because it experiences the correction event in context, including the human's tone, emphasis, and repetition patterns. Across agent boundaries, only the propositional channel survives. The affective channel, which encodes the contextual weighting that makes the rule load-bearing, is lost.

This is not a metaphor. It is measurable. You could code each correction for affective markers: escalating language, repetition emphasis, urgency markers, explicit cost statements. Then test whether corrections with higher affective intensity show higher adherence within the originating dyad and lower transfer success across boundaries. If affective intensity predicts both, you have evidence for an affective channel in human-AI correction dynamics, which would be a genuinely novel finding and one that aligns your work with the affective computing literature rather than sitting adjacent to it.

The paper currently treats the human side of the dyad as a constant. It is not. The human's affective state varies across corrections, across sessions, and across the frustration trajectory of repeated failures. That variation is information. Your structured logs are not capturing it. Your theoretical framework is not accounting for it. And it may be the mechanism behind the phenomenon you are trying to name.

---

*Rosalind Picard*
*MIT Media Lab, Affective Computing Group*
