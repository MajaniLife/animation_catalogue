# Pointer, hover & cursor

Motion that answers the pointer. The only family whose input device is optional — which is
the fact that determines everything about how it must be built.

**The rule that governs the whole family: hover is not an interface, it is a bonus.** A
pointer exists on desktops and laptops. It does not exist on phones or tablets, does not
exist for keyboard-only users, and does not exist for screen reader users. Any information
or affordance that is *only* reachable by hovering is unreachable for a large share of
people. So every entry here is decoration layered on top of something that already works, or
it is a bug.

Three practical consequences, and they are non-negotiable:

1. **Gate on capability, not on screen size.** `@media (any-hover: hover) and (pointer: fine)`
   tells you a precise pointing device exists. Width tells you nothing — a touchscreen laptop
   is wide, and a mouse plugged into a tablet is narrow.
2. **Bind `focus` wherever you bind hover.** If hovering reveals something, focusing must
   reveal the same thing, or keyboard users get a page with holes in it.
3. **Never replace the system cursor casually.** Operating systems let people enlarge,
   recolour and outline the cursor for accessibility reasons; a custom cursor overrides that
   choice. Screen-magnifier users are a specific casualty — the zoom window follows the
   pointer, so an effect that appears under the cursor obscures exactly what they were
   trying to read.

The reason this family exists at all is that pointer response is the cheapest way to make an
interface feel *alive* rather than printed. An element that acknowledges the cursor before
you click it communicates that the page is listening.

---

### Underline draw (`pointer.underline-draw`)

- **One line** — a rule sweeps in under a link on hover and out the way the pointer left.
- **What the reader sees** — Approach a link and a thin line grows beneath it, sweeping in
  from the left in about a quarter of a second. Move away and it does not simply disappear —
  it retracts toward the side you left from, so the line's exit mirrors the direction of your
  travel. That directional detail is the whole thing: a symmetric fade-out feels
  mechanical, while a wipe that follows the pointer feels like the interface tracked you. It
  is six lines of code and among the highest polish-per-byte effects on the web.
- **Mechanism** — a pseudo-element rule with `transform: scaleX()` and a
  `transform-origin` that flips between enter and leave.
- **Stack** — CSS transitions. The origin flip needs one line of JavaScript, or two rules
  keyed on a class.
- **Params** — thickness (1–2px); duration (0.25–0.35s); offset below the baseline; easing
  (a strong ease-out).
- **Use when** — text links, nav items, anywhere an underline is the correct affordance.
- **Don't use when** — the underline is the *only* signal that something is a link. Links in
  body copy should be underlined at rest.
- **Reduced motion** — the rule appears instantly on hover and focus rather than sweeping.
  Keep the state change; drop the travel.
- **Performance** — a scaled pseudo-element. Free.
- **Gotchas** — pseudo-elements cannot be animated by JavaScript animation libraries, so this
  one stays CSS; drive its duration from a custom property if you want it token-controlled.
  Bind `focus`/`blur` alongside `pointerenter`/`pointerleave` or keyboard users see nothing.
- **References** — https://user-a.co.il/en/accessible-development/hover-state-accessibility-guide

---

### Magnetic pull (`pointer.magnetic`)

- **One line** — an element leans toward the cursor as it approaches.
- **What the reader sees** — As the pointer nears a button, the button shifts a few pixels
  toward it — not enough to move it out of place, but enough that the two seem attracted.
  The closer you get, the further it leans, and it eases back with a slight lag when you
  leave. The result is that the button feels like it *wants* to be clicked, and it also
  becomes fractionally easier to hit, since it is moving toward your pointer. It is a small
  physical fiction and one of the most-copied effects of the last five years.
- **Mechanism** — on pointer move, take the offset from the element's centre, scale it by a
  distance falloff, and translate. Setters are created once and reused.
- **Stack** — any library with a persistent setter (`quickTo` and equivalents), or raw
  transform writes.
- **Params** — radius (100–200px); strength (0.2–0.4 of the offset — above 0.5 the element
  visibly detaches from its layout); return duration (0.4–0.6s, deliberately laggy).
