# AGENTS.md — src/app/(app)/

Rules and conventions for the authenticated app route group.

---

### Rule: Content area is capped at 1280px
- **Rule:** The app layout wraps children in `max-w-[1280px]` with responsive padding (`px-4 py-6 sm:px-6 lg:px-8`). Individual pages should not add their own max-width wrapper.
- **Verification:** Inspect rendered page — content should not exceed 1280px.
- **Source:** Spec 18

### Rule: Mobile bottom padding
- **Rule:** The `<main>` element has `pb-16 lg:pb-0` to prevent content from being hidden behind the fixed mobile bottom nav.
- **Verification:** Scroll to bottom of page on mobile viewport — last content should be visible above the nav bar.
- **Source:** Spec 18

### Rule: Loading uses skeleton patterns, not spinners
- **Rule:** Route-level loading states use shadcn `Skeleton` components. No spinners. The app-level `loading.tsx` renders a skeleton card grid.
- **Verification:** Trigger a route transition — skeleton UI should appear, not a spinner.
- **Source:** Spec 18

### Rule: Error boundary logs and offers retry
- **Rule:** `error.tsx` logs the error to console, shows a user-friendly message, and provides a "Try again" button that calls `reset()`. Never show raw error messages or stack traces.
- **Verification:** Force an error — boundary should render with retry button.
- **Source:** Spec 18

### Rule: Onboarding gate via AppShell
- **Rule:** The `(app)/layout.tsx` renders `<AppShell>`, a client component that fetches the user profile and enforces three gates: (1) unonboarded users redirect to `/profile/onboarding`, (2) `pending_review` users redirect to `/pending-approval`, (3) approved users see the full app with sidebar. The onboarding and pending pages render with a minimal layout (no sidebar).
- **Verification:** Navigate to any `(app)` route without completing onboarding — should redirect to `/profile/onboarding`.
- **Source:** Spec 21

### Rule: Layout stays server component
- **Rule:** `layout.tsx` must remain a server component. Client-side logic (hooks, redirects, user context) goes in the `AppShell` client wrapper. This prevents forcing the entire subtree into client rendering.
- **Verification:** `layout.tsx` does NOT have `"use client"` directive.
- **Source:** Spec 21, L003

### Rule: Profile data available via useProfile()
- **Rule:** Any `(app)` page can call `useProfile()` from `@/hooks/use-profile` to access the current user's profile. The query is already warm from the AppShell gate check. Do NOT re-fetch the profile independently — use the shared query key `['profile']`.
- **Verification:** No direct `api.get('/v1/me')` calls in page components.
- **Source:** Spec 21

### Rule: Pending approval page is a dead end
- **Rule:** The `/pending-approval` page has no sidebar and only two actions: "Check Status" (navigates to `/feed`, which re-triggers the gate) and "Sign Out". Users in `pending_review` status cannot access any other app page.
- **Verification:** Manual test — pending_review user should only see the interstitial.
- **Source:** Spec 21, backend answer #1

### Rule: Admin sub-layout provides its own guard and context
- **Rule:** `/admin/layout.tsx` is a nested layout that checks admin role via `useProfile()` and provides `AdminContext` to all admin pages. Non-admins are redirected to `/feed` with an error toast. Admin pages use `useAdminContext()` for the current admin user — they don't call `useProfile()` again.
- **Verification:** Navigate to `/admin` as non-admin — redirected to `/feed`. As admin — layout renders with context available.
- **Source:** Spec 23a

### Rule: Full-screen modals overlay above the layout
- **Rule:** The optimization modal renders as `fixed inset-0 z-50` — it covers the entire viewport including sidebar. It is mounted in the job detail page component, not in the layout. Escape key closes (with unsaved edit confirmation). Background scroll is not explicitly locked.
- **Verification:** Open optimization modal — it covers the sidebar. Press Escape — confirmation or close.
- **Source:** Spec 20
