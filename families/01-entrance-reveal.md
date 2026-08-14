# Entrance & reveal

Content arriving for the first time — on page load, on scroll into view, or when a state
change puts something on screen that was not there before.

This is the family people reach for first and think about least. Its real job is not
decoration but **hierarchy**: the order in which things arrive tells the reader what to
look at first, and a page where everything arrives at once has thrown that away. The
second job is covering latency — an entrance gives the eye something to do during the
fifty milliseconds that would otherwise read as a jump.

The failure mode is uniform: one reveal applied to every element on the page, at the same
duration, triggered at the same threshold. That produces a page that feels like it is
loading permanently. Entrances should be **rationed** — a handful of deliberate arrivals
beats forty identical ones.

A note on triggering, which applies to every scroll-triggered entry below. Three
mechanisms exist, and they are not interchangeable: **CSS `view-timeline`** (free,
compositor, no JavaScript, but see the Firefox flag in `STACKS.md`); **IntersectionObserver**
(tells you an element entered, nothing more); and **`ScrollTrigger.batch()`** (creates one
trigger per target but batches their callbacks inside a short interval, so a grid staggers
as one group rather than as N independent races). For a grid of cards, the batching
distinction is the whole difference between a coordinated cascade and visual noise.

---

### Fade rise (`entrance.fade-rise`)

- **One line** — the default entrance: content fades up a short distance into place.
- **What the reader sees** — The element is not there, then it is, having travelled maybe
  twenty or thirty pixels upward while becoming opaque. The distance is short enough that
  you register arrival rather than travel — nothing flies in from off-screen. It
  decelerates into position over roughly half a second, slowing hard at the end so it
  settles rather than stops. Alone it is nearly invisible as an effect; you notice its
  absence more than its presence. Applied to a stack of elements in sequence it reads as
  the page composing itself top to bottom, calm and unremarkable in the way a well-set
  page is unremarkable.
- **Mechanism** — `opacity` 0→1 and `transform: translateY(20–32px)`→0, ease-out, on a
  fixed duration. Triggered by load or by viewport entry.
- **Stack** — CSS with `@starting-style` or a class toggle; any library. Free tier.
- **Params** — distance (24px default; past ~60px it becomes a slide, which reads
  cheaper); duration (0.5s; below 0.25s it flickers, above 0.9s it drags); ease-out always
  — ease-in-out on an entrance looks hesitant.
- **Use when** — you want hierarchy without anyone noticing the mechanism; body content,
  cards, list items.
- **Don't use when** — the element is the single hero moment of the page. This is the
  baseline, not the statement.
- **Reduced motion** — appear at final opacity with no travel. Keep the stagger delays if
  you like; sequenced appearance without movement still communicates order.
- **Performance** — compositor-only. The cheapest entrance there is. Safe in bulk.
- **Gotchas** — animating from `opacity: 0` on an element that also has `visibility:
  hidden` will not transition unless you handle the discrete property. Elements starting
  at `translateY` inside an `overflow: hidden` parent can trigger scrollbars mid-flight.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/@starting-style ·
  https://web.dev/blog/baseline-entry-animations

---

### Mask rise (`entrance.mask-rise`)

- **One line** — text rises from behind a hard edge, as if into a window.
- **What the reader sees** — A line of type appears from below an invisible boundary and
  slides up into view, and crucially you never see it cross that boundary — it emerges,
  fully formed, from nothing. The line beneath follows about a tenth of a second later,
  then the next, so a paragraph or headline assembles top to bottom over half a second or
  so. Each line decelerates hard on arrival. The impression is of type being *set* rather
  than faded in: composed, editorial, slightly theatrical. On a three-line headline it is
  the single most reliable way to make a page feel deliberately designed.
- **Mechanism** — each line wrapped in a clipping container (`overflow: clip`), the line
  itself animated `translateY(100%)`→0 with a per-line stagger. Requires the text to be
  split into line elements first.
- **Stack** — GSAP SplitText with `mask: "lines"` emits the clipping wrappers for you
  (free since 2025, rewritten in 3.13). Hand-rolled: wrap each line yourself, which breaks
  the moment the text rewraps at a different width.
