# Plugin Updates Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Apply 22 content fixes to the lll-animation plugin — correcting factual errors, adding missing architecture content, updating agents, and adding a new react-integration skill.

**Architecture:** Pure content plugin (markdown files only). No build system, no test runner. Verification is grep-based: confirm old strings are gone, new strings are present. Never commit — user manages all git operations.

**Tech Stack:** Markdown files in `.claude-plugin/` structure. Skill files are `SKILL.md` with YAML frontmatter. Agent files are `agents/*.md` with YAML frontmatter.

**Design doc:** `docs/plans/2026-02-27-plugin-updates-design.md`

---

## Task 1: Create `skills/react-integration/SKILL.md`

**Files:**
- Create: `skills/react-integration/SKILL.md`

**Step 1: Create the directory and file**

Write the following complete content to `skills/react-integration/SKILL.md`:

```markdown
---
name: react-integration
description: >
  Use when a participant asks how to use GSAP or Three.js inside a React
  component, how to target DOM elements with GSAP in React, how to set up
  a Three.js scene in a useEffect, or why their animation works in vanilla
  JS but not in their .tsx file.
user-invocable: true
argument-hint: "e.g. 'GSAP with useRef', 'Three.js in useEffect'"
---

# React Integration

The LLL Animation Workshop uses Vite + React TypeScript. GSAP and Three.js both work with React, but require specific integration patterns to avoid bugs.

## Why React Changes Things

React manages the DOM — your components render JSX which React turns into real DOM elements. This creates two challenges for animation libraries:

1. **Timing:** At the time a component function runs, the DOM elements it describes may not exist yet. GSAP cannot target elements that don't exist.
2. **Refs, not queries:** `document.querySelector('.cube')` is fragile in React. Use `useRef` to get a stable reference to the specific element in your component instance.
3. **Cleanup:** If a component unmounts (during hot module replacement, strict mode double-render, or navigation), any running tweens or render loops must be killed to prevent memory leaks.

The fix for all three: use `useRef` to get a stable reference to the DOM element, and `useEffect` to run animation code after the DOM is ready.

## GSAP + React (Phase 1)

### Setting Up the Ref

```tsx
import { useRef, useEffect } from 'react'
import gsap from 'gsap'

function Cube() {
  // Separate refs for the two animated containers
  const positionRef = useRef<HTMLDivElement>(null) // position container (translate)
  const cubeRef = useRef<HTMLDivElement>(null)     // cube container (roll rotation)

  useEffect(() => {
    const position = positionRef.current
    const cube = cubeRef.current
    if (!position || !cube) return

    function roll() {
      const tl = gsap.timeline({
        onComplete: () => {
          // Reset after each roll — the core technique
          gsap.set(cube, { rotationZ: 0 })
          roll()
        }
      })
      tl.to(position, { x: newX, y: newY, duration: 0.5 })
      tl.to(cube, { rotationZ: -90, duration: 0.5 }, '<')
    }

    roll()

    // Kill tweens when component unmounts (e.g. hot module replacement)
    return () => gsap.killTweensOf([position, cube])
  }, []) // Empty deps = run once after mount

  return (
    <div ref={positionRef} className="position-container">
      <div className="perspective-container">
        <div ref={cubeRef} className="cube">
          {/* 6 face divs */}
        </div>
      </div>
    </div>
  )
}
```

### Key Points

- Attach `ref={positionRef}` to the **position container** (outermost div) — GSAP animates its x/y for movement.
- Attach `ref={cubeRef}` to the **cube container** (innermost) — GSAP animates its rotation for rolling.
- The **perspective container** (middle div) holds the static isometric rotation and is never animated — no ref needed.
- `useEffect` with `[]` runs once after mount. This is the correct place for animation setup.
- Return a cleanup function from `useEffect` to kill tweens on unmount.

## Three.js + React (Phase 2)

### Canvas Ref + useEffect for Scene Setup

```tsx
import { useRef, useEffect } from 'react'
import * as THREE from 'three'
import gsap from 'gsap'

function Cube3D() {
  const canvasRef = useRef<HTMLCanvasElement>(null)

  useEffect(() => {
    const canvas = canvasRef.current
    if (!canvas) return

    // The boilerplate already provides scene, camera, renderer, and render loop.
    // Add your mesh to the existing scene reference from the boilerplate.
    const geometry = new THREE.BoxGeometry(1, 1, 1)
    const material = new THREE.MeshBasicMaterial({ color: 0x4fd1c5 })
    const cube = new THREE.Mesh(geometry, material)
    // scene.add(cube) — use the boilerplate's scene

    // Cleanup on unmount — prevents GPU memory leaks during HMR
    return () => {
      geometry.dispose()
      material.dispose()
      gsap.killTweensOf(cube.rotation)
      gsap.killTweensOf(cube.position)
    }
  }, [])

  return <canvas ref={canvasRef} />
}
```

> **Workshop note:** The starter repo's `Cube3D.tsx` boilerplate already initialises the scene, camera, renderer, and render loop. You don't need to create these — just add your mesh to the existing scene and animate it with GSAP.

### Key Points

- Use `useRef<HTMLCanvasElement>` for the canvas element and pass it to `WebGLRenderer`.
- All Three.js setup goes inside `useEffect` — the canvas must exist in the DOM first.
- Don't create a duplicate render loop — the boilerplate's RAF loop is already running.
- Clean up `geometry.dispose()` and `material.dispose()` on unmount to free GPU memory.
- Kill GSAP tweens in the cleanup to prevent errors when the component unmounts during a tween.

## JSX Differences from HTML

| HTML attribute | JSX equivalent |
|----------------|----------------|
| `class="cube"` | `className="cube"` |
| `style="transform-style: preserve-3d"` | `style={{ transformStyle: 'preserve-3d' }}` |
| `onclick="fn()"` | `onClick={fn}` |
| `<br>` | `<br />` (must be self-closing) |

Inline styles in JSX take a JavaScript object with camelCase property names. For complex animation, prefer external CSS classes — GSAP targets DOM elements directly and does not interact with React's render cycle.

## See Also

- `phase1-css-cube` — CSS cube construction and isometric projection
- `isometric-rolling-cube` — Rolling animation technique and direction logic
- `gsap-expert` — Full GSAP API reference
- `threejs-fundamentals` — Three.js scene setup and Object3D hierarchy
```

