---
name: gsap-expert
description: "This skill should be used when the user needs GSAP v3 API reference, animation patterns, timeline composition, easing functions, ticker integration, or plugin usage guidance."
user-invocable: true
argument-hint: "[topic] e.g. timeline, onComplete, easing, ticker"
---

# GSAP Expert Skill

Expert-level knowledge of the GreenSock Animation Platform (GSAP) v3, synthesized from official GSAP documentation covering 152+ pages of API reference, plugins, eases, helper functions, and resources.

## Source Synthesis

This skill synthesizes knowledge from the official GSAP documentation (gsap.com). All examples, API references, and best practices are drawn from the official docs. The reference file `v3.md` contains the complete 152-page documentation set (~620KB), while `llms.md` provides a navigational index of all documented APIs and helper functions.

**Source confidence**: Medium (scraped documentation). All code examples originate from official GSAP docs and CodePen demos maintained by GreenSock.

## When to Use This Skill

This skill should be triggered when:

### Core Animation
- **Creating animations** with `gsap.to()`, `gsap.from()`, `gsap.fromTo()`, or `gsap.set()`
- **Building timelines** for sequencing complex animation choreography
- **Controlling playback** - play, pause, reverse, seek, timeScale, progress
- **Using keyframes** for multi-step property animations on a single tween
- **Working with easing** functions (CustomEase, CustomBounce, CustomWiggle, RoughEase, SlowMo)
- **Function-based values** - dynamic per-target values using `function(index, target, targets)`

### Scroll & Interaction
- **Implementing scroll-based animations** with ScrollTrigger (pinning, scrubbing, snapping)
- **Adding smooth scrolling** with ScrollSmoother (parallax, data-speed, data-lag)
- **Making elements draggable** with the Draggable plugin (drag, spin, toss with inertia)
- **Detecting gestures** with Observer (unified touch/mouse/pointer/scroll events)
- **Scroll-to animations** with ScrollToPlugin

### Text & SVG
- **Working with text animations** using SplitText (splitting into chars, words, lines with masks)
- **Animating SVG** paths (MorphSVG, DrawSVG, MotionPath)
- **Scramble text effects** with ScrambleTextPlugin

### Layout & Physics
- **Building FLIP animations** with the Flip plugin (layout transitions, element swaps)
- **Implementing physics-based motion** with Physics2D or PhysicsProps plugins
- **Momentum/velocity tracking** with InertiaPlugin

### Framework Integration
- **Using GSAP with React**, Vue, Angular, or other JS frameworks
- **Cleaning up animations** with `gsap.context()` (essential for React/SPA environments)
- **Creating responsive animations** with `gsap.matchMedia()`

### Optimization & Debugging
- **Debugging animation issues** (FOUC, conflicting tweens, ScrollTrigger positioning)
- **Optimizing animation performance** (will-change, GPU acceleration, lazy rendering)
- **Visual debugging** with GSDevTools and ScrollTrigger markers
- **Using GSAP utility methods** (clamp, wrap, snap, random, interpolate, pipe, etc.)

## Key Concepts

### Core Architecture

GSAP revolves around two fundamental concepts:

- **Tween** - A high-performance property setter that animates targets from one state to another. Created via `gsap.to()`, `gsap.from()`, or `gsap.fromTo()`. Can animate ANY property of ANY JavaScript object, not just CSS. Think of it as a "property setter" that updates values over time.
- **Timeline** - A container for Tweens (and other Timelines). Provides sequencing, coordination, and unified playback control (`play()`, `pause()`, `reverse()`, `timeScale()`, etc.). Timelines can be nested for modular animation code. Every animation is placed onto a parent timeline (the `globalTimeline` by default).

### The Position Parameter

The secret to sophisticated timeline choreography. Controls exactly where animations are placed within a timeline:

| Syntax | Meaning |
|--------|---------|
| `3` | Absolute time (3 seconds) |
| `"+=1"` | 1 second after the end of the timeline |
| `"-=1"` | 1 second overlap with the end |
| `"someLabel"` | At a named label |
| `"someLabel+=1"` | 1 second after a label |
| `"<"` | Start of the most recently-added animation |
| `">"` | End of the most recently-added animation |
| `"<0.5"` | 0.5s after the start of the previous animation |
| `">-0.5"` | 0.5s before the end of the previous animation |

### Special Properties (Tween vars)

Key properties available on every tween:

