# AGENTS.md — src/hooks/

Rules for agents working on custom React hooks in this directory.

## Rules

### hook-R001 — Hooks handle SSE resilience
Any hook consuming SSE streams must implement auto-reconnect on drop
and graceful degradation (show last known state + reconnecting indicator).
- **Source:** CLAUDE.md critical rule #7
- **Added:** 2026-03-10
- **Status:** active

### hook-R002 — Typed return values
All hooks must have explicit return type annotations. Never rely on inference
for public hook APIs.
- **Source:** CLAUDE.md critical rule #5
- **Added:** 2026-03-10
- **Status:** active

### hook-R003 — Lazy state initializer for browser APIs
When a hook reads a browser API (e.g., `window.matchMedia`) to set initial state,
use `useState(() => browserAPI())` lazy initializer. Don't use
`useEffect(() => setState(browserAPI()))` — that causes an unnecessary re-render.
The effect should only handle subscription and cleanup.
- **Source:** L021
- **Added:** 2026-03-14
- **Status:** active

### hook-R004 — useCallback before useMemo that depends on it
When wiring a callback into a `useMemo`, define the `useCallback` ABOVE the
`useMemo` in the component body and include it in the dependency array. JavaScript
hoisting doesn't apply to hooks — they must be called in order.
- **Source:** L027
- **Added:** 2026-03-14
- **Status:** active

### hook-R005 — Null-returning validators lose failure-mode granularity
When a caller needs to distinguish multiple failure modes (missing vs. malformed
vs. expired), check preconditions before calling a validation function that returns
`null`. Don't rely on a null-returning helper to encode *why* it returned null.
- **Source:** L017
- **Added:** 2026-03-14
- **Status:** active

### hook-R006 — Mock matchMedia in test files for responsive hooks
jsdom doesn't implement `window.matchMedia`. Any component or hook using media
queries needs a `matchMedia` mock at the top of the test file or in shared test
setup. This is a one-time pattern per hook but easy to forget.
- **Source:** L020
- **Added:** 2026-03-14
- **Status:** active
