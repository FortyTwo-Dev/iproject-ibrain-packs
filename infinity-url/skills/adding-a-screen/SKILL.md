---
name: adding-a-screen
description: Add or change a dashboard screen in Infinity URL, with its empty state, error state, i18n keys, capability gate and component test. Use for any new page or panel in the Nuxt application.
---

# Add a screen

## Objective

Add a screen that matches `docs/14-ecrans.md`, uses the shared theme, holds no
token in persistent storage, and is testable with an empty database.

## Procedure

### 1. Read the specification

- Open `docs/14-ecrans.md` and find the screen. If it is not there, stop: a new
  screen is a product decision.
- Copy the eight fields: role, primary action, displayed data, empty state, error
  state, loading state, capability, i18n namespace.
- If a field is missing, ask rather than invent.

### 2. Routing and access

- One file under `pages/`, the name matching the route in the specification.
- The dashboard is `ssr: false` and `noindex`; the marketing pages are
  prerendered. Do not mix the two.
- Check the capability from `GET /me` **before** rendering the screen. A hidden
  capability hides the entry point *and* the route: never rely on a disabled
  button alone.

### 3. Composition

- Build with `@infinitylib/infinity-theme-vue` wrappers and `it-*` classes.
- No Tailwind utility, no inline palette, no new colour. If a component is
  missing, compose tokens rather than inventing a style.
- State goes in a Pinia store only when it is shared; a screen's local state stays
  local.

### 4. States

Four states are mandatory, and each is a real code path:

| State | Requirement |
| --- | --- |
| Loading | Skeleton, never a spinner on an empty page |
| Empty | A sentence explaining the state **and** the primary action |
| Error | The server's `code` translated, plus a retry that actually retries |
| Success | The nominal rendering |

Never render a blank screen while loading, and never render `0` where the data
has not arrived.

### 5. Data

- Call the API through the generated typed client, never a hand-written fetch.
- The access token comes from memory; on `401`, refresh once, then send the user
  to login. Never retry in a loop.
- Format dates and numbers with `Intl` and the active locale.

### 6. i18n

- Every string is a key in the screen's namespace: `links.list.empty`, not
  `"Aucun lien"`.
- Add the key to both `fr` and `en`. A missing `en` key is acceptable at launch
  **only** if the French text is reused explicitly, never a blank.
- No string concatenation to build a sentence: use the plural and interpolation
  features of the i18n library.

### 7. Test

- One component test per state: empty, error, nominal (and loading when it is not
  trivial).
- Mock the API at the client boundary, not the store.
- One Playwright journey only if the screen is part of the three E2E paths.

### 8. Definition of done

- [ ] The eight fields of the specification are implemented.
- [ ] Capability checked from `/me`.
- [ ] Four states present.
- [ ] Every string is an i18n key, present in `fr` and `en`.
- [ ] No token in `localStorage` or a cookie.
- [ ] Component tests green.
- [ ] No third-party request, no external font, no analytics script.