| Property | Default | Description |
|----------|---------|-------------|
| `duration` | `0.5` | Animation duration in seconds |
| `delay` | `0` | Delay before animation starts |
| `ease` | `"power1.out"` | Easing function |
| `stagger` | `0` | Delay between each target's animation |
| `repeat` | `0` | Number of repeats (-1 for infinite) |
| `yoyo` | `false` | Reverse direction on each repeat |
| `overwrite` | `false` | How to handle conflicting tweens |
| `paused` | `false` | Create in paused state |
| `immediateRender` | varies | Render immediately on creation |
| `keyframes` | - | Array of keyframe objects |
| `onComplete` | - | Callback when animation completes |
| `onUpdate` | - | Callback on each frame update |
| `onStart` | - | Callback when animation starts |
| `onRepeat` | - | Callback on each repeat |
| `onReverseComplete` | - | Callback when reverse completes |

### Plugin Registration

Plugins must be registered before use (except core plugins like CSS, Modifiers, Snap, Attributes, EndArray):

```javascript
gsap.registerPlugin(ScrollTrigger, SplitText, Flip, Draggable);
```

### Installation

```javascript
// npm
npm install gsap

// Then import
import { gsap } from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
```

## Quick Reference

### 1. Basic Tween (from official docs)

```javascript
// Animate elements with class "box" - rotation and translateX over 1 second
gsap.to(".box", { rotation: 27, x: 100, duration: 1 });
```

### 2. Timeline with Sequencing (from official docs)

```javascript
// Tweens are sequenced one-after-the-other by default
let tl = gsap.timeline();
tl.to("#green", { duration: 1, x: 786 })
  .to("#blue", { duration: 2, x: 786 })
  .to("#orange", { duration: 1, x: 786 });
```

### 3. Tween Playback Control (from official docs)

```javascript
let tween = gsap.to(".class", { rotation: 360, duration: 5, ease: "elastic" });

// Control it programmatically
tween.pause();
tween.seek(2);
tween.progress(0.5);
tween.play();
tween.reverse();
tween.timeScale(2); // double speed
```

### 4. Function-Based Values (from official docs)

```javascript
gsap.to(".class", {
  x: 100,
  y: function(index, target, targets) {
    return index * 50; // each target gets a different y value
  },
  duration: 1
});
```

### 5. ScrollTrigger - Basic Scroll Animation (from official docs)

```javascript
gsap.registerPlugin(ScrollTrigger);

gsap.to(".box", {
  scrollTrigger: {
    trigger: ".box",
    start: "top center",   // when top of .box hits center of viewport
    end: "bottom top",     // when bottom of .box hits top of viewport
    scrub: true,           // link animation progress to scroll position
    markers: true,         // visual debugging markers
    pin: true,             // pin the trigger element during animation
  },
  x: 500,
  rotation: 360,
});
```

### 6. ScrollTrigger - Standalone with Callbacks (from official docs)

```javascript
ScrollTrigger.create({
  trigger: ".section",
  start: "top center",
  end: "bottom center",
  toggleClass: "active",
  onEnter: (self) => console.log("entered!", self.direction),
  onLeave: (self) => console.log("left!"),
  onEnterBack: (self) => console.log("entered back!"),
  onLeaveBack: (self) => console.log("left back!"),
  toggleActions: "play pause resume reset",
});
```

### 7. SplitText - Text Animation (from official docs)

```javascript
gsap.registerPlugin(SplitText);

// Split text into chars, words, lines with responsive re-splitting
let split = SplitText.create(".my-text", {
  type: "words, chars",
  autoSplit: true,
  mask: "words",        // wrap words for reveal effects
  onSplit() {
    return gsap.from(split.chars, {
      opacity: 0,
      y: 50,
      stagger: 0.03,
      ease: "back.out",
    });
  }
});
```

### 8. Flip Plugin - Layout Transitions (from official docs)

```javascript
gsap.registerPlugin(Flip);

// Step 1: Capture current state
const state = Flip.getState(".items");

// Step 2: Make DOM/style changes
container.classList.toggle("reorder");

// Step 3: Animate from old state to new state
Flip.from(state, {
  duration: 0.7,
  ease: "power1.inOut",
  stagger: 0.08,
  absolute: true,       // use position:absolute during flip
  onEnter: (elements) => gsap.fromTo(elements, { opacity: 0 }, { opacity: 1 }),
  onLeave: (elements) => gsap.to(elements, { opacity: 0 }),
});
```

### 9. Physics2D Plugin (from official docs)

```javascript
gsap.registerPlugin(Physics2DPlugin);

gsap.to(element, {
  duration: 2,
  physics2D: { velocity: 300, angle: -60, gravity: 400 },
});
// Or with friction
gsap.to(element, {
  duration: 2,
  physics2D: { velocity: 300, angle: -60, friction: 0.1 },
});
// Or with directional acceleration
gsap.to(element, {
  duration: 2,
  physics2D: { velocity: 300, angle: -60, acceleration: 50, accelerationAngle: 180 },
});
```

