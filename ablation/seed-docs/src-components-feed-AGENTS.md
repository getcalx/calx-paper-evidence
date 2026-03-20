# AGENTS.md — src/components/feed/

Rules and conventions for feed components.

---

### Rule: Fit badge color mapping is authoritative
- **Rule:** `FitBadge` defines the canonical color mapping for fit labels: `highly_recommended`=green, `strong_fit`=blue, `good_match`=amber, `possible_fit`=slate. Unknown labels fall back to `possible_fit` styling. All components displaying fit scores must use `FitBadge` — don't roll your own color logic.
- **Verification:** Grep for `fit_label` or `fit_score` usage — should only render through `FitBadge`.
- **Source:** Spec 19

### Rule: JobCard handles its own view tracking
- **Rule:** `JobCard` uses IntersectionObserver internally to detect visibility and fires `onView` after 1 second at 50%+ visibility. The card skips tracking for already-viewed jobs (`is_viewed === true`). Callers pass `onView` but don't manage the observer — the card owns that lifecycle.
- **Verification:** `onView` fires exactly once per card per mount, deduplication handled by `hasTrackedView` ref.
- **Source:** Spec 19

### Rule: Save/pass are one-way in the feed
- **Rule:** Save and pass buttons on `JobCard` are one-way actions — save disables after click, pass hides the button. The `JobDetail` component supports unsave (toggle). Note: the backend now supports unsave/unpass via `DELETE /v1/jobs/{id}/interactions/{type}` (HANDOFF update 2026-03-06), but the frontend card UI hasn't been updated to use it yet.
- **Verification:** Card save button has `disabled={saving || job.is_saved}`. Pass button renders `{!job.is_passed && ...}`.
- **Source:** Spec 19, HANDOFF Q2

### Rule: Salary formatting fallback chain
- **Rule:** Display `salary_string` if available. Otherwise format `min_annual_salary`/`max_annual_salary` as "$XXXk - $XXXk". If only min exists: "$XXXk+". If only max: "Up to $XXXk". If no data: hide the salary line entirely.
- **Verification:** `formatSalary()` helper in job-card.tsx and job-detail.tsx.
- **Source:** Spec 19

### Rule: Passed jobs filtered client-side
- **Rule:** The backend does NOT filter passed jobs from `GET /v1/feed`. The feed page filters them out client-side using the `is_passed` flag before rendering. This is a known workaround — the backend TODO is to add server-side filtering.
- **Verification:** Feed page has `allJobs.filter((job) => !job.is_passed)`.
- **Source:** Spec 19, HANDOFF Q5

### Rule: Job description rendered as HTML
- **Rule:** `JobDetail` renders `job.description` via `dangerouslySetInnerHTML`. The content comes from job postings and may contain HTML. Style with `prose prose-invert` classes for readability.
- **Verification:** `dangerouslySetInnerHTML={{ __html: job.description }}` in job-detail.tsx.
- **Source:** Spec 19

### Rule: Apply button opens optimization modal, not ATS directly
- **Rule:** The Apply button on `JobDetail` does NOT open `job.url` directly. It checks the active focus area's `baseline_resume_id` — if null, shows `NoBaselineDialog` (links to Career Profile). If set, records `clicked_apply` interaction and opens `OptimizationModal` with job context. The modal's approval flow handles the actual ATS redirect.
- **Verification:** Click Apply on a job — optimization modal opens (not external URL). Without baseline resume — dialog prompts setup.
- **Source:** Spec 19 Addendum

### Rule: Resume gating banner is per-session dismissible
- **Rule:** When user has zero resume templates, a `ResumeGatingBanner` appears at the top of the feed. Dismissal uses `useState` (not persisted) — returns on next session. The banner does not block feed access, only informs.
- **Verification:** Banner shows for users with no resumes. Dismiss — stays dismissed until page remount.
- **Source:** Spec 21 Addendum
