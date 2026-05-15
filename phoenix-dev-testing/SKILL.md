---
name: phoenix-dev-testing
description: >-
  Repeatable dev testing rounds for Phoenix features: run plan checklists, log
  findings, classify and fix issues, and sign off before marking PRs ready for
  review. Works before or after a draft PR; multiple rounds push to the same
  branch without updating the PR body. Use when the user says phoenix-dev-testing,
  dev testing, dev test round, or during phoenix-worker Phase 6.
disable-model-invocation: true
---

# Phoenix dev testing

Human-led testing on **local**, **dev**, or **staging** after implementation (Phases 4–5). Supports **multiple rounds** of find → fix → retest. Works **before** the first draft PR or **after** a draft PR is open (common when BE must be deployed).

**Read and update** the canonical build plan (`~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md`), especially **Section 9** and plan metadata.

## Related skills

| Need | Skill |
|------|--------|
| Implement fixes | [phoenix-feature-implementation](../phoenix-feature-implementation/SKILL.md) |
| Commit / push / PR | [phoenix-git-workflow](../phoenix-git-workflow/SKILL.md) |
| Full lifecycle | [phoenix-worker](../phoenix-worker/SKILL.md) |

## When this runs

- **phoenix-worker Phase 6** (repeatable).
- Standalone: `@phoenix-dev-testing` + `@plan file`.

Phase 6 and Phase 7 (git/PR) **may interleave** — see [Flows](#flows).

## Flows

| Flow | Order | Typical use |
|------|--------|-------------|
| **pre_pr** | Implement → dev testing round(s) → draft PR → ready | FE-only, local verification |
| **post_draft_pr** | Implement → draft PR (deploy) → dev testing round(s) → ready | Full-stack, needs deployed BE |
| **ongoing** | Draft PR already open → dev testing round N → push (no PR body edit) | Resume mid-testing |

Record `dev_testing_mode` in plan metadata at intake or first dev-testing session (`pre_pr` \| `post_draft_pr`). Mode is a **hint**, not a lock — the user may open a draft PR mid-testing.

## Round workflow

Copy per session:

```
Dev testing — Round <N>:
- [ ] Load plan Sections 3.7, 4.8, 4.7, 6, and Section 9
- [ ] Set metadata: dev_testing_status=in_progress, dev_testing_round=N
- [ ] Run test matrix (user drives manual steps; agent records pass/fail)
- [ ] Log findings in Section 9 (ID, class, repo, fix step, status)
- [ ] Implement fixes (phoenix-feature-implementation) — same branch if PR open
- [ ] Automated tests per phoenix-git-workflow — push only; do not edit PR body
- [ ] Retest failed scenarios; update Section 9
- [ ] Sign-off or schedule Round N+1
```

### Finding classification

| Class | Meaning | Blocks ready-for-review? |
|-------|---------|---------------------------|
| **P0** | Broken vs approved design / spec | Yes — fix this round |
| **P1** | Agreed polish or spec gap | Yes unless user defers in plan |
| **P2** | Nice-to-have | No — log; optional follow-up ticket |
| **out_of_scope** | New feature | No — new ticket; do not expand plan |

### Fix mapping

| Change | Action |
|--------|--------|
| Localized bug/polish (same API, same files) | Fix in place; add sub-step e.g. `FE-8b`, `BE-7b`, or `DT-1` in Section 9; **no** re-enrichment |
| API / contract change | Mini delta in plan Section 2; re-run [phoenix-be-implementation-plan](../phoenix-be-implementation-plan/SKILL.md) or [phoenix-fe-implementation-plan](../phoenix-fe-implementation-plan/SKILL.md) for affected repo only |
| New surface / major UX | Ask user: extend this feature or new plan/ticket |

After fixes: update plan **revision history** and tick relevant items in Sections 3.7 / 4.8.

## Sign-off (gate for ready-for-review)

Set plan metadata when all are true:

- `dev_testing_status`: `passed` or `waived` (waive only on explicit user request — hotfix, etc.)
- No open **P0**; no open **P1** unless user deferred in Section 9 sign-off
- Automated tests green per [phoenix-git-workflow](../phoenix-git-workflow/SKILL.md) for each repo in scope
- Section 9 sign-off checkboxes complete

Then hand off to **phoenix-git-workflow** → **Mark ready for review** (full PR body written **once** — do not update PR description on earlier pushes).

## PR body rule (token discipline)

| Event | PR description |
|-------|----------------|
| Create draft PR | Static **stub** only — see phoenix-git-workflow |
| Dev-testing pushes | **Do not** `gh pr edit` body |
| Mark ready for review | **One** review-ready summary generated from final diff + Section 9 |

## Agent behavior

1. **User drives manual testing** — agent records outcomes, implements fixes, runs automated tests.
2. **One round at a time** — finish fixes and retest before starting the next round unless the user batches findings.
3. **Do not mark PR ready for review** — that is phoenix-git-workflow after sign-off here.
4. **Do not regenerate PR summaries** during rounds — plan Section 9 holds interim truth.
5. **Same branch / same PR** for all rounds unless the user starts a new scope.

## Entry prompts

**Round 1 (pre-PR):**

```text
@phoenix-dev-testing
@~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md

Round 1 — dev testing before draft PR.
Environment: local
Repos in scope: BE / FE / both
```

**Round N (PR already open):**

```text
@phoenix-dev-testing
@~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md

Round 2. Draft PR open — push fixes to same branch; do not edit PR body.

Findings:
- P0: ...
- P1: ...
```

**Sign-off:**

```text
@phoenix-dev-testing
@~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md

Dev testing complete — mark passed and hand off to phoenix-git-workflow for ready-for-review.
```
