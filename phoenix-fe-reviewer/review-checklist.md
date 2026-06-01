# Phoenix FE reviewer — expanded checklist

Use with [SKILL.md](SKILL.md). **`CLAUDE.md` in `phoenix-fe` is authoritative** — this file is a reviewer aid only.

## Read-only guard

- [ ] No files edited, committed, or pushed on the PR branch
- [ ] Feedback is review text (and optional `gh pr review`), not a patch
- [ ] If implementation was started by mistake, changes reverted before continuing

## Context

- [ ] Ticket / PR title matches the change
- [ ] PR description states what changed and how to verify
- [ ] Scope matches ticket (no unrelated refactors)
- [ ] PR description matches **actual** code (no stale claims about removed routes, components, etc.)
- [ ] FE-only PR does not silently require undeployed BE changes (or BE PR is linked)
- [ ] Copy sheet / `I18n` keys called out when new UI strings were added

## Prior review comments (when user or thread already reviewed)

- [ ] Every prior inline or summary comment listed
- [ ] Each marked: Addressed / Partial / Not addressed / Obsolete (with reason)
- [ ] Gaps tied to file/symbol, not only a high-level table

## Intended outcome (required)

- [ ] PR goal restated in one sentence
- [ ] **Intent matrix** filled: each claimed deliverable vs Met/Partial/No
- [ ] Primary user/operator flow traced in the diff (happy path)
- [ ] Loading / error / empty states present and wired
- [ ] Permissions and feature flags respected
- [ ] Edge cases from ticket/PR handled (not only happy path)
- [ ] No silent no-ops (e.g. button with no context, modal before data)
- [ ] No regression in adjacent flows in the same module
- [ ] Explicit **Intent sign-off** line in the review report

## Improvements and optimizations (suggest only — do not implement)

- [ ] UX improvements noted (modals, toasts, feedback loops)
- [ ] Data-layer improvements (cache vs refetch, invalidation)
- [ ] React structure (effects, memoization, service boundaries)
- [ ] Reuse opportunities (existing components/hooks)
- [ ] i18n / copy gaps listed with suggested keys
- [ ] PR description / test plan updates suggested if out of date

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

## GitHub post (when user asks to submit review)

- [ ] Draft summary body + all inline comments shown in chat **before** posting
- [ ] User **explicitly confirmed** ("post it", "looks good", etc.) — not assumed from "review this PR"
- [ ] Every actionable finding has its **own inline comment** on the relevant line (not only in the review body)
- [ ] Review **body** is 1–3 short, positive sentences (no "Issue 1–4", no skill quotes, no duplicated inline detail)
- [ ] Each inline comment: **heading only** (no issue numbers) + short **suggestion**; collaborative tone
- [ ] No long pasted text from `CLAUDE.md` or testing/feature skills in inline threads
- [ ] `event` matches verdict (`APPROVE` / `COMMENT` / `REQUEST_CHANGES`)

## Merge gate

- [ ] **Intent** Met or Partially met with agreed follow-ups — not Not met
- [ ] All 🔴 Critical findings resolved or accepted with explicit user/lead sign-off
- [ ] Shared-change risk acknowledged in PR or review thread