**Step 2: Verify the file exists with correct frontmatter**

Run: `grep -n "name: react-integration" skills/react-integration/SKILL.md`
Expected output: `2:name: react-integration`

---

## Task 2: Fix isometric rotation formula in `skills/phase1-css-cube/SKILL.md`

**Files:**
- Modify: `skills/phase1-css-cube/SKILL.md` (Isometric Projection section, ~line 147)

**Step 1: Find the current formula**

Run: `grep -n "rotateX(35.264deg)" skills/phase1-css-cube/SKILL.md`
Expected: line 147 with `transform: rotateX(35.264deg) rotateZ(45deg);`

**Step 2: Replace the formula**

In `skills/phase1-css-cube/SKILL.md`, replace:
```css
  transform: rotateX(35.264deg) rotateZ(45deg);
```
with:
```css
  transform: rotateX(-35.264deg) rotateY(45deg);
```

**Step 3: Verify**

Run: `grep -n "rotateX" skills/phase1-css-cube/SKILL.md`
Expected: line shows `rotateX(-35.264deg) rotateY(45deg)` — negative X, Y axis (not Z).

---

## Task 3: Fix rotation formula in `skills/phase1-css-cube/references/desandro-cube-technique.md`

**Files:**
- Modify: `skills/phase1-css-cube/references/desandro-cube-technique.md` (Isometric section, line 189)

**Step 1: Find both occurrences**

Run: `grep -n "rotateX(35.264deg)" skills/phase1-css-cube/references/desandro-cube-technique.md`
Expected: line 189 in the Adapting for Isometric Projection section.

**Step 2: Replace the formula in the code block**

Replace:
```css
  transform: rotateX(35.264deg) rotateZ(45deg);
```
with:
```css
  transform: rotateX(-35.264deg) rotateY(45deg);
```

**Step 3: Update the description text**

The text below the code block currently says:
```
2. **Add isometric rotation** — `rotateX(35.264deg) rotateZ(45deg)` to the scene container
```
Replace with:
```
2. **Add isometric rotation** — `rotateX(-35.264deg) rotateY(45deg)` to the perspective container
```

**Step 4: Update the 3-container HTML structure in this file**

The file currently shows a 2-tier `scene > cube` structure throughout. Replace the HTML Structure section (lines 7–23) with the 3-container version:

```html
<div class="position-container">   <!-- translates x/y only -->
  <div class="perspective-container">  <!-- static isometric rotation only -->
    <div class="cube">               <!-- rolling rotation only -->
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

Update the hierarchy description below it to:
```
Four-tier hierarchy (workshop adaptation):
- `.position-container` — GSAP animates x/y translation only; never rotated
- `.perspective-container` — static isometric rotation `rotateX(-35.264deg) rotateY(45deg)`; never animated
- `.cube` — GSAP animates rolling rotation only; never translated
- `.cube__face` (x6) — individual faces positioned within the cube
```

**Step 5: Update the Adapting for Isometric section CSS**

Replace the entire Adapting for Isometric Projection code block (lines 184–192) with:

```css
/* Position container — GSAP animates x/y translation */
.position-container {
  position: absolute; /* or fixed, depending on layout */
}

/* Perspective container — static isometric rotation, never animated */
.perspective-container {
  transform: rotateX(-35.264deg) rotateY(45deg);
  transform-style: preserve-3d;
  /* NO perspective property — isometric is orthographic */
}

