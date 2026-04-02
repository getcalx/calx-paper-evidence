# Pre-Submission Review: "Stickiness Without Resistance"

**Reviewer:** Adam Grant (organizational psychology lens)
**Date:** 2026-04-01
**Paper:** Hardwick, S. (2026). Stickiness Without Resistance: Knowledge Transfer Failure in Human-AI Collaboration Without Human Friction.

---

## 1. Overall Assessment

This is an ambitious paper that takes Szulanski's stickiness framework seriously and extends it in a direction the original literature never anticipated. The core move -- showing that two of four barriers produce stickiness even when the social/motivational barriers are removed by construction -- is genuinely interesting and, if it holds, constitutes a real contribution to knowledge transfer theory. The paper is weakened by the N=1 design and by several places where the theoretical mapping is asserted rather than earned. It is not ready for a top org psych venue (AMR, OrgSci, AMJ) in its current form, but could be competitive at CSCW or a CHI workshop if the theoretical claims are tightened and the limitations are handled more aggressively.

---

## 2. What Works

**The Szulanski decomposition is the right move.** Breaking Szulanski's four barriers into two social (motivation, relationship quality) and two structural (causal ambiguity, absorptive capacity), then designing a system that eliminates the social pair by construction, is a clean quasi-experimental logic. Reviewers in the knowledge transfer community will recognize this immediately. The 2x2 decomposition makes the contribution legible.

**The architectural vs. process distinction has real theoretical legs.** Mapping this onto Argyris's single-loop/double-loop framework is the strongest theoretical contribution in the paper. The data actually support the mapping: zero recurrence for architectural corrections, 50% persistence with failure chains for process corrections. That is a clean empirical separation that maps onto a 50-year-old theoretical distinction and extends it to a new domain. This is the part that would get you invited to revise.

**The recursive meta-evidence is methodologically interesting.** Building the correction tool while studying corrections, and finding that 41% of Study 2 corrections were about the correction system itself, is an unusual and compelling form of ecological validity. It preempts the "awareness eliminates the effect" objection before reviewers raise it.

**Honest limitations section.** Six specific, actionable limitations that each define a replication pathway. This is how limitations should be written. It reads like someone who actually wants to be challenged rather than someone performing humility. Reviewers notice.

**The Lave and Wenger situated learning application is apt.** The argument that each human-agent dyad is an isolated participation structure with no cross-agent community of practice is the kind of insight that makes the organizational theory community take notice. It is the right framework for explaining why rules-as-artifacts do not carry behavioral force.

---

## 3. What Doesn't Work or Needs Strengthening

**The "absorptive capacity" claim is underdeveloped.** Cohen and Levinthal (1990) define absorptive capacity as dependent on prior related knowledge. You invoke this correctly in the Related Work, but in the findings and discussion, you slide between two different claims. One is that the receiving agent lacks absorptive capacity because it lacks the correction history. The other is that the receiving agent lacks absorptive capacity because it has no experiential context for the rule. These are not the same thing. Cohen and Levinthal's absorptive capacity is about knowledge base overlap: can the recipient recognize, assimilate, and exploit the knowledge based on what it already knows? An LLM has enormous prior knowledge. The question is whether "correction history" qualifies as the kind of prior related knowledge Cohen and Levinthal meant, or whether you are extending the concept to cover something they were not describing (experiential/relational conditioning). Either is defensible, but you need to own the extension explicitly. Right now you treat it as a straightforward application when it is actually a redefinition.

**The causal ambiguity argument needs sharper evidence.** Szulanski's causal ambiguity is specifically about the source's inability to articulate why a practice works. In your system, the source (the human) can articulate the causal chain. The rules were written by the developer who understands the underlying logic. The problem is not that the source cannot explain why -- it is that the explanation does not survive compression into a rule. That is closer to Nonaka's externalization failure than to Szulanski's causal ambiguity. You may be dealing with a different mechanism than the one you are naming. If the developer can articulate the full causal chain but chooses to compress it into a heuristic rule, that is a design choice about codification granularity, not causal ambiguity in Szulanski's sense. Engage this distinction directly or you will get a Reviewer 2 who makes this their entire review.

**The "removing human friction" claim is overstated in places.** You say motivation and relationship quality are "eliminated by design." This is mostly true but needs a caveat. AI agents comply literally, but literal compliance is not the same as motivated adoption. In human systems, a motivated recipient actively seeks to understand and contextualize transferred knowledge. AI agents execute instructions without seeking understanding. You could argue this is actually a form of low motivation (compliance without engagement) rather than motivation eliminated. A careful reviewer will push on this. Address it preemptively.

**The SECI model application is forced.** The claim that the correction pipeline follows the SECI cycle and "breaks at the externalization-to-internalization transition" is stretched. Nonaka's SECI model describes knowledge conversion within an organization through specific social processes. Socialization requires shared experience between humans. Applying "socialization" to what happens within a human-agent dyad requires justification you do not provide. The correction cycle is better described as a variant of experiential learning (Kolb, 1984, which you do not cite) than as SECI. If you keep SECI, defend the analogical mapping or a reviewer will dismantle it.

**The Kahneman System 1/System 2 mapping in the Discussion is decorative.** The claim that compiled mechanisms are "System 1 analogs" for AI agents is metaphorical, not mechanistic. LLMs do not have dual-process architecture. A type system that prevents invalid inputs is not operating as automatic processing in Kahneman's sense -- it is operating as an environmental constraint. You already have the right framework for this (Thaler's choice architecture, which you cite immediately before). The System 1/System 2 mapping adds nothing that choice architecture does not already cover, and it invites the objection that you are applying a cognitive theory to a system that does not have cognition. Cut it or reframe it explicitly as metaphor.

