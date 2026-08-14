# Navigation & menu systems

Motion applied to the surfaces people use to get somewhere else. Headers that hide, menus that
open, submenus that expand, palettes that appear.

Navigation is the highest-stakes surface for motion in an interface, because it is the one
people use when they are already slightly lost. Everything here is judged by a harsher standard
than a hero animation: an entrance that delays a menu by 400ms is not elegant, it is friction
applied at exactly the wrong moment.

**Two constraints govern the whole family.**

**Keyboard behaviour is the feature, and the animation is decoration on top of it.** An
overlay menu needs focus moved into it on open, trapped while it is open — Tab from the last
item loops to the first, Shift+Tab from the first loops to the last — Escape to close, and
focus returned to the trigger on close. Without that, keyboard focus disappears behind the
overlay and the user is stranded. The animation is what happens while all of that is being
done correctly, never a substitute for it.

**A focus trap is right for an overlay and wrong for a submenu.** The distinction matters: a
modal navigation overlay *should* trap, because there is nothing else to reach. A dropdown or
mega-menu submenu should *not* — a submenu that intercepts Tab without an exit is a bug, not
an accessibility feature. Submenu items take `tabindex="-1"`, and Escape closes and returns
focus to the trigger.

The **Popover API** is now supported across Chrome, Edge, Firefox and Safari, and it provides
light-dismiss, top-layer placement and much of this behaviour natively — it should be the
starting point for anything dropdown-shaped in 2026 rather than a hand-rolled equivalent.

---

### Hide-on-scroll header (`nav.hide-on-scroll`)

- **One line** — the header retracts as you scroll down and returns the moment you scroll up.
- **What the reader sees** — Read down a long page and the header slides up out of view,
  giving the content the full window. Scroll up even slightly — the gesture people make when
  they want to go back or navigate — and it drops straight back down, already there before
  you have finished reaching for it. It reads as attentive rather than as a control that
  disappeared: the header seems to know the difference between reading and looking for
  something.
- **Mechanism** — scroll direction detected per update, driving a `translateY` between 0 and
  -100%, with a threshold so it never hides near the top of the page.
- **Stack** — a scroll listener plus a persistent transform setter; CSS
  `animation-timeline: scroll()` can do a simpler version where support allows.
- **Params** — hide threshold (150–250px, so it never hides on the first screen); transition
  (0.25–0.35s); whether the header becomes solid or translucent once past the top.
- **Use when** — long-form content, mobile layouts, anywhere vertical space is scarce.
- **Don't use when** — the header holds controls needed continuously (a filter bar, a running
  total). Hiding a tool is not the same as hiding a signpost.
- **Reduced motion** — the header stays visible permanently. A header that moves in and out of
  view is exactly the kind of peripheral motion the preference is asking you to stop.
- **Performance** — one transform driven by a scroll listener; create the setter once, never a
  tween per scroll event.
- **Gotchas** — apply the solid/scrolled state as soon as the page moves at all, not at the
  hide threshold, or large display type scrolls under a transparent header for the first
  couple of hundred pixels. On iOS the address bar collapsing fires scroll events that can
  trigger a spurious hide.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/animation-timeline

---

### Fullscreen overlay menu (`nav.overlay-menu`)

- **One line** — a panel covers the viewport and its links arrive in sequence.
- **What the reader sees** — Tap the menu control and a coloured panel expands to fill the
  screen — often wiping from one edge rather than fading — and once it has settled the links
  arrive one after another from below, each a tenth of a second behind the last, so the list
  assembles top to bottom in about half a second. Closing reverses it. The staging is what
  makes it feel composed: the panel establishes the space, then the content occupies it,
  rather than everything appearing at once.
- **Mechanism** — a clip or transform reveal on the panel, then a staggered entrance on the
  items, sequenced on one timeline.
- **Stack** — any timeline library; the panel is CSS and the stagger is the only part needing
  coordination.
