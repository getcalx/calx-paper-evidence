# Paper Review: "Stickiness Without Resistance"
## Reviewer: Pedro Domingos (simulated perspective)
## Date: 2026-04-01

---

## 1. Overall Assessment

The paper identifies a real phenomenon and frames it well against Szulanski's stickiness literature. The core observation (behavioral knowledge resists transfer even when motivational barriers are removed) is worth publishing. But the paper's theoretical contribution (the two-type distinction) is weaker than the empirical contribution (the transfer failure itself), and the quantitative claims (compilation threshold, 50% persistence) are built on sample sizes too small to support the precision with which they are stated.

---

## 2. What Works

**The natural experiment is genuinely interesting.** Human-AI collaboration does constitute a clean removal of Szulanski's motivational and relational barriers. That is not a trivial observation. The field has spent decades conflating structural and social barriers to transfer because they co-occur in every human-human study. Removing two of four barriers and showing stickiness persists is a legitimate empirical contribution. The framing is correct: you are not proving the mechanism, you are demonstrating that stickiness survives the removal of its supposed causes.

**The recursive property is well-handled.** Most authors would either hide the fact that they were building the tool they were studying or overclaim it as some kind of proof. You state it plainly and use it as a methodological note about robustness to awareness. That is the right calibration.

**The limitations section is honest.** The six limitations are specific and each defines a concrete replication path. This is how limitations should be written. You do not hedge or minimize. The N=1 acknowledgment is direct. Reviewers will notice this and give you credit for it.

**The Szulanski mapping is precise.** The four-barrier decomposition (two removed, two surviving) is clean enough to carry the paper. If the rest of the theoretical apparatus disappeared, this mapping alone would justify publication as an empirical note.

---

## 3. What Doesn't Work or Needs Strengthening

**The two-type distinction is less novel than presented.** Architectural vs. process corrections maps onto: declarative vs. procedural knowledge (Anderson 1983), single-loop vs. double-loop learning (Argyris 1977), System 1 vs. System 2 analogs (Kahneman 2011), kind vs. wicked environments (Hogarth 2001, via Epstein), and prompt-level vs. meta-level (HyperAgents). You cite all of these. The question a reviewer will ask is: what does the architectural/process distinction add beyond a new label for a known boundary? The paper needs a sharper answer to this. Right now the answer seems to be "we observed it in a human-AI setting," but that is an empirical contribution, not a theoretical one. If the distinction is genuinely new, you need to show where prior categories fail and yours succeeds. If it is an application of known categories to a new domain, say that directly and stop positioning it as a theoretical contribution.

**The classification criterion has a circularity problem you have not fully resolved.** You state that classification is prospective (applied at capture time by intervention type). You then validate by recurrence (architectural = zero recurrence, process = variable recurrence). The paper claims these are independent. They are not fully independent. The criterion "does this change the structure of the system?" is itself partly determined by whether you believe the fix will be permanent. A developer who thinks a code change will eliminate a class of errors classifies it as architectural. A developer who thinks a text rule will suffice classifies it as process. The recurrence outcome is partially predicted by the confidence of the intervention at capture time, not just by the type of the intervention. You need to address this more directly. The borderline case you flag (duplicate model definitions with a grep check) is exactly the kind of case that exposes the issue: the grep check is a mechanism, but a fragile one. If it had failed and the error recurred, would you have reclassified it? If yes, the classification is outcome-dependent. If no, the classification is type-dependent but may produce misclassifications. Either way, the boundary is fuzzier than the paper admits.

**The compilation threshold of 3-4 recurrences is an artifact of small sample size, not a finding.** You have exactly three correction chains exhibiting this pattern. One transitioned at 4, two at 3. From three data points, you cannot distinguish between "the threshold is 3-4" and "the threshold is anywhere from 2-6 and you happened to see 3-4." The confidence interval on n=3 swallows the entire plausible range. Worse, the system's code implements a threshold of 3, which means the system was designed to trigger at 3 recurrences. You are then reporting that the system triggers at approximately 3-4 recurrences. This is not a finding. It is a design parameter being observed. The paper acknowledges "the data points are few" but still presents the threshold as a measurable boundary in the conclusion. It should be presented as a design hypothesis awaiting validation, not as an empirical finding.

**The "50% persistence" figure is imprecise and potentially misleading.** You report that process corrections show "approximately 50% persistence." Persistence of what, exactly? Of the rule being followed? Of the error not recurring? Over what time horizon? In how many cases? The paper does not provide the denominator. If you have 5 process-type error classes and roughly half showed recurrence, the sample size is too small to claim 50% as a rate. It could be 20% or 80% with this N. Either provide the exact calculation with confidence bounds or replace with qualitative language ("roughly half of process-type error classes showed recurrence").

**The accidental control group is too weak to merit its own contribution bullet.** You acknowledge this yourself in Section 5.3: you cannot distinguish "rules without correction cycles are inert" from "those domains had no work." Listing this as Contribution 3 of 6 overstates it. It belongs in the discussion as suggestive evidence, not in the contributions list as a finding.

**The HyperAgents convergence is overstated given the differences.** Your architectural corrections are environment-external. HyperAgents' meta-level improvements include agent-internal mechanisms. These are not the same thing. The convergence is on a vague boundary ("text-based vs. mechanism-based improvements"), which is so general that it would encompass any distinction between passive documentation and active enforcement. You would also "converge" with the entire software engineering literature on the superiority of automated testing over code review comments. The convergence claim needs tighter scoping or significant qualification.

