# SVG & path animation

Motion on vector geometry rather than on boxes. Strokes that draw themselves, shapes that
become other shapes, objects that travel along a route, illustrations that come apart and
reassemble.

What makes this family distinct is that **the geometry itself is animatable**. Everywhere
else you move a rectangle around; here you can change what the shape *is*. That buys effects
nothing else can produce — a line drawing itself, a logo melting into an icon — and it comes
with a cost model that surprises people: only `transform` and `opacity` are composited on
the GPU without layout work, while `fill`, `stroke`, `stroke-dashoffset` and animated path
`d` data all trigger CPU-bound repaints. SVG animation is cheap at icon scale and expensive
at illustration scale, and the boundary between those is sharper than it looks.

**The one trick worth learning first.** The classic line-draw requires knowing a path's
total length, which historically meant measuring it in JavaScript. Setting **`pathLength="1"`**
on the path normalises any path to a 0–1 length, so `stroke-dasharray: 1; stroke-dashoffset: 1`
animating to `0` draws any path with no measurement at all. That single attribute removes the
main reason this family needed JavaScript.

**The morphing constraint.** Paths interpolate point by point, so a naive morph requires both
`d` attributes to have the **same number and type of commands**. Two shapes drawn by hand
almost never do. This is what dedicated morph plugins exist to solve, and why "just animate
between these two icons" is a much bigger request than it sounds.

---

### Line draw (`svg.stroke-draw`)

- **One line** — a stroke reveals itself progressively, as if being drawn.
- **What the reader sees** — A line appears at one end of a shape and extends along it,
  tracing the outline in real time — around a circle, along a signature, through a diagram's
  connector — arriving at the far end after a second or so. Because the line follows the
  path's own direction, it reads unmistakably as *drawing*: a pen moving, not a shape fading
  in. On a logo mark it is elegant; on a diagram it is genuinely explanatory, because the
  order of drawing tells you how to read the drawing.
- **Mechanism** — `stroke-dasharray` set to the path length and `stroke-dashoffset` animated
  from that length to zero, so the dash pattern slides into view.
- **Stack** — CSS keyframes with `pathLength="1"` needs no JavaScript at all. GSAP DrawSVG
  (free since 2025) adds segment control — drawing from the middle outward, or animating a
  visible window along the path.
- **Params** — duration (0.8–2s depending on path length); stagger between paths (0.05–0.15s);
  direction (reverse the offset sign); segment (a window rather than a growing line).
- **Use when** — logos, signatures, icons, diagram connectors, underline flourishes.
- **Don't use when** — the shape is filled rather than stroked. This technique only draws
  outlines; a filled shape has to fake it with a clip.
- **Reduced motion** — the completed drawing, immediately.
- **Performance** — `stroke-dashoffset` repaints but does not reflow. Fine for icons; a
  hundred simultaneously drawing paths in a detailed illustration is a different proposition.
- **Gotchas** — **the path must have a visible `stroke` and `stroke-width` or nothing renders
  at all** — the most common first-attempt failure, and it produces no error. Use
  `animation-fill-mode: forwards` or the stroke resets to invisible when the animation ends.
  Note also that a drawn stroke traces the path's own start point and direction, which for an
  exported shape is wherever the design tool happened to begin — often the wrong place, and
  it must be fixed in the path data, not in the animation.
- **References** — https://cssvg.com/blog/svg-path-animation ·
  https://www.svgai.org/blog/svg-path-animation-tutorial

---

### Shape morph (`svg.morph`)

- **One line** — one shape becomes another by interpolating its geometry.
- **What the reader sees** — A play triangle flows into a pause bar. A menu's three lines
  bend and rotate into a cross. A blob shifts continuously between organic forms, never
  repeating. The shape does not fade out and another fade in — the outline itself deforms,
  points travelling to new positions, so the two states feel like the same object in two
  configurations. Done well it reads as liquid; done badly the outline briefly folds through
  itself and the illusion collapses into a knot.
