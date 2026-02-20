---
name: css-3d-cube
description: >
  CSS 3D cube construction using the minimal 2-div + pseudo-element technique from
  the Julia Miocene tutorial. Covers preserve-3d inheritance, face transforms, color-mix()
  shading, and isometric projection angles. Use this skill whenever the user is building
  a CSS cube, working with CSS 3D transforms, using transform-style preserve-3d,
  constructing cube faces from pseudo-elements, or applying isometric projection with CSS.
  Prefer this approach over a conventional 6-div cube structure.
user-invocable: false
---

# CSS 3D Cube — 2-Div Approach

Build a 3D cube from just 2 HTML elements by using their `::before` and `::after` pseudo-elements as extra faces. Each element contributes 3 faces (itself + 2 pseudo-elements) for a total of 6.

**Reference:** Julia Miocene — "3D Cube With CSS" (miocene.io)

## HTML Structure

```html
<div class="cube">
  <div></div>
</div>
```

That's it. Two divs, no extra markup.

## Face Distribution

Each face is a flat rectangle folded into position using rotation around one of its edges (via `transform-origin`):

| Face | Element | Rotation | Transform-Origin |
|------|---------|----------|-----------------|
| Bottom | `.cube` (root) | none (lies flat) | — |
| Right | `.cube::before` | `rotateY(90deg)` | center right |
| Back | `.cube::after` | `rotateX(-90deg)` | bottom center |
| Left | `.cube div` | `rotateY(-90deg)` | center left |
| Top | `.cube div::before` | `rotateY(-90deg)` | center right |
| Front | `.cube div::after` | `rotateX(-90deg)` | top center |

### How It Works

The root `.cube` element is the bottom face. Its two pseudo-elements fold out from its right edge (right face) and bottom edge (back face).

The inner `div` folds out from the left edge of `.cube` to become the left face. Its pseudo-elements then fold from the div's edges to become the top and front faces — but because the div is already rotated into position, these folds reach the remaining faces of the cube.

This is the key insight: the second set of 3 faces originates from an already-rotated parent, so their transforms compose with the parent's rotation to reach otherwise unreachable positions.

## CSS Custom Properties for Dimensions

Use custom properties so the cube size is easily tuneable. For a true cube, all three dimensions are equal:

```css
.cube {
  --size: 80px;
  --x: var(--size);  /* width */
  --y: var(--size);  /* height */
  --z: var(--size);  /* depth */
}
```

The Miocene approach supports rectangular solids too (different `--x`, `--y`, `--z`), but the workshop uses a cube.

## Core CSS

All faces share common positioning:

```css
.cube,
.cube div,
.cube::before,
.cube::after,
.cube div::before,
.cube div::after {
  position: absolute;
  box-sizing: border-box;
}

.cube::before,
.cube::after,
.cube div::before,
.cube div::after {
  content: '';
  display: block;
}
```

### Face Dimensions and Transforms

```css
/* Bottom face (root element, lies flat) */
.cube {
  width: var(--x);
  height: var(--z);
  transform-style: preserve-3d;
}

/* Right face — folds from right edge */
.cube::before {
  width: var(--z);
  height: var(--y);
  transform-origin: center right;
  rotate: y 90deg;
}

/* Back face — folds from bottom edge */
.cube::after {
  width: var(--x);
  height: var(--y);
  transform-origin: bottom center;
  rotate: x -90deg;
}

/* Left face — folds from left edge */
.cube div {
  width: var(--z);
  height: var(--y);
  transform-style: preserve-3d;
  transform-origin: center left;
  rotate: y -90deg;
}

/* Top face — folds from div's right edge (in rotated space) */
.cube div::before {
  width: var(--x);
  height: var(--z);
  transform-origin: center right;
  rotate: y -90deg;
}

/* Front face — folds from div's top edge (in rotated space) */
.cube div::after {
  width: var(--x);
  height: var(--y);
  transform-origin: top center;
  rotate: x -90deg;
}
```

### Note on `rotate:` vs `transform:`

The code above uses the CSS individual transform property `rotate:` (Chrome 104+, Firefox 72+, Safari 14.1+). The equivalent using the classic `transform:` property:

```css
/* These are equivalent: */
rotate: y 90deg;
transform: rotateY(90deg);
```

Both work. The individual property is cleaner when you only need rotation, and it doesn't conflict with other transforms on the same element.

---

## `transform-style: preserve-3d`

This property is critical and has three important behaviours:

### 1. Not Inherited

`preserve-3d` must be explicitly set on **every element** whose children need to participate in 3D space. In the 2-div approach, both `.cube` and `.cube div` need it — the pseudo-elements of `.cube div` must exist in 3D space to fold into the correct positions.