- **Params** — panel reveal (0.4–0.6s); item stagger (0.05–0.08s); total under 1s; close
  faster than open (0.3s — leaving should never feel slower than arriving).
- **Use when** — mobile navigation, design-led sites where the menu is a moment.
- **Don't use when** — the site has a handful of links that would fit in a bar. A fullscreen
  overlay for four items is ceremony.
- **Reduced motion** — panel and links appear together, no stagger, no wipe.
- **Performance** — one large composited panel; trivial. The items are the only per-element
  cost.
- **Gotchas** — **the accessibility contract is the hard part**: move focus into the panel on
  open, trap it while open, close on Escape, return focus to the toggle. Scroll behind the
  overlay must be locked or the page moves under it. And the menu must be usable before the
  animation finishes — a link that is not clickable for 600ms because it is still animating is
  the most common complaint about this pattern.
- **References** — https://www.uxpin.com/studio/blog/how-to-build-accessible-modals-with-focus-traps/ ·
  https://github.com/thwbh/accessibility-snippets/blob/main/focus-trap/fullscreen-overlay-after.html

---

### Dropdown open (`nav.dropdown`)

- **One line** — a small panel appears beneath its trigger.
- **What the reader sees** — Click a nav item and a panel drops just below it, scaling up
  slightly from its top edge and fading in over about 150 milliseconds, so it appears to
  unfold from the control that opened it rather than materialise nearby. Click elsewhere and
  it vanishes. It is the most-used menu pattern on the web, and the interesting decisions in
  it are all about timing and dismissal rather than about the movement.
- **Mechanism** — a scale from a top transform-origin plus opacity, with the panel in the top
  layer.
- **Stack** — the **Popover API** gives top-layer placement, light dismiss and Escape handling
  natively, with `@starting-style` for the entry animation. This should be the default in 2026
  rather than a hand-built panel.
- **Params** — duration (120–180ms — this is a UI response, not a reveal); origin (top, aligned
  to the trigger); exit faster than entry.
- **Use when** — navigation menus, account menus, any short list attached to a control.
- **Don't use when** — the content is large enough to be a page section. Then it is a mega menu.
- **Reduced motion** — appears instantly.
- **Performance** — free.
- **Gotchas** — **do not trap focus in a dropdown.** Items take `tabindex="-1"` with arrow-key
  navigation, Escape closes and returns focus to the trigger, and Tab moves on rather than
  looping — a submenu that intercepts Tab with no exit path is the classic keyboard trap.
  Opening on hover without an intent delay makes the menu flicker as the pointer crosses the bar.
- **References** — https://www.dfm2html.com/tutorials/accessible-drop-down-menus-in-2026-without-a-framework/

---

### Mega menu reveal (`nav.mega-menu`)

- **One line** — a full-width panel of grouped links expands beneath the bar.
- **What the reader sees** — Point at a top-level item and a wide panel opens below the header,
  its height growing to fit several columns of links, which fade in a beat after the panel has
  reached its size. Move sideways to the next top-level item and the panel does not close and
  reopen — it stays, resizing to the new content while the columns crossfade. That continuity
  is what separates a considered mega menu from four dropdowns sharing a bar.
- **Mechanism** — a single shared panel animating height between measured content sizes, with
  the column contents crossfading; hover intent governs open and close.
- **Stack** — a height or grid-rows transition (see `layout.height-auto`) plus crossfade;
  hover-intent timing is essential.
- **Params** — open delay (100–200ms); close delay (300–500ms, longer than open); resize
  duration (0.2s); content crossfade (0.15s).
- **Use when** — large catalogues, documentation, retail navigation with genuine hierarchy.
- **Don't use when** — the site has fewer than about twenty destinations.
- **Reduced motion** — panel appears and resizes instantly.
- **Performance** — animating height reflows the panel contents; measure once per target
  rather than per frame.
- **Gotchas** — the diagonal-travel problem: a user moving toward the panel passes over
  neighbouring triggers, and a naive implementation switches content under their pointer. The
  close delay, and ideally checking the direction of travel, is what fixes it. Each panel must
  be reachable and dismissible by keyboard independently of hover.
