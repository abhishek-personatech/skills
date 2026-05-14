---
name: phoenix-feature-implementation
description: >-
  Implement an enriched Phoenix feature build plan across phoenix (BE) and
  phoenix-fe (FE) when the plan lives in personal skills docs
  (~/.cursor/skills/docs/feature-plans/). Use after phoenix-feature-planning and
  repo enrichment are done, or when the user says implement the build plan,
  build the feature from the plan, phoenix-feature-implementation, or
  @phoenix-feature-implementation.
disable-model-invocation: true
---

# Phoenix feature implementation (from personal skills plan)

Use this skill when a **build plan is approved and enriched** and you are ready to **write code** in `phoenix` and/or `phoenix-fe`.

## Plan location (canonical)

Personal skills repo — **not** inside `phoenix` or `phoenix-fe`:

```text
~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md
```

Example:

```text
~/.cursor/skills/docs/feature-plans/console-meeting-workflow-bulk-actions-build-plan.md
```

Always **@-attach the plan file** in the implementation prompt so the agent can read it even when the plan is outside the open repo workspace.

## Preconditions

- Design approved (`phoenix-feature-planning`)
- **Backend:** Section 3 enriched (`phoenix-be-implementation-plan` in `phoenix`) — required before FE if API is new/changed
- **Frontend:** Section 4 + 4.5 enriched (`phoenix-fe-implementation-plan` in `phoenix-fe`)
- Plan metadata: `Implementation status` → `Ready to implement` (or note which side is still pending)

## Cursor workspace reality

- **Default:** one repo open per chat — run **BE implementation in `phoenix`**, then **FE implementation in `phoenix-fe`** (separate chats or sessions).
- **Optional:** multi-root workspace with both repos — one chat can implement both if both roots are open; still @ the plan from `~/.cursor/skills/...`.

**Order:** Backend first when Section 2 contract or endpoints are new; frontend after BE is merged or contract is frozen.

---

## Prompt templates (copy-paste)

Replace `<feature-slug>` and paths as needed. Attach the plan file with `@`.

### Backend — open `phoenix` workspace

```text
@phoenix-feature-implementation
@~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md

Implement **Section 3 (Backend plan)** only, in order (Section 3.6 steps).

- Follow CLAUDE.md and patterns cited in the plan.
- Implement exactly against **Section 2** API contract.
- Do not reformat untouched code.
- Run relevant tests before finishing.

Start with step BE-1. Confirm approach for BE-1, then implement.
```

Optional: attach a reference controller/service from `phoenix` as a pattern anchor.

### Frontend — open `phoenix-fe` workspace

```text
@phoenix-feature-implementation
@phoenix-fe-feature
@~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md

Implement **Section 4 (Frontend plan)** and **Section 4.5 API integration**, in order (Section 4.6 steps).

- Follow CLAUDE.md and .cursor/skills/testing/SKILL.md for tests.
- Follow phoenix-scss for any new styles.
- Implement exactly against **Section 2**; use enriched enum/dataLoader names from the plan.
- Do not reformat untouched code.

Start with step FE-1. Confirm approach for FE-1, then implement.
```

Optional: attach a similar feature module from `phoenix-fe` as a pattern anchor.

### Both repos — multi-root workspace (advanced)

```text
@phoenix-feature-implementation
@~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md

Implement this feature end-to-end:
1. Section 3 in phoenix (BE steps in order)
2. Section 4 + 4.5 in phoenix-fe (FE steps in order)

Confirm BE-1 approach before coding. After BE is done and tests pass, move to FE-1.
```

### Resume / partial implementation

```text
@phoenix-feature-implementation
@~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md

Continue implementation from step **BE-4** (or **FE-3**). Steps BE-1–BE-3 are already merged.
```

---

## Agent behavior during implementation

1. Read the plan file; respect **Section 2** as contract.
2. Execute **ordered steps** in the relevant section only unless the user widens scope.
3. **Confirm before the first step** (and before any step that changes UX or API shape).
4. After each logical chunk: run targeted tests; summarize what shipped vs plan.
5. If the plan and code diverge, **update the plan revision history** or flag the user — do not silently drift from Section 2.
6. Do not start `phoenix-feature-planning` or re-enrich unless the user asks.

## Repo-specific skills (reference)

| Repo | Coding skill / conventions |
|------|----------------------------|
| `phoenix-fe` | `phoenix-fe-feature`, `phoenix-scss`, `CLAUDE.md`, `.cursor/skills/testing/SKILL.md` |
| `phoenix` | `CLAUDE.md`, patterns in plan Section 3 |

## After implementation

- Open PRs per repo; link the same plan path in PR description.
- For review feedback: `resolve-review-comments` in the repo with the PR open.