- **Params** — unit (lines default; words for a livelier feel; characters is a different
  entry); stagger (0.08–0.1s per line; below 0.04s reads as one block); distance (100% of
  line height — less exposes the fact that it is a slide).
- **Use when** — headlines, pull quotes, statements. The signature entrance of editorial
  and agency work.
- **Don't use when** — the text is long. Mask-rising a full paragraph makes people wait to
  read. Also avoid on anything that reflows frequently.
- **Reduced motion** — text present at full opacity, no travel, no split. Skip the split
  entirely rather than splitting and not animating — an unnecessary split still costs you
  DOM nodes and screen-reader risk.
- **Performance** — compositor-only once split, but the split itself is a synchronous DOM
  rewrite. Splitting a page of text at load is a real cost.
- **Gotchas** — **split after `document.fonts.ready`** or lines break against the fallback
  font and the whole effect measures wrong. A resize must re-split *and* rebuild the
  animation, or the tween is orphaned and text stays parked off-screen. Split text must
  carry an `aria` strategy or it becomes gibberish to screen readers. Never apply
  `text-wrap: balance` to a split target.
- **References** — https://gsap.com/blog/3-13/ · https://webflow.com/blog/gsap-becomes-free

---

### Clip wipe (`entrance.clip-wipe`)

- **One line** — content is uncovered by a moving edge rather than faded in.
- **What the reader sees** — A hard line sweeps across the element — left to right, or top
  to bottom — and content is present behind it as it passes. Nothing moves and nothing
  fades; the element is simply progressively exposed, like a curtain drawn back or paint
  rolled across a wall. Because the content itself is stationary, it reads as *revealing
  something that was already there* rather than as bringing something in, which is a
  subtly different and more confident feeling. Over about 0.6 seconds on a large block it
  is stately. Under 0.3 it is a snap, and reads as a UI response rather than a reveal.
- **Mechanism** — `clip-path: inset()` or `polygon()` animated from fully clipped to fully
  open, single easing, no opacity change. Diagonal wipes use `polygon()` with two moving
  points.
- **Stack** — CSS keyframes handle it; any library for coordination. Free.
- **Params** — direction; duration (0.6s); ease (a strong ease-out or a custom curve —
  linear reads mechanical); edge softness (a gradient mask instead of a hard clip changes
  the character entirely).
- **Use when** — images, section blocks, anything with a strong rectangular boundary.
- **Don't use when** — the element has irregular shape or transparency; the wipe edge
  gives away the bounding box.
- **Reduced motion** — fully unclipped immediately.
- **Performance** — `clip-path` is GPU-accelerated in modern engines but still repaints
  the affected area. Full-viewport wipes are noticeably more expensive than a card-sized
  one; test on mid-range Android before committing to one at hero scale.
- **Gotchas** — animating `clip-path` between shapes with different point counts does not
  interpolate; keep the point count identical. `overflow: hidden` on an ancestor plus a
  clip on the child produces double-clipping that is hard to debug visually.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/clip-path

---

### Clip expand (`entrance.clip-expand`)

- **One line** — an image frame opens outward while the image inside scales down to meet it.
- **What the reader sees** — The image starts inset — a smaller rectangle floating inside
  the space it will eventually occupy — and the frame grows outward to its full size. At
  the same time the picture inside is scaled slightly larger than its frame and shrinks
  back to true size, so the content appears to settle *into* the opening rather than being
  stretched by it. The two movements are opposite and simultaneous, and that opposition is
  the whole trick: it produces a sense of depth and of the image being placed. Tied to
  scroll it is the single most recognisable "expensive site" move of the last five years,
  which is also its problem.
- **Mechanism** — `clip-path: inset(N%)`→`inset(0)` on the frame, simultaneously
  `scale(1.15)`→`scale(1)` on the inner image. Usually scrubbed against scroll rather than
  played on a clock.
- **Stack** — GSAP ScrollTrigger with `scrub` is the standard; CSS `view-timeline` can do
  it where support allows.
