# Scroll-driven

Motion whose progress is tied to scroll position rather than to a clock. The reader is the
playhead: scroll forward and the animation advances, stop and it stops, scroll back and it
reverses.

That reversibility is the defining property, and it changes how these effects should be
designed. A time-based entrance plays once and is over; a scroll-driven effect is a
**mapping**, and it must make sense at every point in its range — including halfway, and
including backwards. Effects that only look right at their end state are the most common
mistake in this family.

**Trigger versus scrub — the distinction everything else depends on.** A *scroll-triggered*
animation fires when you cross a threshold and then plays on its own clock: a 0.4s entrance
that runs to completion whether you keep scrolling or not. A *scroll-driven* (scrubbed)
animation has no duration of its own — its progress is the scroll offset, so it advances
exactly as fast as you scroll. Reveals should be triggered; parallax, progress bars and
pinned sequences should be scrubbed. Using the wrong one is why some sites feel like they
are fighting your scroll wheel.

**The accessibility position, stated plainly.** Parallax and large-field scroll motion can
trigger vertigo, dizziness and nausea in people with vestibular disorders — a group
estimated at around 35% of adults over 40 having experienced some vestibular dysfunction.
Honouring `prefers-reduced-motion` is not a nicety here; guidance in 2026 is either to
avoid parallax or to make it switchable, and WCAG 2.1 expects the media query to be
respected for any animation. Every entry below states what it becomes when motion is
reduced, and for this family that branch is the difference between a site someone can use
and one that makes them ill.

**Three implementations, not interchangeable.** CSS `scroll-timeline`/`view-timeline` runs
on the compositor and costs nothing, but see the Firefox flag in `STACKS.md`.
IntersectionObserver reports crossings only — no progress value. ScrollTrigger gives
continuous progress, pinning and a refresh lifecycle, and is the only one of the three that
handles the hard cases.

---

### Parallax layer (`scroll.parallax-layer`)

- **One line** — elements move at different rates as you scroll, implying depth.
- **What the reader sees** — As the page scrolls, a background image drifts upward more
  slowly than the text over it, so the two separate: the text advances at the speed of your
  scroll while the background lags, and the gap between them reads unmistakably as
  distance. Nothing announces itself — there is no discrete animation to notice — but the
  section acquires a sense of space that a flat layout does not have. Push the rate
  difference too far and the illusion inverts: instead of depth you perceive the background
  sliding, which is both cheaper-looking and the point at which sensitive readers start to
  feel unwell.
- **Mechanism** — `translateY` (or `yPercent`) scrubbed against scroll progress, with a
  different multiplier per layer. Percentage units keep it proportional across breakpoints.
- **Stack** — CSS `scroll-timeline` where support permits; GSAP ScrollTrigger with `scrub`
  otherwise. Some cases are pure CSS with `background-attachment`, though that has its own
  mobile problems.
- **Params** — depth multiplier (0.1–0.3 of scroll distance for background layers; above
  0.5 it reads as sliding, not depth); number of layers (two or three; more is mush);
  direction.
- **Use when** — hero sections, section breaks, imagery that is atmosphere rather than
  content.
- **Don't use when** — the moving element carries information the reader must track, or the
  section is dense with text.
- **Reduced motion** — **all layers static.** This is the entry where that matters most.
  Do not substitute a smaller parallax; remove it.
- **Performance** — compositor-only if you stay on `transform`. Never animate
  `background-position` for this; it repaints every frame.
- **Gotchas** — parallax on an element also inside a pinned section double-counts the
  scroll and drifts. Layers must be tall enough to cover their travel or an edge appears —
  the classic "gap at the bottom of the hero". `background-attachment: fixed` is effectively
  broken on iOS and janky elsewhere.
- **References** — https://webflow.com/accessibility/checklist/task/avoid-parallax-effects ·
  https://www.webaxe.org/vestibular-issues-parallax-design/ ·
  https://web.dev/articles/prefers-reduced-motion

---

### Progress bar (`scroll.progress-bar`)

- **One line** — a thin bar across the top fills as you move through the page.
- **What the reader sees** — A hairline sits at the very top edge of the window, filling
  from left to right in exact proportion to how far down the document you are. It moves
  only when you move, tracks precisely, and never draws attention to itself — you notice it
  when you want to know how much is left, which is exactly when you should. On a long
  article it converts an unknowable scroll into a measurable one. It is the cheapest
  credibility signal in this entire catalogue: three lines of code, and pages feel
  considered.