/* Cube — GSAP animates rolling rotation only */
.cube {
  --size: 200px;
  --half: calc(var(--size) / 2);
  width: var(--size);
  height: var(--size);
  position: relative;
  transform-style: preserve-3d;
  transform: translateZ(calc(var(--half) * -1)); /* centre on transform origin */
}
```

Update the "Changes from Desandro's original" list to:
```
Changes from Desandro's original:
1. **3 containers instead of 2** — position, perspective, and cube containers each own one transform responsibility. GSAP cannot animate position and rotation on the same element (it overwrites the full `transform` property).
2. **Remove `perspective`** — isometric is orthographic, parallel lines stay parallel
3. **Isometric rotation on perspective container** — `rotateX(-35.264deg) rotateY(45deg)`
4. **`preserve-3d` on both perspective container and cube** — so the 3D structure passes through each level
```

**Step 6: Verify**

Run: `grep -n "perspective-container\|position-container\|rotateX(-35" skills/phase1-css-cube/references/desandro-cube-technique.md`
Expected: multiple matches for all three strings.

---

## Task 4: Fix rotation formula in `skills/workshop-guide/SKILL.md`

**Files:**
- Modify: `skills/workshop-guide/SKILL.md` (Phase Transition table, line 242)

**Step 1: Find the current formula**

Run: `grep -n "rotateX(35.264deg)" skills/workshop-guide/SKILL.md`
Expected: line 242 in the Phase Transition Guide table.

**Step 2: Replace**

Replace:
```
| Isometric view | Container rotation (`rotateX(35.264deg) rotateZ(45deg)`) | `OrthographicCamera` positioned at isometric angle |
```
with:
```
| Isometric view | 3 containers: position (x/y), perspective (`rotateX(-35.264deg) rotateY(45deg)`), cube (roll). Two CSS containers collapse into one `THREE.Group` in Three.js. | `OrthographicCamera` + `THREE.Group` with static isometric rotation |
```

**Step 3: Verify**

Run: `grep -n "rotateX(-35" skills/workshop-guide/SKILL.md`
Expected: line in the Phase Transition table.

---

## Task 5: Fix tech stack label in `skills/workshop-guide/SKILL.md`

**Files:**
- Modify: `skills/workshop-guide/SKILL.md` (lines 18–20)

**Step 1: Find current text**

Run: `grep -n "Vite + TypeScript" skills/workshop-guide/SKILL.md`

**Step 2: Replace line 18**

Replace:
```
**Tech stack:** Vite + TypeScript, HTML, CSS, vanilla JS/TS, GSAP, Three.js
```
with:
```
**Tech stack:** Vite + React TypeScript, GSAP, Three.js
```

**Step 3: Replace line 20**

Replace:
```
Participants receive a starter repo with a working Vite/TS app, an isometric diamond grid background, and all dependencies pre-installed.
```
with:
```
Participants receive a starter repo with a working Vite/React TypeScript app, an isometric diamond grid background, and all dependencies pre-installed (GSAP, Three.js).
```

**Step 4: Verify**

Run: `grep -n "React TypeScript" skills/workshop-guide/SKILL.md`
Expected: 2 matches on lines 18 and 20.

---

## Task 6: Fix CSS direction naming in `skills/isometric-rolling-cube/SKILL.md`

**Files:**
- Modify: `skills/isometric-rolling-cube/SKILL.md` (CSS direction table, lines 73–77)

**Step 1: Locate the CSS direction table**

Run: `grep -n "Right (+X)\|Left (-X)\|Forward\|Backward" skills/isometric-rolling-cube/SKILL.md`
Expected: 4 rows in the CSS direction table around lines 74–77.

**Step 2: Replace the 4 direction rows**

Replace the entire CSS direction table body (the 4 data rows):
```
| Right (+X)           | bottom-right edge | rotateZ(-90deg) | translateX(+S) |
| Left (-X)            | bottom-left edge  | rotateZ(+90deg) | translateX(-S) |
| Forward (-Y screen)  | bottom-front edge | rotateX(+90deg) | translateY(-S) |
| Backward (+Y screen) | bottom-back edge  | rotateX(-90deg) | translateY(+S) |
```
with:
```
| Top-Right (+X)    | bottom-right edge | rotateZ(-90deg) | translateX(+S) |
| Bottom-Left (-X)  | bottom-left edge  | rotateZ(+90deg) | translateX(-S) |
| Bottom-Right (-Y) | bottom-front edge | rotateX(+90deg) | translateY(-S) |
| Top-Left (+Y)     | bottom-back edge  | rotateX(-90deg) | translateY(+S) |
```

Also update the comment on line 70 that says `rotateX(35.264deg) rotateZ(45deg)` to `rotateX(-35.264deg) rotateY(45deg)`.

**Step 3: Verify**

Run: `grep -n "Top-Right\|Bottom-Left\|Bottom-Right\|Top-Left" skills/isometric-rolling-cube/SKILL.md`
Expected: 4+ matches (the table rows plus the Three.js table which already uses correct names).

---

## Task 7: Fix render loop ranking in `agents/phase-transition-helper.md`

**Files:**
- Modify: `agents/phase-transition-helper.md` (Render Loop section, lines 79–83; Build Order step 3, line 139)

**Step 1: Fix the render loop option ranking**

Replace the three options in the Render Loop section:
```
1. `gsap.ticker.add(() => renderer.render(scene, camera))` — synchronises rendering with GSAP's update cycle and avoids redundant frames (recommended)
2. `requestAnimationFrame` loop (simple and reliable)
3. `onUpdate` callback per tween (renders only during active animation — scene freezes between rolls)
```
with:
```
1. `requestAnimationFrame` loop — `requestAnimationFrame` that calls `renderer.render(scene, camera)` every frame. Simple, always works. **(spec preferred)**
2. `gsap.ticker.add(() => renderer.render(scene, camera))` — synchronises rendering with GSAP's update cycle, avoids redundant frames
3. `onUpdate` callback per tween — renders only during active animation; scene freezes between rolls (acceptable but least robust)
```

**Step 2: Fix Build Order step 3**

Replace:
```
3. **Set up the render loop** — get something rendering before trying animation
```
with:
```
3. **Verify the render loop** — the boilerplate provides a RAF loop already; confirm it is running. Do not create a duplicate loop. For React integration, see the `react-integration` skill.
```

**Step 3: Verify**

Run: `grep -n "spec preferred\|Verify the render loop" agents/phase-transition-helper.md`
Expected: both strings present.

---

## Task 8: Add 3-container architecture section to `skills/phase1-css-cube/SKILL.md`

**Files:**
- Modify: `skills/phase1-css-cube/SKILL.md`

**Step 1: Add the new section before "HTML Structure"**

Insert a new section between the "Invocation Behaviour" section and "## HTML Structure" (before line 46):

```markdown
## The 3-Container Architecture

The workshop uses three nested containers, each with exactly one transform responsibility:

```
Position container   → GSAP animates x/y translation only (never rotated)
Perspective container → static isometric rotation, never animated by GSAP
Modeled cube         → GSAP animates rolling rotation only (never translated)
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
```

**Step 2: Update the HTML Structure section**

Replace the current HTML Structure block (lines 46–63) with the 3-container version:

```markdown
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
```

**Step 3: Update the Isometric Projection section (line 147)**

(Already handled in Task 2 — the formula is corrected there.)