- **Params** — inset (15–20%; above 30% the image is a stamp); inner scale (1.1–1.2; must
  exceed the inset or edges show); scroll range (start "top 90%", end "top 35%").
- **Use when** — editorial imagery, project thumbnails, anywhere the image is the content.
  Scrubbed against scroll rather than triggered, this is `scroll.clip-expand`.
- **Don't use when** — the page has more than a handful of images. The move loses all
  effect by the fourth repetition, and it is now common enough to read as a template.
- **Reduced motion** — image at full size, no clip, no scale.
- **Performance** — two animated properties on a large raster is the expensive case.
  Scrubbed, it runs on every scroll frame — this is where mid-range mobile actually
  suffers. Consider playing it once on entry instead of scrubbing.
- **Gotchas** — the inner scale must be *greater* than the clip inset or you reveal the
  frame's background at the edges mid-animation. `object-fit: cover` plus a scaling
  transform can shift the visual crop as it animates.
- **References** — https://gsap.com/blog/3-13/

---

### Scale settle (`entrance.scale-settle`)

- **One line** — the element arrives fractionally too large or too small and resolves.
- **What the reader sees** — The element appears at around 96% of its size and grows into
  place while fading in, over roughly 0.4 seconds. The scale change is small enough that
  you cannot name it — nothing appears to zoom — but the element reads as *approaching*
  rather than appearing, and gains a small amount of presence. Reversed (starting slightly
  large and settling down) the feeling changes: it reads as the element arriving with
  momentum and being caught. Both are subtle to the point of subliminal, and both stop
  working the moment the scale delta is large enough to notice, at which point it becomes
  a zoom and reads cheap.
- **Mechanism** — `transform: scale(0.96)`→1 with `opacity`, ease-out, 0.35–0.5s.
- **Stack** — CSS. Free.
- **Params** — scale delta (0.94–0.98; below 0.9 it is a different, worse effect);
  duration (0.4s); transform-origin (centre by default; changing it changes the whole
  character).
- **Use when** — modals, popovers, cards, anything appearing over existing content.
- **Don't use when** — the element contains text at small sizes; sub-pixel scaling makes
  type shimmer during the transition.
- **Reduced motion** — appear at final scale and opacity.
- **Performance** — compositor-only. Cheap.
- **Gotchas** — scaling a container scales its children's rendered text, which can cause
  visible reflow-like shimmer on low-DPI screens. Avoid scaling anything containing an
  input the user may be about to focus.
- **References** — https://developer.chrome.com/blog/entry-exit-animations

---

### Blur focus (`entrance.blur-focus`)

- **One line** — content resolves from out-of-focus to sharp.
- **What the reader sees** — The element begins soft and indistinct, as though the camera
  has not found focus yet, and sharpens over about half a second while fading in. Usually
  paired with a small upward drift. The effect is genuinely photographic — it borrows the
  language of a lens rather than of a slide — and it gives text in particular a quality of
  *coming into legibility* that no other entrance reproduces. It is also the entrance most
  likely to make a mid-range phone stutter, and the one most often shipped without anyone
  checking that.
- **Mechanism** — `filter: blur(8px)`→`blur(0)` with `opacity`, usually plus a small
  `translateY`.
- **Stack** — CSS; gate it behind an intensity setting if you have one.
- **Params** — blur radius (6–10px; above 16px content is unreadable long enough to feel
  broken); duration (0.5–0.7s — blur needs longer than a fade to read as focus rather than
  as a glitch).
- **Use when** — a single hero element, a featured image, one moment per page.
- **Don't use when** — you have more than two or three on a screen, or you are targeting
  low-end mobile. This is the expensive one.
- **Reduced motion** — sharp immediately, no blur.
- **Performance** — **not compositor-friendly.** Animated `filter: blur()` forces
  repeated re-rasterisation of the element and its subtree; cost scales with painted area
  and blur radius. A full-width blurred hero on mid-range Android is a reliable way to
  drop frames.
- **Gotchas** — blur on a parent blurs everything inside it including fixed-position
  descendants. Some engines snap the final frame when the radius reaches zero, producing a
  visible pop — animate to `blur(0.01px)` if you see it, or accept the snap.
