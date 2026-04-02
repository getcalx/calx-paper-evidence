# Re-Review: "Stickiness Without Resistance" (Revised Draft)
## Reviewer: David Epstein
## Date: 2026-04-01 (R2)

---

## 1. Overall Assessment

The revision addressed the central structural problem in the paper. Section 6.3 was the weakest load-bearing element in the first draft. It is now one of the strongest. The reframe from "wicked environment converted to kind" to "context-variance within a kind domain" is not just more accurate, it changes the argumentative logic of the paper in a way that makes every other section land harder. I will go through each of my original concerns and assess whether the revision resolved them.

---

## 2. Concern-by-Concern Assessment

### 2.1 Kind/Wicked Overreach: RESOLVED

My primary objection was that the mock patching chain was characterized as demonstrating a wicked learning environment when it demonstrably did not. The feedback was fast and accurate. What failed was generalization across varying surface features, not the feedback signal itself.

The rewrite gets this exactly right. Section 6.3 now opens by positioning software as "a relatively kind learning environment" by my criteria, then identifies the actual mechanism: "process corrections face a context-variance problem within a kind domain." The mock patching chain is reframed as showing that "the 'same' error manifests differently each time, with different module paths, different import structures, different test files" while "the feedback is accurate (the test fails)." This is precise. The paper no longer conflates hard generalization with wicked feedback. Those are fundamentally different problems, and the revision treats them as such.

The language "This is a narrower claim than 'manufacturing kind environments'" is the right signal to a reviewer. It tells me the author heard the objection and is scoping deliberately rather than retreating vaguely.

### 2.2 Software as a Kind Domain / Methodological Reframe: RESOLVED, AND IMPROVED

My second concern was that the evidence base comes entirely from software engineering, one of the kindest complex domains that exists, and that the paper failed to reckon with what that means. I argued that the stronger frame is: if transfer fails even in a kind domain, what happens in genuinely wicked ones?

Section 6.3 now makes this argument explicitly: "Observing transfer failure in a kind domain isolates the structural mechanism from feedback-quality confounds. If stickiness persisted only in domains with noisy or delayed feedback, it could be attributed to environmental wickedness rather than architectural properties of the knowledge itself."

This is the argument I asked for, stated cleanly. The software-domain limitation is now positioned as methodological strength. A skeptic who says "but you only studied software" now has to confront: the phenomenon appeared in a domain biased against it. That is better than claiming generality and having the skeptic hand you a scope objection for free.

The final paragraph of 6.3 closes the loop: "Domains with genuinely wicked feedback (strategy, design, medical diagnosis) may exhibit different dynamics." The domain-scope limitation is stated plainly, in the discussion, where it belongs. And "Cross-domain validation" appears in the Future Work section as the final entry. Good placement: it signals awareness without hedging the core finding.

### 2.3 "Manufacturing Kind Environments" Overclaim: RESOLVED

The original draft claimed the compilation pipeline "manufactures kind environments." I flagged this as implying the pipeline could take any wicked domain and make it kind, which is false. It works in software because software has mechanically verifiable correctness criteria. Try it in strategic planning.

The revision replaces this with: "removing the context-dependence that makes process corrections fragile, within a domain that already provides reliable feedback." That is a precise claim about what compilation actually does. It does not claim to manufacture kindness from wickedness. It claims to reduce context-variance in a domain where the feedback loop is already intact. The qualifying clause "within a domain that already provides reliable feedback" is the scoping I asked for.

The follow-up sentence in 6.3 makes the dependency explicit: "The compilation architecture depends on the existence of mechanically verifiable correctness criteria, which software provides and many other domains do not." This is the sentence the first draft was missing. It draws the boundary around what compilation can and cannot do.

### 2.4 Accidental Control Group Demotion: PARTIALLY RESOLVED

I recommended demoting the accidental control group from the contributions list. The revision keeps it in the contributions (now Contribution 3) but significantly requalifies the language: "A suggestive observation from four domains with authored rules and zero active corrections that show zero behavioral change, consistent with instruction without the correction cycle producing no learning. This is not a designed control condition; we present it as a pattern warranting designed replication."

The language is now honest. The word "suggestive" is doing the right work. In the body (Section 5.3), the paper adds: "We cannot rule out the simplest alternative: that those domains were simply unused."

I still think this belongs in the Discussion rather than the contributions list. It is the weakest of the six contributions by a wide margin, and including it at the same level as the transfer stickiness finding and the two-type distinction creates a false equivalence of evidentiary weight. But the qualification is now strong enough that no reviewer will be misled. Call this a judgment call where I would go further but the current position is defensible.

### 2.5 Compilation Threshold Demotion: RESOLVED

I recommended demoting the compilation threshold from "finding" to "preliminary observation." The revision does this throughout. The abstract now reads "A preliminary compilation pattern emerges at approximately 3 to 4 recurrences." Contribution 4 uses "A preliminary compilation pattern." Section 5.4 opens with: "We present this as a preliminary observation from few data points, not an established threshold."

The revision also adds a self-awareness note I did not ask for but that strengthens the section: "We note a circularity risk: the system was designed with this threshold, so observing transitions near it may reflect a design parameter rather than an emergent property." That is the kind of methodological honesty that builds reviewer trust. Most authors would not volunteer that concern. Including it preempts the objection and signals that the author is thinking about the evidence rather than advocating for a conclusion.

### 2.6 Desirable Difficulty: NOT ADDRESSED

