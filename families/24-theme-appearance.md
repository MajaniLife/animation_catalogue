# Theme & appearance transitions

Motion when the whole interface changes its appearance — light to dark, an accent colour, text
size, density, contrast.

This family is unusual in scope: the change touches every pixel simultaneously, so there is no
"element" being animated and no obvious place to put an entrance. It is also unusual in
motivation. A theme change is almost always something the *user* asked for, which means the
animation's job is not to impress but to confirm — and to avoid the two ways this goes wrong: a
jarring full-screen flash, or an interface that appears to break for a moment while it changes.

**The most important thing in this file is not an animation at all.** It is preventing the
flash of the wrong theme on load. During server rendering there is no `window`, no
`localStorage` and no knowledge of the user's system preference, so the server renders a default
— usually light — and a dark-mode user sees a white flash before the correct theme applies. The
fix is a small **blocking inline script in the `<head>`** that reads the stored preference and
sets the theme before first paint. Every theming library ships one, and hand-rolled
implementations routinely omit it.

**The 2026 opportunity is the View Transitions API**, which can capture the entire page before
and after the change and animate between the two — including a circular reveal expanding from
the toggle itself. Support sits around 89%, so it is a progressive enhancement over an instant
switch rather than a foundation.

---

### Mode switch (`theme.mode-switch`)

- **One line** — light becomes dark, everywhere, at once.
- **What the reader sees** — Press the toggle and the whole page changes: backgrounds darken,
  text lightens, borders and shadows invert their logic — all together, over about a quarter of
  a second. Because everything moves in step, it reads as the lights being dimmed rather than
  as elements being restyled one by one. Any element that transitions at a different rate
  immediately looks broken, which is why the coordination matters more than the duration.
- **Mechanism** — a single class or `data-theme` attribute on the root swapping a set of custom
  properties, with a transition on `background-color` and `color` at the root level.
- **Stack** — CSS custom properties; `light-dark()` lets a single declaration hold both values
  where `color-scheme` is set.
- **Params** — duration (200–300ms); transition applied at the root, not per element; ease-out.
- **Use when** — any product offering a theme choice.
- **Don't use when** — the transition is applied to every element individually, which produces
  a ripple of slightly-out-of-sync changes across the page.
- **Reduced motion** — the theme changes instantly. A colour crossfade is not vestibular motion,
  but a full-screen luminance change is worth sparing.
- **Performance** — a full-viewport repaint. One repaint is fine; a transition on
  `*, *::before, *::after` produces thousands of animated properties and is the standard way this
  becomes janky.
- **Gotchas** — set `color-scheme` so form controls, scrollbars and the browser UI follow the
  theme too; without it you get dark pages with white native widgets. Images and illustrations
  authored for one theme need a swap or a filter, not just a background change.
- **References** — https://jonshamir.com/writing/color-mode/ ·
  https://dev.to/bishoy_bishai/implementing-dark-mode-css-variables-system-preference-and-persistence-2a43

---

### No-flash boot (`theme.no-flash`)

- **One line** — the correct theme is applied before the first pixel is drawn.
- **What the reader sees** — Ideally nothing whatsoever. A dark-mode user loads the page and it
  is dark, immediately, with no white flash. The failure is far more memorable than the success:
  a bright flash on every navigation, at whatever time of night they happen to be reading. It is
  not an animation, and it belongs in this file because it is the single most common defect in
  theme implementations.
- **Mechanism** — a small **blocking inline script in `<head>`** that reads the stored preference
  (or the system query) and sets the theme attribute on the root element before the body renders.
- **Stack** — a few lines of inline JavaScript; theming libraries inject exactly this.
- **Params** — none. It must be inline, in the head, and synchronous.
- **Use when** — every server-rendered or statically-generated site with a theme option.
- **Don't use when** — never. There is no acceptable alternative.
- **Reduced motion** — unaffected.
- **Performance** — a tiny blocking script is the correct trade here; deferring it is what causes
  the flash. It must not grow beyond reading a value and setting an attribute.
