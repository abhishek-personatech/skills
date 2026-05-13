---
name: phoenix-fe-feature
description: >-
  Structure and evolve Phoenix FE features: thin index.tsx, service-first hooks,
  optional context, I18n/useCopies, stable render flow, integration tests.
  Use when building or refactoring Phoenix FE features, scaffolding feature
  folders (service.ts, types.ts, components/), implementing user-facing flows,
  or when the user says phoenix-fe-feature, FE feature development, or @phoenix-fe-feature.
---

# Building a Phoenix FE feature

## Core principle

Choose the simplest structure that solves current requirements, then evolve only when there is clear pressure (reusability, prop drilling pain, state coordination complexity).

## Preferred structure

Use this baseline unless the feature is tiny:

- `index.tsx`: composition and wiring only
- `service.ts`: state, side-effects, API calls, derived values, handlers
- `types.ts`: props, service contracts, and feature-local types
- `components/`: presentational pieces with minimal logic
- `context/` (optional): only when multiple deep descendants share mutable state

## Design and implementation guardrails

- Keep one source of truth for feature state in the service hook.
- Keep `index.tsx` thin; avoid business logic there.
- Use context only to avoid meaningful prop drilling; skip it for shallow trees.
- Prefer stable callbacks and memoized derived values when passed deep.
- Use explicit render guards for whole-tree early exits; keep JSX flat and readable.
- Keep prop names consistent with upstream metadata names to reduce mapping errors.
- Avoid premature abstractions; only introduce new layers after repeated usage.

## UX and copy standards

- No hardcoded user-facing strings in feature UI.
- Add copy keys in `I18n` and fetch text via `useCopies()`.
- Keep error/loading/success states explicit and testable.
- For PR review comments, if a suggested fix changes UX or runtime behavior (not just style/refactor), get explicit user approval before applying it.

## Async flow quality checks

- Ensure progression logic cannot dead-end when optional API fields are empty.
- Avoid effects that depend on unstable objects (`props` object identity, inline arrays/objects).
- Prefer narrow effect dependencies (`handler`, scalar lengths, IDs) to prevent repeated side-effects.
- Validate hidden/revealed UI uses current state, not initial props snapshots.

## Testing expectations

For new user-facing feature flows, add integration-style tests close to user behavior:

- loading state
- success render (key sections/data shown)
- error state
- primary interaction paths
- critical edge case paths (empty optional data, gated sections, retry/unhide paths)

Follow workspace testing conventions:

- tests under `__test__/`
- behavior-focused test names
- minimal mocking at boundaries (prefer HTTP boundary over internal hook internals)

## Code quality checklist before PR

- Feature matches `CLAUDE.md` patterns (service-first, copy usage, type placement).
- No dead code / unused imports / stale TODOs in shipped paths.
- All new public props and response fields typed.
- Lints clean on edited files.
- At least one relevant test executed locally.

## Component SCSS

Follow `.cursor/skills/phoenix-scss/SKILL.md` for class naming, CSS custom properties, override patterns, and DOM/token root requirements. Keep feature SCSS co-located with the component.

## Evolving this skill

When a new feature reveals a recurring pattern (or a pitfall), update this skill with the generalized lesson, not ticket-specific details.
