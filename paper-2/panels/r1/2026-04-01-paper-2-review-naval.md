# Paper Review: "Stickiness Without Resistance"
## Reviewer: Naval Ravikant (lens: leverage)
## Date: 2026-04-01

---

## 1. Overall Assessment

This paper discovers a leverage architecture hiding inside an organizational psychology problem and almost fails to notice. The core finding is that one-time behavioral signals can be compiled into permanent environmental mechanisms, which is the definition of code leverage applied to human-AI coordination. The academic framing is precise and honest, but it buries the single most important implication: the compilation pipeline is the first credible description of how behavioral knowledge achieves zero marginal cost of replication.

---

## 2. What Works

**The two-type distinction is a leverage taxonomy.** Architectural corrections have zero marginal cost of enforcement. Process corrections have marginal cost on every invocation (the agent must retrieve, interpret, and apply the rule each time). This is the most important structural insight in the paper and it maps cleanly onto the distinction between labor leverage (process corrections require ongoing human effort to maintain) and code leverage (architectural corrections run without human intervention). You identified this empirically. That is valuable.

**The compilation pipeline is a leverage converter.** You have documented a mechanism that takes a fragile, high-marginal-cost knowledge asset (a text rule requiring ongoing compliance) and converts it into a durable, zero-marginal-cost enforcement mechanism (a structural constraint). This is "productize yourself" applied to behavioral knowledge. One correction relationship produces a mechanism that benefits every future agent in perpetuity. You understand this implicitly but the paper treats it as an organizational learning insight rather than an economic one.

**The recursive evidence is the strongest section.** Building the correction tool while generating corrections about the correction tool is the kind of evidence that cannot be designed in advance. It demonstrates that the phenomenon survives maximum awareness. That is a real finding.

**The honesty about limitations is exactly right.** N=1, single rater, single model family, no designed control. You state them directly and frame them as replication invitations. This is how you build credibility with people who will actually read the paper carefully.

---

## 3. What Doesn't Work or Needs Strengthening

**The compilation threshold is presented as a discovery when it should be presented as a waste metric.** You observe 3-4 recurrences before a process correction gets compiled into an architectural fix. You frame this as "the point at which single-loop learning fails visibly enough to trigger double-loop restructuring." Read that sentence again. You are describing a system that requires 3-4 failures before it does the correct thing. That is not a threshold to celebrate. That is a measurement of how much value is being destroyed by defaulting to process corrections in the first place.

The question the paper should ask: what if you compiled on the first occurrence? What is the cost of those 2-3 intermediate failures? The eight-entry mock patching chain is not evidence of a threshold. It is evidence of a system that tolerated seven unnecessary failures. The paper needs a section, even a short one, on the economics of early compilation versus late compilation. When is a process correction ever the right first response?

**The dyadic non-transferability claim is undersold.** You establish that each human-agent pair must build its own behavioral relationship from scratch. The paper treats this as a knowledge-transfer finding. It is also an economic finding with massive implications. It means that every company deploying agent teams is paying the full correction cost per agent, per human, with no economies of scale on the behavioral plane. Shared memory systems (Mem0, Zep) are selling information-plane solutions to a behavioral-plane problem. You gesture at this in Section 2.5 but you do not make the economic argument explicit. The dyadic constraint is a moat if you solve it and a scaling wall if you do not.

**The paper lacks a cost model.** You report $450 total compute cost, which is fine for credibility. But you never quantify the cost of process correction failure versus the cost of architectural intervention. How much developer time was spent on the eight-entry mock patching chain? How much would it have cost to compile on the first or second occurrence? The compilation pipeline is a capital allocation decision. Treat it like one.

**The "accidental control group" framing is too apologetic.** Four domains with rules and zero corrections. You hedge this heavily ("we cannot rule out that those domains were simply unused"). Fine, the hedge is scientifically appropriate. But the finding is directionally important even with the caveat: rules without the correction cycle behind them are inert. That maps to a broader principle: documentation without skin in the game produces nothing. You could say this more forcefully while keeping the caveat.

**Section 6.3 (Manufacturing Kind Environments) is the best section and it is too short.** Converting wicked learning environments into kind ones through compilation is the mechanism that makes everything else in the paper work. It deserves more space than the Kahneman System 1/System 2 mapping in 6.2, which is competent but not novel.

---

## 4. Suggested Fixes

1. **Add a "Cost of Delayed Compilation" subsection to the Discussion.** Take the mock patching chain (8 entries). Estimate the developer time per recurrence (even roughly). Show that each intermediate process correction had a cost. Then show that the architectural fix had a one-time cost with zero ongoing cost. This makes the compilation pipeline legible as a capital investment, not just a learning mechanism.

2. **Reframe the compilation threshold as a detection problem, not a natural boundary.** The 3-4 recurrence threshold is not intrinsic to the phenomenon. It is the point at which the current system detects the need for compilation. A better system would detect it earlier. State this explicitly. The threshold is an artifact of the detection mechanism, not a property of the knowledge.

3. **Add one paragraph to the Discussion on the economic implications of dyadic non-transferability for agent teams.** Any company running 10 agents with 5 humans has 50 dyadic relationships, each requiring its own correction investment. Shared rules help on the information plane but do not help on the behavioral plane. This is a scaling cost that nobody building multi-agent systems is accounting for. Name it.

4. **Expand Section 6.3 by at least 50%.** The kind/wicked environment conversion is the mechanism. It is doing more theoretical work than anything else in the Discussion. Give it room.

5. **Cut or compress the System 1/System 2 mapping in 6.2.** It is accurate but it is the most predictable section in the paper. Kahneman dual-process is the first thing every reviewer will expect. It does not add enough to justify its length relative to the kind/wicked conversion, which is genuinely novel in this application.

---

## 5. One Thing the Author Probably Has Not Considered

The compilation pipeline is not just a leverage mechanism. It is a judgment amplifier.

Here is the logic. In a leveraged system, the quality of one decision matters more than the quantity of effort. Your compilation pipeline takes a judgment call (this process correction should become an architectural constraint) and amplifies it across every future agent interaction in that environment, forever. That single judgment call has infinite time horizon returns. It is the highest-leverage decision in the entire system.

But the paper treats all compilation events as equivalent. They are not. Some architectural corrections prevent trivial errors (import ordering). Others prevent catastrophic ones (deployment configuration drift). The leverage of each compilation event is proportional to the severity and frequency of the error it eliminates. A compilation that prevents a class of errors occurring 20 times per month at high severity is orders of magnitude more valuable than one preventing a rare cosmetic issue.

This means the compilation pipeline needs a priority function, not just a recurrence threshold. The paper should at least acknowledge that not all compilations are equal and that the ordering of compilation events is itself a high-leverage judgment call. The person deciding what to compile next is making the most consequential decision in the system. That is where specific knowledge (the human's unique understanding of which errors matter most) meets code leverage (the compilation mechanism that eliminates them permanently). That intersection is the entire game.

The paper has all the raw material for this argument. It just does not make it.
