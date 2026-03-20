# Lessons Log — Versed Frontend

Append-only mistake log. Every correction gets an entry.
Each entry records: what happened, root cause, prevention rule.
Prevention rules get distilled into the relevant AGENTS.md file.

Format:
```
### LXXX — <short title>
**What:** <observable symptom>
**Why:** <root cause>
**Prevention:** <specific rule>
**Distilled to:** <AGENTS.md rule ID, or "pending">
```

---

### L001 — Spec code examples must match spec tables
**What:** Token alignment spec table listed Lato weights 400/500/700/900 but the code example only loaded 400/700/900. Implementation would have missed 500 (medium weight for buttons/nav).
**Why:** Code examples get copy-pasted directly — they're the de facto source of truth, not the prose tables above them.
**Prevention:** After writing a spec, cross-check every code example against every table/list that defines the same values. If they diverge, the code example wins and must be correct.
**Distilled to:** N/A (spec writing)

### L002 — Cross-origin storage is never implicit
**What:** PasteInput spec used sessionStorage to pass data from getversed.ai to app.getversed.ai. sessionStorage is origin-scoped — this is dead on arrival.
**Why:** Assumed subdomain = same origin. It doesn't. Different subdomains are different origins for storage APIs.
**Prevention:** Any time data crosses subdomains, specify the transfer mechanism explicitly (API relay, postMessage, URL params). Never assume browser storage works cross-origin.
**Distilled to:** lib-R004

### L003 — Locked values are locked
**What:** Spec changed --border-subtle from #2A2A2A (locked brand review) to #2A2A2E without approval. 1-digit hex deviation.
**Why:** Spec author applied a "warm shift" pattern to borders without checking if the brand review had already locked those values.
**Prevention:** Before changing any token value, check the locked brand review. If the value is locked, use it exactly. Deviations require explicit Spencer approval documented in the spec.
**Distilled to:** comp-R012

### L004 — Animation specs need a reduced-motion counterpart
**What:** .animate-in/.animate-out CSS classes had no prefers-reduced-motion handling. Accessibility gap.
**Why:** Animation specs focused on the visual effect, not the accessibility fallback.
**Prevention:** Every animation definition must include its @media (prefers-reduced-motion: reduce) counterpart. Add it in the same block, not as a follow-up.
**Distilled to:** comp-R006

### L005 — "All sections" language creates contradictions
**What:** "All 7 sections wrapped in SectionWrapper" contradicted "hero has no scroll animation" two lines later.
**Why:** Used sweeping language ("all") when the actual behavior had an exception.
**Prevention:** Be precise about scope. "Sections 2-7" instead of "all 7 sections." Exceptions to universal statements should be resolved by narrowing the statement, not adding a contradicting exception.
**Distilled to:** N/A (spec writing)

### L006 — File structure must account for every referenced section
**What:** Home page Section 3 (LinkedIn Teaser) was in the page layout but missing from the component file structure list. Reviewer flagged it as a missing component.
**Why:** Section was simple enough to build inline but the spec didn't say that — it just omitted it from the file list.
**Prevention:** Every named section in a page layout must have a component entry in the file structure. Consistency with the pattern matters more than saving a file.
**Distilled to:** comp-R011

### L007 — Integration contracts require bilateral specification
**What:** 7 of 41 foil review issues (7, 12, 13, 14, 16, 37, 41) were contract mismatches — one side defined a contract the other side couldn't consume. Examples: FE used `X-Claim-Token` response header but `EventSource` can't read headers. FE used `/v1/paste/store` while BE used `/v1/paste`. FE referenced `SSETranslationCard` types that didn't exist in contracts/types.ts. localStorage key was `versed_claim_token` on one side and unnamed on the other.
**Why:** Each side wrote contracts independently without confirming the other side would consume them correctly. No bilateral contract review step in the spec process.
**Prevention:** When a spec introduces a cross-boundary contract (endpoint, SSE event, localStorage key, type), both the producer spec and consumer spec must define the contract explicitly with matching field names, paths, keys, TTL values, and type definitions. If only one side writes it, it will diverge.
**Distilled to:** CLAUDE.md (Protocol)

