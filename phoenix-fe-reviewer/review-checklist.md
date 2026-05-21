# Phoenix FE reviewer — expanded checklist

Use with [SKILL.md](SKILL.md). **`CLAUDE.md` in `phoenix-fe` is authoritative** — this file is a reviewer aid only.

## Context

- [ ] Ticket / PR title matches the change
- [ ] PR description states what changed and how to verify
- [ ] Scope matches ticket (no unrelated refactors)
- [ ] FE-only PR does not silently require undeployed BE changes (or BE PR is linked)

## Intended outcome

- [ ] Primary user flow works end-to-end in code review (happy path)
- [ ] Loading / error / empty states present and wired
- [ ] Permissions and feature flags respected
- [ ] Edge cases called out in ticket are handled (not only happy path)
- [ ] No regression in adjacent flows in the same module

## Feature structure ([phoenix-fe-feature](../phoenix-fe-feature/SKILL.md))

- [ ] `index.tsx` stays thin (composition only)
- [ ] State and side-effects live in `service.ts` (or established module pattern)
- [ ] Types in `types.ts` (or colocated types file for the feature)
- [ ] Context introduced only for real prop-drilling pain
- [ ] Render guards for whole-tree early exits; JSX readable
- [ ] Effect dependencies stable (IDs/scalars, not whole `props` objects)

## Copy and i18n

- [ ] No hardcoded user-visible strings in UI
- [ ] New keys added to `I18n` and consumed via `useCopies()`

## SCSS ([phoenix-scss](../phoenix-scss/SKILL.md))

- [ ] Class names match TSX and compiled selectors
- [ ] Design tokens preferred over magic values
- [ ] Custom properties defined on a DOM ancestor that exists on all render paths
- [ ] Modifier pattern reuses same token names
- [ ] No orphaned selectors after renames

## Shared / global changes

- [ ] Changes under `src/shared/` are necessary and minimal
- [ ] Author explains why feature-local workaround was insufficient
- [ ] Existing consumers considered (grep usages)
- [ ] No breaking prop rename without migration across callers
- [ ] Global style changes do not unintentionally restyle other apps/surfaces

## Reuse

- [ ] No new component when an existing one supports the case (variant/wrapper)
- [ ] No new util when an existing helper or lodash pattern fits
- [ ] New abstractions justified by repeated use, not one-off convenience

## Type safety and code quality

- [ ] No unnecessary `any` / unsafe casts
- [ ] API response shapes typed; optional fields guarded
- [ ] No dead code, debug logs, or commented-out blocks
- [ ] Names and structure readable without long comment blocks
- [ ] Imports and exports consistent with surrounding files

## Tests (`phoenix-fe/.cursor/skills/testing/SKILL.md`)

- [ ] User-facing flows have integration/unit coverage per team testing skill
- [ ] Tests assert behavior, not implementation trivia
- [ ] Critical edge paths covered (empty data, retry, gated UI)
- [ ] CI / author notes which tests were run

## Merge gate

- [ ] All 🔴 Critical findings resolved or accepted with explicit user/lead sign-off
- [ ] Shared-change risk acknowledged in PR or review thread
