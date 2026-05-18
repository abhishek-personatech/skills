# Feature build plan: Console Meeting Workflow — Bulk Actions & Day Filter

| Field | Value |
|-------|-------|
| **Ticket / ID** | PTI-23163 |
| **Status** | Approved |
| **Document type** | Design |
| **Implementation status** | BE + FE implemented on `asf-PTI-23163-console-meeting-workflow-improvements` (BE: 2 commits; FE: 6 commits) |
| **Author** | agent + user |
| **Repos** | `phoenix` (BE), `phoenix-fe` (FE) |
| **Last updated** | 2026-05-15 |
| **dev_testing_status** | passed |
| **dev_testing_mode** | post_draft_pr |
| **dev_testing_round** | 1 |
| **pr_status** | ready_for_review (BE + FE bodies applied 2026-05-15) |
| **BE PR** | https://github.com/personatech-infra/phoenix/pull/6047 |
| **BE branch** | `asf-PTI-23163-console-meeting-workflow-improvements` |
| **FE PR** | https://github.com/personatech-infra/phoenix-fe/pull/6129 |
| **FE branch** | `asf-PTI-23163-console-meeting-workflow-improvements` |

---

## 1. Overview

### Problem statement

Console admins managing the Meeting Workflow screen must cancel meetings and resend calendar invites **one meeting at a time**, even when all actions target the same participant. This is slow and error-prone during high-volume event operations.

### Goal

Enable console users to perform **participant-scoped bulk cancel** and **bulk resend calendar invite** from the Meeting Workflow screen, and to **filter a participant's meetings by day** client-side after search — without changing the meeting search API.

### Success criteria

- [x] Bulk Cancel appears when participant is searched and cancellable meetings exist (via **Bulk actions** `NavMenuPopover` menu item).
- [x] Bulk Cancel collects Note for Admin (optional), optional **convert to email introduction** when program allows, confirms, calls bulk endpoint, refetches, generic success toast; modal stays open on API failure.
- [x] Bulk Resend Invite follows same visibility rules; confirms, calls bulk endpoint, refetches, generic success toast.
- [x] Day filter shows days from search results and filters the visible list client-side.
- [x] Bulk actions target **all** meetings from participant search (`data`), not day-filtered `filteredData`.
- [x] Ineligible meetings skipped on BE (FE does not surface per-meeting counts).

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
- ~~Bulk cancel `convertToEmailIntroduction` / email intro note~~ — **added in FE** (PTI-23163 Round 1) when BE bulk cancel supports same fields as single cancel; gated by `canShowBulkConvertToEmailIntroduction`.

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
| **Path** | `/api/admin/meeting/bulkCancel` |
| **Auth / roles** | Same as `POST /api/admin/meeting/cancel` |
| **Description** | Cancel all eligible meetings for a registration participant within a program. Skips meetings that fail single-cancel eligibility checks. Applies the same admin note to each successfully cancelled meeting. |