### 10. Draggable (from official docs)

```javascript
gsap.registerPlugin(Draggable);

// Basic draggable with bounds
Draggable.create("#yourID", {
  type: "x,y",
  bounds: "#container",
  inertia: true,             // momentum after release (requires InertiaPlugin)
  onDragEnd: function() {
    console.log("drag ended at", this.x, this.y);
  },
});

// Spinnable (rotation)
Draggable.create("#knob", {
  type: "rotation",
  snap: function(endValue) {
    return Math.round(endValue / 90) * 90; // snap to 90-degree increments
  },
});
```

### 11. gsap.context() - Cleanup for Frameworks (from official docs)

```javascript
// Essential for React/Vue/Angular - collects all animations for easy cleanup
let ctx = gsap.context(() => {
  gsap.to(".box", { x: 100 });
  gsap.from(".title", { opacity: 0 });
  ScrollTrigger.create({ /* ... */ });
});

// Later, clean up ALL animations created in the context
ctx.revert(); // reverts everything and kills all animations
```

### 12. ScrollSmoother - Smooth Scrolling (from official docs)

```javascript
gsap.registerPlugin(ScrollTrigger, ScrollSmoother);

// Requires wrapper/content HTML structure:
// <div id="smooth-wrapper"><div id="smooth-content">...your content...</div></div>
ScrollSmoother.create({
  smooth: 1,              // seconds to "catch up" to native scroll
  effects: true,          // enable data-speed and data-lag attributes
  normalizeScroll: true,  // prevents mobile address bar issues
  onUpdate: (self) => console.log("progress", self.progress),
});
```

Then use HTML attributes for parallax effects:
```html
<div data-speed="0.5">Scrolls at half speed (parallax)</div>
<div data-speed="2">Scrolls at double speed</div>
<div data-lag="0.5">Takes 0.5s to catch up (lagging effect)</div>
<img data-speed="auto" /> <!-- auto-calculates parallax within overflow:hidden parent -->
```

### 13. Modifiers Plugin - Dynamic Value Interception (from official docs)

```javascript
// ModifiersPlugin is built into GSAP core - no registration needed
// Intercept values on each tick for custom logic (snapping, wrapping, clamping)
gsap.to(".box", {
  x: 500,
  modifiers: {
    x: (x) => gsap.utils.wrap(0, 100)(parseFloat(x)) + "px"
  },
  duration: 3,
  repeat: -1,
});
```

### 14. Utility Methods (from official docs)

```javascript
// Clamp a value within a range
gsap.utils.clamp(0, 100, -12);        // 0
gsap.utils.clamp(0, 100, 150);        // 100

// Snap to increments or array values
gsap.utils.snap(5, 13);               // 15
gsap.utils.snap([0, 5, 10], 7);       // 5

// Map one range to another
gsap.utils.mapRange(-10, 10, 0, 100, 5); // 75

// Random values
gsap.utils.random(0, 100, 5);         // random multiple of 5 between 0-100
gsap.utils.random(["red", "blue"]);   // random pick from array

// Compose utility functions
let transform = gsap.utils.pipe(
  gsap.utils.clamp(0, 100),
  gsap.utils.snap(5)
);
transform(8);                          // 10

// Convert selector/NodeList to array
gsap.utils.toArray(".my-elements");

// Interpolate between values
gsap.utils.interpolate("red", "blue", 0.5); // "rgba(128,0,128,1)"

// Distribute values (great for staggers)
gsap.utils.distribute({ base: 0, amount: 100, from: "center" });
```

### 15. Observer - Gesture Detection (from official docs)

```javascript
gsap.registerPlugin(Observer);

Observer.create({
  target: window,
  type: "wheel,touch,pointer",
  onUp: () => goToPreviousSection(),
  onDown: () => goToNextSection(),
  tolerance: 10,           // minimum distance before triggering
  preventDefault: true,
});

// Detect touch capability
if (Observer.isTouch) {
  // touch-capable device
}
if (Observer.isTouch === 1) {
  // touch-only device (no mouse)
}
```

## Available Plugins

### Free Plugins (included in core - no registration needed)
| Plugin | Description |
|--------|-------------|
| **CSS** | Animates CSS properties (transforms, colors, etc.) - core plugin, auto-included |
| **Modifiers** | Intercept animated values on each tick for custom logic - core plugin |
| **Snap** | Snap values to increments or arrays - core plugin |
| **Attributes** | Animate any attribute of any DOM element - core plugin |
| **EndArray** | Animate between arrays of numbers - core plugin |

