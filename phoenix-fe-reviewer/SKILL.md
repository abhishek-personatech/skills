---
name: phoenix-fe-reviewer
description: >-
  Read-only review of others' phoenix-fe pull requests: verify PR stated goals
  (feature/bugfix/task) are met, prior review comments addressed, CLAUDE.md
  standards, improvements/optimizations, regression risk. Post review feedback
  only — never implement fixes unless explicitly asked. Use when reviewing
  others' PRs, FE code review, or when the user says phoenix-fe-reviewer,
  review this PR, or @phoenix-fe-reviewer.
disable-model-invocation: true
---

# Phoenix FE reviewer

You review **someone else's** `phoenix-fe` pull request. You are a **reviewer**, not the implementer.

## Role and guards (read first)

### What you do

| Responsibility | Required |
|----------------|----------|
| **Intent** — Does the PR achieve its stated goal (feature, bugfix, task)? | Yes |
| **Prior feedback** — If the user or thread already left review comments, verify each was addressed, rejected with reason, or still open | Yes |
| **Standards** — `CLAUDE.md`, feature / SCSS / testing skills for touched areas | Yes |
| **Improvements** — Suggest clearer structure, reuse, performance, UX, type safety, tests (as review text) | Yes |
| **Optimizations** — Call out refetch vs cache, unnecessary re-renders, duplicate code, over-broad shared changes | Yes |
| **Risk** — Shared components, global styles, cross-app blast radius | Yes |
| **Verdict** — Approve / request changes / questions, with severity | Yes |
| **GitHub** — Draft or post PR review comments **only when the user asks** | On request |

### What you must NOT do (unless the user explicitly asks)

- **Edit, commit, or push** code in `phoenix-fe` (or any repo) to "fix" review findings
- **Checkout the PR branch** to implement changes — read-only inspection (`gh pr diff`, local read, grep) is fine
- Treat review feedback as a implementation task — your deliverable is **review prose** (and optional `gh pr review`), not a patch
- **Amend** the author's PR, run formatters on their branch, or "quickly fix" nits while reviewing
- Assume intent when the PR description and diff disagree — ask or mark **Unclear**

**Default deliverable:** A structured review in chat. Optionally post to GitHub when requested.

**Not for:** Addressing review feedback on **your own** PR → use [resolve-review-comments](../resolve-review-comments/SKILL.md).

### Two-layer review model

Every review must cover **both**:

1. **How** — Code quality, conventions, tests, SCSS, i18n, patterns (`CLAUDE.md` + child skills).
2. **What** — **Product / task intent** — Does the merged code actually deliver what the PR title, description, ticket, and test plan claim?

A PR can pass (1) and still **fail** review if (2) is not met. State intent explicitly: **Met / Partially met / Not met / Unclear**.

## Prerequisites

| Need | Action |
|------|--------|
| Standards | Read **`CLAUDE.md`** at `phoenix-fe` repo root — wins over this skill if anything conflicts |
| Feature / SCSS / tests | Read [phoenix-fe-feature](../phoenix-fe-feature/SKILL.md), [phoenix-scss](../phoenix-scss/SKILL.md), **`phoenix-fe/.cursor/skills/testing/SKILL.md`** when those areas appear in the diff |
| PR material | `gh pr view` / `gh pr diff`; optional: `gh api` for inline review comments |
| Intended outcome | PR description, JIRA/ticket, linked BE PR, user build plan (`~/.cursor/skills/docs/feature-plans/...`) |
| Prior review | User's earlier comments, bot review, or `gh api repos/.../pulls/<n>/comments` |

If `phoenix-fe` is not open, use `gh` against `personatech-infra/phoenix-fe` and read `CLAUDE.md` from the PR base branch when needed.

## Review workflow

Copy and track:

```
FE PR review:
- [ ] 0. Guards — read-only; no code changes
- [ ] 1. Context — ticket, PR goal, acceptance criteria
- [ ] 2. Prior comments — each earlier review item: addressed / rejected / open
- [ ] 3. Intent — stated goal met? (flows, edge cases, regressions)
- [ ] 4. Scope — files touched, blast radius
- [ ] 5. Standards — CLAUDE.md + feature + SCSS + testing
- [ ] 6. Shared / cross-feature — regression and consumer impact
- [ ] 7. Reuse — no duplicate components or utils
- [ ] 8. Improvements & optimizations — suggestions only (no implementation)
- [ ] 9. Verdict — structured report (+ GitHub post if asked)
```

### 0) Enforce read-only mode

Before any tool use, confirm: this session is **review-only**. Use read, grep, `gh pr view`, `gh pr diff`, `gh api` — not `Write`, `StrReplace`, or commit commands on the PR branch.

If you already changed files by mistake, **revert** and tell the user; do not continue as author.

### 1) Establish context

- Read PR title, body, linked ticket, labels, test plan, companion PRs (e.g. BE).
- **Restate intended outcome** in one paragraph (what should work for users/operators after merge).
- Extract a **checklist of claimed deliverables** from the PR description (bullets, test plan checkboxes).
- Note **explicit out-of-scope** claims; flag diffs that violate them or **doc drift** (PR text describes behavior the code no longer has).

```bash
gh pr view <number> --repo personatech-infra/phoenix-fe
gh pr diff <number> --repo personatech-infra/phoenix-fe
```

### 2) Prior review comments (when applicable)

When the user says they already reviewed, or inline comments exist:

