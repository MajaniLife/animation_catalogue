# Layout & shared-element transitions

The same element persisting across a change in where it is. Not an entrance and not an exit
— a *continuity*: the thing you were looking at is still the thing you are looking at, in a
new position, size or context.

This family is what separates a page that redraws from an application that moves. When a
grid filters and the surviving cards glide to their new positions, the reader keeps track of
them; when it re-renders instantly, they have to re-read the whole grid. The animation is
carrying information — **which item became which** — and that is a genuinely different job
from decoration.

**FLIP is the technique underneath most of it.** First, Last, Invert, Play: record the
element's position before the change, let the DOM update, apply an inverse transform so it
*appears* to be where it was, then animate that transform to zero. The point is that layout
properties — `width`, `height`, `top`, `left` — are never animated at all. They change once,
instantly, and the movement is a compositor-friendly `transform`. Animating layout
properties directly forces reflow every frame and is the single most common cause of janky
"animated layouts".

**The View Transitions API now does much of this natively.** Same-document view transitions
became Baseline in October 2025 (Chrome 111+, Safari 18+, Firefox 144+) — you mark elements
with a `view-transition-name`, change the DOM inside a transition callback, and the browser
performs the morph. Cross-document (multi-page) transitions are a separate, less complete
story — see `families/06` and `STACKS.md`.

The decision between them: **View Transitions for whole-view changes and route navigation,
FLIP for fine-grained control inside a view.** The API is dramatically less code; a FLIP
library gives you per-element easing, interruption and staggering that the browser does not
expose.

---

### Reposition on reorder (`layout.flip-reposition`)

- **One line** — items slide to their new places when a list's order changes.
- **What the reader sees** — A sorted list re-sorts. Rather than the rows blinking into a
  new arrangement, each row travels — the item that was third slides up to first, the two it
  overtook move down past it, and every row arrives in its new slot in about a third of a
  second. Because you can follow individual rows with your eye, you understand *what
  changed* rather than merely seeing that something did. Sorting a table this way is the
  difference between a result you trust and a result you have to re-read.
- **Mechanism** — FLIP: measure all positions, apply the DOM change, invert with transforms,
  animate to zero.
- **Stack** — Motion's layout animations do it declaratively in React; GSAP Flip handles it
  imperatively anywhere; the View Transitions API can do it with names per item.
- **Params** — duration (0.3–0.4s; longer and a long list feels like it is sloshing); easing
  (ease-out); stagger (usually none — simultaneous movement reads as one reorganisation).
- **Use when** — sorting, ranking changes, live-updating leaderboards, reordered
  preferences.
- **Don't use when** — the list is long and the change is total. If every row moves, nothing
  is trackable and you are just animating noise.
- **Reduced motion** — apply the new order instantly. The information is in the order, not
  the travel.
- **Performance** — one measurement pass, then compositor transforms. The measurement forces
  a synchronous layout, so measure once for all elements, never per element in a loop.
- **Gotchas** — measuring in a loop that also mutates the DOM produces layout thrashing and
  is the classic way to make FLIP slower than no animation at all. Interrupted reorders need
  the in-flight transform included in the "first" measurement, or items jump.
- **References** — https://css-tricks.com/animating-layouts-with-the-flip-technique/ ·
  https://motion.dev/docs/react-layout-animations

---

### Filter reflow (`layout.flip-filter`)

- **One line** — filtering a grid moves the survivors instead of redrawing it.
- **What the reader sees** — You click a category and the grid responds in three overlapping
  beats: items that no longer qualify shrink and fade out, the remaining items glide from
  their old positions into a tighter arrangement, and any newly matching items fade in at
  their destinations. The whole thing takes under half a second. Crucially, an item that
  survives the filter is never re-created — you watch it move — so the grid reads as
  *rearranging* rather than reloading, and the connection between the filter you clicked and
  what happened is obvious.
- **Mechanism** — FLIP across the surviving set, with separate enter and exit treatments for
  the others. Exits usually need `position: absolute` during the transition so they do not
  hold space.
- **Stack** — GSAP Flip has this as a core use case; Motion's layout animation handles it in
  React; View Transitions can do the simple version.
- **Params** — move duration (0.35–0.5s); exit duration (shorter than the move — departures
  should not compete); overlap (start moves before exits finish).
- **Use when** — project grids, product catalogues, tag filtering.
- **Don't use when** — filtering usually changes the entire set, or the grid is paginated and
  everything changes anyway.
