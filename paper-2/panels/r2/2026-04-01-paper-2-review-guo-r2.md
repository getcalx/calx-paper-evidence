# Re-Review: "Stickiness Without Resistance" (Revised Draft)

**Reviewer:** Philip Guo (UC San Diego, CSE)
**Lens:** Developer experience, HCI/CSCW research methodology
**Date:** 2026-04-01
**Review round:** R2 (responding to revisions from R1 feedback)
**Target venues:** CHI, CSCW

---

## 1. Overall Assessment

The revision addressed four of my six major concerns. The paper is materially better. The autoethnographic framing is present. The theoretical apparatus is leaner. The accidental control group is demoted. The compilation threshold is hedged. Two issues remain unresolved: the design implications section I asked for does not exist, and the causal language in Discussion is still too strong in places. A third issue has emerged: the new ecological prevalence section (6.7) is doing real work but creates a tonal mismatch that needs careful handling for venue fit.

This is now closer to submittable than not. I would estimate one more focused revision pass, primarily on the Discussion section, before it is ready for external review at CHI or CSCW.

---

## 2. What the Revision Got Right

**The autoethnographic framing is adequate for venue.** Section 4.1 now names the method, cites Ellis and Bochner (2000) and Adams et al. (2015), and makes the explicit trade: external validity for ecological validity. The sentence "The approach trades external validity (a single practitioner's experience) for ecological validity (the corrections emerge from genuine production work, not experimental tasks)" is exactly the framing a reviewer needs to see. You then list the compensating measures (structured capture, prospective classification, public artifacts). This is sufficient. A CHI reviewer who knows autoethnographic methods will recognize the tradition. A reviewer who does not will at least see a named methodology with cited precedent and articulated tradeoffs.

Two refinements would strengthen it further. First, the Ellis and Bochner citation is the canonical reference for autoethnography as a qualitative method, but CHI reviewers respond more to HCI-specific precedents. Consider adding Desjardins and Ball (2018) on autobiographical design or Lucero (2018) on first-person methods in interaction design research. These are closer to what you are doing (a designer/developer systematically studying their own practice with a technology system) than the sociological tradition Ellis and Bochner represent. Second, the autoethnographic framing appears only in 4.1 and is never referenced again. The Discussion should connect back to it at least once, perhaps in 6.6 (Interaction Orientation), to remind reviewers that the methodology was chosen deliberately for this interaction regime.

**Trimming the frameworks helped significantly.** Nonaka and Takeuchi are gone. Kahneman is gone. The extended Thaler choice architecture mapping is gone. What remains is Szulanski (core framing), Argyris (loop distinction), Zollo and Winter (codification threshold), Polanyi (tacit knowledge), Lave and Wenger (situated learning), and Cohen and Levinthal (absorptive capacity). That is still six, which is more than the three I suggested, but the paper is long enough and the mappings are tight enough that six works. Each one does analytical work rather than decorative work. The Lave and Wenger mapping to the accidental control group is now proportionate to the evidence. The Cohen and Levinthal extension (absorptive capacity as correction history, not just domain knowledge) is the paper's most interesting theoretical move and it deserves the space it gets.

The Epstein "kind learning environment" framing in 6.3 survives, and I am glad it does. On re-reading, this section does more work than I initially credited. Establishing that the domain is relatively kind, and that stickiness persists anyway, is an important scope claim. The reframing as "context-variance within a kind domain" rather than the original "manufacturing kindness" is tighter and more defensible.

**The accidental control group is properly demoted.** It is now Contribution 3 but framed as "a suggestive observation" with the explicit caveat "This is not a designed control condition; we present it as a pattern warranting designed replication." Section 5.3 is three paragraphs and ends with "We cannot rule out the simplest alternative: that those domains were simply unused." This is the right weight. A reviewer can take it or leave it without undermining the core claims.

**The compilation threshold is properly hedged.** Section 5.4 now includes the circularity acknowledgment I was hoping for: "We note a circularity risk: the system was designed with this threshold, so observing transitions near it may reflect a design parameter rather than an emergent property." The language "preliminary observation from few data points" and "testable hypothesis for future work" is appropriately cautious. Contribution 4 is labeled "a preliminary compilation pattern." This will not draw fire from reviewers. It is presented as a hypothesis, not a finding.

