---
name: phoenix-feature-planning
description: >-
  Turn raw feature requirements into a reviewed, agent-ready build plan for Phoenix
  backend (phoenix) and frontend (phoenix-fe). Runs a clarification loop, requirement
  summary, confirmation gate, then generates separate BE/FE implementation plans with
  API integration details. Use when planning a feature, writing a build plan, technical
  design, implementation approach, or when the user says phoenix-feature-planning,
  feature build plan, or @phoenix-feature-planning.
disable-model-invocation: true
---

# Phoenix feature planning

Produce a **reviewed, agent-executable build plan** from a raw requirement. Do not jump to implementation until the plan is approved.

## Repositories

| Layer | Repo | Guidelines |
|-------|------|------------|
| Backend | `phoenix` | Explore existing controllers, services, entities, migrations, and tests in-repo before proposing changes |
| Frontend | `phoenix-fe` | Follow `CLAUDE.md`; for implementation/testing after planning, see `phoenix-fe-feature` and `.cursor/skills/testing/SKILL.md` |

This skill is **planning only**. It does not implement code.

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
4. If the codebase is open or paths are known, **explore relevant code** in `phoenix` / `phoenix-fe` to ground questions (existing endpoints, similar features, naming patterns). State what you inspected.

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

### Phase 4: Build plan draft

After confirmation, create **one markdown file** using [plan-template.md](plan-template.md).

**Output location** (default):

- If a planning/docs folder exists in the open workspace, use it.
- Otherwise create: `docs/feature-plans/<feature-slug>-build-plan.md`
- Use kebab-case for `<feature-slug>` (e.g. `daily-meeting-summary-export`).

Tell the user the file path and ask for review.

**Plan quality bar** — each section must be concrete enough that **another agent** can implement without guessing:

- Name files, modules, layers, and patterns to follow
- List API endpoints with method, path, request/response shape, auth, errors
- Order work into sequenced steps with dependencies
- Call out migrations, tests, and verification per layer
- Separate **Backend plan** and **Frontend plan** clearly
- Frontend must include a dedicated **API integration** section tied to backend contracts

If repos are available locally, anchor paths to real locations (e.g. `registration/.../ProgramAgendaController.java`). If not, use `TBD — verify in repo` and note search hints.

---

### Phase 5: Review loop

1. Ask the user to review the build plan file.
2. Collect feedback (inline comments, bullet list, or "LGTM").
3. Update the **same file**; add a **Revision history** entry at the bottom.
4. Re-present a **short changelog** of what changed.
5. Repeat until the user sends **`LGTM` or `approved`** — that alone closes the review loop. Do not require longer phrasing (e.g. "good to implement").

**On approval**, state:

- Final file path
- Suggested implementation order (typically BE contract → FE integration → E2E verification)
- Which skills to use next (`phoenix-fe-feature`, repo testing skill, etc.)

---

## Interaction rules

- **One phase at a time** — do not combine clarification with a full build plan in the same turn unless the user already confirmed a prior summary in the same thread.
- **Prefer small question batches** over a single huge questionnaire.
- **Summarize often** — after every 1–2 clarification rounds, show a compact running summary.
- **No invented requirements** — label assumptions clearly; confirm before planning.
- **No implementation** during this skill unless the user explicitly asks to start coding after plan approval.
- **Keep BE and FE plans separate** inside the document (two major sections), with a shared API contract section the FE plan references.

## Handoff to implementation agents

The approved plan should enable an implementation agent to:

1. Read the **Backend plan** section and work in `phoenix` only.
2. Read the **Frontend plan** + **API integration** sections and work in `phoenix-fe`.
3. Execute steps in dependency order without re-deriving architecture.

Recommend the user start implementation with:

```text
Implement [feature] per docs/feature-plans/<feature-slug>-build-plan.md.
Backend first (or FE with mocked API if noted in plan).
Use phoenix-fe-feature for FE implementation.
```

## Additional resources

- Build plan structure: [plan-template.md](plan-template.md)
