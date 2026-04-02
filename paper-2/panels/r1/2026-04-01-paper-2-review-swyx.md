# Pre-Submission Review: "Stickiness Without Resistance"
## Reviewer: swyx (Shawn Wang)
## Date: 2026-04-01
## Lens: Developer community reception, practitioner utility, ecosystem critique fairness

---

## 1. Overall Assessment

This paper names something real that every practitioner has felt but nobody has written down with rigor: your CLAUDE.md rules don't actually transfer between agents, and the distinction between "rules the agent reads" and "constraints the environment enforces" is the whole game. That naming function is valuable. But the paper is currently dressed in 30 years of organizational psychology theory to describe something a senior engineer could explain in five minutes at a whiteboard, and it risks alienating the exact audience (practitioners) who would benefit most from the finding.

---

## 2. What Works

**The two-type distinction is genuinely useful.** Architectural vs. process corrections is the kind of clean, actionable taxonomy that practitioners adopt immediately. Every developer who has written a linting rule instead of adding another bullet to a style guide already knows this intuitively, but having the names and the data makes it legible, discussable, and buildable. This is the core contribution. Protect it.

**The compilation threshold is a practitioner-ready metric.** "If you've corrected the same thing 3-4 times, stop writing rules and start writing code" is a decision heuristic that people can use tomorrow. It maps directly onto the lived experience of anyone maintaining agent instruction files. This is the paper's strongest bridge to the builder community.

**The recursive meta-evidence is compelling and honest.** 41% of corrections while building the correction system were about the correction system. That is the kind of detail that makes practitioners trust the author. It is the "I tried X and it broke because of Y" artifact that signals real work, not theoretical hand-waving. This is learning in public in the best sense.

**The HyperAgents convergence is well-handled.** You do not overclaim. You note the independent arrival, flag the differences in experimental setup, and let the reader draw conclusions. This is the right way to cite parallel work you discovered after the fact.

**The limitations section is unusually strong.** You state N=1, single model family, single rater, no designed control. Most papers bury their limitations. You lead with them. The practitioner community will respect this because practitioners hate papers that overstate generalizability from small studies. You do not do that.

---

## 3. What Doesn't Work or Needs Strengthening

**The paper is overtheorized for its audience.** You cite Szulanski, Argyris, Polanyi, Nonaka/Takeuchi, Lave/Wenger, Cohen/Levinthal, Zollo/Winter, Kahneman, Thaler, and Epstein. That is ten theoretical frameworks to explain one empirical finding. The practitioner reading this paper (the person maintaining a .cursor/rules directory or wrestling with CLAUDE.md files across a team) will bounce off the Related Work section. They do not need Nonaka's SECI model to understand that copying a rules file to a new agent does not make the new agent behave like the old one. They need the data, the taxonomy, and the practical implications.

If you are targeting an academic venue, the theory is appropriate. If you want practitioners to read this, you need to decide which three frameworks are load-bearing and cut the rest to footnotes. My candidates: Szulanski (the core finding you are extending), Argyris (single-loop vs. double-loop maps directly), and Epstein (kind vs. wicked environments explains the mechanism). Everything else is decorative.

**The MindMeld and Compound Engineering critiques are fair but underspecified.** You say MindMeld has an "implicit transferability assumption" and Compound Engineering has an "implicit sharing model." Both claims are correct. But you do not show specifically what breaks. A practitioner reading this wants to know: "I am using MindMeld (or a similar system). What specifically will fail? On what types of rules? In what conditions?" The paper gestures at this with the 13-of-44 statistic but does not build the bridge. You should show: here is a rule from MindMeld's "battle-tested invariants" category, here is how it would behave in our framework, here is why it works within-dyad and fails cross-dyad. Make the critique concrete enough that a MindMeld user can audit their own rule set.

The Compound Engineering critique is even lighter. You cite it once in the Related Work and never engage with it again. Dan Shipper's thesis is that the compound effect of engineering with agents comes from accumulating learnings across sessions. Your paper directly challenges the transferability of those accumulated learnings. That is a meaningful disagreement. You should engage it directly rather than treating it as a parenthetical citation.

**The "behavioral plane" vs. "information plane" distinction needs a practitioner translation.** These are good conceptual handles but they sound academic. The developer version of this is: "Memory is not behavior. Giving an agent access to a correction log is like giving a new hire the wiki page about the last production incident. They can read it. They cannot feel why it matters." You have this insight in the paper but it is buried in theoretical language. Pull it forward. Name it in terms practitioners already use.

**The paper does not engage with the most obvious practitioner objection.** Any experienced developer reading this will think: "Of course rules don't transfer. That is why we write tests, not comments. This is just the testing vs. documentation debate restated for agents." You need to acknowledge this explicitly and explain why the finding is not obvious. The answer is in the paper (it is that the entire agent memory ecosystem assumes transferability, and $24M+ of venture funding has been deployed on that assumption), but you never make that argument directly. The paper treats the finding as surprising. Practitioners will not find it surprising. You need to reframe: the finding is not surprising to practitioners, but the industry is building infrastructure that assumes the opposite, and nobody has produced the data to challenge it.