- **Mechanism** — `scaleX` from 0 to 1 with a left transform-origin, mapped to document
  scroll progress.
- **Stack** — CSS `animation-timeline: scroll()` is the ideal implementation — no
  JavaScript, runs on the compositor. ScrollTrigger's `onUpdate` where support is a concern.
- **Params** — thickness (2–4px); position (top reads as progress, bottom as a player
  control); colour (accent).
- **Use when** — long-form reading, documentation, single-page sites.
- **Don't use when** — the page is short enough to see the scrollbar do the same job.
- **Reduced motion** — keep it. It is an indicator, not decoration, and it moves only in
  response to the user's own action.
- **Performance** — a single `scaleX`. Effectively free, and free *and* off-main-thread in
  the CSS version.
- **Gotchas** — measure against `scrollHeight - innerHeight`, not `scrollHeight`, or the bar
  never reaches full. Recalculate on resize and after images load, or the mapping is wrong
  on exactly the pages that need it most.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/animation-timeline ·
  https://developer.chrome.com/blog/scroll-triggered-animations

---

### Pinned sequence (`scroll.pin-sequence`)

- **One line** — a section sticks in place while its contents advance through steps.
- **What the reader sees** — You scroll and a section arrives, then stops — it holds
  centred in the window while the page apparently continues to scroll. During that hold its
  contents change: a diagram builds a step at a time, or a headline swaps for the next
  point, each transition advancing as you scroll further. After the last step the section
  releases and the page moves on normally. The effect is that scrolling has become a
  *timeline control* rather than a movement, which is genuinely powerful for explaining a
  process — and genuinely disorienting when the reader did not expect the page to stop
  moving.
- **Mechanism** — a ScrollTrigger with `pin: true` and a scrubbed timeline whose steps are
  positioned across the pin duration. The trigger goes on the **timeline**, never on a
  child tween.
- **Stack** — GSAP ScrollTrigger. CSS `position: sticky` plus a scroll timeline covers
  simple cases; complex sequencing does not have a CSS equivalent.
- **Params** — pin duration (expressed as scroll distance — roughly one viewport height per
  step); number of steps (3–5; beyond that readers assume the page has broken); whether the
  pinned element is the section or an inner wrapper.
- **Use when** — a process with genuinely sequential steps, a product feature walkthrough.
- **Don't use when** — the content is a list rather than a sequence, or on a page people
  visit to find one thing quickly.
- **Reduced motion** — do not pin. Lay the steps out vertically as ordinary stacked content
  — which is also the correct no-JavaScript fallback, and a good test of whether the content
  justified the pinning in the first place.
- **Performance** — pinning inserts spacer elements and rewrites layout at the pin
  boundaries; it forces reflow on refresh. Keep pinned subtrees small.
- **Gotchas** — pinning inside a transformed ancestor breaks position calculations. Nested
  ScrollTriggers on parent and child fight over the same scroll. `pinSpacing` interacts
  badly with sticky siblings. And refresh order matters: pinned sections must recalculate
  in document order, or later triggers measure against pre-pin positions.
- **References** — https://gsap.com/docs/v3/Plugins/ScrollTrigger/ ·
  https://mcpservers.org/agent-skills/greensock/gsap-scrolltrigger

---

### Horizontal rail (`scroll.horizontal-rail`)

- **One line** — vertical scrolling drives horizontal movement through a pinned panel.
- **What the reader sees** — A section pins, and then your downward scrolling starts moving
  content sideways: a row of cards, or a series of full-width panels, travelling right to
  left across the window at exactly the rate you scroll. It is the strongest reframing of
  scroll in this family — down means across — and for a portfolio or a timeline it is
  genuinely elegant. It is also the effect most likely to break someone's expectations: on
  a trackpad it can feel like the page has been hijacked, and on mobile it competes with
  the native horizontal gesture.
- **Mechanism** — pin the section; animate `x`/`xPercent` of an inner track scrubbed
  against vertical scroll. Elements *inside* the track that need their own triggers use
  `containerAnimation`, which lets ScrollTrigger watch the horizontal animation's progress
  instead of the window's.
- **Stack** — GSAP ScrollTrigger. There is no pleasant CSS equivalent.
- **Params** — scroll distance (total horizontal travel maps to a vertical distance you
  choose — too short feels frantic); panel count (3–6); snap behaviour.