### Free Plugins (require `gsap.registerPlugin()`)
| Plugin | Description |
|--------|-------------|
| **ScrollTrigger** | Scroll-based animation triggers, pinning, scrubbing, snapping |
| **Observer** | Unified touch/mouse/pointer event detection (onUp, onDown, etc.) |
| **Draggable** | Make elements draggable, spinnable, tossable |
| **Flip** | FLIP animation technique for layout changes |
| **MotionPathPlugin** | Animate along SVG/custom paths |
| **TextPlugin** | Replace text content character-by-character |
| **EaselPlugin** | Animate EaselJS/CreateJS objects |
| **PixiPlugin** | Animate PIXI.js objects |
| **ScrollToPlugin** | Animate scroll position to a target |
| **CSSRulePlugin** | Animate CSS rules (pseudo-elements) |

### Club/Premium Plugins (require license)
| Plugin | Description |
|--------|-------------|
| **SplitText** | Split text into chars/words/lines for animation with mask support |
| **MorphSVGPlugin** | Morph between SVG shapes (any shape to any shape) |
| **DrawSVGPlugin** | Animate SVG stroke drawing |
| **ScrollSmoother** | Smooth scrolling built on ScrollTrigger with parallax |
| **InertiaPlugin** | Momentum/velocity-based motion with tracking |
| **Physics2DPlugin** | 2D physics (velocity, gravity, friction, acceleration) |
| **PhysicsPropsPlugin** | Physics for individual properties |
| **ScrambleTextPlugin** | Scramble/decode text effects |
| **CustomEase** | Create custom easing curves visually |
| **CustomBounce** | Customizable bounce easing |
| **CustomWiggle** | Customizable wiggle easing |
| **GSDevTools** | Visual debugging timeline UI |

## Easing Reference

GSAP uses the format `"name.direction"` where direction is `in`, `out`, or `inOut`:

| Ease | Aliases | Description |
|------|---------|-------------|
| **power0** | `"none"`, `"linear"` | Constant speed, no acceleration |
| **power1** | `"quad"` | Subtle acceleration |
| **power2** | `"cubic"` | Moderate acceleration |
| **power3** | `"quart"` | Strong acceleration |
| **power4** | `"quint"`, `"strong"` | Very strong acceleration |
| **back** | - | Overshoots then comes back |
| **elastic** | - | Spring-like oscillation |
| **bounce** | - | Bouncing effect |
| **circ** | - | Circular motion feel |
| **expo** | - | Exponential (sharper than power4) |
| **sine** | - | Sinusoidal (gentlest) |
| **steps(n)** | - | Stepped/discrete jumps |

**Special eases (require registration):**
| Ease | License | Description |
|------|---------|-------------|
| **RoughEase** | Free | Randomized/jittery easing |
| **SlowMo** | Free | Slow-motion middle portion |
| **ExpoScaleEase** | Free | Exponential scaling |
| **CustomEase** | Club | Create any curve with SVG path data |
| **CustomBounce** | Club | Configurable bounce (squash, bounces count) |
| **CustomWiggle** | Club | Configurable wiggle (wiggles count, type) |

Default ease: `"power1.out"`

## ScrollTrigger Deep Dive

### Position Syntax

The `start` and `end` properties use the format: `"triggerPosition scrollerPosition"`

**Keywords**: `top`, `center`, `bottom` (vertical) / `left`, `center`, `right` (horizontal)

| Example | Meaning |
|---------|---------|
| `"top center"` | Top of trigger hits center of viewport |
| `"top 80%"` | Top of trigger hits 80% down from top of viewport |
| `"bottom top"` | Bottom of trigger hits top of viewport |
| `"center center"` | Center of trigger hits center of viewport |
| `"top bottom-=100px"` | Top of trigger hits 100px above bottom of viewport |
| `"+=300"` (end only) | 300px beyond where start is |
| `"+=100%"` (end only) | Height of scroller beyond where start is |
| `"clamp(top bottom)"` | Clamped to valid scroll range (v3.12+) |
| `"max"` (end only) | Maximum scroll position |

### toggleActions

Controls what happens at 4 scroll positions: `"onEnter onLeave onEnterBack onLeaveBack"`

Actions: `play`, `pause`, `resume`, `restart`, `reset`, `complete`, `reverse`, `none`

Default: `"play none none none"`

Example: `toggleActions: "play pause resume reset"` - plays on enter, pauses on leave, resumes on enter-back, resets on leave-back.

### Key ScrollTrigger Config Options

