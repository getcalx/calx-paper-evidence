# Review R2: "Stickiness Without Resistance"
## Robert Shiller -- Narrative Economics Lens
### Second review, April 1, 2026

---

## 1. Overall Assessment

The paper is improved in structure. It is not yet improved in narrative architecture. Related Work is tighter. Two new sections (6.6 Interaction Orientation and 6.7 Ecological Prevalence) add real substance. But the core problem from my first review remains unaddressed: the introduction still buries the lede, and the paper's most contagious elements are still arranged for peer review rather than for transmission. I am going to be specific about what moved, what didn't, and what the retitling of 6.3 cost you.

---

## 2. What Improved

**Related Work is leaner and more purposeful.** Cutting the SECI model and the System 1/System 2 material was the right call. The section now makes two moves (human-friction consensus, then current AI landscape) rather than thirteen. It still runs long for my taste, but it no longer kills momentum the way it did in the first draft. A reader who survives the introduction will now survive the literature review. That is progress.

**Section 6.7 (Ecological Prevalence) is the single best addition.** This is the section I did not ask for and should have. Over 70 practitioner reports of rule adherence failure across six tools. A controlled test measuring 0% compliance. A separate measurement at 60%. The practitioner who independently concluded "Your CLAUDE.md is a suggestion. Hooks make it law." This section does something the rest of the paper cannot do alone: it establishes that the phenomenon is not idiosyncratic to your setup. For narrative contagion purposes, this section is critical because it gives the reader a reason to believe this is about THEM, not about you. An N=1 autoethnographic study is easy to dismiss as a curiosity. An N=1 study that documents a phenomenon 70 other practitioners have reported independently is a leading indicator. The ecological prevalence section converts the paper from "interesting finding" to "someone finally formalized what we've all been experiencing." That is a categorically different narrative position.

One specific note: the practitioner quote ("Your CLAUDE.md is a suggestion. Hooks make it law") is more retellable than almost anything else in the paper. It compresses the entire architectural/process distinction into a sentence that a practitioner would actually say out loud. It belongs much earlier. Consider pulling it into the introduction as a framing device, or at minimum into the abstract.

**Section 6.6 (Interaction Orientation) is honest and strengthening.** Identifying that the phenomenon requires a specific interaction style (treating agents as persistent collaborators, not disposable tools) is good epistemics. But it is also good narrative, because it identifies the susceptible population: people who work with agents the way you do. That population is growing fast. The scope condition is also a market signal. The paper is describing the future of the problem, not an edge case in the present.

---

## 3. What Did Not Improve

**The introduction is unchanged.** I flagged this as the primary structural problem. It remains the primary structural problem. The first paragraph is still Szulanski exposition. The paradox still does not arrive until paragraph three. The most transmissible version of this paper's opening is still buried under two paragraphs of literature review that the reader has no reason to care about yet.

I will say this more directly than I did last time. The introduction is suppressing the transmission rate of everything that follows it. A reader who opens this paper and encounters "Szulanski (1996) showed that best practices are sticky" in the first sentence will categorize it as organizational psychology and decide within 15 seconds whether they are in that audience. Most AI practitioners will decide they are not. You lose them before they reach the paradox that would have hooked them.

The opening should be: the paradox, then the evidence, then the theoretical grounding. Not: the theory, then the evidence, then the paradox. The information is the same. The narrative architecture is different. And narrative architecture determines transmission.

I suggested a specific restructure last time. It still applies. First paragraph: the finding (we tried to transfer 237 rules; it failed; every human excuse was removed). Second paragraph: why this matters (thirty years of theory says human friction is the cause; we removed human friction). Third paragraph: what we found (two types, the compilation pattern, the recursive evidence). The Szulanski exposition belongs in Related Work, where readers who want the theoretical genealogy can find it.

**The abstract is still a methods summary.** It reads as a compressed version of the paper rather than as a narrative. An abstract that transmits the story would follow the paradox-evidence-resolution arc. The current abstract front-loads Szulanski, then lists findings. It should front-load the paradox, then deliver the resolution. The specific numbers (151 corrections, 43 days, 8 agents) can appear, but they should serve the story, not structure it.

**Compilation is still a finding among findings, not THE answer.** I flagged this last time as the single most important structural issue beyond the introduction. The paper poses a question: why does knowledge not transfer even when every human excuse is removed? The answer is: because text-based rules are the wrong medium; compilation into environmental mechanisms is the resolution. That question-and-answer structure should be the paper's spine. Currently, compilation appears as one discussion section among six. It is not in the title orbit. It is not the first thing the abstract resolves to. It is not the frame the conclusion lands on with force. The conclusion mentions it but shares the stage with the compilation threshold, the error floor, and HyperAgents convergence. Compilation should be the headline. Everything else is supporting evidence.

---

## 4. The Retitling of 6.3: What It Cost

The original title was "Manufacturing Kind Environments." The new title is "Context-Variance in a Relatively Kind Domain."

This is a significant narrative loss.