- **Reduced motion** — instant re-render, no travel; keep the fade if anything.
- **Performance** — the animation is compositor-only, but the exiting elements taken out of
  flow force a reflow at both ends of the transition.
- **Gotchas** — items that exit must not affect the layout the survivors are moving *into*,
  or the destinations are computed against a stale grid. Filtering rapidly while a transition
  is running requires the library to handle interruption, or positions drift out of sync.
- **References** — https://motion.dev/docs/react-layout-animations

---

### Shared element (`layout.shared-element`)

- **One line** — a thumbnail becomes the hero image of the view it opens.
- **What the reader sees** — Click a card in a grid and its image does not disappear and get
  replaced — it *travels*. The thumbnail grows and moves across the screen, arriving as the
  large image at the top of the detail view, while everything around it fades out and the
  new content fades in. Because the image is continuous, there is no moment where you lose
  your place: the thing you clicked is the thing you are now looking at. It is the single
  most convincing technique in this catalogue for making a website feel like an application.
- **Mechanism** — the same element (or two elements sharing a name) matched across the
  before and after states; the browser or library interpolates position, size and shape.
- **Stack** — View Transitions with `view-transition-name` on both elements is now the
  cleanest route for same-document changes (Baseline October 2025). Motion's `layoutId` does
  the equivalent in React; GSAP Flip does it imperatively.
- **Params** — duration (0.4–0.6s — long enough to follow, short enough not to stall);
  easing; whether surrounding content crossfades or staggers out first.
- **Use when** — grid to detail, list to record, image to lightbox.
- **Don't use when** — the two images are different crops or aspect ratios; the morph will
  visibly distort. Match the aspect ratio or accept the squash.
- **Reduced motion** — a plain crossfade, or an instant change. Never keep the travel.
- **Performance** — the browser snapshots both states; large images are the cost. It is one
  transition, not many, so it is cheaper than it looks.
- **Gotchas** — a `view-transition-name` **must be unique on the page at any moment**. Two
  elements sharing one name at the same time and the transition silently does nothing —
  which is the most common first failure. Names must be assigned before the transition
  starts and cleaned up after, which in a list means setting the name only on the item being
  clicked.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API ·
  https://themotiondesign.com/writing/motion-for-react-view-transitions

---

### Expand in place (`layout.expand-in-place`)

- **One line** — a card grows into a panel without leaving its position.
- **What the reader sees** — Click a card and it expands where it sits — its edges push
  outward, the neighbouring cards move aside to make room, and the extra content fades in
  once the box has finished growing. Nothing navigates and nothing overlays; the layout
  simply reorganises around the thing you chose. It preserves context perfectly, because the
  rest of the page is still visible around the expanded item, and it works particularly well
  for comparison content where you want to open one thing without losing the others.
- **Mechanism** — FLIP on the expanding element and on every neighbour it displaces; the new
  content fades in after the geometry settles.
- **Stack** — Motion's layout animation (React), GSAP Flip with `nested: true`, or View
  Transitions.
- **Params** — expand duration (0.35–0.5s); content fade delay (start at ~60% of the
  expansion); neighbour easing (shared with the expansion).
- **Use when** — FAQ entries, comparison cards, settings sections, inline detail.
- **Don't use when** — the expanded content is long enough to push everything else off
  screen. At that point it should be a page or a modal.
- **Reduced motion** — expand instantly; content appears with it.
- **Performance** — every displaced neighbour is an animated element. In a large grid, limit
  the FLIP to items actually affected.
- **Gotchas** — content revealed inside a growing box will reflow as the box changes width,
  so text visibly rewraps mid-animation; fade it in afterwards, or fix the content width
  during the transition. Scroll position must be managed, or the expanded item can end up
  below the fold.
- **References** — https://motion.dev/docs/react-layout-animations

---

### Height auto (`layout.height-auto`)

- **One line** — the accordion problem: animating to a height nobody knows in advance.
- **What the reader sees** — A disclosure row is clicked and its panel opens: the content
  below slides down as the panel grows to exactly the height the content needs, then stops
  cleanly. Closing reverses it. Done properly it is unremarkable, which is the goal. Done
  badly it is the most familiar broken animation on the web — the panel snapping open with no
  transition at all, or jumping at the end because the height was guessed wrong.
