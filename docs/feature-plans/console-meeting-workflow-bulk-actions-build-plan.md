# Feature build plan: Console Meeting Workflow — Bulk Actions & Day Filter

| Field | Value |
|-------|-------|
| **Ticket / ID** | TBD |
| **Status** | Approved |
| **Document type** | Design |
| **Implementation status** | Pending repo enrichment |
| **Author** | agent + user |
| **Repos** | `phoenix` (BE), `phoenix-fe` (FE) |
| **Last updated** | 2026-05-13 |

---

## 1. Overview

### Problem statement

Console admins managing the Meeting Workflow screen must cancel meetings and resend calendar invites **one meeting at a time**, even when all actions target the same participant. This is slow and error-prone during high-volume event operations.

### Goal

Enable console users to perform **participant-scoped bulk cancel** and **bulk resend calendar invite** from the Meeting Workflow screen, and to **filter a participant's meetings by day** client-side after search — without changing the meeting search API.

### Success criteria

- [ ] Bulk Cancel button appears in the action area when a participant is selected and searched; disabled when no eligible (confirmed + cancellable) meetings exist.
- [ ] Bulk Cancel collects Note for Admin, confirms, cancels all eligible participant meetings via a new bulk endpoint, refreshes list, and shows a generic success toast.
- [ ] Bulk Resend Invite button follows the same visibility/disable rules; confirms and sends via a new bulk endpoint; refreshes list and shows a generic success toast.
- [ ] Day filter shows only days present in search results and filters the visible list client-side.
- [ ] Bulk actions always target **all** meetings from participant search, not the day-filtered subset.
- [ ] Ineligible meetings are skipped silently during bulk operations (no all-or-nothing failure).

### Scope

**In scope**

- New BE bulk cancel endpoint reusing single-cancel validation/rules (`POST /api/admin/meeting/cancel`).
- New BE bulk send calendar invite endpoint reusing single-send rules (`POST /api/admin/meeting/sendCalendarInvite`).
- FE: bulk action buttons, modals/alerts, day filter, eligibility-derived button state, refetch + toast.
- i18n copy for new UI strings.

**Out of scope**

- Backend changes to `/api/admin/meeting/search` or day-based server filtering.
- Scoping bulk actions to the day-filtered subset.
- Per-meeting failure reporting UI or all-or-nothing bulk semantics.
- Non-console surfaces (My Experience, mobile web, etc.).
- Changes to single-meeting cancel/resend UX (beyond shared BE logic reuse).

### Assumptions & constraints

- Single-meeting cancel and resend endpoints encode the authoritative eligibility rules; bulk endpoints delegate to the same service-layer methods.
- Participant display name is available in the Meeting Workflow UI for confirmation copy.
- Day boundaries are derived from meeting date fields already present in search results (no separate timezone conversion layer).
- Bulk action permissions match existing single-meeting cancel/resend permissions.
- Buttons are **disabled** (not hidden) when zero eligible meetings exist; optional tooltip explains why (confirm during FE enrichment).

---

## 2. Shared API contract

> Single source of truth for BE ↔ FE integration. Frontend plan must reference this section.

### Bulk cancel participant meetings

| Property | Value |
|----------|-------|
| **Method** | POST |
| **Path** | `/api/admin/meeting/bulkCancel` *(provisional — align with existing admin meeting route conventions during BE enrichment)* |
| **Auth / roles** | Same as `POST /api/admin/meeting/cancel` |
| **Description** | Cancel all eligible meetings for a registration participant within a program. Skips meetings that fail single-cancel eligibility checks. Applies the same admin note to each successfully cancelled meeting. |

**Request**
```json
{
  "registrationParticipantId": "string — required",
  "programId": "string — required",
  "noteForAdmin": "string — optional; same semantics as single cancel"
}
```

**Response (200)**
- Empty body (`void`) — no per-meeting or aggregate counts returned.

**Errors**
| Status | When |
|--------|------|
| 400 | Missing/invalid `registrationParticipantId` or `programId` |
| 403 | Caller lacks permission to cancel meetings |
| 404 | Participant or program not found |
| 500 | Unexpected server error |

