# AGENTS.md — Dispatch & Session Conventions

Rules for agent orchestration, worktree dispatch, and session hygiene. Distilled
from Specs 32/32A/32B build sessions and process corrections (L083–L092).

Rule IDs: dispatch-R010 through dispatch-R013, session-R001 through session-R003 | Last reviewed: 2026-03-15

### dispatch-R010: Worktree agents create only, never edit
Source: L083 | Added: 2026-03-15 | Status: active

Worktree agents are reliable for creating new files but silently fail on edits to
existing files (copy originals without changes). All edits to existing files
(config.py, router.py, __init__.py, existing tools/prompts) must happen in the main
window, where the orchestrator can verify changes directly.

### dispatch-R011: Test agent prompts must include implementation source
Source: L084 | Added: 2026-03-15 | Status: active

Include the import statements and key function signatures from the actual
implementation files (not just spec-level descriptions). Lazy imports
(`from x import y as z` inside function bodies) are invisible from signatures
alone and cause mock target mismatches. Include the first 30 lines of each
implementation file in the test agent prompt.

### dispatch-R012: Single-session dispatch for independent chunks
Source: L085 | Added: 2026-03-15 | Status: active

When the chunk dependency graph allows 3+ parallel dispatches and chunks are
file-disjoint, prefer single-session worktree dispatch over multi-session builds.
Main window stays lean doing orchestration; context pressure comes from code work,
not dispatch. Reserve multi-session for specs where chunks share files.

### dispatch-R013: Verify between dispatch rounds
Source: L086 | Added: 2026-03-15 | Status: active

After copying files from each agent round, verify: (1) all new modules import
cleanly, (2) router mounts are registered (for API chunks), (3) no duplicate class
names in models. Fix issues before dispatching the next round — prevents compound
failures.

### session-R001: Update memory at session start and end
Source: L088 | Added: 2026-03-15 | Status: active

Every session start: read and verify memory against actual state (git log, branch
status). Every session end: update memory files to reflect what was built, merged,
or changed. Spencer should never have to track state for you. Stale memory that
presents outdated claims as fact is a trust violation.

### session-R002: Apply migrations to live DB as part of spec completion
Source: L091 | Added: 2026-03-15 | Status: active

Every spec that includes a migration .sql file MUST end with applying it to the
live Supabase database via `mcp__supabase__apply_migration`. Verify with
`mcp__supabase__list_migrations`. A committed .sql file that hasn't been applied
is a silent failure — downstream flows, API endpoints, and bootstraps will break
against a DB that doesn't have the tables. This is not optional and not deferred
to "deployment" — Bryce owns the live schema.

### session-R003: Update data contracts when populating previously-empty tables
Source: L092 | Added: 2026-03-15 | Status: active

When a flow or service populates a table for the first time (or changes the
semantics of existing data), update `~/soaria-coordination/contracts/types.ts`
even if the SQL schema is unchanged. "No schema change" != "no contract change."
A table going from empty to populated is a functional change — downstream
consumers (Faye) need to know the shape of data they'll now receive. Check:
does this make new data _available_ that wasn't before?
