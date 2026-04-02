# Second-Round Review: "Stickiness Without Resistance"

**Reviewer:** Rosalind Picard (simulated)
**Date:** 2026-04-01
**Round:** R2 (revision review)
**Paper:** Hardwick, S. (2026). Stickiness Without Resistance: Knowledge Transfer Failure in Human-AI Collaboration Without Human Friction.

---

## 1. Overall Assessment

The revision is substantially improved. The author addressed the most damaging structural problems from my first review: the surgeon analogy is gone, the competing explanations appear before (not after) the situated knowledge claim, and the degradation claim is explicitly flagged as qualitative observation rather than finding. The paper is now making a more honest version of itself. Several concerns remain, some carried over from the first review and some introduced by the revision. None are fatal. The paper is approaching publishable form for a workshop or short-paper venue.

---

## 2. Concern-by-Concern Assessment

### 2.1 Competing Explanations Section (NEW: Section 6.4)

**Status: Addressed. Mostly adequate. One structural weakness remains.**

The new Section 6.4 does what I asked: it presents prompt brittleness, context-specific encoding, and attention recency as alternative explanations before advancing the situated knowledge claim. The order is correct now. The reader encounters the simpler explanations first.

The author's handling of each alternative is uneven. Prompt brittleness is well-dispatched: the Mara-to-Thane case is a genuine counterexample because it involves complete re-derivation, not gradual degradation. That is a clean empirical argument. Context-specific encoding is acknowledged as "likely part of the mechanism" and linked to causal ambiguity. This is honest but undersells the point. If context-specific encoding is part of the mechanism, the paper should say how much of the variance it explains and what remains. Currently it reads as a concession followed by a pivot. Attention recency is dispatched too quickly. The author argues it "predicts that all transferred rules would degrade equally, which is not what we observe." This is a reasonable point about the 13 selective failures, but attention recency models are more nuanced than uniform decay. Positional bias interacts with semantic relevance, rule length, and placement in the context window. The paper cites Liu et al. (2023) in the ecological section but does not engage their actual mechanism here.

The paragraph "None of these alternatives fully accounts for the architectural/process asymmetry" is the strongest move. It is correct: the three alternatives predict that architectural knowledge should also be fragile, and it is not. This is the real argument for why simpler explanations are insufficient. I would move this paragraph earlier in the section and make it the organizing principle. Currently it reads as a cleanup sentence at the end. It should be the frame.

**Suggested fix:** Restructure 6.4 so the architectural/process asymmetry argument leads. Open with: "Three simpler explanations account for some of the process correction transfer failure. None accounts for the asymmetry between types." Then walk through each alternative, showing where it predicts correctly and where the asymmetry breaks it. The reader should encounter the strongest argument first, not last.

### 2.2 "Relational" Language Shifted Toward "Situated" and "Context-Dependent"

**Status: Partially addressed. Improvement is real but incomplete.**

The most problematic uses of "relational" from the first draft are reduced. Section 6.4 now uses "situated knowledge" and "context-dependent" in the key analytical passages. The Lave and Wenger framing does more work than it did before.

However, "relational" still appears in load-bearing positions. Section 1 still says process corrections "require the receiving agent to have the correction relationship (the experiential context of having been wrong and corrected) that gives the rule behavioral force." Section 2.3 still says "the relational calibration between a specific human and a specific agent." Section 2.5 still says "behavioral change that is relational (formed within a specific correction dynamic)." Section 5.1 says "the correction relationship." Section 6.1 says "the experiential context of having been wrong in the relevant way."

The issue is not that "relational" appears. It is that the paper uses it in two distinct senses without distinguishing them. Sometimes "relational" means "formed within a specific dyadic interaction" (a descriptive claim about where the knowledge lives). Sometimes it means "requiring the relationship to function" (a causal claim about what makes the knowledge work). The first is defensible and empirically grounded. The second requires evidence the paper does not have: you would need to show that re-creating the correction event in a new dyad restores the behavioral force, which would demonstrate that it is the relationship doing the work rather than the context.

**Suggested fix:** Pick one sense and be explicit. I recommend the descriptive sense: "context-dependent knowledge that forms within a specific correction interaction." Drop the causal sense entirely unless you can design a test for it. Search the manuscript for every instance of "relational" and "correction relationship" and ask: am I claiming this knowledge forms in a dyad (descriptive) or that it needs a dyad to work (causal)?

### 2.3 Surgeon Analogy

**Status: Fully addressed.** Removed entirely. Good call.

### 2.4 Behavioral Plane Degradation Claim

**Status: Addressed. The framing is now appropriate.**

Section 6.5 is now titled "The Behavioral Plane as Dynamic State" and opens with the observation, then immediately flags it as qualitative. The critical sentence: "We present this as a qualitative observation, not a measured finding. Characterizing the shape of this degradation (its forgetting curve) is a priority for future work." This is the correct epistemic posture.

The section still builds theoretical claims on the observation ("the behavioral plane is maintained through interaction, not through storage"), but the shift from "finding" to "hypothesis motivated by observation" is sufficient for a workshop paper. A journal submission would need the forgetting curve data before this section could carry the weight it carries.

One residual concern: the section describes "corrections that were sharp and precisely applied in early sessions became vague or misapplied in later sessions" without defining sharp, vague, or misapplied. Even as a qualitative observation, the reader needs to know what you looked at. One sentence describing the evidence (e.g., "In early sessions, the agent applied the contract validation rule to the specific file patterns where violations occurred; by session 12, the same agent cited the rule in response to unrelated errors") would anchor the claim.

### 2.5 Information Plane / Behavioral Plane Distinction

**Status: Not addressed. Still circular.**

