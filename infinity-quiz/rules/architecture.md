---
paths:
  - 'src/**'
---

# Architecture

Infinity Quiz is a client-side Vue 3 single page application. There is no
backend, no authentication and no network call: every quiz, attempt, result and
preference lives in `localStorage`, through Pinia and
`pinia-plugin-persistedstate`.

## Routes

| Path | Name | View | Purpose |
| --- | --- | --- | --- |
| `/` | `quiz-list` | `QuizListView` | Catalogue, search, launch a quiz |
| `/quiz/:quizId` | `quiz` | `QuizView` | Play the quiz: questions, timer, feedback |
| `/results` | `results` | `ResultsView` | Score, review mode, restart |
| `/statistics` | `statistics` | `StatisticsView` | Charts built from the attempt history |
| `/manage` | `quiz-management` | `QuizManagementView` | List, duplicate, delete quizzes |
| `/create` | `quiz-create` | `QuizFormView` | Create a quiz, step by step |
| `/edit/:quizId` | `quiz-edit` | `QuizFormView` | Edit an existing quiz |
| `/import-export` | `quiz-import-export` | `QuizImportExportView` | JSON import and export |
| `/:pathMatch(.*)*` | `not-found` | `NotFoundView` | Catch-all |

## Layers

- `src/views/`: route screens. They own page state, read route parameters and
  compose domain components.
- `src/components/itheme/`: the widgets `@infinitylib/infinity-theme` does not
  ship yet (card, badge, alert, dialog, progress, toast, steps, file input).
  `It*` names, typed props in `itheme/types.ts`, `it-*` classes, `--it-*`
  tokens only, no domain knowledge, no store access. Each one is a candidate
  for upstream; see ADR-002.
- `src/components/layout/`: the application shell (`AppLayout`, `AppSidebar`)
  and the `L*` layout primitives (`LContainer`, `LGrid`, `LFlex`).
- `src/components/common/`: cross-feature UI: modal, notification, theme
  toggle.
- `src/components/quiz/`: domain UI, grouped by role: `card/` for the quiz
  surfaces, `form/` for the creation steps, `charts/`, `modal/`.
- `src/components/quiz-import-export/`: the import and export cards.
- `src/composables/`: stateful Vue logic, one concern per file.
- `src/stores/`: Pinia stores, the single source of shared state.
- `src/types/`: domain types. `Quiz`, `Question`, `QuestionResult`, the attempt
  and statistics shapes.
- `src/utils/`: pure functions, no Vue import, no side effect.
- `src/constants/`: score thresholds, pass threshold, storage keys, the theme
  modes and accents. Never repeat those literals elsewhere.
- `src/css/style.css`: the cascade order, declared before the theme package.
- `src/css/brand.css`: the accent, the typeface, the `--iq-*` tokens and the
  deliberate overrides of the package.
- `src/css/itheme/`, `src/css/layout/`, `src/css/views/`, `src/css/quiz/`: the
  application CSS, one file per widget, per shell concern or per screen.
- `src/router/index.ts`: the whole route table.
- `src/__tests__/`: tests, mirrored by layer (`composables/`, `stores/`,
  `utils/`).

## Dependency direction

`views` -> `components` -> `composables` / `stores` -> `utils` / `types` /
`constants`.

- A lower layer never imports a view or a component.
- `components/itheme` and `components/layout` import neither a store nor a
  domain type: they stay reusable, and promotable to the theme package.
- `utils` and `types` import no Vue, no Pinia.
- `stores` never import a component or a composable.

## Barrels

Every component folder ships an `index.ts` re-exporting what it owns, and the
parents (`components/common/index.ts`, `components/quiz/index.ts`,
`components/itheme/index.ts`, `stores/index.ts`, `composables/index.ts`)
aggregate them. Import from the barrel of a folder, not from its files. The
theme package's own wrappers are imported from
`@infinitylib/infinity-theme-vue`, never re-exported by a barrel here.

## Adding code

| You are adding | It goes in |
| --- | --- |
| A screen | `src/views/<Name>View.vue` plus a route in `src/router/index.ts` |
| A reusable visual | `src/components/common/` |
| A widget the theme package lacks | `src/components/itheme/It<Widget>.vue` + `src/css/itheme/<widget>.css` |
| Quiz UI | `src/components/quiz/<role>/` |
| Shared state | `src/stores/`, exported from `src/stores/index.ts` |
| A calculated value | A `computed` in the owning store, or `src/utils/` when pure |
| A domain shape | `src/types/` |
| A threshold, a key, a literal | `src/constants/` |
