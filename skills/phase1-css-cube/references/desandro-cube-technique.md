# Desandro CSS 3D Cube — Complete Implementation

Full step-by-step implementation based on David DeSandro's [Intro to CSS 3D Transforms: Cube](https://3dtransforms.desandro.com/cube).

## HTML Structure

```html
<div class="scene">
  <div class="cube">
    <div class="cube__face cube__face--front">front</div>
    <div class="cube__face cube__face--back">back</div>
    <div class="cube__face cube__face--right">right</div>
    <div class="cube__face cube__face--left">left</div>
    <div class="cube__face cube__face--top">top</div>
    <div class="cube__face cube__face--bottom">bottom</div>
  </div>
</div>
```

Three-tier hierarchy:
- `.scene` — establishes the 3D viewing context (perspective or isometric container)
- `.cube` — the 3D object container, holds faces in shared 3D space
- `.cube__face` (x6) — individual face elements positioned within the cube

## Step-by-Step CSS

### Step 1: Base Styles

```css
.scene {
  width: 200px;
  height: 200px;
  perspective: 600px;
}

.cube {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
}

.cube__face {
  position: absolute;
  width: 200px;
  height: 200px;
  border: 2px solid rgba(0, 0, 0, 0.1);
  font-size: 40px;
  font-weight: bold;
  color: white;
  text-align: center;
  line-height: 200px;
}
```

At this stage all 6 faces overlap — stacked in the same position.

### Step 2: Rotate Faces to Orientation

Apply rotation only — no translation yet. Each face rotates to point in its direction:

```css
.cube__face--front  { transform: rotateY(0deg); }
.cube__face--right  { transform: rotateY(90deg); }
.cube__face--back   { transform: rotateY(180deg); }
.cube__face--left   { transform: rotateY(-90deg); }
.cube__face--top    { transform: rotateX(90deg); }
.cube__face--bottom { transform: rotateX(-90deg); }
```

The faces now fan out from the centre like an open cardboard box, but they all still intersect at the centre.

**Side faces** (front, back, left, right) rotate around the **Y axis**.
**Top and bottom** faces rotate around the **X axis**.

### Step 3: Translate Faces Outward

Push each face outward from the centre by half the cube's size (100px for a 200px cube). The `translateZ` happens **after** the rotation, so it pushes along the face's local Z axis:

```css
.cube__face--front  { transform: rotateY(0deg) translateZ(100px); }
.cube__face--right  { transform: rotateY(90deg) translateZ(100px); }
.cube__face--back   { transform: rotateY(180deg) translateZ(100px); }
.cube__face--left   { transform: rotateY(-90deg) translateZ(100px); }
.cube__face--top    { transform: rotateX(90deg) translateZ(100px); }
.cube__face--bottom { transform: rotateX(-90deg) translateZ(100px); }
```

The cube is now fully formed — 6 faces positioned correctly in 3D space.

**Why the same `translateZ(100px)` works for all faces:** After rotation, each face's local Z axis points outward from its position. `translateZ` always pushes along the element's local Z, not the global Z. This is why rotation must come first.

### Step 4: Centre the Cube (translateZ Compensation)

3D transforms affect text rendering — browsers snapshot the element then re-render with transforms applied, causing slightly fuzzy text. More importantly, the cube is visually centred on its back face, not its geometric centre.

Fix by pushing the entire cube backward:

```css
.cube {
  transform: translateZ(-100px);
}
```

This moves the cube back so the front face sits at Z=0 (the original plane) and the geometric centre aligns with the transform origin. This centering also simplifies animation pivot math.

### Step 5: Show-Face Classes (Interactive Rotation)

Rotate the entire cube to bring each face to the front. The rotation is the **inverse** of the face's own rotation, combined with the centering translateZ:

```css
.cube.show-front  { transform: translateZ(-100px) rotateY(0deg); }
.cube.show-right  { transform: translateZ(-100px) rotateY(-90deg); }
.cube.show-back   { transform: translateZ(-100px) rotateY(-180deg); }
.cube.show-left   { transform: translateZ(-100px) rotateY(90deg); }
.cube.show-top    { transform: translateZ(-100px) rotateX(-90deg); }
.cube.show-bottom { transform: translateZ(-100px) rotateX(90deg); }
```