- **Use when** — a small set of items that genuinely reads as a sequence, and where you can
  afford the interaction cost.
- **Don't use when** — content must be linkable, searchable in place, or reachable quickly.
- **Reduced motion** — a normal vertical or horizontally-scrollable list with native
  scrolling. Do not simply speed it up.
- **Performance** — one long transform; cheap. The pinning and the refresh cost dominate.
- **Gotchas** — three that bite in order. **`ease: "none"` is mandatory** on the horizontal
  tween — any other easing desynchronises scroll position from horizontal position, and it
  is the most common mistake here. **Pinning and snapping are not available on
  containerAnimation-based triggers** — pin the outer panel instead, and animate inside it.
  **Don't animate the trigger element horizontally**, or offset its start/end to compensate.
  Also noted upstream: a containerAnimation trigger may render at the wrong position on the
  very first frame after a refresh, correcting itself once you scroll.
- **References** — https://codepen.io/GreenSock/pen/dymJaXg ·
  https://gsap.com/community/forums/topic/36495-scrolltrigger-in-scrolltrigger-containeranimation-help-and-some-other-aspects/

---

### Clip expand on scroll (`scroll.clip-expand`)

- **One line** — an image opens out of a narrow slot as it enters the viewport.
- **What the reader sees** — An image begins as a letterbox — a wide, shallow band — and as
  you scroll it opens vertically to its full height, the picture inside settling from
  slightly oversized to true scale. Because both edges move outward from the centre, the
  image appears to be *unfolding* into the space rather than growing into it. Tied to scroll,
  the opening tracks the reader's pace exactly, so it feels responsive rather than
  performed. This is the same move as the entrance-family clip expand, distinguished only by
  being scrubbed rather than triggered — which is the difference between the image reacting
  to you and the image playing at you.
- **Mechanism** — `clip-path: inset()` scrubbed from a large inset to zero, with an inverse
  `scale` on the inner image.
- **Stack** — ScrollTrigger with `scrub`; CSS `view-timeline` where supported.
- **Params** — inset; inner scale (must exceed the inset); scroll range (a short range —
  about half a viewport — keeps it crisp).
- **Use when** — one or two feature images per page.
- **Don't use when** — every image on the page does it. It is a strong move with a low
  repetition tolerance.
- **Reduced motion** — image fully open, no scale.
- **Performance** — the expensive scrubbed case: `clip-path` plus `scale` on a large raster,
  recalculated every scroll frame. Test on mid-range Android.
- **Gotchas** — as with its entrance-family twin, the inner scale must exceed the inset or
  the frame's background shows at the edges. Combining with parallax on the same element
  double-transforms it.
- **References** — https://gsap.com/docs/v3/Plugins/ScrollTrigger/

---

### Sticky stack (`scroll.sticky-stack`)

- **One line** — cards pile up, each sticking as the next slides over it.
- **What the reader sees** — A card scrolls up and stops near the top of the window. The
  next card scrolls up behind it and comes to rest slightly lower, overlapping the first;
  the one beneath it sits a little further down again. By the fourth card you are looking at
  a stack, each edge visible, the earliest cards slightly scaled down and dimmed as though
  receding. Then the whole pile scrolls away together. It is a deck being dealt into your
  hand, and it makes a set of items feel like a collection rather than a list.
- **Mechanism** — `position: sticky` with a per-card top offset, optionally with scrubbed
  `scale` and `opacity` on the cards behind.
- **Stack** — the sticky part is pure CSS and works everywhere; the scale/dim refinement
  needs a scroll timeline or ScrollTrigger.
- **Params** — offset step (8–20px per card); count (4–6; beyond that the earliest cards are
  a smear); scale step (0.02–0.04 per card behind).
- **Use when** — a small set of comparable items: pricing tiers, principles, steps.
- **Don't use when** — cards contain long content. Anything below the fold of a stuck card
  is unreachable while it is stuck.
- **Reduced motion** — plain stacked cards, no sticking, no scale.
- **Performance** — sticky is cheap and native. The scale/dim layer is the only real cost.
- **Gotchas** — `position: sticky` fails silently if any ancestor has `overflow: hidden`
  (or `auto`), which is the single most common reason this does not work and produces no
  error. Sticky also does not compose with smooth-scroll libraries that translate the page
  wrapper — the transform-based ones break it, which is precisely why Lenis wraps native
  scroll instead.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/position ·
  https://lovable.dev/guides/scrolling-designs-patterns-when-to-use

