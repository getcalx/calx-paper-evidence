# Pre-Submission Review: "Stickiness Without Resistance"

**Reviewer:** Philip Guo (UC San Diego, CSE)
**Lens:** Developer experience, HCI/CSCW research methodology
**Date:** 2026-04-01
**Target venues:** CHI, CSCW

---

## 1. Overall Assessment

This paper identifies a real and underexplored phenomenon: that knowledge transfer stickiness persists in human-AI collaboration even when the classic human-friction explanations are removed. The core finding (architectural corrections transfer, process corrections do not) is clean, intuitive, and practically useful. However, the paper's methodological foundation is fragile in ways that CHI/CSCW reviewers will attack, and the framing as "evidence for a phenomenon, not proof of generalizability" may not be enough armor. The paper needs to either strengthen its methodological grounding as a rigorous autoethnographic study or reframe itself more explicitly as a systems contribution with empirical motivation.

---

## 2. What Works

**The phenomenon is real and the framing is sharp.** Szulanski is the right anchor. Removing two of four barriers and showing stickiness persists is a clean experimental logic, even if the "experiment" is observational. The sentence "We present evidence that stickiness persists when you remove the humans from one side of the transfer entirely" is the kind of hook that gets a paper read.

**The two-type distinction is the paper's strongest contribution.** Architectural vs. process corrections, classified prospectively by intervention type rather than retroactively by outcome, is methodologically sound and practically actionable. A DX tool builder could use this taxonomy tomorrow. The fact that it maps cleanly onto Argyris's single/double-loop distinction adds theoretical weight without feeling forced.

**The recursive meta-evidence is genuinely interesting.** Building the correction tracker while tracking corrections, and finding that 41% of Study 2 corrections were about the correction system itself, is a compelling ecological validity argument. You're right that if awareness eliminated the effect, the rate on the correction system should approach zero. It doesn't. That is a finding worth preserving.

**The HyperAgents convergence is well-handled.** You don't overclaim. You note the differences in systems, methods, and scope. You frame it as convergence on a boundary rather than mutual validation. This is the right tone.

**The limitations section is unusually honest.** Stating six specific limitations and framing each as a replication invitation is exactly right for a small-N study. Most authors would bury these. Surfacing them builds credibility.

---

## 3. What Doesn't Work or Needs Strengthening

**The autoethnographic framing is underdeveloped.** You're running an autoethnographic study but you never call it that. CHI has a robust tradition of autoethnographic and first-person research (Desjardins and Ball 2018, Lucero 2018, Neustaedter and Sengers 2012). You need to engage that literature explicitly. Right now, reviewers who know autoethnographic methods will see you doing it without acknowledging the tradition, and reviewers who don't will see N=1 without a methodological home. Either way you lose. Name the method. Cite the precedents. Explain why first-person methods are appropriate for surfacing a phenomenon that would be invisible in a controlled lab study (because the correction relationship forms over weeks of production work, not in a 90-minute experiment).

**The "accidental control group" is the weakest element and is overweighted.** Four domains with rules and no corrections, discovered post-hoc, where you cannot distinguish "rules are inert without correction cycles" from "nobody did any work in those domains." You acknowledge this, but you still list it as Contribution 3 and dedicate a findings subsection to it. A CHI reviewer will read this as padding. It is suggestive at best. Demote it to a brief observation within Discussion. Do not list it as a numbered contribution.

**The paper oscillates between "evidence for a phenomenon" and causal claims.** In the introduction, you carefully frame this as "evidence for a phenomenon, not proof of generalizability." Good. But then in Discussion you write things like "This extends Szulanski's framework: stickiness is not solely a human friction problem. It has an architectural component that persists independent of social dynamics." That is a causal claim. You are saying that removing social barriers reveals structural barriers in isolation. Your data are consistent with this interpretation, but they don't demonstrate it. You have one developer whose correction style might produce the stickiness. You have one model family whose context processing might produce the stickiness. The phenomenon is real. The mechanism attribution needs hedging.

**The theoretical apparatus is overfitted.** You invoke Szulanski, Argyris, Zollo and Winter, Polanyi, Nonaka and Takeuchi, Lave and Wenger, Cohen and Levinthal, Epstein, Kahneman, and Thaler. That is ten theoretical frameworks for a paper with 151 corrections from one developer. Each mapping is individually defensible, but the cumulative effect is a paper that feels like it is borrowing authority rather than building an argument. Pick three: Szulanski (the core framing), Argyris (the loop distinction), and one of {Lave and Wenger, Polanyi, Cohen and Levinthal} for the tacit knowledge angle. Move the rest to a "connections to related frameworks" paragraph in Discussion. The Kahneman System 1/System 2 mapping and the Thaler choice architecture mapping are the most expendable. They add metaphorical clarity but not analytical power.

**The practical tooling implications are underdeveloped.** You have a finding with direct DX implications: process corrections decay, architectural corrections don't, and there's a measurable compilation threshold around 3-4 recurrences. A DX tools researcher reading this wants to know: what should a correction-tracking tool actually do with this information? You gesture at "automated compilation pipeline" in Future Work, but the paper would be stronger if Discussion included a concrete design implication section. Something like: "A correction system that tracks recurrence counts and surfaces compilation candidates at the 3-recurrence threshold would operationalize the finding. The system would distinguish between corrections that need reinforcement (process type, below threshold) and corrections that need architectural intervention (process type, at or above threshold). Architectural corrections need neither: they are self-enforcing." This is the bridge from research finding to DX practice. Without it, the paper is a theory contribution dressed in systems clothing. CHI reviewers will want the systems implications spelled out.

