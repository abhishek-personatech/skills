---
name: phoenix-fe-implementation-plan
description: >-
  Enrich an approved Phoenix feature design document with code-audited frontend
  implementation steps in phoenix-fe. Explores apps, routes, API enums, feature
  folders, i18n, and tests; updates Section 4 and API integration of the shared
  build plan. Use when the user has an approved design plan and needs frontend coding
  plan detail, or says phoenix-fe-implementation-plan, FE implementation plan, or
  @phoenix-fe-implementation-plan. Requires the phoenix-fe repository to be open.
disable-model-invocation: true
---

# Phoenix frontend implementation plan

Turn an **approved design document** into a **code-audited frontend implementation plan** for the `phoenix-fe` repo.

**Prerequisite:** Approved design from `phoenix-feature-planning`. Prefer **Section 2 API contract** to be stable (run `phoenix-be-implementation-plan` first when BE contract may still change).

**Scope:** Frontend only. Do not implement feature code in this skill unless the user explicitly asks after plan approval.

## Coding standards & user-provided context

**Always follow project conventions:**

- Read and follow **`CLAUDE.md`** at the `phoenix-fe` repo root for coding guidelines, architecture, and API patterns.
- Match **existing coding patterns** in the same app/surface (console, my-experience, etc.).
- For tests, align with **`.cursor/skills/testing/SKILL.md`**; for SCSS, align with **`phoenix-scss`** when styling is in scope.

**When the user attaches reference files** (@-mentions, chat attachments, or paths in the prompt):

- Treat attached files as **primary pattern references** for folder layout, hooks, API wiring, components, copy usage, and tests.
- Plan changes that **extend the same patterns** — do not introduce a new structure or style when a reference file already shows the way.
- Cite attached reference file(s) in the enrichment plan (“follow pattern from `…`”).

**Safety constraints** (for the plan and any later implementation):

- **Do not break existing features** — avoid changing shared components, routes, or APIs unless explicitly in scope; call out regression risk and tests to run.
- **Do not reformat or touch untouched code** — no drive-by cleanup, import reordering, or whitespace changes in files not required for the feature.
- **Minimal diff scope** — only files needed for the feature; no unrelated refactors.

## Preconditions

- **`phoenix-fe` repo must be open** as the workspace.
- User provides design doc path or FE-relevant sections.
- User may attach **relevant reference files** (similar feature module, dataLoader, route, test) — read and follow their patterns.
- If design doc is missing, stop and ask user to run `phoenix-feature-planning` first.

## Workflow

```
FE Implementation Planning:
- [ ] Phase 1: Load design + confirm FE scope
- [ ] Phase 2: Codebase exploration
- [ ] Phase 3: Draft FE enrichment
- [ ] Phase 4: Review loop until approved
```

## Approval signals

**`LGTM` or `approved` alone is sufficient** to proceed (Phase 1 → Phase 2, or close Phase 4 review loop).

---

### Phase 1: Load design + confirm FE scope

1. Read the design document (especially Section 2 API contract + FE design).
2. Read **user-attached reference files** (if any) and note patterns to mirror.
3. Summarize **FE scope**: surfaces, routes, user flows, API calls needed.
4. List open questions for code exploration.
5. Ask user to confirm or **`LGTM` / `approved`** to explore.

---

### Phase 2: Codebase exploration (required)

Explore `phoenix-fe` before writing paths. **Prefer user-attached files** as anchors when provided; otherwise find the nearest in-repo equivalent.

| Area | Typical location | Notes |
|------|------------------|-------|
| Apps | `src/apps/console/`, `src/apps/myExperience/`, registration app areas | Confirm target surface from design |
| Routes | `src/apps/console/routes/`, myExperience routes | How feature is reached |
| API endpoint enums | `src/apps/console/enums/consoleApi.ts`, `src/apps/myExperience/enum/MyExpApi.ts` | Add or reuse endpoint constants |
| Data loaders / API calls | `**/dataLoader.ts`, service hooks | Follow existing axios + React Query patterns per `CLAUDE.md` |
| Feature modules | under relevant `modules/` tree | Mirror neighboring feature folder layout |
| i18n / copies | `I18n` + `useCopies()` | No hardcoded UI strings |
| Tests | `__test__/` next to feature | `consoleRender` / `myExperienceRender`, `mockGet`/`mockPost` |
| SCSS | co-located `.scss` | See `phoenix-scss` skill |

**Exploration checklist**

- [ ] Identify target app(s): console vs my-experience vs registration
- [ ] Find closest existing feature module (structure + service hook pattern)
- [ ] Locate API enum file and naming pattern for new endpoints
- [ ] Trace how similar feature wires React Query / dataLoader
- [ ] Check route registration pattern
- [ ] Find copy key patterns for similar UI
- [ ] Find integration test example for same app surface
- [ ] Confirm Section 2 API paths match FE enum/base URL conventions

State **what you searched and what you found**.

---

### Phase 3: Draft FE enrichment

Update the **same design document** — enrich **Section 4 (Frontend plan)** and **Section 4.5 API integration**. Align with Section 2; flag mismatches.

Set metadata:

- `Implementation status` → `FE enriched` (or `Ready to implement` if BE already enriched)
- Add revision history entry

Use [fe-enrichment-template.md](fe-enrichment-template.md).

**Quality bar — every step must include:**

- Exact feature folder path under `src/apps/...`
- API enum constant / dataLoader function names (proposed, matching repo style)
- Route path or parent module entry point
- i18n key naming approach
- Test file path + render helper + scenarios
- Loading / error / empty handling tied to specific API calls

Propose feature structure consistent with **`phoenix-fe-feature`** personal skill:

- `index.tsx`, `service.ts`, `types.ts`, `components/`, optional `context/`, `__test__/`

---

### Phase 4: Review loop

1. Summarize FE enrichment + file path.
2. Collect feedback or **`LGTM` / `approved`**.
3. Update document + revision history.
4. On approval:
   - Set `Implementation status` to `Ready to implement` when BE + FE plans are enriched (or note BE pending)
   - Handoff to implementation

**Implementation handoff (after approval):**

```text
@phoenix-feature-implementation
@phoenix-fe-feature
@~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md

Implement Section 4 + Section 4.5 in phoenix-fe, steps in order.
Follow CLAUDE.md, attached reference files, and patterns cited in the plan.
Use .cursor/skills/testing/SKILL.md for tests. Do not reformat untouched code.
```

Plan lives in personal skills docs unless copied into the app repo.

## Interaction rules

- **Repo required** — refuse verified paths without exploring `phoenix-fe`.
- **CLAUDE.md + attached files are authoritative** — prefer user-provided reference files over inventing new patterns.
- **API integration section is mandatory** — map each UI action to Section 2 endpoint + service function + types.
- **Do not implement** unless user asks post-approval.
- **No breaking changes** — plan isolated changes; note regression tests for touched flows.
- **No cosmetic edits** — plan must not include formatting-only or unrelated file changes.
- **Flag BE contract gaps** — if FE cannot map to an endpoint, stop and note BE plan update needed.
- **No backend code** in this skill.

## Additional resources

- FE enrichment section structure: [fe-enrichment-template.md](fe-enrichment-template.md)