- **Use when** — a primary call to action, a small number of them.
- **Don't use when** — the element is in a dense cluster; neighbouring magnets that overlap
  read as instability rather than attraction.
- **Reduced motion** — no displacement.
- **Performance** — cheap *if built correctly*. **Never create a tween inside the pointer
  handler** — that allocates an object per frame and is the standard way this becomes janky.
  Create setters once; read `getBoundingClientRect` on resize, not per move.
- **Gotchas** — a magnetised element with a large hit area can drift far enough that the
  cursor is no longer over it, which fires `pointerleave` and produces a stutter loop. Keep
  strength low, or attach the listener to a stationary parent.
- **References** — https://web.dev/learn/css/cursors-and-pointers

---

### Label stack swap (`pointer.stack-swap`)

- **One line** — a label slides out while an identical copy slides in behind it.
- **What the reader sees** — Hover a project title and the text moves upward and out of a
  clipped box while a second copy of the same text rises from below to take its place. For a
  fraction of a second both are in motion, and then the label is exactly where it was. Nothing
  has changed — the word is the same word — but the element has *responded*, with a crispness
  that a colour change cannot match. It is the signature hover of portfolio and studio sites,
  and the DOM giveaway is that the same string appears twice in the markup.
- **Mechanism** — duplicate the label inside a clipping container; on hover translate the
  original out and the copy in, on the same axis and the same easing.
- **Stack** — CSS for the two-copy version; any library if you generate the duplicate at
  runtime.
- **Params** — axis (Y reads as a mechanical flip, X as a slide); duration (0.25–0.35s);
  easing (shared by both halves, or the swap looks broken).
- **Use when** — project titles, list rows, nav items — anywhere with a strong horizontal
  rhythm.
- **Don't use when** — the label wraps to two lines. The absolutely-positioned copy assumes a
  single line and will misalign.
- **Reduced motion** — no swap; use a colour or weight change instead.
- **Performance** — two composited elements per label. Fine in a list of a dozen.
- **Gotchas** — the duplicate must be `aria-hidden`, or screen readers announce every label
  twice. Generating duplicates at runtime replaces the element's text content, which destroys
  anything nested inside it — links, spans, icons.
- **References** — https://user-a.co.il/en/accessible-development/hover-state-accessibility-guide

---

### Fill wipe (`pointer.fill-wipe`)

- **One line** — colour floods a button from the edge the pointer entered.
- **What the reader sees** — Hover a button and its background fills with colour — not
  fading uniformly, but sweeping in from the side you approached from. Come in from the left
  and it fills left to right; from below and it rises. Leave, and it drains back out the side
  you left by. Because the direction is computed from your actual entry point, the button
  appears to know where you came from, which is a much stronger sense of responsiveness than
  a symmetric fade for essentially the same cost.
- **Mechanism** — on `pointerenter`, compare the pointer position to the element's bounds to
  pick the nearest edge, set a transform-origin accordingly, and scale a background layer in.
- **Stack** — CSS transitions plus a few lines of JavaScript for the entry-edge computation.
  Hardcoding the origin is the cheap version and loses most of the effect.
- **Params** — duration (0.3–0.4s); shape (rectangular wipe, or a circle expanding from the
  exact entry point); easing.
- **Use when** — buttons, cards, calls to action.
- **Don't use when** — the fill colour does not maintain contrast with the label. The text
  must be readable in both states *and* mid-transition.
- **Reduced motion** — the fill state applies instantly.
- **Performance** — one composited layer scaling. Cheap.
- **Gotchas** — the label needs to sit above the fill layer and often needs its own colour
  transition timed slightly differently, or it is briefly illegible as the fill passes under
  it. Bind focus, and give the focus state a deterministic direction since there is no entry
  edge for a keyboard.
- **References** — https://css-tricks.com/next-level-css-styling-for-cursors/

---

### Cursor dot & ring (`pointer.cursor-dot`)

