---
name: react-integration
description: >
  This skill should be used when a participant asks how to use GSAP or Three.js inside a React
  component, how to target DOM elements with GSAP in React, how to set up
  a Three.js scene in a useEffect, or why their animation works in vanilla
  JS but not in their .tsx file.
user-invocable: true
argument-hint: "e.g. 'GSAP with useRef', 'Three.js in useEffect'"
---

# React Integration

The LLL Animation Workshop uses Vite + React TypeScript. GSAP and Three.js both work with React, but require specific integration patterns to avoid bugs.

## Why React Changes Things

React manages the DOM — components render JSX which React turns into real DOM elements. This creates two challenges for animation libraries:

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
      tl.to(position, { x: 100, y: 100, duration: 0.5 })
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

- Use `useRef<HTMLCanvasElement>` for the canvas element if you need to pass it to `WebGLRenderer` — the workshop boilerplate handles this for you, but the pattern is the same.
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
