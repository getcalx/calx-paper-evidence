# Classification Codebook: Architectural vs Process Corrections

**Version:** 1.0
**Date:** 2026-04-01
**Purpose:** Operational decision procedure for classifying corrections as architectural or process. Designed for inter-rater reliability testing.

---

## 1. Decision Procedure

At the time a correction is captured, apply this single question:

**Does the intervention change the structure of the system (code, configuration, infrastructure, hooks) such that the error class becomes mechanically impossible regardless of whether any agent remembers a rule about it?**

- **Yes** = Architectural correction
- **No** = Process correction

The classification depends on what the correction CHANGES, not on what happens afterward. Recurrence data validates the classification but does not define it.

---

## 2. Operational Definitions

### Architectural Correction
An intervention that modifies the system's structure so that an error class cannot recur regardless of agent behavior. After the correction is applied, the error is mechanically impossible.

**Indicators:**
- A code change (library replacement, function consolidation, structural refactor)
- A configuration change (environment variable, build setting, deployment config)
- A hook or gate installation (pre-commit hook, shell hook, CI check)
- A type constraint or validation layer (type system enforcement, import guard)
- A test that blocks the error from reaching production

**Key test:** If every agent in the system had its memory wiped, would the error still be prevented? If yes, the correction is architectural.

### Process Correction
An intervention that adds a text-based rule the agent must remember and execute. The environment is unchanged; only the instruction set is modified.

**Indicators:**
- A rule added to a rules file (CLAUDE.md, AGENTS.md, domain rules)
- A procedural instruction ("always do X before Y")
- A reminder or emphasis of an existing principle
- A behavioral guideline that depends on agent recall

**Key test:** If the rule were removed from the agent's context, would the error be free to recur? If yes, the correction is process.

---

## 3. Worked Boundary Examples

### Example 1: Import-time settings loading (ARCHITECTURAL)

**Error:** Agent imports Django settings module at file top level, causing circular import.
**Correction:** Moved settings import inside function body; added an import guard that raises an error if settings are imported at module level.
**Classification:** Architectural. The import guard prevents the error mechanically. No agent needs to remember the rule.
**Recurrence:** Zero (validated).

### Example 2: Test mock patching (PROCESS)

**Error:** Agent uses incorrect mock patch target (patches the definition site instead of the usage site).
**Correction:** Rule added: "Always patch where the name is looked up, not where it is defined."
**Classification:** Process. The correction is a procedural instruction. The environment permits the error; only the agent's recall prevents it.
**Recurrence:** 8 entries (L017 through L095), validated.

### Example 3: Async/sync blocking (ARCHITECTURAL)

**Error:** Agent calls synchronous database operations inside async request handlers, blocking the event loop.
**Correction:** Replaced synchronous ORM calls with async equivalents; added a runtime check that raises an error if sync database calls are detected in async contexts.
**Classification:** Architectural. The runtime check prevents the error mechanically.
**Recurrence:** Zero (validated).

### Example 4: Schema/model drift (PROCESS)

**Error:** Agent modifies a database model without updating the corresponding API schema, creating a mismatch.
**Correction:** Rule added: "When modifying any model field, update the corresponding schema in the same commit."
**Classification:** Process. The correction depends on the agent remembering to check both locations. The environment does not enforce co-modification.
**Recurrence:** 5 entries (validated).

### Example 5: Duplicate model definitions (BORDERLINE, classified ARCHITECTURAL)

**Error:** Agent creates a new model class that duplicates an existing one, causing namespace conflicts.
**Correction:** Added a grep-based verification check that scans for duplicate class names before allowing model creation.
**Classification:** Architectural (borderline). The grep check is an environmental mechanism, but it depends on being run (it is a script, not an import-time constraint). The mechanism sits at the boundary between environmental enforcement and procedural recall.
**Recurrence:** Zero (validated, consistent with architectural classification).
**Note:** This is the 1 of 11 cases where classification is ambiguous. A second rater might reasonably classify this as process. The grep check enforces, but only if the check is invoked.

### Example 6: Spec before building (PROCESS, Study 2)

**Error:** Agent begins implementation without writing or reading a specification.
**Correction:** Rule added: "Always read or write a spec before beginning implementation."
**Classification:** Process. The environment does not prevent an agent from starting to code. Only the agent's recall of the rule prevents the error.
**Recurrence:** 3 entries in Study 2 (Mara Mar 22, Thane x3 culminating Apr 1), validated.

---

## 4. Classification Rules for Edge Cases

1. **If the correction includes both a structural change AND a rule:** Classify based on the structural change. The rule is supplementary.

2. **If the structural change requires being invoked (not automatic):** Flag as borderline. Classify based on whether invocation is gated (automatic = architectural) or discretionary (discretionary = borderline).

3. **If the correction modifies test infrastructure:** Classify as architectural if the test runs automatically (CI, pre-commit). Classify as process if the test must be manually invoked.

4. **If uncertain:** Classify as process. The conservative default is process because architectural classification makes the stronger claim (zero recurrence expected).

---

## 5. Rater Instructions

For inter-rater reliability testing:

1. Read the correction entry (failure description, root cause, correction action)
2. Apply the decision question in Section 1
3. If the answer is unclear, consult the edge case rules in Section 4
4. Record your classification and confidence (high/medium/low)
5. For low-confidence classifications, note the specific ambiguity

Do NOT read recurrence data before classifying. The criterion is prospective. Recurrence data is used only for validation after classification is complete.

---

## 6. Validation Results

Across 11 error classes examined in Study 1:
- 6 architectural: 6/6 zero recurrence (100% validated)
- 5 process: 5/5 recurrence observed (100% validated)
- 1 borderline case (Example 5 above): classified architectural, zero recurrence observed
- Fisher's exact test on the 6-0 vs 0-5 contingency: p = 0.002
