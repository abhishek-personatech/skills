---
name: phoenix-fe-reviewer
description: >-
  Review phoenix-fe pull requests for coding standards (CLAUDE.md), feature
  structure, SCSS, tests, intended outcome, regression risk, and shared-component
  impact. Flags duplicate components/utils and weak typing. Use when reviewing
  others' PRs, doing FE code review, or when the user says phoenix-fe-reviewer,
  review this PR, or @phoenix-fe-reviewer.
disable-model-invocation: true
---

# Phoenix FE reviewer

Structured review for **someone else's** `phoenix-fe` PR. You are a reviewer, not the author — do not implement fixes unless the user asks.

**Not for:** addressing review feedback on your own PR → use [resolve-review-comments](../resolve-review-comments/SKILL.md).

## Prerequisites

| Need | Action |
|------|--------|
| Standards source | Read **`CLAUDE.md`** at the `phoenix-fe` repo root — it wins over this skill and child skills if anything conflicts |
| Feature / SCSS / tests | Read [phoenix-fe-feature](../phoenix-fe-feature/SKILL.md), [phoenix-scss](../phoenix-scss/SKILL.md), and **`phoenix-fe/.cursor/skills/testing/SKILL.md`** when those areas appear in the diff |
| PR diff | Prefer `phoenix-fe` checked out; use `gh` when reviewing remotely |
| Intended outcome | PR description, JIRA/ticket, and/or user-attached build plan (`~/.cursor/skills/docs/feature-plans/...`) |

If `phoenix-fe` is not open, use `gh pr diff` / `gh pr view` against `personatech-infra/phoenix-fe` and still read `CLAUDE.md` from the default branch if needed (`gh api` or clone).

## Review workflow

Copy and track:

```
FE PR review:
- [ ] 1. Context — ticket, goal, acceptance criteria
- [ ] 2. Scope — files touched, blast radius
- [ ] 3. Outcome — does the diff achieve the stated goal?
- [ ] 4. Standards — CLAUDE.md + feature + SCSS + testing skills
- [ ] 5. Shared / cross-feature — regression and consumer impact
- [ ] 6. Reuse — no duplicate components or utils
- [ ] 7. Quality — clarity, concision, type safety, tests
- [ ] 8. Verdict — structured report
```

### 1) Establish context

- Read PR title, body, linked ticket, and labels.
- Restate **intended outcome** in one paragraph (what should work for the user after merge).
- Note **explicit out-of-scope** claims in the PR; flag changes that violate them.

```bash
gh pr view <number> --repo personatech-infra/phoenix-fe
gh pr diff <number> --repo personatech-infra/phoenix-fe
```

### 2) Map scope and risk

- List changed paths; group by app/surface (`console`, `myExperience`, registration, etc.).
- Flag any path under:
  - `src/shared/` (components, hooks, utils)
  - `src/assets/styles/` (global tokens/themes)
  - shared routes, API clients, enums used app-wide
  - `I18n` / copy infrastructure
- Estimate **blast radius**: who else imports or styles the same symbols?

### 3) Validate intended outcome

- Trace the main user flow in the diff (entry → state → API → UI → edge cases).
- Check loading, error, empty, disabled, and permission-gated paths.
- Confirm async flows cannot dead-end (empty optional API fields, stale effect deps, props vs state for visibility).
- If outcome is unclear from the diff, mark **🟡 Question** — do not assume intent.

### 4) Coding standards (required reads)

Apply **`CLAUDE.md`** first, then skill guardrails for touched areas:

| Area | Check |
|------|--------|
| **Feature structure** | Thin `index.tsx`; logic in `service.ts`; types in `types.ts`; context only when needed ([phoenix-fe-feature](../phoenix-fe-feature/SKILL.md)) |
| **Copy** | No hardcoded user-facing strings; `I18n` + `useCopies()` |
| **SCSS** | BEM-like names, tokens, custom-property roots ([phoenix-scss](../phoenix-scss/SKILL.md)) |
| **Tests** | Coverage for loading/success/error/primary paths and critical edges per testing skill |
| **Diff hygiene** | No drive-by refactors, reformatting, or unrelated files |

### 5) Shared and cross-feature impact (high scrutiny)

Treat changes under `src/shared/`, widely used console components, shared hooks, and global styles as **high risk**.

For each shared change:

1. **Necessity** — Could the feature stay local (wrapper, existing variant, feature SCSS)?
2. **Minimality** — Smallest API/surface change; no speculative props.
3. **Consumers** — Grep importers/usages; list features that could regress.
4. **Contract** — Breaking prop/type/behavior changes called out explicitly.

**🔴 Critical** if a shared change broadens API without strong justification or breaks existing callers.

Feature work should **not** modify shared code unless the PR explains why feature-local workarounds failed ([phoenix-fe-feature](../phoenix-fe-feature/SKILL.md)).

### 6) Reuse — no duplicate components or utils

Before approving **new** files under `components/`, `shared/`, or `utils/`:

- Search the repo for existing components/hooks with the same responsibility (name, UI pattern, prop shape).
- Prefer extending an existing component (variant, composition) over a parallel implementation.
- Same rule for **utility functions** — search `src/utils`, feature-local helpers, and lodash usage patterns already in the module.

**🔴 Critical** for near-duplicate components/utils when an existing one could be extended with a small, safe change.

### 7) Code quality bar

| Topic | Review for |
|-------|------------|
| **Clarity** | Self-explanatory names; logic obvious without excessive comments; no clever one-liners that hide behavior |
| **Concision** | No dead code, unused imports, commented-out blocks, or stale TODOs in shipped paths |
| **Type safety** | No unjustified `any`; public props and API fields typed; narrow types for unions/enums |
| **React** | Stable callbacks/memo where passed deep; narrow effect deps; no effect loops on unstable objects |
| **Security / data** | Sensitive data not logged; correct guards on optional API fields |

### 8) Tests and verification

- New/changed behavior should have tests per **testing** skill; flag missing matrices for user-facing flows.
- Note what the author ran (from PR body or CI); suggest targeted commands when gaps exist:
  ```bash
  yarn test --testPathPattern=<feature> --watchAll=false
  yarn lint
  yarn check-types
  ```

## Severity and report format

Use this structure in the final review:

```markdown
## Summary
<2–3 sentences: does the PR achieve the goal, and overall merge recommendation>

## Intended outcome
<restated goal> — **Met / Partially met / Unclear / Not met**

## Risk areas
- <shared/global/cross-feature items, or "Low — feature-local only">

## Findings

### 🔴 Critical (must fix before merge)
- <file:line or symbol> — <issue> — <why it matters>

### 🟡 Suggestions (should fix or discuss)
- ...

### 🟢 Nitpicks (optional)
- ...

### ❓ Questions for author
- ...

## Standards checklist (abbreviated)
- [ ] CLAUDE.md
- [ ] phoenix-fe-feature (if feature code)
- [ ] phoenix-scss (if styles)
- [ ] testing skill (if behavior/tests)
- [ ] No unjustified shared changes
- [ ] No duplicate component/utils

## Suggested verification
- <manual steps + yarn commands>
```

**Verdict labels:** `Approve` | `Approve with nits` | `Request changes` | `Needs discussion`

## Optional: line comments

When the user wants GitHub review comments, draft concise per-line notes; use `gh pr review` only if the user explicitly asks to submit on their behalf.

## Evolving this skill

When a review surfaces a recurring pitfall, add a short generalized rule here (not ticket-specific prose).
