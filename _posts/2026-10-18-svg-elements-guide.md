---
layout: post
title: SVG Elements — What They Are, How to Use Them, and the Fill/Stroke Trap
subtitle: A practical guide to SVG, from basic shapes to the sneaky fill/stroke behavior that trips up every developer
# published: true
comment: true
tags: [svg, html, vue, frontend, web]
# permalink: /post/2026-10-18-svg-elements-guide
---

## What is SVG?

**SVG** stands for **Scalable Vector Graphics**. Unlike raster images (`.png`, `.jpg`) that store color data per pixel, SVG describes shapes using **mathematical coordinates**. This means:

- ✅ **Infinitely scalable** — looks crisp on any screen, any resolution
- ✅ **Small file size** for simple graphics
- ✅ **Styleable with CSS** — you can change colors, strokes, animations
- ✅ **Accessible** — screen readers can parse SVG content
- ✅ **Animatable** — with CSS or SMIL

SVG is an XML-based format, so it lives right inside your HTML as a tag:

```html
<svg width="100" height="100" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="40" fill="tomato" />
</svg>
```

---

## Core SVG Elements

### `<svg>` — The Container

Every SVG starts here. The key attributes:

| Attribute | What it does |
|---|---|
| `width` / `height` | Physical size on screen |
| `viewBox="minX minY width height"` | The internal coordinate system |
| `xmlns` | Required namespace when SVG is standalone |

```html
<svg width="200" height="200" viewBox="0 0 100 100">
  <!-- everything inside uses a 100x100 coordinate space
       but renders at 200x200 px on screen -->
</svg>
```

> **Tip:** `viewBox` is what makes SVGs scale correctly. The `width`/`height` control display size; `viewBox` controls the internal canvas.

---

### Basic Shape Elements

#### `<rect>` — Rectangle

```html
<rect x="10" y="10" width="80" height="50" rx="8" ry="8"
      fill="#4F46E5" stroke="#312E81" stroke-width="2" />
```

- `rx` / `ry` — rounded corners
- `fill` — background color
- `stroke` / `stroke-width` — border

---

#### `<circle>` — Circle

```html
<circle cx="50" cy="50" r="40" fill="none" stroke="#10B981" stroke-width="3" />
```

- `cx`, `cy` — center coordinates
- `r` — radius

---

#### `<ellipse>` — Ellipse

```html
<ellipse cx="50" cy="50" rx="45" ry="25" fill="#F59E0B" />
```

---

#### `<line>` — A Straight Line

```html
<line x1="10" y1="10" x2="90" y2="90" stroke="#EF4444" stroke-width="2" />
```

---

#### `<polyline>` and `<polygon>` — Multi-point Shapes

```html
<!-- open path (no fill by default looks good) -->
<polyline points="10,80 30,20 50,60 70,30 90,70"
          fill="none" stroke="#6366F1" stroke-width="2" />

<!-- closed shape (last point connects back to first) -->
<polygon points="50,10 90,90 10,90"
         fill="#F472B6" stroke="#BE185D" stroke-width="2" />
```

---

#### `<path>` — The Powerhouse

`<path>` can draw any shape using a mini command language in the `d` attribute:

| Command | Meaning |
|---|---|
| `M x,y` | Move to (start point) |
| `L x,y` | Line to |
| `H x` | Horizontal line to |
| `V y` | Vertical line to |
| `C x1,y1 x2,y2 x,y` | Cubic Bezier curve |
| `A rx ry rot large-arc sweep x,y` | Arc |
| `Z` | Close path |

```html
<!-- A simple arrow -->
<path d="M 10,50 L 70,50 L 55,35 M 70,50 L 55,65"
      fill="none" stroke="#0EA5E9" stroke-width="3"
      stroke-linecap="round" stroke-linejoin="round" />
```

Most icon libraries (Heroicons, Feather, Material Icons) export their icons as `<path>` elements.

---

### Text and Images

#### `<text>` — Render Text Inside SVG

