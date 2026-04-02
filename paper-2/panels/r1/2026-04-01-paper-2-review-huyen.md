# Pre-Submission Review: "Stickiness Without Resistance"
## Reviewer: Chip Huyen
## Date: 2026-04-01

---

## 1. Overall Assessment

The paper identifies a real phenomenon that practitioners encounter constantly: behavioral instructions given to AI agents degrade and fail to transfer, while structural constraints hold. The framing through Szulanski's stickiness model is smart and gives a familiar problem theoretical weight. But the system description lacks the reproducibility standard required for the claims being made, and the "behavioral plane" abstraction obscures a simpler engineering explanation that the paper needs to confront directly rather than avoid.

---

## 2. What Works

**The two-type distinction is the paper's best contribution.** Architectural corrections (zero recurrence) versus process corrections (~50% persistence) is an empirically grounded, actionable taxonomy. A practitioner reading this paper walks away with a concrete decision heuristic: if you have written the same rule three times, stop writing rules and change the system. That is useful. It is also independently supported by the HyperAgents convergence, which strengthens the claim that this is structural rather than accidental.

**The compilation threshold is specific enough to be falsifiable.** Claiming 3-4 recurrences as a transition point is the kind of concrete, testable claim that too many papers in this space avoid. Even if the number shifts with replication, having a number at all is valuable.

**The recursive meta-evidence is genuinely interesting.** Building the correction-tracking tool while tracking corrections on the tool, and finding that 41% of corrections were about the correction system itself, is a strong ecological validity argument. Most researchers would have buried this. Foregrounding it was the right call.

**The limitations section is unusually honest.** Six specific, clearly stated limitations, each framed as a replication opportunity. This is how limitations should be written. The paper does not oversell.

**The accidental control group, while weak, is correctly caveated.** The paper does not overweight this. It says "suggestive rather than conclusive" and moves on. Good discipline.

---

## 3. What Doesn't Work or Needs Strengthening

**The system description is not reproducible.** Section 3 describes shell hooks, orientation gates, pre-edit hooks, and coordination gates, but does not specify them precisely enough for another researcher to reconstruct the system. What exactly does the orientation hook inject? What does the pre-edit hook check? How does the coordination gate determine whether a "recent read marker" qualifies? A practitioner building ML systems reads this and asks: could I set this up? The answer right now is "maybe, if I read the GitHub repo and reverse-engineer it." For an academic paper claiming empirical findings, the system description needs to be self-contained enough that the findings are interpretable without reading the codebase.

**The correction capture methodology has a critical subjectivity problem that Section 7.1 understates.** The paper acknowledges single-researcher bias, but the deeper issue is that the definition of "correction" is never operationalized in a way that another researcher could apply independently. You say an intervention qualifies as a correction "when the developer changed an agent's behavior." But every prompt is an attempt to change an agent's behavior. Where is the line between a correction and a normal instruction? Between a correction and a clarification? The six-field schema describes what gets recorded, not what triggers recording. Without an operationalized trigger criterion, two researchers watching the same session could produce different correction counts. This is not a minor methodological footnote. It is the measurement instrument for your primary dependent variable.

**The "information plane" versus "behavioral plane" distinction needs to earn its keep against a simpler explanation.** Here is the simpler explanation: prompt-based instructions are brittle because LLMs do not reliably follow complex procedural instructions, especially when those instructions compete with other context for attention. This is a well-documented property of the technology. The rules failed to transfer not because they lacked some mystical "correction relationship" but because text-based rules in a context window are a low-reliability intervention mechanism. The paper needs to engage this alternative directly. If the simpler explanation (prompt engineering just does not generalize) accounts for the same data, the "behavioral plane" concept is unnecessary overhead. I am not saying the simpler explanation is complete. I am saying the paper currently does not demonstrate that its explanation adds anything the simpler one does not.

**Section 2.5's critique of agent memory architectures is underdeveloped.** The paper makes a strong claim: "An agent that retrieves a correction artifact is not an agent that has been corrected." This is a meaningful statement that challenges Mem0, Zep, and Letta's shared memory model. But the supporting evidence is thin. You have one transfer event (Bryce-to-Faye) and one within-study replication (Mara-to-Thane). The critique of an entire product category requires either more transfer events or a more careful scoping of the claim. As written, it reads like a product positioning statement disguised as a literature review finding.