| Property | Description |
|----------|-------------|
| `trigger` | Element that triggers the animation |
| `start` / `end` | Scroll positions (see syntax above) |
| `scrub` | `true` or Number (seconds to catch up) |
| `pin` | `true` or element to pin during scroll |
| `snap` | Number, Array, `"labels"`, or Function |
| `markers` | `true` or config object for visual debugging |
| `toggleClass` | CSS class to toggle while active |
| `toggleActions` | Actions at enter/leave/enterBack/leaveBack |
| `once` | `true` to kill after first trigger |
| `horizontal` | `true` for horizontal scroll |
| `scroller` | Custom scroll container element |
| `containerAnimation` | For triggers inside horizontal scroll sections |
| `pinReparent` | `true` if ancestor has transform/will-change |
| `pinSpacing` | `true` (default), `false`, or `"margin"` |
| `anticipatePin` | Number (1 is typical) to avoid pin flash |
| `fastScrollEnd` | `true` or velocity threshold |
| `invalidateOnRefresh` | `true` to flush cached start values on resize |
| `preventOverlaps` | `true` or group string |

### ScrollTrigger Static Methods

| Method | Description |
|--------|-------------|
| `ScrollTrigger.create()` | Create standalone ScrollTrigger |
| `ScrollTrigger.batch()` | Batch-trigger multiple elements |
| `ScrollTrigger.refresh()` | Recalculate all positions |
| `ScrollTrigger.getAll()` | Get array of all ScrollTriggers |
| `ScrollTrigger.getById()` | Find by id |
| `ScrollTrigger.killAll()` | Kill all ScrollTriggers |
| `ScrollTrigger.maxScroll()` | Get max scroll value |
| `ScrollTrigger.normalizeScroll()` | Fix mobile quirks |
| `ScrollTrigger.isInViewport()` | Check element visibility |
| `ScrollTrigger.scrollerProxy()` | Integrate 3rd-party scrollers |
| `ScrollTrigger.sort()` | Re-sort by position |
| `ScrollTrigger.saveStyles()` | Save styles for matchMedia |
| `ScrollTrigger.snapDirectional()` | Snap utility |

## Common Patterns

### Staggered Entrance Animation
```javascript
gsap.from(".card", {
  y: 60,
  opacity: 0,
  duration: 0.8,
  stagger: 0.15,
  ease: "power2.out",
});
```

### Scroll-Triggered Batch Animation
```javascript
ScrollTrigger.batch(".card", {
  onEnter: (batch) => gsap.to(batch, {
    opacity: 1,
    y: 0,
    stagger: 0.1,
  }),
  start: "top 85%",
});
```

### Responsive Animations with matchMedia
```javascript
let mm = gsap.matchMedia();

mm.add("(min-width: 800px)", () => {
  // Desktop animations - automatically reverted on mismatch
  gsap.to(".hero", { x: 200 });
  return () => { /* optional additional cleanup */ };
});

mm.add("(max-width: 799px)", () => {
  // Mobile animations
  gsap.to(".hero", { y: 100 });
});
```

### Random/Dynamic Values with repeatRefresh
```javascript
gsap.to(".star", {
  x: "random(-200, 200)",       // random per target
  y: "random(-200, 200)",
  rotation: "random(0, 360)",
  duration: "random(1, 3)",
  repeat: -1,
  repeatRefresh: true,           // get new random values on each repeat
  ease: "none",
});
```

### Pinned Horizontal Scroll Section
```javascript
let tl = gsap.timeline({
  scrollTrigger: {
    trigger: ".panel-container",
    pin: true,
    scrub: 1,
    snap: 1 / (panels.length - 1),
    end: () => "+=" + document.querySelector(".panel-container").offsetWidth,
  }
});

panels.forEach((panel, i) => {
  tl.to(panel, { xPercent: -100 * i });
});
```

### containerAnimation for Horizontal Triggers
```javascript
// Trigger animations inside a horizontally-scrolling section
let scrollTween = gsap.to(panels, {
  xPercent: -100 * (panels.length - 1),
  ease: "none",
  scrollTrigger: {
    trigger: ".container",
    pin: true,
    scrub: 1,
    end: () => "+=" + document.querySelector(".container").offsetWidth,
  }
});

// Now create triggers relative to the horizontal scroll
gsap.from(".panel-content", {
  opacity: 0,
  y: 50,
  scrollTrigger: {
    trigger: ".panel-content",
    containerAnimation: scrollTween, // key property
    start: "left center",
  }
});
```

### React useGSAP Pattern
```javascript
import { useGSAP } from "@gsap/react";

function MyComponent() {
  const container = useRef();

  useGSAP(() => {
    // All GSAP animations here are automatically cleaned up
    gsap.to(".box", { x: 100 });
    gsap.from(".title", { opacity: 0 });

    ScrollTrigger.create({
      trigger: ".section",
      start: "top center",
      onEnter: () => console.log("entered"),
    });
  }, { scope: container }); // scope limits selector queries to this container

  return <div ref={container}>...</div>;
}
```