- **References** — https://developer.chrome.com/blog/entry-exit-animations

---

### Batch stagger (`entrance.batch-stagger`)

- **One line** — a grid of items reveals as one coordinated group when it comes into view.
- **What the reader sees** — You scroll and a grid of cards arrives — not all at once, and
  not one by one either, but as a wave: the first card, then the next a beat later, then
  the next, across four or six items in roughly half a second, each fading up a short
  distance. Because they are timed as a group rather than triggered individually, the wave
  has a direction and a rhythm; it reads as one gesture. The version that triggers each
  card independently looks superficially similar and is subtly wrong — cards fire at
  slightly different scroll positions and the result is a scatter rather than a cascade.
- **Mechanism** — viewport entry detection across a set, then a single tween with a
  per-item stagger applied to whichever items entered together.
- **Stack** — `ScrollTrigger.batch()` collects the elements that entered within a short
  interval and hands them to one callback, which is exactly the primitive this needs.
  IntersectionObserver can be made to do it with manual buffering. CSS `view-timeline`
  cannot batch — each element gets its own timeline.
- **Params** — stagger (0.08–0.12s); batch max (4–6 — beyond that the last item in a wave
  arrives late enough to feel forgotten); start threshold ("top 85–88%").
- **Use when** — card grids, project lists, logo walls, anything repeated. For single
  elements rather than sets, `scroll.reveal-enter` is the same idea without the batching.
- **Don't use when** — items are large enough that only one fits on screen; batching has
  nothing to batch.
- **Reduced motion** — all items visible, no travel; the stagger can be dropped entirely.
- **Performance** — one observer per item is fine; the batching is a scheduling
  concern, not a rendering one. The animation itself should stay compositor-only.
- **Gotchas** — the big one: **handle entering from both directions and reconcile on
  refresh.** An element only fires "entered" when scrolled into downward. An anchor jump,
  a fast flick, or scrolling back up to a section that was skipped can leave items stuck
  at opacity 0 permanently, and it looks like a rendering bug rather than a motion one.
  Any refresh — font settle, image load, resize — restores trigger state silently without
  firing callbacks, so state must be reconciled there too.
- **References** — https://codepen.io/GreenSock/pen/NWGPxGZ ·
  https://github.com/greensock/gsap-skills/blob/main/skills/gsap-scrolltrigger/SKILL.md

---

### List cascade (`entrance.list-cascade`)

- **One line** — list items arrive in reading order, one after another.
- **What the reader sees** — Rows appear in sequence from the top, each a fraction of a
  second after the last, fading and lifting slightly. Where a grid cascade reads as a
  wave, a list cascade reads as *enumeration* — the page counting its contents. With eight
  or ten rows and a tight stagger the whole list completes in under a second and the
  effect is brisk and organised. Stretch the stagger and the same animation becomes
  laboured, with the reader waiting on the last item; this is the effect most often ruined
  by generosity.
- **Mechanism** — identical to fade-rise with an index-based delay.
- **Stack** — CSS with a custom property per item, or any library's stagger.
- **Params** — stagger (0.04–0.08s for lists — tighter than grids); total ceiling (keep
  the whole cascade under ~1s regardless of count; past 12 items switch to a fixed total
  duration divided across items).
- **Use when** — navigation menus, search results, settings lists, the contents of a
  freshly opened panel.
- **Don't use when** — the list is long, virtualised, or the user is likely to be waiting
  for a specific row. Never stagger anything the user is hunting through.
- **Reduced motion** — all rows present immediately.
- **Performance** — cheap, but a 200-row cascade means 200 animations; cap the count.
- **Gotchas** — cascading a list that the user just filtered makes the interface feel
  slower than it is. Animate on first appearance only, not on every re-render.
- **References** — https://explainx.ai/skills/dylantarre/animation-principles/scroll-animations

---

### Curtain (`entrance.curtain`)

