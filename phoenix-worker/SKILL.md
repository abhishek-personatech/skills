---
name: phoenix-worker
description: >-
  Orchestrates end-to-end Phoenix feature delivery across phoenix (BE) and
  phoenix-fe (FE): design → repo enrichment → implementation → git/PR. Supports
  BE-only, FE-only, or full-stack scope with user confirmation. Delegates to
  phoenix-feature-planning, phoenix-be-implementation-plan,
  phoenix-fe-implementation-plan, phoenix-feature-implementation, and
  phoenix-git-workflow. Use for full features, single-repo features, cross-repo
  work, or when the user says phoenix-worker, deliver a feature, or
  @phoenix-worker.
disable-model-invocation: true
---

# Phoenix worker

Thin **orchestrator** for delivering a feature across Phoenix repos (BE only, FE only, or both). **Read and follow** the child skill named in each phase — do not duplicate their rules here.

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

Track progress in the plan’s metadata and revision history; update **Implementation status** as phases complete. Record **repo scope** in the plan (`BE`, `FE`, or `both`).

## Repo scope (BE / FE / both)

Not every feature touches both repos. **Confirm scope with the user in Phase 0** (and again at design approval if it was unclear). Use **AskQuestion** when helpful.

| Scope | Include | Skip (with user consent) |
|-------|---------|----------------------------|
| **Both** (default) | Phases 2–5 for BE and FE | — |
| **BE only** | Phases 2, 4, 6 for `phoenix` | Phases 3, 5 (FE enrichment + implementation) |
| **FE only** | Phases 3, 5, 6 for `phoenix-fe` | Phases 2, 4 (BE enrichment + implementation) |

**Rules:**

- Do **not** skip a repo’s phases without explicit user consent — infer from the prompt only when the user clearly states BE-only or FE-only.
- Record the agreed scope in the build plan (metadata or intake section) so later sessions stay aligned.
- **FE-only:** design may still reference existing APIs; no BE enrichment unless the user later adds BE work.
- **BE-only:** design should not assume FE delivery in the same feature; mark FE sections N/A in the plan.
- **Both repos, but API unchanged:** still run FE enrichment/implementation; skip BE phases only if the user confirms no `phoenix` changes.
- Phase 6 (git/PR) runs only for repos that had implementation in scope.

## Lifecycle (strict order)

Copy and update this checklist each session:

```
Phoenix worker progress:
- [ ] Phase 0: Intake (ticket, scope, repo scope: BE / FE / both — user confirmed)
- [ ] Phase 1: Design — phoenix-feature-planning
- [ ] Phase 2: BE enrichment — phoenix-be-implementation-plan (phoenix) [skip if FE-only]
- [ ] Phase 3: FE enrichment — phoenix-fe-implementation-plan (phoenix-fe) [skip if BE-only]
- [ ] Phase 4: BE implementation — phoenix-feature-implementation (phoenix) [skip if FE-only]
- [ ] Phase 5: FE implementation — phoenix-feature-implementation (phoenix-fe) [skip if BE-only]
- [ ] Phase 6: Git / draft PR — phoenix-git-workflow (each repo in scope)
- [ ] Phase 7: Review — resolve-review-comments (if needed)
```

### Gates — do not skip

| Before starting | Requires |
|-----------------|----------|
| Phase 2 (BE enrichment) | Design approved; **repo scope includes BE** (user confirmed) |
| Phase 3 (FE enrichment) | Design approved; **repo scope includes FE** (user confirmed); run BE enrichment first when API is new or changed and BE is in scope |
| Phase 4 (BE code) | BE in scope; Section 3 enriched |
| Phase 5 (FE code) | FE in scope; Section 4 + 4.5 enriched; if BE is in scope, BE merged or contract frozen before FE when API changed |
| Phase 6 (PR) | Tests pass per [phoenix-git-workflow](../phoenix-git-workflow/SKILL.md) for each repo in scope |

If scope is unclear after design, ask before skipping any repo’s phases.

## Cursor workspace reality

- **Default:** one repo per chat — run phases in separate sessions when the workspace changes.
- **Order:** backend before frontend when Section 2 introduces or changes endpoints.
- **Multi-root** (both repos open): allowed for advanced cases; still @ the plan file from `~/.cursor/skills/...`.

## Agent behavior

1. **One phase at a time** — finish the current child skill’s workflow before advancing.
2. **Confirm repo scope in Phase 0** — do not skip BE or FE phases without user consent; re-confirm at design approval if scope was tentative.
3. **Read the child SKILL.md** at phase start; follow it fully for that phase.
4. **Summarize handoff** at each gate: what completed, plan path, next skill, which repo to open.
5. **Do not re-plan or re-enrich** mid-implementation unless the user asks or the plan and code diverge materially.
6. **Do not start git workflow** until implementation and tests for that repo are done.

## Entry prompt (full feature)

```text
@phoenix-worker

Feature: <one-line summary>
Ticket: PTI-XXXXX (if known)
Repos: BE / FE / both  ← confirm; skip the other repo only with your explicit OK

Run from the current phase. If no plan exists, start Phase 0 → Phase 1.
```

**BE-only example:**

```text
@phoenix-worker

Feature: Add export endpoint for meeting summaries
Ticket: PTI-XXXXX
Repos: BE only — no phoenix-fe changes
Plan: create new
```

**FE-only example:**

```text
@phoenix-worker

Feature: Update bulk-action labels and empty state copy
Ticket: PTI-XXXXX
Repos: FE only — uses existing APIs, no backend changes
Plan: create new
```

## Resume / partial delivery

```text
@phoenix-worker
@~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md

Continue from Phase <N> (or step BE-4 / FE-2). Phases 0–<N-1> are done.
```

Infer the current phase from plan metadata, revision history, and user message; confirm before coding if unclear.

## Phase handoffs (quick reference)

Handoffs depend on **repo scope**. Skip steps for repos the user excluded.

**After Phase 1 (design approved) — BE in scope:**

```text
Open phoenix → @phoenix-be-implementation-plan + @plan file
```

**After Phase 1 — FE only (no BE):**

```text
Open phoenix-fe → @phoenix-fe-implementation-plan + @plan file
```

**After Phase 2 (BE enrichment) — FE in scope:**

```text
Open phoenix-fe → @phoenix-fe-implementation-plan + @plan file
```

**After enrichment — BE implementation (BE in scope):**

```text
Open phoenix → @phoenix-feature-implementation + @plan file
Implement Section 3 only (BE steps in order).
```

**After BE done (or FE-only after enrichment) — FE in scope:**

```text
Open phoenix-fe → @phoenix-feature-implementation + @phoenix-fe-feature + @plan file
Implement Section 4 + 4.5 (FE steps in order).
```

**After implementation (per repo in scope):**

```text
@phoenix-git-workflow — branch, commit, push, draft PR; link plan path in PR description.
```

**After PR review comments:**

```text
@resolve-review-comments in the repo with the PR open.
```