### Avoiding FOUC (Flash of Unstyled Content)
```css
/* Hide elements initially */
.gsap-hidden { visibility: hidden; }
```
```javascript
// Reveal with animation
gsap.from(".gsap-hidden", {
  opacity: 0,
  y: 30,
  duration: 0.8,
  onStart: function() {
    this.targets().forEach(el => el.classList.remove("gsap-hidden"));
  }
});
```

### quickTo for Performant Repeated Animations
```javascript
// Create a reusable tween for cursor-following effects
let xTo = gsap.quickTo(".follower", "x", { duration: 0.6, ease: "power3" });
let yTo = gsap.quickTo(".follower", "y", { duration: 0.6, ease: "power3" });

window.addEventListener("mousemove", (e) => {
  xTo(e.clientX);
  yTo(e.clientY);
});
```

### Registered Effects (Reusable Animations)
```javascript
gsap.registerEffect({
  name: "fadeIn",
  effect: (targets, config) => {
    return gsap.from(targets, {
      duration: config.duration,
      opacity: 0,
      y: config.y,
      ease: "power2.out",
    });
  },
  defaults: { duration: 1, y: 50 },
  extendTimeline: true, // allows tl.fadeIn(".box")
});

// Use it anywhere
gsap.effects.fadeIn(".box");
// Or on a timeline
tl.fadeIn(".section", "+=0.5");
```

## Complete API Surface

### gsap Global Methods

| Method | Description |
|--------|-------------|
| `gsap.to()` | Animate TO specified values |
| `gsap.from()` | Animate FROM specified values |
| `gsap.fromTo()` | Animate FROM first values TO second values |
| `gsap.set()` | Immediately set properties (zero-duration tween) |
| `gsap.timeline()` | Create a new Timeline |
| `gsap.context()` | Create animation context for cleanup |
| `gsap.matchMedia()` | Responsive animation setup |
| `gsap.matchMediaRefresh()` | Force matchMedia to re-check |
| `gsap.registerPlugin()` | Register GSAP plugins |
| `gsap.registerEffect()` | Register reusable effects |
| `gsap.registerEase()` | Register custom easing functions |
| `gsap.defaults()` | Set default tween properties |
| `gsap.config()` | Configure global settings |
| `gsap.delayedCall()` | Call a function after a delay |
| `gsap.killTweensOf()` | Kill all tweens of target(s) |
| `gsap.getTweensOf()` | Get array of tweens for target(s) |
| `gsap.isTweening()` | Check if target is being tweened |
| `gsap.getProperty()` | Get current property value |
| `gsap.getById()` | Get tween/timeline by id |
| `gsap.exportRoot()` | Export all animations to new timeline |
| `gsap.quickTo()` | Create optimized reusable tween |
| `gsap.quickSetter()` | Create optimized property setter |
| `gsap.ticker` | Access the global tick/render cycle |
| `gsap.globalTimeline` | The root timeline |
| `gsap.parseEase()` | Convert ease string to function |
| `gsap.updateRoot()` | Manually advance the global timeline |

### Utility Methods (`gsap.utils.*`)

| Method | Description |
|--------|-------------|
| `clamp(min, max, value)` | Clamp value within range |
| `snap(increment, value)` | Snap to nearest increment |
| `wrap(min, max, value)` | Wrap value (like modulo) |
| `wrapYoyo(min, max, value)` | Wrap with yoyo behavior |
| `mapRange(inMin, inMax, outMin, outMax, value)` | Map value between ranges |
| `normalize(min, max, value)` | Normalize to 0-1 range |
| `interpolate(start, end, progress)` | Interpolate between values |
| `random(min, max, snap)` | Generate random value |
| `distribute({base, amount, from})` | Distribute values (for staggers) |
| `pipe(...functions)` | Compose functions |
| `toArray(selector)` | Convert to array |
| `selector(scope)` | Scoped selector function |
| `shuffle(array)` | Shuffle array in-place |
| `splitColor(color)` | Split color into components |
| `checkPrefix(property)` | Get vendor-prefixed property |
| `getUnit(value)` | Extract CSS unit string |
| `unitize(fn, unit)` | Add unit to function return value |

### Timeline Methods

