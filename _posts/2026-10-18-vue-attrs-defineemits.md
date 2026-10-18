---
layout: post
title: Vue $attrs and defineEmits — Differences, Use Cases, and the Hidden Pitfall
subtitle: Understanding attribute fallthrough, emit declaration, and the tricky condition-checking trap that breaks your components silently
published: true
comment: true
tags: [vue, vuejs, frontend, javascript, components]
permalink: /post/2026-10-18-vue-attrs-defineemits
---

## Quick Recap — Props vs Attrs vs Emits

Before diving in, let's align on vocabulary:

| Concept | What it is |
|---|---|
| **props** | Data a parent explicitly passes, declared with `defineProps` |
| **emits** | Events a component fires, declared with `defineEmits` |
| **$attrs** | Everything else the parent passes that is NOT a declared prop or emit |

Understanding the boundary between these three is the key to avoiding the bugs this post is about.

---

## `$attrs` — The Fallthrough Bag

### What is it?

`$attrs` (accessed as `useAttrs()` in `<script setup>`) contains all attributes and event listeners that a parent passes to a component **but that are not declared** as props or emits.

```vue
<!-- Parent -->
<MyInput
  id="username"
  class="form-control"
  placeholder="Enter username"
  @focus="handleFocus"
  :modelValue="name"
  @update:modelValue="name = $event"
/>
```

```vue
<!-- MyInput.vue (script setup) -->
<script setup>
const props = defineProps(['modelValue'])
const emit = defineEmits(['update:modelValue'])
</script>
```

In this case `$attrs` contains: `{ id, class, placeholder, onFocus }`.

`modelValue` is NOT in `$attrs` — it was declared as a prop.  
`onUpdate:modelValue` is NOT in `$attrs` — it was declared as an emit.

---

### Fallthrough behavior (the default)

By default, Vue automatically applies `$attrs` to the **root element** of a component. This is called **attribute fallthrough** (or "transparent wrapping").

```vue
<!-- MyInput.vue — NO explicit binding needed -->
<template>
  <input :value="modelValue" @input="emit('update:modelValue', $event.target.value)" />
</template>
```

Result in the DOM:

```html
<!-- Vue automatically puts id, class, placeholder, onFocus on <input> -->
<input id="username" class="form-control" placeholder="Enter username" ... />
```

This is great for simple wrapper components — you get `class`, `style`, `id`, ARIA attributes, and native event listeners for free.

---

### Disabling fallthrough with `inheritAttrs: false`

Sometimes you have multiple root nodes, or you want to put `$attrs` somewhere other than the root:

```vue
<script setup>
import { useAttrs } from 'vue'

defineOptions({ inheritAttrs: false })
const attrs = useAttrs()
</script>

<template>
  <label>
    <span>Username</span>
    <!-- Manually place $attrs on the input, not the label -->
    <input v-bind="attrs" :value="modelValue" />
  </label>
</template>
```

Without `inheritAttrs: false`, both `<label>` (root) AND `<input>` would get the attributes — a double-apply bug.

---

## `defineEmits` — Declaring Your Component's Events

### What is it?

`defineEmits` tells Vue which events this component is allowed to fire. It returns the `emit` function:

```vue
<script setup>
const emit = defineEmits(['change', 'update:modelValue', 'close'])

function handleClick() {
  emit('change', newValue)
}
</script>
```

With TypeScript, you can type the payload:

```ts
const emit = defineEmits<{
  change: [value: string]
  'update:modelValue': [value: number]
  close: []
}>()
```

### What happens when you declare an emit?

When you declare `'update:modelValue'` in `defineEmits`, Vue **removes** `onUpdate:modelValue` from `$attrs`. It no longer falls through.

```vue
<script setup>
// declared here -> removed from $attrs
const emit = defineEmits(['update:modelValue'])
</script>
```

If you do NOT declare it, `onUpdate:modelValue` stays inside `$attrs` and will fallthrough to the root element (or wherever you spread `v-bind="$attrs"`).

---

## Differences at a Glance

| | `$attrs` | `defineEmits` |
|---|---|---|
| **Purpose** | Captures undeclared attributes/listeners | Declares what events this component emits |
| **Where lives** | Runtime object, read via `useAttrs()` | Compile-time declaration macro |
| **Reactivity** | Reactive (updates when parent re-renders) | Static declaration |
| **Effect on DOM** | Falls through to root by default | Strips matching listeners FROM `$attrs` |
| **Use case** | Wrapper/pass-through components | Encapsulated component with explicit API |

---

## Use Cases

