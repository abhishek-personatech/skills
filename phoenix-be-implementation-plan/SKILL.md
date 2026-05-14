---
name: phoenix-be-implementation-plan
description: >-
  Enrich an approved Phoenix feature design document with code-audited backend
  implementation steps in the phoenix repo. Explores controllers, services, entities,
  Flyway migrations, and tests; updates Section 3 of the shared build plan. Use when
  the user has an approved design plan and needs backend coding plan detail, or says
  phoenix-be-implementation-plan, BE implementation plan, or @phoenix-be-implementation-plan.
  Requires the phoenix repository to be open.
disable-model-invocation: true
---

# Phoenix backend implementation plan

Turn an **approved design document** into a **code-audited backend implementation plan** for the `phoenix` repo.

**Prerequisite:** Approved design from `phoenix-feature-planning` (or equivalent design doc with API contract + BE scope).

**Scope:** Backend only. Do not implement feature code in this skill unless the user explicitly asks after plan approval.

## Coding standards & user-provided context

**Always follow project conventions:**

- Read **`CLAUDE.md`** at the `phoenix` repo root when present; if absent, treat **existing code in-repo** as the source of truth.
- Match **existing coding patterns** in the same domain (controller → service → types → tests → migrations).

**When the user attaches reference files** (@-mentions, chat attachments, or paths in the prompt):

- Treat attached files as **primary pattern references** for structure, naming, annotations, error handling, and test style.
- Plan changes that **extend the same patterns** — do not introduce a new style when a reference file already shows the way.
- Cite attached reference file(s) in the enrichment plan (“follow pattern from `…`”).

**Safety constraints** (for the plan and any later implementation):

- **Do not break existing features** — prefer extend/overload/new types over risky refactors; call out regression risk and tests to run.
- **Do not reformat or touch untouched code** — no drive-by cleanup, import reordering, or whitespace changes in files not required for the feature.
- **Minimal diff scope** — only files and methods needed for the feature; avoid broad package moves or unrelated edits.

## Preconditions

- **`phoenix` repo must be open** as the workspace (or agent cwd).
- User provides the design doc path (e.g. `docs/feature-plans/foo-build-plan.md`) or pastes the BE-relevant sections.
- User may attach **relevant reference files** (similar controller, service, test, migration) — read and follow their patterns.
- If the design doc is missing, stop and ask the user to run `phoenix-feature-planning` first.

## Workflow

```
BE Implementation Planning:
- [ ] Phase 1: Load design + confirm BE scope
- [ ] Phase 2: Codebase exploration
- [ ] Phase 3: Draft BE enrichment
- [ ] Phase 4: Review loop until approved
```

## Approval signals

**`LGTM` or `approved` alone is sufficient** to proceed (Phase 1 → Phase 2, or close Phase 4 review loop).

---

### Phase 1: Load design + confirm BE scope

1. Read the design document.
2. Read **user-attached reference files** (if any) and note patterns to mirror.
3. Summarize **BE scope** in 5–10 bullets (endpoints, data changes, services affected).
4. List **open questions** that require code exploration.
5. Ask user to confirm BE scope or **`LGTM` / `approved`** to explore the codebase.

---

### Phase 2: Codebase exploration (required)

Explore `phoenix` before writing paths. Search for similar features and reuse patterns. **Prefer user-attached files** as anchors when provided; otherwise find the nearest in-repo equivalent.

**Primary module:** `registration/` (Gradle/Spring Boot)

| Layer | Typical location | Notes |
|-------|------------------|-------|
| REST controllers | `registration/src/main/java/com/personatech/**/rest/api/**/` | `@RestController`, `@PreAuthorize`, `ResponseEntity` |
| Program controllers | `registration/src/main/java/com/personatech/program/rest/api/**/` | Program/agenda domain |
| API types (DTOs) | `.../rest/types/**/` | `Api*` request/response types |
| Services | `.../service/**/` | Business logic; inject into controllers |
| Entities / models | `com.personatech.common.*` and domain packages | Check existing entities before new tables |
| Flyway migrations | `registration/src/main/resources/db/migration/` | `V*__snake_case_description.sql` |
| Controller tests | `registration/src/test/java/com/personatech/baseunit/controller/**/` | Mockito + `@ExtendWith(MockitoExtension.class)` |
| Service tests | `registration/src/test/java/com/personatech/baseunit/service/**/` | Mirror service package |

**Exploration checklist**

- [ ] Find nearest existing controller + service for the same domain
- [ ] Confirm package naming convention for new types
- [ ] Check whether endpoint belongs under `registration/rest/api` vs `program/rest/api`
- [ ] Identify existing entities/tables to extend vs new migration
- [ ] Read latest migration version prefix for new Flyway file naming
- [ ] Find test examples matching controller vs service level
- [ ] Note auth annotations / role patterns on similar endpoints
- [ ] Check for aspects/validators (`.../service/common/aspect/`)

State **what you searched and what you found** before drafting the plan.

---

### Phase 3: Draft BE enrichment

Update the **same design document file** (preferred) — enrich **Section 3 (Backend plan)** and align **Section 2 (API contract)** if code exploration reveals corrections.

Set metadata:

- `Implementation status` → `BE enriched`
- Add revision history entry

Use [be-enrichment-template.md](be-enrichment-template.md) for section structure.

**Quality bar — every step must include:**

- Exact file path (or explicit “extend existing” path)
- Whether to create vs modify
- Dependency on migration / service / controller order
- Test file path and what to assert
- Manual verification note (endpoint, role, sample payload)

**Do not invent classes** — if uncertain after search, mark `TBD` with the grep/search hint used.

---

### Phase 4: Review loop

1. Present a short summary of BE enrichment + file path.
2. Collect feedback or **`LGTM` / `approved`**.
3. Update the document; revision history entry per change.
4. On approval:
   - Mark BE section ready
   - Remind user to run `phoenix-fe-implementation-plan` in `phoenix-fe` if FE not yet enriched
   - Handoff prompt for BE implementation agent

**Implementation handoff (after approval):**

```text
@phoenix-feature-implementation
@~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md

Implement Section 3 (Backend plan) in phoenix, steps in order.
Follow CLAUDE.md (if present), attached reference files, and patterns cited in the plan.
Do not reformat untouched code.
```

Plan lives in personal skills docs unless copied into the app repo.

## Interaction rules

- **Repo required** — refuse to emit verified paths without exploring `phoenix`.
- **Attached files are authoritative** — when the user provides reference files, mirror their patterns in the plan.
- **Design wins on product scope** — if code exploration suggests scope change, flag it; do not silently change requirements.
- **API contract changes** — if BE exploration changes the contract, update Section 2 and call out FE impact.
- **No breaking changes** — plan additive, isolated changes; note regression tests for touched areas.
- **No cosmetic edits** — plan must not include formatting-only or unrelated file changes.
- **No FE implementation** in this skill.
- **Minimal diffs to design doc** — enrich Section 3; avoid rewriting unrelated sections.

## Additional resources

- BE enrichment section structure: [be-enrichment-template.md](be-enrichment-template.md)