My final point in the first review was the one I flagged as optional but differentiating. I raised the desirable difficulty argument: that the process corrections which resist compilation might be serving an unmeasured function. Each recurrence forces the human-agent dyad to re-engage with the error class in a new context, building structural understanding of the error pattern across manifestations. Compile everything and you remove the learning opportunity. The human's developing expertise narrows.

The revised draft does not engage this argument anywhere. The compilation pipeline is still presented as unambiguously positive: process corrections are fragile, architectural corrections are durable, compilation converts the former into the latter. The paper does not ask whether some process corrections should deliberately not be compiled because the recurring friction builds human expertise that has value beyond the specific error.

This is a missed opportunity rather than a flaw. The paper's core argument does not require the desirable difficulty angle. But without it, the paper presents a one-directional optimization story (compile everything) where there is actually a tension worth naming (compile strategically). The developer who encounters the mock patching error eight times across eight different module structures is building a structural mental model of Python's import system. Compile that away after occurrence three and the mental model is built on three examples instead of eight.

There are two places where this could be introduced without restructuring the paper:

First, in Section 6.2 (The Compilation Architecture), after the paragraph about the distinction between asking an agent to remember and removing the need to remember. A single paragraph acknowledging: "The compilation decision carries a tradeoff we do not measure in this study. Each recurrence of a process correction forces re-engagement with the error class in a novel context, which may build the human's structural understanding of the underlying pattern. A compilation event that eliminates the error also eliminates the learning opportunity. The question of when to compile and when to let the friction persist is itself context-dependent and resistant to codification, properties shared by the process corrections it is meant to replace."

Second, in the Future Work section, after "Automated compilation pipeline." A brief note: "The compilation decision may itself involve a desirable difficulty tradeoff (Bjork and Bjork 2011): recurring process corrections impose friction that degrades immediate performance but may build human expertise that transfers beyond the specific error class. Characterizing which process corrections should be compiled and which should be preserved as learning opportunities is an open question."

Adding both would take approximately 150 words. The return is disproportionate: it transforms the compilation story from a simple optimization into a genuine learning science question, which is what will differentiate this paper from a pure systems contribution.

---

## 3. New Observations on the Revised Draft

### 3.1 Section 6.7 (Ecological Prevalence) is a Strong Addition

This section was not in the version I reviewed the first time, or I missed it. The practitioner discourse evidence (70+ community reports, 0% compliance in one controlled test, the "CLAUDE.md is a suggestion, hooks make it law" practitioner convergence) significantly strengthens the ecological validity claim. The paper is right to keep this out of the formal dataset and present it as convergent support. It answers the "but is this just one person's experience?" objection before it is raised.

### 3.2 Section 6.6 (Interaction Orientation) is Underappreciated

The scope condition that the phenomenon only manifests when the human treats the agent as a persistent collaborator rather than a stateless tool is genuinely important and candidly stated. Most papers in this space would not acknowledge that their findings depend on an interaction style that is currently the minority case. This is the kind of honest scoping that makes a paper age well. As persistent agent memory becomes standard infrastructure, this scope condition relaxes. The paper is positioning a finding for a world that is arriving, not the world that currently dominates.

### 3.3 The Szulanski Mapping is Tighter

The barrier-by-barrier analysis in Section 6.1 now reads cleanly: two barriers removed (motivation, relationship quality), two surviving (causal ambiguity, absorptive capacity), stickiness persists. The extension of absorptive capacity to include "the experiential context of having been wrong and corrected" as the missing prior knowledge is a genuine theoretical contribution. Cohen and Levinthal (1990) defined absorptive capacity as prior related domain knowledge. This paper argues that the relevant prior knowledge for behavioral transfer is not domain expertise but correction experience. That is a non-trivial extension of the concept, and it is stated with appropriate care.

---

## 4. Remaining Recommendations (Priority Order)

1. **Add the desirable difficulty angle.** Two paragraphs, 150 words, disproportionate return. Transforms the compilation story from optimization into a learning science question. This is what separates a systems paper from a contribution to learning theory.

2. **Consider demoting the accidental control group one more notch.** Move it from the numbered contributions list to a supporting observation in the Discussion. The qualification language is honest, but its position in the contributions list still implies evidentiary parity with the other five contributions.

3. **Add a Bjork and Bjork (2011) citation** if you add the desirable difficulty content. "Making things hard on yourself, but in a good way" is the accessible entry point. Bjork (1994) on desirable difficulties is the foundational reference. Either works.

---

## 5. Summary Verdict

The first draft had a publishable core finding wrapped in a theoretical frame that overreached. The revision fixed the frame. Section 6.3 is now the paper's clearest articulation of what the evidence actually shows and what it does not. The kind/wicked concern is fully resolved. The domain-scope limitation is explicitly stated and converted into methodological advantage. The compilation threshold is appropriately qualified. The "manufacturing kind environments" overclaim is replaced with a precise, scoped alternative.

The one substantive gap is the desirable difficulty argument, which remains unaddressed. This is not a fatal omission. The paper stands without it. But including it would elevate the contribution from "we found a pattern and built a pipeline" to "we found a pattern, built a pipeline, and identified a fundamental tension in the compilation decision that connects to decades of learning science." The second version is the paper that gets cited outside the HCI community.

The paper is ready for submission with the desirable difficulty addition. Without it, it is still submittable but will be read as a systems paper rather than a learning science paper. That is a positioning choice, not a quality judgment.
