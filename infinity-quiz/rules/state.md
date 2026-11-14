---
paths:
  - 'src/stores/**'
---

# State

The generic Pinia rule applies. This one lists the state this project actually
owns and how it is persisted.

## The four quiz stores

| Store | File | Owns |
| --- | --- | --- |
| `useQuizStore` | `quiz/quiz-store.ts` | The catalogue: quizzes, categories, tags, search, create/update/delete/duplicate, import/export of the catalogue |
| `useQuizSessionStore` | `quiz/quiz-session-store.ts` | One attempt in progress: current question, answers, skips, verification, score, timer, completion |
| `useQuizHistoryStore` | `quiz/quiz-history-store.ts` | Finished attempts, the material the statistics screen reads |
| `useQuizVerificationStore` | `quiz/quiz-verification-store.ts` | A derived view of the session: whether the current answer is verified, whether feedback shows, which button is offered. It delegates to the session store and owns no state of its own |

Outside the quiz domain: `useThemeStore` (`theme-store.ts`) and
`useNotificationStore` (`notification-store.ts`).

Every store is a setup store (`defineStore('id', () => { ... })`), returns every
property it exposes, and is re-exported from `src/stores/index.ts`.

## Ownership

- An attempt belongs to `useQuizSessionStore`. A view never keeps a copy of the
  current question, the score or the timer.
- The catalogue belongs to `useQuizStore`. A form edits a local draft and commits
  it through an action.
- A finished attempt is written once, to the history, when the session completes.
- A value that one screen needs stays in that screen: no store for it.

## Persistence

`pinia-plugin-persistedstate` writes to `localStorage`. Every store declares an
explicit storage key and an allowlist of the fields it persists:

| Store | Key | Persisted |
| --- | --- | --- |
| quiz | `STORAGE_KEYS.quizzes` | `quizzes` |
| quiz session | `STORAGE_KEYS.session` | `currentQuizId`, `currentQuestionIndex`, `selectedAnswers`, `skippedQuestions`, `verifiedQuestions`, `score`, `isCompleted`, `shuffledQuiz`, `quizSeed`, `timeLeft` |
| quiz history | `STORAGE_KEYS.history` | `results` |
| theme | `STORAGE_KEYS.theme` | `globalTheme` (a `{ mode, accent }` pair) |

- The key comes from `STORAGE_KEYS` in `src/constants/index.ts`, never a literal
  string.
- New persisted state means a new entry in `pick`, and a migration when the
  shape changes: stored data comes from an older build and is untrusted on read.
- Never persist derived state (a score computed from answers, a filtered list) or
  transient UI state (a modal being open, a loading flag).
- `quizSeed` is persisted on purpose: it replays the same shuffle after a reload.
- The theme payload changed shape with the migration: `src/main.ts` rewrites it
  before Pinia rehydrates, and the legacy daisyUI names (`infinity-blue-light`, …)
  still resolve through `LEGACY_THEME_NAMES`. Read a stored theme through
  `normalizeGlobalTheme` / `normalizeQuizTheme`, never directly.

## Testing

`src/__tests__/stores/setup.ts` provides `setupTestPinia()`,
`installTestPinia()` and `readStorage()`. Use them; do not rebuild a Pinia
harness in a spec. See `testing` for the layout.