- **Mechanism** — historically: measure `scrollHeight`, animate from `0` to that pixel value,
  set `height: auto` on completion. Modern CSS can interpolate to `height: auto` directly
  where supported, and `grid-template-rows: 0fr → 1fr` is the widely-supported trick that
  needs no measurement at all.
- **Stack** — CSS for the grid-rows technique; JavaScript measurement otherwise.
- **Params** — duration (0.25–0.4s — this is a UI response, keep it brisk); easing (ease-out
  opening, ease-in closing).
- **Use when** — accordions, disclosure panels, expanding filters, inline forms.
- **Don't use when** — the content height changes *during* the animation (images loading,
  fonts settling) — the measurement will already be wrong.
- **Reduced motion** — open and close instantly.
- **Performance** — animating `height` is a layout property and reflows every frame; it is
  the accepted exception in this catalogue because the alternatives distort content. Keep
  panels small and the duration short.
- **Gotchas** — after opening, set `height: auto` or the panel will not grow when its content
  changes later. The `0fr → 1fr` grid technique needs the inner element to have
  `overflow: hidden` and `min-height: 0`, and it is the version to reach for first because it
  requires no measurement and cannot be wrong. Collapsed content must be hidden from screen
  readers and removed from tab order, not merely clipped.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/interpolate-size

---

### Auto view transition (`layout.view-transition-auto`)

- **One line** — the browser crossfades the whole view for you, in one API call.
- **What the reader sees** — Something changes — a filter applies, a tab switches, a list
  updates — and rather than the page snapping to its new state, the old and new states
  crossfade over a couple of hundred milliseconds. It is subtle to the point of being
  invisible when it works, and its real effect is the absence of a jolt. This is the baseline
  the API gives you before you name a single element.
- **Mechanism** — wrap the DOM update in a view-transition call; the browser snapshots
  before and after and crossfades the two.
- **Stack** — platform. Same-document transitions are Baseline as of October 2025 (Chrome
  111+, Safari 18+, Firefox 144+).
- **Params** — duration and easing via the transition pseudo-elements; which elements get
  names (none, for a plain crossfade).
- **Use when** — any state change large enough that an instant swap feels abrupt.
- **Don't use when** — the change is tiny and local. A crossfade over a single toggling
  button is worse than no transition.
- **Reduced motion** — skip the transition and apply the state change directly.
- **Performance** — snapshotting the viewport is not free; on a heavy page the snapshot is
  the cost, not the animation.
- **Gotchas** — the DOM update must happen *inside* the transition callback, not before it.
  The default root transition crossfades the entire page, which can look wrong when only a
  corner changed — name the changing region to scope it. And you get very little easing
  control compared to a timeline: this is a convenience, not a choreography tool.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API ·
  https://www.drweb.de/layout-animation-flip-view-transitions/

---

### Drag reorder (`layout.drag-reorder`)

- **One line** — dragging an item makes its neighbours move out of the way live.
- **What the reader sees** — Pick up a row and it lifts — a shadow appears, it scales up a
  fraction, and it follows your pointer. As you move it over the list, the other rows part
  ahead of it: the row you are hovering over slides down to open a gap exactly the size of
  what you are carrying. Release, and the dragged row drops into the gap and settles flat.
  At no point do you wonder where it will land, because the gap has been showing you all
  along.
- **Mechanism** — the dragged element follows the pointer directly; the others are FLIP-animated
  each time the projected insertion index changes.
- **Stack** — a drag library plus layout animation; Motion's reorder primitives and GSAP
  Draggable + Flip both cover it.
- **Params** — lift scale (1.02–1.05) and shadow; neighbour shift duration (0.2s — must feel
  immediate); drop settle (0.2–0.3s).
- **Use when** — reorderable lists, kanban columns, playlist editing.
- **Don't use when** — there is no keyboard equivalent. Drag-only reordering is inaccessible;
  provide move-up/move-down controls or keyboard drag.
- **Reduced motion** — no lift, no neighbour animation; move items instantly on drop.
- **Performance** — the dragged element should be promoted to its own layer once; neighbours
  animate only when the index changes, not every frame.
- **Gotchas** — recompute the projected index from the pointer position, not from collision
  with elements that are themselves mid-animation, or the gap oscillates. Auto-scrolling at
  the list edges is required for lists longer than the viewport, and is where most
  implementations stop short.
- **References** — https://motion.dev/docs/react-layout-animations

