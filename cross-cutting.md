# Cross-cutting

The four things that apply to every family: what motion becomes when it is switched off, what
it costs and how to measure that, how separate effects are made to feel like one system, and
the patterns that are reliably wrong.

---

## 1. The reduced-motion contract

`prefers-reduced-motion: reduce` is a user telling you that motion causes them a problem —
vestibular symptoms, migraine, nausea, or simply difficulty tracking a moving interface. It is
not a preference for a calmer aesthetic and it is not optional to honour.

**The rule that makes this tractable: reduced motion is not "animations off", it is
"animations arrive".** Every effect must have a defined end state that is reachable without the
travel, and the page must still communicate everything it communicated before. A reveal becomes
an appearance. A scrubbed effect becomes its end state. A loop becomes a still frame.

### What each family degrades to

| Family | Reduced-motion form |
|---|---|
| Entrance & reveal | Content present at final opacity and position. Keep sequencing delays if hierarchy depends on them; drop the travel. |
| Text & kinetic type | **Do not split the text at all.** Render it normally — an unnecessary split still costs DOM nodes and screen-reader risk. |
| Scroll-driven | Remove parallax and velocity effects entirely. Progress indicators stay (they move only in response to the user). Smooth scroll is turned off. |
| Pointer, hover & cursor | Remove displacement, trails and custom cursors. Keep state changes — colour, underline, fill — applied instantly. |
| Layout & shared-element | Instant layout change. If the motion was carrying "which item became which", announce the change instead. |
| Page & route | Navigate directly. No curtains, no slides, no exit animations. |
| SVG & path | Show the completed drawing. Stop dash flows. |
| 3D & WebGL | Static camera, or the poster-image fallback. Camera movement is among the strongest triggers available in a browser. |
| Physics & gesture | Direct manipulation stays — the user is doing it. Drop inertia, overshoot and spring-back. |
| Micro-interaction | **Keep the feedback**, drop the movement. A press still confirms with colour; a spinner becomes a pulse or a text status. |
| Data-visualisation | Final state immediately. The static chart must carry the same conclusion. |
| Ambient & decorative | Off. Design the still frame deliberately. |

### Implementation notes

- **Branch once, at the top.** One check that sets a flag or picks a variant, not a scattering
  of `if (reduced)` inside every effect. Scattered checks are how a page ends up 80% reduced.
- **Check it at runtime, not only in CSS.** JavaScript-driven motion needs the same branch, and
  it must respond if the preference changes mid-session.
- **Never gate content on an animation that will not run.** Anything starting at `opacity: 0`
  and revealed by JavaScript must have a path to visible when that JavaScript does not execute.
  This is the most common way a reduced-motion or no-JS visitor gets a blank page.
- **Consider `prefers-reduced-data` too.** It is the right signal for background video,
  prerendering and heavy texture loads.

### What is *not* covered by reduced motion

A user who has not set the preference can still be harmed by a page that shakes, flashes or
parallaxes aggressively. Content flashing **more than three times per second is a seizure
risk** and is a hard limit regardless of preferences. And WCAG 2.2.2 requires a pause control
for auto-playing motion over five seconds for *everyone*, not only for those who have opted
out.

---

## 2. Performance budget, and how to measure it

### The properties, ranked

1. **`transform` and `opacity`** — composited. The animation runs off the main thread; no
   layout, no paint. Everything you can express here, express here.
2. **`clip-path`, `filter`, `background-position`, `color`** — repaint, no reflow. Cost scales
   with painted area, which is why a card-sized blur is fine and a full-screen one is not.
3. **`width`, `height`, `top`, `left`, `margin`, `font-size`** — reflow every frame. Never
   animate these; use FLIP.
4. **SVG `d`, `stroke-dashoffset`, `font-variation-settings`** — CPU-bound geometry or glyph
   work per frame. Fine at icon scale, not at illustration scale.

### The metrics that matter in 2026

- **INP (Interaction to Next Paint)** — the responsiveness metric. **Under 200ms is good**,
  200–500ms needs improvement, over 500ms is poor. Heavy animation work is one of the most
  common causes of a failing INP, because an animation occupying the main thread delays the
  paint that answers the user's tap.
- **CLS (Cumulative Layout Shift)** — animation that changes layout mid-load contributes here.
  Reserve space; animate transforms, not geometry.
- **LoAF (Long Animation Frames)** — the API that tells you *which script* caused a long frame,
  where the older Long Tasks API only told you that one occurred. It is the right tool for
  finding animation-induced jank, and it already exposes a pending smoothness metric covering
  frame drops and scroll jank.

### How to actually measure

1. **Profile on the worst device you support**, not on a development machine. The gap between a
   laptop and a three-year-old mid-range phone is larger than any optimisation you will make.
2. **Watch dropped frames, not average FPS.** A steady 60 with four dropped frames at the start
   of every scroll feels worse than a steady 45.
3. **Check the compositor split.** If an animation shows paint or layout work per frame in the
   profiler, it is on the wrong property.
4. **Test with the CPU throttled** (4–6×) and the network throttled. Both change which effects
   are viable.
