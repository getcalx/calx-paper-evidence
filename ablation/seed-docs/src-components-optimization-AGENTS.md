# AGENTS.md — src/components/optimization/

Rules and conventions for optimization (resume optimization modal) components.

---

### Rule: Modal state machine uses discriminated union
- **Rule:** The optimization modal uses a `{ phase: "loading" | "error" | "success" }` discriminated union for state. Each phase renders mutually exclusive UI — no leaking of one phase's data into another. Transitions via `setState({ phase: "..." })`.
- **Verification:** Each phase check uses `state.phase === "..."`. No optional chaining on phase-specific data.
- **Source:** Spec 20

### Rule: Three-panel layout with CSS Grid
- **Rule:** Desktop: `grid-cols-[57fr_43fr]`. Right column splits vertically: `grid-rows-[35fr_65fr]` (fit context top, chat bottom). Mobile: stacked. Every panel uses `overflow-y-auto` for independent scrolling. `min-h-0` on flex containers is critical for overflow to work.
- **Verification:** Each panel scrolls independently. Content doesn't overflow the modal bounds.
- **Source:** Spec 20

### Rule: Inline editor — click to edit, save on blur
- **Rule:** `InlineEditor` renders as `<span role="button">` in read mode, `<input>` or `<textarea>` in edit mode. Enter edit on click/Enter/Space. Save on blur or Enter (single-line) / Cmd+Enter (multiline). Cancel on Escape. Trim whitespace. Only fire onChange if value actually changed. Set cursor to end on focus.
- **Verification:** Click text → becomes editable. Blur → saves. Escape → reverts. Tab through → keyboard accessible.
- **Source:** Spec 20

### Rule: Delta highlighting with tooltip
- **Rule:** Changed text gets `border-l-2 border-blue-500 bg-blue-500/10`. Hovering shows a `DeltaTooltip` with original text and reasoning. Delta matching uses `section_name.toLowerCase().includes()` + exact text match via `findDelta()` helper.
- **Verification:** Optimized fields show blue left border. Hovering shows "Original: ... Why: ..." tooltip.
- **Source:** Spec 20

### Rule: Chat markdown is lightweight, no innerHTML
- **Rule:** Chat messages use a safe `renderMarkdown()` function that handles `**bold**`, `` `code` ``, and `- `/`* ` bullet lists. Returns React nodes — no `dangerouslySetInnerHTML`. User messages right-aligned (blue tint), assistant messages left-aligned (card bg).
- **Verification:** Bold/code/bullets render correctly. No XSS risk from message content.
- **Source:** Spec 20

### Rule: Loading stepper with rotating messages
- **Rule:** Loading shows 4 fixed messages rotating every 5 seconds. Active step: blue pulsing dot. Completed: green dot. Inactive: muted dot. Progress bar width = `((currentStep + 1) / total) * 100%`. Never advances past last message. Cancel button always available.
- **Verification:** Loading appears for ~20s. Steps advance visually. Cancel closes modal.
- **Source:** Spec 20

### Rule: Error codes map to user-friendly messages
- **Rule:** `SERVICE_UNAVAILABLE` / 503 → "temporarily unavailable". `LLM_ERROR` / 502 → "Optimization failed". Default → raw message. "Try Again" resets to loading phase and re-runs mutation. "Close" exits modal.
- **Verification:** Force a 502 error — user sees friendly message, not "Bad Gateway".
- **Source:** Spec 20, HANDOFF Q27

### Rule: Unsaved edits confirmation on close
- **Rule:** Track `hasUnsavedEdits` state (set true on first inline edit). Close button and Escape key check this flag — if true, `window.confirm("You have unsaved changes. Close anyway?")`. Cancel prevents close.
- **Verification:** Edit a field, press Escape — confirmation dialog appears.
- **Source:** Spec 20

### Rule: Approval flow is a choreographed sequence
- **Rule:** "Approve & Download": (1) flush edits to `/user-edit` (best-effort), (2) download DOCX (required — stop on failure), (3) record `clicked_apply` interaction (best-effort), (4) open ATS URL in new tab, (5) close modal after 1.5s delay. Button shows "Approving..." with spinner.
- **Verification:** Click Approve & Download — DOCX downloads, ATS opens, modal closes.
- **Source:** Spec 20

### Rule: Resume preview uses immutable state updates
- **Rule:** All edits go through `updateData(prev => ({ ...prev, field: newValue }))`. This notifies the parent via `onDataChange`, triggers debounced API save, and updates local state. `latestDataRef` in parent tracks latest data for the approval flow.
- **Verification:** Every section handler uses `updateData`. No direct `setData` calls.
- **Source:** Spec 20

### Rule: Chat turn limit and session closed state
- **Rule:** Display "Turn X of Y" from API response. When `session_closed: true`, disable input and show "Chat session complete" message. Clear Chat resets `sessionClosed` and allows a new session.
- **Verification:** Send 5 messages (default max) — input disables. Clear chat — input re-enables.
- **Source:** Spec 20, HANDOFF Q10

### Rule: Fit score color thresholds
- **Rule:** Score >= 80: green. >= 60: blue. >= 40: amber. < 40: red. Each threshold maps to text, border, and bg color variants (e.g., `text-green-400`, `border-green-500/30`, `bg-green-500/10`). Reuse `FitBadge` from feed for the score badge.
- **Verification:** Score of 85 shows green. Score of 55 shows blue. Etc.
- **Source:** Spec 20

### Rule: Locked vs editable resume sections
- **Rule:** Contact, education, and certifications are locked (no InlineEditor). Summary and skills are editable. Experience: headers (company/title/dates) locked, bullet highlights editable with multiline InlineEditor. Skills render as pills with hover-visible remove button + dashed-border "Add" button.
- **Verification:** Click on contact info — nothing happens. Click on a skill — enters edit mode.
- **Source:** Spec 20