**Study 2 is thin.** 17 corrections, 5 active days, 3 agents. The Mara-to-Thane non-transfer is your strongest single example, but it is one example. You are honest about this, but the paper gives Study 2 equal billing with Study 1 (134 corrections, 20 days, 6 agents), and that creates a mismatch between the rhetorical weight and the evidential weight. Consider repositioning Study 2 as a supporting replication rather than a co-equal study.

**Missing: the cost story.** You mention $450 total compute cost. You do not make the argument that this matters. The practitioner audience cares deeply about this. The implicit question for anyone maintaining agent correction infrastructure at scale is: "What does it cost to re-derive corrections that could have been transferred?" Your data suggests that for every new agent, some fraction of the correction history must be re-earned through the correction cycle. That has a compute cost, a time cost, and a quality cost during the re-derivation period. Quantify this even roughly and you have a metric practitioners will cite.

---

## 4. Suggested Fixes

1. **Add a Practitioner Summary section** (1 page, before or after the Abstract) that states the finding, the taxonomy, and the practical implications in zero-jargon language. Target audience: the person who reads Latent Space, maintains a CLAUDE.md, and has 10 minutes. This is the section that gets screenshot-shared on Twitter. The rest of the paper can be as academic as you want.

2. **Trim Related Work by 40%.** Keep Szulanski, Argyris, Epstein, and the agent memory survey. Move Polanyi, Nonaka/Takeuchi, Lave/Wenger, and Zollo/Winter to a "Theoretical Extensions" appendix or compress each to one sentence in-line. The theoretical mapping is real but it is not the contribution. The data is the contribution.

3. **Build a concrete MindMeld example.** Take a real rule from your correction logs. Show it in the format MindMeld would store it. Walk through what happens when Agent B receives it without the correction history. Show the prediction your framework makes, then show the actual data. This single example will do more for practitioner adoption than the entire Nonaka section.

4. **Engage Compound Engineering directly in Discussion.** One paragraph. "Shipper (2026) argues that the compound effect of agent-assisted engineering comes from learning accumulation. Our data suggests this compounding is dyad-local: the learning compounds within the human-agent pair that earned it, but the accumulated rules do not carry their behavioral force to a new pair. The compound interest metaphor breaks at the transfer boundary."

5. **Add a cost-of-re-derivation estimate.** Even a rough one. If Faye needed 44 corrections to reach behavioral calibration despite having 237 transferred rules, and each correction costs approximately N minutes of developer time and M tokens of compute, the re-derivation cost per agent onboarding is calculable. This becomes the economic argument for compilation: compiled mechanisms avoid re-derivation costs entirely.

6. **Name the practitioner objection explicitly.** Add a paragraph in Section 6: "Experienced engineers will recognize this pattern: the software industry learned decades ago that tests are more reliable than comments, that type systems are more reliable than code review conventions, and that CI pipelines are more reliable than developer checklists. The contribution here is not the principle (which practitioners know intuitively) but the evidence that the agent tooling ecosystem has not yet absorbed it. Hundreds of millions of dollars in agent memory and agent governance infrastructure assumes that captured rules transfer behavioral force. Our data suggests they transfer information but not behavior."

---

## 5. One Thing the Author Probably Has Not Considered

The paper frames compilation as the answer to transfer failure. But the practitioner community is about to discover something your paper does not address: **compilation does not scale either, just differently.**

Here is the problem. Every architectural correction is a constraint added to the environment. A type check. A hook. A pre-commit gate. A structural refactor. Each one works perfectly in isolation. But architectural corrections accumulate. At 10 compiled constraints, the environment is well-governed. At 100, it is rigid. At 1,000, it is hostile to the developer. The compilation pipeline solves the transfer problem by moving corrections into the environment, but environments have a carrying capacity for constraints before they become impediments to velocity.

The developer community has already seen this movie. ESLint configs with 200 rules. Pre-commit hooks that take 45 seconds. Type systems so strict that adding a field requires touching 30 files. CI pipelines with 18 required checks. Each individual constraint was a reasonable compilation of a real correction. In aggregate, they create the opposite problem: a kind environment so over-constrained that innovation requires disabling the constraints.

Your paper should at least acknowledge this. The compilation threshold tells you when to compile. You need a corresponding metric for compilation load: when the accumulated compiled constraints begin to impede the system. This is the next research question, and naming it in the paper positions you to own it.

The developer community will find this paper useful if you meet them where they are. Right now, you are asking them to wade through organizational psychology to get to an insight they can feel in their bones. Lower the barrier. The finding is strong. The data is real. The naming is good. The theoretical packaging is the bottleneck.

---

*Reviewed as swyx, through the lens of developer community reception and practitioner utility. Direct assessment, pre-submission.*