- **References** — https://www.dfm2html.com/tutorials/accessible-drop-down-menus-in-2026-without-a-framework/

---

### Drawer slide (`nav.drawer`)

- **One line** — a panel slides in from an edge over the content.
- **What the reader sees** — A panel enters from the left or right, travelling its own width in
  about a third of a second and stopping flush against the edge, while the page behind darkens
  under a scrim. The page does not move; the drawer is over it, which is what distinguishes it
  from a split view where content compresses. On mobile it can be dragged closed, following your
  finger, which makes dismissal feel physical rather than administrative.
- **Mechanism** — `translateX` from ±100% to 0 with a backdrop fade; optionally draggable to
  dismiss (see `gesture.swipe-dismiss`).
- **Stack** — CSS transitions, or `<dialog>` / the Popover API for the top-layer and dismissal
  behaviour.
- **Params** — duration (0.28–0.35s); easing (ease-out on open, ease-in on close); scrim opacity
  (0.4–0.6); drag-to-close threshold.
- **Use when** — mobile navigation, filter panels, secondary tools. On phones a bottom-anchored
  panel is usually the better shape — see `overlay.bottom-sheet`.
- **Don't use when** — the content behind must stay usable. A scrim says "deal with me first".
- **Reduced motion** — appears without sliding; the scrim can still fade.
- **Performance** — one transform plus a backdrop; trivial.
- **Gotchas** — lock body scroll while open and restore the exact scroll position on close, or
  the page jumps to the top — a very common bug. Focus moves in, traps, and returns. The scrim
  must be dismissible by click *and* by Escape.
- **References** — https://www.uxpin.com/studio/blog/how-to-build-accessible-modals-with-focus-traps/

---

### Command palette (`nav.command-palette`)

- **One line** — a search-driven launcher appears over everything on a keystroke.
- **What the reader sees** — Press the shortcut and a panel drops in at the top-centre of the
  screen, scaled up very slightly from 96%, with the rest of the interface dimmed and blurred
  behind it. Type, and results filter live — rows appearing and disappearing without the list
  jumping, the highlighted row sliding between positions as you arrow through them. Press
  Escape and it is gone. The whole open transition is about 150 milliseconds, because the user
  invoked it with a keystroke and is already typing.
- **Mechanism** — a scale-and-fade entry, plus a moving highlight (`layout.tab-indicator`
  technique) and a filtered list with keyed transitions.
- **Stack** — a dialog or popover for the shell; the list is where the work is.
- **Params** — open (120–160ms — anything slower is felt); highlight movement (80–120ms);
  result list transitions (fast, or none while typing).
- **Use when** — tools with many destinations or actions and a keyboard-first audience.
- **Don't use when** — the audience is not keyboard-driven and there is no discoverable
  affordance pointing at it.
- **Reduced motion** — appears instantly; the highlight jumps.
- **Performance** — the list must handle typing at full speed; animating rows on every keystroke
  is the failure mode. Animate the highlight, not the rows.
- **Gotchas** — do not animate list items during filtering — the user is typing and the target
  moving under them is worse than an instant change. Focus goes to the input on open, results
  are announced as a live region, and the current selection needs `aria-activedescendant`.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/Popover_API

---

### Active link indicator (`nav.active-indicator`)

- **One line** — a marker travels to the current nav item instead of blinking to it.
- **What the reader sees** — A bar or pill marks the active navigation item. Move to another
  section and it slides across the bar to the new item, stretching or contracting to match its
  width on the way. You watch a single object move rather than one disappear and another
  appear, which makes the relationship between where you were and where you are explicit.
- **Mechanism** — one shared indicator positioned by `translateX` and `scaleX` to the active
  item's measured rectangle.
- **Stack** — the same technique as `layout.tab-indicator`; CSS custom properties plus a
  measurement, or a shared `view-transition-name` across route changes.
