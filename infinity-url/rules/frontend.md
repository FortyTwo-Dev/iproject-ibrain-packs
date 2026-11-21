---
paths:
  - 'app/pages/**'
  - 'app/components/**'
  - 'app/composables/**'
  - 'app/layouts/**'
  - 'i18n/**'
  - 'locales/**'
  - 'server/**'
  - 'nuxt.config.ts'
  - '**/*.vue'
---

# Frontend

Nuxt 4, Vue 3, Pinia. The screens are specified in `docs/14-ecrans.md`; follow it
rather than inventing a layout.

## Styling

- `@infinitylib/infinity-theme` owns the tokens and the `it-*` classes.
- `@infinitylib/infinity-theme-vue` only maps props to those classes. It ships no
  styles.
- **No Tailwind, no second design system.** A component that needs a style
  composes existing tokens; it does not introduce a palette.
- The dashboard must look like Infinity QRCode: same tokens, same dark theme,
  same accent.

## Capabilities, not builds

`GET /me` returns `capabilities`. The dashboard hides a feature whose capability
is absent; it never branches on an edition flag or on an environment variable.
One front build, always.

## Tokens

- The access token lives **in memory only**. It is never written to
  `localStorage`, `sessionStorage`, a cookie or IndexedDB.
- On load, the app calls `POST /auth/refresh` with `credentials: 'include'` to
  obtain an access token. It never reads the refresh cookie: it is `HttpOnly`.
- Every API call goes to `api.url.infinityproject.fr` with an explicit
  `Authorization: Bearer`.

## i18n

- `@nuxtjs/i18n`, `prefix_except_default`: `/` is French, `/en/…` is English.
- The locale lives **in the URL only**. No locale cookie, no automatic redirect
  based on `Accept-Language`.
- Never hardcode a user-facing string: every label is an i18n key.
- Dates and numbers go through `Intl`, with the active locale.

## No third-party anything

- No external font, no analytics script, no embedded widget, no Google-hosted
  resource. The product claims no third-party request; a marketing page is not an
  exception.
- The QR is generated **client-side** with `@infinitylib/iqrcode-wasm`. The
  destination never leaves the browser.

## Screens

Every screen defines: role, primary action, empty state, error state, loading
state, required capability, i18n namespace. A screen without these is not done.