### Use case 1: Transparent input wrapper (uses `$attrs`)

You want `MyInput` to behave exactly like a native `<input>` from the parent's perspective:

```vue
<!-- MyInput.vue -->
<script setup>
defineOptions({ inheritAttrs: false })
const attrs = useAttrs()

const props = defineProps({
  modelValue: String,
  label: String,         // consumed here, NOT in attrs
})
const emit = defineEmits(['update:modelValue'])
</script>

<template>
  <div class="input-wrapper">
    <label>{{ label }}</label>
    <!-- All other attrs (id, placeholder, disabled, @focus...) go here -->
    <input
      v-bind="attrs"
      :value="modelValue"
      @input="emit('update:modelValue', $event.target.value)"
    />
  </div>
</template>
```

The parent can write:

```vue
<MyInput
  v-model="name"
  label="Username"
  id="username-field"
  placeholder="Enter your name"
  :disabled="isLoading"
  @focus="trackFocus"
/>
```

And `label` is consumed internally while everything else flows through to `<input>`.

---

### Use case 2: Button with explicit emit API (uses `defineEmits`)

A `ConfirmButton` that emits `confirm` and `cancel`:

```vue
<!-- ConfirmButton.vue -->
<script setup>
const emit = defineEmits(['confirm', 'cancel'])
</script>

<template>
  <div class="button-group">
    <button @click="emit('confirm')">OK</button>
    <button @click="emit('cancel')">Cancel</button>
  </div>
</template>
```

```vue
<!-- Parent -->
<ConfirmButton @confirm="save" @cancel="discard" />
```

Here `onConfirm` and `onCancel` are stripped from `$attrs` and handled via `emit`. The component has a clear, intentional public API.

---

### Use case 3: Polymorphic component (uses both)

A `BaseButton` that can render as `<button>`, `<a>`, or a `<RouterLink>`:

```vue
<script setup>
import { computed } from 'vue'
import { useAttrs } from 'vue'

defineOptions({ inheritAttrs: false })
const attrs = useAttrs()

const props = defineProps({
  as: { type: String, default: 'button' },
  to: String,                 // for router-link
})
const emit = defineEmits(['click'])

const tag = computed(() => props.to ? 'router-link' : props.as)
</script>

<template>
  <component
    :is="tag"
    v-bind="attrs"
    :to="props.to"
    @click="emit('click', $event)"
  >
    <slot />
  </component>
</template>
```

---

## The Hidden Pitfall — Checking `attrs.onXxx` as a Condition

This is the most common, hardest-to-debug mistake when working with `$attrs` and `defineEmits` together.

### The scenario

You write a component that conditionally shows a "clickable" style or behavior only when the parent passes a `@click` handler:

```vue
<!-- BAD — Unreliable! -->
<script setup>
import { useAttrs } from 'vue'
const attrs = useAttrs()

// "Show pointer cursor only if parent listens to click"
const isClickable = computed(() => !!attrs.onClick)
</script>

<template>
  <div :class="{ clickable: isClickable }" @click="attrs.onClick?.($event)">
    <slot />
  </div>
</template>
```

**This looks fine.** But now you add an emit declaration:

```vue
<script setup>
import { useAttrs } from 'vue'
const attrs = useAttrs()
const emit = defineEmits(['click'])          // <-- you added this

const isClickable = computed(() => !!attrs.onClick)  // ALWAYS false now!
</script>
```

**The moment you add `defineEmits(['click'])`, `onClick` disappears from `$attrs`.**  
`attrs.onClick` is now always `undefined`, even when the parent passes `@click="..."`.  
`isClickable` is permanently `false`. Your conditional logic silently breaks.

---

### Why does this happen?

Vue's rule: **any event declared in `defineEmits` is removed from `$attrs`**.

| State | Parent passes `@click` | `attrs.onClick` | `defineEmits` declares `click` |
|---|---|---|---|
| No `defineEmits` | yes | ✅ present | — |
| With `defineEmits(['click'])` | yes | ❌ stripped | yes |

The listener still works (it fires when you call `emit('click')`), but it's no longer visible in `$attrs`.

---

### Real-world example of the bug

```vue
<!-- CardItem.vue — A card that optionally shows click styles -->
<script setup>
import { useAttrs, computed } from 'vue'

defineOptions({ inheritAttrs: false })
const attrs = useAttrs()
const emit = defineEmits(['click', 'focus'])  // <-- declared here

// BUG: attrs.onClick and attrs.onFocus are always undefined
// because they were declared in defineEmits and stripped from attrs
const isInteractive = computed(() => !!(attrs.onClick || attrs.onFocus))
</script>

<template>
  <div
    v-bind="attrs"
    :class="['card', { 'card--interactive': isInteractive }]"
    :tabindex="isInteractive ? 0 : undefined"
    @click="emit('click', $event)"
    @focus="emit('focus', $event)"
  >
    <slot />
  </div>
</template>
```