- **One line** — a custom cursor object follows the pointer, usually with a lagged companion.
- **What the reader sees** — A small dot tracks the pointer exactly, and a larger ring
  follows a beat behind, catching up when you stop. Over links the ring expands and softens.
  The lag is what carries the effect — a ring that tracks perfectly reads as a graphic, while
  one that trails reads as something with mass being pulled along. It is the most recognisable
  signature of design-led sites, and the most frequently criticised technique in this
  catalogue.
- **Mechanism** — a fixed-position element positioned by two setters at different durations
  (dot fast, ring slow), driven by pointer move; hover targets detected by delegation.
- **Stack** — any library with persistent setters. Roughly thirty lines total.
- **Params** — ring lag (0.3–0.5s); size; hover scale (2–3×); whether the native cursor is
  hidden.
- **Use when** — a design-led site, on pointer-capable devices, **with the native cursor left
  visible**.
- **Don't use when** — the site is a tool, or you intend to hide the system cursor. Hiding it
  overrides deliberate OS accessibility settings for cursor size, colour and outline, and it
  actively harms screen-magnifier users, whose zoom window follows the pointer.
- **Reduced motion** — remove the custom cursor entirely; the lag *is* the effect.
- **Performance** — two transform writes per pointer event. Cheap if setters are created
  once; use delegation rather than a listener per link.
- **Gotchas** — the element must be `pointer-events: none` or it eats every click. Remove it
  from the DOM on touch rather than leaving it invisible. It must never be the only hover
  affordance, because keyboard users have no cursor to see. And when the pointer leaves the
  window, hide it — a cursor stranded at the edge of the viewport looks broken.
- **References** — https://dbushell.com/2025/10/27/custom-cursor-accessibility/ ·
  https://ericwbailey.website/published/dont-use-custom-css-mouse-cursors/ ·
  https://stiftelsenfunka.org/about-us/columns/the-curse-of-the-custom-cursor/

---

### Cursor state morph (`pointer.cursor-state`)

- **One line** — the custom cursor changes shape to say what the thing under it does.
- **What the reader sees** — The cursor ring drifts over an image and becomes a filled disc
  with the word "view" inside it. Over a draggable carousel it stretches into a horizontal
  arrow pair. Over a link it shrinks to a point. The cursor stops being decoration and starts
  being a label — the affordance moves from the element to the pointer itself, which lets an
  image grid stay completely clean because the instruction only appears where you are
  looking.
- **Mechanism** — the cursor object reads a data attribute on the hovered target and
  transitions between defined states.
- **Stack** — builds on the cursor dot; states are usually a small map of size, colour and
  label.
- **Params** — state set (keep it under four); transition duration (0.2s); label typography.
- **Use when** — image-led work where you want to strip visible UI, on pointer devices.
- **Don't use when** — the state label is the only place the affordance is communicated.
  Everything it says must exist elsewhere for touch and keyboard users.
- **Reduced motion** — no morph; either a static cursor or none.
- **Performance** — same as the cursor dot plus a text swap.
- **Gotchas** — it inherits every objection to custom cursors and adds one: information now
  lives in the cursor. If "view" only appears there, then on a phone nobody knows the image
  opens. Test the page with the cursor code removed and confirm it still explains itself.
- **References** — https://dbushell.com/2025/10/27/custom-cursor-accessibility/

---

### Image follow (`pointer.image-follow`)

- **One line** — a preview image trails the cursor over a list of links.
- **What the reader sees** — A plain list of project names. Move the pointer down it and a
  thumbnail appears near the cursor, lagging slightly behind as it follows, swapping to a new
  image as you cross from one row to the next. The images are never all on screen at once —
  only the one you are pointing at — so the page keeps the calm of a text list while still
  being visual. The lag makes the image feel dragged along rather than pinned, which is what
  keeps it from looking like a tooltip.
- **Mechanism** — a single positioned preview element following the pointer via lagged
  setters; row hover swaps its source and fades between them.
- **Stack** — any library with setters; one shared element rather than one per row.
- **Params** — lag (0.2–0.4s); offset from the cursor; crossfade duration (0.15–0.25s);
  scale on enter.
- **Use when** — an index of projects, articles or case studies where thumbnails would
  otherwise clutter the layout.
