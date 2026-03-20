# AGENTS.md — src/types/

TypeScript type definitions. Backend contract types live here.

## Rules

### type-R001 — Backend types come from contracts
Types for API request/response shapes come from the coordination contracts
(`read_contract("types.ts")`). Do not invent response shapes — sync from contracts.
- **Source:** CLAUDE.md backend integration
- **Added:** 2026-03-10
- **Status:** active

### type-R002 — No `any` types
Every type must be fully specified. Use `unknown` + type guards if the shape
is genuinely unknown at compile time, but never `any`.
- **Source:** CLAUDE.md critical rule #5
- **Added:** 2026-03-10
- **Status:** active

### type-R003 — Wire types mirror producer field names exactly
When writing any TypeScript interface that touches a backend contract, read the
producer's published model first (contracts/types.ts). The wire format uses the
producer's field names, types, formats, and TTLs. Build a mapper on the FE side
if semantic naming differs, but the wire type must match the producer exactly.
- **Source:** L008
- **Added:** 2026-03-14
- **Status:** active

### type-R004 — Canvas node types use base Node, not a single generic
When a ReactFlow canvas has multiple node types with different data shapes (card
nodes vs. identity nodes), use the base `Node` type for the flow nodes array. Let
each node type's wrapper component handle its own data typing via the `nodeTypes`
registry. Don't force all nodes into a single generic.
- **Source:** L019
- **Added:** 2026-03-14
- **Status:** active

### type-R005 — Intersection types prevent canvas data type drift
When a canvas data type represents the same data as a display type, define it as
an intersection: `type JobCardData = JobMatchDisplay & { type: 'job_card' }`. The
discriminator enables the CardData union while the intersection ensures field
changes propagate automatically. Never duplicate fields in a standalone interface.
- **Source:** L026
- **Added:** 2026-03-14
- **Status:** active

### type-R006 — Double-cast for version-compatibility field checks
When a mapper needs to check for fields that may or may not exist on a typed object
(e.g., checking both `user_language` and `source_phrase`), use
`as unknown as Record<string, unknown>` (double cast through unknown). This is an
intentional escape hatch for version compatibility — document why in a comment.
- **Source:** L029
- **Added:** 2026-03-14
- **Status:** active
