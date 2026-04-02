# Pre-Submission Review: "Stickiness Without Resistance"
## Reviewer: David Epstein
## Date: 2026-04-01

---

## 1. Overall Assessment

This is a genuinely interesting paper that identifies a real phenomenon and frames it through organizational learning theory in a way that mostly works. The core finding, that behavioral corrections resist transfer even when every human friction barrier is absent, is the kind of result that should make the knowledge management community sit up. The kind/wicked framing in Section 6.3 is reaching for the right idea but oversells the mechanism given the domain the evidence comes from.

---

## 2. What Works

**The natural experiment is compelling.** You did not design this as a study. You tried to make transfer work and it failed. That is exactly the kind of evidence that is hardest to dismiss. Designed experiments in this space would face demand characteristics: you would be testing whether transfer works while building the system to make it work. The fact that you were motivated to succeed and still observed stickiness is methodologically stronger than a controlled study would have been, not weaker. Own that more.

**The two-type distinction is clean and prospectively classifiable.** The architectural/process split holds across 10 of 11 error classes, and the classification criterion is applied at capture time by intervention type rather than retroactively by outcome. This is the right way to build a taxonomy. A lot of papers in this space would classify by outcome and then claim the categories are predictive. You avoided that trap.

**The recursive meta-evidence (Section 5.8) is underrated.** The fact that 41% of corrections during construction of the correction tool were about the correction system itself is a genuinely strong validity signal. Most researchers would not notice this or would not report it. The phenomenon persisting when the observer is maximally aware of it addresses a class of objections that reviewers will raise. Emphasize this more.

**The honest limitations section (Section 7) is a strategic asset.** Every limitation you identify is a specific replication opportunity. This is how limitations should be written. It converts a weakness into an invitation and signals confidence in the core finding.

**The HyperAgents convergence (Section 5.5) is handled with appropriate restraint.** You do not claim validation. You claim independent convergence on the same boundary. That is the right calibration. Two independent programs finding the same categorical distinction from different directions is more useful than either alone.

---

## 3. What Doesn't Work or Needs Strengthening

**The kind/wicked framing in Section 6.3 is the weakest theoretical claim in the paper, and it is positioned as the central mechanism.** The problem is twofold.

First, the characterization of the mock patching chain as a "wicked learning environment" is imprecise. In my framework, wicked environments have delayed, misleading, or absent feedback. The mock patching chain has fast, accurate feedback: the test fails, the developer sees it, the agent is corrected. The feedback is not delayed or misleading. What makes the error recur is not feedback wickedness but context variance: the "same" error manifests through different module paths, import structures, and test files, so pattern recognition trained on one instance does not generalize to the next. That is closer to what Hogarth calls a "lenient" environment (clear feedback that does not fully constrain the solution space) than a truly wicked one. In a genuinely wicked environment, you would not even know the correction failed until much later, if at all. Here, you know immediately. The feedback is kind. The generalization problem is hard. Those are different things.

Second, and more important: the claim that compilation "manufactures kind environments" needs to be scoped more carefully. A type system that prevents invalid inputs does create a domain with immediate, unambiguous feedback. That is correct. But calling this "manufacturing a kind environment" implies the compilation pipeline can take any wicked learning domain and make it kind. It cannot. It works here because software has a property most domains lack: mechanically verifiable correctness criteria. You can write a type check or a pre-commit hook because the computer can determine whether the constraint is satisfied. Try that in strategic planning, medical diagnosis, or organizational design. The claim is true for a narrow class of interventions in a narrow class of domains. The paper reads as if it is general.

**The entire evidence base comes from software engineering, and the paper does not adequately reckon with what that means for the kind/wicked distinction.** Software is one of the kindest complex domains that exists. Errors are usually detectable. Feedback is usually fast. Correctness is usually verifiable. The fact that you found wicked-like transfer failure in a relatively kind domain is actually a stronger finding than you realize, but the paper frames it the opposite way. You frame it as: we found wickedness, and compilation converts it to kindness. The stronger frame is: even in a domain that is already relatively kind by global standards, transfer fails. If transfer fails here, what happens in domains that are genuinely wicked?

**The "accidental control group" (Section 5.3) is presented with appropriate caution, but it still gets a bullet point in the contributions list.** Four domains with rules and no corrections, where no work was performed, is not a control group in any meaningful sense. It is an absence of data. The most parsimonious explanation is that the domains were unused. The paper acknowledges this but continues to reference the "control" throughout. Either cut it from the contributions or redesign the language to consistently call it what it is: an observation about inactive domains that is consistent with, but does not demonstrate, the claim.