---

### Velocity skew (`scroll.velocity-skew`)

- **One line** — content leans in proportion to how fast you are scrolling.
- **What the reader sees** — Scroll gently and nothing happens. Flick hard and the whole
  content column skews a few degrees, as though the page has inertia and is being dragged
  through the movement; it springs back to upright the instant you stop. The faster the
  scroll, the further it leans. You do not consciously see the skew — what you perceive is
  weight, a sense that the page has mass. It is one of a small number of effects that
  respond to *how* you scroll rather than to where you have got to, and that responsiveness
  is what makes sites feel physical.
- **Mechanism** — read scroll velocity each frame, clamp it, map to `skewY`, and decay to
  zero when velocity drops.
- **Stack** — ScrollTrigger exposes velocity directly. Hand-rolling means differencing
  scroll positions per frame, which is doable but noisier.
- **Params** — maximum skew (2–5deg; past 8 it looks broken); decay (0.3–0.6s back to
  zero); clamp (essential — an unclamped flick produces a 40-degree lurch).
- **Use when** — once, on one wrapper, on a site whose register is already expressive.
- **Don't use when** — the page contains a lot of text, or anything else already skews or
  rotates. Compounded skews are visual mush.
- **Reduced motion** — no skew at all. Whole-viewport rotation is a vestibular trigger.
- **Performance** — a transform on a large wrapper promotes a big layer and repaints text
  through a rotated rasterisation path; heavier than it looks.
- **Gotchas** — apply it to exactly one element, never nested. Skewing a container that
  holds `position: fixed` children breaks their positioning. Text is noticeably softer while
  skewed, so keep the magnitude low and the decay quick.
- **References** — https://gsap.com/docs/v3/Plugins/ScrollTrigger/

---

### Scroll snap (`scroll.snap-sections`)

- **One line** — scrolling settles onto section boundaries instead of stopping anywhere.
- **What the reader sees** — You scroll a short distance and release, and instead of coming
  to rest wherever momentum left you, the page glides the remaining distance and locks a
  section neatly into the window. Every stop is a composed frame. On a portfolio of
  full-bleed images it feels like turning pages; on a marketing page with sections of
  different heights it feels like the page is arguing with you about where to stop.
- **Mechanism** — CSS `scroll-snap-type` on the container and `scroll-snap-align` on the
  children. The JavaScript equivalent intercepts the scroll end and animates to the nearest
  boundary.
- **Stack** — CSS scroll snap is native, well supported and should be the first choice.
- **Params** — strictness (`proximity` is forgiving and usually right; `mandatory` is
  absolute and can trap users); alignment (start/centre).
- **Use when** — full-viewport sections of uniform height, image galleries, onboarding.
- **Don't use when** — sections are taller than the viewport. `mandatory` snapping with
  oversized content can make part of it unreachable — a genuine accessibility failure, not
  a nuisance.
- **Reduced motion** — `proximity` at most, or disable snapping entirely.
- **Performance** — native and free.
- **Gotchas** — snapping fights smooth-scroll libraries: two systems both deciding where
  the page should come to rest produces oscillation. Zoomed-in users and users with large
  text are exactly the people for whom sections exceed the viewport, so `mandatory` is a
  bigger risk than it appears.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/scroll-snap-type

---

### Marquee velocity (`scroll.marquee-velocity`)

- **One line** — a continuously scrolling band speeds up and changes direction with your scroll.
- **What the reader sees** — A band of large type drifts steadily sideways on its own. When
  you scroll down it accelerates in the same direction; scroll up and it slows, stops, and
  runs the other way; stop scrolling and it eases back to its idle drift. The band is always
  moving, but *how* it moves is a readout of what you are doing. That coupling is the entire
  effect — a static marquee reads as a 1990s artefact, and the same marquee reacting to
  scroll velocity reads as expensive.
- **Mechanism** — a seamless looping translation whose playback rate is multiplied by
  clamped scroll velocity, with direction flipped on scroll direction.
- **Stack** — a seamless loop helper plus ScrollTrigger velocity. The loop itself must
  duplicate its contents and wrap the transform, or it seams.
- **Params** — base speed; velocity boost multiplier (2–4×); settle time back to idle
  (~0.2s); whether direction flips.