**Behavior notes**
- Server loads candidate meetings for the participant (same scope as search: `registrationParticipantId` + `programId`).
- For each candidate, run the **same validation and cancel logic** as `POST /api/admin/meeting/cancel` (confirmed status and any additional checks enforced there).
- On per-meeting ineligibility: skip silently, continue processing remaining meetings.
- `noteForAdmin` is persisted on each successfully cancelled meeting (same field as single cancel).
- HTTP 200 indicates the bulk operation completed (individual skips are not surfaced in the response).

---

### Bulk send calendar invite for participant meetings

| Property | Value |
|----------|-------|
| **Method** | POST |
| **Path** | `/api/admin/meeting/bulkSendCalendarInvite` *(provisional)* |
| **Auth / roles** | Same as `POST /api/admin/meeting/sendCalendarInvite` |
| **Description** | Resend calendar invites for all eligible meetings for a registration participant within a program. Skips ineligible meetings using the same rules as single send. |

**Request**
```json
{
  "registrationParticipantId": "string — required",
  "programId": "string — required"
}
```

**Response (200)**
- Empty body (`void`) — no per-meeting or aggregate counts returned.

**Errors**
| Status | When |
|--------|------|
| 400 | Missing/invalid required fields |
| 403 | Caller lacks permission to send invites |
| 404 | Participant or program not found |
| 500 | Unexpected server error |

**Behavior notes**
- Server loads candidate meetings for the participant (`registrationParticipantId` + `programId`).
- For each candidate, run the **same validation and send logic** as `POST /api/admin/meeting/sendCalendarInvite`.
- On per-meeting ineligibility: skip silently, continue.
- HTTP 200 indicates the bulk operation completed (individual skips are not surfaced in the response).

---

### Meeting search (unchanged)

| Property | Value |
|----------|-------|
| **Method** | POST *(confirm verb during BE enrichment)* |
| **Path** | `/api/admin/meeting/search` |
| **Auth / roles** | Existing console admin meeting search permissions |
| **Description** | Returns meetings for a participant within a program. FE uses this response for list display, day filter options, and client-side eligibility preview for button disable state. |

**Request** *(existing — confirm exact shape during enrichment)*
```json
{
  "registrationParticipantId": "string",
  "programId": "string"
}
```

**Response** — existing meeting list payload; FE derives:
- Visible rows (optionally filtered by selected day).
- Day filter dropdown options (unique dates from meeting start/date field).
- Eligible counts for bulk buttons (mirror single-action eligibility rules on FE where possible; BE remains authoritative on execution).

---

### Contract notes

- **Response body:** Bulk cancel and bulk resend return **void** (empty 200 body). FE must not expect counts; use generic success toast + list refetch for feedback.
- **Idempotency:** Bulk cancel on already-cancelled meetings should be skipped (via single-cancel rules). Bulk resend may be repeatable per single-send semantics.
- **Pagination / filters:** Bulk endpoints operate on full participant meeting set for the program, not paginated UI slices.
- **Backward compatibility:** Existing single-meeting endpoints remain unchanged.
- **Feature flag:** None required unless product requests phased rollout.
- **Path naming:** Final paths must match `phoenix` admin meeting controller conventions during `phoenix-be-implementation-plan`.

---

## 3. Backend plan (`phoenix`)

### 3.1 Summary

Add two admin REST endpoints that batch-process a participant's meetings by reusing existing single-meeting cancel and send-calendar-invite service methods. No search API or schema changes. Per-meeting ineligibility is skipped silently; endpoints return `void` on success with no counts.

### 3.2 Codebase anchors

| Area | Path / pattern | Notes |
|------|----------------|-------|
| Admin meeting controller | `TBD — enrich with phoenix-be-implementation-plan` | Locate handler for `/api/admin/meeting/cancel` |
| Cancel service | `TBD` | Extract or call existing cancel method per meeting |
| Send invite service | `TBD` | Extract or call existing send method per meeting |
| Meeting query | `TBD` | Load meetings by `registrationParticipantId` + `programId` |
| Request DTOs | `TBD` | New bulk request DTOs only (no response body) |
| Tests | `TBD` | Unit tests for skip behavior; integration tests for endpoints |