Also update the surrounding prose. Replace:
```
Apply to the `.scene` container, not individual faces:
```
with:
```
Apply to the `.perspective-container`, not individual faces:
```

**Step 4: Verify**

Run: `grep -n "position-container\|perspective-container\|3-Container" skills/phase1-css-cube/SKILL.md`
Expected: multiple matches including the new section heading and HTML example.

---

## Task 9: Add 3-container summary to `skills/phase1-css-cube/references/phase1-tasks.md`

**Files:**
- Modify: `skills/phase1-css-cube/references/phase1-tasks.md` (Task 1.1 Hints section)

**Step 1: Find the hints block**

Run: `grep -n "Hints:" skills/phase1-css-cube/references/phase1-tasks.md`
Expected: line ~19.

**Step 2: Add the 3-container hint**

After the last existing hint bullet in Task 1.1 (the `color-mix()` line), add:

```markdown
- **3-container design:** Use three nested containers, each with one transform responsibility — a position container (x/y movement), a perspective container (static isometric rotation), and the cube itself (rolling rotation). This separation is essential: GSAP overwrites the full `transform` property when animating, so mixing position and rotation on one element breaks the animation.
```

**Step 3: Verify**

Run: `grep -n "3-container\|perspective container" skills/phase1-css-cube/references/phase1-tasks.md`
Expected: the new hint line.

---

## Task 10: Add 3-container summary and starter repo section to `skills/workshop-guide/SKILL.md`

**Files:**
- Modify: `skills/workshop-guide/SKILL.md`

**Step 1: Add "Starter Repository" section**

Insert a new section after line 20 (after the "End result" line and before "## Workshop Structure"):

```markdown
## Starter Repository

Pre-configured: Vite + React TypeScript, isometric diamond grid background, `OrthographicCamera` and RAF render loop (Phase 2 boilerplate), all dependencies installed (GSAP, Three.js).

**Do not modify:** the camera, the grid, or the boilerplate render loop.

**Component files:** `Cube.tsx` (Phase 1 — write your CSS cube here), `Cube3D.tsx` (Phase 2 — write your Three.js cube here).
```

**Step 2: Add 3-container architecture to Task 1.1 key concepts**

Find the Task 1.1 Key concepts line (currently: "`transform-style: preserve-3d` (not inherited..."). Add a new bullet at the start:

```markdown
**Key concepts:** Use three nested containers — position container (GSAP animates x/y only), perspective container (static isometric rotation, never animated), cube (GSAP animates rolling rotation only). GSAP overwrites the full `transform` property — mixing position and rotation on one element breaks both. `transform-style: preserve-3d` (not inherited — set on every element whose children participate in 3D space), rotate + `translateZ(halfSize)` face positioning pattern, `color-mix()` for face shading.
```

**Step 3: Add yoyo bullet to Task 1.3 key concepts**

Find the Task 1.3 key concepts block. After the `"power2.inOut"` easing sentence, add:

```markdown
Optional landing jump: `gsap.to(positionContainer, { y: -jumpHeight, duration: rollDuration / 2, yoyo: true, repeat: 1, ease: "power2.out" })` — `yoyo: true` with `repeat: 1` plays the tween forward then automatically reverses, producing a natural up-then-down bounce on each landing.
```

**Step 4: Add pivot.rotation clarification to Task 2.3 key concepts**

Find the Task 2.3 key concepts block. After the existing GSAP syntax note, add:

```markdown
Note: when using the pivot technique, GSAP targets `pivot.rotation`, not `mesh.rotation` directly. The spec's `gsap.to(mesh.rotation, ...)` example illustrates GSAP's sub-object targeting syntax — in the pivot implementation, `pivot` is the actual animated target.
```

**Step 5: Add transform-origin to the Gotcha table**

Find the Gotcha quick reference table. Add a new row after the "Rotation axis confusion" row:

```markdown
| `transform-origin` misconfiguration | 1.3, 1.4 | CSS only. A 2D `transform-origin` (two values) places the pivot at Z=0, wrong for 3 of 4 directions. Must be a 3D value (three components) targeting the specific bottom edge in 3D space. Distinct from rotation axis confusion — axis can be correct while `transform-origin` is still wrong. |
```

**Step 6: Verify**

Run: `grep -n "Starter Repository\|yoyo\|pivot.rotation\|transform-origin misconfiguration" skills/workshop-guide/SKILL.md`
Expected: 4 matches.

---

## Task 11: Add transform-origin and preserve-3d checks to `agents/animation-reviewer.md`

**Files:**
- Modify: `agents/animation-reviewer.md`

**Step 1: Insert new CSS-only checks before Check 1**

The current Check 1 starts at line 20 (`### 1. Pivot Point Location`). Insert two new checks before it:

```markdown
### CSS-Only Pre-Checks (Phase 1 code only — skip for Three.js)

#### CSS Check A: `transform-origin` Is a 3D Value

**What to check:** CSS rolling depends entirely on `transform-origin` pointing to the correct bottom edge in 3D space.

- Is `transform-origin` set with **three values** (x, y, z), not two? A 2D `transform-origin` places the pivot at Z=0, which is wrong for 3 of the 4 roll directions.
- Does the value change per roll direction? A single fixed `transform-origin` value cannot be correct for all 4 directions — each direction has a different bottom edge.
- Does the Z component place the origin at the cube's surface depth, not at Z=0?

Example of correct pattern: each direction entry in the data table has its own `transform-origin` value, e.g.:
- Top-Right: `transform-origin: S S/2 0`
- Bottom-Left: `transform-origin: 0 S/2 0`
- Bottom-Right: `transform-origin: S/2 S/2 -S/2`
- Top-Left: `transform-origin: S/2 S/2 S/2`

**Fail symptom:** Cube rotates around a wrong edge — may appear to "flip" or tip in the wrong direction for some roll directions but not others.

#### CSS Check B: `preserve-3d` Not Silently Overridden

**What to check:** `transform-style: preserve-3d` is silently disabled by certain CSS properties on ancestor elements, causing faces to flatten during rolls.

Look at every ancestor of the rolling cube container. Check for:
- `overflow` set to anything other than `visible` or `clip`
- `opacity < 1`
- `filter` (including `drop-shadow`)
- `clip-path`
- `isolation: isolate`
- `mix-blend-mode`

Any of these on an ancestor of the cube will disable `preserve-3d` with no error message.

**Fail symptom:** Cube faces were 3D and now appear flat during animation, especially when a visual effect (shadow, blur) was recently added to a parent element.

---
```