### L008 — Always consume the producer's field names, not your own
**What:** Four round-2 blockers were all the same bug: Faye invented field names instead of reading Bryce's published contracts. `text` instead of `content` (Pydantic returns 422). nanoid instead of UUID (FastAPI path param rejects it). camelCase destructuring of snake_case SSE payload (fields are undefined). 24h cookie TTL when claim token is 72h (silent 401s at hour 36).
**Why:** Wrote TypeScript interfaces from what the field *should* be called semantically, not from what Bryce's Pydantic models actually emit. Never read `contracts/types.ts` or Bryce's published models before defining FE types.
**Prevention:** When writing any interface that touches a backend contract, read the producer's published model first. The wire format is the producer's contract — field names, types, formats, TTLs. Mirror it exactly in the TypeScript interface. Build a mapper on the FE side if semantic naming differs, but the wire type must match the producer.
**Distilled to:** type-R003

### L009 — Parallelize independent edits with subagents
**What:** Applied 9 independent one-line fixes sequentially via find-and-replace. Spencer waited through ~15 edit operations that had zero dependencies on each other.
**Why:** Defaulted to sequential tool calls out of habit instead of evaluating which edits were independent and could be dispatched in parallel.
**Prevention:** When a revision brief contains N independent fixes, dispatch them as parallel subagents. Each fix is a self-contained edit with its own file location and replacement text. Only serialize edits that depend on each other's output.
**Distilled to:** CLAUDE.md (Dispatch)

### L010 — Session bootstrap replaces manual coordination reads
**What:** Coordination layer upgrade added machine-truth injection at session start (session-orient.sh) and a read-before-write gate (coordination-gate.sh). Session flow section in CLAUDE.md must reflect that manual `read_board()`/`read_handoff()` calls are no longer needed at session start — bootstrap handles it.
**Why:** The old protocol said "Session start: read_board() + read_handoff()" which was redundant with bootstrap and wasted tool calls. The new gate also blocks stale writes (>30min advisory).
**Prevention:** When infrastructure hooks change the coordination protocol, update CLAUDE.md session flow + protocol sections immediately. Don't let docs diverge from enforcement reality.
**Distilled to:** CLAUDE.md Session Flow + Protocol sections updated

### L011 — Plan dispatch strategy before touching files
**What:** Received a 4-phase plan (foil fixes + 10 new specs) and immediately started editing files sequentially instead of planning the dispatch strategy first. Spencer had to interrupt. This is the second consecutive session where this happened.
**Why:** Pattern: large plan arrives → instinct to start executing immediately → skip the chunking/dispatch decision. The working model requires determining (1) whether work is subagent vs agent-team territory, (2) how to chunk for 200k context windows, and (3) wave sequencing — BEFORE any file touches.
**Prevention:** When a plan has >2 deliverables, the FIRST action is always: assess independence → choose subagent vs agent-team → chunk into waves that fit 200k context → dispatch. Never start editing before the dispatch strategy is set. This is the orchestrator's job, not a step to skip.
**Investigate:** This is the second time in a row. Check if there's a pattern in how plan-mode exit triggers premature execution — the plan output may be priming immediate action instead of dispatch planning.
**Distilled to:** CLAUDE.md (Dispatch)

### L012 — Spec revision agents need pre-sliced reference docs
**What:** All 5 Wave 3 agents spent significant time cross-referencing WIRE_CONTRACTS.md (51KB) against each spec. The heaviest agents (job-search: 25 fixes, recruiter-canvas: 21 fixes) ran longest because they repeatedly re-read wire contracts for each field fix.
**Why:** The prompt template tells agents to read WIRE_CONTRACTS.md but the file is 51KB — nearly filling the context window before the spec + correction file + FE_SPEC_FIXES.md are loaded. Agents that needed heavy cross-referencing hit context pressure.
**Prevention:** For spec revision dispatches against large reference docs, extract only the relevant section of the reference doc into the agent prompt. A recruiter-canvas agent only needs the Spec 28/30 section of WIRE_CONTRACTS, not all 51KB. Pre-slice the reference material per agent.
**Distilled to:** CLAUDE.md (Dispatch)