---

### Tab indicator (`layout.tab-indicator`)

- **One line** — a marker slides between tabs rather than blinking to the new one.
- **What the reader sees** — A row of tabs with a bar under the active one. Click a different
  tab and the bar does not vanish and reappear — it slides across, and if the tabs are
  different widths it stretches or contracts as it travels, arriving fitted to its new label.
  It is a small thing, and it is the difference between a tab strip that feels like a control
  and one that feels like a set of links. The stretch is what sells it: a bar that only
  translates looks like a separate object, while one that resizes looks attached to the tabs.
- **Mechanism** — measure the target tab, animate the indicator's `transform: translateX`
  and `scaleX` (not `left`/`width`) to match.
- **Stack** — CSS custom properties plus a measurement, or FLIP, or a shared
  `view-transition-name` on the indicator.
- **Params** — duration (0.25–0.3s); easing (ease-out); whether the label colour transitions
  with it.
- **Use when** — tabs, segmented controls, filter pills, nav underlines. The same shared-marker
  technique applied to keyboard focus is `micro.focus-ring`.
- **Don't use when** — tabs wrap to multiple lines; a horizontal slide across a line break
  looks broken.
- **Reduced motion** — the indicator jumps to the new tab.
- **Performance** — one element, transform-only. Free.
- **Gotchas** — scaling a bar also scales its border radius, so a pill-shaped indicator
  distorts as it stretches; use a nine-slice approach or accept a rectangle. Re-measure on
  resize and after fonts load, or the indicator sits fractionally off its label.
- **References** — https://css-tricks.com/animating-layouts-with-the-flip-technique/

---

### Modal from origin (`layout.modal-from-origin`)

- **One line** — a dialog grows out of the control that opened it.
- **What the reader sees** — Click a button and the modal does not fade in over the centre of
  the screen — it emerges from the button itself, expanding and moving to its final position
  in a single continuous movement of about a third of a second, while the backdrop darkens
  behind it. Closing reverses it back into the button. The spatial link is explicit: this
  panel belongs to that control, and you know exactly where it will return to. It is a
  markedly stronger sense of place than a centre-screen fade, which could have come from
  anywhere.
- **Mechanism** — FLIP from the trigger's bounding box to the dialog's final geometry, or a
  transform-origin set to the trigger's position relative to the dialog.
- **Stack** — FLIP, or `@starting-style` plus a computed transform-origin for the CSS-only
  version.
- **Params** — duration (0.3–0.4s); backdrop fade (slightly longer, so it does not finish
  first); content fade-in (after the geometry settles).
- **Use when** — popovers, menus, dialogs opened from a specific control.
- **Don't use when** — the dialog is opened by something off-screen or by a keyboard
  shortcut with no origin. Fall back to a centred scale.
- **Reduced motion** — appear at final position with a short fade, no travel.
- **Performance** — cheap, one element plus a backdrop.
- **Gotchas** — focus must move into the dialog when it opens and return to the trigger when
  it closes, regardless of the animation; the motion is decoration on top of a focus contract
  that has to be right. Top-layer elements (`<dialog>`, popover) interact with entry and exit
  animation in ways that are still uneven — exit animations in particular are the weaker half.
- **References** — https://web.dev/blog/baseline-entry-animations

---

### List insert & remove (`layout.list-add-remove`)

- **One line** — adding or deleting an item makes the neighbours move rather than jump.
- **What the reader sees** — Delete a row and it collapses — shrinking and fading — while
  everything below slides up to close the gap in one smooth movement. Add one and the reverse:
  the list opens a space, and the new row settles into it. The eye follows the closing gap, so
  the deletion registers as a deletion rather than as a page that suddenly looks different.
  In a list the user is actively editing, this is what keeps them oriented after every action.
- **Mechanism** — exit animation for the removed item (usually collapsing its height), FLIP
  for everything that shifts, entry animation for anything added.
- **Stack** — Motion's `AnimatePresence` for React; FLIP plus manual exit handling elsewhere.
- **Params** — remove duration (0.2–0.3s); shift duration (0.3s); order (start the shift
  before the removal completes or the list stalls).
- **Use when** — todo lists, cart contents, form field arrays, notification stacks.
- **Don't use when** — items are removed in bulk. Twenty simultaneous collapses is chaos —
  re-render instead.
