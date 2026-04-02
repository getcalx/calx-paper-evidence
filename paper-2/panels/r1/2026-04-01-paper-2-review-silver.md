# Pre-Submission Review: "Stickiness Without Resistance"
## Reviewer: Nate Silver (Signal vs. Noise lens)
## Date: 2026-04-01

---

## 1. Overall Assessment

The paper identifies a genuinely interesting phenomenon and frames it well against established organizational psychology. But the quantitative claims are substantially ahead of the evidence. With 151 total corrections from a single researcher, 3 examples of the compilation threshold, and a phase analysis built on ~5/~8/~15 specifications, the paper is presenting pattern-shaped noise as though it were established signal. The theoretical architecture is stronger than the empirical foundation beneath it.

---

## 2. What Works

**The natural experiment framing is the strongest card in the hand.** Szulanski's four barriers, with two eliminated by construction, is a clean setup. You do not have to torture the data to make this point. The Bryce-to-Faye transfer (237 rules transferred, 44 new corrections, 13 in addressed categories) is a concrete, falsifiable observation. It is the one finding with enough specificity to survive scrutiny.

**The architectural vs. process distinction is intuitively clean and prospectively classifiable.** This is important. Most taxonomies in this space are retroactive (you classify based on what happened next). A taxonomy where the classification criterion is determinable at capture time, before outcome data, is methodologically stronger. The paper correctly identifies this advantage and states it clearly.

**The limitations section is unusually honest for a paper this early.** Listing six specific limitations, each framed as a replication invitation, is the right move. It signals awareness that these are preliminary findings and preempts the strongest objections a reviewer would raise. The N=1 limitation is stated plainly. Good.

**The HyperAgents convergence is well-hedged.** The paper says "we do not claim that HyperAgents validates our findings" and explains why. This is the right confidence level. Independent convergence on a similar boundary is suggestive evidence, not confirmation, and the paper treats it that way.

---

## 3. What Doesn't Work or Needs Strengthening

**The compilation threshold claim is built on three data points.** Three correction chains from Study 2 (the deactivate_rule chain at 4 recurrences, C006 at 3, and clean-exit at 3) constitute the entire evidence base for the "approximately 3 to 4 recurrences" threshold. This is presented as a finding and carried into the conclusion as a measurable boundary. It is not a measurable boundary. It is three observations that are consistent with each other. Any number between 2 and 6 would be "consistent with" three data points at 3, 3, and 4. The confidence interval on this estimate, if you computed one, would be enormous.

The deeper problem: the system's code implements a threshold of 3 recurrences. This means the tool was built with a prior about where the threshold should be. That prior then shaped which corrections got flagged as compilation candidates. The "finding" may be an artifact of the implementation decision. The paper acknowledges this in passing ("The system's code implements a threshold of 3 recurrences. The most documented example transitioned at 4.") but does not grapple with the circularity. The code enforced a threshold. The data shows corrections hitting the threshold. This is not independent evidence for the threshold's location.

**The phase analysis (5.6 to 3.0 to 2.9) does not support the claims made about it.** Three phase-level observations with approximate specification counts. Phase 1 is ~5 specs, acknowledged as "directional rather than precise." The decline from 5.6 to 3.0 could be regression to the mean. Initial specifications likely had higher correction rates because the system was new, the developer was calibrating the correction methodology, and the agents had no accumulated rules. Any learning system shows a declining error rate early on. This is not evidence of a floor. It is evidence of a system warming up.

The "plateau" from 3.0 to 2.9 is based on two data points. A difference of 0.1 corrections per specification across two phases with different specification counts and uncontrolled complexity is not a signal. It is noise indistinguishable from flat. You cannot distinguish a plateau from a continuing slow decline from a random fluctuation with two observations.

**The "error floor" reinterpretation in Section 5.6 is motivated reasoning.** The paper first presents 2.9 as a plateau, then reinterprets it: "the system is pushing into harder territory while maintaining the same correction rate." This reinterpretation makes the finding unfalsifiable. If the rate declines, it is learning. If the rate is flat, it is "maintaining rate against harder problems." If the rate increases, presumably it would be "even harder problems." There is no outcome that would disconfirm this framing. That is the hallmark of a narrative, not an empirical claim.

The paper has no independent measure of specification difficulty. Without one, "Phase 3 specifications addressed harder integration problems" is an assertion, not evidence. A difficulty metric (lines of code per spec, number of cross-system integrations, time to completion, number of files touched) would make this reinterpretation testable. Without it, the reinterpretation is unfalsifiable and should be either cut or explicitly labeled as speculation.

**The 50% persistence figure for process corrections is underspecified.** "Approximately 50% persistence" is stated repeatedly but its derivation is unclear. Fifty percent of what, measured how? Is this the fraction of process correction chains that show at least one recurrence? The fraction of individual correction instances that fail to prevent the next instance? The fraction of error classes that recur? These are different quantities. The paper needs to state the numerator, the denominator, and the measurement criterion.

