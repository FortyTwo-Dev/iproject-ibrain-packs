---
paths:
  - 'src/**/*.vue'
  - 'src/css/**'
---

# Styling

`@infinitylib/infinity-theme` for the tokens and the component classes, plus
`@infinitylib/infinity-theme-vue` for the wrappers the package ships. No Tailwind,
no daisyUI, no component-scoped CSS framework. ADR-002
(`docs/decisions/`) records the migration and supersedes ADR-001.

## The cascade

`src/css/style.css` declares the layer order before anything else, because the
first declaration wins:

```css
@layer it.tokens, it.base, it.variants, components, it.overrides;
```

`index.html` loads that file, then `src/main.ts` imports the theme package, then
`brand.css`, then the application stylesheets. Declaring the order after the
package would silently reorder the layers.

## Where a style lives

| Need | Where |
| --- | --- |
| A token the package names | Bind it in `src/css/brand.css`, never a literal in a component |
| The accent, the typeface, the brand guard | `src/css/brand.css` |
| A widget the package does not ship | `src/css/itheme/<widget>.css` + `src/components/itheme/It<Widget>.vue` |
| The shell, the rail, the layout primitives | `src/css/layout/*.css` |
| A screen's own composition | `src/css/views/<screen>.css` |
| A quiz-specific surface | `src/css/quiz/index.css` |
| A deliberate override of a package knob | `@layer it.overrides` in `brand.css`, with a comment saying why |

A component never carries a `<style>` block, and no colour, radius or spacing
value is hard-coded: the value comes from an `--it-*` token, or from an `--iq-*`
token declared in `brand.css` when the package has no equivalent.

## Widgets and variants

- A typed variant uses a `Record<Variant, string>` of complete class names, as in
  `ItBadge.vue`; never assemble a class from a variable.
- A widget written here is a candidate for upstream: `it-*` classes, `--it-*`
  tokens only, one file per widget, an accessible name and a visible focus.
  ADR-002 states the promotion contract, including the collision rule on the
  `components` layer.
- A control the package ships (`ItButton`, `ItInput`, `ItSelect`, `ItTextarea`,
  `ItCheckbox`, `ItRadio`, `ItField`, `ItOption`) is imported from
  `@infinitylib/infinity-theme-vue` and used directly. No local wrapper copies it.
- The package styles every bare `<button>` as a text button, padding included.
  A custom button therefore states its own box (padding, border, size) or the
  glyph inside it gets squeezed.

## Themes

- The accent is the charte green, `#00ff96`, bound once in `brand.css` on
  `:root[data-accent='green']`. One application, one accent.
- The mode is a choice: `data-theme="light|dark"` on `<html>`, written by
  `useThemeStore` and normalized on read.
- `--iq-accent-ink` is the accent mixed towards the foreground, because a bright
  accent fails 4.5:1 as small text. A new accent binds its own ink.
- A quiz may carry `theme: { mode, accent }`; it is applied while the quiz plays
  and the global selection is restored afterwards. A quiz without a theme keeps
  the global one.
- Tints are mixed in `oklab`, not `oklch`: white has no hue to interpolate
  towards in oklch, and a blue accent came out lavender.

## Accessibility

Contrast, focus visibility and touch target size are not a styling afterthought.
A surface is checked in both modes, with its accent, and a variant that fails
contrast in one of them is a defect, not a preference.
