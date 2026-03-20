# Root Orchestrator (Reid) — Lessons Log

> Append-only. Each correction or learning gets logged here.
> Periodically distill patterns into rules.md.

## 2026-03-15 — Coordination Layer Collapsed Under Build Velocity

**Correction:** Spencer had to remind all three agents (Reid, Bryce, Faye) 3-4 times in a single day what had been done, what was where, and who owned what. No agent had a shared source of truth. Specs are in a flat directory with no index, no status, no traceability. Memory files are stale. The Linear migration never happened. Spencer became the context router between his own agents, which is the exact opposite of what the system is for.

**Context:** The pivot sprint (March 10-15) moved faster than the organizational layer. 18 specs in one directory, PRDs in another, design spec elsewhere, backend specs directory empty, correction files mixed with specs, no way to tell what's built vs in progress vs dead. Every new session started cold because no persistent artifact tracked project state across agents.

**Takeaway:** Build velocity without coordination velocity creates MORE cognitive load on Spencer, not less. Before the next sprint: one index file readable by all agents, updated as work ships. Not memory, not Linear, not scattered headers. One file. Five columns: what, where, status, owner, last updated. Reid owns keeping it current. This is non-negotiable infrastructure.

## 2026-03-15 — Raw Persona Agent Output Must Be Saved Verbatim

**Correction:** Spencer had to explicitly ask for the raw Vivian design review output to be saved. Reid had created a processed correction prompt for Faye but did not save Vivian's actual output as its own artifact. Spencer wants the unedited agent perspective preserved every time — not just the reformatted action doc.

**Context:** Vivian ran a full landing page design review against Meridian spec. Reid immediately reformatted the output into a correction list for Faye (~/brain/docs/agents/prompts/active/2026-03-15-faye-vivian-corrections.md) but didn't save the raw review. Spencer caught it and said "every time we do something like this, the entire output from the persona agent is captured in a doc, verbatim, no editing."

**Takeaway:** After every persona agent dispatch (Vivian, Prism thinkers, foil reviewers, etc.), the FIRST action is saving the raw verbatim output to the appropriate directory (evaluations for reviews, agents/prompts for session work). The processed/reformatted action doc is a SECOND, separate step. The raw output is the primary artifact — it preserves the agent's full reasoning, not just the distilled fixes.

## 2026-03-17 — Spec Boundary Problem: BE Specs Without FE Specs

**Correction:** Spec 34 (Tier 2 classification) was scoped as backend-only. It shipped the classification engine and was reviewed by backend foils. But the PRD (§4.2) defines a full dual-translation UX for Tier 2 users. No FE spec was ever written for the Tier 2 experience. Result: 40-65% of expected users get a 3-line handler instead of the full experience described in the PRD.

**Context:** The coordination layer tracked BE spec completion but didn't systematically derive FE specs from PRD sections that had BE dependencies. This is the same pattern that explains most anonymous flow gaps — BE work ships, FE spec for the UX implications never gets created.

**Takeaway:** After every BE spec is locked, explicitly check: "Does this spec's output change what the user sees?" If yes, create an FE spec before marking the feature as "done" in coordination. Reid owns this check.

## 2026-03-17 — Document Contradictions Accumulate Silently

**Correction:** Full-system audit found 14 contradictions between strategy.md, the codebase, and the PRD. strategy.md still listed STAR stories as Pro-gated, referenced "Oxidized palette," and had stale entry points — all overridden by Spencer's corrections months ago. Footer said "Soaria, Inc." CTA showed [redacted-email].

**Context:** Corrections happen in conversation. Strategy docs don't get updated in the same session. Over time, the source-of-truth document drifts from reality.

**Takeaway:** When a decision overrides a strategy/product doc, update that doc in the same session. Never defer doc updates. Reid must audit strategy.md and PRD alignment at least monthly.

## 2026-03-17 — Code Line Numbers Are Volatile References

**Correction:** Audit files referenced specific line numbers (CaptureFlow.tsx:413, Footer.tsx:17, etc.) from agent reads. These will shift with any development work, creating stale pointers.

**Context:** Spencer flagged this: "before any work is done in any of those places, whoever goes into the codebase and verifies... if we do literally any amount of development it's going to shift those."

**Takeaway:** Reference code by component name + function/section or by search pattern (grep-able string), not by line number. Line numbers are useful for point-in-time verification but should not be persisted in coordination docs or issue trackers.

## 2026-03-19 — Having a Memory File Does Not Mean Applying It