- **Gotchas** — SSR is where this bites: the server has no `window`, no `localStorage` and no
  access to the system preference, so it can only guess. Also suppress the theme transition on
  first paint — otherwise the correct theme *animates in* from the default, which looks like the
  flash you were trying to prevent.
- **References** — https://www.notanumber.in/blog/fixing-react-dark-mode-flickering ·
  https://victordibia.com/blog/gatsby-fouc/ ·
  https://www.simonporter.co.uk/posts/what-the-fouc-astro-transitions-and-tailwind/

---

### Circular reveal (`theme.circular-reveal`)

- **One line** — the new theme expands outward from the toggle you pressed.
- **What the reader sees** — Press the theme button and darkness spreads from that exact point:
  a circle grows from beneath your finger, sweeping across the page until it covers everything,
  the old theme visible outside the expanding edge and the new one inside it. It takes half a
  second and it is unmistakably the most satisfying way to change a theme, because the change
  visibly originates from the control you used.
- **Mechanism** — a View Transition captures before and after states; the new snapshot is
  revealed with an expanding `clip-path: circle()` centred on the toggle's coordinates.
- **Stack** — the View Transitions API with a custom animation on the new-view pseudo-element;
  the click coordinates set the circle's origin.
- **Params** — duration (400–600ms); origin (the toggle's centre); radius (to the furthest
  viewport corner, computed per press).
- **Use when** — a deliberate theme control on a site where the flourish is welcome.
- **Don't use when** — support is required universally, or the toggle is off screen when the
  theme changes programmatically.
- **Reduced motion** — skip the transition and swap the theme instantly. This is a large
  full-screen sweep and exactly the sort of motion the preference exists to suppress.
- **Performance** — the browser snapshots the whole viewport twice; on a heavy page the
  snapshot dominates. Support is around 89%, so the fallback is an instant switch — which is
  perfectly acceptable and should be verified as such.
- **Gotchas** — the radius must be computed to the *farthest* corner from the click, or the
  circle stops short and leaves a visible ring of the old theme. Elements with their own
  `view-transition-name` are captured separately and will not be inside the sweeping circle —
  which is usually surprising and occasionally useful.
- **References** — https://jonshamir.com/writing/color-mode/ ·
  https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API

---

### Toggle icon morph (`theme.toggle-icon`)

- **One line** — the sun becomes a moon.
- **What the reader sees** — The toggle's icon transforms rather than swapping: the sun's rays
  retract into the disc, the disc slides slightly and a bite is taken out of one edge, and it is
  a crescent moon — over about three hundred milliseconds. It is the small pleasure that makes
  people press the theme toggle twice just to watch it, and it is one of the few places in an
  interface where that is a perfectly good outcome.
- **Mechanism** — an SVG morph or coordinated transforms on sub-paths (see `svg.icon-state`),
  with the ray elements scaling out as the crescent mask moves in.
- **Stack** — CSS transforms on grouped SVG paths handle it; a morph plugin for genuinely
  different geometry.
- **Params** — duration (250–350ms); slight rotation on the disc; rays retracting before the
  crescent forms.
- **Use when** — the theme control is a visible, frequently-used button.
- **Don't use when** — the toggle is a three-way control including "system". Sun and moon cannot
  express a third state; use labels.
- **Reduced motion** — the icon swaps without morphing.
- **Performance** — trivial.
- **Gotchas** — the icon must describe **what pressing it will do**, and be labelled
  accordingly — an ambiguous sun/moon with no accessible name is a coin flip for anyone who
  cannot see the current theme. Use `aria-pressed` or a clear label like "Switch to dark theme".
- **References** — https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Element/animate

---

### System preference sync (`theme.system-sync`)