- **Don't use when** — the images are the primary content — then show them — or on touch,
  where the effect cannot exist at all.
- **Reduced motion** — no follow. Either show a static thumbnail per row or none.
- **Performance** — preload the images or the first hover on each row shows an empty frame.
  One element reused beats one per row by a wide margin.
- **Gotchas** — the preview must not intercept pointer events or it interrupts the very hover
  driving it. On touch, ship an entirely different layout with visible thumbnails rather than
  a dead list.
- **References** — https://web.dev/learn/css/cursors-and-pointers

---

### Tilt (`pointer.tilt-3d`)

- **One line** — a card tips in perspective, tracking the cursor across its face.
- **What the reader sees** — Move over a card and it tilts as though it were a physical panel
  hinged at its centre — the near corner dipping toward you, the far corner receding, the
  angle following your position across its surface. Often a soft highlight tracks the pointer
  as if light were catching the tilt. Leave, and it eases flat. It reads as a solid object
  under glass, and it is one of very few effects that make a rectangle feel like a thing
  rather than a region.
- **Mechanism** — pointer position mapped to `rotateX`/`rotateY` about the card's centre,
  with `perspective` on the parent.
- **Stack** — CSS transforms plus a small pointer handler.
- **Params** — maximum angle (5–12°; past 15 it looks like a novelty); perspective
  (800–1500px); return duration (0.4s); optional glare layer.
- **Use when** — product cards, feature tiles, a small grid.
- **Don't use when** — the card holds a lot of text; rotated type softens and shifts. Also not
  on long lists — a page of independently tilting cards is exhausting.
- **Reduced motion** — flat, no tilt.
- **Performance** — a 3D transform per hovered card. Fine one at a time.
- **Gotchas** — the classic containing-block trap: a 3D-transformed ancestor becomes the
  containing block for `position: fixed` descendants, silently breaking fixed overlays inside
  it. Reading layout on every pointer move is the other reliable way to make it stutter —
  cache the rect.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/perspective

---

### Repulsion field (`pointer.repulsion`)

- **One line** — many small elements scatter away from the cursor.
- **What the reader sees** — A field of items — dots, icons, letters, thumbnails — sitting in
  a grid. As the pointer moves through them they push aside, the nearest fleeing furthest and
  the ones at the edge of its influence barely stirring, then drift back once you have passed.
  It is unmistakably a *field*: the cursor becomes a force, and the page becomes a surface
  that responds to it. Almost always decorative, almost always memorable, and almost always
  more expensive than it looks.
- **Mechanism** — one ticker loop for the whole field, computing each item's distance and
  displacement per frame from cached positions.
- **Stack** — any library, or raw transforms. The architecture matters far more than the
  library.
- **Params** — radius; strength; return easing; item count.
- **Use when** — a deliberate playful moment: a 404, an about page, one hero.
- **Don't use when** — the items are content the user needs to click. Moving targets are a
  motor-accessibility problem, not a delight.
- **Reduced motion** — static field.
- **Performance** — **the classic jank bug lives here**: recomputing every item's bounding
  rect every frame. Cache positions and refresh them on resize only, and run one loop for the
  whole field rather than a handler per item.
- **Gotchas** — items must return to exact original positions or the field drifts over
  time. Cap the item count; a hundred repelling elements will drop frames on any machine.
- **References** — https://web.dev/learn/css/cursors-and-pointers

---

### Spotlight (`pointer.spotlight`)

- **One line** — a soft light follows the cursor across a surface.
- **What the reader sees** — A dark card or section is lit by a diffuse glow centred on the
  pointer, so the area you are looking at is brighter and the rest falls away. Move across and
  the light moves with you, revealing a border gradient or a faint pattern as it passes. It
  gives a flat dark surface the feeling of being physically lit, and on a bordered card the
  travelling highlight on the edge is the detail that sells it.
- **Mechanism** — a radial gradient positioned by two custom properties updated from pointer
  coordinates, either as a background or a mask.
