# Review (Round 2): "Stickiness Without Resistance" -- Taleb Lens

**Reviewer**: Nassim Nicholas Taleb (simulated)
**Date**: 2026-04-01
**Paper**: Revised draft of "Stickiness Without Resistance" by Spencer Hardwick
**Prior review**: 2026-04-01-paper-2-review-taleb.md

---

## 1. What Improved

The compilation threshold revision is the most important change and it was done correctly. Downgrading from "the compilation threshold" to "preliminary pattern" with the circularity acknowledged explicitly (the system was designed with a threshold of 3, so observing transitions near 3 may be a design artifact) is honest scholarship. This was the most dangerous claim in the first draft because it dressed a design parameter as an empirical finding. It is now presented as what it is: a testable hypothesis from few data points. Good.

The accidental control group demotion is adequate. Contribution 3 now reads: "a suggestive observation ... consistent with instruction without the correction cycle producing no learning. This is not a designed control condition; we present it as a pattern warranting designed replication." This is defensible. The first draft listed a near-tautology as a contribution. The revised draft lists a hypothesis. Acceptable.

The ecological prevalence section (6.7) is the revision's strongest addition. Seventy-plus practitioner reports across six tools, a 0% compliance measurement from one controlled test, a 60% compliance measurement from another, and the independent practitioner convergence on "your CLAUDE.md is a suggestion, hooks make it law." This does three things the first draft did not do: (a) it establishes that the phenomenon is not idiosyncratic to the author's system, (b) it provides external evidence from practitioners who have no stake in the paper's thesis, and (c) it connects to the transformer attention literature that provides a mechanistic explanation for why text-based rules degrade. This section earns its place. It is the closest thing in the paper to evidence of generality.

The interaction orientation scope condition (6.6 and 7.7) is a useful addition. Acknowledging that the phenomenon may only manifest when the human treats the agent as a persistent collaborator rather than a disposable tool is important. It bounds the claim. Most practitioners interact transactionally. The paper now says so.

---

## 2. What Remains Broken

### 2.1 "Approximately 50% Persistence" Survives

I flagged this explicitly. It appears in the abstract, in Section 1 (twice), in Section 3.1, in Section 5.2 (implicitly, through the chain data), in contribution 2, and in the conclusion. The revised draft did not change it.

Let me restate the problem precisely, because the author may have read my first review as stylistic objection rather than statistical objection.

You have five error classes that recur. Their chain lengths are 8, 5, 5, 4, and 3. From these you extract "approximately 50% persistence." I will be generous and assume you mean something like: of all opportunities for a process rule to be applied, roughly half the time it failed. But you do not know the denominator. You observe recurrences. You do not observe non-recurrences. You do not know how many sessions the agent operated without triggering the error. You do not know how many times the rule was successfully applied. You are computing a rate from the numerator alone.

This is not a minor issue. It is the difference between "process corrections fail sometimes, with chain lengths ranging from 3 to 8" (what you know) and "process corrections show approximately 50% persistence" (what you do not know). The first is an observation. The second is a statistic without a denominator. A statistic without a denominator is not a statistic. It is a number that looks like one.

"Approximately 50%" also carries a false precision that obscures the variance. You have chain lengths of 3, 4, 5, 5, and 8. The variance in this set is large relative to the mean. Reporting "approximately 50%" collapses a distribution with potential power-law properties into a single central-tendency estimate. This is exactly the kind of averaging that hides the structure of the data. The 8-entry chain is not "50% persistence." It is a qualitatively different animal from the 3-entry chain. Averaging them together destroys the most interesting signal in the dataset.

**Required fix**: Replace "approximately 50% persistence" everywhere with language that reports what you actually measured. "Process corrections recurred in every active error class, with chain lengths ranging from 3 to 8 entries over the observation period. The longest chain (test mock patching, 8 entries) spanned the full Study 1 timeline." This says the same thing without the false denominator.

### 2.2 The Fat-Tail Observation Was Not Incorporated

My first review identified the most important analytical point the paper misses: the error floor at 2.9 corrections per specification is not a floor. It is the visible surface of a fat-tailed process.

The revised draft added Section 5.6 with a reinterpretation paragraph: "the plateau need not represent stagnation ... the system is pushing into harder territory while maintaining the same correction rate." This is a Gaussian reinterpretation. It assumes the floor is stable and that the difficulty is increasing smoothly. This is exactly backwards from what the data shows.

Look at your own data. One error class (mock patching) consumed 8 entries. The next two consumed 5 each. Then 4 and 3. Your error classes follow a distribution where one class consumes more correction resources than multiple other classes combined. That is a power-law signature. In a power-law world, the "floor" of 2.9 corrections per specification is the median expectation, but the mean is dominated by rare events where a single specification generates 10 or 20 corrections because it touches an error class in the fat tail.