**Step 2: Update the summary line**

Replace:
```
- Checks passed: N/8
```
with:
```
- Checks passed: N/10 (CSS: includes 2 CSS-only pre-checks A and B; Three.js: 8 checks)
```

**Step 3: Verify**

Run: `grep -n "CSS Check A\|CSS Check B\|N/10" agents/animation-reviewer.md`
Expected: 3 matches.

---

## Task 12: Update `animation-reviewer.md` trigger timing

**Files:**
- Modify: `agents/animation-reviewer.md` (frontmatter description, lines 4–8; body line 14)

**Step 1: Update frontmatter description**

Replace the frontmatter description block:
```yaml
  Use proactively after completing Tasks 1.3–1.5 (CSS rolling, random direction, boundary)
  or Tasks 2.3–2.4 (Three.js rolling, boundary) to check for common failure modes including
  pivot points, reparenting, rotation axes, cumulative rotation tracking, grid snapping,
  render loops, and boundary handling.
```
with:
```yaml
  Use proactively after completing Task 1.3 (CSS rolling in one direction) or Task 2.3
  (Three.js rolling in one direction) — catch pivot and reset bugs before building direction
  selection on top of a broken foundation. Optionally run again after Task 1.5/2.4
  (boundary enforcement) to check boundary handling. Checks pivot points, transform-origin,
  preserve-3d, reparenting, rotation axes, cumulative rotation, grid snapping, render loops,
  and boundary handling.
```

**Step 2: Update the body trigger line**

Replace:
```
Run this review after a participant completes **Tasks 1.3–1.5** (CSS rolling, random direction, boundary enforcement) or **Tasks 2.3–2.4** (Three.js rolling, boundary enforcement).
```
with:
```
Run this review after a participant completes **Task 1.3** (CSS — rolling works in one direction) or **Task 2.3** (Three.js — rolling works in one direction). Run again optionally after Task 1.5/2.4 to verify boundary handling. Catching pivot and reset bugs early — before building direction selection (Task 1.4/2.3+) on top of a broken foundation — saves significant debugging time.
```

**Step 3: Verify**

Run: `grep -n "Task 1.3\|catch pivot" agents/animation-reviewer.md`
Expected: matches in both frontmatter and body.

---

## Task 13: Add remaining updates to `agents/phase-transition-helper.md`

**Files:**
- Modify: `agents/phase-transition-helper.md`

**Step 1: Add THREE.Group concept mapping row**

Find the Isometric View section in the concept mapping. After the "What to tell them:" block in that section, add a new sub-section:

```markdown
**THREE.Group as unified container:** In Three.js, the two CSS containers (position container + perspective container) collapse into a single `THREE.Group`. The Group holds both the static isometric tilt and manages position — Three.js does not need two separate parent objects for this:

```js
const isoGroup = new THREE.Group()
// Apply static isometric rotation once (equivalent to CSS perspective container)
isoGroup.rotation.x = -35.264 * (Math.PI / 180)
isoGroup.rotation.y = 45 * (Math.PI / 180)
scene.add(isoGroup)
// Add cube to the group — its position is now managed relative to the group
isoGroup.add(cube)
```

Moving the cube across the grid: update `isoGroup.position.x` and `isoGroup.position.z` (world-space), not the cube's local position within the group.
```

**Step 2: Add `backface-visibility` mapping to Cube Construction section**

After the "What to tell them:" block in the Cube Construction section, add:

```markdown
**`backface-visibility`:** If their Phase 1 code uses `backface-visibility: hidden` on face divs (hides faces rotating away from the viewer), the Three.js equivalent is `material.side = THREE.FrontSide` — which is the default. They likely don't need to add anything. `THREE.DoubleSide` would be the equivalent of NOT using `backface-visibility: hidden`.
```

**Step 3: Add `updateMatrixWorld()` note to Pivot Point section**

Find the 5-step pivot cycle list. After step 2 (`pivot.attach(cube)`), add a note:

```markdown
> **Important:** If the pivot was just created and positioned in the same frame as `attach()`, call `pivot.updateMatrixWorld()` after positioning and before `attach()`. Stale world matrices cause the same visual jump as using `add()` instead of `attach()`.
```

**Step 4: Add closing handoff after Build Order**

After the last item in the Build Order Recommendation list (item 9, "Test cumulative rotation"), add:

```markdown
Once a single roll works in Three.js (step 5 above), run the **`animation-reviewer`** agent to catch common failure modes before building direction selection. It is much easier to fix pivot and reset bugs now than after the full rolling system is wired up.
```

**Step 5: Verify**

Run: `grep -n "THREE.Group\|backface-visibility\|updateMatrixWorld\|animation-reviewer" agents/phase-transition-helper.md`
Expected: 4+ matches.

---

## Task 14: Create `skills/isometric-rolling-cube/references/` directory and files

