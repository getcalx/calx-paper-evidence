# Paper Review R2: "Stickiness Without Resistance"
## Reviewer: Naval Ravikant (lens: leverage)
## Date: 2026-04-01
## Status: Second review (post-revision)

---

## 1. Overall Assessment

Significantly improved. The System 1/2 cut was the right call. The ecological prevalence section (6.7) does real work. The interaction orientation section (6.6) is honest in a way most papers are not. The paper is now tighter, more honest about its scope, and makes a stronger empirical case.

But the economic argument is still not in the paper. The revisions made it a better academic paper. They did not make it a better leverage paper. You are still describing the phenomenon more carefully than you are describing the value of solving it.

---

## 2. What Improved

**The ecological prevalence section changes the game.** 70+ practitioner reports across 6 platforms, 0% compliance in controlled tests, 60% compliance in separate measurement, 8+ community tools built around the limitation. This is no longer N=1 discovering a curiosity. This is N=1 formally characterizing a phenomenon that an entire ecosystem is experiencing and building workarounds for. The Montes quote ("Your CLAUDE.md is a suggestion. Hooks make it law") is practitioners independently arriving at your architectural/process distinction in their own vocabulary. That is convergent evidence of the strongest kind: people who have never read your paper are building systems that embody your thesis.

This changes the leverage calculation. My first review treated the paper as interesting-if-replicated. The ecological evidence makes it interesting now. You have formal mechanism plus informal prevalence. That combination is how you get a paper that practitioners actually cite.

**The System 1/2 cut was correct.** Section 6.2 (now "The Compilation Architecture") is cleaner and more focused. You removed the most predictable section and replaced it with nothing, which was the right move. The paper does not miss it.

**Section 6.3 is better but different from what I asked for.** I said expand the "manufacturing kind environments" argument. You rewrote it as "context-variance in a relatively kind domain," which is a narrower and more defensible claim. You argue that software engineering is already kind (fast feedback, clear correctness criteria) and that process corrections fail because the same error manifests with different surface features, not because the environment is wicked. This is more honest than the original. You gave up a bigger claim for a more precise one. I respect the trade.

**Interaction orientation (6.6) is genuinely novel.** Most papers bury their scope conditions in limitations. You promoted yours to the Discussion and made an affirmative argument: this phenomenon only manifests when the human treats the agent as a persistent collaborator. Transactional users will never see it because they re-prompt rather than correct. This is important because it identifies the user population for whom the findings matter (and it is the user population that is growing fastest as agent memory becomes standard infrastructure).

---

## 3. What Is Still Missing

**The compilation priority function.** I flagged this in my first review as the "one thing the author probably has not considered." The revision does not address it. The paper still treats all compilation events as equivalent. They are not. An architectural correction that prevents deployment configuration drift is orders of magnitude more valuable than one that prevents import ordering errors. The compilation pipeline needs a severity weighting, or at minimum an acknowledgment that the ordering of compilation events is a high-leverage judgment call. You do not need to solve this in the paper. You need to name it. One paragraph in the Discussion or Future Work. The person deciding what to compile next is making the most consequential decision in the system. Say it.

**The cost of delayed compilation.** I asked for this explicitly. The revision does not include it. The eight-entry mock patching chain is still presented as evidence of a compilation threshold rather than evidence of value destruction. You now frame the 3-4 recurrence pattern more carefully (acknowledging the circularity risk, noting it may reflect a design parameter). Good. But you still do not ask the question: what did those 2-3 intermediate failures cost? Even a rough estimate (hours of developer time per recurrence, multiplied across the chain) would convert the compilation threshold from a learning theory observation into an economic argument. Every recurrence before compilation is wasted developer time. That sentence does not appear in the paper.

**The economic implications of dyadic non-transferability for agent teams.** I flagged this specifically. The revision adds the ecological prevalence section, which helps, but the economic argument is still implicit. Let me make it explicit so you can see what is missing.

Any company deploying 10 agents across 5 humans has 50 dyadic relationships. Each one requires its own correction investment. There are no economies of scale on the behavioral plane. Shared memory systems (Mem0, Zep, Letta) sell information-plane solutions to a behavioral-plane problem. Your ecological evidence now shows that the entire ecosystem is experiencing this. 70+ reports. Community tools. 0% compliance measurements. These practitioners are paying the behavioral correction cost per agent, per human, with no scaling mechanism. And nobody is accounting for it because nobody has named it.