### Step 6: Transition for Smooth Rotation

```css
.cube {
  transition: transform 1s;
}
```

Adding a CSS `transition` makes class changes animate smoothly. In the workshop, GSAP replaces CSS transitions for the rolling animation, but this demonstrates the concept.

## Using CSS Custom Properties

Make the cube size tuneable:

```css
.cube {
  --size: 200px;
  --half: calc(var(--size) / 2);
  width: var(--size);
  height: var(--size);
  transform: translateZ(calc(var(--half) * -1));
}

.cube__face {
  width: var(--size);
  height: var(--size);
}

.cube__face--front  { transform: rotateY(0deg) translateZ(var(--half)); }
.cube__face--right  { transform: rotateY(90deg) translateZ(var(--half)); }
.cube__face--back   { transform: rotateY(180deg) translateZ(var(--half)); }
.cube__face--left   { transform: rotateY(-90deg) translateZ(var(--half)); }
.cube__face--top    { transform: rotateX(90deg) translateZ(var(--half)); }
.cube__face--bottom { transform: rotateX(-90deg) translateZ(var(--half)); }
```

Change `--size` to resize the entire cube — all faces and positioning update automatically.

## Face Shading with color-mix()

Generate a cohesive palette from a single base colour:

```css
.cube {
  --color: #4fd1c5;
  --bg: #1a1a2e;
}

.cube__face--top    { background: color-mix(in srgb, var(--color) 100%, var(--bg)); }
.cube__face--front  { background: color-mix(in srgb, var(--color) 90%, var(--bg)); }
.cube__face--right  { background: color-mix(in srgb, var(--color) 80%, var(--bg)); }
.cube__face--left   { background: color-mix(in srgb, var(--color) 80%, var(--bg)); }
.cube__face--back   { background: color-mix(in srgb, var(--color) 60%, var(--bg)); }
.cube__face--bottom { background: color-mix(in srgb, var(--color) 20%, var(--bg)); }
```

Higher percentage = brighter (more base colour, less background). Top face gets full colour (brightest, catching "light"), sides are slightly darker, and faces pointing away are progressively darker.

Participants choose their own palette — these percentages are a starting point.

## Adapting for Isometric Projection

The Desandro tutorial uses `perspective: 600px` on `.scene` for a vanishing-point 3D look. The workshop requires **isometric projection** (no perspective distortion):

```css
.scene {
  width: 200px;
  height: 200px;
  /* perspective: 600px -- REMOVED for isometric */
  transform: rotateX(35.264deg) rotateZ(45deg);
  transform-style: preserve-3d;
}
```

Changes from Desandro's original:
1. **Remove `perspective`** — isometric is orthographic, parallel lines stay parallel
2. **Add isometric rotation** — `rotateX(35.264deg) rotateZ(45deg)` to the scene container
3. **Add `preserve-3d`** to `.scene` — so the cube's 3D structure passes through

The angle `35.264deg` = `atan(1 / sqrt(2))` produces true isometric foreshortening where all three visible faces appear equal.

## Key Concepts

### Transform Order

CSS transforms apply right-to-left (last function first in visual effect). For face positioning:

```css
/* Rotate to orientation, THEN translate outward */
transform: rotateY(90deg) translateZ(100px);
```

The `translateZ` executes in the **already-rotated** coordinate space, pushing the face outward along its local Z axis.

### Why 6 Divs Instead of 2 + Pseudo-Elements

The Desandro 6-div approach has several advantages for this workshop:
- **Explicit and readable** — each face is a named element, easy to debug in DevTools
- **Uniform positioning pattern** — every face uses the same rotate + translateZ formula
- **Flexible for content** — each face div can contain child elements, text, images
- **Matches the spec** — the Notion workshop spec explicitly calls for 6 child divs

### backface-visibility

Optional but useful — hides the back of faces when they rotate away from the viewer:

```css
.cube__face {
  backface-visibility: hidden;
}
```

In isometric view with opaque face shading, this is less necessary since back faces are occluded by front faces. Useful if faces have transparency or during rotation animations.
