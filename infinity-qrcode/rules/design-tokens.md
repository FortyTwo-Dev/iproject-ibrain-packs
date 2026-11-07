---
paths:
  - 'src/theme/**'
  - 'src/components/theme/**'
  - 'src/css/**'
  - 'src/**/*.vue'
---

# Design tokens

The application uses the in-repo Infinity Theme library. The visual identity is
the logo yellow as the single accent, light/dark modes, square corners and the
existing layout proportions (ADR-0005).

## Themes

- The theme is scoped by `.it-theme` on `<html>`.
- The mode is `data-theme="light|dark"` on `<html>`.
- The accent is always `data-accent="yellow"` on `<html>`; the library keeps the
  accent mechanism, but the application ships no other accent.
- Persist the mode in `localStorage`. The accent is not stored.
- Do not reintroduce the old `light-blue` / `dark-red` classes or the
  cookie-per-domain hack.

## Tokens

- Library tokens use `--it-*`: `--it-bg`, `--it-fg`, `--it-surface`,
  `--it-border`, `--it-accent`, `--it-accent-fg`, `--it-danger`.
- Component tokens use `--it-button-*` and `--it-input-*`.
- No utility framework is installed (ADR-0008). The page arrangement lives
  in `src/css/layout/` as named application classes on the structural
  tokens (`--it-space-*`, `--it-gutter`, `--it-container-*`,
  `--it-section-gap`, `--it-stack-gap`); the brand layer (the yellow
  accent binding, the typefaces, the bright-accent behaviours) lives in
  `src/css/brand.css`.
- Before inventing a new class, read the existing vocabulary of
  `src/css/layout/` (`app-container`, `page-*`, `stack`, `row`,
  `section-stack`, `panel-title`, `notice-*`, `error-box`, ...).
- Every class referenced in a template must resolve to the theme package or
  to the application CSS. An inert class name is a hidden regression: audit
  the vocabulary whenever a styling source is added or removed.
- Do not hard-code a colour that a token already names. A hexadecimal value in a
  component is only acceptable as data (for example a QR colour preview).
- Radius is a token (`--it-radius`), currently `0px`. Do not introduce rounded
  corners in one component.

## Components

- Base HTML elements are styled by the library inside `.it-theme`.
- Use `it-*` classes only for variants and sizes:
  `it-btn-primary`, `it-btn-sm`, `it-input-lg`, `it-h5`, `it-p`, `it-img`.
- Header navigation uses `.it-nav-link`; the current page is marked by
  `aria-current="page"` and gets the accent underline. Do not style the active
  route as a primary button.
- Mutually exclusive choices use `.it-choice-group` (with `-2` or `-4`) and a
  `.it-choice` label wrapping a native radio. Do not rebuild it with buttons.
- Selects stay native. Style the closed control with `appearance: none` and an
  inset chevron. Where `appearance: base-select` is supported, style
  `::picker(select)` with the theme tokens; other browsers keep the native
  picker. Do not add a select library.
- Do not add a second component style in a Vue file. The CSS library is the
  source of truth.
- The Vue wrappers live in `src/components/theme/` and contain no `<style>`.

## Typography

- The base typeface is Space Grotesk, loaded from `src/assets/fonts` under the
  SIL OFL 1.1.
- OpenDyslexic, under the Bitstream Vera license, is an accessibility
  alternative. The header button toggles `data-font="dyslexic"` on `<html>`;
  the choice is persisted in `localStorage` under `readingFont`.
- Never set a font family in a component. Use `--it-font-sans`, which the
  `data-font` override replaces.

## Cascade

Layer order is declared once, at the top of `src/css/style.css`:

```
it.tokens, it.base, it.variants, components, it.overrides
```

- `src/css/style.css` is loaded by the index before the module scripts, so
  its statement wins and the theme package's own statement merges into it.
- `components` carries the application classes of `src/css/layout/`,
  above the theme variants: an app class overrides a library default at
  equal specificity.
- `it.overrides` stays last for a deliberate, documented exception.

## Verification

- There is no styleguide page. Check a token, a base element or a variant on
  the app itself.
- Check light and dark modes and the focus/disabled states.

## Layout

- Generator, desktop: three columns. Content type on the left (it fills the
  column), preview then content fields in the middle, options on the right.
- Generator, mobile: content type, content fields, preview, options. The fields
  come before the preview so the form is reachable without scrolling past an
  empty preview.
- The preview is capped: the on-screen QR fits the preview box and never
  resizes the page. `moduleSize` only controls the exported file, not the
  preview.
- Learn: controls, stage and explanation. Mobile shows the stage first and the
  controls below.
- Keep the header/footer identity: theme controls in the header, version in the
  footer.

## Accessibility

- Both modes meet contrast requirements.
- Focus is visible.
- Motion respects `prefers-reduced-motion`.