5. **Measure with the page doing its real work** — fetching, hydrating, decoding images. An
   animation profiled on an idle page tells you very little.

### A workable budget

- Ambient loops: **one** per page, paused off-screen.
- Simultaneous animated elements: keep under ~50; batching and instancing exist for a reason.
- Blur, backdrop-filter and full-screen `clip-path`: **one at a time**, never during scroll.
- WebGL: cap device pixel ratio at 1.5–2, keep draw calls under 100, dispose explicitly.
- Anything running during scroll gets the strictest scrutiny — scroll is when the main thread
  is already busiest.

---

## 3. Orchestration

Making separate effects feel like one system rather than a collection of tricks.

### Share a motion language

Three decisions, made once, applied everywhere:

- **A duration scale.** Something like instant 0.15s / fast 0.3s / base 0.6s / slow 1.2s. Every
  animation picks from the scale rather than inventing a number.
- **An easing family.** One ease-out for arrivals, one ease-in-out for movement between states,
  one dramatic curve reserved for the rare big moment. Mixed easings are the fastest way to
  make a page feel like it had several authors.
- **A distance scale.** Text rises by one amount, cards by another, and those two numbers are
  used everywhere.

The test that this is working: changing the ease and the base duration in one place should
visibly change the character of the entire page, coherently. If some part of it does not
change, something is hardcoding a value it should be reading.

### Sequence with overlap, not gaps

Elements in a sequence should start before the previous one finishes. Strict sequencing —
each element waiting for the last — produces a total duration that is the sum of the parts and
reads as stalling. Overlapping produces a total closer to the longest part and reads as one
gesture.

### Stagger by meaning, not by index

Stagger in reading order for lists, from the centre outward for radial compositions, from the
point of interaction for anything triggered by a click. A stagger that follows DOM order when
DOM order is not visual order looks like a bug.

### Gate the intro

If a preloader, a font load or a route transition owns the opening moment, everything
downstream must wait on it explicitly. An entrance that plays behind a curtain is an entrance
nobody sees, and it is a common cause of "the animation didn't run".

### Budget the whole interaction

Exit animation plus network plus entrance plus arrival stagger is what the user experiences.
Each may be defensible alone; together they can add a second to every navigation.

---

## 4. Anti-patterns

Things that are reliably wrong, collected from the twelve family files.

### Structural

- **Animating layout properties.** `width`, `height`, `top`, `left` every frame. Use FLIP.
- **Handling only `onEnter`.** Reveals that never fire for elements entered from below, jumped
  to by anchor, or flicked past — and refresh restores trigger state silently without firing
  callbacks. This single class of bug produces more "the animation is broken" reports than
  everything else combined.
- **Creating tweens inside pointer or scroll handlers.** Allocating an animation object per
  frame. Create setters once.
- **Reading layout inside an animation loop.** `getBoundingClientRect` per frame per element is
  the classic jank bug; cache and refresh on resize.
- **Two systems writing the same property.** A magnetic hover and a parallax on one element; a
  hover-pause and a velocity modifier on one marquee. Last writer wins and the result is
  unpredictable.
- **Two RAF loops.** A smooth-scroll library and an animation library on separate tickers
  desynchronise by a frame or two, and every scrubbed effect reads as jitter.

### Perceptual

- **Everything animates.** When forty elements have entrances, none of them means anything.
  Ration them.
- **Durations too long.** Feedback over 300ms feels broken; entrances over 800ms feel slow.
  The most common single fix to a bad motion system is halving every duration.
- **Ease-in on an entrance.** Things arriving should decelerate. Accelerating into position
  looks hesitant.
- **Animation as the only signal.** Colour-only toggles, silent toasts, tick-only
  confirmations. Every one of these is invisible to a screen reader.
- **Motion in peripheral vision while reading.** Floating shapes beside body copy, marquees
  next to paragraphs. Genuinely distracting, and for some readers disabling.

### Honesty

- **Charts that mislead.** Index-keyed transitions that silently change what an element means;
  eased time axes; interpolated values between real observations.
- **Optimistic UI without a visible rollback.** Silently reverting a failed action is worse
  than having waited.
- **Loading animation as theatre.** A curtain that hides a slow page does not make it fast; it
  makes the slowness deliberate.

### Accessibility

- **Hover-only affordances.** No focus binding, no touch path.
- **Split text without the ARIA repair.** `aria-label` on the container, `aria-hidden` on the
  fragments — and do not split text containing links.
- **Custom cursors that hide the native one.** Overrides an OS accessibility setting and
  actively harms screen-magnifier users.
- **Auto-playing motion without a pause control.** Over five seconds, alongside other content:
  WCAG 2.2.2, Level A. And once stopped it must not restart.
- **Anything flashing more than three times per second.** A seizure risk and a hard limit.

---

## The short version

Animate `transform` and `opacity`. Measure on a slow phone with the CPU throttled. Share one
duration scale and one easing family. Handle both scroll directions and reconcile on refresh.
Give every effect a defined end state that is reachable without motion, and make sure nothing —
content, feedback, or meaning — exists only inside the animation.