| Method | Description |
|--------|-------------|
| `to()` / `from()` / `fromTo()` / `set()` | Add tweens |
| `add()` | Add tween, timeline, label, or callback |
| `addLabel()` / `removeLabel()` | Manage labels |
| `addPause()` / `removePause()` | Add/remove pause points |
| `call()` | Insert a function call |
| `play()` / `pause()` / `resume()` | Playback control |
| `reverse()` / `restart()` | Direction control |
| `seek()` / `time()` / `progress()` | Position control |
| `timeScale()` | Speed control |
| `kill()` / `clear()` | Cleanup |
| `invalidate()` | Flush cached values |
| `revert()` | Revert all changes |
| `tweenTo()` / `tweenFromTo()` | Animate playhead position |
| `getChildren()` / `getTweensOf()` | Query contents |
| `then()` | Promise-based completion |

### Tween Methods

Same as Timeline methods above (both extend Animation), plus:

| Method | Description |
|--------|-------------|
| `targets()` | Get array of target objects |
| `ratio` | Current eased progress (read-only) |

## Helper Functions

The official docs include 20+ helper functions for common patterns. Key ones available in `v3.md`:

| Helper | Description |
|--------|-------------|
| **seamlessLoop** | Seamlessly loop elements along the x-axis |
| **LottieScrollTrigger** | Hook a Lottie animation up to ScrollTrigger |
| **imageSequenceScrub** | Scrub through a canvas image sequence |
| **getScrollLookup** | Get scroll position for any element (ScrollTrigger-aware) |
| **callAfterResize** | Debounced resize handler |
| **stopOverscroll** | Stop overscroll behavior (even iOS Safari) |
| **tickGSAPWhileHidden** | Force GSAP updates in hidden browser tabs |
| **killChildTweensOf** | Kill all tweens on child elements |
| **formatNumber** | Format numbers with commas and decimals |
| **getNestedLabelTime** | Find a label's time in nested timelines |
| **addWeightedEases** | Create weighted custom eases |
| **blendEases** | Blend two easing functions |
| **smoothOriginChange** | Change transformOrigin without visual jump |
| **trackDirection** | Track scroll/animation direction changes |
| **weightedRandom** | Weighted random value generation |

## Physics2D Config Reference

The Physics2DPlugin config object properties (from official docs):

| Property | Default | Description |
|----------|---------|-------------|
| `velocity` | `0` | Initial velocity in pixels/second |
| `angle` | `0` | Initial angle in degrees (-60 = upper right) |
| `gravity` | `null` | Downward acceleration (pixels/s) |
| `acceleration` | `null` | Acceleration (mutually exclusive with gravity) |
| `accelerationAngle` | `null` | Direction of acceleration in degrees |
| `friction` | `0` | 0 to 1 (0.02 is a light amount, 1 stops all motion) |
| `xProp` | `"x"` | Property name for x-axis |
| `yProp` | `"y"` | Property name for y-axis |

Note: Easing is ignored for physics properties. Physics tweens are fully reversible.

## Reference Files

This skill includes comprehensive documentation in `references/`:

### v3.md
- **Source**: Official GSAP documentation (gsap.com)
- **Confidence**: Medium (scraped web content)
- **Coverage**: 152 pages of GSAP v3 API documentation
- **Contents**: Complete API reference for Tweens, Timelines, all plugins (ScrollTrigger, SplitText, Draggable, Flip, MorphSVG, DrawSVG, MotionPath, Physics2D, Observer, ScrollSmoother, and more), easing functions, utility methods, helper functions, and resource articles (React integration, common mistakes, FOUC, accessibility, etc.)
- **Size**: ~620KB - very comprehensive; use search/grep to find specific topics
- **Structure**: Each API entry starts with `## MethodName` followed by `**URL:**` and `**Contents:**` sections. Code examples follow `Example N (unknown):` markers.

### llms.md
- **Source**: GSAP documentation sitemap/index
- **Confidence**: Medium (structural index)
- **Contents**: Complete hierarchical listing of all 152+ documentation pages with their paths, organized by category (GSAP core, Timeline, Tween, Utils, Eases, HelperFunctions, Plugins, Resources). Essential for navigating the full API surface and finding specific method/property documentation paths.

### index.md
- **Source**: Generated index
- **Confidence**: Medium
- **Contents**: Summary pointing to v3.md as the main documentation file

## Working with This Skill

### For Beginners
1. Start with the **Key Concepts** section to understand Tweens and Timelines
2. Try Quick Reference examples 1-4 (basic tweens, timelines, playback control)
3. Read about the **Position Parameter** - it's the key to timeline mastery
4. Learn `gsap.context()` early if using React/Vue/Angular (example 11)
5. Use `markers: true` in ScrollTrigger during development
6. Read the "Avoiding FOUC" pattern to prevent layout shifts