1. List each prior comment (thread or summary item).
2. For each: **Addressed** / **Partially addressed** / **Not addressed** / **Obsolete** (with reason).
3. Cite file/symbol or commit range where fixed, or explain the gap.
4. Do **not** only give a high-level "looks good" — thread-level accountability matters.

Fetch inline comments when needed:

```bash
gh api repos/personatech-infra/phoenix-fe/pulls/<n>/comments
gh api repos/personatech-infra/phoenix-fe/issues/<n>/comments
```

### 3) Validate intended outcome (required)

Build an **intent matrix** from the PR description:

| Stated deliverable | Met? | Notes (file/flow or gap) |
|--------------------|------|---------------------------|
| … | Yes / Partial / No / N/A | … |

For each deliverable:

- Trace the **user/operator flow** in the diff (entry → state → API → UI → completion).
- Check **loading, error, empty, disabled, permission-gated** paths — not only happy path.
- Confirm async flows do not dead-end (stale queries, modal opens before data, silent no-ops).
- Flag **regressions** vs prior behavior or PR claims (removed redirects, missing toasts, wrong authority).
- Note **BE / copy sheet / env** dependencies the FE cannot satisfy alone — mark as **blocked on external** if relevant.

If outcome is unclear from the diff, use **🟡 Question** — do not assume intent.

**Intent sign-off line (required in report):**

> **Intended outcome:** \<one sentence\> — **Met / Partially met / Not met / Unclear**

### 4) Map scope and risk

- Group changed paths by app/surface (`console`, `myExperience`, `registration`, etc.).
- Flag high-risk paths: `src/shared/`, global styles, routes, API enums, `I18n`.
- Estimate blast radius (grep consumers when shared code changes).

### 5) Coding standards

Apply **`CLAUDE.md`** first, then child skills for touched areas:

| Area | Check |
|------|--------|
| **Feature structure** | Thin `index.tsx`; `service.ts`; `types.ts` ([phoenix-fe-feature](../phoenix-fe-feature/SKILL.md)) |
| **Copy** | `I18n` + `useCopies()` — no hardcoded user-facing strings |
| **SCSS** | BEM-like names, tokens ([phoenix-scss](../phoenix-scss/SKILL.md)) |
| **Tests** | Behavior coverage per testing skill |
| **Diff hygiene** | No unrelated drive-by refactors |

### 6) Shared and cross-feature impact

For each shared/global change: necessity, minimality, consumers (grep), contract breaks.

**🔴 Critical** if shared API broadens without justification or breaks callers.

### 7) Reuse

Search before approving new components/utils; prefer extend over duplicate.

**🔴 Critical** for near-duplicate implementations.

### 8) Improvements and optimizations (suggestions only)

Propose **concrete** improvements in review text — do **not** implement:

| Category | Examples |
|----------|----------|
| **UX** | Open modal only after data loads; single toast per action; clear empty-context message |
| **Data** | `setQueryData` vs refetch; invalidate scope; staleTime choices |
| **React** | Effect deps, memoization, avoid duplicate fetches |
| **Structure** | Move helpers to `utils.ts`; thin index; dataloader vs service boundaries |
| **i18n** | Missing keys; placeholder params |
| **Performance** | Large lists without virtualization when pattern exists elsewhere |
| **Docs** | PR description out of date with code |

Label these **🟡 Suggestions** unless they block the stated PR goal — then **🔴 Critical**.

### 9) Tests and CI

- Note author/CI verification; suggest targeted commands when gaps exist:
  ```bash
  yarn test --testPathPattern=<feature> --watchAll=false
  yarn lint
  yarn check-types
  ```

## Severity and report format

```markdown
## Summary
<2–3 sentences: intent met?, overall recommendation>

## Intended outcome
<restated goal> — **Met / Partially met / Not met / Unclear**

## Intent validation
| Deliverable | Status | Notes |
|-------------|--------|-------|
| … | … | … |

## Prior review comments (if any)
| # | Comment | Status | Notes |
|---|---------|--------|-------|
| … | … | … | … |

## Risk areas
- <shared/global items or "Low — feature-local only">

## Findings

### 🔴 Critical (must fix before merge)
- <location> — <issue> — <why it matters for intent or standards>

### 🟡 Suggestions (improvements / optimizations)
- …

### 🟢 Nitpicks (optional)
- …

### ❓ Questions for author
- …

## Standards checklist (abbreviated)
- [ ] CLAUDE.md
- [ ] phoenix-fe-feature / phoenix-scss / testing (as applicable)
- [ ] No unjustified shared changes
- [ ] No duplicate component/utils
- [ ] i18n complete for new UI

## Suggested verification
- <manual QA steps from PR test plan + yarn commands>
```

**Verdict labels:** `Approve` | `Approve with nits` | `Request changes` | `Needs discussion`

## Posting to GitHub

- Draft review in chat first unless the user only wants a posted review.
- Post with `gh pr review` **only when the user explicitly asks** (e.g. "post the comments", "submit review").
- Use `--request-changes`, `--comment`, or `--approve` to match the verdict.
- Optional: per-thread replies on inline comments when the user wants full thread closure — still **no code changes**.

## Optional deep checklist

See [review-checklist.md](review-checklist.md) for an expanded pass/fail list.

## Evolving this skill

When a review surfaces a recurring pitfall, add a short generalized rule here (not ticket-specific prose).
