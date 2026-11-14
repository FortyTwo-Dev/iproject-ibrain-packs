---
paths:
  - 'src/components/**'
  - 'src/views/**'
  - '*.vue'
---

# Components

The generic component and accessibility rules apply. This adds the shape this
project uses.

## File anatomy

`<script setup lang="ts">` first, then `<template>`, then `<style>` only when a
utility class genuinely cannot express the result. Props are a typed `interface`
with `withDefaults`, emits are a type literal:

```ts
const props = withDefaults(defineProps<Props>(), { size: 'md' })
const emit = defineEmits<{ select: [answerIndex: number]; skip: [] }>()
```

## The two wrapper families

- `src/components/daisy-ui/`: `D*`. A wrapper maps its typed props to daisyUI
  classes and renders the markup; it holds no state, no store, no domain type.
  Classes come from a `Record<Variant, string>` (`DButton.vue`) or from a pattern
  safelisted in `src/css/main.css`; see `styling`.
- `src/components/layout/`: `L*`. Layout primitives and the shell. They accept
  slots and layout props, nothing else.

A wrapper that needs to know about a quiz has outgrown its layer: the domain
component belongs in `src/components/quiz/` and uses the wrapper.

## Domain components

- Cards in `quiz/card/`, form steps in `quiz/form/`, charts in `quiz/charts/`.
- A component receives data through props and signals intent through emits.
  Reading a store directly is reserved for surfaces that are the page's own
  content, and never for a daisyUI or layout component.
- A view owns its page state (`useQuizForm`, route parameters) and composes
  domain components; it does not re-implement scoring or shuffling.

## Template

- `@event` and `:prop` shorthands. Kebab-case attributes.
- `v-for` always keys on a stable id, never on the index when the list reorders.
- `v-if` and `v-for` never on the same element: filter in a computed.
- No expression longer than a name in the template; move it to a computed.
- Every interactive control is reachable and operable with the keyboard, and has
  an accessible name.
- Clean up what you start: a timer, an interval or a listener is cleared in
  `onUnmounted`.
