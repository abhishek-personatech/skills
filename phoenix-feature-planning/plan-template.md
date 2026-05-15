# Feature build plan: [Feature name]

| Field | Value |
|-------|-------|
| **Ticket / ID** | [e.g. PTI-12345] |
| **Status** | Draft \| In review \| Approved |
| **Document type** | Design \| Design + Implementation |
| **Implementation status** | Pending repo enrichment \| BE enriched \| FE enriched \| Ready to implement \| Implemented |
| **Repo scope** | BE \| FE \| both |
| **Dev testing status** | not_started \| in_progress \| passed \| waived |
| **Dev testing round** | 0 |
| **Dev testing mode** | pre_pr \| post_draft_pr |
| **PR status** | none \| draft \| ready_for_review |
| **Author** | [agent + user] |
| **Repos** | `phoenix` (BE), `phoenix-fe` (FE) |
| **Last updated** | [YYYY-MM-DD] |

---

## 1. Overview

### Problem statement
[What problem this solves and for whom]

### Goal
[Measurable outcome]

### Success criteria
- [ ] [Criterion 1]
- [ ] [Criterion 2]

### Scope
**In scope**
- ...

**Out of scope**
- ...

### Assumptions & constraints
- ...

---

## 2. Shared API contract

> Single source of truth for BE ↔ FE integration. Frontend plan must reference this section.

### [Endpoint 1 name]

| Property | Value |
|----------|-------|
| **Method** | GET \| POST \| PUT \| PATCH \| DELETE |
| **Path** | `/api/...` |
| **Auth / roles** | ... |
| **Description** | ... |

**Request**
```json
{
  "field": "type — notes"
}
```

**Response (200)**
```json
{
  "field": "type — notes"
}
```

**Errors**
| Status | When |
|--------|------|
| 400 | ... |
| 403 | ... |
| 404 | ... |

### [Endpoint 2 name]
...

### Contract notes
- Idempotency: ...
- Pagination / sorting / filters: ...
- Backward compatibility: ...
- Feature flag / config keys: ...

---

## 3. Backend plan (`phoenix`)

### 3.1 Summary
[One paragraph: what changes on the backend]

### 3.2 Codebase anchors
| Area | Path / pattern | Notes |
|------|----------------|-------|
| Controller | `registration/src/main/java/...` | ... |
| Service | `...` | ... |
| Repository / entity | `...` | ... |
| DTO / request-response | `...` | ... |
| Tests | `registration/src/test/java/...` | ... |

### 3.3 Data model & migrations
| Change | Type | Details |
|--------|------|---------|
| [table/column] | new \| alter \| index | Flyway: `V...__....sql` |

**Migration notes**
- ...

### 3.4 Implementation steps (ordered)

| Step | Task | Files / components | Depends on |
|------|------|-------------------|------------|
| BE-1 | ... | ... | — |
| BE-2 | ... | ... | BE-1 |
| BE-3 | Add/update REST endpoint | `...Controller` | BE-2 |
| BE-4 | Service logic | `...Service` | BE-2 |
| BE-5 | Unit / integration tests | `...Test` | BE-3, BE-4 |

**Per-step detail**

#### BE-1: [Title]
- **What:** ...
- **Where:** `path/to/file`
- **How:** ...
- **Done when:** ...

#### BE-2: [Title]
...

### 3.5 Validation & rollout (BE)
- [ ] Unit tests for service logic
- [ ] Controller/API tests
- [ ] Migration runs clean on empty + existing DB
- [ ] Manual API check (curl / Postman): ...
- [ ] Permissions verified for roles: ...

---

## 4. Frontend plan (`phoenix-fe`)

### 4.1 Summary
[One paragraph: what changes on the frontend]

### 4.2 Codebase anchors
| Area | Path | Notes |
|------|------|-------|
| Feature entry | `src/.../FeatureName/` | ... |
| Routes | `...` | ... |
| Shared API client | `...` | ... |
| Copy / i18n | `I18n` keys | ... |
| Tests | `__test__/` | per `.cursor/skills/testing/SKILL.md` |

### 4.3 UX / UI breakdown
| Screen / state | Behavior | Copy keys |
|----------------|----------|-----------|
| Loading | ... | ... |
| Success | ... | ... |
| Empty | ... | ... |
| Error | ... | ... |