- **Use when** — a band of client names, services, or a keyword strip between sections.
- **Don't use when** — the words matter individually and must be readable at rest.
- **Reduced motion** — paused, showing a static row.
- **Performance** — a permanent transform loop; cheap but never idle. Pause it off-screen.
- **Gotchas** — **it must ship a pause control.** Continuously auto-playing motion lasting
  more than five seconds falls under WCAG 2.2.2, and a marquee is the textbook case. Two
  systems writing the same timescale — a hover-pause and a velocity modifier, say — will
  fight unless one explicitly overwrites the other.
- **References** — https://www.w3.org/WAI/WCAG22/Understanding/pause-stop-hide.html

---

### Scrub video / sequence (`scroll.frame-sequence`)

- **One line** — scrolling scrubs through a frame-by-frame image sequence.
- **What the reader sees** — A product rotates, or an object assembles, and it does so in
  perfect lockstep with your scrolling — scroll half a section and you are halfway through
  the rotation; scroll back and it reverses exactly. It looks like video under your control
  rather than animation playing at you, and for showing a physical object it is the most
  convincing technique available on the web. It is also, by a wide margin, the heaviest
  thing in this file: a smooth sequence is dozens to hundreds of images, all of which must
  be there before it can be smooth.
- **Mechanism** — preload an image sequence, draw the frame matching scroll progress to a
  canvas. (Scrubbing a `<video>` element is the tempting alternative and is unreliable —
  seeking is not frame-accurate and stalls on network.)
- **Stack** — canvas plus ScrollTrigger; frames pre-encoded at build time.
- **Params** — frame count (60–150 for a full rotation; fewer stutters, more is
  unaffordable); resolution; preload strategy.
- **Use when** — a hardware product page where the object is the argument, and you have the
  budget.
- **Don't use when** — the payload matters, the connection may be poor, or the object could
  be shown in three photographs.
- **Reduced motion** — a single representative frame, or a static image.
- **Performance** — the worst case in this catalogue. Hundreds of decoded images in memory,
  a canvas redraw every scroll frame. Budget it explicitly; on mobile, consider not shipping
  it at all.
- **Gotchas** — decode frames ahead of use or the first pass stutters. Memory, not
  bandwidth, is usually the limit on mobile. Provide a meaningful still for anyone who never
  gets the sequence.
- **References** — https://gsap.com/docs/v3/Plugins/ScrollTrigger/

---

### Section colour shift (`scroll.theme-shift`)

- **One line** — the page's palette changes as you pass between sections.
- **What the reader sees** — You scroll from one section to the next and the background
  changes — not with a hard cut at the boundary, but as a crossfade over the last part of
  the outgoing section, text colour inverting with it so contrast holds throughout. Light
  becomes dark; the accent shifts. Because it happens across a range rather than at a line,
  you rarely catch the moment it changes, only that the page has a different temperature
  than it did a moment ago. It gives a long single-page site the feeling of moving through
  rooms.
- **Mechanism** — scroll ranges mapped to CSS custom property values on the root, either
  crossfaded or stepped at boundaries.
- **Stack** — ScrollTrigger's enter/leave callbacks toggling a class is the robust version;
  scrubbed colour interpolation is the smooth one.
- **Params** — transition length (a third of a viewport); which tokens change (background,
  text, accent — as a set, never individually).
- **Use when** — long single-page sites with distinct chapters.
- **Don't use when** — there are more than three or four palettes. Beyond that it reads as
  a slideshow.
- **Reduced motion** — the shifts can stay; a colour crossfade is not vestibular motion.
  Keep the transitions short and avoid any accompanying movement.
- **Performance** — repaints the viewport background; cheap, but do it on the root, not on
  every element.
- **Gotchas** — every intermediate state must pass contrast, not just the endpoints. Any
  fixed UI over the changing background — a nav, a cursor — needs to change with it or it
  will vanish against one of the palettes.
- **References** — https://web.dev/articles/prefers-reduced-motion

---

### Reveal on enter (`scroll.reveal-enter`)

- **One line** — the workhorse: elements animate in once when they first come into view.
- **What the reader sees** — Content below the fold is not there until you reach it; as
  each block arrives at roughly the lower fifth of the window it fades and lifts into place
  over about half a second. Scroll past quickly and things are simply there. Scroll slowly
  and the page appears to be laying itself out just ahead of your reading. It is the most
  used effect on the modern web and the most invisible when done correctly — its entire
  purpose is to prevent the page from feeling like a static wall.
