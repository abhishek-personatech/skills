---
name: phoenix-git-workflow
description: >-
  Phoenix (phoenix + phoenix-fe) git workflow: PTI branch naming (ASF-PTI-…),
  coding standards, tests, commit format, push, and reviewer-friendly PRs.
  Use when creating a branch, committing, pushing, opening a PR, or when the
  user says phoenix-git-workflow, git workflow, branch and PR, or
  @phoenix-git-workflow.
disable-model-invocation: true
---

# Phoenix git workflow

Use for **any change** in `phoenix` (backend) or `phoenix-fe` (frontend): bug fixes, small improvements, refactors, or test-only updates.

## Before you start — always ask

1. **JIRA ID** — every time. Format is always `PTI-#####` (e.g. `PTI-23197`). Do not reuse from a prior chat.
2. **Task slug** — ask the user to type a short kebab-case name each time (e.g. `fix-null-check`, `update-pagination`). Do not fetch from JIRA.
3. **Repo** — confirm `phoenix` or `phoenix-fe` if unclear.
4. **Branch base** — before creating the branch, confirm where to branch from. **Default:** current local branch; always get explicit confirmation (user may want `upstream/release` or another base).
5. **PR base** — before opening the PR, confirm target branch. **Default:** `upstream/release`; always get explicit confirmation.

---

## Branch naming

```text
ASF-<JIRA_ID>-<task-slug>
```

- **Developer initial:** `ASF` (fixed)
- **JIRA ID:** `PTI-#####` from the user
- **Task slug:** kebab-case from the user

**Example:** `ASF-PTI-23197-fix-null-check`

### Create the branch

After the user confirms the base:

```bash
# If branching from current branch (most common):
git checkout -b ASF-PTI-23197-fix-null-check

# If user confirms a remote base (e.g. upstream/release):
git fetch upstream release
git checkout -b ASF-PTI-23197-fix-null-check upstream/release
```

Adjust remotes and ref names to match the repo.

---

## Code change standards

| Repo | Standards |
|------|-----------|
| `phoenix-fe` | `CLAUDE.md` |
| `phoenix` | `CLAUDE.md` |

- Read files you will edit; **match existing patterns** (structure, naming, imports, error handling).
- **Do not reformat** untouched code.
- Keep the diff minimal.

### Tests (when applicable)

| Repo | Test guidance |
|------|---------------|
| `phoenix-fe` | `.cursor/skills/testing/SKILL.md` |
| `phoenix` | `CLAUDE.md` (test conventions) |

- Update or add tests when behavior changes.
- Use commit `taskType` `test` when the change is **only** test files.
- Run relevant tests before commit.

---

## Commit message format

```text
<jira-id>:<taskType> <description of change>
```

**taskType** (pick one):

| Type | Use when |
|------|----------|
| `bug` | Bug fixes |
| `task` | Feature work, improvements, non-bug enhancements |
| `refactor` | Code improvement **without** behavior change |
| `test` | Test-only changes |

**Examples:**

```text
PTI-23197:bug Fix null check on meeting list pagination
PTI-23197:task Add bulk action confirmation dialog
PTI-23197:refactor Extract shared date formatter utility
PTI-23197:test Add coverage for empty selection edge case
```

---

## Push

```bash
git push -u origin ASF-PTI-23197-fix-null-check
```

---

## Pull request

### 1. Confirm base branch

Ask the user before creating the PR:

> Which branch should this PR target? Default is **upstream/release** — confirm or specify another.

### 2. PR title

Use the JIRA ID and a short, clear title:

```text
PTI-23197: Fix null check on meeting list pagination
```

### 3. PR body template (reviewer-friendly)

Fill every section so reviewers rarely need to ask follow-ups:

```markdown
## Summary
[1–3 sentences: what this change does and why.]

## JIRA
[PTI-#####](<jira-url-if-known>)

## What changed
- [Bullet list of concrete code/test changes]

## Impact
- **User-facing:** [None / describe UI or API behavior change]
- **Risk:** [Low / Medium — why]
- **Areas touched:** [e.g. meeting list, pagination hook]

## How to verify
1. [Step a reviewer or QA can follow]
2. [Expected result]

## Tests
- [ ] Unit/integration tests added or updated
- [ ] [Command run, e.g. `npm test -- <path>` or backend equivalent]
- [ ] Manual verification: [brief note if applicable]
```

Create the PR only after base is confirmed:

```bash
gh pr create --base release --head ASF-PTI-23197-fix-null-check \
  --title "PTI-23197: Fix null check on meeting list pagination" \
  --body-file /tmp/pr-body.md
```

Adjust `--base`, remotes, and repo flags to match the project.

---

## End-to-end checklist

```
- [ ] JIRA ID collected (PTI-#####)
- [ ] Task slug collected from user
- [ ] Branch base confirmed (default: current local branch)
- [ ] Branch created: ASF-<JIRA_ID>-<task-slug>
- [ ] Code follows CLAUDE.md and local patterns
- [ ] Tests updated if applicable
- [ ] Commit: PTI-#####:<taskType> <description>
- [ ] Pushed to origin
- [ ] PR base confirmed (default: upstream/release)
- [ ] PR opened with reviewer-friendly body
```