### 3.3 Data model & migrations

| Change | Type | Details |
|--------|------|---------|
| None expected | — | Bulk endpoints reuse existing meeting tables and cancel/note fields |

**Migration notes**
- No Flyway migration anticipated unless audit/logging requires a new table (out of scope unless discovered during enrichment).

### 3.4 Implementation steps (ordered)

| Step | Task | Files / components | Depends on |
|------|------|-------------------|------------|
| BE-1 | Locate single cancel + send invite flows; document eligibility checks | Controller, service | — |
| BE-2 | Add bulk request DTOs | DTO package | BE-1 |
| BE-3 | Implement bulk cancel service (loop + skip) | Service layer | BE-1, BE-2 |
| BE-4 | Implement bulk send invite service (loop + skip) | Service layer | BE-1, BE-2 |
| BE-5 | Expose `POST .../bulkCancel` endpoint | Admin meeting controller | BE-3 |
| BE-6 | Expose `POST .../bulkSendCalendarInvite` endpoint | Admin meeting controller | BE-4 |
| BE-7 | Unit + API tests | Test packages | BE-5, BE-6 |

**Per-step detail**

#### BE-1: Audit single-meeting eligibility rules
- **What:** Read `POST /api/admin/meeting/cancel` and `POST /api/admin/meeting/sendCalendarInvite` handlers and downstream services. List all checks (status, permissions, timing, participant binding, etc.).
- **Where:** `TBD — enrich with phoenix-be-implementation-plan`
- **How:** Trace from controller → service → domain validation.
- **Done when:** Checklist of rules documented and mapped to bulk loop.

#### BE-3: Bulk cancel service
- **What:** Query all meetings for participant+program; for each, invoke shared cancel logic with `noteForAdmin`; catch ineligibility and skip.
- **Where:** `TBD`
- **How:** Prefer calling the same private/package method used by single cancel rather than duplicating validation.
- **Done when:** Eligible meetings cancelled; ineligible skipped silently; endpoint returns void on completion.

#### BE-4: Bulk send invite service
- **What:** Same pattern as BE-3 using single send-calendar-invite logic.
- **Where:** `TBD`
- **How:** Reuse single-send service method per meeting.
- **Done when:** Eligible invites sent; ineligible skipped silently; endpoint returns void on completion.

### 3.5 Validation & rollout (BE)

- [ ] Unit tests: all eligible → all cancelled/sent; mixed eligible/ineligible → eligible processed, ineligible skipped; none eligible → void 200, no side effects on ineligible
- [ ] Controller/API tests for auth, 400, 403, 404
- [ ] Manual API check: bulk cancel with note persisted on each cancelled meeting
- [ ] Manual API check: bulk resend triggers same side effects as single send
- [ ] Permissions verified for same roles as single endpoints

---

## 4. Frontend plan (`phoenix-fe`)

### 4.1 Summary

Extend the Console Meeting Workflow screen with two bulk action buttons and a day filter. Buttons appear when a participant filter is applied and search has run. Bulk cancel uses a two-step flow (note modal → confirm alert); bulk resend uses a confirm alert only. After success, refetch search results and show a generic success toast (no count from API). Day filter is purely client-side on the search response.

### 4.2 Codebase anchors

| Area | Path | Notes |
|------|------|-------|
| Meeting Workflow feature | `TBD — enrich with phoenix-fe-implementation-plan` | Console meeting workflow screen |
| API endpoint enums | `TBD` e.g. `src/apps/console/enums/consoleApi.ts` | Add bulk endpoint constants |
| Single cancel / resend UI | `TBD` | Mirror eligibility UX and copy patterns |
| Copy / i18n | `I18n` keys | New modal, alert, toast, day filter labels |
| Tests | `__test__/` | Per repo testing skill |

### 4.3 UX / UI breakdown