- **Mechanism** — interpolation between two path definitions, matching points pairwise.
- **Stack** — GSAP MorphSVG (free since 2025) handles mismatched point counts and can pick a
  sensible starting correspondence. Native CSS `d` interpolation exists but requires the paths
  to be command-compatible already, which in practice means authoring them that way.
- **Params** — duration (0.3–0.5s for icon states, seconds for ambient blobs); shape index
  (which point on shape B pairs with the first point of shape A — this single number decides
  whether the morph is elegant or twisted); morph type (rotational interpolation fixes most
  mid-morph kinks).
- **Use when** — icon state changes, logo transitions, one ambient organic shape.
- **Don't use when** — the two shapes are unrelated. Morphing a house into a chart is a
  novelty, not communication.
- **Reduced motion** — swap between the two shapes instantly.
- **Performance** — animating `d` is CPU-bound repainting every frame. **Keep point counts
  under about 200**; a morph between two detailed illustrations will drop frames.
- **Gotchas** — resolve the shape index once during development and hard-code it; leaving the
  automatic calculation in production pays that cost on every run. Convert primitives
  (`<rect>`, `<circle>`) to paths first — they cannot morph as they are. Both shapes need
  compatible command types, not just compatible counts.
- **References** — https://www.svgai.org/blog/research/svg-animation-encyclopedia-complete-guide ·
  https://webflow.com/blog/gsap-becomes-free

---

### Motion path (`svg.motion-path`)

- **One line** — an element travels along a defined route, optionally turning to face it.
- **What the reader sees** — A dot moves along a winding line — a route on a map, a wire in a
  diagram, an arc across a hero — following every curve exactly, and if it is set to
  auto-rotate, tilting as it goes so it always points the way it is travelling. Unlike a
  translate, which can only go in straight lines between keyframes, this follows arbitrary
  geometry. The effect is of something *travelling* rather than being moved, and it is the
  clearest way to show a journey, a connection, or a process flowing between stages.
- **Mechanism** — CSS `offset-path` with `offset-distance` animated from 0% to 100%, and
  `offset-rotate` for facing; or a library's motion-path plugin doing the same with more
  control.
- **Stack** — CSS `offset-path` is native and needs no library. GSAP MotionPath adds
  alignment to another element, path editing helpers and easing per segment.
- **Params** — duration; auto-rotate on or off; start and end offsets (travelling a portion
  of a path); alignment.
- **Use when** — maps, process diagrams, a decorative element following an arc, an icon
  travelling a connector.
- **Don't use when** — a straight translate would do. The path machinery is only worth it
  when the route is genuinely curved.
- **Reduced motion** — place the element at its final position on the path, no travel.
- **Performance** — `offset-distance` animates transforms under the hood and is comparatively
  cheap. Many travellers on one path are fine; many paths are the cost.
- **Gotchas** — the path's coordinate system is the element's containing block, so a path
  authored in an SVG's viewBox will not line up when applied to an HTML element without
  conversion. Auto-rotation is relative to the path tangent, so an asset that already points
  "up" needs an offset baked in.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/offset-path

---

### Stroke dash flow (`svg.dash-flow`)

- **One line** — a dashed line's dashes travel along it continuously.
- **What the reader sees** — A dashed connector between two boxes, and the dashes are moving
  — sliding steadily from one end to the other, like marching ants or fluid in a pipe. Nothing
  else changes; the line stays where it is. It reads immediately as *flow*: data moving between
  services, a route being followed, a process in progress. It is the cheapest possible way to
  make a static diagram look live, and it is a loop, so it must be treated as ambient motion.
- **Mechanism** — a repeating `stroke-dashoffset` animation with a dash pattern that repeats
  seamlessly.
- **Stack** — CSS keyframes. No library.
- **Params** — dash pattern; speed (a full pattern-length per 0.5–1.5s); direction.
- **Use when** — architecture diagrams, route lines, loading and processing states.
- **Don't use when** — several lines flow at once at different speeds; the diagram becomes
  agitated.
- **Reduced motion** — static dashes. This is continuous motion and it must stop.
- **Performance** — a permanent repaint of the stroked area. Small per line, but it never
  stops — pause it when off-screen.
