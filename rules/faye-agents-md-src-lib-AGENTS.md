# AGENTS.md — src/lib/

Utility functions, API client, auth helpers, and shared logic.

## Rules

### lib-R001 — API client uses environment variables
All API URLs come from `process.env.NEXT_PUBLIC_API_URL`. Never hardcode URLs.
- **Source:** CLAUDE.md critical rule #4
- **Added:** 2026-03-10
- **Status:** active

### lib-R002 — Auth token passthrough
Every authenticated API call must include `Authorization: Bearer <token>` header.
Token comes from Supabase Auth session, never from localStorage directly.
- **Source:** CLAUDE.md backend integration
- **Added:** 2026-03-10
- **Status:** active

### lib-R003 — No secrets in client code
Supabase anon key is the only key allowed in client bundles. Service role keys,
API secrets, and any other credentials must never appear in frontend code.
- **Source:** CLAUDE.md critical rule #4
- **Added:** 2026-03-10
- **Status:** active

### lib-R004 — Cross-origin storage is never implicit
Any time data crosses subdomains (e.g., getversed.ai → app.getversed.ai), specify the transfer mechanism explicitly: API relay, postMessage, or URL params. Never assume browser storage (localStorage, sessionStorage, cookies without explicit domain) works cross-origin. Different subdomains are different origins for storage APIs.
- **Source:** L002
- **Added:** 2026-03-14
- **Status:** active
