# Paper Review (R2): "Stickiness Without Resistance"
## Reviewer: Pedro Domingos (simulated perspective)
## Date: 2026-04-01
## Status: Second review, responding to revisions

---

## 1. Overall Assessment

The paper is substantially improved. The author took the first review seriously and made structural changes rather than cosmetic ones. The claim calibration is better across the board. The contribution hierarchy is closer to what the evidence supports. Several of the changes introduce new strengths. A few introduce new problems. The paper is now closer to publishable, but two issues remain that would draw fire from a methods-oriented reviewer.

---

## 2. What Improved

**The compilation threshold is now correctly framed.** The revision downgrades it from "a measurable compilation threshold" to "a preliminary compilation pattern" and explicitly names the circularity risk: the system implements a threshold of 3, so observing transitions near 3 may reflect the design parameter. This was the most important fix from R1. The author went further than I asked by stating "we present the pattern as a testable hypothesis for future work, not as an established finding." Good. Section 5.4 now reads as honest preliminary observation rather than overclaimed finding. The contribution list (item 4) matches: "A preliminary compilation pattern" with appropriate hedging. This is the right calibration.

**The accidental control group is properly scoped.** It is no longer framed as a standalone contribution. The contribution list (item 3) now reads "a suggestive observation" and explicitly states "this is not a designed control condition; we present it as a pattern warranting designed replication." Section 5.3 adds the key sentence: "We cannot rule out the simplest alternative: that those domains were simply unused." This was the second most important fix from R1. The finding is retained without overclaiming. Exactly right.

**System 1/System 2 mapping is gone.** The Kahneman framing was the weakest theoretical connection and removing it tightens the paper. The author did not try to keep it in a reduced form, which would have been worse. Clean removal.

**The kind/wicked rewrite is a genuine improvement over the original.** Section 6.3 now makes a narrower and more defensible claim: not that the system "manufactures kind environments" but that process corrections face "context-variance in a relatively kind domain." The eight-entry mock patching chain is used well here. The surface features vary while the underlying error class remains the same, which is exactly the problem. The sentence "the feedback is accurate (the test fails), but the surface features vary enough that the rule fails to generalize across instances" is precise. The section also now acknowledges the scope limitation directly: "domains with genuinely wicked feedback may exhibit different dynamics." This is a better section than the original and adds analytical depth rather than just hedging.

**The competing explanations section (6.4) is new and necessary.** Before claiming that process correction failure is relational, the paper now considers three simpler alternatives: prompt brittleness, context-specific encoding, and attention recency. Each is addressed with specificity. The argument that none of the three fully accounts for the architectural/process asymmetry is correct: if the issue were purely prompt-level, architectural knowledge delivered as text should also degrade, and it does not. This section will satisfy the reviewer who would otherwise write "have you considered that LLMs just lose track of long instructions?" The answer is yes, and here is why that is not sufficient.

**The autoethnographic methodology framing (Section 4.1) is appropriate.** Citing Ellis and Bochner (2000) and Adams et al. (2015) places the work in a recognized methodological tradition rather than leaving it as an ad hoc single-practitioner study. The framing is honest: "trades external validity for ecological validity." This will not eliminate reviewer concerns about N=1 but it positions the paper correctly within a tradition that has accepted and refined single-practitioner studies.

**The interaction orientation scope condition (Section 6.6) is a smart addition.** The observation that the stickiness phenomenon may only manifest when the human treats the agent as a persistent collaborator rather than a stateless tool is important for the field. Most people use agents transactionally. The paper is now explicit that its findings describe a specific interaction regime. This preempts the reviewer who would write "most people don't work this way." The author acknowledges that and argues the interaction style is becoming more common. The addition of Section 7.7 as a formal limitation reinforces the point.

**The ecological prevalence section (6.7) strengthens the N=1 defense.** Citing 70+ practitioner reports of rule adherence failure across six tools, including controlled measurements (0% compliance in one test, 60% in another), establishes that the phenomenon is not idiosyncratic to this researcher. The Montes quote ("Your CLAUDE.md is a suggestion. Hooks make it law") is a practitioner independently arriving at the architectural/process distinction. The Liu et al. (2023) and Wang et al. (2026) citations connect the mechanism to known properties of transformer architectures. The author correctly separates this from the formal dataset: "convergent support, not part of our formal dataset."

---

## 3. What Still Needs Work