- **Mechanism** — viewport entry crossing a threshold triggers a fixed-duration animation.
  Triggered, not scrubbed.
- **Stack** — CSS `view-timeline` (free, compositor, but check Firefox); IntersectionObserver;
  ScrollTrigger — see `entrance.batch-stagger` for why batching matters with sets.
- **Params** — threshold (start when the element is 10–15% into the viewport); duration
  (0.4–0.6s); once or every time (once, almost always).
- **Use when** — general page content, throughout.
- **Don't use when** — above the fold. Elements already in view on load should be part of
  the load sequence, not waiting for a scroll that may never come.
- **Reduced motion** — everything visible; no travel.
- **Performance** — cheap. The risk is quantity, not cost.
- **Gotchas** — the big one, again: **handle entry from both directions, and reconcile on
  refresh.** An element that was skipped past, jumped over by an anchor link, or scrolled up
  into from below may never fire its enter callback and will sit at opacity 0 permanently.
  Any refresh — fonts settling, images loading, a resize — restores trigger state silently
  without firing callbacks, so state must be reconciled there as well. This single class of
  bug accounts for most "the animation didn't run" reports.
- **References** — https://developer.chrome.com/blog/scroll-triggered-animations ·
  https://www.clcreative.co/blog/should-you-use-the-intersection-observer-api-or-gsap-for-scroll-animations

---

### Smooth scroll (`scroll.smooth-normalise`)

- **One line** — scrolling is interpolated rather than immediate.
- **What the reader sees** — The page follows your wheel with a slight lag and a glide,
  easing to a stop rather than halting. Everything acquires a small amount of weight and the
  motion feels continuous instead of stepped. On a design-led site it is the substrate that
  makes every other scroll effect feel expensive, because scrubbed animations inherit the
  smoothing. It is also the most contested technique here: it overrides a behaviour the
  user's operating system already defined for them, and some people find it disorienting or
  simply wrong.
- **Mechanism** — intercept scroll input, maintain an interpolated position, and drive the
  scroll from a single animation loop shared with the animation library.
- **Stack** — Lenis wraps native scroll rather than replacing it, so sticky positioning,
  anchors and native accessibility survive. Older transform-based smooth-scroll libraries
  move a wrapper element, which is what breaks `position: sticky` and in-page anchors.
- **Params** — lerp/duration (0.08–0.12 lerp is gentle; heavier values feel like syrup);
  whether to normalise wheel input across browsers.
- **Use when** — a design-led site with scrubbed effects that benefit from smoothed input.
- **Don't use when** — the site is a tool, a documentation set, or anything people navigate
  by keyboard and search.
- **Reduced motion** — **turn it off entirely** and restore native scrolling. Smoothing is
  itself motion the user did not ask for.
- **Performance** — one RAF loop. The rule is that there must be exactly one: run the smooth
  scroller on the same ticker as the animation library, or two loops drift a frame or two
  apart and every scrubbed effect reads as jitter.
- **Gotchas** — keyboard scrolling, `scroll-behavior: smooth`, anchor jumps and browser
  find-in-page all need to keep working; test them specifically, because they are the first
  things to break. Programmatic jumps must notify the scroll library or triggers measure
  against a stale position.
- **References** — https://lovable.dev/guides/scrolling-designs-patterns-when-to-use

---

## Family notes

**Scrub what is continuous, trigger what is discrete.** Position, size, colour and rotation
are continuous and belong on a scrub. An element arriving is discrete and belongs on a
trigger with its own duration. Scrubbing an entrance makes it stutter with the scroll wheel;
triggering a parallax makes it lurch.

**One scroll authority.** Smooth scroll, snap, pinning and a scrubbed timeline all believe
they decide where the page is. Pick one arbiter, run everything on a single ticker, and do
not let two libraries write the same scroll position.

**Refresh is a first-class event.** Fonts settle, images load, the viewport resizes, an
anchor jumps — each invalidates every measurement this family depends on. Effects must
recalculate on refresh *and* reconcile their visual state there, because refresh restores
trigger state without firing the callbacks that would have produced it.

**Reduced motion is not decoration here.** Parallax, velocity skew, frame sequences and
smooth scroll are all vestibular triggers. The reduced branch must remove the motion, not
scale it down — and the page must still make sense once it is gone.
