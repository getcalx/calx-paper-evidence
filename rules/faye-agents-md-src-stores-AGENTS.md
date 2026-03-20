# AGENTS.md — src/stores/

Client-side state management (Zustand or similar).

## Rules

### store-R001 — Typed store slices
Every store must have explicit TypeScript interfaces for its state and actions.
- **Source:** CLAUDE.md critical rule #5
- **Added:** 2026-03-10
- **Status:** active

### store-R002 — No server state in stores
Server data (jobs, resumes, user profile) should be managed by the data fetching
layer (React Query or similar), not duplicated in client stores.
Client stores are for UI state: canvas positions, expanded cards, chat input, etc.
- **Added:** 2026-03-10
- **Status:** active
