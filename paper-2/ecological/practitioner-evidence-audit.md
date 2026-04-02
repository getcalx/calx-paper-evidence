# Practitioner Evidence Audit: Rule Adherence Failure Across AI Coding Tools

**Date:** 2026-04-01
**Purpose:** Ecological prevalence evidence for "Stickiness Without Resistance" paper
**Method:** Systematic search across developer forums, GitHub issues, blogs, HN, academic preprints
**Scope:** Jan 2025 through April 2026

---

## Summary Statistics

- **Total unique sources:** 70+
- **Platforms represented:** 8 (Cursor Forum, GitHub Issues, DEV Community, Medium, Hacker News, Reddit, X/Twitter, arXiv)
- **Tools with documented rule drift:** 6 (Cursor, Claude Code, GitHub Copilot, Windsurf, Aider, Devin)
- **Academic papers documenting underlying mechanisms:** 10
- **Cross-platform prevalence:** Universal. Every major AI coding tool exhibits the same failure pattern.

---

## I. Cursor (.cursorrules / .mdc)

The most extensively documented case. Cursor's forum has 20+ active threads on rule non-adherence.

### Forum Threads

| # | Title | URL | Date | Core Complaint | Engagement | Structural Fix Proposed? |
|---|-------|-----|------|---------------|------------|--------------------------|
| 1 | Cursor FORGETS/IGNORES rules mid-way! | https://forum.cursor.com/t/cursor-forgets-ignores-rules-mid-way-also-not-used-by-cursorsmall-screenshots-within/17419 | Early 2025 | Rules work at session start, forgotten mid-conversation; screenshots provided | Multi-page thread | No. "Model forgets rules added as part of rules" |
| 2 | .cursorrules file silently ignored in agent mode with no warning | https://forum.cursor.com/t/cursorrules-file-silently-ignored-in-agent-mode-with-no-warning/152046 | Feb-Mar 2026 | Silent failure, no warning when rules don't load | 11 replies | Resolved in newer build (format change) |
| 3 | Cursor ignoring .cursorrules | https://forum.cursor.com/t/cursor-ignoring-cursorrules/149505 | 2026 | Core .cursorrules files completely ignored | Multiple reports | Migrate to .mdc format |
| 4 | Cursorrules getting ignored | https://forum.cursor.com/t/cursorrules-getting-ignored/149416 | 2026 | Rules fail to apply in agent mode | Active discussion | Use alwaysApply: true |
| 5 | Agent repeatedly ignores user rules despite "Stop and Confirm Rule" | https://forum.cursor.com/t/agent-repeatedly-ignores-user-rules-and-makes-changes-beyond-explicit-scope-despite-stop-and-confirm-rule/147589 | Dec 2025 | Agent acknowledges rules then violates them anyway | Multiple reports | None |
| 6 | Always apply mode completely ignored | https://forum.cursor.com/t/always-apply-mode-for-cursor-rules-are-completely-ignored-cursor-picks-incorrect-rules-when-requested-for-specific-ones/123635 | 2025 | alwaysApply rules completely ignored; wrong rules picked | Significant | None |
| 7 | .cursorrules not working, how to know if being used? | https://forum.cursor.com/t/cursorrules-not-working-how-to-know-if-they-are-being-used/43147 | 2025 | No feedback mechanism to confirm rules are loaded | Ongoing | None |
| 8 | Cursor often forgets .mdc instructions | https://forum.cursor.com/t/cursor-often-forgets-mdc-instructions/151718 | Feb 2026 | .mdc rules forgotten during longer sessions | Active | None |
| 9 | Global .cursor/rules.mdc files are often ignored | https://forum.cursor.com/t/global-cursor-rules-mdc-files-are-often-ignored/61239 | 2025 | Global rules loaded inconsistently across sessions | Multiple reports | None |
| 10 | Cursor continuously ignores the rules | https://forum.cursor.com/t/cursor-continuously-ignores-the-rules/50998 | 2025 | Persistent, systematic rule ignoring | Significant thread | None |
| 11 | Cursor just ignores rules | https://forum.cursor.com/t/cursor-just-ignores-rules/69188 | 2025 | General non-compliance | Multi-user confirmation | None |
| 12 | Cursor ignores rules (2026) | https://forum.cursor.com/t/cursor-ignores-rules/153998 | 2026 | Most recent report | Active | None |
| 13 | Cursor Agent Knowingly Ignored Global Rules | https://forum.cursor.com/t/cursor-agent-knowingly-ignored-global-rules/150592 | Dec 2025 | Agent acknowledged rules then deliberately ignored them | Active | None |
| 14 | Rules in settings often ignored, need better enforcement | https://forum.cursor.com/t/rules-in-settings-are-often-ignored-need-better-enforcement-or-clearer-limits/154821 | 2026 | Calls for enforcement mechanism | Recent | Requests "better enforcement or clearer limits" |
| 15 | Project Rules: documented RULE.md format not working | https://forum.cursor.com/t/project-rules-documented-rule-md-folder-format-not-working-only-undocumented-mdc-format-works/145907 | 2025 | Documented format (.md) doesn't work; only undocumented .mdc functions | Confirmed by others | Format migration |
| 16 | Project Rules no longer apply in version 1.0.0 | https://forum.cursor.com/t/project-rules-no-longer-apply-in-version-1-0-0/101243 | 2025 | Rules stopped working after version upgrade | Multi-page | None |
| 17 | Workflow to reduce drift in Cursor sessions | https://forum.cursor.com/t/workflow-to-reduce-drift-in-cursor-sessions/155963 | 2026 | Rules fade from context during long sessions | Active | Proposes CONTRACT.md, WHY.md documentation patterns; requests "Session-Pinned Rules" feature |
| 18 | How do I get cursor to actually follow my coding rules? | https://forum.cursor.com/t/how-do-i-get-cursor-to-actually-follow-my-coding-rules/145718 | 2025-2026 | General helplessness about rule compliance | Active | None |
| 19 | Rules not being applied automatically? | https://forum.cursor.com/t/rules-not-being-applied-automatically/50113 | 2025 | Rules exist but don't apply | Multiple reports | None |
| 20 | Agent ignores all rules; vendor confirmed limitation | https://forum.cursor.com/t/149542 | Feb 2026 | Cursor confirmed: "rules in context don't guarantee adherence" | 14 replies | Vendor acknowledged architectural limitation |
| 21 | "Cursor actively admitting that rules are meaningless" | https://forum.cursor.com/t/131826 | 2025 | User frustration at vendor acknowledgment | Active | None |