```html
<text x="50" y="55" text-anchor="middle"
      font-family="sans-serif" font-size="12" fill="#1F2937">
  Hello SVG
</text>
```

#### `<image>` — Embed a Raster Image Inside SVG

```html
<image href="/img/avatar.png" x="10" y="10" width="80" height="80" />
```

---

### Grouping and Reuse

#### `<g>` — Group Elements

Apply transforms or styles to a whole group at once:

```html
<g fill="#6366F1" stroke="none" transform="translate(10, 10)">
  <circle cx="20" cy="20" r="15" />
  <rect x="45" y="5" width="30" height="30" rx="4" />
</g>
```

#### `<defs>` + `<use>` — Define Once, Reuse Anywhere

```html
<defs>
  <circle id="dot" r="5" />
</defs>

<use href="#dot" cx="20" cy="20" fill="red" />
<use href="#dot" cx="50" cy="20" fill="blue" />
<use href="#dot" cx="80" cy="20" fill="green" />
```

#### `<symbol>` — Reusable Icon Pattern

```html
<defs>
  <symbol id="icon-star" viewBox="0 0 24 24">
    <path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z" />
  </symbol>
</defs>

<use href="#icon-star" x="0" y="0" width="24" height="24" fill="gold" />
<use href="#icon-star" x="30" y="0" width="24" height="24" fill="silver" />
```

---

## Real-world Use Cases

| Use Case | Why SVG? |
|---|---|
| **App icons / UI icons** | Crisp at any size, styleable with CSS |
| **Logo** | Must look perfect on any display |
| **Charts and data viz** | D3.js draws charts by generating SVG dynamically |
| **Loaders / spinners** | Lightweight, animatable without JavaScript |
| **Illustrations** | Figma / Illustrator can export SVGs directly |
| **Maps** | Interactive click-regions on country shapes |
| **Signature / drawing tools** | Build using `<path>` generated on pointer events |

---

## `fill` and `stroke` — The Sneaky Trap

Here is where things get interesting. `fill` and `stroke` are the two most important visual properties in SVG. But their **behavior changes dramatically** depending on **how you embed the SVG**.

### What are fill and stroke?

```html
<circle cx="50" cy="50" r="40"
  fill="#4F46E5"
  stroke="#312E81"
  stroke-width="3"
/>
```

- `fill` — the inside color of the shape
- `stroke` — the border/outline color
- `stroke-width` — how thick the border is
- `fill="none"` — transparent inside (just the outline)
- Both accept any CSS color: hex, `rgb()`, `hsl()`, named colors, or **`currentColor`**

---

### The 3 Ways to Embed SVG and Why It Matters

#### 1. Inline SVG — Full control

```html
<body>
  <svg viewBox="0 0 24 24" width="48" height="48">
    <path d="M12 2l3.09 6.26L22 9.27..." fill="#4F46E5" />
  </svg>
</body>
```

When SVG is **inline**, it is part of the DOM. This means:

- CSS selectors work: `.icon path { fill: red; }`
- CSS variables work: `fill: var(--brand-color)`
- JavaScript can manipulate it: `document.querySelector('circle').setAttribute('fill', 'blue')`
- You can use `currentColor` to inherit text color from parent

---

#### 2. As `<img src="icon.svg">` — Sandboxed, CSS is blind

```html
<img src="/img/icons/star.svg" width="48" height="48" alt="star" />
```

When loaded via `<img>`, the SVG is treated as an **opaque image**. The browser renders it in isolation:

- External CSS **cannot** reach inside
- `fill` and `stroke` **cannot** be changed from the outside
- CSS variables from your page are **not** inherited
- Simple to use, loads with HTTP caching

**The only escape hatch:** hardcode the `fill` inside the `.svg` file itself, or use a CSS filter trick (ugly but works for simple color changes):

```css
/* Hack: convert to desired color using filter */
img.icon {
  filter: invert(1) sepia(1) saturate(5) hue-rotate(200deg);
}
```

---

#### 3. As a Vue Component — Back to inline power