- **One line** — the interface follows the operating system, and changes when it does.
- **What the reader sees** — At sunset the OS switches to dark, and the page — open in a
  background tab — is dark when you return to it. If you are looking at it when the switch
  happens, the page transitions along with everything else on the machine. Nothing was pressed;
  the interface simply agreed with the system, which is what most people expect by default.
- **Mechanism** — `prefers-color-scheme` in CSS, plus a `matchMedia` change listener so the
  switch applies live rather than only on reload.
- **Stack** — CSS media query; the listener is a few lines.
- **Params** — the same transition as `theme.mode-switch`; no separate treatment.
- **Use when** — as the default for every site, with an explicit override available.
- **Don't use when** — the user has made an explicit choice. A stored preference must beat the
  system, or you are overriding a deliberate decision every sunset.
- **Reduced motion** — the change applies instantly.
- **Performance** — free.
- **Gotchas** — listen for changes rather than reading once at load, or an open tab keeps the
  old theme all evening. The three-state model — light, dark, system — is what users actually
  need, and "system" must be a real stored value rather than the absence of a choice.
- **References** — https://dev.to/weatherclockdash/dark-mode-in-firefox-extensions-respecting-system-preferences-1iok

---

### Accent colour change (`theme.accent-change`)

- **One line** — the user picks a colour and the interface adopts it.
- **What the reader sees** — Choose a swatch from a row of colours and every accented element in
  the interface changes together — buttons, links, focus rings, the active nav marker, chart
  series — over about two hundred milliseconds. Because they all shift in unison you perceive it
  as one property of the product changing rather than as many elements being restyled, which is
  precisely the impression a design token system should give.
- **Mechanism** — one custom property updated at the root, with a transition on the properties
  that consume it.
- **Stack** — CSS custom properties; registering them with `@property` allows the browser to
  interpolate colours smoothly rather than snapping.
- **Params** — duration (150–250ms); the derived shades recomputed from the base rather than
  listed.
- **Use when** — personalisable products, workspace or project colour coding, white-labelling.
- **Don't use when** — the accent carries meaning elsewhere. If your error state is red and a
  user picks red as their accent, the interface has lost a signal.
- **Reduced motion** — colours change instantly.
- **Performance** — one property change repainting the elements that use it; cheap unless the
  transition is applied globally.
- **Gotchas** — **contrast must be enforced, not hoped for.** A user-chosen accent will
  eventually be pale yellow behind white text; derive foreground colours from the chosen value
  and check the ratio, or restrict the palette to swatches that pass. Unregistered custom
  properties do not interpolate — the colour will jump rather than transition.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/@property

---

### Density switch (`theme.density-switch`)

- **One line** — the interface tightens or relaxes its spacing.
- **What the reader sees** — Switch from comfortable to compact and everything closes up: row
  heights shrink, padding reduces, more of the table fits on screen — all in one movement of
  about a quarter of a second, so the layout appears to condense rather than to be rebuilt. It
  is the least common theme change and one of the most valued by people who use a tool all day
  on a small screen.
- **Mechanism** — a spacing scale swapped at the root, with the resulting layout change animated
  where feasible — or applied instantly, which is often the better call.
- **Stack** — CSS custom properties driving padding and row heights.
- **Params** — duration (200–250ms) if animated at all; instant is defensible.
- **Use when** — data-dense tools: tables, mail clients, dashboards, IDE-like interfaces.
- **Don't use when** — the change touches thousands of rows. Animating a density change on a
  large table animates layout properties across the whole document, which is the most expensive
  thing in this catalogue.
- **Reduced motion** — applies instantly.
- **Performance** — this animates *layout*, not transform — the exception to the rule in
  `cross-cutting.md`, and it is only acceptable because it is rare, deliberate and brief. On a
  large table, do not animate it at all.
- **Gotchas** — compact modes routinely break minimum touch-target sizes; enforce a floor
  regardless of the density setting. Preserve scroll position — condensing the layout moves
  everything under the reader otherwise.