**The circularity problem is acknowledged but not resolved.** Section 5.4 names the circularity risk, which is progress. But the paper still does not address the deeper issue I raised in R1: the classification criterion itself. Section 4.2 states that the classification is "prospective, applied at the time of capture based on the nature of the intervention, not retroactively from outcome data." The paper then validates the classification by noting that recurrence data matches the classification for 10 of 11 error classes. But the classification of "does this change the structure of the system?" still partly depends on the developer's belief about whether the fix will be permanent.

In R1, I suggested a specific resolution: reframe the criterion as independent verifiability. Architectural corrections change the environment in ways that are independently verifiable by a third party (a hook either exists or does not, a type constraint either compiles or does not). Process corrections add text whose behavioral effect is not independently verifiable. This reframing dissolves most of the circularity because it shifts the criterion from "does this change structure?" (partly subjective) to "is the enforcement mechanism observable independent of the agent's behavior?" (testable). The revision did not take this suggestion. The criterion remains "does the intervention change code, configuration, or infrastructure?" which is cleaner than "is it structural?" but still admits borderline cases that are resolved by the developer's judgment about mechanism strength, not by an independent criterion.

This is not fatal. The 10-of-11 validation is strong empirical support. But a methods reviewer will probe the borderline case (the grep-based check) and ask: if the grep check had failed to catch a duplicate model definition, would you have reclassified the correction? The paper still does not answer this question. Add one paragraph to Section 4.2 that addresses independent verifiability as the underlying criterion. The revision need not replace the current criterion; it should articulate why the current criterion works in practice (because most architectural corrections produce independently verifiable environmental changes) and where it is weakest (when the enforcement mechanism is itself fragile).

**The "approximately 50% persistence" figure remains underspecified.** R1 asked for the exact denominator. The revision still uses "approximately 50% persistence" in the abstract, Section 1 (paragraph 5), Section 5.2 (via the phrase "five error classes recurred"), and the conclusion. The reader can now reconstruct the number: 5 of 11 error classes in Study 1 showed recurrence, which is 45.5%, rounded to "approximately 50%." But this reconstruction requires reading across sections. The paper should state in one place: "Of 11 error classes examined in Study 1, 6 showed zero recurrence after architectural fixes and 5 showed persistent recurrence despite process-type rules, yielding an approximately 45% recurrence rate for the full set." This is a one-sentence fix. Without it, a reviewer will still ask "50% of what?"

**The HyperAgents convergence claim is tighter but still structurally problematic.** Section 5.5 adds key differences (environment-external vs. agent-internal mechanisms, human-agent dyads vs. agent-only optimization). Section 2.7 is more careful: "We do not claim that HyperAgents validates our findings." Good. But the claim is still listed as Contribution 5: "Independent convergence with HyperAgents on the boundary between declarative and procedural behavioral learning." The problem is that "independent convergence" is a strong claim that implies mutual support, and then the body text immediately walks it back. The convergence is real but it is convergence on a very broad boundary (text instructions transfer poorly, mechanisms transfer well). As I noted in R1, this is also consistent with the entire software engineering literature on automated testing vs. code comments. The convergence with HyperAgents is not more meaningful than the convergence with, say, the continuous integration movement of the 2000s. A sentence acknowledging that this broad boundary is well-established in software engineering, and that the contribution of the HyperAgents convergence is its appearance in a controlled agent-only setting, would properly scope the claim.

---

## 4. New Issues Introduced by the Revisions

**The ecological prevalence section (6.7) needs a methodological note about the practitioner evidence.** The section cites "over 70 distinct community reports" and two specific measurements (0% and 60% compliance). The 70+ number sounds rigorous but the underlying evidence is community discourse: forum posts, blog posts, tweets. These are not controlled measurements. The two specific measurements (nedcodes and Montes) are closer to controlled tests but their methodologies are unknown to the reader. The paper should add one sentence clarifying that the 70+ reports are practitioner discourse, not peer-reviewed studies, and that the two compliance measurements are informal tests whose methodologies are detailed in the forthcoming paper [Hardwick2026c]. Without this, a reviewer may treat the 70+ number as though it carries the evidentiary weight of 70 independent replications, which it does not.