**The HyperAgents convergence is presented too strongly for what it is.** You are careful in places ("we do not claim that HyperAgents validates our findings") but the framing across the paper treats it as near-confirmation. Two independent programs finding a boundary between text-level and mechanism-level improvements is suggestive, but the systems, populations, metrics, and even the nature of the "mechanisms" are all different. The convergence is on a high-level distinction, not on a specific finding. Dial back the weight it carries in the abstract and conclusion.

**Missing citation: Kolb (1984).** The entire paper is about experiential learning -- knowledge that forms through the experience of being corrected, not through receiving information. Kolb's experiential learning cycle (concrete experience, reflective observation, abstract conceptualization, active experimentation) maps directly onto your correction pipeline. Its absence is conspicuous.

**Missing citation: Argote (1999/2013).** Linda Argote's work on organizational learning and knowledge transfer, particularly on the decay of organizational knowledge and the role of experience in knowledge retention, is directly relevant. Her finding that knowledge depreciates rapidly in organizations parallels your behavioral plane degradation observation.

---

## 4. Suggested Fixes

1. **Redefine absorptive capacity explicitly.** Add a paragraph in Section 6.1 that says: "We extend Cohen and Levinthal's concept of absorptive capacity from prior domain knowledge to prior correction experience. The receiving agent possesses broad domain knowledge but lacks the specific experiential residue of having been wrong in the relevant way. This is an extension of the original concept, not a direct application." Own the extension.

2. **Reframe causal ambiguity as codification loss.** The developer can explain why a rule matters. The rule itself cannot. That is not causal ambiguity in Szulanski's sense (the source cannot explain). It is closer to what I would call "codification compression loss" -- the inevitable information loss when contextual, relational knowledge is compressed into a transferable artifact. Position this as a new mechanism that operates alongside causal ambiguity rather than as an instance of it. This actually strengthens your contribution: you are identifying a fifth barrier, not just confirming two of four.

3. **Cut or heavily qualify the System 1/System 2 mapping.** Replace it with a paragraph explicitly stating that the dual-process metaphor is illustrative, not mechanistic, or cut the section entirely and let choice architecture carry the theoretical weight.

4. **Add Kolb (1984) and Argote (1999).** Kolb belongs in Section 2.3 alongside Lave and Wenger. Argote belongs in Section 2.1 alongside Szulanski. Both directly support your argument.

5. **Tighten the SECI application.** Either defend why "socialization" applies within a human-agent dyad (what counts as shared experience when one participant is an LLM?) or replace the SECI framing with Kolb's experiential learning cycle, which requires only that the learner undergoes concrete experience followed by reflection. The correction cycle is a cleaner fit for Kolb than for Nonaka.

6. **Add a "boundary conditions" subsection to the Discussion.** Where does the stickiness-without-resistance framing apply and where does it not? Are there human-AI collaboration types where the process/architectural distinction would not hold? (Creative writing? Strategic reasoning? Tasks without clear correctness criteria?) Specifying the boundary strengthens the claim by making it precise rather than universal.

7. **Downweight HyperAgents in the abstract.** Move the convergence mention from the abstract to the findings/discussion only. The abstract is already dense. Let the core contribution (stickiness persists without human friction, two types with different transfer properties) carry the weight.

---

## 5. One Thing the Author Probably Hasn't Considered

The paper treats "stickiness without resistance" as if removing resistance reveals the underlying structural barriers. But there is a more provocative reading. What if resistance is not just a barrier to transfer -- what if it is, in some cases, a mechanism that enables transfer?

Here is the logic. In human teams, a recipient who resists a transferred practice has to engage with it. They push back, ask why, negotiate implementation details, argue about applicability to their context. That friction forces exactly the kind of contextual processing that your data shows is missing when process rules are transferred to compliant AI agents. The agent reads the rule and follows it literally. The resistant human reads the rule and fights with it, and in fighting with it, builds the contextual understanding that makes the rule load-bearing.

This connects to my work on productive conflict. Psychological safety does not mean absence of friction. It means friction in service of understanding. The most effective teams disagree more, not less. Your AI agents are maximally "safe" -- zero friction, zero resistance -- and this may be precisely why the rules do not stick. They are followed without being understood.

If this holds, it reframes your contribution. The paper is currently framed as "stickiness persists despite removing resistance." The deeper finding might be "removing resistance makes transfer harder, not easier, because resistance was doing epistemic work." That is a much more interesting claim. It means Szulanski's four barriers are not all pure impediments. Some of them (particularly the relational friction around arduous relationships) may serve a dual function: they impede transfer AND they force the kind of engagement that enables it.

You do not need to make this the central claim. But mentioning it in the Discussion as an alternative interpretation would significantly strengthen the paper. It turns a limitation (AI compliance is not the same as motivated adoption) into a theoretical insight.

---

## Summary Verdict

The core contribution is real: demonstrating that knowledge transfer stickiness has a structural component that survives the removal of social barriers is worth publishing. The paper needs tighter engagement with the specific mechanisms in the knowledge transfer literature (especially around absorptive capacity and causal ambiguity), the removal of decorative theory (System 1/System 2), the addition of missing citations (Kolb, Argote), and a more careful handling of the SECI model. With those fixes, this is competitive at CSCW. Without them, Reviewer 2 at any org psych venue will find the theoretical seams and pull.

The "resistance as enabler" angle in Section 5 is the kind of counterintuitive reframing that turns a solid empirical contribution into a paper people actually talk about. Consider it seriously.
