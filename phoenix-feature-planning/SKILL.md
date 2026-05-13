---
name: phoenix-feature-planning
description: >-
  Turn raw feature requirements into a reviewed design document for Phoenix
  backend (phoenix) and frontend (phoenix-fe). Runs a clarification loop, requirement
  summary, confirmation gate, then generates a design-level build plan (scope, API
  contract, BE/FE breakdown) without code-audited paths. Use when planning a feature,
  writing a design doc, technical design, or when the user says phoenix-feature-planning,
  feature build plan, or @phoenix-feature-planning.
disable-model-invocation: true
---

# Phoenix feature planning (design document)

Produce a **reviewed design-level build plan** from a raw requirement. This skill does **not** require a repo to be open and does **not** produce code-audited file paths.

**After design approval**, enrich with repo-specific implementation detail using:

- `phoenix-be-implementation-plan` — open `phoenix` repo
- `phoenix-fe-implementation-plan` — open `phoenix-fe` repo

Do not jump to implementation until the **design plan** is approved and (optionally) repo-specific plans are enriched.

## Repositories

| Layer | Repo | This skill (design only) | Repo-specific enrichment |
|-------|------|--------------------------|--------------------------|
| Backend | `phoenix` | Logical API + data model + work breakdown | `phoenix-be-implementation-plan` |
| Frontend | `phoenix-fe` | UX surfaces + integration approach + work breakdown | `phoenix-fe-implementation-plan` |

This skill is **planning only**. It does not implement code and does not require codebase access.

## Workflow (strict phases)

Copy and track progress:

```
Planning Progress:
- [ ] Phase 1: Intake
- [ ] Phase 2: Clarification loop
- [ ] Phase 3: Requirement summary + user confirmation
- [ ] Phase 4: Build plan draft
- [ ] Phase 5: Review loop until approved
```

**Never skip phases.** Do not write the build plan file until Phase 3 is explicitly confirmed.

## Approval signals

Treat any of these as **sufficient approval** to proceed to the next phase — do not ask for additional confirmation:

- `LGTM` (any common casing)
- `approved` / `approve`

**Phase 3:** `LGTM` or `approved` on the requirement summary → immediately proceed to Phase 4 (draft the build plan).

**Phase 5:** `LGTM` or `approved` on the build plan → mark plan **Approved**, close the review loop, and give the implementation handoff.

If the user sends only `LGTM` or `approved` with no other text, infer which phase you are in from context and advance accordingly.

---

### Phase 1: Intake

When the user provides a raw requirement:

1. Restate the goal in 2–4 sentences (your understanding, not a plan yet).
2. Note what is **in scope**, **out of scope**, and **unknown**.
3. Ask the **first batch** of clarification questions (3–7 max per turn). Prefer `AskQuestion` when available.

**Clarification themes** (ask only what applies):

- User personas and surfaces (console, my-experience, admin, mobile web, etc.)
- Acceptance criteria and happy path
- Edge cases, error handling, permissions/auth
- Data model changes, existing entities to extend vs new tables
- API contract expectations (read vs write, pagination, filters)
- Backward compatibility and feature flags
- Dependencies on other teams, services, or in-flight work
- Rollout, migration, and rollback
- Non-functional needs (performance, audit, i18n, analytics)
- Deadline or phasing (MVP vs follow-up)

---

### Phase 2: Clarification loop

Repeat until requirements are sufficiently clear:

1. Incorporate user answers.
2. Publish a **short delta summary** (what changed since last turn).
3. Ask the **next** clarification questions — do not re-ask resolved items.
4. If the codebase happens to be open, you may skim it to improve **questions** — but do not block on code access. Do not present code paths as verified in this phase.

**Stop clarifying when:** no material ambiguities remain, or the user says to proceed.

---

### Phase 3: Requirement summary + confirmation gate

Present a structured summary:

```markdown
## Requirement summary (pending your confirmation)

### Problem / goal
...

### In scope
- ...

### Out of scope
- ...

### Users & surfaces
...

### Acceptance criteria
1. ...

### Assumptions
- ...

### Open questions (if any remain)
- ...

### BE touchpoints (high level)
- ...

### FE touchpoints (high level)
- ...
```

Then ask: **"Confirm this summary so I can draft the build plan, or tell me what to change."**

**`LGTM` or `approved` alone is enough** to proceed to Phase 4. Update and re-present only if the user requests changes.

---

### Phase 4: Design document draft

After confirmation, create **one markdown file** using [plan-template.md](plan-template.md). Set document **type** to `Design` and **implementation status** to `Pending repo enrichment`.

**Output location** (default):

- If a planning/docs folder exists in the open workspace, use it.
- Otherwise create: `docs/feature-plans/<feature-slug>-build-plan.md`
- Use kebab-case for `<feature-slug>` (e.g. `daily-meeting-summary-export`).

Tell the user the file path and ask for review.

**Design document quality bar** — concrete at the **product/engineering design** level:

- Clear scope, acceptance criteria, assumptions
- Logical API contract (method, path, payloads, auth, errors) — paths may be provisional
- Separate **Backend design** and **Frontend design** sections
- Ordered work breakdown without claiming verified file paths
- Frontend **API integration approach** (which calls, when, error handling) tied to the shared contract
- Mark code locations as `TBD — enrich with phoenix-be-implementation-plan / phoenix-fe-implementation-plan` unless explicitly verified in an open repo this turn

---

### Phase 5: Review loop

1. Ask the user to review the build plan file.
2. Collect feedback (inline comments, bullet list, or "LGTM").
3. Update the **same file**; add a **Revision history** entry at the bottom.
4. Re-present a **short changelog** of what changed.
5. Repeat until the user sends **`LGTM` or `approved`** — that alone closes the review loop. Do not require longer phrasing (e.g. "good to implement").

**On approval**, state:

- Final file path
- Next step: run repo-specific enrichment skills before implementation
- Suggested order: `phoenix-be-implementation-plan` in `phoenix` → `phoenix-fe-implementation-plan` in `phoenix-fe` → implement

---

## Interaction rules

- **One phase at a time** — do not combine clarification with a full design doc in the same turn unless the user already confirmed a prior summary in the same thread.
- **Prefer small question batches** over a single huge questionnaire.
- **Summarize often** — after every 1–2 clarification rounds, show a compact running summary.
- **No invented requirements** — label assumptions clearly; confirm before planning.
- **No implementation** during this skill unless the user explicitly asks to start coding after full planning is done.
- **Keep BE and FE sections separate** inside the document, with a shared API contract section the FE section references.
- **Do not pretend to know code paths** without repo access — stay at design level.

## Handoff after design approval

```text
Design approved: docs/feature-plans/<feature-slug>-build-plan.md

Next:
1. Open phoenix → @phoenix-be-implementation-plan — enrich Backend section
2. Open phoenix-fe → @phoenix-fe-implementation-plan — enrich Frontend + API integration
3. Implement using enriched plan + phoenix-fe-feature (FE) / repo patterns (BE)
```

## Additional resources

- Build plan structure: [plan-template.md](plan-template.md)
