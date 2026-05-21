### 1. Phoenix worker — full feature (primary)

Use `@phoenix-worker` to run the full lifecycle: design → enrichment → implementation → **dev testing** → git/PR → review. Not every feature needs both repos — say **BE only**, **FE only**, or **both** up front; the agent will skip the other side only with your explicit consent. Open **one repo per chat** unless you use a multi-root workspace.

**New feature (no plan yet):**

```
@phoenix-worker

Feature: <one-line summary>
Ticket: PTI-XXXXX
Repos: BE / FE / both  ← e.g. "FE only — no backend changes"
Dev testing: pre_pr | post_draft_pr  ← optional; post_draft_pr common for full-stack
Plan: create new

Start from Phase 0. Guide me through design, enrichment, implementation, dev testing, and PRs.
```

**BE-only:**

```
@phoenix-worker

Feature: <summary>
Ticket: PTI-XXXXX
Repos: BE only — no phoenix-fe changes
Plan: create new
```

**FE-only:**

```
@phoenix-worker

Feature: <summary>
Ticket: PTI-XXXXX
Repos: FE only — existing APIs, no backend changes
Plan: create new
```

**Resume with an existing plan:**

```
@phoenix-worker
@~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md

Continue from the current phase. Phases completed so far: <list or "infer from plan">.
```

**Phases (in order; Phase 6 ↔ 7 may interleave):**

| Phase | What | Workspace |
|-------|------|-----------|
| 0 | Intake (ticket, scope, **repo scope: BE / FE / both**) | Any |
| 1 | Design | `phoenix-feature-planning` — no repo required |
| 2 | BE enrichment | Open `phoenix` — skip if FE-only |
| 3 | FE enrichment | Open `phoenix-fe` — skip if BE-only |
| 4 | BE implementation | Open `phoenix` — skip if FE-only |
| 5 | FE implementation | Open `phoenix-fe` — skip if BE-only |
| 6 | Dev testing (repeatable rounds) | `phoenix-dev-testing` — before or after draft PR |
| 7 | Git / PR | `phoenix-git-workflow` — stub at draft; **full body only at ready-for-review** |
| 8 | Review comments | Repo with PR — `resolve-review-comments` |

Plan file (default): `~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md` — always @-attach it when switching repos or sessions.

**Dev testing only (mid-flight):**

```
@phoenix-dev-testing
@~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md

Round 2. Draft PR open — push fixes; do not edit PR body until ready for review.
```

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

### 5. Mark PR ready for review (after dev testing sign-off)

@phoenix-git-workflow
@~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md

Dev testing passed. Write full PR description once, then gh pr ready. Do not update PR body on earlier pushes.

### 6. Review someone else's phoenix-fe PR

Open `phoenix-fe` (or provide PR number). Attach build plan if you have it.

```
@phoenix-fe-reviewer

PR: personatech-infra/phoenix-fe#<number>
Ticket: PTI-XXXXX

Read-only review — do not edit or commit code.

1. Does the PR achieve its stated goal (feature/bugfix/task)?
2. Were my prior review comments addressed (if any)?
3. CLAUDE.md + phoenix-fe-feature / phoenix-scss / testing for touched areas.
4. Suggest improvements and optimizations in review text only.
5. Post to GitHub only if I ask.
```

Optional deep checklist: `~/.cursor/skills/phoenix-fe-reviewer/review-checklist.md`