### L013 — SSE-to-JSON corrections cascade through entire specs
**What:** recruiter-canvas-ui had its entire data flow rewritten (SSE to JSON) — 36+ edits. resume-optimization-ui had the same in Wave 2. Both were the longest-running agents in their waves.
**Why:** When the response model changes from streaming (SSE) to synchronous (JSON), it's not a field rename — it rewrites loading states, data flow diagrams, component lifecycle, error handling, and test expectations. Every section that references "stream" needs revision.
**Prevention:** Flag SSE-vs-JSON as the first check in any spec revision. If the response model is wrong, fix the data flow section first, then cascade through the spec. Don't interleave it with field-level fixes — it touches everything.
**Distilled to:** CLAUDE.md (Dispatch)

### L014 — Batch RFC filing after spec revision, not during
**What:** Plan correctly deferred RFC filing to Phase 2 (post-revision). This worked well — agents marked RFC needs inline during revision, then we compiled and filed 6 RFCs in one batch after all specs were done.
**Why:** Filing RFCs during revision would have required coordination mid-flight and risked duplicate RFCs across parallel agents. The inline `<!-- RFC NEEDED -->` comment pattern let agents flag needs without blocking.
**Prevention:** This is the right pattern. Spec revision agents mark RFC needs with inline comments. Orchestrator compiles and files RFCs as a batch after all revisions complete. Never file RFCs from inside a revision agent.
**Distilled to:** CLAUDE.md (Verification)

### L015 — Component agent prompts need full type context
**What:** Session 2 component agents (TranslationCard, GapCard, etc.) each needed the full prop interface, design token list, ARIA requirements, and interaction specs baked into the prompt. Agents that were missing design token values or exact prop shapes had to infer or read files mid-flight, slowing them down.
**Why:** Agents run in isolated context — they can read files but starting with all the information is faster than discovering it. The spec is 1000+ lines but each agent only needs ~100 lines of relevant context.
**Prevention:** When dispatching component agents, pre-extract: (1) the exact Props interface, (2) the visual structure spec, (3) relevant design tokens, (4) ARIA requirements, (5) test cases. Don't send the full spec — slice the relevant section per agent.
**Distilled to:** CLAUDE.md (Dispatch)

### L016 — Vitest needs path alias config for @/ imports
**What:** Vitest tests failed to resolve `@/` imports until `resolve.alias` was added to vitest.config.ts. Next.js handles this via tsconfig paths but Vitest doesn't read tsconfig paths by default.
**Why:** Vitest uses Vite's resolver, not TypeScript's. The `@/*` path alias in tsconfig.json is only for the TS compiler — Vitest needs its own `resolve.alias` config.
**Prevention:** When setting up a Next.js project with Vitest, always configure `resolve.alias` in vitest.config.ts to match tsconfig paths. This is a one-time setup but easy to miss.
**Distilled to:** done (vitest.config.ts updated)

### L017 — Validation functions that return null lose failure-mode granularity
**What:** `readAnonSession()` returns `null` for both "no localStorage entry" and "entry exists but malformed JSON/shape." `attemptClaim()` called it and got `null`, then returned `{ reason: "no_token" }` when the correct result was `{ reason: "malformed" }`. Test caught it.
**Why:** The function was designed as a simple "parse or null" helper. The caller needed to distinguish two null paths (missing vs. corrupt) but the helper collapsed them into one.
**Prevention:** When a caller needs to distinguish multiple failure modes, check preconditions (e.g., `localStorage.getItem()` existence) before calling the validation function. Don't rely on a null-returning helper to encode why it returned null.
**Distilled to:** hook-R005

### L018 — Integration wiring breaks existing tests via import cascade
**What:** Adding `useAuth()` + `SavePrompt` to the translate page caused all 11 existing translate page tests to fail. `SavePrompt` imports `EmailCapture` which imports `supabase` which throws without env vars. The `useAuth()` call also threw because no `AuthProvider` wrapper existed in the test render tree.
**Why:** New imports in a component cascade through the module graph. Any module that throws at import time (like Supabase client init without env vars) breaks the entire test file. And hooks that require providers break all renders.
**Prevention:** When integrating new dependencies into an existing component, immediately check that component's test file. Add mocks for every new module in the import chain that has side effects or requires providers. Do this in the same step as the integration edit, not after seeing test failures.
**Distilled to:** comp-R014

