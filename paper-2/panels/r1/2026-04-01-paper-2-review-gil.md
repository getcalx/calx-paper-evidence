# Pre-Submission Review: "Stickiness Without Resistance"

**Reviewer:** Yolanda Gil
**Angle:** AI research methodology and scientific rigor
**Date:** April 1, 2026
**Target venues cited by author:** CHI 2027, CSCW

---

## 1. Overall Assessment

This paper makes a genuinely interesting observation: knowledge transfer stickiness persists when the recipient side of the transfer is an AI agent, removing the motivational and relational barriers that have dominated the organizational psychology literature for three decades. The architectural-vs-process distinction is intuitive and well-motivated. However, the paper's methodological foundation is too thin for the strength of its claims, and its engagement with AI knowledge representation literature is essentially nonexistent, which is a serious gap given that the core phenomenon is about why explicit knowledge fails to produce behavioral change in computational agents.

---

## 2. What Works

**The core observation is novel and publishable.** The insight that Szulanski's stickiness framework applies to human-AI dyads, and that removing two of four barriers still produces transfer failure, is a contribution worth making. The framing is clean: you set up the expectation that AI agents should be ideal knowledge recipients, then show they are not. That is the kind of counterintuitive finding that earns attention at CHI.

**The architectural-vs-process distinction has face validity.** The idea that corrections modifying the environment (zero recurrence) behave differently from corrections modifying instructions (50% persistence, long failure chains) is both intuitively sound and practically useful. The 8-entry mock patching chain is a vivid illustration. Reviewers will remember it.

**Honest limitation handling.** Section 7 is unusually direct for a paper at this stage. You name the single-rater problem, the lack of a designed control, the single model family. This is good practice and builds credibility. The framing of limitations as replication invitations is effective.

**The recursive meta-evidence is compelling as a methodological footnote.** Building the correction tool while studying corrections, and finding that 41% of the corrections are about correction tracking itself, is a nice piece of ecological validity. You do not oversell it.

**The Mara-to-Thane case is your strongest single data point.** Same human, same product, shared memory, written rule available, and the receiving agent still independently reacquired the correction through three recurrences over ten days. That is the paper's most vivid illustration of the phenomenon.

---

## 3. What Doesn't Work or Needs Strengthening

**The classification scheme is insufficiently formalized for reproduction.** The architectural-vs-process distinction is presented as a prospective classification, but the operationalization is narrative, not procedural. You say architectural corrections "modify the structure of the system itself" and process corrections "add text-based rules." A reviewer will ask: where is the decision procedure another researcher could follow? What counts as "structure"? Is a configuration file change architectural? Is a prompt template restructuring architectural or process? The boundary cases matter, and the paper provides one borderline example (duplicate model definitions) without a systematic treatment of the boundary. You report 10/11 clean classifications, but with a single rater who designed the scheme, this is not informative about reproducibility. An operational codebook with inclusion/exclusion criteria and worked boundary examples is necessary.

**N=1 is not just a limitation, it is a structural threat to every claim.** You acknowledge this, but I want to push harder. The paper makes six numbered contributions. Contributions 1, 2, 3, 4, and 6 all rest on one person's correction behavior. The compilation threshold of 3-4 recurrences could be a property of Spencer Hardwick's patience before escalating to structural fixes, not a property of human-AI correction dynamics. The 50% persistence rate for process corrections could reflect this particular developer's rule-writing style. Every quantitative finding is confounded with the researcher. At CHI or CSCW, at least one reviewer will argue this is a single-case study that belongs in a workshop or as a poster, not a full paper. You need a stronger methodological defense than "the limitations are specific invitations to replicate."

**The "accidental control group" is not a control group.** You almost say this yourself in Section 5.3 ("we cannot rule out the simplest alternative: that those domains were simply unused"), but then you list it as Contribution 3 in the introduction. Four domains with authored rules and no corrections, where no work was performed, cannot support the claim that "instruction without the correction cycle produces no learning." It supports the claim that "domains where no work happened showed no corrections," which is trivially true. Remove this from the contributions list or reframe it as a hypothesis for future designed experiments.

**The HyperAgents convergence is oversold.** You say "independent convergence" repeatedly and list it as Contribution 5. But the systems are fundamentally different: yours involves human-agent dyads with autoethnographic correction capture; HyperAgents involves agent-only optimization loops with quantitative metrics. The mapping from your "architectural vs. process" to their "meta-level vs. prompt-level" is a loose analogy, not a formal correspondence. Two groups observing that some improvements transfer better than others is not convergence on a boundary. It is two groups making a similar qualitative observation in different contexts. Downgrade this from a contribution to a discussion point.

**Autoethnographic methodology needs explicit methodological framing.** The paper never uses the word "autoethnography," but that is what this is: a practitioner systematically studying their own practice. There is a legitimate tradition of this in HCI (Cunningham and Jones, Desjardins and Ball, Lucero). You should name the method, cite the methodological literature that supports it, and address its known threats (confirmation bias, selective attention, motivated reasoning). Right now the methodology section reads as "I logged corrections while building software," which sounds ad hoc rather than methodologically grounded.

**Statistical claims without statistical tests.** "Approximately 50% persistence" for process corrections. Based on what denominator? How many process corrections total, how many recurred? The paper uses "approximately" throughout without confidence intervals, significance tests, or even simple counts that would let the reader verify the claim. For a field study with 151 data points, descriptive statistics with proper denominators are the minimum. A chi-square test on recurrence by correction type would take one line and would dramatically strengthen the architectural-vs-process claim.

---

## 4. Suggested Fixes