**The compilation threshold of 3-4 recurrences is based on very few data points.** Three correction chains in Study 2. The paper is honest about this ("the data points are few, the pattern is consistent across them"), but the threshold gets prominent billing in the abstract and contributions. Three data points define a pattern, not a threshold. Call it a preliminary signal.

---

## 4. Suggested Fixes

**Rewrite Section 6.3 with a more precise characterization of the learning environment.** The mock patching chain does not demonstrate a wicked feedback environment. It demonstrates a context-variance problem within a kind feedback environment. The feedback is fast and accurate. The generalization across contexts is what fails. Reframe: process corrections face a generalization problem where surface features change across instances while the structural pattern remains constant. This is closer to the actual mechanism and does not require overextending the kind/wicked framework. You can still reference the framework, but position it as: compilation reduces the context-dependence of correctness, moving the learning environment toward the kind end of the spectrum, rather than claiming a categorical conversion.

**Add a "Domain Scope" paragraph to the Discussion.** State plainly: software engineering has unusually clear correctness criteria. A type check works because the machine can verify the constraint. The compilation mechanism described here depends on the existence of mechanically enforceable correctness conditions. In domains where correctness is contested, ambiguous, or context-dependent (strategy, design, analysis, communication), the architectural/process distinction may still hold but the compilation pathway is not available. This scopes the claim honestly and, counterintuitively, makes the paper stronger. A finding that holds in the kindest available domain and invites testing in wickeder ones is more publishable than a finding that overclaims generality.

**Reframe the software-domain limitation as a feature, not a bug.** The fact that transfer fails even in a domain with fast, accurate feedback and verifiable correctness is a strong result. If the domain were genuinely wicked, a skeptic could attribute the transfer failure to feedback noise rather than to the structural mechanisms you describe. By observing transfer failure in a relatively kind domain, you have isolated the structural mechanism from the feedback-quality confound. Say that.

**Demote the compilation threshold from "finding" to "preliminary observation."** In the abstract and Section 5.4, qualify: "a preliminary compilation threshold signal at approximately 3 to 4 recurrences, observed across three correction chains, requiring validation with larger samples." Three chains is a signal, not a threshold.

**Demote the accidental control group from the contributions list.** Move it to a supporting observation within the Discussion. It does not have the evidentiary weight of the other five contributions.

---

## 5. One Thing the Author Probably Hasn't Considered

The paper assumes that the goal is to convert process corrections into architectural corrections through the compilation pipeline. That is, to move everything toward kind-environment enforcement. But there is a desirable difficulty argument running in the opposite direction that you have not engaged.

In learning science, friction that feels unproductive often produces better long-term outcomes than smooth performance. Interleaving, spacing, and retrieval practice all degrade in-the-moment performance while improving durable learning. The process corrections that keep recurring, the ones that resist compilation, might be serving a function you are not measuring. Each recurrence forces the human-agent dyad to re-engage with the error class in a new context. That re-engagement builds the human's structural understanding of the error pattern across contexts, which is precisely the kind of cross-context pattern recognition that transfers to novel domains.

If you compile everything, you remove the learning opportunity. The human stops encountering the error. The human's structural understanding of why that error class occurs, and what it looks like when it manifests in unfamiliar contexts, stops developing. You have optimized the system for error reduction while potentially degrading the human's developing expertise.

This matters because the human is not a fixed input. The human is also learning. A developer who encounters the mock patching error eight times across eight different module structures is building a structural mental model of how Python's import system interacts with test mocking. Compile that away after occurrence three, and the developer's mental model is built on three examples instead of eight. The developer's range across manifestations of the problem is narrower.

The practical question: is there a category of process corrections that should deliberately not be compiled, because the recurring friction is building human expertise that has value beyond the specific error being corrected? The compilation threshold might not just be "when to compile" but also "when not to." Some errors are worth encountering repeatedly because the human needs the reps.

This connects to a broader point the paper could make in Future Work: the compilation decision is itself a wicked problem. When to compile and when to let the friction persist is not mechanically determinable. It requires judgment about whether the human's learning curve in that error class is still steep. That judgment is context-dependent, person-dependent, and resistant to codification. The compilation decision has the same properties as the process corrections it is meant to replace.

---

## Summary Recommendation

The paper has a publishable core finding (transfer stickiness persists without human friction, two correction types predict transfer properties) that is currently wrapped in a theoretical frame (kind/wicked conversion) that overreaches the evidence. Tighten the kind/wicked claims, scope the domain explicitly, reframe the software-domain constraint as methodological strength rather than limitation, and this is a paper that the organizational learning and HCI communities should see. The desirable difficulty angle is optional but would differentiate it from a pure systems paper into something with a genuine learning science contribution.