**The abstract is now too long.** The revised abstract is a single paragraph of approximately 300 words that tries to summarize all six contributions, the two studies, the recursive property, the ecological evidence, and the HyperAgents convergence. It reads as a compressed version of the introduction rather than a stand-alone summary. An abstract should be 150 to 200 words for a paper of this length. Cut the HyperAgents sentence, the ecological evidence mention, and the accidental control group. Focus the abstract on: (1) the Szulanski extension (stickiness without human friction), (2) the two-type distinction with the persistence asymmetry, (3) the recursive meta-evidence. Those are the three things a reader scanning the abstract needs to decide whether to read the paper.

**The contribution list has grown to six items plus a limitations count.** This is too many contributions for a paper with N=1 and 151 data points. The revision successfully downgraded the compilation threshold and the control group, but they are still numbered contributions. The paper would be stronger with three contributions stated as primary findings and three stated as preliminary observations. Right now they are all formatted identically, which gives the impression of six parallel claims at equal evidentiary strength. A simple restructuring: "This paper makes three primary contributions: [1, 2, 6]. Three additional observations are reported as preliminary patterns warranting replication: [3, 4, 5]." This does not require removing any content, only restructuring how it is presented.

---

## 5. The Representation Spectrum Revisited

In R1, I suggested that the two-type distinction might be endpoints on a representation quality spectrum rather than a binary. The underlying variable would be inferential distance: how much inference is required between the knowledge representation and the behavioral output. Architectural corrections minimize inferential distance (the constraint is directly executable). Process corrections maximize it (the agent must parse, contextualize, and generate compliant behavior from natural language under varying context).

The revision did not engage this suggestion directly, but the new Section 6.3 (context-variance in a kind domain) implicitly moves in this direction. The observation that "the same error manifests differently each time, with different module paths, different import structures, different test files" is a description of high inferential distance: the rule is abstract, the application context varies, and the agent must bridge the gap each time. The compilation pipeline is, from this perspective, a mechanism for reducing inferential distance.

I note that the author chose not to adopt the continuous framing and retained the binary taxonomy. This is a defensible choice for this paper given the data. With 11 error classes, 6 architectural and 5 process, there is not enough data to demonstrate a spectrum. The binary is what the data supports. But I would add one sentence in the discussion acknowledging that the two types may be endpoints on a continuum of representation directness, with the binary classification being sufficient for the current dataset. This inoculates against the reviewer who will argue the taxonomy is reductive without requiring the author to restructure the analysis.

---

## 6. Minor Points

1. Section 3.1 states "approximately 82,000 lines of code (42,000 Python, 40,000 TypeScript)." The precision of "42,000" and "40,000" is inconsistent with "approximately." Either say "approximately 82,000 lines" without the breakdown or provide exact counts.

2. The references list has formatting inconsistencies. The Hu et al. (2025) and Wang et al. (2026) entries lack complete publication information. The Shipper and Klaassen (2025) entry is cited as [CompoundEng2026] in the text but listed under 2025 in the references.

3. Section 2.6 (Multi-Agent Evaluation) feels vestigial. The zero-overlap confusion matrix and specificity degradation are reported in Hardwick (2026a), not in this paper. Either expand this section to connect it to the current findings or cut it to a single sentence in the introduction.

4. The nedcodes (2026) and Montes (2026) citations in Section 6.7 do not appear in the references list. Add them or cite them as informal practitioner reports with URLs.

---

## 7. Summary Verdict

The paper addressed the most important concerns from R1. The compilation threshold, the accidental control group, and the theoretical overreach are all properly scoped now. The new material (competing explanations, ecological prevalence, interaction orientation, autoethnographic framing) adds genuine substance. The kind/wicked rewrite is better than the original.

Two items remain unresolved: the classification criterion needs the independent verifiability paragraph, and the 50% persistence figure needs its denominator stated explicitly. Neither requires restructuring. Both are one-paragraph fixes.

The paper is now making three strong claims supported by its evidence (Szulanski extension, two-type asymmetry, recursive meta-evidence), two preliminary observations properly hedged (compilation pattern, inactive domains), and one convergence claim that needs slightly more qualification (HyperAgents). The contribution list should be restructured to reflect this hierarchy.

If the author makes the classification criterion fix, states the persistence denominator, trims the abstract, and restructures the contribution list into primary and preliminary tiers, this paper is ready for submission. The N=1 limitation is real and reviewers will raise it, but the autoethnographic framing, the ecological prevalence evidence, and the honest limitations section collectively make the case that this is a well-characterized phenomenon from a single researcher that warrants multi-researcher replication. That is a publishable position.

Verdict: Accept with minor revisions. The revisions are specific, bounded, and achievable in a single editing pass.