- **Gotchas** — the animation distance must equal exactly one dash-plus-gap or the loop
  visibly jumps at the seam. Adding `stroke-linecap: round` changes the effective pattern
  length and reintroduces the jump you just fixed.
- **References** — https://cssvg.com/blog/svg-path-animation

---

### Icon state transition (`svg.icon-state`)

- **One line** — an icon animates between its meanings rather than swapping.
- **What the reader sees** — Tap the menu icon: its three lines rotate and converge into a
  cross, over about a fifth of a second. Tap play: the triangle splits and squares off into
  two pause bars. The icon never disappears — you watch it become the other thing, which
  makes the relationship between the two states obvious and confirms the tap in the same
  gesture. It is the highest-value micro-detail in most interfaces, because these icons are
  pressed constantly.
- **Mechanism** — for simple cases, transforms on the individual sub-paths (rotate and
  translate three rectangles into a cross). For genuinely different geometry, a morph.
- **Stack** — CSS transforms cover the common cases and should be the first attempt; morphing
  only when the shapes cannot be reconciled with rotation and translation.
- **Params** — duration (0.15–0.25s — this is feedback, keep it fast); easing (ease-out, or a
  slight overshoot for a springier feel); origin per sub-path.
- **Use when** — menu toggles, play/pause, expand/collapse, check states.
- **Don't use when** — the two icons mean unrelated things. Morphing implies a relationship.
- **Reduced motion** — swap instantly between states.
- **Performance** — transform-only where possible, which keeps it compositor-friendly and
  cheap even on a toolbar full of icons.
- **Gotchas** — the icon must expose its state to assistive technology (`aria-expanded`,
  `aria-pressed`); the animation communicates nothing to a screen reader. Sub-path transforms
  need correct `transform-origin` and `transform-box: fill-box`, which is the setting people
  miss and then wonder why everything rotates around the wrong point.
- **References** — https://effect-labs.com/en/pages/blog/svg-animations.html

---

### Progress ring (`svg.progress-ring`)

- **One line** — a circular stroke fills to show a proportion.
- **What the reader sees** — A ring, and a coloured arc growing around it clockwise from the
  top, stopping at the fraction it represents. For a determinate value it sweeps to position
  and stops; for an indeterminate one it rotates continuously while its arc length breathes
  in and out, so it never suggests a specific amount of progress. It is the standard vocabulary
  for "some of this is done", legible at any size, and it costs one path.
- **Mechanism** — the same dasharray/dashoffset technique as the line draw, on a circle, with
  the offset mapped to the value rather than animated to zero.
- **Stack** — CSS with `pathLength="1"`, which makes the maths trivial: offset equals
  `1 - progress`.
