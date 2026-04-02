# Re-Review: "Stickiness Without Resistance" (Revised Draft)
## Reviewer: Nate Silver (Signal vs. Noise lens)
## Date: 2026-04-01 (Round 2)

---

## 1. What Improved

The revisions addressed three of my five concerns. Two well, one partially.

**Compilation threshold: meaningfully downgraded.** Section 5.4 now leads with "We present this as a preliminary observation from few data points, not an established threshold." The circularity risk is stated directly: "the system was designed with this threshold, so observing transitions near it may reflect a design parameter rather than an emergent property." This is the right confidence level for three data points. The section header changed from (implied) finding to "A Preliminary Compilation Pattern." The contributions list (item 4) matches: "The data points are few but consistent." The conclusion still says "preliminary compilation pattern" with the parenthetical "approximately 3 to 4 recurrences." This is honest.

**Accidental control group: correctly demoted.** Contribution 3 now reads: "a suggestive observation... This is not a designed control condition; we present it as a pattern warranting designed replication." Section 5.3 says "suggestive rather than conclusive" and names the simplest alternative. Section 4.4 says "We cannot rule out the alternative that these domains were simply unused rather than unresponsive to rules." This was my primary concern. Fixed.

**Ecological prevalence section: a real addition.** Section 6.7 is new and cites 70+ practitioner reports, a 0% compliance test, a 60% compliance measurement, and academic mechanism papers on positional attention bias and long-context degradation. This moves the paper from "one person's experience" toward "characterizing a phenomenon the ecosystem already knows exists." The framing is correct: "convergent support, not part of our formal dataset." Smart.

---

## 2. What Did Not Change (and Should Have)

**The phase analysis and error floor remain the weakest section of the paper.** Section 5.6 is essentially unchanged from the draft I reviewed. My original review made two specific requests:

1. Construct the daily/weekly time series from the 95 timestamped corrections that exist in the Study 1 data. The paper now acknowledges in Limitation 7.5 that "daily timestamps exist for 95 of 134 Study 1 corrections, sufficient for a time series plot, but we did not construct one." This is an unusual choice. You have the data. You acknowledge you have the data. You added a limitation about not using the data. But you did not use the data.

2. Explicitly state that three phase-level observations cannot establish a floor. The paper does not do this. The conclusion still says "a correction rate floor (approximately 2.9 corrections per specification in Study 1)" and frames it as a finding that "suggest[s] measurable boundaries that warrant formal testing."

The reinterpretation in Section 5.6 is also unchanged: "The system is pushing into harder territory while maintaining the same correction rate." My original review identified this as unfalsifiable because there is no independent difficulty metric. It remains unfalsifiable. Nothing was added to make it testable.

This is now the primary confidence-exceeds-evidence gap in the paper. Three phase averages (5.6, 3.0, 2.9) from approximately 5, 8, and 15 specifications respectively, with a "plateau" inferred from two data points (3.0 and 2.9), an untestable reinterpretation layered on top, and a conclusion that presents "2.9" as a measurable boundary. The 95 timestamped corrections sitting unused in the dataset make this harder to defend, not easier. A reviewer will ask why the time series was not constructed, and "we added a limitation about it" is not an answer.

**The 50% persistence figure remains underspecified.** The paper says "approximately 50% persistence" in the abstract, introduction, contributions list, Section 5.2, and conclusion. At no point does it define the numerator, denominator, or measurement criterion. My R1 review asked: fifty percent of what, measured how? This was not addressed.

Looking at the data described: five process error classes recurred, six architectural error classes did not. If "50% persistence" means "5 of 11 error classes showed recurrence," that is 45%, not 50%, and the denominator includes architectural corrections that should not be in the same calculation. If it means something else, the paper does not say what. This number appears six times in the paper and is never defined. It will not survive a methods-focused reviewer.

---

## 3. What Still Overreaches

**The conclusion promotes the error floor to equal standing with the compilation threshold.** The final paragraph reads: "A preliminary compilation pattern (approximately 3 to 4 recurrences before a process correction is identified as needing architectural intervention) and a correction rate floor (approximately 2.9 corrections per specification in Study 1) suggest measurable boundaries that warrant formal testing."

The compilation threshold was correctly downgraded throughout the paper. But the error floor was not. The word "preliminary" modifies the compilation pattern but not the correction rate floor. In the same sentence. The compilation threshold has three data points and an acknowledged circularity concern. The error floor has two data points (Phase 2 and Phase 3) and no acknowledged circularity concern. The error floor claim is weaker than the compilation threshold claim, but receives less hedging. This is backwards.

**The abstract still carries forward claims at full strength that the body hedges.** Compare:

- Abstract: "A preliminary compilation pattern emerges at approximately 3 to 4 recurrences" (correctly hedged)
- Abstract: "Four domains with authored rules but no active correction history show zero behavioral change, a suggestive pattern consistent with situated learning theory" (correctly hedged)
- Abstract: "process corrections, which add text-based rules, show approximately 50% persistence" (not hedged, not defined)

The body does a better job with appropriate uncertainty than the abstract does. The abstract should match the body's level of caution.

---

## 4. The Detection Bias Gap

My R1 review raised detection bias as "one thing the author probably hasn't considered": the researcher's threshold for what constitutes a correction shifts over time, producing a non-stationary capture process that could explain the phase analysis, the error floor, and the persistence estimates.

