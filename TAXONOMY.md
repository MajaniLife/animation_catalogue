# Taxonomy

Six axes. Every entry can be placed on all six, and the combination is what makes the
catalogue searchable by intent rather than by library.

## 1. Family

The organising axis, one file each under `families/`.

| Family | What unites it |
|---|---|
| Entrance & reveal | Content arriving for the first time |
| Text & kinetic typography | Motion applied to letterforms and lines |
| Scroll-driven | Progress tied to scroll position rather than a clock |
| Pointer, hover & cursor | Motion that answers the pointer |
| Layout & shared-element | The same element persisting across a layout change |
| Page & route transitions | Motion across a navigation boundary |
| SVG & path | Vector-specific: stroke, morph, motion along a path |
| 3D & WebGL | Depth, camera, shader, real geometry |
| Physics, drag & gesture | Motion with momentum and constraint |
| Micro-interaction & feedback | Small motion that confirms a state change |
| Data-visualisation motion | Motion that carries quantitative meaning |
| Ambient & decorative | Continuous motion with no state to report |

## 2. What drives it

The single most useful discriminator, because it determines what can go wrong.

- **Time** — a fixed duration. Plays the same every time. Easiest to reason about.
- **Scroll** — progress mapped to scroll offset. Reversible by definition; needs its
  measurement refreshed whenever layout changes.
- **Pointer** — position or velocity of the cursor. Cannot exist on touch, so it always
  needs a non-pointer story.
- **State** — a change in the application. The animation is a consequence, not a decoration.
- **Physics** — a simulation with velocity and constraint. No fixed duration; you tune
  forces, not timings.
- **Ambient** — an unconditioned loop. Runs whether or not anyone is looking, which is
  both its charm and its cost.

## 3. Motion primitive

What is actually being changed. Ordered roughly by price.

| Primitive | Cost | Note |
|---|---|---|
| `transform`, `opacity` | Compositor-only | The safe pair. Everything else is a decision. |
| `clip-path`, `mask` | GPU, but repaints | Cheap enough at moderate size, expensive full-screen |
| `filter` (blur especially) | Expensive | Blur is the single most common cause of mobile jank |
| Layout properties | Worst | `width`, `height`, `top`, `margin` force reflow every frame |
| Path / stroke | Moderate | SVG-specific; `stroke-dashoffset` and morphing behave differently |
| 3D / shader | Own budget | Leaves the DOM's cost model entirely |
| Scroll position | Depends | Smooth-scroll hijacking has accessibility consequences |

## 4. Perceptual effect

What the motion does to the person reading. The axis that maps intent to entry, and the
one the decision tree in `INDEX.md` is built on.

- **Directs attention** — the eye goes somewhere specific.
- **Establishes hierarchy** — order of arrival implies order of importance.
- **Confirms causation** — this happened *because* you did that.
- **Creates depth** — layers at different rates read as space.
- **Signals quality** — no information, pure production value.
- **Occupies waiting** — the animation is what you look at while something else happens.
- **Carries data** — the motion itself is the information.

An effect that lands in none of these is decoration, which is legitimate — but it should
be labelled, because decoration is the first thing to cut under a performance budget.

## 5. Cost tier

- **Free** — CSS only, no library, no measurable bundle cost.
- **Cheap** — a few KB, or a library already present for other reasons.
- **Moderate** — a dedicated library or plugin, 10–40 KB gzipped.
- **Heavy** — a rendering stack of its own: WebGL, physics, a runtime player.

Cost is bundle *and* runtime. An effect that is free to download and repaints the whole
viewport every frame is not cheap.

## 6. Accessibility posture

- **Safe** — nothing to do beyond honouring reduced motion.
- **Needs a fallback** — the effect carries meaning that must survive without motion.
- **Needs a control** — the user must be able to stop it. Anything that auto-plays for
  more than five seconds is in this category by WCAG 2.2.2, marquees included.
- **Hazard** — parallax, zoom and large-field motion can trigger vestibular symptoms.
  These need reduced motion honoured properly, not decoratively.

## The intensity ladder

One site-level setting, three rungs. This is what stops a page being either dead or
exhausting — and it is a design decision, not a per-component one.

| | Restrained | Default | Maximal |
|---|---|---|---|
| Entrances | Opacity only | Mask-rise by line | Per-character cascade |
| Hover | Underline, colour | + magnetic, label swap | + fill wipes, image follow |
| Scroll | Batched reveal | + parallax, clip expand | + pinned sequences, velocity skew |
| Ambient | None | One marquee or gradient | Multiple simultaneous loops |
| Cursor | Native | Native | Custom cursor object |
| Transitions | Instant | Cross-fade | Full curtain choreography |
| Extra weight | 0 KB | ~10 KB | 40 KB+ |

**Reduced motion is not a fourth rung.** It is the runtime branch of whichever rung was
authored — the page still works, still communicates hierarchy, and simply arrives at its
end states without the travel.

Moving up a rung should be a deliberate act with a reason attached. The most common
failure in animated sites is not choosing the wrong effect; it is running three effects
at maximal on the same screen and calling the result "dynamic".
