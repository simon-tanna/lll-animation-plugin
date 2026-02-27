---
name: phase1-css-cube
description: >
  This skill should be used when the user asks to "build a CSS 3D cube", "construct cube faces",
  "position faces with translateZ", "set up preserve-3d", "apply isometric projection",
  "work on task 1.1", "work on task 1.2", "create a 6-div cube", or "add face shading".
  Covers Tasks 1.1–1.2 only: the Desandro 6-div cube construction technique with rotate +
  translateZ face positioning, and isometric projection. For Tasks 1.3–1.6 (rolling,
  directions, boundaries, cursor), use the `isometric-rolling-cube` skill.
  Replaces the deprecated css-3d-cube skill (Julia Miocene 2-div approach).
user-invocable: true
argument-hint: "[topic] e.g. faces, preserve-3d, isometric, translateZ, task 1.1"
---

# Phase 1 — CSS 3D Cube (Desandro 6-Div Technique)

Build a 3D cube from 6 explicit HTML `<div>` elements, positioned using rotate + `translateZ`. This is the primary reference for Phase 1 of the LLL Animation Workshop.

**Source:** David DeSandro — [Intro to CSS 3D Transforms: Cube](https://3dtransforms.desandro.com/cube)

## Invocation Behaviour

When invoked without a specific topic or task number, prompt the user:

> Which Phase 1 task are you working on?
>
> | Task | Description |
> |------|-------------|
> | 1.1 | Construct the 3D cube (HTML + CSS) |
> | 1.2 | Apply isometric projection |
> | 1.3 | Rolling animation with GSAP |
> | 1.4 | Random direction selection |
> | 1.5 | Window boundary enforcement |
> | 1.6 | Cursor following (optional) |
>
> Or describe what you need help with.

Once the task is identified:
- **Tasks 1.1–1.2:** Use this skill's content directly. Load `references/desandro-cube-technique.md` for full implementation details and `references/phase1-tasks.md` for exact requirements and acceptance criteria.
- **Tasks 1.3–1.6:** Load the `isometric-rolling-cube` skill for rolling technique and `references/phase1-tasks.md` for task requirements. Also load `gsap-expert` for GSAP API reference.

If the user mentions "rolling", "animation", "direction", or "boundaries" without a specific task number, delegate to the `isometric-rolling-cube` skill directly — these topics are outside this skill's scope.

When the user specifies a topic (e.g. "preserve-3d", "face positioning"), skip the prompt and address it directly using the relevant sections below.

## The 3-Container Architecture

The workshop uses three nested containers, each with exactly one transform responsibility:

```
Position container    → GSAP animates x/y translation only (never rotated)
Perspective container → static isometric rotation, never animated by GSAP
Cube container        → GSAP animates rolling rotation only (never translated)
```

**Why this matters:** GSAP overwrites the entire CSS `transform` property when it animates. If the isometric rotation and the x/y position are on the same element, GSAP's position animation destroys the isometric rotation. Separating them into three containers prevents this conflict.

In JSX:
```tsx
<div ref={positionRef} className="position-container">   {/* GSAP translates */}
  <div className="perspective-container">                {/* static isometric rotation */}
    <div ref={cubeRef} className="cube">                 {/* GSAP rotates for roll */}
      {/* face divs */}
    </div>
  </div>
</div>
```

For GSAP targeting in React (using refs), see the `react-integration` skill.

## HTML Structure

Use a 3-container hierarchy — position container, perspective container, cube, 6 face divs:

```html
<div class="position-container">    <!-- translates x/y — GSAP target for movement -->
  <div class="perspective-container">  <!-- static isometric rotation — never animated -->
    <div class="cube">              <!-- rolling rotation — GSAP target for rolling -->
      <div class="cube__face cube__face--front">front</div>
      <div class="cube__face cube__face--back">back</div>
      <div class="cube__face cube__face--right">right</div>
      <div class="cube__face cube__face--left">left</div>
      <div class="cube__face cube__face--top">top</div>
      <div class="cube__face cube__face--bottom">bottom</div>
    </div>
  </div>
</div>
```

**Roles:** `.position-container` is the GSAP target for x/y movement. `.perspective-container` holds the static isometric rotation (`rotateX(-35.264deg) rotateY(45deg)`) and is never animated. `.cube` establishes `preserve-3d` for the faces and is the GSAP target for rolling rotation.

## Face Positioning — The Core Pattern

Each face is positioned in two steps:

1. **Rotate** to its orientation (around Y for sides, around X for top/bottom)
2. **`translateZ(halfSize)`** to push outward from the cube's centre

For a 200px cube (`halfSize = 100px`):

| Face | Transform |
|------|-----------|
| Front | `rotateY(0deg) translateZ(100px)` |
| Right | `rotateY(90deg) translateZ(100px)` |
| Back | `rotateY(180deg) translateZ(100px)` |
| Left | `rotateY(-90deg) translateZ(100px)` |
| Top | `rotateX(90deg) translateZ(100px)` |
| Bottom | `rotateX(-90deg) translateZ(100px)` |

**Transform order matters.** Rotate first to orient the face, then translate along the face's now-local Z axis to push it outward.

## Essential CSS

```css
.scene {
  width: 200px;
  height: 200px;
  /* perspective: 600px -- used for standard 3D, REMOVED for isometric */
}

.cube {
  --size: 200px;
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  transform: translateZ(calc(var(--size) / -2)); /* Centre cube on its transform origin */
}

.cube__face {
  position: absolute;
  width: var(--size);
  height: var(--size);
}
```

The `translateZ(-100px)` on `.cube` pushes the entire cube backward so its visual centre sits at the transform origin. This simplifies animation pivot math later.

## `transform-style: preserve-3d`

This property is **critical** — without it on `.cube`, all faces flatten to 2D.

**Not inherited.** Must be set explicitly on every element whose children participate in 3D space.

**Silently broken by:** `overflow` (other than `visible`/`clip`), `opacity < 1`, `filter`, `clip-path`, `isolation: isolate`, `mix-blend-mode`, `contain: paint`. If faces mysteriously flatten, check ancestors for these properties.

**Creates a stacking context**, affecting `z-index` layering.

## Face Shading

Generate depth shading from a single base colour using `color-mix()`:

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

Higher percentage = brighter (more base colour). In isometric view, top + two side faces are visible, so those shading values matter most. Participants choose their own palette.

## Isometric Projection (Task 1.2)

Apply to the `.perspective-container`, not individual faces:

```css
.perspective-container {
  transform: rotateX(-35.264deg) rotateY(45deg);
  transform-style: preserve-3d;
  /* NO perspective property — isometric is orthographic */
}
```

The angle `35.264deg` equals `atan(1 / sqrt(2))` — the mathematically correct isometric angle where all three visible faces are equally foreshortened.

**Key distinction from Desandro reference:** The Desandro tutorial uses `perspective` on `.scene` for a vanishing-point 3D look. Isometric projection specifically requires **no perspective** — parallel edges must stay parallel. Remove or omit the `perspective` property.

The `.scene` container needs `preserve-3d` too, so the cube's 3D structure isn't flattened by the isometric rotation.

## Phase 1 Task Overview

| Task | Focus | Primary Skill |
|------|-------|---------------|
| 1.1 | Construct the 3D cube (HTML + CSS) | **This skill** |
| 1.2 | Apply isometric projection | **This skill** |
| 1.3 | Rolling animation with GSAP | `isometric-rolling-cube` |
| 1.4 | Random direction selection | `isometric-rolling-cube` |
| 1.5 | Window boundary enforcement | `isometric-rolling-cube` |
| 1.6 | Cursor following (optional) | `isometric-rolling-cube` |

For detailed task requirements, hints, and acceptance criteria, consult `references/phase1-tasks.md`.

For Tasks 1.3–1.6, load the `isometric-rolling-cube` skill for rolling animation technique and `gsap-expert` for GSAP API reference.

## Common Pitfalls

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Missing `preserve-3d` on `.cube` | All faces flat/overlapping | Add `transform-style: preserve-3d` |
| `overflow: hidden` on ancestor | Faces disappear or flatten | Remove or use `overflow: clip` |
| `opacity < 1` on ancestor | 3D flattens to 2D | Move opacity to non-3D wrapper |
| Using `perspective` with isometric | Edges converge, not parallel | Remove `perspective` property |
| Wrong transform order | Faces in wrong positions | Rotate first, then `translateZ` |
| Forgetting `translateZ(-half)` on cube | Cube offset from expected centre | Add centering transform to `.cube` |
| Unequal face dimensions | Rectangular box, not cube | Ensure width = height = `--size` |
| Missing `backface-visibility: hidden` | Back faces visible through front during rotation | Add `backface-visibility: hidden` to `.cube__face` (optional but useful during animation) |

## Related Skills

- `isometric-rolling-cube` — Rolling animation, pivot technique, direction logic, boundary enforcement
- `react-integration` — Using GSAP with React refs (`useRef`, `useEffect`), JSX differences from HTML
- `workshop-guide` — Task-by-task navigation, acceptance criteria, "what do I do next?"
- `gsap-expert` — GSAP timelines, easing, callbacks

## Additional Resources

### Reference Files

- **`references/desandro-cube-technique.md`** — Complete step-by-step CSS implementation with all code examples from the Desandro tutorial, including show-face classes and transition animation
- **`references/phase1-tasks.md`** — Full Phase 1 task specifications (1.1–1.6) with requirements, hints, and acceptance criteria from the workshop spec
