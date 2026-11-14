---
id: vue/testing
paths:
  - 'src/**'
---

# Testing in this project

This rule replaces the generic `vue/testing` rule. Where it is silent, the
generic determinism rules still apply.

## Runner

Vitest on the Bun runtime, not Bun's own test runner: `bun test` cannot compile
a `.vue` single file component.

| Command | Use |
| --- | --- |
| `bun run test:unit` | Watch mode while working |
| `bun run test:unit <filter>` | One file or one name |
| `bun test:coverage` | The full suite with coverage, the pre-push gate |
| `bun run type-check` | `vue-tsc --build` |
| `bun run lint` | oxlint then eslint |

`vitest.config.ts` sets `environment: 'jsdom'`, excludes `e2e/**`, and inlines
`zod` to work around a Bun `__esModule` interop bug (oven-sh/bun#3393). Do not
remove that inline list: without it, named imports from `zod` resolve to
`undefined` and the suite fails for the wrong reason.

## Layout

Tests live under `src/__tests__/`, mirrored by layer, never beside the source:

| Layer | Path | Example |
| --- | --- | --- |
| Composables | `src/__tests__/composables/<name>.spec.ts` | `useScore.spec.ts` |
| Stores | `src/__tests__/stores/<name>.spec.ts` | `quiz-session-store.spec.ts` |
| Utils | `src/__tests__/utils/<name>.spec.ts` | `array-utils.spec.ts` |
| Components | `src/__tests__/components/<feature>/<Name>.spec.ts` | `components/quiz/QuizQuestionCard.spec.ts` |

A test file is named after its subject in kebab-case. The store harness lives in
`src/__tests__/stores/setup.ts` and is imported, never copied.

## What must be covered

| Target | Cases |
| --- | --- |
| Score and thresholds | Exact threshold values, just below, just above, zero questions, all wrong |
| Attempt lifecycle | Start, answer, skip, navigate, verify, complete, restart; a reload restores the same attempt |
| Shuffle | The same seed gives the same order; a different seed gives another; answers and questions shuffle independently |
| Timer | Expiry completes the attempt, a paused attempt does not drain, remaining time survives a reload |
| Verification | A valid quiz passes, each invalid shape is reported with its reason, a quiz without questions is rejected |
| Import and export | Round trip preserves a quiz, malformed JSON is rejected with a message, an unknown field does not crash |
| Validation | Every Zod rule: missing field, empty title, out-of-range answer index, wrong type |
| Statistics | Empty history, one attempt, several attempts: averages, distribution, pass rate |
| Theme | Valid name accepted, invalid name rejected, an imported theme can be applied then removed |
| Components | Rendering states (empty, loading, error), emitted events, keyboard order, the visible effect of a selection |

## Invariants

- A persisted attempt rehydrates to the same question index, answers and score.
- `score` equals the number of correct answers, and a skipped question is never
  counted as correct.
- Completing an attempt appends exactly one history entry; restarting does not.
- The seeded shuffle is reproducible from `quizSeed` alone.

## Rules

- Deterministic: no real timer, no network, no unseeded random. Seed through
  `useSeededRandom` / `seedrandom` and fake the clock with `vi.useFakeTimers()`.
- Reset Pinia between tests with `setupTestPinia()`. Isolation is the harness's
  job: never add a reset to a store so a test passes.
- Do not weaken an assertion to get green; fix the code or the test.
- Coverage threshold: `lines: 80`, enforced by `vitest.config.ts`; the `lcov`
  report feeds the SonarQube analysis. Raise the case coverage, never lower the
  threshold.
- Before pushing: `bunx --bun oxlint . --fix && bunx --bun eslint . --fix --cache
  && bun run type-check && bun test:coverage`.