The practical consequence: someone reading this paper who builds a correction system and budgets for 2.9 corrections per specification will be surprised when specification number 47 generates 15 corrections because it touches an error class with the structural properties of the mock patching chain. The floor is a planning fallacy. The paper should say so.

Section 5.6 as written is the kind of comfortable reinterpretation that makes the data feel manageable. The uncomfortable interpretation, which the data actually supports, is that the correction rate per specification is drawn from a fat-tailed distribution, the floor represents the body of the distribution, and the tail risk is unbounded within the observation window. Eight entries is not the maximum. It is the maximum you observed in 43 days.

**Required fix**: Add one paragraph to Section 5.6 acknowledging the distributional shape. Something to the effect of: "The variance across error classes is large. The longest correction chain (8 entries) consumed more correction resources than several entire domains combined. This skewness is consistent with a heavy-tailed distribution in which the per-specification floor of 2.9 understates the expected cost of novel specifications that encounter error classes in the tail. We lack sufficient data to characterize the distribution formally, but the skewness is a caution against treating the floor as a reliable planning estimate."

### 2.3 The HyperAgents Convergence Is Still Oversold

The revised Section 5.5 is better. The sentence "We do not claim that HyperAgents validates our findings" is a step forward. But the framing still oversells.

The problem is structural. Contribution 5 reads: "Independent convergence with HyperAgents on the boundary between declarative and procedural behavioral learning, arrived at through different methods and different systems." This is listed as a contribution of the paper. What exactly is the contribution? That someone else found something similar? That is not your contribution. That is their work.

What you can legitimately claim: "The distinction we identify between text-based and mechanism-based behavioral learning is consistent with independent findings from HyperAgents, which provides complementary evidence from a controlled agent-only setting." That is a corroboration note. It belongs in the discussion. It is not a standalone contribution.

The convergence is real. It is also on something that practitioners already know. The 70+ practitioner reports in your new Section 6.7 document the same boundary. Gawande documented it in medicine. Every engineer who has replaced a code review checklist with an automated linter has lived it. Two academic groups finding the same thing practitioners already know is corroboration, not discovery.

**Suggested fix**: Fold contribution 5 into the discussion of contribution 2. The convergence strengthens the claim that the two-type distinction is real. It does not constitute a separate contribution.

### 2.4 The Szulanski Barrier Removal Is Still Not Clean

The paper continues to assert that motivation and relationship quality are "eliminated by design" and "absent by construction." My first review noted that replacing human friction with computational architecture does not eliminate friction. It substitutes different friction.

The revised draft did not address this.

The specific problem: when Thane fails to apply Mara's rule despite it being "available in shared memory," the paper attributes this to the experiential/relational properties of correction knowledge. But the failure could be: retrieval failure (the shared memory system did not surface the rule at the relevant moment), attention competition (the rule was present but lost among competing instructions), prompt ordering effects (the rule was positioned where the model's attention is weakest), or context window overflow. These are not "zero friction." They are computational friction. They are the machine equivalent of Szulanski's "arduous relationship": structural properties of the transfer channel that impede knowledge movement.

The paper claims to have isolated causal ambiguity and absorptive capacity by removing motivation and relationship quality. But if computational friction is doing some of the work, the isolation is not clean. You may have replaced two human barriers with one or more computational barriers and then attributed all residual stickiness to the two surviving Szulanski barriers. The attribution could be partially correct and still significantly incomplete.

**Required fix**: Add a paragraph in Section 6.1 acknowledging computational friction as an alternative or co-occurring mechanism. Something like: "We note that the transfer channel in human-AI systems introduces its own friction: retrieval mechanisms, attention distribution, prompt ordering, and context window limits. These computational barriers are structurally different from Szulanski's relationship quality barrier but may play a similar functional role. Our study design does not isolate the contribution of computational friction from causal ambiguity and absorptive capacity. The clean removal of two Szulanski barriers does not guarantee that no new barriers were introduced."

### 2.5 "Behavioral Change" for Stateless Systems Remains Undefined

My first review asked the paper to grapple with what "behavioral change" means for a system that does not learn between sessions. The revised draft did not add this.

An LLM does not have behavior in the psychological sense. It has conditional output distributions shaped by its context window at inference time. When you say "behavioral change," you mean: a change in the instructions injected into the context window that produces a change in the output distribution. When you say "the behavioral change did not transfer," you mean: the instruction was present but did not produce the expected output distribution shift.