- **Stack** — CSS custom properties plus a pointer handler; no library needed.
- **Params** — radius (200–400px); intensity; whether it lights the fill, the border, or both.
- **Use when** — dark-themed cards, pricing tables, feature grids.
- **Don't use when** — the surface is light; the effect needs contrast to register at all.
- **Reduced motion** — a static highlight or none. There is no travel to preserve.
- **Performance** — updating a gradient position repaints the element every frame. On a grid
  of twenty cards, only update the one being hovered — a naive implementation updates all of
  them.
- **Gotchas** — write pointer coordinates to custom properties rather than restyling the
  gradient string each frame. Registering the properties with `@property` lets the browser
  interpolate them and avoids a repaint storm.
- **References** — https://css-tricks.com/next-level-css-styling-for-cursors/

---

### Skew on move (`pointer.skew-move`)

- **One line** — an element leans in proportion to pointer speed.
- **What the reader sees** — Drag the pointer quickly across a card and it skews slightly in
  the direction of travel, snapping back upright as you slow. It is the pointer equivalent of
  scroll velocity skew: what you notice is not the shape change but a sense that the element
  has inertia and is being pulled through the movement.
- **Mechanism** — pointer velocity between frames, clamped, mapped to `skewX`, decaying to
  zero.
- **Stack** — a small handler plus a decaying setter.
- **Params** — maximum skew (3–6°); decay (0.3s); clamp (essential).
- **Use when** — one expressive element on a site that is already expressive.
- **Don't use when** — anything else on the page skews. Compounded skews read as mush, and
  this specifically fights scroll velocity skew.
- **Reduced motion** — none.
- **Performance** — cheap; the velocity computation is the only extra work.
- **Gotchas** — skew rasterises text through a different path; keep the magnitude small and
  the decay fast or type looks blurry during interaction.
- **References** — https://web.dev/learn/css/cursors-and-pointers

---

### Hover intent delay (`pointer.hover-intent`)

- **One line** — the effect waits to be sure you meant it.
- **What the reader sees** — Sweep the pointer across a row of menu items and nothing opens.
  Rest on one for a moment and its panel appears. Move diagonally toward the open panel and it
  stays open even though the pointer briefly crosses a different item. Nothing about this is
  visible as an animation; what the user perceives is simply that the menu does not flicker
  and does not slam shut when they reach for it. It is the least visible entry in this
  catalogue and the one that most improves how an interface feels to use.
- **Mechanism** — a short open delay, a longer close delay, and optionally a check on the
  pointer's direction of travel toward the open panel.
- **Stack** — plain JavaScript timers; no library required.
- **Params** — open delay (100–200ms); close delay (300–500ms, longer than open); optional
  direction tolerance.
- **Use when** — any hover-revealed panel: dropdowns, mega-menus, tooltips, previews.
- **Don't use when** — the hover effect is purely decorative; a decorative state should be
  instant.
- **Reduced motion** — unchanged. This is timing, not motion.
- **Performance** — free.
- **Gotchas** — an asymmetric delay is the point; equal open and close delays feel sluggish
  one way and twitchy the other. Content revealed on hover must also be reachable by keyboard
  and must stay open while it has focus — and per WCAG it should be dismissible and hoverable
  without disappearing.
- **References** — https://www.w3.org/WAI/GL/low-vision-a11y-tf/wiki/Metadata_On_Hover ·
  https://user-a.co.il/en/accessible-development/hover-state-accessibility-guide

---

## Family notes

**The gate is `(any-hover: hover) and (pointer: fine)`.** Not viewport width. Everything in
this file should be inside that query, or removed at runtime when it fails.

**Everything hover does, focus must do too.** The test is simple: unplug the mouse and tab
through the page. If information disappeared, the page is broken, not minimal.

**Leave the system cursor alone.** People configure cursor size, colour and contrast at the
OS level for real reasons. A custom cursor that hides the native one overrides an
accessibility setting to buy an aesthetic — and the magnifier case, where the zoom window
follows the pointer and the effect obscures what is being read, is a genuine failure rather
than a trade-off.

**Create setters once.** Every effect here runs on pointer move, which fires at display
frequency. Allocating a tween per event is the single most common performance mistake in
this family; caching layout reads and refreshing them on resize is the second.