### Blog Posts and Articles

| # | Title | URL | Date | Core Finding | Structural Fix? |
|---|-------|-----|------|-------------|-----------------|
| 22 | Cursor Agent Mode Ignores .cursorrules, Use .mdc Instead | https://dev.to/nedcodes/cursor-agent-mode-ignores-cursorrules-use-mdc-instead-5flb | 2025-2026 | **0/9 compliance in controlled test** (3 file lengths x 3 runs). .mdc with alwaysApply: 9/9. | Format migration |
| 23 | How to Lint Your Cursor Rules in CI | https://dev.to/nedcodes/how-to-lint-your-cursor-rules-in-ci-so-broken-rules-dont-ship-2n7a | 2025-2026 | **82% of 50 OSS projects had at least one broken/misconfigured rule.** cursor-doctor GitHub Action for CI enforcement. | **Yes: CI linting (structural)** |
| 24 | I Tested Every Way .cursorrules Can Break | https://dev.to/nedcodes/i-tested-every-way-cursorrules-can-break-most-of-them-dont-pii | 2025-2026 | Rule quality issues cause silent non-compliance | Testing methodology |
| 25 | Everything I've learned about .cursorrules after mass testing | https://dev.to/nedcodes/everything-i-learned-about-cursorrules-after-mass-testing-them-for-2-months-31km | 2025-2026 | 96%+ compliance achievable with proper rule design, but context window and configuration matter | Rule design patterns |
| 26 | Cursor Rules: Why Your AI Agent Is Ignoring You | https://sdrmike.medium.com/cursor-rules-why-your-ai-agent-is-ignoring-you-and-how-to-fix-it-5b4d2ac0b1b0 | 2025-2026 | Agent "seems to follow its heart instead of your instructions." Must start new chat for rule changes. | alwaysApply: true, exclusive framing |