- **Reduced motion** — instant removal and reflow.
- **Performance** — fine at list scale. The removed element must be taken out of flow during
  its exit or it holds space and the shift starts late.
- **Gotchas** — the deleted item must be removed from the accessibility tree immediately, not
  when its animation finishes, or a screen reader still sees it. Announce the change via a live
  region — the animation communicates nothing to someone not looking at it.
- **References** — https://motion.dev/docs/react-layout-animations

---

### Masonry settle (`layout.masonry-settle`)

- **One line** — an irregular grid reflows into place as items load or resize.
- **What the reader sees** — A masonry grid of mixed-height cards. As images finish loading
  and the true heights become known, the columns rebalance — cards slide up or across into
  their final slots rather than snapping there. On a slow connection you watch the grid
  settle over a second or two, which turns what would be an alarming series of jumps into
  something that reads as the layout finding its shape.
- **Mechanism** — FLIP on each recalculation of the layout, triggered by image load, resize
  or content change.
- **Stack** — a masonry layout engine plus FLIP; CSS masonry, where available, has no
  interpolation of its own.
- **Params** — duration (0.3–0.4s); whether to stagger by column (usually not — simultaneous
  reads as one settle).
- **Use when** — image grids with unknown aspect ratios, Pinterest-style layouts.
- **Don't use when** — you can reserve the correct aspect ratio up front. Preventing the
  reflow entirely beats animating it — this technique is damage control, not a feature.
- **Reduced motion** — items snap to their new positions.
- **Performance** — a full re-measure per event; debounce resize, and batch image-load
  recalculations rather than reflowing per image.
- **Gotchas** — cards animating while the user is trying to click one is a real usability
  problem; consider locking the layout once it has settled. Always set `width`/`height` or
  `aspect-ratio` on images so most of the reflow never happens.
- **References** — https://css-tricks.com/animating-layouts-with-the-flip-technique/

---

### Split view open (`layout.split-view`)

- **One line** — a side panel opens and the main content compresses to accommodate it.
- **What the reader sees** — Click an item and a detail panel slides in from the right while
  the list beside it narrows to make room — the two movements exactly complementary, so the
  panel appears to push rather than to cover. The list stays visible and usable throughout.
  It is the master-detail pattern in motion, and the compression is what distinguishes it
  from a drawer sliding over the top: nothing is hidden, so nothing has to be dismissed to
  get back.
- **Mechanism** — a grid or flex track animating its size, with the panel translating in;
  ideally the container animates and the children reflow, rather than animating each child.
- **Stack** — CSS grid track transitions where the browser supports interpolating them;
  otherwise FLIP on the container.
- **Params** — duration (0.3–0.4s); easing (shared between the panel and the compression);
  minimum main width before the panel becomes an overlay instead.
- **Use when** — mail clients, admin tables, file browsers — anything where context matters
  as much as detail.
- **Don't use when** — the viewport is narrow. Below a threshold the panel must become a full
  overlay, and the transition should change with it.
- **Reduced motion** — instant layout change.
- **Performance** — the container is genuinely changing layout, so text in the main column
  rewraps every frame. Keep it short, and consider fading the main content slightly during
  the transition to mask the rewrap.
- **Gotchas** — the reflow of the main column is the expensive part and the part that looks
  worst; a fixed-width inner wrapper that translates instead of reflowing avoids it at the
  cost of a brief clipped edge. Focus should move to the panel when it opens.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API

---

## Family notes

**Never animate layout properties.** Measure, change, invert, play. `width`, `height`, `top`
and `left` change once and instantly; the movement is a transform. The only sanctioned
exception in this file is `height` on a disclosure panel, and even there the
`0fr → 1fr` grid technique avoids it.

**Measure once, mutate once.** Interleaving reads and writes in a loop produces layout
thrashing that makes FLIP slower than doing nothing. Batch all measurements, then all
mutations.

**View Transitions first for view-level changes; FLIP for control.** The API is Baseline for
same-document work as of October 2025 and is a fraction of the code. Reach for FLIP when you
need per-element easing, staggering or interruption — the API gives you very little of that.

**`view-transition-name` must be unique at the moment of the transition.** Two elements with
the same name and the transition silently does nothing. In a list, that means naming only the
item being acted on.

**The motion carries meaning here, so the reduced-motion branch must carry it too.** If an
animation is showing *which item became which*, its static equivalent needs to make that
knowable another way — usually by keeping the change small and announcing it in a live region.
