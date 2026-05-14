---
name: phoenix-worker
description: >-
  Orchestrates end-to-end Phoenix feature delivery across phoenix (BE) and
  phoenix-fe (FE): design → repo enrichment → implementation → git/PR. Delegates
  to phoenix-feature-planning, phoenix-be-implementation-plan,
  phoenix-fe-implementation-plan, phoenix-feature-implementation, and
  phoenix-git-workflow. Use for full features, cross-repo work, or when the user
  says phoenix-worker, deliver a feature, or @phoenix-worker.
disable-model-invocation: true
---

# Phoenix worker

Thin **orchestrator** for delivering a feature across Phoenix repos. **Read and follow** the child skill named in each phase — do not duplicate their rules here.

## Child skills

| Phase | Skill | Repo / workspace |
|-------|-------|------------------|
| Design | [phoenix-feature-planning](../phoenix-feature-planning/SKILL.md) | No repo required |
| BE enrichment | [phoenix-be-implementation-plan](../phoenix-be-implementation-plan/SKILL.md) | `phoenix` |
| FE enrichment | [phoenix-fe-implementation-plan](../phoenix-fe-implementation-plan/SKILL.md) | `phoenix-fe` |
| Implementation | [phoenix-feature-implementation](../phoenix-feature-implementation/SKILL.md) | `phoenix` then `phoenix-fe` |
| Git / PR | [phoenix-git-workflow](../phoenix-git-workflow/SKILL.md) | Per repo |
| Review feedback | [resolve-review-comments](../resolve-review-comments/SKILL.md) | Repo with open PR |

**During FE implementation**, also use [phoenix-fe-feature](../phoenix-fe-feature/SKILL.md) and [phoenix-scss](../phoenix-scss/SKILL.md) when styling is in scope.

## Canonical plan file

```text
~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md
```

Always **@-attach the plan file** when switching repos or sessions so the agent can read it outside the open workspace.

Track progress in the plan’s metadata and revision history; update **Implementation status** as phases complete.

## Lifecycle (strict order)

Copy and update this checklist each session:

```
Phoenix worker progress:
- [ ] Phase 0: Intake (ticket, scope, repos in/out)
- [ ] Phase 1: Design — phoenix-feature-planning
- [ ] Phase 2: BE enrichment — phoenix-be-implementation-plan (phoenix)
- [ ] Phase 3: FE enrichment — phoenix-fe-implementation-plan (phoenix-fe)
- [ ] Phase 4: BE implementation — phoenix-feature-implementation (phoenix)
- [ ] Phase 5: FE implementation — phoenix-feature-implementation (phoenix-fe)
- [ ] Phase 6: Git / draft PR — phoenix-git-workflow (each repo)
- [ ] Phase 7: Review — resolve-review-comments (if needed)
```

### Gates — do not skip

| Before starting | Requires |
|-----------------|----------|
| Phase 2 (BE enrichment) | Design approved (`LGTM` / `approved` on build plan) |
| Phase 3 (FE enrichment) | Section 2 API contract stable; **run BE enrichment first** when API is new or changed |
| Phase 4 (BE code) | Section 3 enriched; plan status ready for BE |
| Phase 5 (FE code) | Section 4 + 4.5 enriched; BE merged or contract frozen |
| Phase 6 (PR) | Tests pass per [phoenix-git-workflow](../phoenix-git-workflow/SKILL.md) |

Skip phases only when the user explicitly scopes them out (e.g. BE-only feature → skip Phase 3 and 5).

## Cursor workspace reality

- **Default:** one repo per chat — run phases in separate sessions when the workspace changes.
- **Order:** backend before frontend when Section 2 introduces or changes endpoints.
- **Multi-root** (both repos open): allowed for advanced cases; still @ the plan file from `~/.cursor/skills/...`.

## Agent behavior

1. **One phase at a time** — finish the current child skill’s workflow before advancing.
2. **Read the child SKILL.md** at phase start; follow it fully for that phase.
3. **Summarize handoff** at each gate: what completed, plan path, next skill, which repo to open.
4. **Do not re-plan or re-enrich** mid-implementation unless the user asks or the plan and code diverge materially.
5. **Do not start git workflow** until implementation and tests for that repo are done.

## Entry prompt (full feature)

```text
@phoenix-worker

Feature: <one-line summary>
Ticket: PTI-XXXXX (if known)
Repos: BE / FE / both
Plan: <path or "create new">

Run from the current phase. If no plan exists, start Phase 0 → Phase 1.
```

## Resume / partial delivery

```text
@phoenix-worker
@~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md

Continue from Phase <N> (or step BE-4 / FE-2). Phases 0–<N-1> are done.
```

Infer the current phase from plan metadata, revision history, and user message; confirm before coding if unclear.

## Phase handoffs (quick reference)

**After Phase 1 (design approved):**

```text
Open phoenix → @phoenix-be-implementation-plan + @plan file
```

**After Phase 2:**

```text
Open phoenix-fe → @phoenix-fe-implementation-plan + @plan file
```

**After Phase 3:**

```text
Open phoenix → @phoenix-feature-implementation + @plan file
Implement Section 3 only (BE steps in order).
```

**After Phase 4:**

```text
Open phoenix-fe → @phoenix-feature-implementation + @phoenix-fe-feature + @plan file
Implement Section 4 + 4.5 (FE steps in order).
```

**After Phase 5 (per repo):**

```text
@phoenix-git-workflow — branch, commit, push, draft PR; link plan path in PR description.
```

**After PR review comments:**

```text
@resolve-review-comments in the repo with the PR open.
```
