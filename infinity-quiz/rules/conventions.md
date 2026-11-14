---
paths:
  - 'src/**'
---

# Conventions

The generic packs already carry the Vue and TypeScript baseline. This rule only
states what is specific to Infinity Quiz; where it is silent, the generic rule
applies.

## TypeScript

- The project is strict through `@vue/tsconfig/tsconfig.dom.json`, and
  `noUncheckedIndexedAccess` is on: an indexed read returns `T | undefined`.
  Handle that case, do not assert it away.
- No `any`. Narrow an `unknown` with a type guard.
- `interface` for an object shape or a component's props; `type` for a union, an
  intersection, a mapped type.
- Annotate the return type of an exported function. Let inference handle a local
  variable whose type is obvious.
- A literal set of values is `as const` plus a derived union, as in
  `src/constants/themes.ts` (`THEME_COLORS` -> `ThemeColor`). No `enum`.
- Validate data that crosses a trust boundary (an imported JSON file, a value
  read from storage) with the Zod schema in `src/utils/validation.ts`, then use
  its inferred type. A cast is not validation.

## Naming

| Thing | Convention | Example |
| --- | --- | --- |
| Variable, function | camelCase, verb for a function | `currentQuestion`, `calculateScore` |
| Boolean | `is` / `has` / `can` / `should` prefix | `isCompleted`, `canSubmit` |
| Constant | UPPER_SNAKE_CASE | `PASS_THRESHOLD`, `STORAGE_KEYS` |
| Component file | PascalCase, one component per file | `QuizQuestionCard.vue` |
| View | PascalCase, `View` suffix | `ResultsView.vue` |
| Theme widget | `It` prefix, like the package | `ItCard.vue`, `ItBadge.vue` |
| Layout primitive | `L` prefix | `LContainer.vue`, `LGrid.vue` |
| Composable file | `useX` camelCase | `useSeededRandom.ts` |
| Store file | kebab-case, `-store` suffix | `quiz-session-store.ts` |
| Any other module | kebab-case | `array-utils.ts`, `theme.ts` |
| Test file | kebab-case, `.spec.ts` suffix | `useScore.spec.ts` |
| Store id | camelCase, the name after `use` in the store's hook | `useQuizSessionStore` -> `'quizSession'` |

## Shape of a module

- One concern per file; split around 300 lines.
- Functions stay around 20 lines. Guard clauses and early returns over nesting.
- Three parameters maximum; a fourth argument becomes an object.
- Array methods over indexed loops; `for...of` when a loop is needed.
- No magic number or string: a threshold goes to `src/constants/`, a persisted
  key to `STORAGE_KEYS`.
- A comment explains a decision or a workaround. It never restates the code.

## Imports

- `@/` for anything that crosses a layer, from `src/components`, `src/views`
  and `src/router`.
- Inside the logic layers (`stores`, `composables`, `utils`) a relative path is
  the norm, and one level (`../stores`) is enough. The nested quiz stores still
  read `../../types` and `../../constants`; that two-level form is the deviation
  to align when one of those files is edited, not a pattern to copy.
- Order: `vue`, `pinia`, `vue-router`, then other packages, then `@/`, then
  relative. `import type` for a type-only import.
- A file that deviates is aligned when it is edited; no dedicated rewrite sweep.

## Tooling

| Command | What it does |
| --- | --- |
| `bun run lint` | oxlint then eslint, both with `--fix` |
| `bun run type-check` | `vue-tsc --build` |
| `bun run format` | Prettier over `src/` |
| `bun test:coverage` | The whole suite with coverage |

Prettier owns the form: no semicolon, single quote, 100 columns. Never
hand-format against it, and never disable a lint rule to make a change pass.