- **One line** — a solid panel covering the content slides away to expose it.
- **What the reader sees** — The screen, or a section of it, is covered by a solid block of
  colour. The block moves — up, or apart from the centre — and the content beneath is
  revealed already in place, static and complete. Nothing about the content itself
  animates; the drama is entirely in the covering. Full-screen and slow, this is the
  gesture that opens a fashion or agency site: authoritative, a little theatrical, and
  entirely about withholding. Its cost is measured in seconds of the visitor's attention,
  which is why it belongs on a first load and almost nowhere else.
- **Mechanism** — an overlay panel animated by `transform: translateY(-100%)` or a
  `scaleY` from a top origin, with the underlying content untouched.
- **Stack** — CSS or any library. Typically the exit half of a preloader.
- **Params** — duration (0.8–1.2s — this is one of the few effects that earns a long
  duration); ease (a strong ease-in-out or expo; linear is lifeless here); panel count
  (one, or several offset by a small delay for a slatted effect).
- **Use when** — first load, page transitions, deliberately gating a reveal.
- **Don't use when** — on repeat visits, or anywhere the user is returning to content they
  have already seen. Then it is a toll booth.
- **Reduced motion** — remove the panel immediately with no travel; do not simply
  fade it, since a long fade is still a long wait.
- **Performance** — one full-viewport composited layer. Cheap to animate, but it forces a
  large paint when it leaves.
- **Gotchas** — the panel must not trap focus or intercept clicks after it has visually
  left; remove it from the DOM rather than leaving it at `translateY(-100%)`. If it owns
  the entrance sequence, everything downstream must wait on it — an entrance that fires
  behind a curtain is an entrance nobody sees.
- **References** — https://developer.chrome.com/blog/entry-exit-animations

---

### Split part (`entrance.split-part`)

- **One line** — two halves separate to reveal what is between them.
- **What the reader sees** — A single surface splits along its centre and the two halves
  travel in opposite directions — left and right, or up and down — exposing the content
  behind. The symmetry is the point: because both halves move equally, the eye is drawn to
  the centre line, which is precisely where the revealed content sits. It has a mechanical,
  shutter-like quality, closer to a camera aperture or a pair of doors than to a fade. Done
  in half a second it is crisp; done slowly it becomes ceremonial, and works exactly once
  per site.
- **Mechanism** — two panels translated in opposite directions, or a single `clip-path`
  polygon with two edges moving apart.
- **Stack** — CSS for the two-panel version; SVG or `clip-path` for the aperture version.
- **Params** — axis; duration (0.5–0.9s); ease (ease-in-out; both halves must share it or
  the symmetry breaks).
- **Use when** — a single hero reveal, a page transition, an intentional "opening".
- **Don't use when** — repeatedly. The symmetry that makes it striking also makes it
  memorable, and a memorable effect used six times is an annoying effect.
- **Reduced motion** — content exposed immediately, panels removed.
- **Performance** — two composited layers, cheap. The large paint on completion is the cost.
- **Gotchas** — sub-pixel gaps at the seam show the background through a hairline during
  the animation; overlap the panels by a pixel. Panels left in the DOM at their end
  positions can extend the scrollable area sideways.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/clip-path

---

### Rule draw (`entrance.rule-draw`)

- **One line** — hairline rules extend from one end to the other as sections arrive.
- **What the reader sees** — A thin line, the kind that separates sections in an editorial
  layout, grows from its left edge across the width of the container in about half a
  second. It is a small thing — a one-pixel line — but it converts a static divider into
  an event, and when several rules draw in sequence down a long page they knit the
  sections together as a set. Paired with a label arriving above it, the combination reads
  as a heading being *ruled off*, which is the most typographically literate entrance in
  this family and among the cheapest.
- **Mechanism** — `transform: scaleX(0)`→1 with `transform-origin: left`, ease-out.
- **Stack** — CSS. Free.
- **Params** — duration (0.5–0.8s); origin (left for LTR reading; centre for a symmetric
  feel); delay relative to the text it accompanies (rule slightly after the label reads
  best).
- **Use when** — section dividers, underlines beneath headings, table rules, timeline rails.
- **Don't use when** — the rule is decorative filigree rather than structure; then it is
  motion for its own sake.