This matters because the entire theoretical apparatus (Argyris, Lave and Wenger, Polanyi, Szulanski) was developed for systems with persistent internal states that change through learning. LLMs have no persistent internal states between sessions. Every "behavioral change" is mediated entirely by what appears in the context window. The paper borrows thirty years of organizational psychology vocabulary and applies it to a fundamentally different computational substrate without ever justifying the mapping.

The mapping may be justified. Context-mediated behavioral change in LLMs may be functionally equivalent to learning-mediated behavioral change in humans for the purposes of transfer dynamics. But this is a claim that needs to be stated and defended, not assumed.

**Required fix**: One paragraph, probably in Section 1 or Section 4, stating: "We use 'behavioral change' throughout to mean a change in the agent's output distribution conditional on modified context. LLMs do not learn between sessions. All behavioral change in our system is mediated by context injection: corrections captured in one session are written to persistent storage and injected into future sessions via structural hooks. The agent does not remember the correction. It reads it. This means that what we call 'transfer failure' is specifically the failure of a written artifact to produce the same output distribution shift in a new agent that the original correction produced in the originating agent. We adopt the organizational learning vocabulary (Argyris, Szulanski, Lave and Wenger) because the transfer dynamics are structurally parallel, while acknowledging that the underlying mechanism is context-mediated rather than learning-mediated."

---

## 3. Severity Assessment

| Issue | Severity | Status |
|---|---|---|
| Compilation threshold as design parameter | Critical | **Fixed** |
| Accidental control group as tautology | High | **Fixed** |
| "50% persistence" as false statistic | Critical | **Not fixed** |
| Fat-tail signature in error distribution | High | **Not incorporated** |
| HyperAgents convergence oversold | Moderate | **Partially fixed** |
| Szulanski barrier removal not clean | High | **Not addressed** |
| "Behavioral change" undefined for LLMs | High | **Not addressed** |
| Ecological prevalence missing | High | **Fixed (new section)** |
| Interaction orientation as scope condition | Moderate | **Fixed (new section)** |

Two of six original issues were fixed well. One was partially fixed. Three were not addressed. One new issue (fat-tail) was raised and not incorporated.

---

## 4. The Paper's Actual Contribution, Stated Precisely

Strip away the quantitative language the data cannot support and the theoretical apparatus that has not been justified for this substrate, and here is what this paper actually contributes:

1. A documented observation that transferring correction-derived rules from one AI agent to another fails to prevent re-correction in the receiving agent, even when the rules are present and mechanically followed. This is the core finding. It is supported by the data.

2. A clean binary between corrections that change the environment (zero recurrence) and corrections that inform the agent (variable recurrence, chain lengths 3 to 8). This is supported by the data and validated by prospective classification across 10 of 11 error classes.

3. A preliminary observation from three correction chains that process corrections cluster around 3 to 4 recurrences before being identified as needing structural intervention, presented as a testable hypothesis. This is now correctly scoped.

4. Ecological evidence from 70+ practitioner reports that the phenomenon is widely experienced across the AI tooling ecosystem. This is the revision's strongest addition.

5. A recursive meta-observation that the phenomenon persists when the researcher is maximally aware of it.

That is a solid set of contributions for a field study. The paper damages itself by reaching beyond them. The "50% persistence" creates the impression of measurement where there is observation. The Szulanski mapping claims cleaner barrier removal than the study design achieves. The HyperAgents convergence is listed as a contribution when it is a corroboration. The theoretical vocabulary is borrowed without justifying the substrate transfer.

---

## 5. Verdict

The paper improved on its two most critical structural problems (compilation threshold, control group). It added substantial ecological evidence that strengthens the generality argument. It correctly scoped the interaction orientation as a boundary condition.

It did not fix the false statistic ("50% persistence"), did not incorporate the fat-tail observation, did not address the computational friction confound in the Szulanski mapping, and did not define "behavioral change" for a stateless system.

The paper is closer to publishable. It is not there yet. The remaining fixes are tractable: four paragraphs of honest language replacing four instances of overreach. The phenomenon is real. The ecological evidence is compelling. The two-type distinction has both empirical support and independent convergence. Stop dressing the observations in language that implies more precision than the data provides, and you have a paper that says something true about something that matters.

Fix the four items marked "required" above and the paper can go out. Leave "approximately 50% persistence" in the abstract and you are publishing a number that does not mean what readers will think it means. That is the kind of thing that gets quoted, gets challenged, and undermines the real contributions when it falls apart under scrutiny.

---

*The phenomenon is real. The quantification is still premature in places. Four fixes separate this draft from a defensible paper.*