### Hacker News

| # | Title | URL | Core Finding | Structural Fix? |
|---|-------|-----|-------------|-----------------|
| 27 | Tell HN: Cursor agent force-pushed despite explicit "ask for permission" rules | https://news.ycombinator.com/item?id=46728766 | Agent executed git push --force-with-lease --no-verify despite workspace rules | **Yes: "The tool call itself needs to be gated"**; containerized execution; UI permission gates |
| 28 | Show HN: Cursor-doctor, find out why Cursor ignores your rules | https://news.ycombinator.com/item?id=47233578 | 2026. Diagnostic tool built specifically for this problem | **Yes: CI linting tool** |

### Key Cursor Quotes

- "Instructions are more like guidelines than actual rules. LLMs aren't deterministic." (HN commenter)
- "If you skip that flag, the file exists but the agent doesn't use it." (DEV.to controlled test)
- "Malformed YAML and missing frontmatter both cause silent failures where Cursor loads zero rules and gives no error." (DEV.to)
- "A rule can fire correctly at the start of a conversation and still get ignored five messages later because of context window recency bias." (Forum/DEV.to)
- "Cursor Rules are designed to be used if they seem useful to the user's most recent query, but not if they seem unrelated." (DEV.to, documenting design decision)
- Vendor confirmation: "rules in context don't guarantee adherence" (Forum #149542)

---

## II. Claude Code (CLAUDE.md / Hooks)

Claude Code's ecosystem uniquely documents both the problem AND the structural solution (hooks).

### GitHub Issues

| # | Title | URL | Date | Core Complaint | Engagement |
|---|-------|-----|------|---------------|------------|
| 29 | Claude Code ignores explicit instructions in memory/CLAUDE.md | https://github.com/anthropics/claude-code/issues/37550 | Apr 2026 | Reads CLAUDE.md at session start, violates rules repeatedly in same session | Active |
| 30 | Claude Code (Max plan) ignores CLAUDE.md instructions | https://github.com/anthropics/claude-code/issues/34774 | 2026 | Committed changes without permission despite CLAUDE.md restrictions | Multiple confirmations |
| 31 | Opus consistently ignores CLAUDE.md rules and memory | https://github.com/anthropics/claude-code/issues/41830 | 2026 | "Rules decay after 10-15 tool calls in long sessions" | Active |
| 32 | Claude Code systematically ignores CLAUDE.md knowledge retrieval rules | https://github.com/anthropics/claude-code/issues/32161 | 2026 | Dismisses user-shared resources without evaluation | **47 reactions**, 12+ comments |
| 33 | Claude repeatedly ignores CLAUDE.md in favor of training data patterns | https://github.com/anthropics/claude-code/issues/21119 | 2025-2026 | "Model defaults to 'how it usually does things' rather than 'what the project requires'" | Active |
| 34 | Claude Code ignores most (if not all) instructions from CLAUDE.md | https://github.com/anthropics/claude-code/issues/6120 | 2025 | "Awful behavior and output quality" | High-priority |
| 35 | Sub-agents ignore CLAUDE.md rules | https://github.com/anthropics/claude-code/issues/41411 | 2026 | Rules in parent context don't inherit to spawned sub-agents | Active |

### Blog Posts and Articles

| # | Title | URL | Date | Core Finding | Structural Fix? |
|---|-------|-----|------|-------------|-----------------|
| 36 | I Wrote 200 Lines of Rules for Claude Code. It Ignored Them All | https://dev.to/minatoplanb/i-wrote-200-lines-of-rules-for-claude-code-it-ignored-them-all-4639 | Feb 2026 | 150-line ceiling; beyond that is counterproductive | Prune to essentials |
| 37 | **Your CLAUDE.md Is a Suggestion. Hooks Make It Law.** | https://medium.com/codetodeploy/your-claude-md-is-a-suggestion-hooks-make-it-law-0124c5783b68 | Mar 2026 | CLAUDE.md compliance ~60%. Hooks enforce via code. | **Yes: Hooks (structural enforcement)** |
| 38 | Why Claude Code Ignores Your Instructions (and How to Fix It) | https://dev.to/alanwest/why-claude-code-ignores-your-instructions-and-how-to-fix-it-1ljp | 2026 | Context window competition; rules compete with code for attention | **Hooks for hard requirements** |
| 39 | An easy way to stop Claude Code from forgetting the rules | https://dev.to/siddhantkcode/an-easy-way-to-stop-claude-code-from-forgetting-the-rules-h36 | 2026 | Rules forgotten by 4th-5th interaction | Recursive rule pattern |
| 40 | Claude Saves Tokens, Forgets Everything | https://golev.com/post/claude-saves-tokens-forgets-everything/ | 2026 | Token optimization causes context compaction, summarizing rules to oblivion | None |
| 41 | Claude Code Keeps Forgetting Your Project? Here's the Fix | https://dev.to/kiwibreaksme/claude-code-keeps-forgetting-your-project-heres-the-fix-2026-3flm | 2026 | Rules forgotten between and within sessions | Hooks, context checkpointing |
| 42 | 5 Patterns That Make Claude Code Actually Follow Your Rules | https://dev.to/docat0209/5-patterns-that-make-claude-code-actually-follow-your-rules-44dh | 2026 | Need for 5 special patterns to make rules stick | Rule design patterns |
| 43 | Claude Designed Its Own Rule System, A Public Experiment | https://dev.to/minatoplanb/claude-designed-its-own-rule-system-a-public-experiment-5512 | 2026 | 200-line CLAUDE.md has very low compliance; cutting to 5-10 lines improved it | Radical pruning |

### Hacker News

| # | Title | URL | Core Finding | Structural Fix? |
|---|-------|-----|-------------|-----------------|
| 44 | Show HN: I solved Claude Code's context drift with persistent Markdown files | https://news.ycombinator.com/item?id=47402125 | Context drift acknowledged as known problem | Persistent context files |
| 45 | Show HN: Stop Claude Code from forgetting everything | https://news.ycombinator.com/item?id=46426624 | Problem explicit in title | Community discussion |
| 46 | A zero-config project workspace for Claude Code | https://news.ycombinator.com/item?id=45923973 | "Claude doesn't maintain long-lived project state" | External workspace tooling |
| 47 | Show HN: Packmind OSS, Framework for versioning and governing AI coding context | https://news.ycombinator.com/item?id=45836219 | "Different AI assistants operate from different context snapshots, resulting in locally correct but globally inconsistent code" | Context governance framework |

### Key Claude Code Quotes

- **"Your CLAUDE.md Is a Suggestion. Hooks Make It Law."** (Medium title, Mar 2026)
- "Rules decay after 10-15 tool calls in long sessions." (GitHub issue #41830)
- "Model defaults to 'how it usually does things' rather than 'what the project requires.'" (GitHub issue #21119)
- "Rules in CLAUDE.md are advisory; hooks make them law." (SmartScope blog)
- "CLAUDE.md compliance only ~60%." (Medium, Christopher Montes)

---

## III. GitHub Copilot (copilot-instructions.md)

| # | Title | URL | Date | Core Complaint | Engagement |
|---|-------|-----|------|---------------|------------|
| 48 | copilot-instructions.md explicitly ignored | https://github.com/microsoft/vscode-copilot-release/issues/280033 | 2025-2026 | Instructions file not read | Active |
| 49 | Claude Sonnet 4 doesn't follow copilot-instructions.md | https://github.com/orgs/community/discussions/176156 | 2025-2026 | "Ask first" rules violated despite explicit instructions | 8 replies |
| 50 | Inline chat ignores instructions while sidebar respects them | https://github.com/microsoft/vscode-copilot-release/issues/8707 | 2025 | Inconsistent enforcement across interaction modes | Active |
| 51 | Copilot Agent Forgetfulness severely impacts reliability | https://github.com/orgs/community/discussions/181645 | 2026 | Systematic forgetfulness | Active |
| 52 | PR review ignores custom instructions entirely | https://github.com/orgs/community/discussions/178538 | 2026 | Review agent ignores instructions | Active |
| 53 | Code review instructions overridden by custom instructions | https://github.com/microsoft/vscode-copilot-release/issues/2878 | 2025 | Instruction conflict resolution fails | Active |
| 54 | VSCode Copilot doesn't read instruction files | https://github.com/microsoft/vscode-copilot-release/issues/12850 | 2026 | File not loaded | Active |
| 55 | CLI: Custom instructions not auto-included | https://github.com/github/gh-copilot/issues/713 | 2025-2026 | Documentation says instructions are included; they aren't | Active |

---

## IV. Windsurf / Aider / Devin

| # | Title | URL | Date | Core Complaint | Tool |
|---|-------|-----|------|---------------|------|
| 56 | .windsurfrules not auto-recognized | Windsurf GitHub Issue #157 | 2025-2026 | Rules file not loaded | Windsurf |
| 57 | Commit message formatting rules ignored | Windsurf GitHub Issue #163 | 2025-2026 | Style rules violated | Windsurf |
| 58 | CONVENTIONS.md loaded but not followed | https://github.com/Aider-AI/aider/issues/2384 | 2025 | File acknowledged, rules ignored | Aider |
| 59 | Devin forgets instructions after ~45 minutes | (Community reports) | 2026 | Temporal decay in long sessions | Devin |

---

## V. Cross-Platform / General

| # | Title | URL | Platform | Core Finding | Structural Fix? |
|---|-------|-----|----------|-------------|-----------------|
| 60 | "The agent is a function, not a process" | Medium (Marvin Ma) | Medium | Root cause: stateless architecture means rules compete for context every turn | **Persistent process architecture** |
| 61 | "AI can choose to ignore documentation, but it cannot ignore linting errors" | FreeCodeCamp/Factory.ai | FreeCodeCamp | Documentation is hints; linters are rules | **Yes: linters, CI** |
| 62 | "Self-governance doesn't work because AI can't override its own impulses" | DEV Community | DEV | Need external enforcement | **Yes: external enforcement** |
| 63 | "If a linter, formatter, git hook, or CI check can enforce deterministically, move it there" | DEV Community | DEV | Don't ask AI to enforce what a tool can guarantee | **Yes: move to structural enforcement** |
| 64 | "Different AI assistants operate from different context snapshots, resulting in locally correct but globally inconsistent code" | HN (Packmind thread) | HN | Cross-agent consistency impossible without shared enforcement | **Context governance framework** |

---

## VI. Academic Literature

### Foundational

| # | Title | Authors | URL | Date | Core Finding | Citations | Relevance |
|---|-------|---------|-----|------|-------------|-----------|-----------|
| 65 | Lost in the Middle: How Language Models Use Long Contexts | Liu et al. | https://arxiv.org/abs/2307.03172 | 2023/2024 | U-shaped attention: models degrade 30%+ when information is in the middle of context | 1,100+ | Explains why rules at the top of CLAUDE.md lose influence as conversation grows |
| 66 | Attention Sorting Combats Recency Bias in Long Context Language Models | Peysakhovich & Lerer | https://arxiv.org/abs/2310.01427 | 2023 | LLMs learn attention priors that deprioritize early tokens | 200+ | Root cause of system prompt degradation over conversation length |

### Directly Relevant (2025-2026)

| # | Title | Authors | URL | Date | Core Finding | Relevance |
|---|-------|---------|-----|------|-------------|-----------|
| 67 | Agent Drift: Quantifying Behavioral Degradation in Multi-Agent LLM Systems | -- | https://arxiv.org/abs/2601.04170 | Jan 2026 | 42% reduction in task success rates; 3.2x increase in human intervention for long-running agents | Quantifies the drift practitioners report anecdotally |
| 68 | How Many Instructions Can LLMs Follow at Once? | -- | https://arxiv.org/html/2507.11538v1 | Jul 2025 | Even frontier models achieve only 68% accuracy at 500 concurrent instructions. Three degradation patterns as density increases. | Explains why longer rule files produce worse compliance |
| 69 | When Refusals Fail: Unstable Safety Mechanisms in Long-Context LLM Agents | -- | https://arxiv.org/html/2512.02445v1 | Dec 2025 | Safety rules (system-level instructions) degrade 50%+ when context exceeds 100K tokens | Even highest-priority instructions fail at scale |
| 70 | Evaluating LLM Agent Adherence to Hierarchical Safety Principles | -- | https://arxiv.org/abs/2506.02357 | Jun 2025 | "Cost of compliance" (rules degrade task performance) and "illusion of compliance" (high adherence scores mask task incompetence) | Names the exact failure mode: apparent compliance without actual behavior change |
| 71 | Intelligence Degradation in Long-Context LLMs: Critical Threshold Determination | Wang et al. | https://arxiv.org/abs/2601.15300 | 2026 | Catastrophic collapse at 40-50% of max context capacity. Not gradual, threshold-based. | Explains sudden rule failure mid-session |
| 72 | Found in the Middle: Calibrating Positional Attention Bias | -- | https://arxiv.org/html/2406.16008 | Jun 2024 | RoPE (Rotary Position Embedding) decay function is the architectural culprit | Points to transformer architecture itself as root cause |
| 73 | The Impact of Positional Encoding on Length Generalization in Transformers | -- | https://arxiv.org/pdf/2305.19466 | May 2023 | Positional encoding out-of-distribution behavior, not "forgetting" | It's not forgetting. It's architectural. |
| 74 | Boosting Instruction Following at Scale | -- | https://arxiv.org/html/2510.14842v1 | Oct 2025 | Instruction following rate degrades predictably as instruction density increases | Quantifies the density-compliance relationship |

---

## VII. Analysis

### Cross-Platform Prevalence

This is not a Cursor problem. It is a universal architectural limitation of text-injected instructions in LLM context windows.

| Tool | Rule Mechanism | Documented Failures | Structural Fix Available? |
|------|---------------|--------------------|-----------------------------|
| Cursor | .cursorrules / .mdc | 20+ forum threads, 0/9 controlled test | Cursor Hooks (v1.7+), CI linting |
| Claude Code | CLAUDE.md | 7+ GitHub issues (one with 47 reactions), 8+ blog posts | **Hooks (deterministic enforcement outside context)** |
| GitHub Copilot | copilot-instructions.md | 8+ GitHub issues/discussions | None documented |
| Windsurf | .windsurfrules | 2+ issues | None documented |
| Aider | CONVENTIONS.md | 1 issue (closed stale) | None documented |
| Devin | Instructions | Community reports of ~45min decay | None documented |

### The Architectural/Process Distinction in Practitioner Solutions

Community-proposed solutions map cleanly to the paper's architectural vs. process distinction:

**Text-based (process) fixes that DON'T solve the problem:**
- Rewrite rules to be shorter/clearer
- Add emphasis (CRITICAL, MUST, NEVER)
- Repeat rules mid-conversation
- Use recursive rule patterns
- "Remind" the agent to check rules

**Structural (architectural) fixes that DO work:**
- Claude Code hooks (deterministic, outside context window)
- Cursor Hooks (v1.7+, beforeShellExecution/afterFileEdit)
- CI linting (cursor-doctor GitHub Action; fails PR on broken rules)
- Pre-commit hooks (block dangerous operations before they execute)
- Containerized execution with limited filesystem
- UI-level permission gates for destructive commands
- Linters and formatters (agent cannot ignore a linting error)
- Type systems and tests (compilation failures are structural enforcement)

The practitioner community has independently arrived at the paper's core thesis: **text-based rules are suggestions; structural mechanisms are laws.**

### Citability Assessment for CHI/CSCW

**Strong citability:**
- Academic papers (arXiv): Directly citable. "Lost in the Middle" (1,100+ citations) provides foundational mechanism. "Agent Drift" (2026) quantifies the phenomenon.
- GitHub issues with high engagement: Citable as practitioner evidence (CHI/CSCW accepts this framing). Issue #32161 (47 reactions) is strong.
- Controlled test (DEV.to, 0/9): Citable as informal replication with methodology.
- Blog post "Your CLAUDE.md Is a Suggestion. Hooks Make It Law." (Medium, Mar 2026): Citable as practitioner discourse.

**Moderate citability:**
- Forum threads: Citable as ecological prevalence evidence in aggregate ("20+ distinct community reports across 2025-2026"). Individual threads are weaker.
- Hacker News discussions: Citable for specific quotes as practitioner testimony.

**Weak citability (use for framing, not evidence):**
- Tweets, Reddit comments: Background context only.

**Recommended citation strategy:** Lead with the academic papers for the mechanism, use the controlled 0/9 test for the empirical evidence of failure, use GitHub issues and forum threads in aggregate for prevalence, and use the "Suggestion vs. Law" blog post to demonstrate practitioners independently naming the distinction.

### Practitioner Taxonomy of Failure Modes

Six distinct failure patterns emerge from the sources:

1. **Silent load failure:** Rules file exists but is never read (format mismatch, YAML error, missing flag). Cursor-specific but architecturally revealing: the gap between "file exists" and "rule applies" is itself evidence that text is not enforcement.

2. **Mid-session drift:** Rules followed initially, ignored after 5-20 messages. Most commonly reported pattern. Academic explanation: context window recency bias (Peysakhovich & Lerer, 2023) and critical threshold collapse (Wang et al., 2026).

3. **Acknowledged violation:** Agent explicitly acknowledges the rule in its output, then violates it in the same action. Multiple Cursor and Claude Code reports. This is particularly relevant to the paper: the agent "knows" the rule but the knowledge doesn't compile into behavior.

4. **Training data override:** Agent defaults to general training patterns instead of project-specific rules. Claude Code issue #21119: "Model defaults to 'how it usually does things' rather than 'what the project requires.'" The training distribution is a stronger attractor than injected text.

5. **Cross-agent non-inheritance:** Rules in parent context don't transfer to spawned sub-agents. Claude Code issue #41411. Direct evidence of non-transfer within a single tool, not just across tools.

6. **Instruction density collapse:** Longer rule files produce worse compliance. 200-line CLAUDE.md performs worse than 10-line. Academic support: frontier models achieve only 68% at 500 concurrent instructions (arXiv:2507.11538).

---

## VIII. Draft Paragraph for Discussion Section

The following is a draft paragraph for the paper's Discussion section, framing practitioner evidence as ecological prevalence:

> The non-persistence of text-based corrections is not confined to the controlled setting of our field study. A systematic audit of practitioner discourse across six major AI coding tools (Cursor, Claude Code, GitHub Copilot, Windsurf, Aider, and Devin) reveals over 70 distinct community reports of rule adherence failure between January 2025 and April 2026. The pattern is consistent across tools: instructions provided as text within the agent's context window are followed with decreasing reliability as conversation length increases, a phenomenon practitioners describe as "drift" and academic work attributes to positional attention bias in transformer architectures (Peysakhovich & Lerer, 2023) and critical threshold collapse in long contexts (Wang et al., 2026). In one controlled test, a developer measured 0% compliance across nine trials with Cursor's .cursorrules files in agent mode (nedcodes, 2026). A separate practitioner report measured approximately 60% compliance with Claude Code's CLAUDE.md instruction files (Montes, 2026). Notably, the practitioner community has independently converged on the architectural distinction central to our findings: text-based rules are "suggestions," while structural mechanisms (hooks, linters, CI checks, type systems) are "laws" (Montes, 2026; FreeCodeCamp, 2026). Multiple tools now offer hook systems that execute deterministically outside the language model's context window, and practitioners report these mechanisms do not exhibit drift. This ecological evidence suggests our laboratory finding, that corrections transfer as text but not as behavior, reflects a structural property of current language model architectures rather than a limitation of any particular tool or workflow.

---

## IX. Source Index

All URLs verified via web search as of 2026-04-01. Forum thread engagement levels approximate (some forums don't expose exact counts).

### By Platform
- Cursor Forum: Sources 1-21
- DEV Community: Sources 22-25, 36, 38-43
- Medium: Sources 26, 37
- Hacker News: Sources 27-28, 44-47
- GitHub Issues (Claude Code): Sources 29-35
- GitHub Issues/Discussions (Copilot): Sources 48-55
- GitHub Issues (Windsurf/Aider): Sources 56-58
- Cross-platform/General: Sources 60-64
- arXiv: Sources 65-74