- **Reduced motion** — rule at full width immediately.
- **Performance** — a scaled one-pixel element. Effectively free.
- **Gotchas** — scaling a 1px line can produce a blurry sub-pixel edge mid-animation on
  fractional-DPI displays. Animate `scaleX` on a wrapper with a solid background rather
  than on a bordered element.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/transform-origin

---

### Flip in (`entrance.flip-3d`)

- **One line** — the element rotates in from an angle, as if hinged.
- **What the reader sees** — The element starts tipped away from you along its horizontal
  axis, its top edge further away, and swings forward into the flat plane of the screen
  over about half a second while fading in. Because it involves real perspective, it reads
  as a physical object rotating rather than as a graphic changing — closer to a card being
  laid on a table than to anything else in this family. Kept to fifteen or twenty degrees
  it is a discreet lift. Pushed to ninety it becomes a full flip, which is a distinctly
  2014 flavour and reads as dated now.
- **Mechanism** — `perspective` on the parent, `transform: rotateX(15deg)`→0 on the child,
  with `opacity`.
- **Stack** — CSS. Free.
- **Params** — angle (12–20°); perspective distance (600–1200px — smaller is more extreme);
  transform-origin (top edge for a hinge; centre for a tilt).
- **Use when** — a card appearing over a surface, a dropdown, a toast.
- **Don't use when** — the element contains a lot of text; rotated type is briefly
  unreadable and the anti-aliasing shifts.
- **Reduced motion** — flat and visible immediately.
- **Performance** — compositor-only, but 3D transforms promote layers; do not apply to
  dozens of elements at once.
- **Gotchas** — a 3D-transformed ancestor becomes the containing block for
  `position: fixed` descendants, which silently breaks fixed headers and modals inside it.
  This one costs people a full day, reliably.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/perspective

---

### Skeleton swap (`entrance.skeleton-swap`)

- **One line** — placeholder shapes are replaced by the real content they were standing in for.
- **What the reader sees** — Grey blocks in the approximate shape of the content sit in
  place, often with a slow shimmer travelling across them. When the data arrives the blocks
  do not disappear — they become the content, crossfading in place over a fifth of a second
  while keeping identical position and size. Because nothing moves, there is no jump, and
  the transition from "loading" to "loaded" is almost unnoticeable. Done badly — skeletons
  a different size from the real content — the swap produces exactly the layout shift it
  was meant to prevent, and is worse than no skeleton at all.
- **Mechanism** — crossfade between placeholder and content, no transform. The shimmer is a
  separate looping gradient animation — see `micro.skeleton-shimmer`.
- **Stack** — CSS for both halves; framework state decides the swap.
- **Params** — crossfade duration (0.15–0.25s — fast; this is a state change, not a
  reveal); shimmer period (1.5–2s; faster reads as anxious); minimum display time (~300ms,
  to avoid a flash when data arrives instantly).
- **Use when** — content whose shape is known before its data arrives: feeds, tables, cards.
- **Don't use when** — you cannot predict the shape. A skeleton that guesses wrong is a lie
  that the layout then corrects in front of the user.
- **Reduced motion** — drop the shimmer loop entirely (it is continuous ambient motion),
  keep the crossfade or make it instant.
- **Performance** — the shimmer is a permanently running animation; if a screen holds
  thirty skeletons, that is thirty loops competing with the very fetch you are waiting on.
- **Gotchas** — measure the real content and match the skeleton to it, not the reverse.
  Skeletons must be hidden from assistive technology, and the loading state announced
  properly instead.
- **References** — https://web.dev/blog/baseline-entry-animations

---

### Starting-style entry (`entrance.starting-style`)

- **One line** — the CSS-native way to animate something appearing from `display: none`.
- **What the reader sees** — Functionally this is fade-rise or scale-settle; what is
  distinctive is *where* it works. A dialog, popover or tooltip that previously snapped
  into existence now fades and scales in, including elements entering the top layer, with
  no JavaScript timing tricks. To the reader it is simply that things which used to appear
  abruptly now arrive.
