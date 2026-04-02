# Re-Review: "Stickiness Without Resistance" (Revised Draft)
## Reviewer: swyx (Shawn Wang)
## Date: 2026-04-01 (R2)
## Lens: Developer community reception, practitioner utility, ecosystem critique fairness

---

## 1. Overall Assessment

Better. Meaningfully better. The Nonaka SECI cut was the right call. Kahneman System 1/2 being gone removes the most "I took a psych class" energy from the paper. The ecological prevalence section (Section 6.7) is the single biggest improvement: it transforms the paper from "one guy noticed a thing" to "one guy formally characterized a thing that 70+ practitioners are complaining about independently." That is a different paper. It is a much more publishable paper.

The theoretical load is still heavier than optimal for a practitioner audience, but it is no longer the bottleneck. The bottleneck has shifted. I will get specific.

---

## 2. What Improved

**The ecological prevalence section lands.** Section 6.7 does exactly what I asked for: it connects the formal N=1 finding to the broader practitioner experience. The nedcodes 0% compliance stat, the Montes 60% compliance measurement, the "CLAUDE.md is a suggestion, hooks make it law" quote. This is the section that will get screenshotted and shared. It takes the paper from an isolated autoethnography to a formal characterization of a known ecosystem problem. The decision to present it as "convergent support, not part of our formal dataset" is the right epistemic move. It keeps the formal claims bounded while establishing that the phenomenon is real and widespread.

**The kind/wicked rewrite (Section 6.3) is tighter and more honest.** "Context-variance in a relatively kind domain" is a better frame than the original. You are no longer trying to manufacture kindness. You are observing that even in a kind domain (software, where tests pass or fail), process corrections face a context-variance problem because the "same" error manifests with different surface features each time. That is a narrower, more defensible claim. The mock patching example now does real work: 8 entries, each referencing prior corrections, each encountering the same conceptual error in a different module/path context. That is a clean illustration of why text rules are fragile even when feedback is clear.

**The interaction orientation section (6.6) is a smart addition.** Naming the scope condition that the phenomenon requires a "persistent collaborator" interaction style, not a "disposable tool" style, is honest and it preempts a real objection. Most people using Cursor or Copilot do not correct their agent. They re-prompt. They would never see this phenomenon. Saying that upfront, and then noting that the persistent-collaborator regime is growing (agent memory, CLAUDE.md, project-level instructions), positions the paper correctly: this is about the future mode of work, not the current median experience.

**Cutting Nonaka and Kahneman reduced Related Work by the right amount.** Section 2 is still long but it is no longer a literature review masquerading as a methods paper. Every remaining framework (Szulanski, Argyris, Zollo/Winter, Polanyi, Lave/Wenger, Cohen/Levinthal) now does specific work in the argument. I can trace a line from each citation to a claim in the findings. That was not true in the previous draft.

---

## 3. What Still Needs Work

**The practitioner summary is still missing.** I flagged this in R1: add a 1-page, zero-jargon section before or after the Abstract that states the finding, the taxonomy, and the practical implications for someone who maintains a CLAUDE.md or .cursor/rules directory and has 10 minutes. The ecological prevalence section partially serves this function, but it lives in Section 6.7, which is page 15+. The person who would benefit most from this paper will not reach page 15. They need the payoff on page 1. A practitioner summary at the front of the paper is the highest-leverage single change you could still make. Write it for the person reading Latent Space on their phone. Three paragraphs:
1. What we found (rules don't transfer, two types, one type works and one doesn't).
2. What it means for you (if you have corrected the same thing 3-4 times, stop writing rules and start writing hooks/tests/constraints).
3. What the ecosystem is getting wrong (agent memory systems assume transferability, the data says otherwise).

**The MindMeld critique improved but is still one layer too abstract.** You now say practitioner systems "share an implicit transferability assumption" and that "the cross-dyad extension requires empirical support that has not been provided." This is correct but it is still a claim about MindMeld, not a demonstration. My R1 asked for a concrete walkthrough: take a real rule, show it in MindMeld's format, walk through what happens when Agent B receives it without the correction history. You did not do this. I understand it may feel like punching down at a specific product. It is not. It is showing the reader how to audit their own rule set. The reader does not need to use MindMeld specifically. They need to see the failure mode in concrete terms so they can recognize it in their own system. One worked example, in a paragraph or a sidebar, would do more than the abstract critique.

**The Compound Engineering engagement is still a parenthetical.** You cite Shipper in Section 2.4 and the reference list. You never engage the thesis. Dan's argument is that the compound effect of engineering with agents comes from accumulating learnings across sessions. Your paper says those accumulated learnings are dyad-local and do not carry behavioral force across agent boundaries. That is a meaningful disagreement with a widely-read piece. You do not need to be adversarial. You need one paragraph in the Discussion that says: "The compound effect Shipper describes is real within the dyad that earned it. Our evidence suggests the compounding does not extend across dyad boundaries. The compound interest metaphor breaks at the transfer point." This is a two-minute addition that significantly increases the paper's relevance to the practitioner audience, because Compound Engineering is one of the most-read pieces in this space.