**Correction:** Spencer corrected me for the 15th+ time on "rules don't transfer." I had the memory file (`feedback-rules-dont-transfer.md`), it was well-written with the full reasoning, and I STILL told Spencer that Bryce and Faye starting new Calx sessions "don't start from zero" because their ACE lessons carry forward. This directly contradicts the core ablation finding — the exact thesis of the company.

**Context:** Spencer asked whether fresh sessions for Calx made sense. I confidently said the lessons/rules carry forward. When challenged, I hedged ("partially right"). Spencer was justifiably furious — this isn't an edge case, it's the core thesis.

**Takeaway:** Memory exists to be APPLIED, not just stored. When making claims about how the system works, actively check constraints before speaking — especially on topics corrected before. Structural fix: core thesis constraints now live in summary.md, injected at session start. But structure alone doesn't fix this — the habit of verifying before asserting is the real fix.

## 2026-03-19 — Flat Memory Index Creates Equal-Weight Illusion

**Correction:** 106 memory files in a flat index. "Rules don't transfer" (core company thesis) sat at the same visual weight as "specs live in ~/PycharmProjects/soaria-backend/specs/". Nothing signaled which memories are load-bearing and which are operational details.

**Context:** Spencer suggested the summary.md pattern from the frontend/backend codebases — a living doc that reflects current state and points to things you need to reference. Not a flat list, but a prioritized orientation.

**Takeaway:** Memory needs hierarchy. A living summary (updated every session) carries the high notes — current state, core constraints, active workstreams, locked decisions. The index organizes by topic with critical items marked. Individual files hold detail. Three tiers, not one flat list.

## 2026-03-19 — Static Hooks Go Stale

**Correction:** The session-orient.sh hook had hardcoded behavioral bullets that hadn't been updated as the project evolved. It injected a truncated (head -25) priorities file.

**Context:** Restructured to inject summary.md instead — a living doc that gets updated as state changes.

**Takeaway:** Hooks should point to living docs, not contain hardcoded content. The hook's job is to inject the right file, not to BE the content.

## 2026-03-20 — Obsidian MCP Reveals Structural Debt Invisible to File Trees

**Observation:** First full vault exploration via Obsidian MCP surfaced five structural problems in minutes that months of file-tree navigation never caught:

1. **No `#calx` tag exists.** Everything — including the Calx ecosystem thesis — is still tagged `#versed`. The ontology hasn't caught up with the pivot.
2. **The ecosystem thesis is a graph island.** Zero wikilinks in or out. The most important strategic doc floats disconnected from the evidence, evaluations, and PRDs it references.
3. **Agent lesson files have zero graph connections.** The correction data that proves the paper's thesis isn't linked to anything structurally — only discoverable by content search.
4. **Hex color codes leaked into tags.** Brand palette values (`#A0785A`, `#C4654A`, etc.) polluting the tag namespace.
5. **Duplicate docs across locations.** Same files in `docs/product/` and `product/` with different version numbers, both claiming canonical status.
6. **Core concept has no hub note.** "Correction compounding" appears in 50+ docs but has no `[[correction-compounding]]` hub. Semantically central, structurally invisible.

**Context:** Spencer asked me to explore the vault via MCP and find patterns a file tree wouldn't reveal. The graph neighborhood queries, tag queries, and cross-vault search exposed all of this in ~4 tool calls.

**Takeaway:** When setting up the Calx vault, establish structural foundations from day one: tag taxonomy, hub notes for core concepts, wikilink conventions, no duplicate locations. The MCP server makes structural debt immediately visible — use it as an ongoing health check, not just a search tool. Vault structure is part of the methodology.

## 2026-03-20 — Vault as Recommended Calx Infrastructure

**Observation:** During discussion of the Calx product, the pattern of "vault for thinking, repo for code" emerged as a natural architectural split. The Obsidian MCP eliminates the token cost of file-tree crawling — one search call vs. 5-6 Glob/Read cycles. This isn't just developer convenience; it's a real cost reduction for agent workflows.

**Context:** Spencer confirmed this is the intended install story: `calx init` recommends the Obsidian MCP for non-code documents. Specs, PRDs, architecture decisions live in the vault; code lives in the repo. Agents pull what they need via MCP.

**Takeaway:** The vault + MCP pattern is part of the Calx product recommendation, not an internal tool choice. Document this in the Calx setup flow. The token savings alone are a selling point — structured access vs. blind file crawling.
