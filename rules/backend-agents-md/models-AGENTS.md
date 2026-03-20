# AGENTS.md — Model Layer Conventions

Rules for working in `src/soaria/models/`. Distilled from Lessons L004, L019.

Rule IDs: mdl-R001 through mdl-R009 | Last reviewed: 2026-03-14

### mdl-R001: No Duplicate Class Names
Source: L004 | Added: 2026-03-02 | Status: active

Python silently overwrites duplicate class definitions in the same file. The second
definition replaces the first with zero warnings. (L004)

Before writing any model class, check for existing definitions:
```bash
rg "^class " src/soaria/models/<file>.py | sort | uniq -d
```

If this returns anything, you have a duplicate. Fix it before proceeding.

### mdl-R002: Never Copy-Paste Models
Source: L004 | Added: 2026-03-02 | Status: active

Write each model from scratch or use inheritance. Copy-pasting and modifying is
how L004 happened — a later session re-defined classes that already existed.

### mdl-R003: Schema Verification
Source: Spec #8 | Added: 2026-03-02 | Status: active

Before referencing a database column in a model, verify it exists in:
1. `specs/SOARIA_V7_DATA_CONTRACTS.md` (canonical schema)
2. The spec's prerequisite SQL (for columns added by that spec)

### mdl-R004: Common Schema Drift
Source: Spec #8 | Added: 2026-03-02 | Status: active

Watch for these known mismatches between code and schema:
- `employee_count` (int) vs `employee_count_range` (text) — schema uses the range
- Columns that exist in JSONB but not as standalone columns
- Columns added by spec prerequisites that haven't been migrated yet

When in doubt, check the actual database schema, not just the data contracts doc.

### mdl-R005: Test Fixtures Match Model Constraints
Source: L019 | Added: 2026-03-02 | Status: active

Every fixture field must be verified against both the Pydantic model AND the DB
schema. Required `NOT NULL` fields with `DEFAULT NOW()` must have realistic
timestamps, never `None`. (L019)

```python
# Wrong
{"updated_at": None}

# Right
{"updated_at": "2026-03-01T00:00:00+00:00"}
```

### mdl-R006: Bounded Text Fields on Request Models
Source: L045 | Added: 2026-03-06 | Status: active

All user-facing text fields in request models must have `Field(max_length=N)`.
Guidelines: titles/names 500, reasons 1K, chat messages 5K, free-text 10K,
STAR sections 50K. Response models are exempt (they reflect DB state). (L045)

### mdl-R007: Migration Columns Require Matching Pydantic Response Fields
Source: L059 | Added: 2026-03-14 | Status: active

Every migration column addition requires a corresponding Pydantic response model
field AND a TypeScript contract field update. Pydantic silently drops fields the
model doesn't declare (no error, no log). Every spec chunk that touches migration
DDL must explicitly list the Pydantic model changes in the same chunk. (L059)

### mdl-R008: Cross-Module Values Must Be Constants or Enums
Source: L065 | Added: 2026-03-14 | Status: active

Values that cross module boundaries (metadata types, event names, status strings)
must be defined once as a constant or enum and imported everywhere. Never hardcode
matching strings in separate files. Spec 27 had `"follow_up_reminder"` in one module
and `"reminder"` in another — feature failed silently. (L065)

### mdl-R009: Sentinel Values for Unlimited Tiers, Not Nullable Integers
Source: L067 | Added: 2026-03-14 | Status: active

Use sentinel value (999999999) for "unlimited" in quota/limit systems. Avoids
nullable columns, None comparison guards, and `TypeError` from `int >= None`.
The comparison `usage >= 999999999` just works. Document the sentinel in the
constants file alongside tier config. (L067)