**4.1 Formalize the classification scheme.** Write an operational codebook: definition of each type, decision criteria, boundary cases with worked examples. Ideally, have 2-3 independent raters classify a subset of the 151 corrections blind and report Cohen's kappa. If you cannot get raters before submission, at minimum publish the codebook and the full correction log so that inter-rater reliability can be assessed post-publication.

**4.2 Name the method.** Add a paragraph to Section 4 explicitly framing this as autoethnographic field research. Cite the HCI autoethnography literature. State the known threats and how your design mitigates or fails to mitigate them. This gives reviewers a methodological frame to evaluate rather than dismissing the work as "just one person's anecdote."

**4.3 Provide complete descriptive statistics.** For every quantitative claim: state the count, the denominator, and the basis. "13 of 44 corrections (30%) fell in categories the transferred rules addressed" is good. "Approximately 50% persistence" is not. Report: N process corrections, N that recurred at least once, N that recurred 2+ times, N that recurred 3+ times. Report the same for architectural corrections. A simple 2x2 contingency table (type x recurrence) with a Fisher's exact test would be the strongest single addition to the paper.

**4.4 Demote the accidental control group and HyperAgents convergence.** Move both from the contributions list to discussion. They are interesting observations that motivate future work, not standalone contributions supported by this data.

**4.5 Add a figure.** The paper has zero figures. A correction timeline showing when architectural fixes eliminate error classes (flat zero after intervention) versus process corrections recurring (sawtooth pattern) would be more compelling than any paragraph of text. Reviewers at CHI expect visual evidence.

**4.6 Engage the knowledge representation literature (see Section 5 below).**

---

## 5. One Thing the Author Probably Hasn't Considered

The paper engages organizational psychology (Szulanski, Argyris, Nonaka), situated learning (Lave and Wenger), behavioral economics (Kahneman, Thaler, Epstein), and the agent memory literature. It does not engage the AI knowledge representation literature at all. This is a significant gap, and I think it actually weakens the paper's core argument.

Your central finding is that explicit declarative knowledge (rules) fails to produce reliable behavioral change in computational agents, while structural/procedural interventions succeed. This is precisely the distinction that knowledge engineering has been studying for forty years. The gap between declarative representations (ontologies, rule bases, knowledge graphs) and procedural execution (planning, workflow systems, constraint enforcement) is one of the oldest problems in AI.

Specifically:

**Ontologies and the frame problem.** Your process corrections fail because the rule does not carry the contextual conditions under which it applies. This is a version of the frame problem (McCarthy and Hayes, 1969): specifying all the conditions under which a rule holds and all the ways those conditions might change is intractable. Your architectural corrections succeed because they sidestep the frame problem: instead of specifying when the rule applies, they make the environment enforce it universally. The knowledge engineering community has known for decades that purely declarative knowledge bases require inference engines, context management, and closed-world assumptions to be behaviorally effective. Your process corrections are declarative knowledge without an inference engine.

**Knowledge-level analysis.** Newell (1982) distinguished the knowledge level (what an agent knows and wants) from the symbol level (how that knowledge is encoded and processed). Your behavioral plane is a knowledge-level concept. Your finding that rules at the symbol level (text in a context window) fail to produce reliable behavior at the knowledge level is consistent with Newell's framework: the symbol-level representation is not sufficient to determine knowledge-level behavior without the right processing architecture.

**Workflow and constraint systems.** Your architectural corrections function like workflow constraints in scientific workflow systems (Gil et al., 2011; Gil and Ratnakar, 2013). These systems learned the same lesson you are documenting: encoding best practices as declarative guidelines for human scientists to follow is unreliable. Encoding them as executable workflow constraints that the system enforces produces compliance. The parallel is direct and would strengthen your theoretical grounding considerably.

**Why this matters for the paper:** right now, your theoretical frame is "organizational psychology meets behavioral economics." That is interesting but it makes the paper legible primarily to social scientists. Adding knowledge representation gives you a second theoretical anchor in computer science. It also makes the contribution more precise: you are not just observing that "knowledge doesn't transfer." You are observing that declarative behavioral knowledge in LLM agents fails in predictable ways that the knowledge engineering community has characterized formally, and that the solution (compilation into environmental constraints) recapitulates solutions the knowledge engineering community arrived at decades ago in different systems. That makes the paper stronger, not weaker. It positions your finding as convergent evidence from a new setting for a known architectural pattern, which is exactly the kind of claim that survives peer review.

I would add a subsection to Related Work (perhaps 2.4.5 or a new 2.9) covering the knowledge representation angle, and a paragraph in the Discussion connecting your compilation architecture to the declarative-procedural gap in knowledge engineering. This does not require restructuring the paper. It requires perhaps 500 words and 5-6 citations.

---

## Summary Verdict

The core observation is publishable. The current draft is not ready for CHI or CSCW submission. The methodological apparatus needs tightening (formalize the classification, name the method, add descriptive statistics, add at least one figure), two of the six contributions need demotion, and the theoretical frame has a significant gap in knowledge representation. These are all fixable in one revision pass. The paper's honest limitation handling and the strength of the Mara-to-Thane case give it a foundation that most first drafts lack. Fix the method, tighten the claims, fill the KR gap, and this is a credible submission.

--- 

*Yolanda Gil is a Research Professor of Computer Science and Spatial Sciences at the University of Southern California and a Fellow of the Association for the Advancement of Artificial Intelligence (AAAI). Her research focuses on intelligent interfaces for knowledge capture and discovery, including work on collaborative knowledge-based systems, scientific workflows, knowledge engineering, and provenance. She directed the Information Sciences Institute's Interactive Knowledge Capture group and has published extensively on how to make knowledge systems that are usable, reproducible, and scientifically rigorous.*