**Files:**
- Create: `skills/isometric-rolling-cube/references/direction-data.md`
- Create: `skills/isometric-rolling-cube/references/boundary-algorithm.md`
- Modify: `skills/isometric-rolling-cube/SKILL.md` (remove extracted content, add links)

**Step 1: Create `direction-data.md`**

Create `skills/isometric-rolling-cube/references/direction-data.md` with this content:

```markdown
# Direction Data Reference

Full direction tables for both CSS (Phase 1) and Three.js (Phase 2) implementations.

## Three.js Direction Data (Y-up, XZ ground plane)

For a cube centred at `(gx, S/2, gz)`:

| Direction   | Screen    | Translation | Pivot Offset from Centre | Rotation Axis | Angle |
| ----------- | --------- | ----------- | ------------------------ | ------------- | ----- |
| Top-Right   | +X        | (S, 0, 0)   | (+S/2, -S/2, 0)          | (0, 0, -1)    | PI/2  |
| Bottom-Left | -X        | (-S, 0, 0)  | (-S/2, -S/2, 0)          | (0, 0, +1)    | PI/2  |
| Bottom-Right| +Z        | (0, 0, S)   | (0, -S/2, +S/2)          | (+1, 0, 0)    | PI/2  |
| Top-Left    | -Z        | (0, 0, -S)  | (0, -S/2, -S/2)          | (-1, 0, 0)    | PI/2  |

The rotation axis can be derived: `cross(direction, downVector)` where `downVector = (0, -1, 0)`.

The pivot offset is always: half a cube-width toward the roll direction on the ground axis, plus half a cube-width downward (to the bottom edge).

## CSS Direction Data (pre-isometric flat plane)

The isometric rotation (`rotateX(-35.264deg) rotateY(45deg)`) is applied to the perspective container. The cube rolls in the container's local coordinate system (flat plane before rotation):

| Direction    | `transform-origin`  | Rotation        | Translation    |
| ------------ | ------------------- | --------------- | -------------- |
| Top-Right    | bottom-right edge   | rotateZ(-90deg) | translateX(+S) |
| Bottom-Left  | bottom-left edge    | rotateZ(+90deg) | translateX(-S) |
| Bottom-Right | bottom-front edge   | rotateX(+90deg) | translateY(-S) |
| Top-Left     | bottom-back edge    | rotateX(-90deg) | translateY(+S) |

### CSS `transform-origin` Values (3D)

For a cube of side `S` centred at its own origin, the bottom edge origins in 3D space:

| Direction    | `transform-origin`       |
| ------------ | ------------------------ |
| Top-Right    | `S S/2 0`                |
| Bottom-Left  | `0 S/2 0`                |
| Bottom-Right | `S/2 S/2 calc(S / -2)`   |
| Top-Left     | `S/2 S/2 calc(S / 2)`    |

The exact values depend on the cube construction approach (specifically how faces are positioned and whether a `translateZ` centering transform is applied to the cube container). The principle is always the same: origin goes to the bottom edge in the roll direction, in 3D space (three-component value required).
```

**Step 2: Create `boundary-algorithm.md`**

Create `skills/isometric-rolling-cube/references/boundary-algorithm.md` with this content:

```markdown
# Boundary and Cursor Algorithm Reference

## Direction Filtering Algorithm

Before each roll, check which of the 4 directions would keep the cube in bounds. The steps:

1. Calculate the candidate position for each direction: `(gridX + deltaX, gridY + deltaY)`
2. Check each candidate against the grid bounds
3. Filter the directions list to only valid options
4. Pick randomly from the filtered list

Filtering is essential — re-rolling a single rejected direction is not enough, because in corners multiple directions may be invalid simultaneously.

```js
function getValidDirections(gridX, gridY, bounds) {
  return DIRECTIONS.filter(dir => {
    const nx = gridX + dir.deltaX
    const ny = gridY + dir.deltaY
    return nx >= bounds.minX && nx <= bounds.maxX
        && ny >= bounds.minY && ny <= bounds.maxY
  })
}

function rollNext() {
  const valid = getValidDirections(gridX, gridY, bounds)
  if (valid.length === 0) return // should not happen if bounds are set correctly
  const dir = valid[Math.floor(Math.random() * valid.length)]
  // ... animate roll in dir
}
```

## CSS (Phase 1) — Viewport Pixel Bounds

Track the cube's logical grid position `(gridX, gridY)` and convert to pixel coordinates to check against `window.innerWidth` / `window.innerHeight`. The isometric rotation means screen-space movement is diagonal — a single grid step moves the cube diagonally on screen, so bounds checks must account for both the X and Y pixel displacement per step.

## Three.js (Phase 2) — Grid Coordinate Bounds

The simplest approach: define bounds in grid coordinates and check the logical position directly, avoiding camera-to-world projection math entirely.

The boilerplate's `OrthographicCamera` frustum properties (`left`, `right`, `top`, `bottom`) are in **camera-space**, not world-space — for the rotated isometric camera, these don't map directly to ground-plane boundaries.

Alternative: unproject the frustum corners onto the ground plane for precise world-space bounds, but grid-coordinate bounds are simpler and sufficient for the workshop.

## Cursor Following — Direction Bias Algorithm

1. Calculate the vector from the cube's current position to the cursor position
2. For each of the 4 isometric directions, compute the dot product between the cursor vector and the direction's movement vector
3. Assign higher probability to directions with larger positive dot products
4. Use weighted random selection — directions aligned with the cursor get higher weight, but all valid directions remain possible

A simple weighting scheme: give cursor-aligned directions 3x or 4x the weight of non-aligned directions. This produces a visible trend toward the cursor while preserving organic movement.

### Fallback to Random

When the cursor is inactive or outside the viewport, revert to fully random direction selection (all valid directions weighted equally).

### CSS (Phase 1) — Screen-Space Vector Math

Both the cube position and cursor position are in screen-space pixels. Calculate the direction vector directly from pixel coordinates and compare against the 4 isometric screen-space direction vectors.

### Three.js (Phase 2) — Screen-to-World Conversion

Convert the screen-space cursor position to world-space coordinates before computing the direction vector. With an `OrthographicCamera`, the conversion is simpler than with a perspective camera — use `Vector3.unproject()` or `Raycaster` to project the cursor onto the ground plane (Y=0). The direction-biasing logic from Phase 1 carries over directly once the cursor position is in world-space.
```

