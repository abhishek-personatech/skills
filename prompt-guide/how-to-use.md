### 1. Phoenix worker — full feature (primary)

Use `@phoenix-worker` to run the full lifecycle: design → BE/FE enrichment → implementation → git/PR. The agent delegates to child skills one phase at a time. Open **one repo per chat** unless you use a multi-root workspace.

**New feature (no plan yet):**

```
@phoenix-worker

Feature: <one-line summary>
Ticket: PTI-XXXXX
Repos: BE / FE / both
Plan: create new

Start from Phase 0. Guide me through design, enrichment, implementation, and PRs.
```

**Resume with an existing plan:**

```
@phoenix-worker
@~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md

Continue from the current phase. Phases completed so far: <list or "infer from plan">.
```

**Phases (in order):**

| Phase | What | Workspace |
|-------|------|-----------|
| 0 | Intake (ticket, scope) | Any |
| 1 | Design | `phoenix-feature-planning` — no repo required |
| 2 | BE enrichment | Open `phoenix` |
| 3 | FE enrichment | Open `phoenix-fe` |
| 4 | BE implementation | Open `phoenix` |
| 5 | FE implementation | Open `phoenix-fe` |
| 6 | Git / draft PR | Per repo — `phoenix-git-workflow` |
| 7 | Review comments | Repo with PR — `resolve-review-comments` |

Plan file (default): `~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md` — always @-attach it when switching repos or sessions.

---

### 2. Backend — open phoenix

@phoenix-feature-implementation
@~/.cursor/skills/docs/feature-plans/console-meeting-workflow-bulk-actions-build-plan.md

Implement Section 3 (Backend plan) only, steps in order (Section 3.6).
Follow CLAUDE.md and Section 2 API contract.
Do not reformat untouched code.
Start with BE-1 — confirm approach, then implement.

### 3. Frontend — open phoenix-fe (after BE contract is stable / merged)

@phoenix-feature-implementation
@phoenix-fe-feature
@~/.cursor/skills/docs/feature-plans/console-meeting-workflow-bulk-actions-build-plan.md

Implement Section 4 and Section 4.5, steps in order (Section 4.6).
Follow CLAUDE.md, testing skill, and Section 2.
Do not reformat untouched code.
Start with FE-1 — confirm approach, then implement.

### 4. Resume mid-flight (implementation only)

@phoenix-feature-implementation
@~/.cursor/skills/docs/feature-plans/console-meeting-workflow-bulk-actions-build-plan.md

Continue from step BE-4. BE-1–BE-3 are done.
