---
paths:
  - 'src/composables/**'
---

# Composables

The generic composable rule applies. What follows is where the boundary sits in
this project.

## Reach for a composable when

- the logic has state that lives as long as a component (`useQuizForm` owns a
  draft quiz, `useSeededRandom` owns a generator);
- the logic touches a lifecycle hook: an interval, a listener, a subscription;
- two components share the same behaviour without sharing it globally.

## Do not use one when

- the value is pure: it belongs in `src/utils/`. `useScore` composes the pure
  helpers, it does not replace them.
- the state must survive navigation: it belongs in a store. A composable that
  calls `defineStore` internally hides the ownership; export the store instead.
- the logic serves one component: keep it in that component.

## Shape

- One file per composable, `useX` camelCase, exported from
  `src/composables/index.ts`.
- Take primitives, refs or a typed options object; return an object of refs,
  computeds and functions. Never return an internal `let`.
- Clear what you start in `onUnmounted`, and make `stop` / `reset` idempotent:
  the timer and the seeded random generator are the two places this matters.
- A computed over an action whenever the value is derived: the score is not
  stored, it is computed from the answers.
- Document the purpose in one JSDoc sentence on the exported function. Skip
  `@param` / `@returns` when the signature says it.

## Current set

| Composable | Concern |
| --- | --- |
| `useQuiz` | Reads the catalogue and launches an attempt |
| `useAttempt` | Drives one attempt: answer, navigate, complete |
| `useScore` | Score and threshold evaluation |
| `useResults` | Result summary and review material |
| `useFeedback` | Immediate feedback after a selection |
| `useQuizList` | Catalogue filtering, search, categories |
| `useQuizManagement` | Create, duplicate, delete flows |
| `useQuizForm` | The multi-step creation form and its validation |
| `useQuizImportExport` | JSON import and export |
| `useStatistics` | Aggregates over the history |
| `useTheme` | Global theme, quiz theme, imported themes |
| `useSeededRandom` | Reproducible shuffle from `quizSeed` |

A new composable that overlaps one of these is a duplication: extend the
existing one, or name the difference explicitly in the pull request.