- **References** — https://www.setproduct.com/blog/data-table-ui-design

---

### Text size scaling (`theme.text-scale`)

- **One line** — the user makes everything bigger, and the layout copes.
- **What the reader sees** — Increase the text size preference and type grows throughout — not
  just body copy but headings, labels and buttons — while the layout reflows around it: lines
  rewrap, cards get taller, and at the largest setting some multi-column areas become single
  column. Nothing is clipped and nothing overlaps. The reflow is visible and takes a moment, and
  that is fine; what matters is that the page still works at the end of it.
- **Mechanism** — a root font-size or scale custom property with everything sized in relative
  units; the reflow is layout, usually unanimated.
- **Stack** — `rem`-based sizing throughout; the discipline is in never using fixed pixel type.
- **Params** — scale steps (e.g. 87.5% / 100% / 112.5% / 125%); transitions generally off.
- **Use when** — any product with an accessibility settings panel, and by default via honouring
  browser font size.
- **Don't use when** — you would animate the reflow. Watching a whole page resize smoothly is
  disorienting and slow; change it and let it settle.
- **Reduced motion** — instant, which is the default here anyway.
- **Performance** — a full relayout; unavoidable, and rare enough not to matter.
- **Gotchas** — this is the test that finds every fixed height, every `overflow: hidden` clipping
  a label and every button that only fits its text at one size. Zoom and OS text-size settings
  produce the same effects, so the work is required regardless of whether you ship the control.
- **References** — https://www.w3.org/WAI/WCAG22/Understanding/resize-text.html

---

### Contrast mode (`theme.contrast-mode`)

- **One line** — the interface switches to maximum legibility.
- **What the reader sees** — Borders that were subtle become solid. Backgrounds separate more
  strongly from text. Decorative gradients, translucency and low-contrast placeholder greys are
  replaced by flat, high-contrast equivalents. It looks plainer and considerably more legible,
  and for the people who need it, it is the difference between usable and not.
- **Mechanism** — a token set swapped by `prefers-contrast: more`, plus honouring
  `forced-colors: active` where the OS overrides colours entirely.
- **Stack** — CSS media queries and a second token set.
- **Params** — applied instantly; no transition.
- **Use when** — always. This is a preference the platform already exposes.
- **Don't use when** — never; but note that in forced-colors mode the system is overriding your
  palette, and fighting it is the wrong response.
- **Reduced motion** — unrelated preference, but frequently set together; honour both
  independently.
- **Performance** — free.
- **Gotchas** — in `forced-colors: active`, background images and box-shadows may be dropped
  entirely by the OS, so anything conveyed only by a shadow or a background disappears — use
  `forced-color-adjust` deliberately and test rather than assuming. Transitions should be
  suppressed in this mode; someone using it is not there for the flourish.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/@media/forced-colors

---

### Theme preview (`theme.preview-swatch`)

- **One line** — hovering a theme option shows what it will look like before committing.
- **What the reader sees** — In a settings panel, a row of theme swatches. Point at one and a
  small preview card beside it — a miniature of the interface with its own header, sidebar and
  button — restyles to that theme. Move along the row and the miniature changes with each. Pick
  one and the whole page adopts it. You get to try four themes without living through four
  full-page changes.
- **Mechanism** — a scoped preview element with its own theme tokens applied locally, updating
  on hover intent.
- **Stack** — CSS custom properties scoped to the preview container rather than the root.
- **Params** — preview transition (150ms); hover intent delay (~150ms so sweeping the row does
  not strobe).
- **Use when** — products offering more than two or three themes.
- **Don't use when** — there are only light and dark. Just switch.
- **Reduced motion** — the preview updates instantly.
- **Performance** — trivial; the preview is a small scoped subtree.
- **Gotchas** — the preview must be reachable by keyboard and reflect focus as well as hover, or
  it only exists for pointer users. Scope the tokens carefully — a preview that leaks its
  variables to the root restyles the entire page on hover, which is a memorable bug.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/--*

