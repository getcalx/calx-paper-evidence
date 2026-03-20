# Cross-Domain vs Same-Domain Detection Matrix

Issue categories detected by each foil type across all review rounds (R2 dispatch on 10 FE specs, individual reviews of Specs 32-34).

## Matrix

| Issue Category | Backend Foil (Nolan) | Frontend Foil (Sloane) | Both |
|---|:---:|:---:|:---:|
| **Cross-domain (boundary width)** | | | |
| Wire format mismatches (field names) | X | | |
| Endpoint path errors (wrong URLs) | X | | |
| Response shape errors (missing TS fields) | X | | |
| Enum value mismatches | X | | |
| SSE contract issues (invented event types) | X | | |
| Auth/permission boundary assumptions | X | | |
| FE contract completeness on BE specs | X | | |
| **Same-domain (internal depth)** | | | |
| SQL/migration correctness | X | | |
| Column drift | X | | |
| RLS policy gaps | X | | |
| Function signature errors | X | | |
| Race conditions | X | | |
| UX flow completeness | | X | |
| Accessibility (ARIA, focus, keyboard) | | X | |
| Mobile layout gaps | | X | |
| Brand/copy violations | | X | |
| Performance targets missing | | X | |
| State completeness | | X | |
| Data representation correctness | X | | |
| Product logic errors | | X | |
| Deployment/operational | X | | |
| **Overlap categories** | | | |
| Error handling/mapping | | | X |
| Cross-spec contract drift | | | X |
| Cross-domain handoff gaps | | | X |

## Summary

- **Backend foil detected ~15 categories** — dominated by boundary/integration issues and backend-internal correctness
- **Frontend foil detected ~7 categories** — dominated by UX, accessibility, brand, and product alignment
- **Overlap: 3 categories** — error handling, contract drift, handoff gaps
- **Zero overlap on core findings** — wire format mismatches (Nolan) vs ARIA/mobile/brand patterns (Sloane)

## Source Evidence

- Backend foil findings: `../corrections/backend-foil/lessons.md` (entries dated 2026-03-12)
- Frontend foil systematic patterns: `per-spec-corrections/CROSS_CUTTING.md`
- Per-spec correction files: `per-spec-corrections/*.md` (17 files)
- Individual spec reviews: `per-spec-reviews/*.md` (8 files)
