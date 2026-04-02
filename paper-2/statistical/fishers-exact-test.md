# Fisher's Exact Test: Correction Type vs Recurrence

**Computed:** 2026-04-01
**Data source:** 11 error classes from Study 1 (Hardwick 2026a)

---

## Contingency Table

|  | Zero recurrence | Recurrence observed | Total |
|---|---|---|---|
| **Architectural** | 6 | 0 | 6 |
| **Process** | 0 | 5 | 5 |
| **Total** | 6 | 5 | 11 |

## Error Classes

### Architectural (zero recurrence)
1. Import-time settings loading
2. Async/sync blocking
3. Duplicate model definitions (borderline, see codebook)
4. LLM output validation
5. Deployment configuration drift
6. Anonymous path gaps

### Process (recurrence observed)
1. Test mock patching: 8 entries (L017, L026, L031, L077, L084, L087, L090, L095)
2. Schema/model drift: 5 entries
3. Cross-boundary contracts: 5 entries
4. Router-not-mounted: 3 entries
5. Flow/caller-not-updated: 4 entries

## Computation

Fisher's exact test computes the probability of observing a table as extreme as or more extreme than the observed table under the null hypothesis that correction type and recurrence are independent.

For the 2x2 table:
- a = 6 (architectural, zero recurrence)
- b = 0 (architectural, recurrence)
- c = 0 (process, zero recurrence)
- d = 5 (process, recurrence)
- n = 11

P(observed) = C(6,6) * C(5,5) / C(11,6) = (1 * 1) / 462 = 0.00216

**p = 0.002 (one-tailed)**

The probability of observing a perfect 6-0 vs 0-5 split by chance (if type and recurrence were independent) is 0.2%.

## Interpretation

The prospective classification criterion (applied at capture time based on intervention type) perfectly predicts recurrence outcome across all 11 error classes. The separation is statistically significant at p < 0.01.

**Caveat:** The test validates the classification/recurrence association. It does not prove the classification criterion is the correct one, nor does it establish causation. The borderline case (duplicate model definitions) would change the table to 5-1 vs 1-4 if reclassified, yielding p = 0.015 (still significant at p < 0.05).