**The Szulanski mapping is not as clean as the paper presents.** Szulanski's "causal ambiguity" refers to the source being unable to articulate why a practice works. In your system, the source (the developer) presumably can articulate why the rule matters. The ambiguity is not in the source's knowledge but in the artifact's compression of that knowledge. This is a different mechanism. The paper should acknowledge that it is extending the concept rather than applying it directly. Similarly, "absorptive capacity" in Cohen and Levinthal (1990) refers to prior related knowledge enabling assimilation. Your agents have extensive prior knowledge (they are foundation models trained on vast corpora). What they lack is experiential context from a specific dyad. Calling this "absorptive capacity" stretches the concept. It might be more honest to say you have identified a new barrier that is adjacent to absorptive capacity but distinct from it.

**The token and cost reporting, while transparent, is not connected to any efficiency claim.** You report $450 and 4.12 billion tokens. Why? If you are making a cost-efficiency argument, connect it to something. If not, it is noise. In production ML systems, we report infrastructure costs when they are part of the evaluation. Here they seem to be there for credibility signaling.

---

## 4. Suggested Fixes

1. **Add a system specification appendix.** Include the exact hook scripts, the rule injection format, the context window structure, and the correction log schema as a technical appendix. This does not need to be in the main text. It needs to exist. If the artifacts are on GitHub, a pinned commit hash with a mapping table ("Hook X corresponds to file Y at commit Z") would suffice.

2. **Operationalize the correction trigger.** Define a decision rule that another researcher could apply. For example: "An intervention was logged as a correction when the developer issued a directive that (a) referenced a specific failure in the agent's prior output, (b) prescribed a behavioral change for future sessions, and (c) was not a first-time instruction for a new task." Test this by having someone else apply it to a sample of your sessions. Report agreement rate. Even a small inter-rater study on 20 corrections would substantially strengthen the methodology.

3. **Confront the "prompt engineering is just brittle" alternative directly.** Add a subsection in Discussion that states the simpler explanation, evaluates whether it accounts for the data, and identifies what the behavioral plane concept explains that the simpler version does not. The strongest counter-argument you have is the Mara-to-Thane case: the rule was not merely in a prompt, it was in persistent memory, available at session start, and still failed. If prompt brittleness were the full explanation, rules that are reliably loaded should work. The fact that they do not is your evidence for something beyond prompt reliability. Make this argument explicitly.

4. **Tighten the Szulanski mapping or rename the mechanisms.** Either acknowledge that you are extending causal ambiguity and absorptive capacity beyond their original definitions (and explain what the extension adds), or introduce your own terms for the specific barriers you observe. "Artifact compression loss" and "experiential capacity" would be more precise and would avoid the appearance of concept-stretching.

5. **Expand Section 2.5 or narrow the claim.** Either provide more transfer events that test the agent memory critique (even thought experiments about what Mem0-style shared memory would and would not capture in your system), or scope the claim to what your data supports: "Our evidence suggests that shared memory is insufficient for this specific class of behavioral knowledge." Do not let it read as a general indictment of agent memory architectures based on two data points.

6. **Cut or justify the cost reporting.** If $450 is meant to demonstrate accessibility for replication, say that in one sentence: "The total compute cost of $450 suggests the methodology is accessible to independent researchers without institutional compute budgets." If it is not serving a specific argument, remove it.

---

## 5. One Thing the Author Probably Has Not Considered

**You have a data flywheel problem, and it is the same one you are describing in the paper.**

Your correction capture process generates the data that trains the compilation pipeline. More corrections produce better compilation threshold estimates, which produce better architectural interventions, which reduce future corrections, which reduces the data available to improve the pipeline. This is a classic exploitation-exploration tradeoff in production ML systems. The system optimizes itself toward fewer corrections (which is the goal), but in doing so it degrades the signal quality available for future learning.

The practical consequence: your compilation threshold of 3-4 recurrences is calibrated on a system that is still generating abundant corrections. As the system matures and corrections become rarer, the threshold signal becomes noisier. You may need more recurrences to confidently identify compilation candidates, precisely because the system is working. This is analogous to the cold-start problem in recommendation systems: the better the system gets, the harder it becomes to detect when it is wrong.

This is worth a paragraph in Discussion or Future Work. It connects your finding to a well-understood production ML challenge and gives practitioners a concrete warning: do not treat the compilation threshold as a fixed constant. It is a function of the system's maturity, and it will shift as the correction rate declines. The error floor you observe at ~2.9 corrections per specification might be partly an artifact of this dynamic: not the true minimum error rate, but the rate at which the correction signal becomes too sparse to drive further compilation.

This also strengthens your paper, because it means the compilation pipeline is not a one-time intervention. It requires the same kind of continual monitoring and recalibration that any production ML system requires. The algorithm (compilation) is the smallest part. The data pipeline (correction capture), the monitoring (recurrence detection), and the feedback loop (threshold recalibration) are where the system succeeds or fails long-term.
