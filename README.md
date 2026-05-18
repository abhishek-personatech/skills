# Phoenix Cursor Skills

Shared [Cursor Agent Skills](https://cursor.com/docs/agent/skills) for Phoenix backend (`phoenix`) and frontend (`phoenix-fe`) work: new features, bug fixes, refactors, dev testing, git/PR workflow, and review feedback.

Skills are markdown playbooks the agent reads when you `@mention` them in chat. They encode team conventions (branch naming, plan structure, implementation order, PR rules) so delivery stays consistent across engineers.

## Quick start

### 1. Clone into your Cursor skills directory

Personal skills live at `~/.cursor/skills/`. Clone this repo there (or symlink it):

```bash
git clone <this-repo-url> ~/.cursor/skills
```

If you already have other skills in that folder, clone to a subfolder or merge carefully so you do not overwrite unrelated skills.

### 2. Open the right repo in Cursor

| Work | Open in Cursor |
|------|----------------|
| Design / planning | Any workspace (no repo required) |
| Backend | `phoenix` |
| Frontend | `phoenix-fe` |
| Full-stack | **One repo per chat** (recommended), or a multi-root workspace with both |

### 3. Start a chat with a skill

Type `@` in Agent chat and pick a skill (e.g. `@phoenix-worker`), or paste a prompt from [prompt-guide/how-to-use.md](prompt-guide/how-to-use.md).

Always include your JIRA ticket, repo scope (**BE / FE / both**), and—when resuming—`@` the feature build plan file.

### 4. Prerequisites

- **Cursor** with Agent mode and skills enabled
- Access to **`phoenix`** and/or **`phoenix-fe`** repos
- **`gh`** CLI authenticated for PRs and review comments (used by git and review skills)
- For FE work: follow each repo’s **`CLAUDE.md`** and `phoenix-fe`’s `.cursor/skills/testing/SKILL.md` (referenced by FE skills)

## What this repo contains

```
.
├── README.md                          # This guide
├── prompt-guide/
│   └── how-to-use.md                  # Copy-paste prompts for common flows
├── docs/
│   └── feature-plans/                 # Canonical build plans (per feature)
│       └── <feature-slug>-build-plan.md
├── phoenix-worker/                    # End-to-end orchestrator (start here for full features)
├── phoenix-feature-planning/          # Design → build plan (no repo required)
├── phoenix-be-implementation-plan/    # BE enrichment in phoenix
├── phoenix-fe-implementation-plan/    # FE enrichment in phoenix-fe
├── phoenix-feature-implementation/    # Implement from an enriched plan
├── phoenix-fe-feature/                # FE structure, hooks, tests
├── phoenix-scss/                      # FE styling conventions
├── phoenix-dev-testing/               # Repeatable dev test rounds
├── phoenix-git-workflow/              # Branch, commit, draft PR, ready-for-review
└── resolve-review-comments/           # Address PR review feedback
```

**Build plans** are the single source of truth for a feature. They live here—not inside `phoenix` or `phoenix-fe`:

```text
~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md
```

When switching repos or starting a new session, **always `@`-attach the plan file** so the agent can read it outside the open workspace.

## Recommended workflows

### Full feature (design → ship)

Use **`@phoenix-worker`** for the full lifecycle. Say up front whether the work is **BE only**, **FE only**, or **both** so the agent does not touch the wrong repo.

| Phase | What | Skill | Workspace |
|-------|------|-------|-----------|
| 0 | Intake (ticket, scope, repo scope) | `phoenix-worker` | Any |
| 1 | Design | `phoenix-feature-planning` | No repo required |
| 2 | BE enrichment | `phoenix-be-implementation-plan` | `phoenix` |
| 3 | FE enrichment | `phoenix-fe-implementation-plan` | `phoenix-fe` |
| 4 | BE implementation | `phoenix-feature-implementation` | `phoenix` |
| 5 | FE implementation | `phoenix-feature-implementation` + `phoenix-fe-feature` / `phoenix-scss` | `phoenix-fe` |
| 6 | Dev testing (repeatable) | `phoenix-dev-testing` | Per repo in scope |
| 7 | Git / PR | `phoenix-git-workflow` | Per repo in scope |
| 8 | Review comments | `resolve-review-comments` | Repo with open PR |

Phases 6 and 7 can overlap (e.g. draft PR first, then test on dev/staging). The **full PR description** is written only when marking **ready for review**, after dev testing sign-off—not on every push.

See [prompt-guide/how-to-use.md](prompt-guide/how-to-use.md) for ready-to-copy prompts (new feature, BE-only, FE-only, resume, dev testing only, mark PR ready).

### Planning only

```
@phoenix-feature-planning

Feature: <one-line summary>
Ticket: PTI-XXXXX
```

Runs clarification → requirement summary → design build plan → review until approved. No codebase access required.

### Implementation only (plan already approved)

Open the target repo, attach the plan, and use `@phoenix-feature-implementation`:

- **Backend:** implement Section 3 in `phoenix`
- **Frontend:** implement Section 4 (+ 4.5) in `phoenix-fe`; also `@phoenix-fe-feature` (and `@phoenix-scss` when styling changes)

Backend first when APIs are new or changed; frontend after the contract is stable or merged.

### Bug fix or small change

You do not need the full worker lifecycle. Typical pattern:

1. Open the affected repo (`phoenix` or `phoenix-fe`)
2. `@phoenix-git-workflow` for branch/PR conventions
3. Repo-specific skills as needed (`phoenix-fe-feature`, `phoenix-scss` for FE)
4. `@phoenix-dev-testing` if the change needs a structured test pass
5. `@resolve-review-comments` when addressing PR feedback

### Dev testing mid-flight

```
@phoenix-dev-testing
@~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md

Round N. <context: pre-PR / draft PR open / fixes pushed>
```

### Mark PR ready for review

After dev testing passes:

```
@phoenix-git-workflow
@~/.cursor/skills/docs/feature-plans/<feature-slug>-build-plan.md

Dev testing passed. Write full PR description once, then mark ready for review.
```

## Skills reference

| Skill | When to use |
|-------|-------------|
| **phoenix-worker** | End-to-end feature delivery; default for “build this feature” |
| **phoenix-feature-planning** | Design doc / build plan from requirements |
| **phoenix-be-implementation-plan** | Enrich Section 3 with real paths in `phoenix` |
| **phoenix-fe-implementation-plan** | Enrich Section 4 with real paths in `phoenix-fe` |
| **phoenix-feature-implementation** | Write code from an enriched plan |
| **phoenix-fe-feature** | FE folder structure, hooks, context, integration tests |
| **phoenix-scss** | CSS/SCSS changes in `phoenix-fe` |
| **phoenix-dev-testing** | Structured dev test rounds and sign-off |
| **phoenix-git-workflow** | Branch from release, commits, draft PR, ready-for-review |
| **resolve-review-comments** | Triage and fix PR review comments one by one |

Each skill’s full rules live in its `SKILL.md` (e.g. `phoenix-worker/SKILL.md`).

## Conventions worth remembering

1. **Repo scope** — State BE / FE / both explicitly. The agent should not skip a repo without your consent.
2. **One repo per chat** — Default for implementation; use separate sessions for BE then FE unless you use multi-root.
3. **Plan file** — Create and update under `docs/feature-plans/`; `@`-attach on every handoff.
4. **Approval** — Planning skills treat `LGTM` / `approved` as gates between phases.
5. **PR body** — Draft PRs use a minimal stub; the full description is written once at ready-for-review.
6. **FE tests & styles** — Follow `phoenix-fe`’s `CLAUDE.md`, `.cursor/skills/testing/SKILL.md`, and `phoenix-scss`—not ad-hoc patterns.

## Example: new full-stack feature

```
@phoenix-worker

Feature: Bulk actions on console meeting workflow
Ticket: PTI-12345
Repos: both
Dev testing: post_draft_pr
Plan: create new

Start from Phase 0. Guide me through design, enrichment, implementation, dev testing, and PRs.
```

After design approval, the plan is saved as:

`~/.cursor/skills/docs/feature-plans/console-meeting-workflow-bulk-actions-build-plan.md`

(An example plan ships in this repo under `docs/feature-plans/`.)

## Contributing

- **New or updated skills** — Edit the relevant `SKILL.md`; keep frontmatter `name` and `description` accurate so `@` discovery works.
- **Prompt templates** — Add copy-paste examples to `prompt-guide/how-to-use.md`.
- **Feature plans** — Commit plans for features the team shares; use `<feature-slug>-build-plan.md` naming.

Pull requests to this repo should be reviewed like any shared tooling: changes affect how every engineer’s agent behaves.

## Further reading

- [prompt-guide/how-to-use.md](prompt-guide/how-to-use.md) — Prompt cookbook
- [phoenix-worker/SKILL.md](phoenix-worker/SKILL.md) — Phase checklist and orchestration rules
- [phoenix-feature-planning/plan-template.md](phoenix-feature-planning/plan-template.md) — Build plan structure
- Cursor docs: [Agent Skills](https://cursor.com/docs/agent/skills)
