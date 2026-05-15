---
name: phoenix-git-workflow
description: >-
  Phoenix git workflow for phoenix-fe (frontend) and Phoenix (backend): branch from
  release, JIRA-style naming, focused commits, tests before push, draft PRs with a
  minimal stub body, and a single review-ready PR description when marking ready
  for review (after phoenix-dev-testing). Ask clarifying questions when ticket,
  scope, or PR target is unclear. Use when the user says phoenix-git-workflow, git
  workflow, create a branch, push, draft PR, or mark PR ready for review.
---

# Phoenix git workflow

Applies to **phoenix-fe** (React/TypeScript) and **Phoenix** (Spring Boot/Gradle). Detect which repo you are in before running project-specific commands.

## Before coding — ask when unclear

Use **AskQuestion** (or ask conversationally) before starting when any of these are unknown:

| Topic       | Ask                                                                            |
| ----------- | ------------------------------------------------------------------------------ |
| JIRA ticket | Ticket id (e.g. `PTI-12345`) for branch name and commit prefix                 |
| Base branch | `release` vs `staging` vs another branch to branch from                        |
| Scope       | Exact files/behaviour in vs out of scope                                       |
| Tests       | Whether to add/update tests for the change                                     |
| PR target   | Which branch the PR should merge into                                          |

Do **not** block on ticket id for exploratory investigation; do block before creating a branch or commit if the user expects a JIRA-prefixed branch/commit and none was given.

## Branch setup

1. Confirm working tree is clean (or stash/commit unrelated work).
2. Fetch and update the agreed base branch (usually `release`):
   ```bash
   git fetch upstream
   git checkout release && git pull upstream release
   ```
3. Create a feature branch:
   ```bash
   git checkout -b PTI-XXXXX-short-description
   ```
   Use kebab-case after the ticket id; omit ticket prefix only if the user explicitly says no ticket.

## Implementation discipline

- Match existing patterns in the touched module; read surrounding code first.
- Keep diffs focused — no drive-by refactors.
- **phoenix-fe:** follow `CLAUDE.md` and relevant skills (`phoenix-fe-feature`, `testing`, `phoenix-scss`).
- **Phoenix (BE):** follow `CLAUDE.md` and existing Spring Boot patterns in the touched module; read surrounding service, repository, and test code first.

## Before commit

Fix failures before committing.

### phoenix-fe (frontend)

1. Run lints/types on edited files when practical (`yarn lint`, `yarn check-types`).
2. Run affected tests (`yarn test --testPathPattern=... --watchAll=false`).

### Phoenix (backend, Spring Boot)

1. Run a Gradle build and tests for the affected scope:
   ```bash
   ./gradlew build
   ```
   For a narrower check, run tests for the relevant module or class:
   ```bash
   ./gradlew test --tests 'com.example.SomeServiceTest'
   ```

## Commit message

**Format (both repos):** `featureName:commitType message`

- `featureName` — alphanumeric or hyphen-separated (e.g. `email-snippet`, `PTI-23100`)
- `commitType` — one of `task`, `test`, `bug`, `refactor`
- `message` — short imperative description

**Examples:**

```
email-snippet:task Disable snippet properties until production ready
PTI-23100:task Show force-sync button when only action applies
```

If the user provides a JIRA id, prefer it as `featureName` (e.g. `PTI-23100:task ...`).

**phoenix-fe only:** commit messages are validated by `scripts/validate-commit-message.js`; merge commits skip validation.

**Phoenix (BE):** no commit-message validator script — follow the same format by convention.

## Push and PR

Push to `origin` (your fork); target **`upstream/release`** as the PR base (default for both repos).

```bash
git push -u origin PTI-XXXXX-short-description
```

### Two PR description modes (do not mix)

| Step | PR body | Agent rule |
|------|---------|------------|
| **Create draft PR** | Minimal **stub** only | Use [pr-draft-stub.md](pr-draft-stub.md) — fill ticket + plan path; **do not** LLM-summarize the diff |
| **Dev-testing pushes** | Unchanged | `git push` only — **never** `gh pr edit` body between rounds |
| **Mark ready for review** | Full **review-ready** body **once** | After `phoenix-dev-testing` sign-off (`passed` \| `waived`); generate from **final** diff + plan Section 9 |

Draft PRs may be opened **before**, **during**, or **after** dev testing — see [phoenix-dev-testing](../phoenix-dev-testing/SKILL.md). Only **ready-for-review** gets the full description.

### Create draft PR (stub body)

Copy [pr-draft-stub.md](pr-draft-stub.md), replace `PTI-XXXXX` and `<feature-slug>`, save as a temp file, then:

**phoenix-fe:**

```bash
gh pr create --draft \
  --repo personatech-infra/phoenix-fe \
  --base release \
  --head <your-github-user>:PTI-XXXXX-short-description \
  --title "PTI-XXXXX: <short title>" \
  --body-file /path/to/pr-draft-stub-filled.md
```

**Phoenix (backend):**

```bash
gh pr create --draft \
  --repo personatech-infra/phoenix \
  --base release \
  --head <your-github-user>:PTI-XXXXX-short-description \
  --title "PTI-XXXXX: <short title>" \
  --body-file /path/to/pr-draft-stub-filled.md
```

Update plan metadata: `pr_status: draft`, record PR URL(s) in Section 9 or plan metadata.

### Push during dev testing (no body edit)

```bash
git push origin PTI-XXXXX-short-description
```

Do **not** run `gh pr edit` unless marking ready for review.

### Mark ready for review (full body — once)

**Requires:** plan `dev_testing_status` is `passed` or `waived` (see phoenix-dev-testing).

1. Generate the review-ready body from the **final** diff and plan Section 9 (dev test matrix + rounds completed).
2. Write to a temp file; update the PR **once**:

```bash
gh pr edit <number> --body-file /path/to/pr-review-ready.md
gh pr ready <number>
```

For cross-repo features, each repo PR gets its own body; cross-link the sibling PR in **Related PRs**.

**Review-ready body** (generate at this step only):

- **What changed** — concise summary of the final diff.
- **Why** — ticket context or problem being solved.
- **How to verify** — manual steps from Section 9 + commands (e.g. `yarn test ...`, `./gradlew test ...`).
- **Related PRs** — link sibling repo PR if applicable.
- **Plan** — path to build plan file.

Update plan metadata: `pr_status: ready_for_review`.
