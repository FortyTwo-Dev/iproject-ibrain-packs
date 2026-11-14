---
paths:
  - 'src/**/*.vue'
  - 'src/css/**'
---

# Styling

Tailwind CSS v4 and daisyUI 5, no component-scoped CSS framework, no CSS file
besides `src/css/main.css`.

## Where a style lives

| Need | Where |
| --- | --- |
| A utility, a layout | Tailwind classes in the template |
| A daisyUI variant of a prop | A `Record<Prop, string>` of literal classes, as in `DButton.vue` |
| A pattern daisyUI builds dynamically | `@source inline()` in `src/css/main.css` |
| A colour, a radius | The daisyUI theme blocks in `src/css/main.css` |
| A layout class shared by several views | `src/css/main.css`, with the other application layout classes |

## Dynamic classes

Tailwind v4 detects classes statically. A class assembled by interpolation is
invisible to it and disappears from the production build unless a pattern
safelists it. ADR-001 (`docs/decisions/`) settles the rule:

- a typed, project-specific variant uses a `Record` of full class names;
- a daisyUI class built from a prop (`card-${size}`) is only acceptable when the
  exact pattern is in the `@source inline()` list of `src/css/main.css`;
- `class="custom-${value}"` on a class Tailwind does not know is a bug: the page
  loses its style in production while looking fine in development.

Adding a variant means adding its class to the `Record` or its pattern to the
safelist, in the same change.

## Themes

- Built-in themes are generated in `src/constants/themes.ts`
  (`infinity-<color>-<variant>`, default `infinity-blue-light`); the daisyUI
  blocks live in `src/css/main.css`.
- The global theme belongs to `useThemeStore`; a quiz may carry its own `theme`,
  applied while it is played and restored afterwards.
- An imported theme is validated by `src/utils/theme-import.ts` before it is
  stored; a theme name is `[A-Za-z0-9_-]+`, never injected as raw CSS.
- Dark and light are both first-class: a new surface is checked in the default
  light theme and in `infinity-blue-dark`.

## Accessibility

Contrast, focus visibility and touch target size are not a styling afterthought:
a daisyUI variant that fails contrast in one of the shipped themes is a defect.