| Screen / state | Behavior | Copy keys |
|----------------|----------|-----------|
| Participant not selected | No bulk buttons; no day filter | — |
| Participant selected, pre-search | No bulk buttons; no day filter | — |
| Participant searched, results loaded | Show day filter + bulk buttons in action area | `TBD` |
| Bulk Cancel disabled | Zero eligible (confirmed + cancellable) meetings | Tooltip: no eligible meetings `TBD` |
| Bulk Resend disabled | Zero eligible meetings per single-resend rules | Tooltip `TBD` |
| Bulk Cancel — step 1 | Modal: Note for Admin (optional/required per single-cancel parity) | `TBD` |
| Bulk Cancel — step 2 | Confirm alert: `"<participant-name>'s meetings will be marked cancelled, are you sure to proceed?"` | `TBD` |
| Bulk Resend — confirm | Alert: `"Calendar invite will be sent for <participant-name>'s meetings"` | `TBD` |
| Bulk action loading | Disable buttons / show loading on confirm | `TBD` |
| Bulk action success | Refetch search; generic success toast | `TBD` |
| Bulk action error | Toast or inline error; no partial UI breakdown | `TBD` |
| Day filter | Dropdown of unique days from results; filters table only | `TBD` |
| Day filter cleared | Show all meetings from search | — |

### 4.4 Feature structure (proposed)

Extend existing Meeting Workflow module rather than a new top-level feature:

```
MeetingWorkflow/   (existing — exact path TBD)
├── index.tsx              # wire day filter + bulk buttons
├── service.ts             # bulk API calls, eligibility helpers, refetch
├── types.ts               # bulk request types
├── components/
│   ├── BulkCancelNoteModal.tsx
│   ├── BulkActionConfirmAlert.tsx   # shared or separate per action
│   └── DayFilter.tsx
└── __test__/
    ├── MeetingWorkflow.bulk.test.tsx
    └── mockData.ts
```

### 4.5 API integration

> Implement exactly against **Section 2**. Do not diverge without updating this plan.

| UI action | API (from contract) | Hook / service function | Request mapping | Response mapping | Error handling |
|-----------|---------------------|-------------------------|-----------------|------------------|----------------|
| Load meetings | `POST /api/admin/meeting/search` | existing search handler | participant + program filters | meeting list → state | existing error handling |
| Bulk cancel | `POST /api/admin/meeting/bulkCancel` | `bulkCancelMeetings` | `registrationParticipantId`, `programId`, `noteForAdmin` | void → generic success toast + refetch | toast on 4xx/5xx |
| Bulk resend | `POST /api/admin/meeting/bulkSendCalendarInvite` | `bulkSendCalendarInvites` | `registrationParticipantId`, `programId` | void → generic success toast + refetch | toast on 4xx/5xx |

**Integration details**
- **Client / module:** Existing console axios instance and API helper patterns — `TBD during enrichment`
- **Types:** Define bulk request types in `types.ts` aligned with BE DTOs (no response body)
- **Loading / error state:** Disable bulk buttons and show loading on modal/alert confirm while request in flight
- **Caching / refetch:** On bulk success, invalidate/refetch meeting search query for current participant+program
- **Optimistic updates:** No — wait for server response, then refetch
- **Eligibility preview (button disable):** Derive from loaded search results using same rules as single actions (confirmed for cancel; single-resend rules for resend).
- **Day filter:** Pure FE — filter displayed rows by selected date; do not pass day to bulk APIs

### 4.6 Implementation steps (ordered)

| Step | Task | Files | Depends on |
|------|------|-------|------------|
| FE-1 | Types + API client for bulk endpoints | `types.ts`, `service.ts`, API enum | BE contract (can mock) |
| FE-2 | Eligibility helpers + button disabled state | `service.ts` | FE-1, search result shape |
| FE-3 | Day filter component + client-side filtering | `DayFilter.tsx`, `index.tsx` | Search results |
| FE-4 | Bulk cancel modal + confirm flow | `BulkCancelNoteModal.tsx`, alerts | FE-1, FE-2 |
| FE-5 | Bulk resend confirm flow | alert component / shared | FE-1, FE-2 |
| FE-6 | Wire action area buttons + post-success refetch/toast | `index.tsx`, `service.ts` | FE-3–FE-5 |
| FE-7 | i18n copy keys | I18n | FE-4–FE-6 |
| FE-8 | Integration tests | `__test__/` | FE-6 |
| FE-9 | SCSS (if needed) | co-located `.scss` | FE-3–FE-6 |