My R1 concern about the compilation threshold being a property of the developer rather than the system: the paper addresses this partially in 7.1 (Single Researcher) but never engages the specific cognitive mechanism I raised. The threshold could reflect Spencer Hardwick's pattern-recognition latency (humans need 3-5 instances to recognize a pattern as non-coincidental) rather than a systemic property of correction dynamics. One sentence in 5.4 or 7.1 would close this: "The threshold may also reflect the researcher's pattern-recognition latency: cognitive science suggests humans typically require 3-5 instances before recognizing a recurring pattern, and the observed threshold falls within this range." This matters because it changes the design implication from "set the threshold at 3" to "learn the threshold per developer."

---

## 3. What Still Needs Work

**The design implications section is missing.** This was Suggested Fix 3 in my R1 review, and it is the revision's most significant omission. The paper identifies a phenomenon with direct tooling implications and then stops short of spelling them out. Section 8 (Future Work) mentions an "automated compilation pipeline" in one paragraph, but this is a research direction, not a design implication.

What I am asking for is 300-400 words in Discussion (a new subsection 6.8, or folded into 6.2) that translates the findings into concrete design guidance for someone building a correction-aware developer tool. Three behaviors, mapped to the evidence:

1. For architectural corrections: no tracking needed. They are self-enforcing. A tool that tracks them adds noise without value.
2. For process corrections below the compilation threshold: track recurrence, surface the correction at the point of relevant action, reinforce through repetition.
3. For process corrections at or above the threshold: surface the pattern as a compilation candidate. Flag it for the developer as "this correction has recurred N times; consider an architectural intervention."

This is the bridge between "here is a phenomenon" and "here is what a practitioner should do about it." CHI reviewers in particular want to see design implications in papers that study systems. Without them, the paper reads as pure theory, which is fine for Organization Science but undersells the contribution at a systems venue. You have the evidence. You have the taxonomy. State the design consequence.

**The causal language in Discussion is still too strong in two places.** Section 6.1, final paragraph: "This extends Szulanski's framework: stickiness is not solely a human friction problem. It has an architectural component that persists independent of social dynamics." And Section 9 (Conclusion): "The surviving mechanism is architectural, not social." Both of these are causal claims that go beyond what the data supports. You have one developer and one model family. The stickiness you observe is consistent with an architectural mechanism, but you have not isolated it from developer-specific or model-specific confounds.

The fix is mechanical. Change "It has an architectural component that persists independent of social dynamics" to "Our findings are consistent with stickiness having an architectural component that persists when social dynamics are removed." Change "The surviving mechanism is architectural, not social" to "The mechanisms that survive the removal of social barriers are architectural in character." Same content, defensible phrasing. This is not hedging for its own sake. A reviewer who sees the strong version will question the methodology. A reviewer who sees the hedged version will accept the claim.

**The $450 cost figure persists in the abstract and conclusion.** My R1 review asked you to drop it from those locations and keep one mention in Section 3. It remains in all three. This reads as pitching accessibility rather than reporting a research contribution. The relevant cost for a CHI audience is the 43 days of researcher time. The dollar figure belongs in the system description, where it contextualizes the computational resources used. In the abstract and conclusion, it is a distraction.

---

## 4. The New Ecological Prevalence Section

Section 6.7 is new and I want to address it specifically because it is doing something interesting but creates a venue-fit tension.

What it does well: it establishes that the process-correction-failure phenomenon is not idiosyncratic to this study. The "70 distinct community reports" figure, the 0% compliance measurement, the 60% compliance measurement, and the practitioner convergence on "your CLAUDE.md is a suggestion, hooks make it law" are all compelling evidence that the phenomenon is ecologically real. The distinction between formal dataset and convergent support is clearly drawn.

The venue-fit problem: this section reads like a survey contribution grafted onto an autoethnographic study. The methodological register shifts from "I systematically documented my own practice" to "I audited practitioner discourse across six tools." These are different research activities with different validity claims. An autoethnographic study derives its strength from depth and reflexivity within one practice. A survey of practitioner discourse derives its strength from breadth and representativeness across practices. Combining them in one Discussion section without acknowledging the methodological shift will confuse reviewers about what kind of paper this is.

Three options, in order of my preference:

1. **Keep it but reframe.** Add one sentence at the top of 6.7 that explicitly names the methodological shift: "We step outside the autoethnographic frame to report ecological evidence that the phenomenon we formally characterize is widely experienced." Then at the end, return to the frame: "The formal contribution of this paper remains the mechanism analysis from our autoethnographic study. The ecological evidence establishes that the phenomenon warrants the attention we give it." This lets you keep the content while signaling to reviewers that you know what you are doing methodologically.