The revised draft does not address this. Limitation 7.1 covers the single researcher. Limitation 7.6 covers single rater classification. Neither covers detection stationarity: the possibility that the same event that would be logged as a correction in Week 1 goes unlogged in Week 3 because the researcher has habituated to it. The ecological prevalence section (6.7) actually sharpens this concern: if the practitioner community reports 0% to 60% compliance rates, the question of how the researcher decided what counted as non-compliance in this study becomes more pointed, not less.

This is not a fatal flaw. It is a missing limitation that a careful reviewer will identify. Adding a 7.8 on detection stationarity (the correction capture process may not be stationary over a 43-day period, and the error floor in particular could partly reflect detection fatigue) would preempt it.

---

## 5. Specific Remaining Issues

**The 11 error classes and the null hypothesis.** My R1 asked for the probability of observing a clean 6/6 vs 5/5 split under random assignment. This was not addressed. The calculation: with 11 items randomly assigned to two bins, the probability of getting a perfect split where all items in one bin share a property (zero recurrence) and all items in the other do not, depends on the base rate. If we take the observed rate (6 of 11 with zero recurrence), the probability of a random partition producing a clean split is computable via the hypergeometric distribution. For 11 items, 6 with zero recurrence, split into bins of 6 and 5, the probability that all 6 zero-recurrence items land in one bin is (C(6,6) * C(5,5)) / C(11,6) = 1/462. That is p = 0.0022. This is a strong result. It should be in the paper. The fact that it was not computed suggests the author may not have realized how strong it is.

**The Mara-to-Thane case carries more weight than its presentation suggests.** This is the cleanest transfer failure in the paper: same human, same product, same organizational memory, ten-day gap, three independent recurrences. It controls for the codebase confound that weakens the Bryce-to-Faye case. The paper presents it as a supporting example. It should be the lead example in Section 5.1. The Bryce-to-Faye transfer is more dramatic (237 rules, 44 corrections) but has more confounds (different codebase, different agent architecture). The Mara-to-Thane case is smaller but cleaner.

**Section 2.2 front-runs the compilation threshold as though it is established.** The Related Work section discusses the compilation pattern at full theoretical weight ("The compilation pattern we observe at approximately 3 to 4 recurrences is consistent with the transition point between loops"), then Section 5.4 downgrades it to preliminary. A reader encounters the claim at full strength before encountering the hedge. Related Work should use the same "preliminary pattern" language that Findings uses.

---

## 6. Is the Confidence-Exceeds-Evidence Gap Closed?

Partially. The paper is meaningfully more honest than the previous draft. The compilation threshold and accidental control group are now presented at appropriate confidence levels. The ecological prevalence section adds real value.

But three gaps remain:

1. **Error floor (the biggest).** Two phase-level observations, no time series despite available data, an unfalsifiable reinterpretation, and a conclusion that presents "2.9" as a measurable boundary. This is the compilation threshold problem from R1 but without the fix that the compilation threshold received.

2. **50% persistence (a definition gap, not an evidence gap).** Appears six times, never defined. This is fixable in ten minutes but has not been fixed.

3. **Detection stationarity (a missing threat).** The paper discusses single-researcher and single-rater limitations but not the possibility that the capture process itself changes over time.

The confidence-exceeds-evidence gap is about 60% closed. The two claims that were overreaching (compilation threshold, accidental control) are now appropriately hedged. The two claims that were underspecified (error floor, 50% persistence) remain underspecified. The missing threat (detection bias) remains missing.

---

## 7. Recommended Fixes (Priority Order)

1. **Build the time series.** You have 95 timestamped corrections. Plot daily or weekly correction rates. If it shows a smooth decline with a visible floor, the phase analysis becomes real evidence. If it shows noise, the floor claim should be cut. Either way, the paper is stronger for having used its own data. This is the single highest-value revision remaining.

2. **Define "approximately 50% persistence."** State the numerator, denominator, and measurement criterion. If the number changes when you define it precisely, use the precise number.

3. **Hedge the error floor to match the compilation threshold.** If you keep the phase analysis as-is (without the time series), the conclusion should read something like: "a preliminary correction rate decline (from approximately 5.6 to approximately 2.9 corrections per specification across three phases in Study 1)" rather than "a correction rate floor." The word "floor" implies a demonstrated asymptote. Two observations do not demonstrate an asymptote.

4. **Label the Section 5.6 reinterpretation as speculation.** One sentence: "Without an independent measure of specification difficulty, this reinterpretation is speculative and is offered as a hypothesis for future investigation."

5. **Add Limitation 7.8 on detection stationarity.** Two to three sentences acknowledging that the correction capture threshold may shift over a 43-day period, and that a declining correction rate could partly reflect detection habituation rather than system improvement.

6. **Compute the hypergeometric p-value for the binary taxonomy.** The result (p approximately 0.002) strengthens the paper's strongest empirical claim. It costs one paragraph.

7. **Match Section 2.2 hedging to Section 5.4.** Use "preliminary pattern" language in Related Work, not just in Findings.

---

## Bottom Line

The revisions show good editorial judgment on which claims to downgrade. The compilation threshold and accidental control group are now appropriately presented. The ecological prevalence section is a genuine improvement. The error floor is now the primary overreach. It received the least revision of any claim despite having the thinnest evidence base. Fix the error floor, define the 50% number, add the detection stationarity limitation, and the confidence-evidence gap is closed. The paper's core contribution (stickiness persists without human friction, and two correction types show structurally different transfer properties) does not depend on any of the claims I am asking you to fix. You can afford to present them honestly.
