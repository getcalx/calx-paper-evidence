# Pre-Submission Review: "Stickiness Without Resistance"
## Reviewer: Richard Thaler
## Date: 2026-04-01

---

## 1. Overall Assessment

This is a genuinely interesting paper that identifies a real phenomenon and locates it within the right theoretical neighborhood. The core finding, that knowledge transfer stickiness survives the removal of motivational barriers, is clean and worth publishing. The paper's weakness is not the data or the phenomenon but the precision of the theoretical claims, particularly around choice architecture. Some of the framework borrowing is load-bearing and some is decorative. The author needs to know the difference before a reviewer less sympathetic to the project points it out less gently.

---

## 2. What Works

**The Szulanski extension is the best move in the paper.** Taking a 30-year-old framework, removing two of its four barriers by construction, and showing the phenomenon persists anyway is exactly how to make a contribution. The mapping is clean: motivation eliminated, relationship quality eliminated, causal ambiguity survives, absorptive capacity survives. That is a well-structured natural experiment. The paper earns the right to cite Szulanski because it does something with the framework rather than just borrowing vocabulary.

**The two-type distinction is genuinely useful.** The architectural/process split is not just a classification scheme. It is an actionable design variable. That is the test of a good behavioral distinction: can someone use it to make a decision? Yes. If you know a correction is process-type, you know it will degrade. If you know it is architectural, you know it will hold. That is a practical contribution independent of how the theory shakes out.

**The eight-entry mock patching chain is worth the price of admission.** Eight corrections, each acknowledging the previous ones, each failing to prevent the next occurrence. That is not an anecdote. That is a behavioral pattern with a clear structure. I would move this into a figure or a table if it is not already. It is the single most persuasive piece of evidence in the paper.

**The recursive meta-evidence section (5.8) is handled with the right level of modesty.** The author could have oversold the "building the correction tool produced corrections about correction tracking" result. Instead, the framing is restrained: it is a methodological note about robustness, not a separate claim. Good discipline.

**The limitations section is unusually honest.** Six specific threats, each stated as an invitation to replicate. This is how limitations should read: not as ritual self-flagellation but as a research agenda for others. The single-rater limitation on classification (Section 7.6) is the most important one and the author correctly identifies it.

---

## 3. What Doesn't Work or Needs Strengthening

**Section 6.2: The choice architecture claim is half right and half borrowed vocabulary.**

The paper says: "Corrections are evidence that the default environment is producing errors. A process rule attempts to change the agent's choice within the existing environment. A compilation event redesigns the defaults."

The first sentence is correct and interesting. Corrections ARE signals that the default behavioral path is wrong. That is genuine choice architecture reasoning: diagnose the default, measure what happens when people follow it, redesign accordingly. The correction log is functioning as a sludge audit for agent behavior, which is a framing the paper should lean into harder.

The second sentence is where it starts to slip. A process rule does not "attempt to change the agent's choice within the existing environment." It attempts to change the agent's behavior through instruction. In choice architecture, the distinction matters enormously. A nudge restructures the environment so that the desired behavior becomes the path of least resistance while preserving the option to deviate. A process rule is not a nudge. It is a mandate delivered through text. The agent is not choosing to follow the rule. It is instructed to follow the rule and sometimes fails to execute the instruction. That is not a choice architecture problem. That is a compliance problem, closer to what I would call a "channel factor" issue (Lewin, via Ross and Nisbett): the rule exists but the channel through which it must flow to produce behavior is unreliable.

The third sentence, "Compilation removes the possibility of choosing wrong," is the most problematic. If compilation removes the possibility of choosing at all, it is not choice architecture. It is a constraint. Choice architecture preserves the ability to choose otherwise. Auto-enrollment in a 401(k) is choice architecture because the employee CAN opt out. A pre-commit hook that blocks untested code is not choice architecture. It is a hard constraint. The agent cannot choose to skip the test. There is no libertarian component. You have removed the choice entirely.

This matters because the paper is positioning compilation as a nudge-level intervention. It is not. It is a mandate implemented through infrastructure. That can be a perfectly good intervention, but calling it choice architecture borrows theoretical credibility the mechanism has not earned. The paper should acknowledge that compilation sits on a spectrum: some compiled mechanisms are genuine choice architecture (smart defaults that agents can override) and some are hard constraints (hooks that cannot be bypassed). The paper currently treats them as one category.

**Section 6.3: "Manufacturing kind environments" overreaches.**

The Epstein kind/wicked distinction is deployed correctly at the descriptive level: process corrections do operate in a wicked environment and architectural corrections do create kind environments. But the paper then claims that "the compilation pipeline converts the learning environment itself from wicked to kind." This is too strong. The compilation pipeline converts one specific error class's environment from wicked to kind. The overall learning environment remains wicked. The agent still encounters novel errors in novel contexts. Each compilation event creates a local pocket of kindness in a globally wicked landscape. The paper should say this. The current framing implies that enough compilation eventually makes the whole environment kind, which is not what the data shows. The error floor at 2.9 corrections per specification is evidence that the environment remains wicked despite accumulated compilation.

**The "accidental control group" (Section 5.3) needs a harder interrogation of the alternative explanation.**

