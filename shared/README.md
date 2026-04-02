# Shared Evidence: Methodology and Correction Format

Artifacts shared across Papers 1, 2, and 3.

---

## Correction Capture Format

Each correction entry records six fields:

1. **Failure:** What happened, in what context
2. **Root cause:** The structural condition that permitted the error (not just the surface error)
3. **Correction:** The concrete behavioral change prescribed
4. **Distillation target:** Which rule store should receive the derived rule
5. **Recurrence chain:** Links to prior corrections addressing the same error class
6. **Date and specification identifiers:** For temporal and per-specification analysis

### Study 1 Format
Append-only markdown files, one per active domain. Format: `L{NNN}` entries with structured metadata.

### Study 2 Format
Structured SQLite database (calx.db) with the same six fields. Entries numbered `C{NNN}`.

## Classification Criterion

See `paper-2/classification/codebook.md` for the full operational decision procedure.

**Single question:** Does the intervention change the system's structure such that the error class becomes mechanically impossible regardless of whether any agent remembers a rule about it?

- Yes = Architectural
- No = Process

Applied prospectively at capture time. Validated (not defined) by recurrence data.

## Agent Roster

### Study 1 (6 agents)
| Agent | Role | Domain |
|---|---|---|
| Bryce | Backend builder | soaria-backend |
| Faye | Frontend builder | versed_fe |
| Nolan | Cross-domain reviewer | Assigned outside expertise |
| Sloane | Same-domain reviewer | Assigned within expertise |
| Reid | Orchestrator | Coordination |
| Prism | Lens rotator | Strategy dispatch |

### Study 2 (3 agents, Reid spans both)
| Agent | Role | Domain |
|---|---|---|
| Reid | Orchestrator | Coordination (spans both studies) |
| Thane | Backend builder | calx-serve |
| Mara | OSS packaging | getcalx |

### Combined: 8 unique agents across 43 days, 2 products