**The 11 error classes are not enough to establish a clean binary.** Six architectural (zero recurrence) and five process (recurrence). The paper claims "predictably different transfer properties." With 11 classes, a single misclassification changes the story. The borderline case (duplicate model definitions) is acknowledged but treated as an outlier rather than as evidence that the boundary is fuzzy. The base rate question: if you randomly assigned 11 error classes to two bins and measured recurrence, how often would you get a clean split by chance? With small numbers, the answer is: more often than you would think. The paper should report or at least discuss the probability of observing this separation under the null hypothesis.

**The accidental control group proves less than it appears.** Four domains with rules and zero corrections. The paper correctly flags the alternative explanation (those domains were simply unused). But it then treats the finding as "suggestive" and carries it as a contribution in the introduction. A finding where the primary alternative explanation is "nothing happened because nobody was working there" is not suggestive. It is ambiguous. The domains had authored rules but no active work. The absence of behavioral change is the expected outcome regardless of whether rules-without-correction-cycles are inert, because there was no work to generate corrections. This should be demoted from contribution status to a methodological observation.

---

## 4. Suggested Fixes

**On the compilation threshold:** Reframe from "we identify a measurable threshold" to "we observe a consistent pattern across the limited cases available." Explicitly address the circularity with the implemented threshold of 3. Acknowledge that three data points cannot establish a threshold's location with any precision. Present it as a hypothesis to be tested with more data, not as a finding. If you want to keep the specific number, compute and report a confidence interval or credible interval. If the interval runs from 2 to 7, that is informative about how uncertain the estimate is.

**On the phase analysis and error floor:** Either (a) construct the daily/weekly time series from the 95 timestamped corrections and present it, which would be far more informative than three phase averages, or (b) explicitly state that three phase-level observations cannot establish a floor and present the numbers as descriptive statistics rather than as evidence for a plateau. The time series data apparently exists. Use it or explain why you did not. Three data points binned into phases is a choice that discards information.

**On the reinterpretation in 5.6:** Cut it or label it explicitly as an unfalsifiable speculation that requires a difficulty metric to test. Do not present it alongside empirical findings as though it carries equivalent weight. The honest version is: "We observe a flat rate in Phases 2 and 3. We cannot determine whether this represents a floor, continued decline, or changing task complexity, because specification difficulty was not measured."

**On the 50% persistence claim:** Define it precisely. State the numerator and denominator. If possible, report a confidence interval. "Approximately 50%" with no precision or measurement definition will not survive peer review.

**On the accidental control group:** Move it from the contributions list to a methodological note. Frame it as: "We observe that four domains with authored rules but no active work showed no behavioral change. This is consistent with our framework but also consistent with the simpler explanation that no work was performed. A designed experiment is needed to distinguish these explanations." This is what the limitations section already says. The contributions list should match.

**On the binary taxonomy:** Report the probability of observing 6-of-6 zero-recurrence in one category and 5-of-5 recurrence in the other under the null of random assignment. If the p-value is small, that strengthens the claim. If it is not small, acknowledge it. This is a simple calculation and the paper should include it.

---

## 5. One Thing the Author Probably Hasn't Considered

**The correction capture process itself is a biased sample, and the bias may produce the pattern you are reporting.**

You are the sole researcher, and you decide in real time what constitutes a "correction." This means your capture process has a detection threshold that varies with context, fatigue, task complexity, and time. Early in a build, you are more likely to notice and log corrections because the system is new and errors are salient. Late in a build, you have adapted to the system's quirks. Some things that would have been corrections in Week 1 are now absorbed as "how it works" and not logged.

This is not the single-rater limitation you already identify. That is about classification accuracy. This is about detection bias in the sample itself. If your correction capture rate varies systematically over time, the phase analysis, the error floor, the persistence estimates, and the compilation threshold are all conditioned on a non-stationary detection process.

The specific risk: the "error floor" at 2.9 may partly reflect detection fatigue. Not that errors stopped, but that your threshold for what counts as a correction shifted upward as you habituated to the system. Regression to the mean in your own attention, not in the system's error rate.

This is testable. If the correction logs from Study 2 (where you were building the correction tool and therefore maximally attentive to what constitutes a correction) show a higher capture rate per unit of work than late-phase Study 1, that is evidence for detection bias. If they do not, it weakens the concern. Either way, the paper should discuss detection stationarity as a threat to validity. Right now, the paper assumes that the correction capture process is stationary. It almost certainly is not.

---

## Bottom Line

The theoretical framing is ahead of the empirical evidence. The architectural vs. process distinction is a real contribution. The compilation threshold and error floor are hypotheses, not findings, at the current sample size. The accidental control group is too confounded to carry weight. The phase analysis needs either a proper time series or explicit acknowledgment that three observations cannot establish a curve shape.

The paper would be stronger if it presented fewer claims with appropriate uncertainty than more claims with false precision. The interesting result (stickiness persists without human friction, and two correction types show different persistence) is strong enough to carry a paper. The quantitative claims (3-4 recurrences, 2.9 floor, 50% persistence) need either more data or wider error bars. Right now, the confidence exceeds the evidence. Close that gap before submission.