### 4.4 Feature structure (proposed)
```
FeatureName/
├── index.tsx
├── service.ts
├── types.ts
├── components/
│   └── ...
└── __test__/
    ├── FeatureName.test.tsx
    └── mockData.ts
```

### 4.5 API integration

> Implement exactly against **Section 2**. Do not diverge without updating this plan.

| UI action | API (from contract) | Hook / service function | Request mapping | Response mapping | Error handling |
|-----------|---------------------|-------------------------|-----------------|------------------|----------------|
| [e.g. Load list] | `GET /api/...` | `useFeatureService.fetchList` | query params → ... | `response.items` → `ListItem[]` | toast + retry |
| [e.g. Submit form] | `POST /api/...` | `submit` | form values → body | `id` → navigate | inline field errors |

**Integration details**
- **Client / module:** [axios instance, base path, existing API helper]
- **Types:** define in `types.ts` — align field names with BE DTOs
- **Loading / error state:** ...
- **Caching / refetch:** [React Query keys if applicable]
- **Optimistic updates:** yes/no — ...
- **Feature flag / config:** ...

### 4.6 Implementation steps (ordered)

| Step | Task | Files | Depends on |
|------|------|-------|------------|
| FE-1 | Types + API client functions | `types.ts`, `service.ts` | BE contract (can mock until BE-3) |
| FE-2 | Service hook (state, handlers) | `service.ts` | FE-1 |
| FE-3 | Presentational components | `components/` | FE-2 |
| FE-4 | Wire `index.tsx` + route | `index.tsx` | FE-3 |
| FE-5 | i18n copy keys | `I18n` | FE-3 |
| FE-6 | Integration tests | `__test__/` | FE-4 |
| FE-7 | SCSS (if needed) | co-located `.scss` | FE-3 |

**Per-step detail**

#### FE-1: [Title]
- **What:** ...
- **Where:** ...
- **How:** ...
- **Done when:** ...

### 4.7 Validation & rollout (FE)
- [ ] Follow `CLAUDE.md` patterns
- [ ] No hardcoded user-facing strings
- [ ] Integration tests: loading, success, error, primary interaction
- [ ] Manual test on target surface(s): ...
- [ ] API integrated against [dev/staging] environment

---

## 5. Cross-cutting concerns

| Topic | BE | FE |
|-------|----|----|
| Auth / permissions | ... | ... |
| Analytics / logging | ... | ... |
| i18n | N/A or ... | ... |
| Performance | ... | ... |
| Rollout / feature flag | ... | ... |

---

## 6. Suggested execution order

1. ...
2. ...
3. ...

**Parallel work (if safe)**
- ...

**Blockers**
- ...

---

## 7. Risks & mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| ... | ... | ... |

---

## 9. Dev testing

| Field | Value |
|-------|-------|
| **Status** | not_started \| in_progress \| passed \| waived |
| **Environment** | local \| dev \| staging |
| **Mode** | pre_pr \| post_draft_pr |
| **PR URLs** | BE: … \| FE: … |

### Test matrix

| # | Scenario | Surface | Role | Pass? |
|---|----------|---------|------|-------|
| 1 | ... | ... | ... | ☐ |

### Round log

#### Round 1 — YYYY-MM-DD

| ID | Finding | Class | Repo | Fix step | Status |
|----|---------|-------|------|----------|--------|
| DT-1 | ... | P0 \| P1 \| P2 \| out_of_scope | BE \| FE | BE-7b / FE-8b | open \| fixed \| deferred |

### Sign-off (required before ready-for-review)

- [ ] All P0 from latest round fixed
- [ ] All P1 fixed or explicitly deferred by user
- [ ] Automated tests green (per phoenix-git-workflow) for repos in scope
- [ ] `dev_testing_status` set to `passed` or `waived` in metadata above
- [ ] User sign-off for marking PR(s) ready for review

---

## 10. Revision history

| Date | Author | Change |
|------|--------|--------|
| YYYY-MM-DD | agent | Initial draft |
| YYYY-MM-DD | agent | [Review feedback summary] |
| YYYY-MM-DD | agent | Dev testing Round N — [summary] |