**Step 3: Update `skills/isometric-rolling-cube/SKILL.md` body**

Remove the full CSS direction table (lines ~69–86) and replace with a brief summary + reference link:

```markdown
### Direction Data (CSS — pre-isometric flat plane)

Full `transform-origin` values and translation vectors for all 4 directions: see `references/direction-data.md`.

Key principle: `transform-origin` must be a **3D value (three components)** targeting the specific bottom edge for each direction. A 2D value places the pivot at Z=0, which is wrong for 3 of the 4 directions.
```

Remove the full Boundary Enforcement section (lines ~208–230) and Cursor Following section (lines ~234–257) and replace with:

```markdown
## Boundary Enforcement

Before each roll, filter valid directions against grid bounds. See `references/boundary-algorithm.md` for the full direction filtering algorithm, CSS viewport-pixel bounds approach, and Three.js grid-coordinate bounds approach.

## Cursor Following

Weighted direction bias toward cursor position. See `references/boundary-algorithm.md` for the full bias algorithm, CSS screen-space approach, and Three.js `Vector3.unproject()` conversion for world-space cursor coordinates.
```

**Step 4: Add yoyo jump and updateMatrixWorld note**

In the GSAP Animation Patterns section, after the Easing sub-section, add:

```markdown
### The Landing Jump (yoyo)

Add a natural bounce at each landing with `yoyo: true`:

```js
// Runs alongside the roll animation — plays up then automatically reverses
gsap.to(positionContainer, {
  y: -jumpHeight,
  duration: rollDuration / 2,
  yoyo: true,
  repeat: 1,
  ease: "power2.out"
})
```

`yoyo: true` with `repeat: 1` plays the tween forward then automatically reverses — the position container rises then falls back, producing the characteristic up-then-down bounce at each landing.
```

In the Phase 2 pivot technique, Step 2 (`Attach cube to pivot`), add the note:

```markdown
> **`updateMatrixWorld()` caveat:** If the pivot was created and positioned in the same frame as `attach()`, call `pivot.updateMatrixWorld()` after setting the position and before calling `pivot.attach(cube)`. Without this, Three.js may use a stale world matrix and the cube will visually jump — identical in appearance to the `add()` vs `attach()` bug.
```

Also update the Render Loop section to mark RAF as preferred:

Replace:
```
1. **Continuous rAF loop** — `requestAnimationFrame` that calls `renderer.render()` every frame. Simple, always works.
```
with:
```
1. **Continuous rAF loop** — `requestAnimationFrame` that calls `renderer.render()` every frame. Simple, always works. **(spec preferred — the boilerplate provides this)**
```

**Step 5: Verify**

Run: `grep -n "direction-data\|boundary-algorithm\|yoyo\|updateMatrixWorld" skills/isometric-rolling-cube/SKILL.md`
Expected: reference links to both new files, plus the new sections.

Run: `ls skills/isometric-rolling-cube/references/`
Expected: `direction-data.md` and `boundary-algorithm.md`.

---

## Task 15: Collapse `skills/css-3d-cube/SKILL.md` (deprecated)

**Files:**
- Modify: `skills/css-3d-cube/SKILL.md`

**Step 1: Replace body with redirect**

Keep the frontmatter exactly as-is (lines 1–9 including `user-invocable: false`). Replace everything after line 9 (the entire body) with:

```markdown
# CSS 3D Cube (Deprecated)

This skill is deprecated. Use these active skills instead:

- `phase1-css-cube` — CSS cube construction (Desandro 6-div technique, face positioning, isometric projection)
- `isometric-rolling-cube` — Rolling animation, pivot technique, direction logic, boundary enforcement
```

**Step 2: Verify**

Run: `wc -l skills/css-3d-cube/SKILL.md`
Expected: ~15 lines (frontmatter + short body).

Run: `grep -n "Deprecated\|phase1-css-cube" skills/css-3d-cube/SKILL.md`
Expected: 2 matches.

---

## Task 16: Add workshop disclaimer to `skills/threejs-animation/SKILL.md`

**Files:**
- Modify: `skills/threejs-animation/SKILL.md`

**Step 1: Insert callout after the frontmatter**

After the closing `---` of the frontmatter (line 6) and before `# Three.js Animation` (line 8), insert:

```markdown
> **Workshop note:** This workshop drives all animation with GSAP — see the `gsap-expert` skill for rolling animation, timelines, and easing. This skill covers Three.js-native animation (keyframe tracks, `AnimationMixer`, skeletal animation, morph targets) which are not used in the LLL Animation Workshop.

```

**Step 2: Verify**

Run: `grep -n "Workshop note" skills/threejs-animation/SKILL.md`
Expected: line 8.

---

## Task 17: Fix `threejs-fundamentals/SKILL.md` Quick Start material

**Files:**
- Modify: `skills/threejs-fundamentals/SKILL.md` (Quick Start code block, lines 31–40)

**Step 1: Replace the material and lighting in the Quick Start**

