---
name: phoenix-git-workflow
description: >-
  Phoenix git workflow for phoenix-fe (frontend) and Phoenix (backend): branch from
  release, JIRA-style naming, focused commits, tests before push, and draft PRs
  with review-ready descriptions. Ask clarifying questions when ticket, scope, or
  PR target is unclear. Use when the user says phoenix-git-workflow, git workflow,
  create a branch, or prepare a PR for phoenix-fe or Phoenix.
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

```bash
git push -u origin PTI-XXXXX-short-description
```

Open a **draft PR** against **`upstream/release`** (default for both repos).

**phoenix-fe:**

```bash
gh pr create --draft \
  --repo personatech-infra/phoenix-fe \
  --base release \
  --head <your-github-user>:PTI-XXXXX-short-description
```

**Phoenix (backend):**

```bash
gh pr create --draft \
  --repo personatech-infra/phoenix \
  --base release \
  --head <your-github-user>:PTI-XXXXX-short-description
```

Push to `origin` (your fork); target the upstream repo `release` branch as the base.

**PR description** (review-ready):

- **What changed** — concise summary of the diff.
- **Why** — ticket context or problem being solved.
- **How to verify** — manual steps and/or commands (e.g. `yarn test ...`, `./gradlew test ...`).
