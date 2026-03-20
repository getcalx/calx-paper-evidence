# AGENTS.md — src/components/profile/

Rules and conventions for profile components.

---

### Rule: Section edit pattern
- **Rule:** Use the `ProfileSection` wrapper for any profile card that supports view/edit toggling. It handles the edit button, save/cancel footer, and loading state. Individual sections provide view content as `children` and edit form as `editForm` prop.
- **Verification:** Every editable section in the profile page should use `<ProfileSection>`.
- **Source:** Spec 21

### Rule: Form ref pattern for external submit
- **Rule:** Profile edit forms expose a `formRef` callback that provides a `{ submit: () => void }` handle. The parent (profile page) stores these in a `useRef` map and calls `formRef.current[section].submit()` when the ProfileSection save button is clicked. This avoids nesting `<form>` elements.
- **Verification:** Code review — form submit must be triggered via ref, not a submit button inside ProfileSection.
- **Source:** Spec 21

### Rule: Only one section edits at a time
- **Rule:** The profile page tracks `editingSection` as a single `SectionKey | null`. Starting edit on one section automatically closes any other. This prevents conflicting mutations.
- **Verification:** Only one `isEditing={true}` should exist in the profile page at any time.
- **Source:** Spec 21

### Rule: Focus area PUT replaces all
- **Rule:** The focus area, location preference, and deal breaker APIs are full-replacement PUTs. The editor must maintain the full array in local state and submit all items, not just the changed one. Optimistic updates replace the entire array.
- **Verification:** Mutation calls send the complete array, not a single item.
- **Source:** Spec 21, HANDOFF.md

### Rule: Max 3 focus areas
- **Rule:** Backend enforces a maximum of 3 focus areas. The "Add Focus Area" button must be disabled when 3 exist. Show a message explaining the limit.
- **Verification:** Button has `disabled={focusAreas.length >= 3}`.
- **Source:** Spec 21, backend answer #14

### Rule: Dialog form reset on close
- **Rule:** Dialog forms (focus area, location preference) must call `reset()` when the dialog closes to clear stale values. Handle this in the `onOpenChange` callback.
- **Verification:** Open add dialog, fill fields, cancel, reopen — fields should be empty.
- **Source:** Spec 21

### Rule: Resume templates section is read-only
- **Rule:** The `ResumeTemplatesSection` on the profile page displays templates uploaded by admins. Users cannot upload/delete templates themselves. Show template name, job_focus badge, active status, and created date. Empty state: "Your admin will upload your baseline resumes."
- **Verification:** No upload/delete buttons in the templates section. Empty state message visible for users with no templates.
- **Source:** Spec 21 Addendum

### Rule: Focus area baseline_resume_id links to templates
- **Rule:** The focus area editor's `baseline_resume_id` field is a select populated from `useResumeTemplates()`. Options show `template_name (job_focus)`. "None" option clears to null. Disabled when no templates exist. This replaced the earlier "coming soon" placeholder.
- **Verification:** Open focus area editor — baseline resume dropdown shows available templates (or is disabled).
- **Source:** Spec 21 Addendum
