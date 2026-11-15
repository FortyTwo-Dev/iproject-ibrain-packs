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

## Props

A prop that is not required declares its default. This is not cosmetic: Vue
casts an absent `boolean` prop to `false` before any component code runs, so a
fallback written in the body never fires.

```ts
// Dead code: `showFeedback` is already `false` when the caller omits it.
const props = defineProps<{ showFeedback?: boolean }>()
const showFeedback = computed(() => props.showFeedback ?? true)

// What the intent looks like when it is declared.
const props = withDefaults(defineProps<Props>(), { showFeedback: true })
```

- A boolean prop declares its default, `false` unless there is a reason.
- A component never tests a boolean prop against `undefined`: the third state
  does not survive the cast, so it cannot be expressed that way.
- A string or number prop that must stay absent says so explicitly:
  `withDefaults(defineProps<Props>(), { label: undefined })`.

## The two wrapper families

- `src/components/itheme/`: `It*`. A widget the theme package does not ship
  yet maps its typed props to `it-*` classes and renders the markup; it holds
  no state, no store, no domain type. Classes come from a
  `Record<Variant, string>` (`ItBadge.vue`); see `styling`. A control the
  package already ships is imported from `@infinitylib/infinity-theme-vue`
  and used directly, with no local copy.
- `src/components/layout/`: `L*`. Layout primitives and the shell. They accept
  slots and layout props, nothing else.

A wrapper that needs to know about a quiz has outgrown its layer: the domain
component belongs in `src/components/quiz/` and uses the wrapper.

## Domain components

- Cards in `quiz/card/`, form steps in `quiz/form/`, charts in `quiz/charts/`.
- A component receives data through props and signals intent through emits.
  Reading a store directly is reserved for surfaces that are the page's own
  content, and never for a theme widget or a layout component.
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
