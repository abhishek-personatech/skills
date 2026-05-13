---
name: phoenix-scss
description: >-
  Phoenix FE SCSS and CSS: BEM-style class names, CSS custom properties, tokens,
  overrides, nesting, and global vs component styles. Use for any CSS change,
  CSS fix, CSS update, stylesheet work, styling, SCSS/Sass edits, theme or
  layout tweaks, class renames, or reviewing style-related PRs in this repo.
---

# Phoenix FE — SCSS conventions

## When this skill applies

Use whenever the work involves **CSS or SCSS**: changes, fixes, updates, refactors, new styles, design-token usage, BEM/class naming, custom properties, or style review—even if the user only says “CSS”, “styles”, or “scss”.

## Where styles live

- **Global tokens and themes:** `src/assets/styles/` (`_theme_constant.scss`, `variables.scss`, `_mixin.scss`, etc.) — see `CLAUDE.md` for the full styling overview.
- **Component / feature styles:** Co-locate `*.scss` next to the component (e.g. `meetingPrep/meetingPrep.scss`).

Prefer **design tokens** (`var(--primary)`, `var(--text-md)`, …) over raw hex or magic numbers when a token already exists.

## Class naming (BEM-like, Phoenix FE)

- **Block:** single root segment per feature area, e.g. `meetingPrep`, `meetingPrep_sectionWrapper`.
- **Sub-parts:** use a **double hyphen** before the segment name (not `_subpart` for new code):  
  `meetingPrep_generatedText--paragraph`, `meetingPrep_state--icon`, `meetingPrep_attendeeIdentity--name`, `meetingPrep_sectionWrapper--nested`.
- **Modifier-style sections** on the same block read clearly in TSX and match selectors compiled from nested `&--foo` under `&_block`.

Keep TSX `className` strings aligned with compiled class names from the SCSS (search both when renaming).

## CSS custom properties (`--variableName`)

- **Naming:** camelCase after the double hyphen, e.g. `--sectionContentPadding`, `--metricChipBorderColor`.
- **Define defaults** on the **smallest real DOM root** that wraps all consumers. If SCSS nests variables under `.myBlock { --foo: … }`, at least one mounted element must have class `myBlock` in the tree, or `var(--foo)` will not resolve. Portals / modals: put that root class on an ancestor that actually wraps the UI (e.g. content wrapper), not only on leaf nodes.
- **One conceptual token, one name:** expose a single property (e.g. `--sectionContentPadding`) and **reassign it on modifier ancestors** instead of introducing `--sectionContentPaddingTight` _and_ separate `padding` overrides, when the same property is what changes.
- **Do not** promote every literal to a custom property: **one-off** values that are not reused and do not participate in a clear override story can stay as plain declarations.

### Theming a sub-surface (e.g. metric chip)

Group related overridable properties into **namespaced** variables on the block (e.g. `--metricChipBorderColor`, `--metricChipBackground`, `--metricIconValueColor`) with defaults on the block; **modifier classes** on the same block only reassign those variables. Shared layout rules consume `var(--…)` once.

### Section layout tokens (padding, header, title)

Same pattern: defaults on the component root; **nested section wrappers** override `--sectionHeaderPadding`, `--sectionTitleFontSize`, `--sectionContentPadding` so `meetingPrep_sectionHeader` / `meetingPrep_sectionContent` keep a single `var()` each.

## SCSS authoring

- **Nesting:** Prefer an explicit parent when using the direct child combinator: `& > .meetingPrep_sectionContent` rather than bare `> .meetingPrep_sectionContent` when the team wants symmetry with other rules.
- **`&` + BEM:** Under `&_generatedText`, direct children use `& > &--list` / `& > &--paragraph` so compiled selectors stay `.meetingPrep_generatedText > .meetingPrep_generatedText--list`.
- **Imports:** Component SCSS that uses `@include` relies on the app’s Sass include paths (CRA / shared partials); do not assume standalone `sass` CLI compiles a single file without those imports.

## Numbers and literals

- Prefer **at most one digit after the decimal** for line-height and similar when the team calls it out (e.g. `1.4` not `1.45`), unless an existing design token dictates otherwise.

## Checklist when touching SCSS

- [ ] Class names in TSX match compiled selectors after SCSS changes.
- [ ] Custom properties are defined on an ancestor that **exists in the DOM** for every render path (modal, storybook, tests).
- [ ] Modifier overrides reassign the **same** token names where the design is “same knob, different value.”
- [ ] No dead selectors left after renames; grep for old class strings.

## Evolving this skill

When a new UI area establishes a repeatable SCSS pattern (or a pitfall, e.g. missing token root), add a short generalized rule here—not ticket-specific copy.