**The Discussion section overreaches with theoretical mappings.** Argyris, Zollo and Winter, Thaler, Kahneman, Epstein, Lave and Wenger, Nonaka and Takeuchi, Polanyi, Cohen and Levinthal. Nine theoretical frameworks applied to a dataset of 151 corrections from one person. Each mapping is individually reasonable but collectively they create the impression of a theory-rich paper built on a data-poor foundation. A reviewer trained in any one of these traditions will notice that the mapping is surface-level. Pick three. Go deep. Drop the rest or compress them into a single "additional theoretical connections" paragraph.

---

## 4. Suggested Fixes

1. **Reframe the contribution hierarchy.** Lead with the empirical contribution (stickiness persists without human friction) as the primary contribution. Demote the two-type distinction to a secondary observation that awaits validation. This is more defensible and actually more interesting: the Szulanski extension is clean and novel. The two-type distinction is a rediscovery of known categories in a new setting.

2. **Address the circularity directly.** Add a paragraph in Section 4.2 that names the circularity risk: "A skeptic could argue that the zero-recurrence outcome for architectural corrections is partly predicted by the developer's confidence at classification time, not by a structural property of the correction type." Then explain why you believe it is not fully circular: architectural corrections change the environment in ways that are independently verifiable (a hook either exists or does not, a type constraint either compiles or does not). The environment change is observable by a third party. Process corrections add text that is observable but whose behavioral effect is not independently verifiable. This independent verifiability is the real criterion, not "does it change code." Sharpen the criterion and the circularity problem mostly dissolves.

3. **Downgrade the compilation threshold to a hypothesis.** Replace "A measurable compilation threshold of approximately 3 to 4 recurrences" (Contribution 4) with something like "A preliminary observation that process corrections were identified as needing architectural intervention after 3-4 recurrences in all three observed chains, suggesting a compilation threshold that warrants systematic measurement." In the conclusion, present it as a testable hypothesis, not a finding.

4. **Provide exact numbers for the persistence claim.** State the denominator. "Of N process-type error classes observed, M showed recurrence within the study period." Let the reader compute the rate. If the N is small (it is), the honest presentation is the counts, not the percentage.

5. **Move the accidental control group out of the contributions list.** It can remain in the findings as suggestive. It should not be a numbered contribution.

6. **Trim the theoretical frameworks in the Discussion to three.** I would keep Argyris (single/double-loop, it is the tightest mapping), Szulanski (it is the paper's anchor), and either Epstein's kind/wicked distinction (the "manufacturing kind environments" section is the most original part of the Discussion) or Kahneman (System 1/2 mapping is intuitive for a broad audience). Drop the rest or consolidate into one paragraph. The paper will read as more rigorous with fewer frameworks applied more deeply.

7. **Tighten the HyperAgents section.** State the convergence claim as: "Both programs independently found that text-based behavioral guidance transfers less reliably than mechanism-based enforcement." Acknowledge that this is a broad finding consistent with multiple literatures, not a specific confirmation of your two-type taxonomy. The timeline independence is worth noting, but the "convergence" framing should be modest.

---

## 5. One Thing the Author Probably Hasn't Considered

**The two-type distinction may be an artifact of the representation, not of the phenomenon.**

Here is what I mean. You classify corrections as architectural or process based on the intervention surface: does it change code or does it change text? But from a machine learning perspective, the real question is whether the knowledge is represented in a form that the learning system can act on reliably. Your architectural corrections succeed not because they are "structural" in some deep sense, but because they encode the knowledge in a representation that is directly executable by the environment (code, hooks, type constraints). Your process corrections fail not because they are "textual" in some deep sense, but because they encode the knowledge in a representation that requires a lossy inference step (the agent must parse the rule, identify the relevant context, and generate compliant behavior).

This is the representation problem from my "Useful Things" paper applied to your setting. The algorithm (the LLM) is the same in both cases. What changes is the representation. Architectural corrections give the system a representation where the relevant pattern is directly visible and enforceable. Process corrections give the system a representation where the relevant pattern must be inferred from natural language under variable context.

The implication is that your two types are not a binary. They are endpoints on a representation quality spectrum. A process correction written in extremely precise, context-tagged, machine-parseable format might approach architectural durability. An architectural correction implemented with a fragile check (your grep-based example) might approach process fragility. The real variable is not "type of intervention" but "how much inference is required between the knowledge representation and the behavioral output."

This reframing would actually strengthen your paper. Instead of a binary taxonomy (which reviewers will attack as reductive), you would have a continuous dimension (representation directness) with your two types as the observable endpoints in your data. The compilation threshold becomes the point at which the representation is upgraded from high-inference to low-inference. And the practical implication shifts from "convert process corrections to architectural ones" to "reduce the inferential distance between the knowledge representation and the behavioral output," which is a more general and more useful design principle.

If you take this seriously, it also connects to the agent memory literature more productively. Mem0 and Zep are optimizing representation for retrieval (can the system find the knowledge?). Your finding is that retrieval is necessary but not sufficient: the representation must also minimize inferential distance to behavior (can the system act on the knowledge without lossy interpretation?). That is a contribution the agent memory community would actually engage with.

---

## Summary Verdict

Publish-worthy as an empirical contribution with the Szulanski extension as the lead. The two-type distinction and the compilation threshold need significant qualification or reframing. The paper currently reads as though it has three major findings when it has one major finding (stickiness survives friction removal), one well-observed pattern (two types with different transfer properties, awaiting validation), and one preliminary signal (compilation threshold from three data points). Calibrate the claims to the evidence and this is a solid paper. Overclaim and reviewers will use the small N to dismiss the whole thing.
