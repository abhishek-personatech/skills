---
name: phoenix-scss
description: Apply Phoenix frontend SCSS conventions for mixin usage, selector patterns, typography, and safe review adoption. Use when editing .scss/.css files in phoenix-fe or when the user asks for CSS/SCSS refactors, styling fixes, or review-comment styling updates.
disable-model-invocation: true
---

# Phoenix SCSS Conventions

Use this skill for SCSS/CSS work in `phoenix-fe`.

## Core rules

- Prefer existing project mixins over raw CSS when equivalent mixins exist.
- Keep selectors consistent with existing nesting and `&` patterns.
- Do not introduce behavior/UX changes from review comments without explicit user approval.
- Keep changes minimal and localized; avoid broad visual churn unless requested.

## Mixin-first guidelines

- Layout/centering:
  - Prefer `@include flex_CTR`, `@include flex_V_CTR`, `@include flex_V_CTR_H_STR`, `@include inlineFlex_CTR`.
  - Remove redundant manual properties already covered by a mixin.
- Positioning:
  - Prefer `@include position(...)` over raw `position`, `top/right/bottom/left` blocks when possible.
- Sizing:
  - Prefer utility mixins like `@include square(...)`, `@include circle(...)` where applicable.
- Typography:
  - Use only supported font weight tokens for `@include fontWeight(...)`: `"b"`, `"bb"`, `"r"`, `"l"`, `"t"`.
  - Do not use unsupported tokens (for example, avoid `sb`).

## Selector style

- In nested blocks, prefer explicit direct-child selectors with `& > ...` for readability and consistency.
- Keep nesting shallow; avoid unnecessary selector specificity.

### Class naming (block segments + child)

Use **underscores** between the base block and camelCase sub-segments, and a **double hyphen `--`** for the **child / sub-element** class (not another underscore before the child name).

- **Pattern:** `{base}_{camelCaseSubclass}--{childClass}`
- **Example:** `meetingPrep_generatedText_list` → **`meetingPrep_generatedText--list`**

When adding or renaming classes in SCSS, follow this pattern so child pieces are visibly distinct from block segments. Older modules may still use `&_element`; prefer `--element` for **new** child classes unless you are extending an existing `_` pattern in that file.

## Variants and “flavours”: custom properties, not overwrites

When a component already sets a CSS property (e.g. `padding`, `border-radius`) and a **variant or flavour** needs a **different value** for that same property, **do not** add a competing declaration that “overrides” the base in another modifier block. Instead **expose a surface as a variable** (CSS custom property preferred in `phoenix-fe`) and change the **value of that variable** per flavour.

**Reference:** `src/assets/elements/status/ptStatus.scss`

- Defaults live in something like `@mixin statusVars` (`--statusPadding`, `--statusBorderRadius`, …).
- The base styles **consume** those: e.g. `padding: var(--statusPadding);`, `border-radius: var(--statusBorderRadius);`.
- Flavours/mutators **only reassign variables**, e.g. `@mixin variantPillBordered` sets `--statusPadding: 2px 6px;` and `--statusBorderRadius: 50px;` instead of redefining raw `padding` / `border-radius` against the base mixin output.

Apply the same idea elsewhere: if you need a second padding for another flavour, add or reuse a `--componentPadding`-style token and switch it per modifier; avoid duplicating the longhand property unless there is no shared hook.

## Copy/polish guardrails

- Avoid adding `letter-spacing` unless there is explicit design direction.
- Preserve existing spacing/rhythm unless change is required by request or bug.
- Do not introduce hardcoded user-facing copy changes in SCSS-driven states without matching product direction.

## Review-comment handling policy

- If a review comment is a pure style/convention fix, apply directly.
- If applying a review comment changes UX or runtime behavior, stop and ask the user for permission first.

## Execution checklist

- [ ] Reuse existing mixins where possible.
- [ ] Remove redundant CSS replaced by mixins.
- [ ] Validate fontWeight tokens are supported.
- [ ] Keep selector patterns consistent with surrounding code.
- [ ] For variant-specific values of the same property, prefer **custom properties** (see `ptStatus.scss`) instead of overwriting longhand in another modifier.
- [ ] New or renamed classes follow `{base}_{camelCaseSubclass}--{childClass}` (child uses `--`, not `_`).
- [ ] Confirm no unintended UX/behavior changes.
- [ ] Run lints on edited files.