- **Params** — duration (0.25–0.3s); easing (ease-out); whether it animates on route change or
  only on click.
- **Use when** — horizontal navigation, section tabs, sidebar navigation with a rail.
- **Don't use when** — nav items wrap across lines; a horizontal slide across a line break
  looks broken.
- **Reduced motion** — the indicator jumps.
- **Performance** — one element; free.
- **Gotchas** — re-measure after fonts load and on resize, or the marker sits slightly off its
  label. The indicator is decorative — the active state must also be conveyed with
  `aria-current`, not by position alone.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/transform

---

### Breadcrumb collapse (`nav.breadcrumb-collapse`)

- **One line** — a long trail folds into an ellipsis and expands on demand.
- **What the reader sees** — A breadcrumb trail too long for its space shows the first item, an
  ellipsis, and the last two. Click the ellipsis and the hidden middle expands horizontally,
  the trail widening to reveal the intermediate levels while the surrounding layout adjusts.
  It solves a genuine spatial problem, and the expansion is what tells you the levels were
  always there rather than newly fetched.
- **Mechanism** — a width or `grid-template-columns` transition on the collapsed segment, with
  the hidden items fading in as space appears.
- **Stack** — CSS transition on the container; the collapse logic is the real work.
- **Params** — expand duration (0.2–0.3s); whether it collapses again on blur.
- **Use when** — deep hierarchies: file browsers, documentation, category trees.
- **Don't use when** — the hierarchy is three levels deep. Nothing needs collapsing.
- **Reduced motion** — expand instantly.
- **Performance** — a width transition reflows the row; small and acceptable at this scale.
- **Gotchas** — the collapsed items must remain in the accessibility tree or the trail is
  incomplete for screen readers regardless of what is visible. The ellipsis needs a real label
  ("Show 3 more levels"), not a bare glyph.
- **References** — https://www.w3.org/WAI/ARIA/apg/patterns/breadcrumb/

---

### Bottom tab bar (`nav.tab-bar`)

- **One line** — the mobile tab bar's icons animate as sections change.
- **What the reader sees** — A row of icons fixed at the bottom of the screen. Tap one and its
  icon fills or swaps to its active variant while the label brightens, and the previously active
  item fades back — a change of about 200 milliseconds with no movement of the bar itself. Some
  implementations bounce the tapped icon slightly, which reads as acknowledgement rather than
  as decoration.
- **Mechanism** — an icon state transition (`svg.icon-state`) plus colour, with an optional
  short scale pulse on the tapped item.
- **Stack** — CSS transitions; the icons are the only moving part.
- **Params** — transition (150–250ms); pulse (scale 1.15, 150ms) if used; indicator movement if
  the bar has one.
- **Use when** — mobile web apps with three to five top-level sections.
- **Don't use when** — there are more than five destinations; a tab bar is not a menu.
- **Reduced motion** — states change without the pulse.
- **Performance** — trivial. The bar must respect the device safe area, which is a layout
  concern rather than a motion one but is what most implementations get wrong.
- **Gotchas** — the active tab needs `aria-current`, and the tap target must meet minimum size
  independent of the icon's visual size. Do not animate the bar's position on scroll — a
  navigation control that hides on a phone is very hard to get back.
- **References** — https://www.w3.org/WAI/ARIA/apg/patterns/

---

### Scroll-spy highlight (`nav.scroll-spy`)

- **One line** — the sidebar marks which section you are currently reading.
- **What the reader sees** — A table of contents beside a long document, with the current
  section's entry marked. As you scroll, the mark moves down the list in step with your
  reading, brightening the new entry and dimming the old, so a glance sideways tells you where
  you are in the whole. On a documentation page it is the difference between reading a page and
  navigating a document.
- **Mechanism** — IntersectionObserver on each section heading, driving an active class; the
  indicator moves using the shared-marker technique.
- **Stack** — IntersectionObserver plus a moving indicator. CSS `scroll-timeline` can drive a
  simple version.
- **Params** — the observer's root margin (usually a band near the top third, so a section
  becomes "current" as it reaches reading position); transition (0.2s).