**The "this is just tests vs. comments" objection is still unnamed.** I flagged this in R1. Any senior engineer reading this paper will think: "We already know this. Tests beat comments. Type systems beat code review conventions. CI pipelines beat developer checklists. This is not new." You still do not name this objection or explain why the finding matters despite the pattern being familiar. The ecological prevalence section gets close (the industry is building infrastructure that assumes the opposite), but it does not make the argument explicitly. You need one paragraph, probably in the Discussion, that says something like: "Software engineers have known for decades that enforcement beats instruction: tests over comments, types over conventions, CI gates over checklists. The contribution is not the principle, which practitioners know intuitively, but the evidence that the agent tooling ecosystem has not absorbed it. Hundreds of millions of dollars in agent memory infrastructure assumes that captured rules transfer behavioral force across agent boundaries. Over 70 practitioner reports suggest they do not. The finding is not surprising. The industry's failure to act on it is."

**Section 2.3 (Tacit Knowledge and Situated Learning) is the remaining theoretical overhang.** Polanyi, Lave/Wenger, and Cohen/Levinthal are all saying versions of the same thing: knowledge is contextual, participation constitutes learning, prior experience enables assimilation. Three frameworks for one idea is two too many. I would pick Cohen/Levinthal (absorptive capacity), because it does the most direct work in your Szulanski barrier analysis, and compress Polanyi and Lave/Wenger to one-sentence supporting citations within the Cohen/Levinthal discussion. This is the last place where the theory-to-insight ratio is unfavorable. Everything else in Related Work now earns its space.

**The cost-of-re-derivation argument is still absent.** You state $450 total compute. You do not make the economic argument. If Faye needed 44 new corrections despite 237 transferred rules, and if each correction costs developer time + compute tokens + quality degradation during the re-derivation period, then every new agent onboarding has a calculable re-derivation cost. Architectural corrections avoid this cost entirely. That is the economic case for compilation. You do not need precision. You need a back-of-envelope estimate that practitioners can scale to their own team size. "If each correction costs approximately 15 minutes of developer attention and the re-derivation rate is 30% of the transferred rule set, onboarding a new agent to an existing codebase costs approximately N developer-hours of behavioral recalibration." This is the number that engineering managers will cite when justifying investment in hook infrastructure over rule-file maintenance.

---

## 4. Revised Priority List

From R1 I flagged 6 fixes. Here is where they stand:

| Fix | R1 Status | R2 Status |
|-----|-----------|-----------|
| Practitioner summary (page 1) | Missing | Still missing. Highest leverage remaining. |
| Trim Related Work | Too long | Improved. One more cut (Section 2.3 compression) would finish it. |
| Concrete MindMeld example | Missing | Still missing. One worked example needed. |
| Engage Compound Engineering | Parenthetical citation | Still parenthetical. One paragraph needed. |
| Cost-of-re-derivation estimate | Missing | Still missing. Back-of-envelope is fine. |
| Name the practitioner objection | Missing | Still missing. One paragraph needed. |

New items from this revision:

| Fix | Status |
|-----|--------|
| Ecological prevalence section | Landed. No changes needed. |
| Kind/wicked rewrite | Landed. No changes needed. |
| Interaction orientation scope condition | Landed. No changes needed. |
| Nonaka/Kahneman cuts | Landed. Right calls. |

---

## 5. The Compilation Load Question (Revisited)

In R1 I raised compilation load as a follow-on research question: compiled constraints accumulate, and at some point the accumulated hooks/tests/gates become hostile to developer velocity. The paper now has a natural place for this. Section 6.2 (The Compilation Architecture) ends with the double-loop framing. Add one paragraph at the end:

"The compilation architecture addresses transfer failure by converting process corrections into environmental mechanisms. An open question is compilation load: the accumulated weight of compiled constraints on the environment. Software teams have observed this dynamic with linting configurations, pre-commit hooks, and CI pipelines, where each individual constraint is justified but the aggregate creates rigidity. Characterizing the point at which compilation load begins to impede velocity, the constraint ceiling, is a natural extension of the compilation threshold we identify here."

Two sentences. It names the problem, positions you to own it, and signals to practitioners that you understand the full lifecycle, not just the first half.

---

## 6. Bottom Line

The paper moved from "interesting finding buried in organizational psychology" to "interesting finding with clear ecosystem relevance and ecological validation." The ecological prevalence section is the revision's strongest addition. The interaction orientation scope condition shows intellectual honesty that practitioners respect.

The remaining work is not theoretical. It is translational. The finding is strong. The data is honest. The ecological evidence is compelling. What is left is meeting practitioners at the door: a summary they can read in 3 minutes, one concrete worked example, one explicit engagement with the most-read piece in the space (Compound Engineering), one paragraph naming the obvious objection, and one rough economic number. None of these require new data or new theory. They require repackaging what is already in the paper for the audience that needs it most.

The paper is close. These are finishing moves, not structural changes.

---

*Reviewed as swyx, R2. Same lens: developer community reception, practitioner utility, ecosystem critique fairness.*