- **Params** — thickness; start angle (rotate -90° to begin at twelve o'clock); transition
  duration on value change (0.3s); cap style.
- **Use when** — upload and download progress, scores, completion meters, timers.
- **Don't use when** — the value is unknown and you use the determinate version anyway. A
  progress ring that stalls at 90% is worse than a spinner.
- **Reduced motion** — determinate rings still update; the indeterminate spinning version
  should be replaced with a static or stepped indicator.
- **Performance** — one stroked path repainting on change. Negligible.
- **Gotchas** — `pathLength="1"` is what makes this maintainable; without it every radius
  change requires recomputing the circumference. The value must also be exposed as text or
  `aria-valuenow` — a ring alone tells a screen reader nothing.
- **References** — https://www.boundev.ai/blog/svg-animation-css-tutorial-guide

---

### Filter distortion (`svg.filter-distort`)

- **One line** — SVG filters warp, blur or fracture content over time.
- **What the reader sees** — Type or an image ripples as though seen through moving water, or
  splits into coloured channels and reassembles, or dissolves into grain. The distortion is
  continuous and organic in a way transforms cannot produce — edges bend rather than sliding,
  and the whole surface behaves like a material rather than a graphic. It is the most
  visually distinctive thing in this family and by a wide margin the most expensive.
- **Mechanism** — an SVG filter chain (`feTurbulence` for noise, `feDisplacementMap` for
  warping, `feGaussianBlur`, `feColorMatrix`) with one or more of its attributes animated.
- **Stack** — SVG filters applied via CSS `filter: url(#id)`; any library to animate the
  attribute values.
- **Params** — turbulence frequency and octaves (octaves are the cost driver — two is usually
  enough); displacement scale; animation speed.
- **Use when** — one hero moment, a transition flourish, a deliberate texture on a brand site.
- **Don't use when** — anywhere near a performance budget, or on large areas, or on mobile.
- **Reduced motion** — a static filtered state, or no filter.
- **Performance** — **the most expensive technique in this catalogue.** Filters rasterise the
  entire filtered region every frame on the CPU in many engines; animating filter primitives
  on a full-width element will drop frames on hardware that handles everything else here
  comfortably. Measure before committing, on the worst device you support.
- **Gotchas** — filter regions clip by default, so displaced pixels get cut off at the edges
  unless the region is expanded. Filters flatten the element into a raster, so text inside
  loses subpixel rendering and gets noticeably softer. Support for individual primitives is
  uneven — test each one, not the concept.
- **References** — https://www.svgai.org/blog/research/svg-animation-encyclopedia-complete-guide

---

### Clip reveal (`svg.clip-reveal`)

- **One line** — content is revealed through a moving vector shape.
- **What the reader sees** — An image appears through a hole in the shape of something else —
  a circle expanding from a point, a letterform, a torn-paper edge sweeping across. Because
  the mask is vector geometry, the boundary can be any shape at all, not just a rectangle,
  and it can change shape as it moves. A circular wipe growing from where the user clicked is
  the everyday version; a wordmark-shaped window over video is the show-off version.
- **Mechanism** — `clip-path: url(#id)` referencing an SVG `<clipPath>`, or a `<mask>`, with
  the shape's geometry or transform animated.
- **Stack** — CSS `clip-path` with SVG geometry; animate the inner shape's attributes or
  transform.
- **Params** — shape; origin; duration (0.5–0.8s); whether the shape scales or morphs.
- **Use when** — brand-shaped reveals, transitions between images, expanding-circle
  navigation reveals.
- **Don't use when** — a rectangular clip would read the same. The vector version is more
  expensive to author and to render.
- **Reduced motion** — content fully visible, no clipping animation.
- **Performance** — clipping with a referenced SVG shape is more expensive than a CSS
  `inset()` or `circle()`; use the CSS basic shapes when the geometry allows.
- **Gotchas** — `clipPathUnits` decides whether coordinates are absolute or relative to the
  bounding box, and getting it wrong makes the clip vanish or misalign at different sizes —
  set `objectBoundingBox` and work in 0–1 units for anything responsive.
- **References** — https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Element/clipPath

---

### Handwriting signature (`svg.signature`)

- **One line** — a signature or hand-lettered word writes itself.
- **What the reader sees** — A pen stroke begins and moves as a hand would — fast through the
  long sweeps, slowing at the tight turns, lifting between letters — until a name stands
  written on the page. It takes a second or two. Unlike a mechanical line draw at constant
  speed, the variation in pace is what makes it read as *handwriting* rather than as a
  machine tracing an outline. It is a specific, high-impact use of stroke drawing, most often
  on an about page or a sign-off.
- **Mechanism** — a line draw across multiple paths, sequenced in writing order, with per-path
  durations tuned to the length and character of each stroke.
- **Stack** — the same dasharray technique; the work is in the asset and the timing, not the
  code.
- **Params** — per-path duration; gaps between strokes (30–80ms, to imply the pen lifting);
  overall pace.
- **Use when** — a signature, a logotype, one hand-lettered phrase.
- **Don't use when** — the lettering is thick or filled. Drawing traces an outline, so a bold
  brush script gets drawn as a hollow contour, which looks wrong.
- **Reduced motion** — the finished signature.
- **Performance** — a handful of paths; trivial. The asset size is the real cost, since
  detailed lettering can produce heavy path data.
- **Gotchas** — the paths must be ordered and directed as a hand would write them, which
  almost never matches the export order from the design tool. This is asset preparation, and
  it is most of the work. The name must also exist as real text somewhere for screen readers.
- **References** — https://allsvgicons.com/blog/animating-svg-paths-css-framer-motion/

---

### Diagram build (`svg.diagram-build`)

- **One line** — a diagram assembles itself in the order you should read it.
- **What the reader sees** — An architecture diagram, empty. A box fades in, then a connector
  draws from it to a second box which appears as the line arrives, then the next, until the
  whole system is on screen — built in dependency order over three or four seconds. Because
  each element arrives when the narration reaches it, the diagram teaches its own structure:
  you understand what connects to what because you watched it connect. It is the single most
  useful application of SVG animation and the least fashionable.
- **Mechanism** — a timeline combining node fades, stroke draws on connectors, and label
  reveals, sequenced deliberately. Usually scroll-driven so the reader controls the pace.
- **Stack** — a timeline library for the sequencing; the individual moves are all elsewhere in
  this file.
- **Params** — per-step duration (0.3–0.6s); overlap; whether it plays on a clock or scrubs
  with scroll (scroll is better — reading pace varies).
- **Use when** — technical explanation, architecture pages, documentation, onboarding.
- **Don't use when** — the diagram is decorative, or the reader needs to study it. An
  animated diagram must end in a stable, complete state they can look at as long as they like.
- **Reduced motion** — the complete diagram immediately.
- **Performance** — many small paths; fine. Scrubbing is the more expensive mode, and still
  cheap at this scale.
- **Gotchas** — the diagram must be complete and readable without JavaScript, with the
  animation as an enhancement. And it needs a text equivalent: a sequence of fading boxes
  conveys nothing to a screen reader, so the same structure has to exist as a list or a
  description.
- **References** — https://effect-labs.com/en/pages/blog/svg-animations.html

---

### Counter arc (`svg.counter-arc`)

- **One line** — an arc sweeps to a value while a number counts alongside it.
- **What the reader sees** — A gauge: an arc sweeping from its start position around to a
  point, with a figure in the middle rising in step so the two reach their destination
  together. The pairing is what makes it work — the arc gives you the proportion at a glance
  and the number gives you the precision, and because they move together you read them as one
  fact rather than two.
- **Mechanism** — dash-offset arc animation synchronised with a numeric counter on the same
  timeline.
- **Stack** — the dash technique plus a proxy-value counter; one timeline drives both.
- **Params** — sweep duration (1–1.5s); shared easing (essential — if the arc and the number
  use different curves they visibly desynchronise); arc extent.
- **Use when** — dashboards, scores, single headline metrics.
- **Don't use when** — the value updates frequently in real time; a constantly re-animating
  gauge is unreadable.
- **Reduced motion** — final arc and final number, no sweep.
- **Performance** — trivial.
- **Gotchas** — the two must share a timeline, not two animations of the same duration, or
  they drift. The numeric label needs tabular figures or it reflows every frame.
- **References** — https://www.boundev.ai/blog/svg-animation-css-tutorial-guide

---

## Family notes

**`pathLength="1"` first.** It normalises any path to a 0–1 length and removes the
measurement step that made line drawing a JavaScript problem. Most of this file gets simpler
because of that one attribute.

**Know which properties are cheap.** `transform` and `opacity` composite on the GPU;
`stroke-dashoffset`, `fill`, `stroke` and animated `d` all repaint on the CPU. That ordering
is the whole performance story here, and it explains why an icon animation is free and an
animated illustration is not.

**Morphing is a data problem, not an animation problem.** Matching point counts and command
types, and choosing the right starting correspondence, is where the work is. Resolve the
shape index in development and hard-code it.

**Prepare the asset.** Path order, path direction and start points come out of design tools
essentially at random, and they determine what a drawing animation looks like. Half the
effort in this family is in the SVG file, not the code.

**SVG animation is silent to assistive technology.** A diagram that builds, a gauge that
sweeps and a signature that writes all convey exactly nothing without a text equivalent.
`<title>`, `aria-label`, or real adjacent text — not optional.