You have the evidence. You have the framework. You have the ecological prevalence. You do not have the paragraph that ties them together into an economic argument. The dyadic constraint is the moat if you solve it. It is the scaling wall for everyone who does not. One paragraph in Section 6 that says this explicitly would be the most cited paragraph in the paper by practitioners.

**A cost model, even a rough one.** $450 total compute is reported. But the cost that matters is not compute. It is developer time. How many hours went into the 151 corrections? How many hours per recurrence in the mock patching chain? What is the implied hourly cost of process correction failure versus the one-time cost of architectural compilation? I understand you may not have tracked hours precisely. A rough estimate with stated assumptions is better than silence. The compilation pipeline is a capital allocation decision. The paper describes its mechanics without pricing it.

---

## 4. Assessment of Specific Revisions

| My R1 Suggestion | What You Did | Verdict |
|---|---|---|
| Cut/compress System 1/2 (6.2) | Cut entirely, replaced with tighter compilation section | Better than asked |
| Expand 6.3 (manufacturing kind environments) | Rewrote as context-variance argument, narrower claim | Different but defensible. More honest. |
| Add cost of delayed compilation | Not addressed | Still missing |
| Reframe threshold as detection problem | Partially. Circularity risk acknowledged, design parameter noted. But still presented as a finding rather than as evidence of detection latency. | Half done |
| Economic implications of dyadic non-transferability | Not addressed directly. Ecological prevalence section adds supporting evidence but the economic paragraph is absent. | Still missing |
| Compilation priority function | Not addressed | Still missing |

---

## 5. The Leverage Argument the Paper Still Will Not Make

Here is what the paper describes, restated in leverage terms.

Process corrections are labor. Each one requires ongoing human effort (the agent must retrieve, interpret, and apply the rule each session, and the human must re-correct when it fails). The marginal cost of enforcement never reaches zero. The 50% persistence rate means that roughly half of all process corrections will require re-investment indefinitely.

Architectural corrections are code. Once compiled, they enforce at zero marginal cost forever. No retrieval. No interpretation. No application failure. The environment handles it.

The compilation pipeline is a leverage converter. It takes labor (fragile, high marginal cost) and converts it to code (durable, zero marginal cost). Each compilation event is a permanent reduction in ongoing enforcement cost.

The 3-4 recurrence threshold is a waste metric. It measures how many times you pay the labor cost before converting to code. A system that compiles on first occurrence eliminates 2-3 rounds of wasted labor. A system that never compiles pays the labor cost forever.

The dyadic constraint means there are no economies of scale on the labor side. Every human-agent pair builds its own behavioral relationship. Shared rules help on the information plane (what agents know) but not on the behavioral plane (how they work with a specific person). At enterprise scale, with hundreds of agents and dozens of humans, the behavioral correction cost grows combinatorially while the information cost stays flat.

The only way to bend the scaling curve is to compile faster and more aggressively. The compilation pipeline is not a learning mechanism. It is a scaling mechanism. The priority function (which corrections to compile first) is the highest-leverage decision in the system because it determines how fast you reduce the combinatorial behavioral cost.

Every piece of this argument exists in the paper as raw material. It is never assembled. I understand this is an academic paper and the audience is organizational psychology and HCI researchers. But the practitioners who are your actual early audience, the 70+ people generating those community reports, they think in costs and leverage, not in Argyris loops. One subsection in the Discussion that makes the economic argument explicitly would be the bridge between the academic contribution and the practitioner audience.

---

## 6. Final Verdict

The paper is now a solid piece of academic work. The ecological prevalence section elevates it from "interesting single case" to "formal characterization of a widely-experienced phenomenon." The scope conditions are stated honestly. The limitations are specific and generative. The System 1/2 cut made room for better material.

Three things would make it complete:

1. One paragraph on the economics of dyadic non-transferability at scale. The combinatorial cost argument. You have all the evidence. Assemble it.

2. One paragraph naming the compilation priority function. Not solving it. Naming it. Acknowledging that not all compilations are equal and that the ordering decision is the highest-leverage judgment in the system.

3. One rough cost estimate of delayed compilation. Even "the eight-entry mock patching chain represents approximately X hours of developer time that would have been eliminated by compiling at the second recurrence" is sufficient. Price the waste.

These are not structural changes. They are three paragraphs that convert the academic contribution into an economic one. The paper works without them for an academic audience. It does not work without them for the audience that will actually build on it.

The paper has gotten more honest with each revision. That is the right direction. Now make it more useful.