- **Use when** — documentation, long articles, anything with a table of contents.
- **Don't use when** — sections are shorter than the viewport; the highlight will thrash
  between them.
- **Reduced motion** — the highlight changes without sliding.
- **Performance** — observers are cheap; do not compute positions on every scroll frame.
- **Gotchas** — define the "current" band carefully or two sections are both partly visible and
  the highlight oscillates. Sync the URL hash carefully — updating it on every section change
  fills the user's history with entries they never chose to visit.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API

---

### Skip link reveal (`nav.skip-link`)

- **One line** — a hidden link appears when focused, offering to jump past the navigation.
- **What the reader sees** — Nothing at all, until a keyboard user presses Tab on a fresh page:
  a link slides down from the top edge reading "Skip to content", takes focus, and disappears
  again when focus moves on. For a sighted mouse user it never exists. For a keyboard user it
  saves tabbing through forty navigation links on every single page.
- **Mechanism** — a visually-hidden link that becomes visible on `:focus`, usually with a short
  slide from above the viewport edge.
- **Stack** — CSS. Roughly ten lines total.
- **Params** — transition (100–150ms); position (top-left, over everything).
- **Use when** — every site with more than a few navigation links. This is not optional.
- **Don't use when** — never.
- **Reduced motion** — appears without sliding; it must still appear.
- **Performance** — free.
- **Gotchas** — hide it with a clip or off-screen positioning, **never** with `display: none` or
  `visibility: hidden`, which make it unfocusable and therefore useless. The target must be
  focusable — give the content container `tabindex="-1"` — or the jump moves the scroll but not
  the focus, which helps nobody.
- **References** — https://www.w3.org/WAI/WCAG22/Techniques/general/G1

---

### Sticky sub-nav (`nav.sticky-subnav`)

- **One line** — a secondary bar detaches and pins as you scroll past it.
- **What the reader sees** — A row of section links sits in the page under the header. Scroll
  past it and it lifts — gaining a border and a background as it becomes stuck beneath the
  main header — and stays there while you move through the section. Scroll back and it drops
  into its original position and loses the border. The state change is what communicates
  "this is now floating over the content" rather than the bar seeming to duplicate itself.
- **Mechanism** — `position: sticky` plus a state class applied when the element becomes stuck,
  detected with a sentinel element and IntersectionObserver.
- **Stack** — CSS sticky is native; the stuck-state detection is the only JavaScript.
- **Params** — the offset it sticks at (must account for the main header); state transition
  (0.2s).
- **Use when** — long documents with sections, product pages with jump links, filter bars.
- **Don't use when** — the header already hides on scroll; two moving bars fight each other.
- **Reduced motion** — sticking is not motion; keep it. Only the shadow or border transition is
  animation, and it can be instant.
- **Performance** — native sticky is free and does not run JavaScript per frame.
- **Gotchas** — `position: sticky` fails silently if any ancestor has `overflow: hidden` or
  `auto`, which is the single most common reason it does not work. There is no `:stuck`
  selector — the sentinel-plus-observer trick is the standard workaround.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/position

---

## Family notes

**Focus management is the feature.** Move focus in, trap it in overlays, return it on close,
close on Escape. Every entry here is unusable without that, and no amount of animation quality
compensates.

**Trap overlays, never submenus.** A modal navigation panel should trap focus because there is
nothing else to reach. A dropdown that intercepts Tab with no exit path is a keyboard trap and
a defect.

**Reach for the Popover API first.** Top-layer placement, light dismiss and Escape handling
come free, across all major browsers, and `@starting-style` animates the entry. Hand-rolled
dropdowns in 2026 are usually reimplementing this badly.

**Navigation motion must be fast.** 150–350ms across this entire family. Menus are opened by
people who already know where they want to go.

**Never hide a control the user needs.** Hiding a header is fine; hiding a filter bar, a cart
total or a mobile tab bar is taking away a tool because they scrolled.