### For Intermediate Users
1. Master **ScrollTrigger** (examples 5-6) - the most commonly used plugin
2. Learn **SplitText** for text animations (example 7)
3. Explore **utility methods** (example 14) - they compose beautifully with `pipe()`
4. Use `gsap.matchMedia()` for responsive animations
5. Try **Flip** for layout transition animations (example 8)
6. Use `gsap.quickTo()` for cursor-following and repeated animations
7. Register custom effects with `gsap.registerEffect()` for reusable patterns
8. Read the "Common Patterns" section for real-world recipes

### For Advanced Users
1. Dive into **Modifiers** for custom per-tick value interception (example 13)
2. Use **Physics2D/PhysicsProps** for realistic motion (example 9)
3. Combine **Flip** with ScrollTrigger for layout transition animations
4. Explore **helper functions** in v3.md (seamless loops, Lottie integration, image sequences)
5. Use `containerAnimation` for triggers inside horizontal scroll sections
6. Use `ScrollTrigger.normalizeScroll()` for consistent mobile behavior
7. Implement `ScrollTrigger.scrollerProxy()` for 3rd-party smooth scrolling libraries
8. Use `gsap.ticker` for custom render loop hooks
9. Explore `ScrollTrigger.batch()` for efficient viewport-entry animations
10. Use `Flip.batch()` for coordinating multiple independent Flip animations (React)

### Navigating the Reference Documentation
The main reference file (`v3.md`) is large (~620KB). Best practices for finding information:

1. **Use `llms.md` first** - Find the exact API path you need from the hierarchical index
2. **Grep for section headers** - Each API entry starts with `## MethodName`
3. **Search by URL pattern** - Entries include their official URL (e.g., `https://gsap.com/docs/v3/Plugins/ScrollTrigger.md`)
4. **Look for "Examples:" blocks** - Code examples follow `Example N (unknown):` markers
5. **Plugin docs** include "Config Object" sections listing all available options
6. **Resource articles** cover practical topics: React integration, common mistakes, FOUC prevention, accessibility, SVG animation, Webflow/WordPress integration

## Important Notes & Gotchas

### Critical Behavior
- **immediateRender**: Defaults to `true` for `from()` and `fromTo()` tweens, `false` for `to()`. This is a common source of confusion - a `from()` tween renders its starting values immediately when created, which can cause unexpected jumps.
- **Overwrite behavior**: Default is `false` (no overwriting). Use `overwrite: "auto"` to auto-kill conflicting properties, or `overwrite: true` to kill all tweens on the same target. This prevents animation conflicts.
- **Easing ignored for Physics**: Physics2D and PhysicsProps plugins ignore any `ease` property - motion is determined by physics parameters only.

### Framework Integration
- **React/Framework cleanup**: Always use `gsap.context()` (or `useGSAP` hook) and call `.revert()` in cleanup functions to prevent memory leaks. Never create animations without proper cleanup in SPA environments.
- **`useGSAP` hook**: The `@gsap/react` package provides a `useGSAP` hook that handles context creation and cleanup automatically. Use `{ scope: containerRef }` to limit selector queries.

### ScrollTrigger Pinning
- **Never animate the pinned element itself** - animate children inside it instead. ScrollTrigger pre-calculates positions and animating the pinned element will throw off measurements.
- Use `pinReparent: true` if ancestor has `transform` or `will-change` (which breaks `position: fixed`)
- `pinSpacing` defaults to `false` in flex containers since padding works differently
- Use `anticipatePin: 1` to avoid the flash of unpinned content during fast scrolling
- Create ScrollTriggers in **top-to-bottom order** (or use `refreshPriority`) for correct pin spacing calculations

### ScrollSmoother
- Requires specific HTML structure: `<div id="smooth-wrapper"><div id="smooth-content">...content...</div></div>`
- `normalizeScroll: true` prevents mobile address bar showing/hiding and fixes iOS Safari bugs
- `data-speed` effects should not be nested
- Built on top of ScrollTrigger - register both plugins

### Performance
- Use `will-change` sparingly; it can break `position: fixed` in some browsers
- Use `gsap.quickTo()` and `gsap.quickSetter()` for high-frequency updates (mousemove)
- `ScrollTrigger.batch()` is more efficient than individual ScrollTriggers for many elements
- `scrub: 1` (or another number) is smoother than `scrub: true` but introduces lag
- **ModifiersPlugin** is a core plugin and does NOT need `gsap.registerPlugin()`

### containerAnimation Caveats
- The container's animation MUST use `ease: "none"` (linear)
- Pinning and snapping are NOT available on containerAnimation-based ScrollTriggers
- Avoid animating the trigger element horizontally, or offset start/end values accordingly

## Updating

To refresh this skill with updated documentation:
1. Re-run the scraper with the same configuration
2. The skill will be rebuilt with the latest information
3. Check the [GSAP changelog](https://gsap.com/docs/v3/changelog) for version-specific updates