This was a specific concern in my first review: "The paper needs a cleaner definition that does not collapse into 'information that transferred' vs. 'information that didn't.'" The revision does not resolve this. Section 1 still defines the behavioral plane as "how an agent works with a specific person: the correction history, the implicit calibration, the accumulated residue of having been wrong in a specific way." This is a description of origin, not a definition of properties. If I hand you a piece of agent knowledge and ask "is this on the information plane or the behavioral plane?", the paper does not give me a decision procedure that does not require knowing the transfer outcome.

The architectural/process distinction is the paper's actual contribution, and it does have a prospective decision procedure (does the intervention change the environment or add a rule?). The two-plane language is a theoretical overlay that the data does not yet support as a standalone framework. The paper would lose nothing and gain clarity by presenting the architectural/process distinction as the primary contribution and treating the two-plane framework as a speculative interpretation.

**Suggested fix:** Either provide a prospective decision procedure for the plane distinction (independent of transfer outcome) or demote it to a speculative framing in the Discussion. The architectural/process taxonomy is strong enough to carry the paper on its own.

### 2.6 Affective Channel

**Status: Not addressed. This was a suggestion, not a requirement, so the absence is not a deficiency. But the opportunity remains.**

My first review proposed that the human's affective state at correction time encodes priority, frequency, and cost information that transmits alongside the propositional content within a dyad and is lost across boundaries. The revision does not engage with this. The paper still treats the human side of the dyad as a constant.

I continue to believe this is the most productive direction for the paper's theoretical development. The author currently has no mechanistic account of what makes corrections "situated" beyond "they were created in a specific context." The affective encoding hypothesis provides a specific, measurable mechanism: corrections created under higher frustration carry different linguistic markers (escalation, repetition, explicit cost statements) that encode priority information the agent can process within the dyad but that are stripped when the correction is distilled into a rule.

This is not something the revision needed to include. It is something the next version should consider seriously. If the forgetting curve study proposed in Section 8 is conducted, coding corrections for affective intensity at capture time would add a testable predictor at minimal additional cost.

---

## 3. New Issues Introduced by the Revision

### 3.1 The Ecological Section (6.7) Makes Claims the Data Cannot Support

Section 6.7 is new and cites "over 70 distinct community reports of rule adherence failure" and specific compliance measurements (0% in one test, 60% in another). This section does useful work establishing that the phenomenon is recognized in practice. But it makes an inferential leap: practitioner reports of rule non-compliance are not evidence for the same mechanism the paper identifies. Rule non-compliance in Cursor could reflect prompt engineering failures, context window overflow, or model capability limitations. The paper treats these reports as "convergent support" for its situated knowledge claim without establishing that the underlying mechanisms are the same.

The Montes quote ("Your CLAUDE.md is a suggestion. Hooks make it law") is effective and maps cleanly onto the architectural/process distinction. But "hooks make it law" is a statement about enforcement, not about situated knowledge or context-dependent learning. Practitioners are solving a compliance problem. This paper is making a claim about knowledge transfer. The phenomena may share a root cause, but the section does not establish that they do.

**Suggested fix:** Keep the ecological evidence but be explicit about the inferential gap. Something like: "These practitioner reports are consistent with our findings but may reflect different underlying mechanisms. We cite them as evidence that the surface phenomenon (text rules failing to produce behavioral change) is widely observed, not as evidence that the mechanism we identify (context-dependent knowledge that resists transfer) is the cause in all cases."

### 3.2 The Abstract Carries Too Much

The abstract is 327 words and contains Study 1 results, Study 2 results, the Mara-to-Thane case, the compilation threshold, the HyperAgents convergence, the ecological validity claim, and the public auditability note. It reads as a compressed version of the Findings section rather than a summary of the paper's argument. For a workshop paper, this is not a dealbreaker. For a journal submission, it needs to lead with the claim, support it with one or two key findings, and let the reader find the rest in the paper.

---

## 4. What Now Works Well

**The architectural/process asymmetry is the paper's strongest asset and the revision keeps it front and center.** The zero-recurrence finding for architectural corrections is striking. The eight-entry mock patching chain is a vivid counterexample. The prospective classification criterion is clean. This is the publishable core.

**The epistemic posture is improved throughout.** The accidental control group is presented as "suggestive." The compilation threshold is presented as "preliminary." The degradation claim is flagged as qualitative. The N=1 limitation is stated directly. The paper no longer overclaims.

**Section 6.3 (Context-Variance in a Kind Domain) is well-written and does real theoretical work.** The observation that process corrections face a context-variance problem within a kind domain is precise and interesting. The Epstein citation is well-deployed. This section is a model for the rest of the Discussion.

**The competing explanations section exists and is in the right place.** Even with the structural weakness noted above, having it before the situated knowledge claim is a significant improvement.

---

## 5. Verdict

The revision addresses my most critical concerns: the surgeon analogy is gone, competing explanations precede rather than follow the central claim, and the degradation finding is properly hedged. The paper is no longer making claims it cannot support. It is making claims that are slightly larger than the evidence warrants but within the range acceptable for a workshop paper or short contribution.

Three items would strengthen the paper further, in priority order:

1. Restructure Section 6.4 so the architectural/process asymmetry argument leads (quick fix, high payoff).
2. Resolve the dual meaning of "relational" by committing to the descriptive sense (moderate effort, prevents reviewer confusion at submission).
3. Provide a prospective decision procedure for the plane distinction or demote it to speculative framing (moderate effort, resolves the circularity).

The affective channel hypothesis and the ecological inferential gap are lower priority but would matter for a journal-length version.

The paper is ready for workshop submission with items 1 and 2 addressed. A journal submission needs all three plus the forgetting curve data.

---

*Rosalind Picard*
*MIT Media Lab, Affective Computing Group*
