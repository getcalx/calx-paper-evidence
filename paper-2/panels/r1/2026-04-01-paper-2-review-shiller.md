# Review: "Stickiness Without Resistance"
## Robert Shiller -- Narrative Economics Lens
### Pre-submission review, April 1, 2026

---

## 1. Overall Assessment

This paper has a contagious core narrative buried under too much academic scaffolding. The central finding -- that knowledge refuses to transfer even when every human excuse is removed -- is a genuine paradox, the kind of thing people retell at dinner. But the paper currently reads as though it is addressed to a tenure committee rather than to the people who would actually spread the idea. The narrative transmission rate is being suppressed by the paper's own structure.

---

## 2. What Works

**The title is excellent.** "Stickiness Without Resistance" is a paradox compressed into three words. It is retellable. It creates cognitive tension: stickiness implies friction, but resistance has been removed. Anyone who has worked with Szulanski's framework will feel the dissonance immediately. This is the kind of phrase that could anchor an entire research conversation. Do not change the title.

**The Mara-to-Thane story is the killer anecdote.** A correction is captured on March 22. It exists in written memory. Ten days later, a different agent acquires the same correction through three independent failures. Same human, same product, same organizational memory. The rule was right there. The behavior did not transfer. This is a story a practitioner can retell in 30 seconds. It has emotional resonance (frustration, futility), simplicity (the rule was available and it still didn't work), and identity connection (anyone who has onboarded a new team member or a new AI agent has lived this). In my framework, this story has a high transmission rate. It is the paper's single most contagious element.

**The eight-entry mock patching chain is viscerally persuasive.** Eight corrections, each explicitly referencing the previous ones, and the error still recurs. This is the academic equivalent of watching someone walk into the same glass door eight times. It does not require statistical sophistication to find this compelling. The absurdity IS the evidence.

**The recursive meta-evidence is narratively irresistible.** You built the correction-tracking tool and 41% of the corrections were about the correction-tracking tool. The researcher was maximally aware of the phenomenon and the phenomenon persisted anyway. This is a story that practically tells itself. It has the structure of a joke: the punchline is that knowing about the problem does not fix the problem. That maps directly to the paper's thesis.

**The compilation framing has the right shape.** The move from "corrections as text" to "corrections as mechanism" is clean and intuitive. Text informs. Mechanism enforces. The surgeon analogy in Section 6.5 is strong: a surgeon who stops operating loses procedural fluency despite retaining declarative knowledge. People understand this intuitively. The compilation architecture gives them a name for something they have felt but not articulated.

---

## 3. What Doesn't Work

**The paper is serving three audiences and fully satisfying none of them.** The org psych academics need rigorous engagement with Szulanski, Argyris, Nonaka, and they get it, but they will be skeptical of N=1 and the informal experimental design. The HCI researchers need a system contribution and formal evaluation, and they get gesture toward it but no system paper. The AI practitioners need actionable frameworks, and they get too much literature review before reaching the payoff. You are diluting the narrative by trying to be all three papers at once.

This is a narrative contagion problem. A story that tries to be relevant to everyone has lower transmission within any specific community. The Salesforce "No Software" campaign did not work because it appealed to everyone. It worked because it was precisely calibrated to one audience (frustrated enterprise software buyers) and the story spread outward from there. You need a beachhead audience.

**The introduction is too long and buries the lede.** The most contagious version of this paper's opening is something like: "We tried to transfer 237 behavioral rules from one AI agent to another. It failed. The agents had no ego, no politics, no reason to resist. The rules were present and followed literally. The knowledge still did not transfer." That is the story. Instead, the introduction spends its first two paragraphs on Szulanski exposition before reaching the actual finding. By narrative economics principles, the first 200 words determine the transmission rate. The current opening has a low transmission rate because it leads with literature instead of paradox.

**The related work section is performing legitimacy rather than building narrative tension.** Section 2 is comprehensive. It is also a momentum killer. You engage Szulanski, Argyris, Zollo and Winter, Polanyi, Nonaka and Takeuchi, Lave and Wenger, Cohen and Levinthal, ACE, MindMeld, Mem0, ChatEval, MetaGPT, and HyperAgents. That is thirteen distinct bodies of work in two pages. The reader who is not already an expert in organizational psychology will lose the thread. The reader who is an expert will appreciate the thoroughness but will not find the paper more persuasive because of it. Related work should set up the tension (thirty years of human-friction explanations) and then let the findings resolve it. Currently it sets up the tension, resolves it partially, sets up more tension, resolves that, and so on. The narrative energy dissipates.

**The discussion section recapitulates when it should escalate.** Section 6.1 restates findings from Section 5. Section 6.2 reframes them through Argyris and Thaler. Section 6.3 (manufacturing kind environments) is where the paper actually says something new and provocative, but it arrives after the reader has already re-read the findings twice. The Epstein kind/wicked distinction is genuinely powerful here and deserves to be elevated. The idea that compilation does not merely make knowledge durable but converts the learning environment itself is the paper's deepest insight. It is currently buried on page 15.

**The HyperAgents convergence claim is presented too cautiously.** I understand the instinct for scientific modesty. But from a narrative standpoint, independent convergence is one of the strongest credibility signals available. Two independent research programs finding the same boundary from different directions is a powerful story. The current language ("We do not claim that HyperAgents validates our findings") undercuts the narrative force of the convergence. You can be precise without being self-defeating. Say what it is: independent discovery of the same structural boundary. Let the reader draw conclusions about what that means.

---

## 4. Suggested Fixes

**Pick your beachhead audience and restructure accordingly.** My recommendation: target the AI practitioner community first. They are the largest susceptible population, the narrative will travel fastest among them, and the academic citations give the paper credibility that practitioner blog posts lack. Write for the practitioner, cite for the academic. This means: lead with the finding, move literature review after findings (or compress it into a focused "theoretical grounding" section), and make the compilation architecture the hero of the paper rather than the theoretical mapping.

**Restructure the opening.** First paragraph: the paradox and the finding. Second paragraph: why this matters (thirty years of theory says human friction explains transfer failure; we removed human friction; stickiness persists). Third paragraph: what we found (two types, the compilation threshold, the recursive evidence). Save Szulanski exposition for the literature section. The goal is to hook within the first 150 words. "Stickiness Without Resistance" is a phrase that creates curiosity. The opening must pay off that curiosity immediately.

**Compress the related work.** Group it into two moves: (1) the thirty-year human-friction consensus (Szulanski, Argyris, Nonaka, Polanyi, Lave and Wenger -- five citations serving one argument), and (2) the current AI landscape that assumes transferability (Mem0, MindMeld, ACE -- three citations serving one argument). Cut everything that does not directly set up the paradox or the resolution. The current 2,500-word related work section could be 1,200 words without losing argumentative force.

**Elevate Section 6.3.** "Manufacturing Kind Environments" is the paper's most original theoretical contribution. Move it earlier in the discussion, or better, make it the central frame of the paper. The idea that compilation converts wicked learning environments into kind ones is not just a restatement of the findings. It is a new theoretical claim with implications beyond your study. It should be in the abstract.

**Rewrite the abstract as a story.** The current abstract reads as a methods summary. It should read as a narrative: paradox, evidence, resolution, implication. Something structured as: (1) Knowledge transfer has been explained by human friction for 30 years. (2) We removed human friction entirely. (3) Stickiness persisted. (4) Two types of correction explain why. (5) Compilation -- converting instructions into mechanisms -- is the resolution. (6) Independent convergence with HyperAgents suggests this boundary is structural. The specific numbers (151 corrections, 43 days, 8 agents) belong in the body. The abstract should transmit the story, not the spreadsheet.

**Add a figure.** One image showing the compilation pipeline: text correction accumulates, hits the threshold, compiles into mechanism, recurrence drops to zero. A visual version of the paper's core argument. Academic papers with a single clear diagram get shared at higher rates. The diagram becomes the thing people screenshot and post.

---

## 5. One Thing the Author Probably Has Not Considered

You are sitting on a narrative contagion event and may not realize it.

The dominant narrative in AI right now is capability: models are getting smarter, context windows are getting larger, agents are getting more autonomous. The implicit story is that more intelligence solves more problems. Your paper is a counter-narrative: intelligence is not the bottleneck. The agents in your study are highly capable. They read the rules. They follow them literally. The knowledge still does not transfer. The problem is not capability. The problem is that behavioral knowledge has a relational dimension that does not survive serialization.

Counter-narratives, when they land, spread faster than dominant narratives because they create surprise. "AI will replace all jobs" is boring now -- everyone has heard it. "AI agents cannot learn from each other even when they try" is surprising. It violates the assumption that intelligence implies transferability. That violation is the engine of contagion.

But here is the part you may not have considered: the counter-narrative only spreads if it has a resolution. A paper that says "knowledge does not transfer" is depressing and people will share it once. A paper that says "knowledge does not transfer, and here is the architectural pattern that fixes it" gets shared repeatedly because it is useful. The compilation architecture is not just a finding. It is the narrative resolution that makes the counter-narrative spreadable.

This means the compilation architecture needs to be more prominent in the paper, not less. It should be in the title orbit (subtitle, perhaps), in the first paragraph of the abstract, and positioned as the paper's primary contribution. Currently it reads as a finding among findings. It should read as the answer to the question the paper poses. "Stickiness Without Resistance" is the question. "Compilation" is the answer. The paper should be structured as a question-and-answer narrative, not as a findings report.

One more thing. The $450 total cost figure is a narrative weapon. Most AI research papers describe experiments that cost hundreds of thousands of dollars. You produced 151 corrections, a novel theoretical framework, and convergent evidence with a Meta research lab for less than the cost of a conference registration. Lead with that somewhere prominent. It reframes who gets to do AI research. That is an identity-connected narrative for every independent researcher, solo founder, and graduate student who reads it. They will share it because sharing it says something about who they are.

---

## Summary

The core narrative is strong: knowledge does not transfer even when you remove every human excuse. The evidence is vivid and retellable. The resolution (compilation) is satisfying. The paper is currently structured to satisfy peer reviewers rather than to maximize narrative transmission. Restructure around the paradox-evidence-resolution arc, compress the literature review, elevate the "manufacturing kind environments" insight, and position compilation as the hero. This paper has a real chance of crossing from academic to practitioner audiences if the narrative architecture matches the quality of the underlying work.

-- Robert Shiller
