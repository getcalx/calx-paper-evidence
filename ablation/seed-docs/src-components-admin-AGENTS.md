# AGENTS.md — src/components/admin/

Rules and conventions for admin components.

---

### Rule: Admin guard lives in the layout
- **Rule:** Role checking happens in `(app)/admin/layout.tsx`, not in individual pages. The layout fetches `useProfile()`, checks `role === "admin"`, and redirects non-admins to `/feed` with an error toast. Child pages access the admin user via `useAdminContext()` — never re-fetch the profile.
- **Verification:** No admin page imports `useProfile` directly. All use `useAdminContext()` for admin identity.
- **Source:** Spec 23a

### Rule: Confirmation dialogs for destructive admin actions
- **Rule:** Status change to "suspended" and all role changes require an `AlertDialog` confirmation. Config type: `{ title, description, onConfirm }`. Suspending warns about access loss. Role changes warn about access gain/loss. Use `useState<ConfirmationConfig | null>` to control dialog visibility.
- **Verification:** Click suspend or change role — confirmation dialog must appear before mutation fires.
- **Source:** Spec 23a

### Rule: Self-demotion guard (UI-side)
- **Rule:** When viewing your own admin profile, the role select is disabled with `opacity-50` and helper text "Cannot change your own role". Compare `user.id === currentAdminId` from `useAdminContext()`. Status select remains enabled.
- **Verification:** Navigate to your own admin detail page — role dropdown must be disabled.
- **Source:** Spec 23a, backend answer #20

### Rule: Badge config as Record<string, { label, className }>
- **Rule:** Status and role badges use a `Record<string, { label: string; className: string }>` config object. Colors: amber for pending, green for approved, red for suspended, gray for waitlisted, blue for admin, muted for candidate. Use shadcn `Badge` with `variant="secondary"`.
- **Verification:** Grep for hardcoded badge colors — they should only exist in the config objects.
- **Source:** Spec 23a

### Rule: File upload uses hidden input + visible button
- **Rule:** DOCX upload uses a hidden file input (`className="sr-only"`) triggered by a visible button. Accept only `.docx`. Validate client-side with Zod `z.instanceof(FileList)`. Show selected filename. Clear input ref on success: `fileInputRef.current.value = ""`.
- **Verification:** File input has `accept=".docx"` and `className="sr-only"`.
- **Source:** Spec 23a

### Rule: Batch form uses useFieldArray
- **Rule:** Forms with dynamic item count (case studies) use React Hook Form's `useFieldArray`. Each item gets its own bordered section with a remove button (hidden when `fields.length === 1`). Submissions are sequential — loop through items, POST each, report final tally via toast ("N created" or "M of N failed").
- **Verification:** Add 3 items, remove middle one, submit — remaining 2 should POST sequentially.
- **Source:** Spec 23a

### Rule: Collapsible optional fields
- **Rule:** Forms with many optional fields (case studies) wrap them in a toggle section. Button text: "Show more fields" / "Hide optional fields". Track expanded state per item: `Record<number, boolean>`. Required fields are always visible.
- **Verification:** Required fields visible by default; optional fields hidden until toggled.
- **Source:** Spec 23a

### Rule: URL state for filters and pagination
- **Rule:** Admin list page stores all filter state (search, status, role, page) in URL search params. Use `useSearchParams()` + `useRouter()` + `usePathname()`. Reset to page 1 when filters change (not pagination). This makes links shareable and supports browser back/forward.
- **Verification:** Change a filter — URL params update. Copy URL, paste in new tab — same view appears.
- **Source:** Spec 23a

### Rule: Table responsive pattern
- **Rule:** Tables wrap in `overflow-x-auto` with `rounded-lg border border-white/5`. Headers: `text-xs font-medium uppercase tracking-wider text-text-muted`. Rows: `cursor-pointer border-b border-white/5 transition-colors hover:bg-white/5`. Show skeleton rows during loading, "No [items] found" when empty.
- **Verification:** Resize to mobile — table scrolls horizontally without layout break.
- **Source:** Spec 23a