Replace:
```js
const material = new THREE.MeshStandardMaterial({ color: 0x00ff00 });
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);

// Add light
scene.add(new THREE.AmbientLight(0xffffff, 0.5));
const dirLight = new THREE.DirectionalLight(0xffffff, 1);
dirLight.position.set(5, 5, 5);
scene.add(dirLight);
```
with:
```js
// MeshBasicMaterial needs no lighting — matches workshop Task 2.1
const material = new THREE.MeshBasicMaterial({ color: 0x4fd1c5 });
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);
```

**Step 2: Update camera line (remove z=5, it's for perspective camera)**

Replace:
```js
camera.position.z = 5;
```
with:
```js
// Note: for an OrthographicCamera (used in the workshop), position the camera
// at an isometric angle rather than straight-on. The boilerplate provides this.
camera.position.z = 5; // example for PerspectiveCamera only
```

**Step 3: Verify**

Run: `grep -n "MeshBasicMaterial\|AmbientLight\|DirectionalLight" skills/threejs-fundamentals/SKILL.md`
Expected: `MeshBasicMaterial` present, no `AmbientLight` or `DirectionalLight`.

---

## Task 18: Add resource links to `skills/workshop-guide/references/resources.md`

**Files:**
- Modify: `skills/workshop-guide/references/resources.md`

**Step 1: Add new section for Facilitator Guides**

After the "## Workshop Spec" section (after line 40), add:

```markdown
## Facilitator Guides

These documents were added by the workshop facilitator and provide additional architectural context beyond the spec:

- **[Crash Course: 3D cube animations (CSS vs Three.js)](https://www.notion.so/labrys/Crash-Course-3D-cube-animations-CSS-vs-Three-js-313ced2a7e13801eb0f8e5ed0057e568)** — Explains the 3-container CSS architecture (position/perspective/modeled-cube separation), the `THREE.Group` isometric container, and the `yoyo` jump effect. Good conceptual overview of how both phases work architecturally.

- **[3D Cube CSS vs Three.js — Performance Comparison](https://www.notion.so/labrys/3D-Cube-CSS-vs-Three-js-30bced2a7e138077b50ce60c97faddd2)** — Explains why CSS is fine for a single cube but Three.js/WebGL is needed for 1,000+ cubes. Useful context for Phase 2 motivation.
```

**Step 2: Verify**

Run: `grep -n "Facilitator Guides\|Crash Course\|Performance Comparison" skills/workshop-guide/references/resources.md`
Expected: 3 matches.

---

## Task 19: Narrow `skills/phase1-css-cube/SKILL.md` description scope

**Files:**
- Modify: `skills/phase1-css-cube/SKILL.md` (frontmatter description, lines 3–10)

**Step 1: Replace description**

Replace the frontmatter description:
```yaml
description: >
  This skill should be used when the user asks to "build a CSS 3D cube", "construct cube faces",
  "implement Phase 1", "position faces with translateZ", "set up preserve-3d", "apply isometric
  projection", "work on task 1.1", "work on task 1.2", "create a 6-div cube", "add face shading",
  or is researching, planning, or implementing any step in Phase 1 (Tasks 1.1–1.6) of the LLL
  Animation Workshop. Covers the Desandro 6-div cube construction technique with rotate +
  translateZ face positioning, isometric projection, and Phase 1 task guidance.
  Replaces the deprecated css-3d-cube skill (Julia Miocene 2-div approach).
```
with:
```yaml
description: >
  This skill should be used when the user asks to "build a CSS 3D cube", "construct cube faces",
  "position faces with translateZ", "set up preserve-3d", "apply isometric projection",
  "work on task 1.1", "work on task 1.2", "create a 6-div cube", or "add face shading".
  Covers Tasks 1.1–1.2 only: the Desandro 6-div cube construction technique with rotate +
  translateZ face positioning, and isometric projection. For Tasks 1.3–1.6 (rolling,
  directions, boundaries, cursor), use the `isometric-rolling-cube` skill.
  Replaces the deprecated css-3d-cube skill (Julia Miocene 2-div approach).
```

**Step 2: Add react-integration delegation line to Related Skills**

Find the "## Related Skills" section. Add a line:
```markdown
- `react-integration` — Using GSAP with React refs (`useRef`, `useEffect`), JSX differences from HTML
```

**Step 3: Verify**

Run: `grep -n "Tasks 1.1–1.2 only\|react-integration" skills/phase1-css-cube/SKILL.md`
Expected: both strings present.

---

## Final Verification

Run each of these to confirm no old strings remain:

```bash
# Old rotation formula — should return no results
grep -rn "rotateX(35.264deg)" skills/ agents/

# Old tech stack label — should return no results
grep -rn "vanilla JS/TS\|Vite + TypeScript" skills/ agents/

# Old direction names — should return no results in isometric-rolling-cube CSS table
grep -n "Right (+X)\|Left (-X)\|Forward (-Y\|Backward (+Y" skills/isometric-rolling-cube/SKILL.md

# Old render loop ranking — gsap.ticker should no longer be "(recommended)"
grep -n "gsap.ticker.*recommended" agents/phase-transition-helper.md

# New content present
grep -rn "react-integration" skills/ agents/
grep -rn "THREE.Group" agents/phase-transition-helper.md
grep -rn "yoyo" skills/isometric-rolling-cube/SKILL.md
grep -rn "CSS Check A\|CSS Check B" agents/animation-reviewer.md
grep -rn "Starter Repository" skills/workshop-guide/SKILL.md
```

All "old strings" checks should return no results. All "new content" checks should return at least 1 result each.

---

## Execution Options

Plan complete and saved to `docs/plans/2026-02-27-plugin-updates-plan.md`.

**1. Subagent-Driven (this session)** — Dispatch a fresh subagent per task, review between tasks. Use `superpowers:subagent-driven-development`.

**2. Parallel Session (separate)** — Open a new session and use `superpowers:executing-plans` to work through the plan with checkpoints.
