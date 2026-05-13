---
name: resolve-review-comments
description: Resolve pull request review comments in a safe, repeatable loop (triage, fix, validate, commit, push). Use when the user mentions PR review comments, review feedback, addressing comments, making PR merge-ready, or asks to fix comments one by one.
disable-model-invocation: true
---

# Resolve review comments

## Quick start (one-by-one with confirmations)

1. Get the list of review comments using `gh` (default path).
   - If `gh` is unavailable/unauthed, ask the user to fix auth or paste comments.
2. Pick **one** comment to resolve next; restate it in one sentence.
3. Before editing, state the **approach** and ask for confirmation.
4. Implement the smallest correct fix.
5. Validate (lint/typecheck/tests as appropriate).
6. Stage only the relevant files, commit with a concise message, push.
7. Move to the next comment.

## Tool policy

- Use `gh` for all PR review comments/threads/checks.
- Use `git` for local branch state, diffs, staging, commit, and push.

## Key constraint: Git vs review comments

- Review comments live in the hosting platform (GitHub/GitLab) — **not in Git history**.
- Git can fetch PR refs/branches, but **cannot list review threads**.
- Therefore:
  - If `gh` cannot be used, request the user to paste review comments.
  - Continue using Git for fetch/diff/commit/push tasks.

## Workflow

### 1) Establish the working branch and base

- Confirm current branch and cleanliness:
  - `git branch --show-current`
  - `git status`
- Ensure the base branch is up to date:
  - `git fetch origin <base>`
  - `git fetch upstream <base>` (if upstream exists and PR targets it)
- See PR range diff:
  - `git diff <base>...HEAD --stat`

### 2) Collect review comments (choose one)

#### Preferred (GitHub CLI)

- Verify auth once:
  - `gh auth status`

- Fetch review comments (line-level):
  - `gh api repos/<org>/<repo>/pulls/<PR#>/comments`
- Fetch PR reviews (approvals / request changes / review bodies):
  - `gh api repos/<org>/<repo>/pulls/<PR#>/reviews`
- Fetch issue comments on the PR:
  - `gh api repos/<org>/<repo>/issues/<PR#>/comments`
- Quick consolidated view:
  - `gh pr view <PR#> --repo <org>/<repo> --comments`

#### If `gh` is not allowed/available

- Ask the user to paste each review comment (verbatim) including:
  - file path, line/section, expected change, and severity (if provided).

### 3) Triage each comment quickly

Classify into one of:
- **Correctness/Crash**: null/empty guards, runtime errors, wrong branching.
- **Data/Types**: wrong types, optional fields, API mismatch.
- **UX**: loading/error states, empty states, disabled states.
- **Performance**: memoization, unnecessary renders, large lists.
- **Maintainability**: naming, duplication, structure, dead code.
- **Tests**: missing scenarios, fragile selectors, coverage dips.

### 4) Confirm approach before coding

For each comment:
- Restate the issue and proposed fix in 1–2 sentences.
- Call out behavior changes (if any).
- Ask the user to confirm proceeding.

### 5) Make the fix with local safety checks

Defaults:
- Read the file(s) first.
- Keep changes minimal and consistent with existing patterns.
- Avoid speculative refactors.

Validation guidance:
- If you touched TS/exports/types: run `yarn tsc --noEmit` (or rely on pre-push hook if present).
- If a component behavior changed: add/adjust tests where practical.
- Use targeted test runs when possible (single test file / single module).

### 6) Commit and push (only relevant files)

- Verify diff is limited:
  - `git diff --name-only`
- Stage only files for the comment:
  - `git add <paths...>`
- Commit (concise, why-focused):
  - `git commit -m "<scope>:<type> <short reason>"`
- Push:
  - `git push origin HEAD`

### 7) Close the loop

After each comment:
- Summarize what changed and what was validated.
- Confirm the next comment to tackle.

## Example prompts to use during the loop

- “Next comment: <quoted comment>. Proposed fix: <1 sentence>. OK to implement?”
- “I’m going to stage only <file list> and commit as: <message>. OK?”
- “This changes behavior for <edge case>. Confirm that’s desired?”

## Recovery snippets

- Re-authenticate `gh`:
  - `gh auth logout -h github.com`
  - `gh auth login -h github.com`
- Set default repo (optional but helpful):
  - `gh repo set-default <org>/<repo>`