- **Mechanism** — `@starting-style` supplies the "from" values for an element's first
  style update, since transitions do not otherwise fire on initial render or on a change
  out of `display: none`. `transition-behavior: allow-discrete` lets discrete properties
  like `display` participate.
- **Stack** — pure CSS. Replaces the `setTimeout`-then-add-a-class pattern entirely.
- **Params** — whatever you are transitioning; the mechanism itself has no knobs.
- **Use when** — dialogs, popovers, tooltips, anything toggling `display`.
- **Don't use when** — you need choreography across several elements; this is a
  single-element primitive.
- **Reduced motion** — wrap the transition declaration in a media query; the element still
  appears, without travel.
- **Performance** — free; it is the browser's own transition machinery.
- **Gotchas** — three real ones. **Specificity**: `@starting-style` has equal specificity to
  the original rule, so it must come *after* it or be nested inside, otherwise it is
  overridden and silently does nothing. **Transitions only** — it has no effect on CSS
  animations. **Exit is not the mirror of entry**: animating out of `display: none` also
  needs the `overlay` property for top-layer elements, which was **not** Baseline as of
  that guidance, so exit animations remain the weaker half. `overlay.top-layer-exit`
  documents the working technique. `@starting-style` and
  `allow-discrete` became Baseline in August 2024 (Chrome 117+, Firefox 129+, Safari 17.4+).
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/@starting-style ·
  https://web.dev/blog/baseline-entry-animations ·
  https://developer.chrome.com/blog/entry-exit-animations

---

### Hero sequence (`entrance.hero-sequence`)

- **One line** — the composite: a first screen whose elements arrive in a scripted order.
- **What the reader sees** — Not one effect but a short film. A small label appears first,
  low-key, at the top. The headline follows, arriving line by line from behind its own
  edges. A rule draws beneath it. The supporting sentence fades up, then the scroll cue
  last, slightly after everything else has settled, so it reads as an invitation rather
  than part of the furniture. The whole thing takes about a second and a half and has a
  clear beginning and end. What the reader takes from it is not any individual movement but
  an *order of importance* — they now know what this page is about and where to look next,
  which is the entire justification for the expense.
- **Mechanism** — a single timeline with elements at explicit offsets, usually overlapping
  rather than strictly sequential; a shared easing family holds it together.
- **Stack** — a timeline library earns its weight here. Hand-counting delays across six
  elements is where CSS stops being the right tool.
- **Params** — total duration (1.2–2s; past 2.5s visitors start scrolling through it);
  overlap (start each element before the previous finishes — gaps read as stalling); one
  ease across the whole sequence.
- **Use when** — the first screen of a site whose first impression matters commercially.
- **Don't use when** — the page is a tool people use daily. The second visit is where a
  hero sequence turns into a toll.
- **Reduced motion** — everything present immediately. The hierarchy must survive in the
  layout itself, not only in the timing.
- **Performance** — the sequence competes with font loading and the largest image on the
  page. Gate it on fonts being ready, or lines will be measured against the fallback and
  the whole choreography lands wrong.
- **Gotchas** — if a preloader owns the intro, everything here must wait on it explicitly.
  Anything scroll-triggered below the fold must not fire while the hero is still playing.
  And the sequence should be skippable — a visitor who scrolls immediately should not be
  fighting an animation.
- **References** — https://gsap.com/blog/3-13/ ·
  https://gsapvault.com/blog/how-to-animate-on-scroll

---

## Family notes

**Ration them.** Fifteen entries here does not mean fifteen entrances per page. A strong
page uses two or three: one composite for the hero, one workhorse for body content, one
distinctive move reserved for a single moment.

**One easing family.** The fastest way to make separate entrances feel like one system is
to share a curve and a duration scale across all of them. Mixed easings read as mixed
authorship.

**The trigger is half the effect.** Threshold, direction handling and refresh reconciliation
decide whether a reveal feels responsive or broken. See the gotchas under
`entrance.batch-stagger` — that failure mode accounts for more "the animation didn't work"
bug reports than every easing decision combined.

**Reduced motion is not "off".** The end state must be reachable and the hierarchy must
survive. If an entrance is the only thing communicating order of importance, the layout is
under-designed.