2. **Move it to Related Work.** Section 2 could include a subsection on practitioner evidence for instruction-adherence failure. This positions it as motivation rather than as a finding, which is arguably more honest since the systematic audit is forthcoming [Hardwick2026c].

3. **Cut it.** If the paper is already at or near page limits, this is the most expendable Discussion section. The formal findings stand without it.

I recommend option 1. The content strengthens the paper's impact, but it needs methodological framing to avoid confusing the submission's identity.

---

## 5. Structural and Presentational Notes

**The abstract is too long.** At roughly 350 words, it reads as a compressed introduction rather than an abstract. CHI abstracts run 150-250 words. Cut the HyperAgents convergence, the ecological validity argument, and the cost figure from the abstract. Keep: the phenomenon (stickiness persists without human friction), the key finding (two types with different transfer properties), the dataset (43 days, 8 agents, 151 corrections), and the scope claim (evidence for a phenomenon, not proof of generalizability). Everything else can be discovered by reading the paper.

**The contributions list has six items.** That is too many for a paper of this scope. Contributions 1 and 2 are the core. Contributions 3 and 4 are now properly hedged as suggestive/preliminary. Contribution 5 (HyperAgents convergence) and Contribution 6 (recursive meta-evidence) are interesting but arguably not "contributions" in the CHI sense (they do not represent new methods, systems, or empirical findings from this study). Consider collapsing to four: (1) stickiness persists, (2) two types with different transfer properties, (3) preliminary compilation pattern with caveats, (4) full public artifact trail enabling replication. Move HyperAgents convergence and recursive meta-evidence to Discussion observations rather than numbered contributions.

**Section 6.5 (Behavioral Plane as Dynamic State) reports a qualitative observation with no data.** "We present this as a qualitative observation, not a measured finding" is honest, but the section sits in Discussion alongside sections that are grounded in the formal dataset. Either add a concrete example (which session, which correction, what did the degradation look like) or move this to Future Work as a hypothesis. As written, it is a claim about recall degradation that the paper cannot support.

**The Mara-to-Thane case carries a lot of weight.** It appears in the abstract, the introduction (Section 1), the findings (Section 5.1), and the discussion (Section 6.1). It is your strongest cross-study evidence and it is a single case. The repetition is fine for emphasis, but make sure the description is identical each time. I noticed minor variations in how the timeline is described ("ten days later" in some places, specific dates in others). Pick one version and use it consistently.

---

## 6. Venue Recommendation

**CHI 2027** (deadline September 2026) is the better target. The autoethnographic framing, the system contribution angle, and the design implications (once you add them) all fit CHI's scope. CHI reviewers will want to see:

- Named methodology (done)
- Design implications (not done, add them)
- Reflexivity about the researcher's role (partially done in 4.1 and 7.1, could be stronger)
- Clear scope claims (done well throughout)

**CSCW** is also viable but the paper would need to lean harder into the collaborative dynamics angle (how does correction transfer failure affect team-level knowledge management in human-AI teams). Right now the paper is more about the individual practitioner's relationship with agents, which is CHI territory.

**Do not submit to a pure KM or org-psych venue** (e.g., Organization Science, Management Science) without substantially more data. Those communities will expect multi-site, multi-researcher studies with statistical power. The autoethnographic frame will not land there.

---

## 7. Summary Verdict

The revision addressed the structural problems from R1. The autoethnographic framing is adequate. The theoretical apparatus is proportionate. The weak contributions are properly hedged. Two items remain from my original list (design implications, causal hedging) and one new issue emerged (ecological prevalence section needs methodological framing). The abstract needs cutting. The cost figure needs relocating. Section 6.5 needs grounding or relocation.

None of these are hard to fix. The core phenomenon, the two-type distinction, and the Szulanski barrier analysis are the genuine contributions, and they are solid. The paper's argument is clear and the evidence, while limited in scope, is presented honestly.

Recommendation: **One more revision pass, focused on Discussion.** Add design implications (6.8 or folded into 6.2). Hedge the two causal sentences. Frame the ecological prevalence section methodologically. Cut the abstract to 200 words. Relocate the cost figure. After that, this is ready for external review.

Estimated distance from submission-ready: one focused session. The bones are strong and the flesh is now mostly in the right places.
