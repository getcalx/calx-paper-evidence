# Contract Corrections: Subscription & Trial UI

Source: Nolan (Backend Foil) cross-review

---

## CRITICAL: Tier Enum Mismatch

```ts
// WRONG (spec):
tier: "free" | "pro"

// RIGHT (BE Spec 25):
tier: "free" | "trial" | "pro" | "founding_member"
```

FE must handle all 4 values. If BE returns `"trial"` or `"founding_member"`, current `tier === "free"` and `tier === "pro"` checks both return false → UI limbo.

## CRITICAL: Status Enum Mismatch

```ts
// WRONG (spec):
status: "trialing" | "active" | "cancelled" | "free"

// RIGHT (BE Spec 25):
status: "free" | "trialing" | "pro" | "founding_member" | "past_due" | "cancelled"
```

- `"active"` does NOT exist on backend
- Add `"past_due"` handling → PaymentIssueBanner with Stripe portal link
- Add `"founding_member"` handling → treat as equivalent to "pro" for feature gating
- `status === "active"` checks → `status === "pro" || status === "founding_member"`

## CRITICAL: Unlimited Sentinel Mismatch

```ts
// WRONG: -1 means unlimited
optimizations_limit: -1
applications_limit: -1

// RIGHT: null means unlimited
optimizations_limit: number | null  // null = unlimited
applications_limit: number | null   // null = unlimited
```

## No SubscriptionSummary in Published Contracts

`SubscriptionSummary` exists in BE Spec 25 but NOT in `contracts/types.ts` or `api-spec.json`. File RFC to publish it. Mark FE types as provisional.

## Data Source Ambiguity

- Spec reads from `GET /v1/me`
- BE Spec 25D defines `GET /v1/billing/subscription` as canonical endpoint
- **Decide and document which one.** After Stripe redirect, specify which gets refetched.

## Wire Fields FE Ignores

Backend sends `is_pro: bool` and `show_pre_commitment: bool` — FE ignores both and re-implements the logic client-side. Use the BE flags instead.

## Endpoint Fixes

```
// WRONG: PreCommitmentPrompt calls
POST /v1/billing/checkout { plan: "founding_member" }

// RIGHT: should call
POST /v1/billing/pre-commit
// This applies the 90-day free coupon. /checkout does not.
```

Add `createPreCommitSession()` to subscription-api.ts.

## Stripe Redirect Fix

```
// WRONG: redirects to /settings?billing=success
// PROBLEM: /settings route doesn't exist

// FIX: redirect to /?billing=success (canvas root)
// Add query param handler to canvas shell for subscription refetch + success toast
```

## Content Fixes

- Day 12 message references `optimizationsUsed` — always -1, meaningless. Remove. Use applications + proactive insight count instead.
- 90-day founding member free period completely absent from UI. Add to PreCommitmentPrompt: "First 90 days free — no charge until [date]." Add to UpgradeModal founding member column.
- PreCommitmentPrompt uses "Lock in" (loss framing) → change to "Join as a founding member" (agency framing per brand review)
- STAR story capture listed as "never gated" but PRD says Pro-only → fix feature gating table
- `founding_eligible` boolean needed for UpgradeModal pricing display (distinct from `isFoundingMember` which is post-conversion)

## Missing Endpoint

`GET /v1/feed/preview-count` doesn't exist. File RFC. Stub the function to return 0 until published.