**The cost framing is distracting.** You mention "$450 in compute costs" in both the abstract and conclusion. I understand the impulse: it signals accessibility. But in a CHI paper, the relevant cost is researcher time (43 days of full-time production work). The dollar figure reads as a pitch rather than a research contribution. Drop it from the abstract. If you want to keep it, one mention in the system description is sufficient.

**Study 2 is thin.** 17 corrections over 5 days, 3 agents. The Mara-to-Thane non-transfer is a single case. It is a vivid case, and you present it correctly as a within-researcher replication. But the paper leans on Study 2 more heavily than its size warrants. The compilation threshold finding (Section 5.4) rests on three correction chains from Study 2, each with 3-4 recurrences. That is 9-12 data points supporting what you frame as a measurable boundary. Be more explicit about the thinness. "The data points are few. The pattern is consistent across them" is good. But then don't list it as Contribution 4 with the same weight as the transfer stickiness finding (Contribution 1), which rests on 237 transferred rules and 44 corrections.

---

## 4. Suggested Fixes

1. **Add an autoethnographic methods section.** Two paragraphs in Methodology. Name the tradition. Cite CHI/CSCW autoethnographic precedents. Argue that first-person, longitudinal, production-context methods are necessary for surfacing phenomena that form over weeks of embedded practice. This is your strongest methodological defense and you are currently not making it.

2. **Restructure contributions by strength.** Contribution 1 (stickiness persists) and Contribution 2 (two types with different transfer properties) are strong. Contribution 5 (HyperAgents convergence) is solid. Contribution 6 (recursive meta-evidence) is interesting. Contributions 3 (accidental control) and 4 (compilation threshold) are suggestive but rest on thin data. Reframe 3 as an observation, not a contribution. Keep 4 but add explicit caveats about sample size.

3. **Add a "Design Implications" subsection to Discussion.** Lay out what a correction-aware DX tool should do differently given these findings. Distinguish three tool behaviors: (a) track recurrence for process corrections, (b) surface compilation candidates at threshold, (c) skip tracking for architectural corrections (they're already self-enforcing). This takes 300-400 words and transforms the paper's practical contribution.

4. **Trim the theoretical framework to three core references.** Szulanski, Argyris, and one tacit-knowledge theorist. Move the rest to a paragraph in Discussion or a footnote. The paper is stronger when it goes deep on fewer frameworks than wide on many.

5. **Hedge the causal claims in Discussion.** Change "stickiness is not solely a human friction problem" to "our findings are consistent with stickiness having an architectural component that persists independent of social dynamics." Change "the surviving mechanism is architectural, not social" to "the mechanisms that survive the removal of social barriers are architectural." Same content, defensible phrasing.

6. **Drop the dollar amount from the abstract and conclusion.** Keep one mention in Section 3.

7. **Be explicit about what "approximately 50% persistence" means.** Is this 50% of process correction types that show at least one recurrence? 50% of individual process correction instances that recur? The number is load-bearing and currently ambiguous.

---

## 5. One Thing the Author Probably Hasn't Considered

**The compilation threshold might be a property of the developer, not the system.**

You present the 3-4 recurrence threshold as a property of the correction dynamics. But there's an alternative explanation rooted in how developers learn and make decisions. Cognitive science research on pattern recognition suggests that humans typically need 3-5 instances of a pattern before they recognize it as a pattern rather than a coincidence (see Kahneman's own work on the "law of small numbers," ironically). The threshold you're observing might not be the point at which "process corrections fail visibly enough to trigger architectural intervention." It might be the point at which *you, Spencer Hardwick*, recognize that you're seeing a pattern.

This matters because it changes the design implication. If the threshold is systemic (a property of how process corrections decay), then a tool should surface compilation candidates at 3 recurrences for every developer. If the threshold is cognitive (a property of how humans recognize patterns in their own correction data), then the threshold will vary by developer experience, domain familiarity, and individual pattern-recognition style. The tool should let each developer set their own threshold, or learn it from their behavior.

You could address this in one sentence in Limitations: "The compilation threshold may reflect the researcher's pattern-recognition latency rather than a systemic property of correction dynamics." But the deeper point is that your DX tool design should probably treat the threshold as a learnable parameter, not a constant. That makes the tool more interesting, not less.

This connects to something I've seen repeatedly in my research on how developers learn programming tools: the threshold at which someone decides "I need to automate this" versus "I'll just remember to do it manually" is highly individual. Some developers automate after the second failure. Some never automate. The fact that you observe consistency at 3-4 may say more about your personal automation instinct than about the correction dynamics themselves. A multi-researcher replication would distinguish these, which reinforces your own Future Work priority.

---

## Summary Verdict

The core phenomenon is real and publishable. The two-type distinction is a genuine contribution. The paper needs methodological grounding in autoethnographic traditions, pruning of the theoretical apparatus, stronger design implications, and more careful hedging of causal claims. With those fixes, this is a solid CHI or CSCW contribution. Without them, it will get desk-rejected on methodology by reviewers who see N=1 without a framework for why N=1 is appropriate here. The autoethnographic framing is the single highest-leverage fix.

Recommendation: **Revise and resubmit** (to yourself, before sending to a venue). The bones are strong. The methodology section and the theoretical economy need work.