### L019 — ReactFlow CanvasFlowNode type doesn't accommodate mixed node data shapes
**What:** Canvas page TypeScript errored because IdentityNode's data shape (user, vocabStats, isExpanded) doesn't match `CanvasFlowNode`'s data type (`{ cardType, cardData, isCompact }`). The identity node is fundamentally different from card nodes.
**Why:** `CanvasFlowNode` was defined as `ReactFlowNode<{ cardType; cardData; isCompact }>` — a single generic type for all nodes. But the canvas has two distinct node types with incompatible data shapes.
**Prevention:** When a ReactFlow canvas has multiple node types with different data shapes, use the base `Node` type for the flow nodes array. Let each node type's wrapper component handle its own data typing via the `nodeTypes` registry. Don't force all nodes into a single generic.
**Distilled to:** type-R004

### L020 — jsdom doesn't support window.matchMedia
**What:** Canvas page tests all failed with `TypeError: window.matchMedia is not a function` because the `useIsMobile` hook calls `window.matchMedia` on mount.
**Why:** jsdom (Vitest's default DOM environment) doesn't implement `matchMedia`. Any component using media queries needs this mocked.
**Prevention:** When writing a responsive hook that uses `window.matchMedia`, add a matchMedia mock at the top of the test file (or in a shared test setup). This is a one-time per-hook pattern, easy to forget.
**Distilled to:** hook-R006

### L021 — React lint flags synchronous setState in useEffect body
**What:** `useIsMobile` hook called `setIsMobile(mql.matches)` synchronously inside useEffect. React's lint rule flagged this as a cascading render risk.
**Why:** Synchronous setState in effect body runs after render, triggering an immediate re-render. The fix is to use a lazy state initializer (`useState(() => window.matchMedia(...).matches)`) so the initial value is correct from the first render.
**Prevention:** When a hook needs to read a browser API to set initial state, use `useState(() => browserAPI())` lazy initializer instead of `useEffect(() => setState(browserAPI()))`. The effect should only handle subscription/cleanup.
**Distilled to:** hook-R003

### L022 — Server components cannot have event handler props
**What:** `TranslationCard` (marketing, server component) had `onMouseEnter`/`onMouseLeave` handlers for hover glow effect. `FounderPhoto` had `onError` on `<Image>` for fallback. Both failed at build time: "Event handlers cannot be passed to Client Component props."
**Why:** Server components are serialized to HTML — they can't attach DOM event listeners. Any interactivity requires `"use client"` or CSS-only solutions. Subagents wrote JS event handlers without checking the component's rendering model.
**Prevention:** When a component is a server component (no `"use client"`), hover/focus effects must use CSS-only (Tailwind `hover:`, `group-hover:`, CSS `:hover`). Image fallbacks in server components must use CSS `object-fit` + background color or a client wrapper. Add "server component constraint" to subagent prompts when dispatching.
**Distilled to:** comp-R007

### L023 — Route groups isolate auth from SSG pages
**What:** Root layout wrapped all pages in AuthProvider, which imported Supabase client, which threw at build time (missing env vars) for statically generated marketing pages. `npm run build` failed on /translate and /canvas prerendering.
**Why:** AuthProvider triggers Supabase client initialization at module import time. SSG pages don't need auth but inherited it from the root layout. No separation between auth-required and public pages.
**Prevention:** Use Next.js route groups `(app)` and `(marketing)` to separate layouts. Auth providers go in `(app)/layout.tsx` with `export const dynamic = 'force-dynamic'`. Marketing layout has Nav + Footer only. Root layout stays minimal (fonts, globals, metadata).
**Distilled to:** comp-R013

### L024 — Internal links must use next/link Link component
**What:** Nav, Footer, and CTAButton used `<a href="/">` and `<a href="/for-employers">` for internal routes. Next.js ESLint rule `@next/next/no-html-link-for-pages` flagged them as errors.
**Why:** Subagents wrote standard HTML anchors. Next.js requires `<Link>` for internal navigation (client-side routing, prefetching).
**Prevention:** Add to subagent prompts: "For internal routes (href starts with `/`), use `<Link>` from `next/link`. Use `<a>` only for external URLs or `mailto:` links. This is enforced by Next.js lint."
**Distilled to:** comp-R008

### L025 — Subagent prompts must specify rendering model constraints
**What:** Three server component issues in this session (L022 + L024) all came from subagents not knowing the constraints of server vs client components in Next.js App Router.
**Why:** Subagent prompts specified `"use client"` requirements per component but didn't explain the SERVER component constraints (no event handlers, no hooks, must use `<Link>` for internal routes).
**Prevention:** When dispatching component subagents for Next.js App Router, include a "Server Component Rules" section in the prompt: (1) no event handlers (onX props), (2) no hooks, (3) no browser APIs, (4) use `<Link>` for internal routes, (5) hover/focus effects must be CSS-only. This saves a round of integration fixes.
**Distilled to:** comp-R007

### L026 — Intersection types for canvas data avoid type drift
**What:** `JobCardData` was previously a standalone interface that duplicated fields from `JobMatchDisplay`. When `JobMatchDisplay` was updated (e.g. `jobTitle` → `title`), `JobCardData` diverged silently. Tests and components broke at runtime, not compile time.
**Why:** Two interfaces describing the same data shape with no structural relationship. Changes to one don't propagate to the other.
**Prevention:** When a canvas data type represents the same data as a display type, define it as an intersection: `type JobCardData = JobMatchDisplay & { type: 'job_card' }`. The discriminator enables the CardData union while the intersection ensures field changes propagate automatically.
**Distilled to:** type-R005

### L027 — Moving useCallback before useMemo that depends on it
**What:** `handleOptimize` was defined AFTER the `flowNodes` useMemo that referenced it. React Compiler couldn't preserve the memoization because the dependency wasn't yet defined. Lint errored: "Existing memoization could not be preserved."
**Why:** Added `onOptimize: handleOptimize` to the flowNodes builder but placed the `handleOptimize` definition lower in the component. JavaScript hoisting doesn't apply to `useCallback` — hooks must be called in order.
**Prevention:** When adding a callback to a useMemo's data, define the callback ABOVE the useMemo and add it to the dependency array. Check hook ordering whenever wiring new callbacks into existing memos.
**Distilled to:** hook-R004

### L028 — role="article" doesn't support aria-expanded
**What:** CanvasCard used `role="article"` with `aria-expanded` attribute. Lint rule `jsx-a11y/role-supports-aria-props` flagged this — the `article` role doesn't support `aria-expanded` per ARIA spec.
**Why:** The original implementation chose `role="article"` for semantic meaning but then needed expand/collapse state indication. These are incompatible.
**Prevention:** When a component needs `aria-expanded`, use a role that supports it: `region`, `button`, `treeitem`, `row`, etc. Check the ARIA role spec for supported states before combining role + aria attributes.
**Distilled to:** comp-R009

### L029 — Dual field name mappers need explicit type escape hatch
**What:** `mapVocabularyMapping` needed to check both `user_language` (deployed) and `source_phrase` (Spec 24 future). TypeScript rejected `wire as Record<string, unknown>` because `WireVocabularyMapping` has known fields and TS won't allow a direct cast to a wider type (TS2352).
**Why:** The mapper pattern of checking for fields by string key conflicts with TypeScript's strict structural typing. The wire type has explicit fields, so `Record<string, unknown>` is not a supertype.
**Prevention:** When a mapper needs to check for fields that may or may not exist on a typed object, use `as unknown as Record<string, unknown>` (double cast through unknown). This is an intentional escape hatch for version compatibility mappers — document why in a comment.
**Distilled to:** type-R006

### L030 — Worktree agents drift from shared type prep
**What:** 4 worktree agents built against slightly different type shapes than the shared prep on `feature/canvas-engine`. StarStoryCard expected `{text, score}` section objects instead of plain strings + separate `sectionScores`. StoryConnector expected `fromPosition`/`toPositions` not in the type. 5 components destructured `className` not in their props interfaces. ~38 type errors on merge.
**Why:** Shared types were prepped on the main working tree, but each worktree agent also modified the shared type files independently, creating divergent shapes. The agents wrote components against their local type definitions, not the canonical prep.
**Prevention:** When dispatching worktree agents with shared type prep on main: (1) Tell agents which type files are canonical and MUST NOT be modified — components adapt to the types, not the other way around. (2) Copy the canonical type files into each worktree before dispatch so agents see the real shapes. (3) If an agent needs a type change, it should flag it with a comment, not make the change.
**Distilled to:** CLAUDE.md (Dispatch)

### L031 — Price constants must not include formatting symbols
**What:** `PRO_PRICE_STANDARD = '$29/mo'` caused double-dollar rendering (`$$29/mo`) in JSX because components used `${PRO_PRICE_STANDARD}` (literal `$` + constant that already starts with `$`). 4 test failures across UpgradeModal and PreCommitmentPrompt.
**Why:** Constants included display formatting (dollar sign, `/mo` suffix) but the components also added formatting. Neither side knew the other was responsible for the `$` prefix.
**Prevention:** Price constants should be raw display values without units: `'$29'` not `'$29/mo'`. Components add the period suffix (`/mo`) in their own markup. This separates data from presentation and prevents double-formatting.
**Distilled to:** done (constants updated)

### L032 — Reconcile before running tests, not after
**What:** Ran full test suite before checking type errors, then had to re-run after type fixes, then re-run after price fixes. Three full test runs (6.5s each) instead of one.
**Why:** Rushed to verify test count without first confirming clean typecheck. Type errors always cascade into test failures.
**Prevention:** After merging worktree output: typecheck first (`tsc --noEmit`), fix all type errors, THEN run the full test suite exactly once. Typechecking is faster than test execution and catches most merge issues.
**Distilled to:** CLAUDE.md (Verification)

### L033 — React Compiler rejects Date.now() anywhere during render
**What:** `useStaleCheck` hook called `Date.now()` inside `useMemo`, then inside `useRef()` initializer. Both flagged by `react-hooks/purity` — React Compiler treats `Date.now()` as impure in any render-path code.
**Why:** React Compiler enforces strict purity — all code that runs during render must be idempotent. `Date.now()` returns different values on re-render, violating this. Even `useRef(Date.now())` runs during render.
**Prevention:** For time-dependent logic, use `useState(() => Date.now())` (lazy initializer is allowed) or extract to a pure function that takes `now` as a parameter. The pure function approach (`computeStaleCheck(updatedAt, appliedAt, status, now)`) is cleaner — caller captures time once, function stays pure and testable without fake timers.
**Distilled to:** done (use-stale-check.ts refactored to pure function)

### L034 — aria-expanded is not supported by role="article"
**What:** 7 lint warnings across ApplicationCard, JDCenterNode, RepellentCard, JDDashboard, StarStoryCard — all use `role="article"` with `aria-expanded`. The ARIA spec doesn't support `aria-expanded` on the `article` role.
**Why:** Spec says `role="article"` for semantic card meaning + `aria-expanded` for expand/collapse state. These are incompatible per ARIA spec. This is L028 recurring — same root cause, now at scale across multiple components.
**Prevention:** When a card component needs expand/collapse, use `role="region"` (supports `aria-expanded`) instead of `role="article"`. Or use no role and rely on `aria-expanded` on a child button element. Check ARIA role support before combining role + aria-state attributes.
**Distilled to:** comp-R009

### L035 — Subagent tests validate component behavior, not spec acceptance criteria
**What:** Subagent-written tests tend toward "renders correctly, fires callbacks" rather than encoding specific spec ACs. Example: spec says "drawer row click closes drawer, pans canvas, and selects card" but test only checks `onSelectApplication` fires — not the full integration chain.
**Why:** Subagents build components in isolation. They test the component's contract (props in, callbacks out) but can't test cross-component integration. The spec ACs often span multiple components.
**Prevention:** Session 9 integration tests must encode the spec ACs that span components. Component-level tests validate the building blocks; integration tests validate the assembled behavior. Flag ACs that require multi-component wiring during planning so they're explicitly deferred to integration, not assumed covered.
**Distilled to:** CLAUDE.md (Verification)

### L036 — CodeApplyModal test already existed — check before planning
**What:** Plan included Chunk 4 (CodeApplyModal test) as a deliverable. File already existed with 12 tests covering ARIA dialog, Escape key, form submission, loading/error states.
**Why:** Audit checked for test file existence at planning time but missed the admin/ directory. Session 8 agents had already built it.
**Prevention:** Before adding "write tests for X" to a plan, run `ls src/components/{dir}/*.test.tsx` to check existing coverage. The file list check should happen in the audit, not be deferred to the build agent.
**Distilled to:** CLAUDE.md (Verification)

### L037 — Worktree agents that don't commit leave uncommitted changes in the worktree directory
**What:** Agent A and C completed work in worktrees but didn't commit. Their changes existed as modified files in `.claude/worktrees/agent-xxx/` directories, not as git commits on worktree branches. Had to manually copy files instead of merging branches.
**Why:** Agents were told to verify (run tests, lint) but the prompt didn't explicitly say "commit your changes." The worktree isolation pattern assumes agents commit, but they don't by default.
**Prevention:** When dispatching worktree agents, end the prompt with explicit instructions: "After verification passes, stage and commit all changes with a descriptive message." This enables clean branch merging instead of manual file copying.
**Distilled to:** CLAUDE.md (Dispatch)

### L038 — ESLint picks up worktree copies during npm run check
**What:** `npm run check` reported 61 warnings — 20 pre-existing from src/ but duplicated 3x because ESLint also scanned `.claude/worktrees/agent-xxx/src/`. Misleading output suggests new issues when there are none.
**Why:** ESLint config doesn't exclude `.claude/` directory. Worktree copies are full repo clones, so ESLint finds all the same source files again.
**Prevention:** Add `.claude/` to ESLint ignore patterns (eslint.config.mjs or .eslintignore). Alternatively, clean up worktree directories before running lint. Both are worth doing — ignore prevents confusion, cleanup prevents stale files.
**Distilled to:** CLAUDE.md (Verification)

### L039 — Stubbed endpoints pass tests but hide integration failures
**What:** Instead of wiring components to the real endpoints defined in specs and wire contracts, placeholder/stub values were used. Tests were written against the stubs — so they passed — but the actual integration was never validated. Links pointed to `APP_URL` (bare domain) instead of `${APP_URL}/translate` as the specs intended.
**Why:** When backend endpoints aren't yet live or the FE is built ahead of the BE, the path of least resistance is stubbing. Tests then encode the stub values as "correct," creating a false green. The real contract is never tested until production.
**Prevention:** Two options Spencer is evaluating: (1) Bundle FE/BE deployments around shared dependency chunks so integration is validated per-chunk, or (2) Save all integration tests for end-of-sprint. Spencer leans toward option 1. Either way — tests must assert against the contract-defined values (from specs/wire contracts), not against whatever placeholder the component currently uses. If the endpoint isn't live yet, the test should still encode the correct target and the component should use the correct path, even if the request will 404 in dev.
**Distilled to:** pending (awaiting Spencer's decision on bundled deploys vs sprint-end integration)

### L040 — CSS Grid with hidden/shown elements breaks column count
**What:** TranslationCard used `md:grid-cols-2` with 4 child elements, where one was `md:hidden` and another was `hidden md:block`. On desktop, 3 visible elements in a 2-column grid caused the recruiter panel to wrap to row 2, leaving half the card as empty space ("half negative space on right").
**Why:** `display: none` removes elements from layout, so `md:grid-cols-2` with 3 visible children puts item 3 in row 2 col 1. The separator consumed one of the two columns, pushing the recruiter panel down. The visual result: candidate panel + separator in row 1, recruiter panel alone in row 2 with empty col 2.
**Prevention:** When a grid has show/hide children across breakpoints, count the VISIBLE children at each breakpoint and set `grid-cols-N` accordingly. If a separator element should appear between two panels, use `grid-cols-[1fr_1px_1fr]` (3 columns with explicit separator sizing) rather than `grid-cols-2` with a third element. The column template must match the number of visible children.
**Distilled to:** comp-R010

### L041 — Orientation hook blocks main window when session starts with agent dispatch
**What:** Main window dispatched 5 worktree agents as the first action. When agents completed and main window attempted direct edits (round 3 fixes), the `require-orientation.sh` hook blocked all 5 Edit calls — no marker file existed because the session never triggered a direct Read/Edit before dispatching.
**Why:** The orientation marker is created by `session-orient.sh` at session start, but if the main window's first actions are all Agent tool calls (no Read/Edit), the marker may not exist when direct edits finally happen mid-session. Worktree agents each have their own environment and don't create the marker for the main window.
**Prevention:** When a session will start with agent dispatch and follow up with direct edits, `touch` the orientation marker early — either in the bootstrap hook or as a deliberate first step after confirming docs were read (CLAUDE.md injected at session start counts). Alternatively, the hook could check for agent activity as a proxy for orientation.
**Distilled to:** pending

### L042 — Spec revision that adds items to a data array must grep for all hardcoded counts
**What:** Round 2 review added 3 second-person questions to FAQ_DATA (6 → 9 entries), but only updated the data constant. Six other locations hardcoded "6 questions" — acceptance criteria, test assertions, chunk descriptions, and a stale reference JSON-LD block. If built as-is, tests would assert `6 <dt>` elements but FAQ_DATA would produce 9, causing immediate test failure.
**Why:** Spec revision agents focused on the data change (adding questions to FAQ_DATA) without searching for downstream references to the old count. The stale JSON-LD reference block was particularly dangerous — it showed 6 hardcoded entries that an implementation agent would likely copy verbatim instead of generating from FAQ_DATA.
**Prevention:** When a spec revision adds or removes items from a data array/constant, the revision agent must grep the entire spec for the old count (e.g., `grep -n "6 questions\|6 <dt\|6 <dd\|with 6"`) and update every reference. Include this as a mandatory post-edit verification step in revision agent prompts: "After modifying any data array, grep the spec for the old item count and update all references."
**Distilled to:** CLAUDE.md (Dispatch)

### L043 — Bulk palette migration: subagent reads don't satisfy Edit's read gate
**What:** Attempted 17 parallel Edit calls on component files. All failed with "File has not been read yet" — even though an Explore subagent had just read every one of them.
**Why:** Edit tool requires the main conversation context to have read the file. Subagent reads don't transfer. With 25+ files needing the same mechanical find-replace, reading each one first would burn context for no benefit.
**Prevention:** For bulk mechanical find-replace across many files (like rgba `196, 101, 74` → `232, 168, 76`), use `sed -i` via Bash. Reserve Edit tool for targeted changes where understanding file context matters. If using Edit, read each file in the main context first.
**Distilled to:** comp-R015

### L044 — Design system migration requires exhaustive negative grep
**What:** After replacing all named tokens (Wave 1) and main rgba patterns (Wave 2), four files still had old Oxidized rgba values: admin components (no-space Tailwind classes), vocabulary-gap blockquote fallbacks, TranslationCard neutral color, and a UserActions hover state.
**Why:** Initial grep covered the most common patterns but missed variant forms — compact rgba in Tailwind classes (`rgba(196,101,74` vs `rgba(196, 101, 74`), fallback values in inline styles, and files not in the original plan's file list (UserActions, JDOptimizeOverlay, JDDashboard).
**Prevention:** After bulk replacement, run a comprehensive negative grep for ALL old RGB triplets (both spaced and unspaced) and iterate until zero results. The sweep pattern: `grep -rn 'rgba(OLD_R' src/ | grep -v specs/` — repeat for every old color. Don't trust the plan's file list as exhaustive.
**Distilled to:** comp-R016