The paper acknowledges that the simplest alternative is that those four domains were simply unused. But it does not push hard enough on what this means for the claim. If the domains were unused, the finding is trivially true: rules without work produce no corrections because there is no work to generate corrections. That is not evidence that "instruction without the correction cycle produces no learning." That is evidence that learning requires activity. The paper needs to either: (a) find evidence that agents did interact with those domains in some capacity and still showed no behavioral change, or (b) downgrade the control group language significantly. Calling it a "control group," even with "accidental" as a modifier, implies a comparison that the data does not support if the domains were genuinely dormant. An alternative framing: "We observe that rules exist in four domains with zero activity. This is consistent with the hypothesis that instruction alone is insufficient, but does not test it. A designed experiment would assign identical tasks to agents with and without transferred rules."

**The Kahneman dual-process mapping in Section 6.2 is too neat.**

Saying process corrections are System 2 and architectural corrections are System 1 analogs is elegant but misleading. System 1 operates through learned associations and heuristics. A pre-commit hook is not a heuristic. It is an external constraint on the environment. Calling it a "System 1 analog" suggests it works like automatic cognition. It does not. It works like a locked door. You do not need automatic cognition to be stopped by a locked door. The analogy should be flagged as loose or dropped. The paper is stronger without it.

---

## 4. Suggested Fixes

1. **Reframe the choice architecture paragraph (Section 6.2).** Distinguish between compiled mechanisms that are genuine choice architecture (defaults that agents can override, friction that slows but does not prevent) and compiled mechanisms that are hard constraints (hooks that block, type systems that reject). Acknowledge that much of what the paper calls "compilation" is closer to regulation than to nudging. The interesting question is whether some compilation events CAN be designed as nudges rather than mandates, and the paper does not ask this question.

2. **Add a "sludge audit" framing.** The correction log is functioning as a sludge audit: it identifies places where the default environment produces errors that require costly intervention. This is legitimate choice architecture reasoning and the paper underplays it. The correction capture process IS the diagnostic tool that a choice architect would use. Frame the correction log as the audit and the compilation pipeline as the redesign. This is more accurate than the current "corrections are evidence that defaults are wrong" framing because it names the methodology, not just the observation.

3. **Scope the kind environment claim.** Change "converts the learning environment from wicked to kind" to "converts specific error classes from wicked to kind learning environments, while the system-level environment remains wicked." Support with the error floor data: 2.9 corrections per specification persists despite compilation, which means the overall environment remains wicked enough to produce errors at a steady rate.

4. **Downgrade the accidental control group.** Change "accidental control group" to "accidental quasi-control observation" or similar. Be explicit that you cannot distinguish "rules without correction cycles are inert" from "inactive domains produce no corrections because no work occurs." The first is an interesting finding about learning. The second is a tautology. Your data does not distinguish between them. State this clearly and make the case for the designed experiment that would.

5. **Drop or heavily qualify the System 1/System 2 mapping.** The dual-process framework is a theory about cognition. Architectural corrections are environmental constraints. These are different things. If you want the Kahneman reference, use it for the process correction side only: "Process corrections require deliberative application, consistent with the fragility of System 2 processing under cognitive load." Do not extend the analogy to architectural corrections. They do not need a cognitive framing because they bypass cognition entirely. That is their strength.

6. **Add one sentence to the compilation threshold discussion (Section 5.4) connecting it to my SMarT work.** The compilation threshold is the point at which the cost of continued process correction exceeds the cost of architectural intervention. This is the same economic logic as Save More Tomorrow: the intervention works because it is applied at the moment when the cost-benefit ratio has shifted. Below the threshold, process correction is cheaper. Above it, compilation is cheaper. Name this. It connects your threshold to a literature that reviewers in behavioral economics will recognize immediately.

---

## 5. One Thing the Author Probably Hasn't Considered

The paper treats the compilation pipeline as a response to failure: corrections accumulate, process corrections fail, compilation converts the fragile type to the durable type. The framing is reactive. Errors happen, then the system adapts.

But there is a proactive version of this that the paper does not explore. In my work on defaults, the most powerful finding is not that defaults can be redesigned after problems emerge. It is that defaults can be DESIGNED well from the beginning when you know the behavioral patterns in advance. The paper has generated a dataset of 151 corrections with a clean two-type taxonomy. That dataset is itself a design resource. If you know that process corrections in "test mock patching" will recur 8 times before anyone compiles them, the rational move is to pre-compile: to design the architectural constraint before the first correction occurs, based on the pattern observed in this study.

The paper's data enables predictive compilation. You know which domains produce process corrections. You know the approximate recurrence rate. You know the compilation threshold. You have enough information to identify candidate compilation targets prospectively, before the correction cycle begins, using the frequency and type distributions from this dataset as base rates. This turns the correction log from a reactive audit into a predictive design tool.

This is where the choice architecture framing becomes genuinely interesting rather than borrowed. If you can predict which domains will produce process corrections and pre-install architectural mechanisms, you are designing the choice environment before the agent enters it. That IS choice architecture. That is designing the cafeteria before lunch, not rearranging the food after everyone has already made bad choices. The paper currently describes the rearrangement. The dataset enables the design. I would add a paragraph in the Discussion or Future Work naming this possibility. It is the strongest connection between your data and my framework, and you are leaving it on the table.

---

*Review prepared for pre-submission improvement. The paper is publishable with revisions. The core phenomenon is real and the Szulanski extension is a genuine contribution. Tighten the theoretical claims, downgrade the borrowed vocabulary to earned vocabulary, and this holds up.*