```vue
<!-- Parent -->
<CardItem @click="openDetail">...</CardItem>
<!-- Expected: card has 'card--interactive' class -->
<!-- Actual: isInteractive is false, class is missing -->
```

---

### The Fix — Use explicit props as feature flags

Instead of sniffing `$attrs`, add an explicit prop:

```vue
<!-- GOOD — Explicit and predictable -->
<script setup>
import { useAttrs } from 'vue'

defineOptions({ inheritAttrs: false })
const attrs = useAttrs()
const emit = defineEmits(['click', 'focus'])

const props = defineProps({
  interactive: {    // explicit flag — no ambiguity
    type: Boolean,
    default: false,
  }
})
</script>

<template>
  <div
    v-bind="attrs"
    :class="['card', { 'card--interactive': props.interactive }]"
    :tabindex="props.interactive ? 0 : undefined"
    @click="props.interactive && emit('click', $event)"
    @focus="props.interactive && emit('focus', $event)"
  >
    <slot />
  </div>
</template>
```

```vue
<!-- Parent — intent is crystal clear -->
<CardItem interactive @click="openDetail">...</CardItem>
```

---

### Alternative Fix — Check listeners before stripping

If you truly need to detect whether the parent passed a listener **without** an explicit prop, do NOT declare that emit in `defineEmits`. Let it stay in `$attrs` and handle it manually:

```vue
<!-- Only safe if you intentionally skip defineEmits for that event -->
<script setup>
import { useAttrs, computed } from 'vue'

defineOptions({ inheritAttrs: false })
const attrs = useAttrs()
// NOTE: 'click' is NOT in defineEmits — so onClick stays in attrs
const emit = defineEmits(['focus'])

const isClickable = computed(() => typeof attrs.onClick === 'function')
</script>

<template>
  <div
    v-bind="attrs"
    :class="['card', { 'card--interactive': isClickable }]"
  >
    <slot />
  </div>
</template>
```

> This works but has a trade-off: you lose the ability to use `emit('click')`, and your component's API is implicit. **Prefer the explicit prop approach for production code.**

---

## Summary of Rules to Follow

1. **Declare all intended events in `defineEmits`** — it documents your API and gives TypeScript type safety.

2. **Never rely on `attrs.onXxx` to detect if a listener was passed** when that same event is declared in `defineEmits`. The listener will always be absent from `$attrs`.

3. **Use `inheritAttrs: false` + `v-bind="$attrs"`** whenever your component wraps a non-root element (inputs, selects, etc.).

4. **Use explicit boolean props as feature flags** instead of sniffing `$attrs` for listeners.

5. **`useAttrs()` is reactive** — it updates when the parent re-renders, so it's safe in computed properties and watchers.

---

## Quick Reference

```vue
<script setup>
import { useAttrs, computed } from 'vue'

// Step 1 — opt out of automatic fallthrough
defineOptions({ inheritAttrs: false })

// Step 2 — declare props you consume in this component
const props = defineProps({
  label: String,
  interactive: Boolean,
})

// Step 3 — declare events you fire
const emit = defineEmits(['change', 'close'])

// Step 4 — get the rest (everything NOT in props/emits)
const attrs = useAttrs()
// attrs now contains: class, style, id, aria-*, data-*, native listeners
// it does NOT contain: label, interactive, onChange, onClose

// Step 5 — safe condition check using props, not attrs
const showLabel = computed(() => !!props.label)
</script>

<template>
  <div class="wrapper">
    <label v-if="showLabel">{{ label }}</label>
    <!-- $attrs flows to the real interactive element -->
    <input v-bind="attrs" @change="emit('change', $event.target.value)" />
    <button @click="emit('close')">X</button>
  </div>
</template>
```

---

## References

1. [Vue 3 Docs — Fallthrough Attributes](https://vuejs.org/guide/components/attrs.html)
2. [Vue 3 Docs — defineEmits](https://vuejs.org/api/sfc-script-setup.html#defineprops-defineemits)
3. [Vue 3 Docs — useAttrs](https://vuejs.org/api/composition-api-helpers.html#useattrs)
4. [Vue 3 Docs — v-model with components](https://vuejs.org/guide/components/v-model.html)