"Manufacturing Kind Environments" was a phrase that did intellectual work. It took Epstein's kind/wicked distinction and turned it into an action: compilation does not merely fix errors, it manufactures kindness in the learning environment itself. That is a theoretical claim with legs. It is retellable. It implies an entire research program. A reader encountering the phrase "manufacturing kind environments" would understand immediately that the paper is proposing something beyond error correction. The phrase reframes the entire compilation architecture as an environmental intervention.

"Context-Variance in a Relatively Kind Domain" is descriptive. It tells you what the section discusses. It does not make a claim. It does not create cognitive tension. It does not give the reader a phrase they can carry away and use in their own thinking. It is the kind of section title that belongs in a findings section, not in the discussion.

I understand why you changed it. The content of the section IS more measured now. You narrowed the claim from "compilation manufactures kind environments" to "compilation removes context-dependence within a domain that already provides reliable feedback." That is more defensible. But the narrower claim can still carry the bolder title. "Manufacturing Kind Environments" is not falsified by the narrower argument. A system that removes context-dependence from feedback within a kind domain is manufacturing a kinder version of that domain. The title was accurate. It was also contagious. The revision is accurate and inert.

My recommendation: restore the original title. Keep the more measured content. The tension between a bold section title and careful argumentation within it is a feature, not a bug. It signals that the author has a vision and is disciplining it with evidence. The current configuration signals that the author had a vision and retreated from it.

If you are concerned about overclaiming, add a single sentence at the end of the section: "We present this as a framing hypothesis warranted by the data, not as an established theoretical result." That provides the epistemic hedge without sacrificing the narrative force of the title.

---

## 5. Narrative Architecture: Where Things Stand

Here is the current narrative flow as a reader experiences it:

1. Szulanski exposition (low energy, academic audience only)
2. The paradox (arrives too late)
3. Findings enumerated (high density, low narrative)
4. Literature review (tighter but still long)
5. System description (necessary, fine)
6. Methodology (necessary, fine)
7. Findings (the good stuff, but reader may be fatigued)
8. Discussion (six subsections, the best material is in the back half)
9. Conclusion (adequate, not forceful)

Here is the narrative flow that maximizes transmission:

1. The paradox (we removed every human excuse; stickiness persisted)
2. The killer evidence (Mara-to-Thane, the mock patching chain, the 41% recursive meta-evidence)
3. The resolution (compilation: text does not transfer, mechanism does)
4. The ecological prevalence (70+ practitioner reports, you formalized what they experienced)
5. The theoretical grounding (Szulanski, Argyris, Zollo and Winter confirm the pattern)
6. The deep insight (manufacturing kind environments)
7. The convergence (HyperAgents found the same boundary independently)
8. The invitation (N=1, here are the six ways to replicate, $450, all artifacts public)

You do not need to reorder the paper into this exact sequence. Academic conventions demand certain structures. But the ENERGY of the paper should follow this arc. The introduction should accomplish items 1 through 3 in compressed form. The ecological prevalence section should move much earlier (it currently arrives at 6.7, after five discussion subsections). The "manufacturing kind environments" insight should be elevated, not buried.

---

## 6. The $450 Figure

I flagged this last time as a narrative weapon. It is still underdeployed. It appears in Section 3.3 (the extended dataset description) and in the conclusion. It should appear in the abstract or the introduction. The reason is not vanity. The reason is identity connection. Every researcher reading this paper who does not have Meta's compute budget will see $450 and think: I could do this. That thought converts a reader into a potential replicator. The $450 figure is not a cost detail. It is an invitation. Position it as one.

---

## 7. Specific Line-Level Notes

**6.3, paragraph 2:** "The eight-entry mock patching chain from Study 1 demonstrates this: the 'same' error manifests differently each time, with different module paths, different import structures, different test files." This is excellent. The scare quotes around "same" are doing real work. This sentence alone conveys why process corrections fail. It should be in the introduction.

**6.7, final paragraph:** "A full synthesis of the formal and ecological evidence is forthcoming [Hardwick2026c]." This is good practice (signaling that more is coming) but it slightly undercuts the current paper. The reader might think: should I wait for the next one? Consider reframing: "A comprehensive synthesis is forthcoming; this paper provides the formal mechanism that the ecological evidence converges upon."

**Section 9 (Conclusion):** "The surviving mechanism is architectural, not social." This is a strong sentence. It should be in the abstract. It might be the paper's second most transmissible phrase after the title.

---

## 8. Summary

The revisions show a paper that is becoming more disciplined. The ecological prevalence section is a genuine contribution to the narrative's credibility. The interaction orientation scope condition is honest and smart. Related Work is tighter.

But the three things that would most increase transmission remain unaddressed:

1. The introduction still leads with theory instead of paradox. This is the single highest-leverage change available.

2. Compilation is still one finding among many instead of the paper's central answer. Restructuring the narrative around a question (stickiness without resistance) and an answer (compilation) would give the paper a spine that everything else hangs on.

3. "Manufacturing Kind Environments" was traded for a descriptive title that does no intellectual work. The boldest theoretical contribution is now wearing the most cautious label.

The paper is getting more precise with each revision. Precision is important. But precision without narrative force produces papers that are cited by the 50 people who find them, not papers that reshape how a community thinks. This paper has the raw material to do the latter. The narrative architecture is what stands between the material and the impact.

-- Robert Shiller
