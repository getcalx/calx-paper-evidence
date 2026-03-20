# AGENTS.md — src/components/applications/

Rules and conventions for application tracking components.

---

### Rule: Status badge color mapping is authoritative
- **Rule:** `StatusBadge` with exported `statusColors` record defines the canonical color mapping: applied=blue, screening=amber, interviewing=purple, offer=green, rejected=red, ghosted=slate, withdrew=slate. All status displays must use `StatusBadge` or reference `statusColors` — never hardcode status colors.
- **Verification:** Grep for status color classes — they should only exist in `statusColors` definition.
- **Source:** Spec 22

### Rule: Application card is keyboard navigable
- **Rule:** Cards use `role="button"` with `tabIndex={0}` and handle Enter/Space keypress for navigation. Layout: job title + company (top), status badge (top-right), metadata row (date, resume source), optional notes preview (truncated at 100 chars).
- **Verification:** Tab to a card, press Enter — navigates to detail page.
- **Source:** Spec 22

### Rule: Resume source labels
- **Rule:** Map enum values to display labels: `soaria_optimized` → "Soaria Optimized", `user_original` → "Original Resume", `unknown` → "Unknown". Use these consistently in cards, stats, and forms.
- **Verification:** No raw enum values displayed to users.
- **Source:** Spec 22

### Rule: Stats panel structure
- **Rule:** Three sections in order: (1) horizontal scrollable stat mini-cards (total + per-status, clickable to filter), (2) conversion funnel rates with `formatRate()` (multiply < 1 values by 100, one decimal, "--" for null), (3) resume source comparison table with green highlight on best interview rate.
- **Verification:** Stats panel renders all three sections. Null rates show "--".
- **Source:** Spec 22

### Rule: Create dialog adapts to context
- **Rule:** When opened from a job detail page, pass `defaultJobId`/`defaultJobTitle`/`defaultCompanyName` — job field becomes read-only. When opened from applications list, show helper text directing user to job detail page. Resume version auto-selects if only one exists. Applied date defaults to today, max is today.
- **Verification:** Open from job detail — job pre-filled and locked. Open from list — shows helper text.
- **Source:** Spec 22

### Rule: Interview timeline is a vertical line with nodes
- **Rule:** Rounds rendered as vertical timeline: absolute left border for connector line, round number circles as nodes, content to the right. Sorted by `round_number`. Last round has no connector below. Notes collapsed by default with chevron toggle. Duration formatted as "45 min" / "1h 30m".
- **Verification:** Multiple rounds show connected vertical line. Click notes toggle — content expands.
- **Source:** Spec 22

### Rule: Outcome form has conditional fields by type
- **Rule:** `outcome_type` controls which fields render: offer types show compensation breakdown (base, total comp, equity, signing bonus, etc.) + negotiation section. Rejected shows reason + stage. Withdrew shows reason select + details textarea. Ghosted shows no extra fields. Component toggles between display mode and edit mode.
- **Verification:** Select "offer_received" — compensation fields appear. Select "ghosted" — only type selector visible.
- **Source:** Spec 22

### Rule: Interview round type and outcome labels
- **Rule:** Round types map to labels (phone_screen → "Phone Screen", technical → "Technical", etc.). Outcome badges have colors: pending=slate, moved_forward=green, rejected=red, no_show=amber, cancelled=slate. Withdrawal reasons also have label mappings.
- **Verification:** All enum values display human-readable labels.
- **Source:** Spec 22

### Rule: Notes dirty tracking on detail page
- **Rule:** Detail page notes use `notesSaved` (server value), `notesText` (user input), and `notesDirty` (computed as `notesText !== notesSaved`). Save button only appears when dirty. Uses `prevServerNotesRef` to detect server data changes and sync local state without overwriting user edits.
- **Verification:** Type in notes — Save button appears. Save — button disappears. Navigate away and back — server value persists.
- **Source:** Spec 22

### Rule: Cursor-based pagination with Load More
- **Rule:** Uses `useInfiniteQuery` with `initialPageParam: undefined`, `getNextPageParam` returning `next_cursor ?? undefined`. Pages flattened via `useMemo` + `flatMap`. "Load More" button only shows when `hasNextPage` is true, disabled during `isFetchingNextPage`.
- **Verification:** Load 20+ applications — Load More button appears. Click — next page loads. No more pages — button disappears.
- **Source:** Spec 22

### Rule: Filter tabs toggle on/off
- **Rule:** Status filter tabs use `role="tablist"` / `role="tab"`. "All" clears the filter. Clicking an already-active status tab clears back to "All" (toggle behavior). Filter change resets pagination.
- **Verification:** Click "Screening" tab — filters. Click "Screening" again — shows all.
- **Source:** Spec 22
