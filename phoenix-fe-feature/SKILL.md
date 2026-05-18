---
name: phoenix-fe-feature
description: >-
  Structure and evolve Phoenix FE features: thin index.tsx, service-first hooks,
  optional context, I18n/useCopies, stable render flow, integration tests.
  Follow phoenix-fe CLAUDE.md for coding guidelines and the repo testing skill for tests.
  Use when building or refactoring Phoenix FE features, scaffolding feature
  folders (service.ts, types.ts, components/), implementing user-facing flows,
  or when the user says phoenix-fe-feature, FE feature development, or @phoenix-fe-feature.
---

# Building a Phoenix FE feature

## Repository guidelines (required)

When working in `phoenix-fe`, read and follow **`CLAUDE.md`** at the repository root for coding guidelines, architecture patterns, and conventions. This skill adds feature-structure guardrails on top of `CLAUDE.md`; if anything conflicts, **`CLAUDE.md` wins**.

For test cases, read and follow **`.cursor/skills/testing/SKILL.md`** in the `phoenix-fe` repository. Do not invent alternate testing conventions—use that skill for structure, render helpers, mocking, and coverage expectations.

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

## Shared / common components (`src/shared/`, cross-feature console components)

During **feature work**, prefer solving the requirement inside the feature module (`apps/console/modules/...`) using existing shared APIs.

**Do not modify shared or widely reused components** (e.g. `src/shared/components/*`, shared hooks, `MenuDropdown`, `ContextMenu`) unless **all** of the following are true:

1. The user has **explicitly approved** the shared change (or asked for it directly).
2. There is a **solid, stated reason** the feature cannot be implemented without it (e.g. no supported prop/variant, bug in the shared component, contract required by multiple features).
3. The change is **minimal** (smallest diff that unblocks the feature) and does not expand shared surface area without need.

When a gap appears (unsupported variant, missing `testId`, styling hook):

- First try **feature-local** workarounds: wrapper element, existing variants (`Variant.PRIMARY`, `Variant.ERROR`), composition, feature SCSS under `workflow.scss` / feature `*.scss`.
- If a shared change is still necessary, **stop and ask the user** with a short proposal (what to change, why feature-local is insufficient, impact on other consumers).

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

Follow **`.cursor/skills/testing/SKILL.md`** in the `phoenix-fe` repo for all test authoring.

For new user-facing feature flows, ensure coverage includes at minimum:

- loading state
- success render (key sections/data shown)
- error state
- primary interaction paths
- critical edge case paths (empty optional data, gated sections, retry/unhide paths)

## Code quality checklist before PR

- Changes comply with **`CLAUDE.md`** at the repository root.
- Tests comply with **`.cursor/skills/testing/SKILL.md`**.
- No dead code / unused imports / stale TODOs in shipped paths.
- All new public props and response fields typed.
- Lints clean on edited files.
- At least one relevant test executed locally.

## Component SCSS

Follow `.cursor/skills/phoenix-scss/SKILL.md` for class naming, CSS custom properties, override patterns, and DOM/token root requirements. Keep feature SCSS co-located with the component.

## Evolving this skill

When a new feature reveals a recurring pattern (or a pitfall), update this skill with the generalized lesson, not ticket-specific details.