```css
.cube {
  transform-style: preserve-3d;  /* for ::before, ::after, and div */
}
.cube div {
  transform-style: preserve-3d;  /* for div::before and div::after */
}
```

Missing `preserve-3d` on the inner div is one of the most common bugs — the top and front faces collapse flat onto the left face instead of folding out into 3D.

### 2. Forced Flat by Certain Properties

Several CSS properties silently force `preserve-3d` back to `flat`, even if explicitly set. The most common culprits:

| Property | Triggers flat when... |
|----------|----------------------|
| `overflow` | any value other than `visible` or `clip` |
| `opacity` | any value less than `1` |
| `filter` | any value other than `none` |
| `clip-path` | any value other than `none` |
| `isolation` | set to `isolate` |
| `mix-blend-mode` | any value other than `normal` |
| `mask-image` | any value other than `none` |
| `contain` | `paint` or similar containment |

If faces mysteriously flatten, check ancestors for any of these properties.

### 3. Creates a Stacking Context

`preserve-3d` creates a new stacking context, which affects `z-index` layering with other elements on the page.

---

## Face Shading with `color-mix()`

Generate depth shading from a single base colour by mixing it with a background colour at different percentages:

```css
.cube {
  --color: #4fd1c5;       /* base cube colour (mint/teal) */
  --background: #1a1a2e;  /* page background */

  --top-color:    color-mix(in srgb, var(--color) 100%, var(--background)); /* = base color unchanged */
  --front-color:  color-mix(in srgb, var(--color) 90%, var(--background));
  --side-color:   color-mix(in srgb, var(--color) 80%, var(--background));
  --back-color:   color-mix(in srgb, var(--color) 60%, var(--background));
  --bottom-color: color-mix(in srgb, var(--color) 20%, var(--background));
}
```

Higher percentage = brighter (more of the base colour, less background). The top face gets 100% (lightest, catching the most "light"), sides get 80%, and faces pointing away get progressively darker.

Apply to each face via `background`:

```css
.cube              { background: var(--bottom-color); }
.cube::before      { background: var(--side-color); }    /* right */
.cube::after       { background: var(--back-color); }    /* back */
.cube div          { background: var(--side-color); }    /* left */
.cube div::before  { background: var(--top-color); }     /* top */
.cube div::after   { background: var(--front-color); }   /* front */
```

Participants choose their own colour palette — the percentages are a starting point. In isometric view, typically three faces are visible (top + two sides), so the top and side colours matter most.

---

## Isometric Projection

Apply to a **container** element wrapping the cube, not to individual faces:

```css
.iso-container {
  transform: rotateX(35.264deg) rotateZ(45deg);
  transform-style: preserve-3d;
}
```

The angle `35.264deg` is derived from `atan(1 / sqrt(2))` — the mathematically correct angle for true isometric projection where all three visible faces are equally foreshortened.

### Important Notes

- The container needs `preserve-3d` too, so the cube's 3D structure isn't flattened
- Do NOT use CSS `perspective` on the container — that creates vanishing-point distortion (perspective projection). Isometric projection is orthographic: parallel lines stay parallel
- The cube rolls within this container's local coordinate system (the flat plane before rotation is applied). The isometric transform rotates everything into the camera view

### Why Not Apply to Individual Faces?

The isometric rotation creates the *viewing angle*. It's a property of the camera/viewport, not the cube. Applying it to a container keeps the cube's own transforms clean for rolling animation.

---

## Common Mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Missing `preserve-3d` on inner div | Top and front faces are flat/invisible | Add `transform-style: preserve-3d` to `.cube div` |
| `overflow: hidden` on ancestor | Faces disappear or flatten | Remove overflow or use `overflow: clip` |
| `opacity < 1` on ancestor | 3D flattens to 2D | Move opacity to a wrapper that doesn't contain 3D children |
| Using CSS `perspective` | Edges converge, not parallel | Remove `perspective` — isometric is orthographic |
| Isometric rotation on cube instead of container | Rolling transforms fight with view rotation | Apply isometric to a parent container element |
| Forgetting `content: ''` on pseudo-elements | Only 3 faces visible instead of 6 | Add `content: ''` to all `::before` and `::after` rules |
| Wrong face dimensions for depth | Cube looks like a rectangular box | For a cube, `--x`, `--y`, `--z` must all be equal |

## See Also

- `isometric-rolling-cube` — Rolling animation technique (transform-origin shifting, GSAP patterns)
- `workshop-guide` — Task progression, acceptance criteria, gotcha warnings
- `gsap-expert` — GSAP animation API reference