---

### Seasonal / event theme (`theme.seasonal`)

- **One line** — a temporary skin over the normal appearance.
- **What the reader sees** — For a week, the product wears a different coat: a warmer palette, a
  small decoration in the header, perhaps a subtle background pattern. Nothing about the layout
  or the controls changes, and there is a visible way to turn it off. When the period ends it
  reverts on its own and the product looks as it always did.
- **Mechanism** — an additional token layer applied over the base theme, gated by date or a flag,
  with an opt-out stored per user.
- **Stack** — the same token mechanism as any other theme; the governance is the substance.
- **Params** — entrance on first view (a gentle fade, once); an always-available opt-out.
- **Use when** — a genuine brand moment, kept light.
- **Don't use when** — it reduces contrast, adds motion to a working interface, or cannot be
  dismissed. A tool people use daily is not a greetings card.
- **Reduced motion** — no decorative animation; the palette change alone.
- **Performance** — should add nothing measurable; if the seasonal layer ships an animation
  library, it has failed.
- **Gotchas** — every seasonal palette must still pass contrast — festive colours are frequently
  the low-contrast ones. Make the opt-out obvious and persistent, and be aware that a holiday
  theme is not universally welcome; make it easy to decline rather than clever to find.
- **References** — https://dev.to/bishoy_bishai/implementing-dark-mode-css-variables-system-preference-and-persistence-2a43

---

### Themed media swap (`theme.media-swap`)

- **One line** — images and illustrations change with the theme.
- **What the reader sees** — Switch to dark mode and the illustrations change too: the diagram
  that was black-on-white becomes light-on-dark, the logo swaps to its light variant, and the
  screenshot changes to the dark-mode capture. Without this, a dark page is punctuated by
  glowing white rectangles, which is the most common visual failure of a dark theme and the one
  people notice most.
- **Mechanism** — `<picture>` with a `prefers-color-scheme` media source, or theme-aware SVG
  using `currentColor`; a short crossfade if the swap is visible.
- **Stack** — `<picture>` and media queries; SVG with `currentColor` avoids the swap entirely.
- **Params** — crossfade (150–200ms) only when the swap happens live; none on load.
- **Use when** — any site with diagrams, logos, illustrations or screenshots and a dark theme.
- **Don't use when** — the image is a photograph. Photographs do not have a dark variant, and
  dimming them slightly is usually better than inverting them.
- **Reduced motion** — swap without crossfading.
- **Performance** — `<picture>` fetches only the matching source, so this costs nothing extra on
  load; a live swap may fetch a second asset, so preload it if the toggle is prominent.
- **Gotchas** — SVGs authored with `currentColor` follow the theme for free and are always the
  better choice for diagrams and icons. Avoid blanket CSS filter inversion — it turns
  photographs into negatives and rarely produces a legible diagram.
- **References** — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/picture

---

## Family notes

**Prevent the flash before you animate anything.** A blocking inline script in the head that
applies the stored theme before first paint is the highest-value item in this file, and the most
frequently missing.

**Transition at the root, not on everything.** One transition on the root's background and text
colour reads as one change. A transition on `*` produces thousands of animated properties and a
visible ripple.

**Set `color-scheme`.** Without it, native form controls, scrollbars and the browser's own
surfaces stay light inside your dark page.

**A stored choice beats the system; the system beats a default.** Three states — light, dark,
system — with "system" stored explicitly rather than inferred from absence.

**Every theme must pass contrast, including the ones the user invented.** Derive foregrounds
from chosen accents and verify, or constrain the palette to values that already pass.

**Suppress the flourish where the preference says so.** In reduced-motion or forced-colors mode,
change the theme instantly. Someone using those settings is not asking for a circular reveal.
