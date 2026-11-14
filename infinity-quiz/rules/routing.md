---
paths:
  - 'src/router/**'
  - 'src/views/**'
---

# Routing

The generic routing rule applies. Project specifics:

- One route table, in `src/router/index.ts`, built with
  `createWebHistory(import.meta.env.BASE_URL)`. There is no nested layout route:
  `AppLayout` wraps the router view once, in `src/App.vue`.
- Every route is lazy: `component: () => import('@/views/<Name>View.vue')`.
- Names are kebab-case and stable, they appear in `<RouterLink :to="{ name }">`:
  `quiz-list`, `quiz`, `results`, `statistics`, `quiz-management`,
  `quiz-create`, `quiz-edit`, `quiz-import-export`, `not-found`.
- A route that carries a parameter sets `props: true` and the view receives it
  as a typed prop. Only a composable that owns the flow may read the route
  (`useQuiz` reads `route.params.quizId`); a view does not reach into
  `route.params` itself.
- The catch-all `/:pathMatch(.*)*` to `NotFoundView` stays last.
- Adding a screen means adding its row to `architecture` in the same change.

## No guards

The application is local-first with no account: there is no auth guard, no
`meta.requiresAuth`, no redirect to a login screen. A route that needs a quiz
that does not exist sends the user back to the catalogue from the view, with a
notification; a guard that silently redirects hides a data problem.

## Navigation

- `router.push` from a composable or a store action only when the flow owns the
  transition (finishing an attempt goes to the results). A component that only
  offers a destination uses `<RouterLink>`.
- Leaving an attempt in progress is a decision, not an accident: the session is
  persisted, so a back navigation does not lose answers.
