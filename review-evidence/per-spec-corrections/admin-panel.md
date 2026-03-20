# Contract Corrections: Admin Panel

Source: Nolan (Backend Foil) cross-review

---

## Field Renames

| Spec Says | Contract Says | Location |
|-----------|--------------|----------|
| `role: "user"` | `role: "candidate"` | UserActions, updateUserRole, all role change logic |
| `per_page` query param | `page_size` | UserListParams, fetch calls, PaginatedResponse |
| `AdminUser.name` | `UserSummary.full_name` | AdminUser type, table rendering |

## Type Shape Fixes

**`AdminUser` → align to `UserSummary` from types.ts:**
```ts
// WRONG (spec):
interface AdminUser { id, name, email, plan, role, founding_member, created_at }

// RIGHT (contract):
interface AdminUser {
  id: string;
  full_name: string;
  email: string;
  account_status: string;
  role: string;
  current_job_title: string | null;
  current_company: string | null;
  career_stage: string | null;
  focus_area_count: number;
  application_count: number;
  last_active_at: string | null;
  created_at: string;
  onboarding_completed_at: string | null;
}
```

**Remove from AdminUser:** `plan`, `founding_member` — these fields don't exist on `UserSummary`. Either file RFC to add them or remove Plan/FM columns from table until subscription system provides the data.

## Endpoint Fixes

| Spec Endpoint | Actual Endpoint | Issue |
|--------------|-----------------|-------|
| `GET /v1/admin/stats` | `GET /v1/admin/dashboard` | Different response shape (`DashboardStats`) |
| `POST /v1/admin/founding-codes` | Does not exist | **Block chunks 6-8, file RFC** |
| `GET /v1/admin/founding-codes` | Does not exist | **File RFC** |
| `POST /v1/admin/founding-codes/apply` | Does not exist | **File RFC** |
| `GET /v1/admin/founding-codes/pool` | Does not exist | **File RFC** |
| `GET /v1/admin/activity` | Does not exist | Derive from user list or drop section |

## Parameter Fixes

- Backend `GET /v1/admin/users` does NOT support `sort` or `order` params. Only supports: `page`, `page_size`, `status`, `search`. Remove sort dropdown or implement client-side sort on current page only.

## Error Response Shape

```ts
// WRONG (spec):
{ "error": "unauthorized" }

// RIGHT (contract):
{ "detail": "Authenticated but not authorized", "error_code": "AUTH_FORBIDDEN" }
```

Update all error parsing to read `detail` and `error_code`.

## Additions

- Backend already supports `DELETE /v1/admin/users/{id}` (soft delete) and `PUT /v1/admin/users/{id}/status` (suspend/reactivate). Consider adding to UserActions dropdown.
- `account_status` valid values: `"pending_review" | "approved" | "suspended" | "waitlisted"`