**Per-step detail**

#### FE-3: Day filter
- **What:** Extract unique calendar dates from meeting list; render dropdown; filter displayed rows when a day is selected.
- **Where:** `TBD`
- **How:** Use the same date field as the meeting list display; `useMemo` for unique days sorted chronologically.
- **Done when:** Only days with meetings appear; clearing filter shows full list; bulk actions unaffected by selection.

#### FE-4: Bulk cancel flow
- **What:** Button → Note for Admin modal → on submit → confirmation alert with participant name → call bulk cancel API.
- **Where:** `TBD`
- **How:** Reuse existing modal/alert primitives from console meeting workflow or shared components.
- **Done when:** Happy path matches acceptance criteria; note passed to API.

### 4.7 Validation & rollout (FE)

- [ ] Follow `CLAUDE.md` patterns
- [ ] No hardcoded user-facing strings
- [ ] Integration tests: buttons disabled with zero eligible; bulk cancel two-step flow; bulk resend confirm; day filter options and filtering
- [ ] Manual test: participant search → day filter → bulk actions still target full set
- [ ] API integrated against dev/staging after BE endpoints deployed

---

## 5. Cross-cutting concerns

| Topic | BE | FE |
|-------|----|----|
| Auth / permissions | Reuse single cancel/send permission checks | Hide/disable if user lacks permission (match single-action gating) |
| Analytics / logging | Log bulk operation with participantId, programId | Optional: track bulk action clicks/completions |
| i18n | N/A | All modal, alert, toast, day filter, tooltip copy |
| Performance | Batch may be large — consider async/job if participant has hundreds of meetings (flag as risk) | Client-side day filter only; no extra API calls |
| Rollout / feature flag | None required | None required |

---

## 6. Suggested execution order

1. **BE-1** — Audit single cancel/send rules (blocks accurate FE eligibility preview).
2. **BE-2 → BE-6** — Implement and deploy bulk endpoints.
3. **FE-1 → FE-2** — Types, API client, eligibility helpers (can start with mocked BE).
4. **FE-3** — Day filter (no BE dependency; can ship independently).
5. **FE-4 → FE-7** — Bulk UI flows + i18n.
6. **FE-8** — Integration tests.
7. End-to-end QA on console Meeting Workflow.

**Parallel work (if safe)**
- FE day filter (FE-3) in parallel with BE bulk endpoints.
- FE bulk UI with mocked API in parallel with BE once contract is frozen.

**Blockers**
- FE bulk actions need deployed bulk endpoints (or mocks) for E2E validation.
- Exact single-cancel/resend eligibility rules must be documented in BE-1 before FE disable logic is finalized.

---

## 7. Risks & mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Participant has very many meetings | Long-running bulk request, timeout | BE enrichment: consider batching or async job if existing patterns support it; set reasonable timeout + loading UX |
| FE eligibility preview diverges from BE | Button enabled but all skipped | Mirror rules from BE-1 checklist; integration tests; refetch shows unchanged list |
| Day date field ambiguous (multi-day events) | Wrong day bucket in filter | Use same date field as list column during enrichment |
| Silent skips confuse admins | User thinks nothing happened | Generic success toast + refetch so list reflects actual cancellations/sends |

---

## 8. Revision history

| Date | Author | Change |
|------|--------|--------|
| 2026-05-13 | agent | Initial draft from approved requirement summary |
| 2026-05-13 | agent | Bulk cancel/resend APIs return void (no counts); FE uses generic success toast |
| 2026-05-13 | user | Approved (LGTM) |
