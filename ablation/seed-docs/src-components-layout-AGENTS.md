# AGENTS.md — src/components/layout/

Rules and conventions for layout components (sidebar, mobile nav, shell).

---

### Rule: Sidebar and mobile nav use path-prefix matching
- **Rule:** Active route detection uses `pathname.startsWith(href)` so that sub-routes (e.g., `/feed/saved`) highlight the parent nav item.
- **Verification:** Navigate to a sub-route and confirm the parent item is highlighted.
- **Source:** Spec 18

### Rule: Admin-only UI is prop-gated
- **Rule:** Admin-only elements (admin nav link, LLM spend indicator) are controlled by an `isAdmin` prop. The actual role check comes from the user profile API (Spec 21). Never hardcode admin visibility.
- **Verification:** Render sidebar with `isAdmin={false}` — admin link and LLM indicator must not appear.
- **Source:** Spec 18

### Rule: Mobile nav excludes admin link
- **Rule:** The mobile bottom nav does not include the Admin link. Admins use the desktop sidebar.
- **Verification:** Inspect mobile nav — should have exactly 4 tabs (Feed, Resumes, Applications, Profile).
- **Source:** Spec 18

### Rule: Layout passes user and isAdmin from session
- **Rule:** `(app)/layout.tsx` is responsible for reading the user session and passing `user` and `isAdmin` props to the sidebar. Currently stubbed with `null`/`false` — wired in Spec 21.
- **Verification:** Check that layout.tsx provides props to `<Sidebar>`.
- **Source:** Spec 18