**Request**
```json
{
  "registrationParticipantId": "uuid — required",
  "programId": "uuid — required",
  "note": "string — optional; same semantics as single cancel (`ApiMeetingCancel.note`)",
  "convertToEmailIntroduction": "boolean — optional; default false; same semantics as single cancel",
  "emailIntroductionNote": "string — optional; required when convertToEmailIntroduction is true"
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
- `note` is persisted on each successfully cancelled meeting (same field as single cancel).
- `convertToEmailIntroduction` and `emailIntroductionNote` are passed through to each `cancelByAdmin` call (same as single cancel).
- HTTP 200 indicates the bulk operation completed (individual skips are not surfaced in the response).

---

### Bulk send calendar invite for participant meetings

| Property | Value |
|----------|-------|
| **Method** | POST |
| **Path** | `/api/admin/meeting/bulkSendCalendarInvite` |
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
- **Path naming:** Confirmed — routes live on `MeetingController` under `/admin/meeting/*` (global `/api` prefix applied by Spring config, same as existing cancel/search/send endpoints).

---

## 3. Backend plan (`phoenix`)

### 3.1 Summary

Add two admin REST endpoints on the existing `MeetingController` that batch-process a participant's meetings for a program by reusing `MeetingService.cancelByAdmin` and `MeetingService.sendCalendarInvite`. Meeting scope matches participant search: `findAllByRegistrationParticipantIdAndProgramId`. Per-meeting ineligibility is skipped silently (catch `BusinessException` for cancel; send already no-ops when ineligible). Endpoints return empty `200` with no counts. No search API, schema, or Flyway changes.

### 3.2 Similar features / patterns to follow

> User-attached reference files: `MeetingController.java` (single cancel + send), `MeetingService.java` (service logic).

| Reference feature | Controller | Service | Why relevant |
|-------------------|------------|---------|--------------|
| Single meeting cancel | `registration/src/main/java/com/personatech/program/rest/api/participantagenda/MeetingController.java` | `registration/src/main/java/com/personatech/program/service/programagenda/MeetingService.java` | `POST /admin/meeting/cancel` → `cancelByAdmin` — bulk must delegate per meeting |
| Single calendar invite resend | same `MeetingController.java` | same `MeetingService.java` | `GET /admin/meeting/sendCalendarInvite` → `sendCalendarInvite(programParticipationId)` — bulk calls once per resolved participation |
| Participant meeting search | same `MeetingController.java` | same `MeetingService.java` | `POST /admin/meeting/search` with `registrationParticipantId` + `programId` — same meeting scope for bulk |
| Program-scoped admin write | `registration/src/main/java/com/personatech/registration/rest/api/meetingmatching/MeetingMatchingController.java` | — | `@PreAuthorize("hasPermission(#request.programId, 'ProgramEval', 'PROGRAM_WRITE')")` — bulk auth pattern |
| Cancel API integration test | — | — | `registration/src/test/java/com/personatech/basespringboot/programagenda/ProgramAgendaListingTest.java` — `cancelMeeting()` MockMvc pattern |

### 3.3 Codebase anchors (verified)

| Layer | Path | Action |
|-------|------|--------|
| Controller | `registration/src/main/java/com/personatech/program/rest/api/participantagenda/MeetingController.java` | extend |
| Service | `registration/src/main/java/com/personatech/program/service/programagenda/MeetingService.java` | extend |
| Bulk cancel request DTO | `registration/src/main/java/com/personatech/program/rest/types/programagenda/ApiMeetingBulkCancelRequest.java` | create |
| Bulk send request DTO | `registration/src/main/java/com/personatech/program/rest/types/programagenda/ApiMeetingBulkSendCalendarInviteRequest.java` | create |
| Single-cancel DTO (reuse) | `registration/src/main/java/com/personatech/program/rest/types/programagenda/ApiMeetingCancel.java` | extend — no change; bulk builds instances |
| Search DTO (scope reference) | `registration/src/main/java/com/personatech/program/rest/types/programagenda/ApiMeetingSearch.java` | extend — no change |
| Admin response (FE eligibility) | `registration/src/main/java/com/personatech/program/rest/types/programagenda/ApiMeetingForAdmin.java` | extend — no change |
| Participation DTO | `registration/src/main/java/com/personatech/registration/rest/types/participation/ApiMeetingProgramParticipationForAdmin.java` | extend — no change |
| Repository query | `MeetingRepository.findByAgendaTypeAndProgramParticipations_EventParticipant_RegistrationParticipant_IdAndProgramId` (via `MeetingService.findAllByRegistrationParticipantIdAndProgramId`) | extend — no change |
| Calendar invite sender | `registration/src/main/java/com/personatech/registration/service/participant/MeetingProgramParticipantService.java` | extend — no change |
| Migration | — | none |
| Service unit test | `registration/src/test/java/com/personatech/baseunit/service/programagenda/MeetingServiceBulkActionTest.java` | create |
| Integration test | `registration/src/test/java/com/personatech/basespringboot/programagenda/MeetingBulkActionControllerTest.java` | create |

### 3.4 Data model & migrations

| Change | Type | Migration file | Notes |
|--------|------|----------------|-------|
| None | — | none | Bulk reuses `Meeting`, `ProgramParticipation`, and existing `note` field on meeting |

### 3.5 API contract adjustments (if any)

| Change | Reason |
|--------|--------|
| Confirmed paths: `POST /api/admin/meeting/bulkCancel`, `POST /api/admin/meeting/bulkSendCalendarInvite` | Match `MeetingController` `/admin/meeting/*` convention |
| Bulk send uses `POST` + JSON body (single send is `GET` + query param) | Symmetric bulk request shape; no change to single endpoint |
| Bulk auth: `ProgramEval` + `PROGRAM_WRITE` on `programId` | Program-scoped console workflow; single cancel/send use per-meeting/per-participation eval — document for FE permission gating |
| Bulk cancel supports `convertToEmailIntroduction` + `emailIntroductionNote` | Aligned with single cancel + FE Round 1 (commit `5cf84e8167`) |
| Request field `note` (not `noteForAdmin`) | Matches `ApiMeetingCancel.note` |
| DTOs suffixed with `Request` | `ApiMeetingBulkCancelRequest`, `ApiMeetingBulkSendCalendarInviteRequest` |

### 3.6 Implementation steps (ordered)

| Step | Task | File(s) | Depends on | Done when |
|------|------|---------|------------|-----------|
| BE-1 | Audit single cancel/send eligibility | Plan §3.6 step detail | — | Checklist mapped to bulk loop |
| BE-2 | Create bulk request DTOs | `ApiMeetingBulkCancelRequest.java`, `ApiMeetingBulkSendCalendarInviteRequest.java` | BE-1 | Types match Section 2 contract |
| BE-3 | Add participation resolver + `bulkCancelByAdmin` | `MeetingService.java` | BE-1, BE-2 | Unit tests pass |
| BE-4 | Add `bulkSendCalendarInvite` | `MeetingService.java` | BE-1, BE-2 | Unit tests pass |
| BE-5 | Expose `POST /admin/meeting/bulkCancel` | `MeetingController.java` | BE-3 | Integration test passes |
| BE-6 | Expose `POST /admin/meeting/bulkSendCalendarInvite` | `MeetingController.java` | BE-4 | Integration test passes |
| BE-7 | Service + controller tests | `MeetingServiceBulkActionTest.java`, `MeetingBulkActionControllerTest.java` | BE-5, BE-6 | green locally |

#### BE-1: Audit single-meeting eligibility rules

- **What:** Trace `cancelByAdmin` and `sendCalendarInvite`; document checks for bulk skip behavior.
- **Where:** `MeetingService.java` (L130–189 cancel, L371–383 send)
- **Pattern:** follow existing single-meeting flows — no new validation in bulk layer.

**Cancel (`cancelByAdmin` → `cancel`)**

| Check | On failure | Bulk behavior |
|-------|------------|---------------|
| Meeting not already `CANCELLED` | `BusinessException` (`meeting.already.cancelled`) | Skip |
| Program not after reconciliation | `BusinessException` (`meeting.cancellation-not-allowed`) | Skip |
| Primary participation for `registrationParticipantId` on meeting | `NoSuchElementException` if missing | Skip before calling |
| Email intro conversion | `BusinessException` if invalid | Skip (same as other cancel failures) |

**Not enforced on BE today:** `CONFIRMED`-only status (FE preview may filter; bulk must mirror single cancel).

**Resend (`sendCalendarInvite`)**

| Check | Bulk behavior |
|-------|---------------|
| `programStatus.isEmailAndInviteAllowed()` | No-op (no exception) |
| `!agenda.getStatus().isCancelled()` | No-op |
| `agenda.isFuture()` | No-op |

Aligns with `ApiMeetingForAdmin.canSendCalendarInvite`. Bulk calls `sendCalendarInvite` once per meeting for the participation belonging to `registrationParticipantId` (not counterparty).

#### BE-2: Bulk request DTOs

- **What:** Create `ApiMeetingBulkCancelRequest` (`registrationParticipantId`, `programId`, `note`, `convertToEmailIntroduction`, `emailIntroductionNote`) and `ApiMeetingBulkSendCalendarInviteRequest` (`registrationParticipantId`, `programId`) with `@NotNull` on required fields.
- **Where:** `registration/src/main/java/com/personatech/program/rest/types/programagenda/`
- **Pattern:** follow `ApiMeetingSearch.java` / `ApiMeetingCancel.java` (Lombok `@Data`, jakarta validation).
- **Done when:** `note` and email-intro fields map to `ApiMeetingCancel` in service.

#### BE-3: Bulk cancel service

- **What:** Load meetings via `findAllByRegistrationParticipantIdAndProgramId`; for each, resolve primary participation for `registrationParticipantId`; build `ApiMeetingCancel`; call `cancelByAdmin`; catch `BusinessException` and skip.
- **Where:** `MeetingService.java`
- **Pattern:** follow `cancelByAdmin` / `cancelMeetingByProgramParticipantCancellation` loop patterns.
- **Auth:** N/A at service layer.
- **Errors:** `baseProgramService.get(programId)` propagates not-found; per-meeting `BusinessException` swallowed.
- **Transaction:** Do **not** annotate `bulkCancelByAdmin` with `@Transactional` — each `cancelByAdmin` commits independently.
- **Done when:** Eligible meetings cancelled with shared note; ineligible skipped; no throw on all-ineligible.

#### BE-4: Bulk send invite service

- **What:** Same meeting load; resolve participation per meeting; call `sendCalendarInvite(participation.getId())`.
- **Where:** `MeetingService.java`
- **Pattern:** follow `sendCalendarInvite` — ineligibility is already silent.
- **Done when:** Eligible invites sent; ineligible no-ops; no throw on all-ineligible.

**Shared helper (recommended):**

```java
private ProgramParticipation findPrimaryParticipationForRegistrationParticipant(
    Meeting meeting, UUID registrationParticipantId)
```

Filter `meeting.getParticipations()` where `eventParticipant.registrationParticipant.id` matches and `role.isPrimary()`.

#### BE-5: Bulk cancel controller endpoint

- **What:** `POST /admin/meeting/bulkCancel` → `meetingService.bulkCancelByAdmin` → `ResponseEntity.ok().build()`.
- **Where:** `MeetingController.java`
- **Pattern:** follow `cancelMeeting()` endpoint structure.
- **Auth:** `@PreAuthorize("hasPermission(#request.programId, 'ProgramEval', 'PROGRAM_WRITE')")`
- **Done when:** 200 empty body; 403 without program write.

#### BE-6: Bulk send controller endpoint

- **What:** `POST /admin/meeting/bulkSendCalendarInvite` → `meetingService.bulkSendCalendarInvite` → `ResponseEntity.ok().build()`.
- **Where:** `MeetingController.java`
- **Pattern:** follow `sendCalendarInvite()` but `POST` + `@RequestBody`.
- **Auth:** same as BE-5.
- **Done when:** 200 empty body.

#### BE-7: Tests

- **What:**

| Test | File | Assert |
|------|------|--------|
| `bulkCancelByAdmin_allEligible_cancelsAll` | `MeetingServiceBulkActionTest.java` | N meetings → N `cancelByAdmin` calls |
| `bulkCancelByAdmin_mixedEligible_skipsIneligible` | same | cancelled meeting skipped; eligible cancelled |
| `bulkCancelByAdmin_noneEligible_noThrow` | same | void return, zero cancellations |
| `bulkSendCalendarInvite_delegatesPerMeeting` | same | correct `programParticipationId` per meeting |
| `bulkCancel_success` / `bulkSend_success` | `MeetingBulkActionControllerTest.java` | MockMvc POST, 200, DB state |
| `bulkCancel_forbidden` | same | 403 without `PROGRAM_WRITE` |

- **Pattern:** follow `ProgramAgendaListingTest.cancelMeeting()` — `MockMvcRequestBuilders.post`, `x-auth-token`, JSON body; extend `ProgramAgendaBaseTest`.
- **Run:** `./gradlew :registration:test --tests "com.personatech.baseunit.service.programagenda.MeetingServiceBulkActionTest"` and `--tests "com.personatech.basespringboot.programagenda.MeetingBulkActionControllerTest"`

### 3.7 Verification checklist (BE)

- [x] `./gradlew :registration:test --tests "com.personatech.baseunit.service.programagenda.MeetingServiceBulkActionTest"` (6/6, 2026-05-15)
- [x] `./gradlew :registration:test --tests "com.personatech.basespringboot.programagenda.MeetingBulkActionControllerTest"` (user sign-off Round 1; local run may require aligned test DB)
- [ ] No Flyway migration required
- [ ] Manual: `POST /api/admin/meeting/bulkCancel` with admin token — note persisted on each cancelled meeting
- [ ] Manual: `POST /api/admin/meeting/bulkSendCalendarInvite` — same side effects as `GET /api/admin/meeting/sendCalendarInvite`
- [ ] Manual: mixed eligible/ineligible participant — eligible processed, request still 200
- [ ] Section 2 contract matches implemented request shapes and void response

---


## 4. Frontend plan (`phoenix-fe`)

### 4.1 Summary

Extend the existing **Meetups Meeting Workflow** module (`meetups/list/`) — no new top-level feature folder. Add **Bulk Cancel**, **Bulk Resend Calendar Invite**, and a **Day filter** to the participant-scoped workflow.

**UI placement:** Bulk action buttons render in `meetingTopFilters_right` in `components/filters/index.tsx`, adjacent to the existing **Accept All** button — same visibility gate (`showAdditionalFilters` + participant search completed). Wrap buttons in `HasProgramWriteAuthority` (single-meeting actions use this in `index.tsx`).

**Participant filter:** `participant.tsx` already wires `selectParticipantHandler` → `setRegistrationParticipant`; **no changes required** there unless product later moves the day filter into the left search row. Bulk visibility/disable logic lives in `service.tsx` using `state.searchedRegistrationParticipantId`, `state.currentRegistrationParticipant`, and raw search `data` (not day-filtered `filteredData`).

**Flows:** Bulk cancel = outline `Button` → note modal (`FormTextField` for admin note, same validation as single cancel) → `initConfirmationAlert` with participant name → `POST bulkCancel` → `getParticipantsByCriteria(apiRequestObjectRef.current)` → generic success toast. Bulk resend = outline `Button` → `initConfirmationAlert` only (mirror `acceptAllMeetingsClickHandler`) → `POST bulkSendCalendarInvite` → refetch → toast. Day filter = client-side only via `applyAllFilters`; bulk APIs always use full participant search set.

### 4.2 Similar features / patterns to follow

> User-attached reference: Accept All bulk action in `components/filters/index.tsx` + `service.tsx`.

| Reference feature | Path | Why relevant |
|-------------------|------|--------------|
| Accept All bulk action | `meetups/list/components/filters/index.tsx` (L65–77), `service.tsx` (L369–446) | **Primary pattern** — `canShow*` / `shouldDisable*` memos, `meetingTopFilters_right` placement, `initConfirmationAlert`, refetch via `getParticipantsByCriteria`, `isSameParticipant` guard |
| Participant search filter | `meetups/list/components/filters/participant.tsx` | Sets `registrationParticipantId` + `setRegistrationParticipant`; search commits `searchedRegistrationParticipantId` in `searchClickHandler` |
| Single meeting cancel | `meetups/list/components/CancelMeeting.tsx`, `service.tsx` `handleCancelMeeting` | Note field + `Validation.textFieldValidation` (`MIN/MAX_NOTE_LENGTH_IN_MEETING`); confirmation alert before API |
| Single calendar invite resend | `meetups/list/components/SendInvite.tsx`, `MeetingActions.tsx` (L40–46) | `meeting.canSendCalendarInvite` eligibility flag on `ApiMeetingForAdmin` |
| Meeting type side filter | `service.tsx` `useSelectFilter` + `meetingSubType` | Day filter can reuse `useSelectFilter` hook pattern |
| API / dataloader | `meetups/list/dataloader.ts` | `usePostApiData` mutations; mirror `useAcceptAllMeetings` / `useCancelMeeting` |
| Authority gating | `apps/console/components/authority` `HasProgramWriteAuthority` | Bulk buttons require program write (aligns with BE `PROGRAM_WRITE`) |
| Integration tests | `meetups/list/__test__/meetups.test.tsx` | `consoleRender`, mock `useMeetingListContext` / dataloader |

### 4.3 Codebase anchors (verified)

| Area | Path | Action |
|------|------|--------|
| Route parent | `src/apps/console/modules/program/workflow/index.tsx` (L21) | extend — no route change; `MeetingList` already mounted |
| Feature entry | `src/apps/console/modules/program/workflow/meeting/meetups/list/index.tsx` | extend — `MeetingListProvider` / `MeetingList`; list reads `service.data` (filtered) |
| Service hook | `src/apps/console/modules/program/workflow/meeting/meetups/list/service.tsx` | extend — bulk handlers, eligibility memos, day filter state, `applyAllFilters` |
| Service types | `src/apps/console/modules/program/workflow/meeting/meetups/list/types.d.ts` | extend — new properties on `MeetingListingService` |
| Top filters / bulk button UI | `src/apps/console/modules/program/workflow/meeting/meetups/list/components/filters/index.tsx` | extend — add Bulk Cancel, Bulk Resend, Day filter in `meetingTopFilters_right` |
| Participant filter | `src/apps/console/modules/program/workflow/meeting/meetups/list/components/filters/participant.tsx` | **no change** (already provides participant context) |
| Dataloader | `src/apps/console/modules/program/workflow/meeting/meetups/list/dataloader.ts` | extend — `useBulkCancelMeetings`, `useBulkSendCalendarInvites` |
| API enum | `src/apps/console/enums/consoleApi.ts` (near L404–415) | extend — `BULK_CANCEL_MEETINGS`, `BULK_SEND_CALENDAR_INVITE` |
| Shared request types | `src/shared/types/ApiMeetingBulkCancel.ts`, `ApiMeetingBulkSendCalendarInvite.ts` | create — align with Section 2 |
| Eligibility helpers | `src/apps/console/modules/program/workflow/meeting/meetups/list/utils.ts` | extend — `getEligibleMeetingsForBulkCancel`, `getEligibleMeetingsForBulkResend`, `getUniqueMeetingDays` |
| Bulk cancel note modal | `src/apps/console/modules/program/workflow/meeting/meetups/list/components/BulkCancelMeetingsModal.tsx` | create |
| Meeting time display | `src/apps/console/modules/program/workflow/meeting/components/MeetingTime.tsx` | reference — day buckets must match `time.timeFrom` + `time.timeZoneDisplayStr` |
| SCSS | `src/apps/console/modules/program/workflow/workflow.scss` (`.meetingTopFilters`, L589+) | extend — gap/layout for extra buttons + day dropdown if needed |
| i18n | `src/shared/types/I18n.ts` | extend — new keys (Section 4.6) |
| Tests | `src/apps/console/modules/program/workflow/meeting/meetups/list/__test__/bulkActions.test.tsx` | create (or extend `meetups.test.tsx`) |
| Reducer | `meetups/list/reducer.ts` | extend — no change expected; reuse existing participant state |

### 4.4 Proposed feature structure

Extend existing module — **do not** create a sibling `MeetingWorkflow/` folder:

```
src/apps/console/modules/program/workflow/meeting/meetups/list/
├── index.tsx                          # no bulk UI here; list + context only
├── service.tsx                        # + bulk handlers, day filter, eligibility memos
├── types.d.ts                         # + MeetingListingService bulk/day props
├── dataloader.ts                      # + useBulkCancelMeetings, useBulkSendCalendarInvites
├── utils.ts                           # + eligibility + day extraction helpers
├── components/
│   ├── filters/
│   │   ├── index.tsx                  # + Bulk Cancel, Bulk Resend, Day filter buttons
│   │   └── participant.tsx          # unchanged
│   └── BulkCancelMeetingsModal.tsx    # new — note step only
└── __test__/
    ├── meetups.test.tsx               # existing
    └── bulkActions.test.tsx           # new integration tests
```

### 4.5 API integration (verified against Section 2)

| UI action | API (Section 2) | API enum | Dataloader hook | Service handler | Request mapping | Response / post-success | Error UX |
|-----------|-----------------|----------|-----------------|-----------------|-----------------|-------------------------|----------|
| Load meetings | `POST /api/admin/meeting/search` | `ConsoleApi.PARTICIPANTS_FOR_MEETINGS` | `useGetParticipantsForMeeting` | `getParticipantsByCriteria` | `{ programId, registrationParticipantId, … }` from Formik + `apiRequestObjectRef` | `ApiMeetingForAdmin[]` → raw `data` state → `filteredData` | existing axios / network handler |
| Bulk cancel | `POST /api/admin/meeting/bulkCancel` | `ConsoleApi.BULK_CANCEL_MEETINGS = "admin/meeting/bulkCancel"` | `useBulkCancelMeetings` | `bulkCancelMeetingsClickHandler` | `{ programId, registrationParticipantId: state.searchedRegistrationParticipantId, noteForAdmin }` | `void` → `successToast` + `getParticipantsByCriteria(apiRequestObjectRef.current)` | centralized error toast (existing mutation pattern); no partial-count UI |
| Bulk resend | `POST /api/admin/meeting/bulkSendCalendarInvite` | `ConsoleApi.BULK_SEND_CALENDAR_INVITE = "admin/meeting/bulkSendCalendarInvite"` | `useBulkSendCalendarInvites` | `bulkResendInvitesClickHandler` | `{ programId, registrationParticipantId: state.searchedRegistrationParticipantId }` | `void` → toast + refetch | same as bulk cancel |

**Integration notes**

- **HTTP client:** `usePostApiData` from `shared/datalayer` via `dataloader.ts` — same as `useAcceptAllMeetings` / `useCancelMeeting`. No new axios instance.
- **Auth / headers:** Inherited from `AXIOS_INSTANCE` interceptors (`x-auth-token`, etc.). UI gated by `HasProgramWriteAuthority` on bulk buttons.
- **Refetch:** Use `getParticipantsByCriteria(apiRequestObjectRef.current)` (same as `acceptAllMeetingsClickHandler` L440) — preserves current search criteria without re-submitting Formik.
- **Raw vs filtered data:** Service keeps unfiltered meetings in internal `data` state (`useState`); exposes `filteredData` as `data` on context. **Bulk eligibility must read internal raw `data`**, not context `filteredData`, so day/meeting-type filters do not affect bulk targeting.
- **Optimistic updates:** None — wait for 200, then refetch.
- **Day filter:** Add `selectedMeetingDay` to service; include in `applyAllFilters` comparing calendar date of `meeting.time.timeFrom` (use `DateTimeHelper` with meeting `time.timeZoneDisplayStr` for bucket labels consistent with `MeetingTime`). Clear day filter on `searchClickHandler` / `handleResetFilters`.
- **Mock strategy for tests:** `jest.spyOn(dataloaderModule, 'useBulkCancelMeetings')` + `consoleRender` with participant search flow; follow `meetups.test.tsx` patterns.

**Proposed shared types**

```typescript
// src/shared/types/ApiMeetingBulkCancel.ts
export interface ApiMeetingBulkCancel {
  registrationParticipantId: string;
  programId: string;
  noteForAdmin?: string;
}

// src/shared/types/ApiMeetingBulkSendCalendarInvite.ts
export interface ApiMeetingBulkSendCalendarInvite {
  registrationParticipantId: string;
  programId: string;
}
```

### 4.6 UX / copy (concrete)

| Screen / state | UI location | Behavior | Copy keys (proposed) |
|----------------|-------------|----------|----------------------|
| No participant search | — | Hide bulk buttons + day filter | — |
| Participant selected, not searched | — | Hide bulk buttons + day filter (`searchedRegistrationParticipantId` unset) | — |
| Participant searched, results loaded | `meetingTopFilters_right` | Show day filter + bulk buttons inside `showAdditionalFilters` | see below |
| Participant changed, not re-searched | bulk buttons visible | Disabled via `!isSameParticipant` (mirror Accept All L379–390) | `console.meeting.bulk.action.participant.not.searched.tooltip` |
| Bulk Cancel disabled | bulk button | `disabled` + tooltip when zero cancellable meetings in **raw** `data` | `console.meeting.bulk.cancel.disabled.tooltip` |
| Bulk Resend disabled | bulk button | `disabled` + tooltip when no meeting has `canSendCalendarInvite` in raw `data` | `console.meeting.bulk.resend.disabled.tooltip` |
| Bulk Cancel step 1 | `BulkCancelMeetingsModal` | Modal with Note for Admin (`FormTextField`, optional, `MIN/MAX_NOTE_LENGTH_IN_MEETING`) | `console.meeting.bulk.cancel.modal.title`, reuse `console.meeting.screen.note.for.admin` |
| Bulk Cancel step 2 | `initConfirmationAlert` | Confirm with participant full name | `console.meeting.bulk.cancel.confirmation` — `{0: currentRegistrationParticipant.fullName}` |
| Bulk Resend confirm | `initConfirmationAlert` | Single confirm (no note modal) | `console.meeting.bulk.resend.confirmation` — `{0: fullName}` |
| Bulk Cancel success | toast | After refetch | `console.meeting.bulk.cancel.success` |
| Bulk Resend success | toast | After refetch | `console.meeting.bulk.resend.success` |
| Day filter | `meetingTopFilters_right` (before bulk buttons) | `useSelectFilter` or `FormSelectDropdownField`; options = unique days from raw `data`; "All days" clears filter | `console.meeting.day.filter.placeholder`, `console.meeting.day.filter.all.days` |
| Day filter active | meeting list | `index.tsx` list shows subset; result count reflects filtered rows | — |
| Bulk action in flight | confirm handler | `try/catch`; disable confirm while awaiting (existing alert pattern) | — |

**Visibility / disable logic (mirror Accept All)**

```typescript
// Show bulk actions when participant filter was searched and list has data (not Connections program)
const canShowBulkParticipantActions = useMemo(
  () => Boolean(state.searchedRegistrationParticipantId) && showAdditionalFilters,
  [state.searchedRegistrationParticipantId, showAdditionalFilters]
);

// Cancel: FE preview — meeting.status !== CANCELLED (matches MeetingActions L26)
const shouldDisableBulkCancel = !isSameParticipant || !rawData.some(m => m.status !== ProgramAgendaStatus.CANCELLED);

// Resend: FE preview — meeting.canSendCalendarInvite (matches MeetingActions L40)
const shouldDisableBulkResend = !isSameParticipant || !rawData.some(m => m.canSendCalendarInvite);
```

> **BE alignment note:** Section 3 BE-1 documents additional cancel checks (reconciliation, already cancelled). FE disable is a **preview** only; server skips ineligible meetings silently. Do not block the button on FE-only stricter rules unless product requires it.

### 4.7 Implementation steps (ordered)

| Step | Task | File(s) | Depends on | Done when |
|------|------|---------|------------|-----------|
| FE-1 | Shared bulk request types | `src/shared/types/ApiMeetingBulkCancel.ts`, `ApiMeetingBulkSendCalendarInvite.ts` | Section 2 | Types compile |
| FE-2 | API enum + dataloader hooks | `consoleApi.ts`, `dataloader.ts` | FE-1 | Hooks mockable in tests |
| FE-3 | Eligibility helpers | `utils.ts` | Search result shape | Unit-testable pure functions |
| FE-4 | Service: memos + handlers + day filter state | `service.tsx`, `types.d.ts` | FE-2, FE-3 | Handlers call API + refetch |
| FE-5 | Bulk cancel note modal | `components/BulkCancelMeetingsModal.tsx` | FE-4 | Note → confirm → API |
| FE-6 | Wire UI in top filters | `components/filters/index.tsx` | FE-4, FE-5 | Buttons + day filter render with correct gates |
| FE-7 | i18n keys | `src/shared/types/I18n.ts` (+ backend copy feed) | FE-5, FE-6 | No hardcoded strings |
| FE-8 | Integration tests | `__test__/bulkActions.test.tsx` | FE-6 | Scenarios below pass |
| FE-9 | SCSS (if needed) | `workflow.scss` | FE-6 | Buttons align in `meetingTopFilters_right` |

#### FE-1: Shared bulk request types
- **What:** Add `ApiMeetingBulkCancel` and `ApiMeetingBulkSendCalendarInvite` matching Section 2 JSON bodies.
- **Where:** `src/shared/types/ApiMeetingBulkCancel.ts`, `ApiMeetingBulkSendCalendarInvite.ts`
- **Pattern:** follow `ApiAdminAcceptAllMeetingsRequest`
- **Done when:** Imported by `dataloader.ts` without `any`

#### FE-2: API enum + dataloader
- **What:** Add `BULK_CANCEL_MEETINGS` and `BULK_SEND_CALENDAR_INVITE` to `ConsoleApi`; export `useBulkCancelMeetings` and `useBulkSendCalendarInvites` via `usePostApiData<void, RequestType>`.
- **Where:** `src/apps/console/enums/consoleApi.ts`, `meetups/list/dataloader.ts`
- **Pattern:** follow `useAcceptAllMeetings` (L24–26)
- **Done when:** Enum values match BE paths `admin/meeting/bulkCancel` and `admin/meeting/bulkSendCalendarInvite`

#### FE-3: Eligibility + day helpers
- **What:** Pure functions: `getMeetingsEligibleForBulkCancel(data)`, `getMeetingsEligibleForBulkResend(data)`, `getUniqueMeetingDays(data)` using `meeting.time.timeFrom` + timezone display string.
- **Where:** `meetups/list/utils.ts`
- **Pattern:** follow `applyMeetingTypeFilter` / `meetingListingFilterFunction` style
- **Done when:** Helpers used by service memos; day list sorted chronologically

#### FE-4: Service extension
- **What:** Add state `showBulkCancelModal`, `selectedMeetingDay`; memos `canShowBulkParticipantActions`, `shouldDisableBulkCancel`, `shouldDisableBulkResend`; handlers `bulkCancelMeetingsClickHandler`, `bulkResendInvitesClickHandler`, `handleBulkCancelSubmit(note)`; extend `applyAllFilters` for day; reset day on `searchClickHandler` / `handleResetFilters`.
- **Where:** `service.tsx`, `types.d.ts`
- **Pattern:** follow `acceptAllMeetingsClickHandler` (L422–446) + `handleCancelMeeting` confirmation chain
- **Done when:** Bulk APIs receive `registrationParticipantId` from `state.searchedRegistrationParticipantId`; refetch uses `apiRequestObjectRef.current`

#### FE-5: Bulk cancel note modal
- **What:** Modal with single `FormTextField` for admin note; on valid submit close modal and open `initConfirmationAlert`; on confirm call bulk cancel mutation.
- **Where:** `components/BulkCancelMeetingsModal.tsx`
- **Pattern:** follow `CancelMeeting.tsx` note field validation (L128–143) — **without** participant dropdown or email-intro checkbox (out of scope per Section 1)
- **Done when:** `noteForAdmin` passed to API; modal controlled by `showBulkCancelModal` on service

#### FE-6: Top filters UI
- **What:** In `meetingTopFilters_right`, inside `showAdditionalFilters` + `HasProgramWriteAuthority`:
  1. Day filter dropdown (when `canShowBulkParticipantActions`)
  2. Bulk Cancel outline `Button` with `tooltipContent` when disabled
  3. Bulk Resend outline `Button` with `tooltipContent` when disabled
  Place after `canShowAcceptAllMeetings` block (L65–77) — same bulk-action cluster.
- **Where:** `components/filters/index.tsx`
- **Pattern:** follow Accept All `Button` props (`isOutline`, `disabled`, `tooltipContent`)
- **Done when:** Buttons hidden until participant search; disabled states match memos

#### FE-7: i18n
- **What:** Add keys from Section 4.6 table to `I18n.ts` (backend copy pipeline handles runtime strings).
- **Where:** `src/shared/types/I18n.ts`
- **Done when:** All new UI strings use `copies.get(...)`

#### FE-8: Integration tests
- **What:** Scenarios per `.cursor/skills/testing/SKILL.md`:
  - Participant search → bulk buttons appear in top-right filter area
  - Zero eligible meetings → buttons disabled with tooltip copy
  - Bulk cancel: open modal → enter note → confirm alert → mutation called with correct payload
  - Bulk resend: click → confirm alert → mutation called
  - Day filter: options match mock meeting dates; selecting day reduces rendered list count; bulk handler still receives full raw set (spy `getParticipantsByCriteria` payload unchanged)
- **Where:** `__test__/bulkActions.test.tsx`
- **Pattern:** `consoleRender`, mock `dataloaderModule`, partial `useMeetingListContext` override — follow `meetups.test.tsx`
- **Run:** `yarn test --testPathPattern=bulkActions`

#### FE-9: SCSS
- **What:** Ensure `meetingTopFilters_right` accommodates day dropdown + two extra outline buttons without wrap breakage on tablet.
- **Where:** `workflow.scss` `.meetingTopFilters_right`
- **Pattern:** follow `phoenix-scss` skill — extend existing block, no new global classes unless needed
- **Done when:** Visual parity with Accept All button row

### 4.8 Verification checklist (FE)

- [x] `yarn test --testPathPattern=meetup.bulk.actions --watchAll=false` (5 tests, 2026-05-15)
- [x] `yarn check-types` — new types + service interface compile (implicit via CI / local dev)
- [x] Manual: Console → Program Workflow → Meetings → search by **Participant** → **Bulk actions** menu + day filter in `meetingTopFilters_right` (Round 1)
- [x] Manual: change participant without re-search → bulk menu items hidden (`isSameParticipant`); day filter hidden
- [x] Manual: day filter → list shrinks; bulk payload uses full search set (integration test + Round 1)
- [x] Manual: bulk cancel note (+ optional email intro) → confirm → refetch
- [x] Manual: bulk resend → confirm → refetch; generic success toast only
- [x] API integrated against dev/staging after BE bulk endpoints deployed (user sign-off Round 1)
- [x] Scoped to `meetups/list/` + shared bulk request types + `consoleApi` enum (no unrelated drive-by in feature commits)

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

## 9. Dev testing

| Field | Value |
|-------|-------|
| **Status** | passed |
| **Environment** | dev / staging (user-confirmed) |
| **Mode** | post_draft_pr |
| **PR URLs** | BE: https://github.com/personatech-infra/phoenix/pull/6047 · FE: https://github.com/personatech-infra/phoenix-fe/pull/6129 |

### Test matrix

#### Frontend (Round 1)

| # | Scenario | Surface | Role | Pass? |
|---|----------|---------|------|-------|
| 1 | Participant search → **Bulk actions** menu + day filter visible in top-right | Console → Program Workflow → Meetings | Admin w/ program write | ☑ |
| 2 | All meetings cancelled → bulk menu hidden (no eligible actions) | Same | Admin | ☑ |
| 3 | Participant changed without re-search → bulk items / day filter hidden | Same | Admin | ☑ |
| 4 | Bulk cancel: note modal → confirm → `POST bulkCancel` with `registrationParticipantId` + `programId` (+ optional `note`, email intro fields) → refetch → success toast | Same | Admin | ☑ |
| 5 | Bulk cancel API failure → modal remains open (retest confirm) | Same | Admin | ☑ |
| 6 | Bulk resend: menu → confirm → `POST bulkSendCalendarInvite` → refetch → success toast | Same | Admin | ☑ |
| 7 | Day filter reduces visible rows; bulk still targets full search set (no day in payload) | Same | Admin | ☑ |
| 8 | Optional **Convert to email introduction** on bulk cancel when program + meetings allow | Same | Admin | ☑ |
| 9 | `yarn test --testPathPattern=meetup.bulk.actions` | CI / local | — | ☑ |

#### Backend (Round 1)

| # | Scenario | Surface | Role | Pass? |
|---|----------|---------|------|-------|
| B1 | `POST /api/admin/meeting/bulkCancel` — eligible meetings cancelled, shared `note` persisted | API / Console via FE | Admin w/ `PROGRAM_WRITE` | ☑ |
| B2 | `POST /api/admin/meeting/bulkCancel` — mixed eligible/ineligible → 200, ineligible skipped | API | Admin | ☑ |
| B3 | `POST /api/admin/meeting/bulkCancel` — `convertToEmailIntroduction` + `emailIntroductionNote` forwarded per meeting | API / Console | Admin | ☑ |
| B4 | `POST /api/admin/meeting/bulkSendCalendarInvite` — eligible meetings receive invite; ineligible no-op | API / Console | Admin | ☑ |
| B5 | Bulk endpoints return void 200 (no per-meeting counts) | API | Admin | ☑ |
| B6 | `MeetingServiceBulkActionTest` (6 unit tests) | CI / local | — | ☑ |
| B7 | `MeetingBulkActionControllerTest` (4 integration tests) | CI / aligned test DB | — | ☑ (user sign-off) |

### Round log

#### Round 1 — 2026-05-15 (retrospective catch-up)

| ID | Finding | Class | Repo | Fix step | Status |
|----|---------|-------|------|----------|--------|
| DT-1 | Separate outline Bulk Cancel / Resend buttons crowded the filter row | P1 | FE | `ecdf475e15` — `NavMenuPopover` **Bulk actions** dropdown | fixed |
| DT-2 | Bulk cancel should support email introduction parity with single cancel | P1 | FE | `21a71dd762` — `convertToEmailIntroduction` + `emailIntroductionNote` on request + modal | fixed |
| DT-3 | Bulk cancel modal closed before user saw API error | P0 | FE | `f0dca99f0c` — keep modal open until cancel succeeds | fixed |
| DT-4 | Plan used `noteForAdmin` / `ApiMeetingBulkCancel` names | P2 | FE | `18ec910be7` — `note`, `ApiMeetingBulkCancelRequest` | fixed (doc only) |
| DT-5 | Bulk cancel modal body uses hardcoded HTML; note copy not wired | P2 | FE | `b43e336ee9` — `console.meeting.bulk.cancel.note` via `useCopies` | fixed |
| DT-6 | Plan contract used `noteForAdmin`; single cancel uses `note` | P1 | BE | `5cf84e8167` — `ApiMeetingBulkCancelRequest.note` | fixed |
| DT-7 | Plan marked email intro out of scope for bulk cancel | P1 | BE | `5cf84e8167` — `convertToEmailIntroduction` + `emailIntroductionNote` on bulk request | fixed |
| DT-8 | DTO names in plan lacked `Request` suffix | P2 | BE | `5cf84e8167` — `ApiMeetingBulk*Request` | fixed (doc + code) |

### Sign-off (required before ready-for-review)

- [x] All P0 from latest round fixed
- [x] All P1 fixed or explicitly deferred by user
- [x] Automated tests green — FE: `meetup.bulk.actions.test.tsx` (5/5); BE: `MeetingServiceBulkActionTest` (6/6 local, 2026-05-15)
- [x] `dev_testing_status` set to `passed` in metadata
- [x] User sign-off for dev testing (2026-05-15 catch-up)
- [x] BE review-ready PR body applied — https://github.com/personatech-infra/phoenix/pull/6047 (2026-05-15)
- [x] FE review-ready PR body — https://github.com/personatech-infra/phoenix-fe/pull/6129 (2026-05-15)

### Implementation deltas vs plan (FE — for reviewers)

| Plan | Shipped |
|------|---------|
| Two outline buttons (Bulk Cancel, Bulk Resend) | `ContextMenu` (⋮ dots); menu items shown only when eligible (`b43e336ee9`) |
| Buttons **disabled** + tooltips when zero eligible | Menu items **hidden** when no eligible meetings; no disabled-tooltip copy keys |
| `noteForAdmin` on bulk cancel request | `note` on `ApiMeetingBulkCancelRequest` |
| `ApiMeetingBulkCancel` / `ApiMeetingBulkSendCalendarInvite` types | `ApiMeetingBulkCancelRequest` / `ApiMeetingBulkSendCalendarInviteRequest` |
| `__test__/bulkActions.test.tsx` | `__test__/meetup.bulk.actions.test.tsx` |
| Bulk cancel without email intro | Optional convert-to-email-intro + note (BE-aligned) |
| `console.meeting.day.filter.all.days` + disabled tooltips | Not added; day filter uses `useSelectFilter` placeholder only |
| FE-9 SCSS changes | Not required — existing `meetingTopFilters_right` layout sufficient |

### Implementation deltas vs plan (BE — for reviewers)

| Plan | Shipped |
|------|---------|
| `ApiMeetingBulkCancel` / `ApiMeetingBulkSendCalendarInvite` | `ApiMeetingBulkCancelRequest` / `ApiMeetingBulkSendCalendarInviteRequest` |
| `noteForAdmin` on bulk cancel request | `note` (matches `ApiMeetingCancel`) |
| Bulk cancel always `convertToEmailIntroduction = false` | Request fields passed through to each `cancelByAdmin` |
| `bulkSendCalendarInvite` only no-ops on ineligibility | Also catches `BusinessException` per meeting (debug log + skip), symmetric with bulk cancel |
| BE-7 integration tests green locally | `MeetingServiceBulkActionTest` green locally; `MeetingBulkActionControllerTest` requires aligned test DB in some environments |

---

## 8. Revision history

| Date | Author | Change |
|------|--------|--------|
| 2026-05-13 | agent | Initial draft from approved requirement summary |
| 2026-05-13 | agent | Bulk cancel/resend APIs return void (no counts); FE uses generic success toast |
| 2026-05-13 | user | Approved (LGTM) |
| 2026-05-13 | agent | BE plan enriched via `phoenix-be-implementation-plan` skill (§3 restructured to template) |
| 2026-05-13 | agent | FE plan enriched via `phoenix-fe-implementation-plan` skill — verified anchors in `meetups/list/`, Accept All pattern, API integration, ordered FE steps |
| 2026-05-15 | agent | Catch-up (FE): metadata (PTI-23163, PR #6129, `dev_testing_status: passed`); Section 9 Round 1 retrospective; success criteria + §4.8 ticked; FE implementation deltas documented |
| 2026-05-15 | agent | Catch-up (BE): metadata (PR #6047, 2 commits); Section 2 contract + §3 anchors/steps aligned to shipped DTOs and email-intro fields; §3.7 ticked; Section 9 BE matrix + deltas + DT-6–DT-8 |
| 2026-05-18 | agent | FE UX: ContextMenu bulk menu (`b43e336ee9`); DT-5 fixed; FE PR #6129 review-ready body enriched; plan deltas updated |