In Vue, you can import an SVG as a component, which effectively renders it **inline**:

**Option A — Using `vite-svg-loader` (recommended in Vite projects):**

```js
// vite.config.js
import svgLoader from 'vite-svg-loader'

export default {
  plugins: [vue(), svgLoader()]
}
```

```vue
<script setup>
import StarIcon from '@/assets/icons/star.svg?component'
</script>

<template>
  <!-- SVG is rendered inline — full CSS control! -->
  <StarIcon class="icon" width="48" height="48" />
</template>

<style scoped>
.icon {
  /* This WORKS because SVG is inline in the DOM */
  color: #4F46E5;
}
.icon:hover {
  color: #7C3AED;
}
</style>
```

**Option B — Write a Vue wrapper component:**

```vue
<!-- StarIcon.vue -->
<template>
  <svg viewBox="0 0 24 24" :width="size" :height="size" v-bind="$attrs">
    <path
      d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"
      fill="currentColor"
    />
  </svg>
</template>

<script setup>
defineProps({ size: { default: 24 } })
</script>
```

```vue
<!-- Usage: color is controlled by CSS 'color' property -->
<StarIcon size="48" class="text-indigo-600 hover:text-purple-600" />
```

Because `fill="currentColor"` is set, the icon color follows the CSS `color` property of its parent — zero extra code needed.

---

### The `currentColor` Pattern — The Golden Rule

The most maintainable pattern for reusable SVG icons:

```html
<!-- In your SVG file / template: use currentColor -->
<svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
  <path
    stroke="currentColor"
    stroke-width="2"
    stroke-linecap="round"
    d="M12 4v16M4 12h16"
  />
</svg>
```

Then in CSS:

```css
.icon { color: #4F46E5; }           /* changes stroke/fill */
.icon:hover { color: #7C3AED; }     /* hover state */
.dark .icon { color: #A5B4FC; }     /* dark mode */
```

This is how virtually all modern icon libraries (Heroicons, Phosphor, Lucide) work.

---

### Summary: fill/stroke Behavior by Embedding Method

| Method | External CSS | CSS Variables | `currentColor` | JS Manipulation |
|---|---|---|---|---|
| **Inline `<svg>`** | Yes | Yes | Yes | Yes |
| **`<img src=".svg">`** | No | No | No | No |
| **`<object>` / `<iframe>`** | No (isolated) | No | No | No |
| **Vue/React component** | Yes | Yes | Yes | Yes |
| **CSS `background-image: url()`** | No | No | No | No |

> **Rule of thumb:** If you need to style an SVG (colors, hover, dark mode), always go **inline** — either literally inline in HTML, or as a framework component. Reserve `<img>` for decorative SVGs where the style is fixed.

---

## Practical Tips

1. **Clean up exported SVGs** — Tools like [SVGO](https://github.com/svg/svgo) remove unnecessary attributes that Figma/Illustrator add.

2. **Use `viewBox`, skip fixed `width`/`height` in the file** — Let the consumer control the size:

```html
<!-- Good: flexible -->
<svg viewBox="0 0 24 24">...</svg>

<!-- Avoid: hardcoded size in file -->
<svg width="24" height="24">...</svg>
```

3. **Add `aria-label` for meaningful icons:**

```html
<svg aria-label="Close menu" role="img" viewBox="0 0 24 24">...</svg>
```

4. **`fill="none"` + `stroke`** is the pattern for outline icons (Heroicons style). **`fill="currentColor"` + no stroke** is the pattern for solid icons.

---

## References

1. [MDN Web Docs — SVG element reference](https://developer.mozilla.org/en-US/docs/Web/SVG/Element)
2. [CSS-Tricks — Using SVG](https://css-tricks.com/using-svg/)
3. [Heroicons](https://heroicons.com/) — great example of the `currentColor` pattern
4. [vite-svg-loader](https://github.com/jpkleemans/vite-svg-loader) — load SVGs as Vue components in Vite
5. [SVGO](https://github.com/svg/svgo) — SVG optimization tool
