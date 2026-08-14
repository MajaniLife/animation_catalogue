# Data-visualisation motion

Motion where the movement itself carries quantitative meaning. Bars growing to their values,
a line drawing across a time axis, a scatter plot rearranging between two views, a number
counting to a total.

This family has a constraint no other family has: **it can lie.** Everywhere else, a badly
chosen animation is ugly or annoying. Here, a badly chosen animation changes what the reader
believes the data says. That makes the governing test unusually concrete —

> If removing the motion would change the viewer's interpretation of the data, the animation
> is misleading.

**Object constancy is the central principle.** Each visual element must be locked to a
specific datum: the bar for August is *always* the bar for August, in every state and through
every transition. When a chart re-sorts or re-filters, elements must move to their new
positions rather than being destroyed and recreated, so the reader can follow individual data
points with their eye. Without constancy, elements silently change what they represent
mid-animation, which is the most common way an animated chart becomes a false one.

**Animate to reveal, not to decorate.** The legitimate uses are narrow and worth stating
plainly: showing *change over time*, showing *a transition between two views of the same
data*, and *directing attention* to one figure among many. Almost everything else is
decoration on a chart, and decoration on a chart costs more than decoration elsewhere,
because it competes with the information for the same visual channel.

**Minimise occlusion.** If elements pass over each other during a transition, they cannot be
tracked, and the constancy you carefully preserved is lost anyway. Stagger, route around, or
split the transition into stages.

---

### Bar grow (`dataviz.bar-grow`)

- **One line** — bars rise from the baseline to their values on first view.
- **What the reader sees** — The axes are drawn and the plot area is empty. Then the bars grow
  upward from the zero line, all at once or in quick succession from left to right, reaching
  their heights in about three quarters of a second and stopping cleanly. Because they grow
  *from the baseline*, the direction of growth matches the direction the value is measured in,
  and the animation reinforces rather than confuses the encoding. It is the standard chart
  entrance and one of the few in this family that is essentially always safe.
- **Mechanism** — `scaleY` from 0 with a bottom transform-origin, or animating the bar's height
  or `y` attribute.
- **Stack** — CSS or any charting library's entrance option; D3 transitions for custom work.
- **Params** — duration (0.6–0.8s); stagger (0–0.05s per bar — left to right if the axis is
  ordered, none otherwise); easing (ease-out).
- **Use when** — a chart entering the viewport for the first time.
- **Don't use when** — the chart is one of many on a dashboard the user is scanning, or it
  re-animates on every data refresh.
- **Reduced motion** — bars at full height immediately.
- **Performance** — `scaleY` is compositor-friendly but distorts the bar's stroke and any label
  inside it; animating height is more correct and more expensive. For a handful of bars, prefer
  correctness.
- **Gotchas** — never stagger so slowly that the reader compares bars that have arrived against
  bars still growing — for a moment the chart shows false relationships. Keep the total under
  about a second. Bars for negative values must grow *downward* from the same baseline, which
  naive implementations get wrong.
- **References** — https://blog.pixelfreestudio.com/best-practices-for-animating-data-visualizations/

---

### Line draw (`dataviz.line-draw`)

- **One line** — a time series draws itself from left to right.
- **What the reader sees** — The line begins at the earliest point and extends rightward along
  its true path, rising and falling exactly as the data does, arriving at the present after
  about a second. Because the drawing follows the time axis, it reads as *time passing* — the
  chart is replaying its own history — and peaks arrive in the order they actually occurred,
  which is genuinely informative rather than decorative.
- **Mechanism** — `stroke-dashoffset` from the path length to zero (see `svg.stroke-draw`),
  with `pathLength="1"` removing the measurement step.
- **Stack** — CSS on an SVG path; any charting library that exposes the path.
- **Params** — duration (0.8–1.5s depending on series length); easing (linear — a time axis
  should advance at constant rate, and easing here misrepresents the pacing of time);
  whether points and labels appear as the line passes them.
- **Use when** — a single time series being introduced, especially where the shape is the story.
- **Don't use when** — comparing several series. Lines drawing at once are unreadable, and
  drawing them in sequence implies an order that may not exist.
- **Reduced motion** — the complete line.
- **Performance** — `stroke-dashoffset` repaints but does not reflow; fine for a chart.
- **Gotchas** — **use linear easing.** An ease-out on a time-series draw makes the earlier
  period appear to pass faster than the later one, which is a misrepresentation, not a style
  choice. Data points and tooltips must not be interactive until the line has reached them.
- **References** — https://idl.cs.washington.edu/files/2007-AnimatedTransitions-InfoVis.pdf

---

### Value transition (`dataviz.value-transition`)

- **One line** — the chart moves between two datasets rather than redrawing.
- **What the reader sees** — Switch the year selector and the bars do not blink to new heights
  — each one travels to its new value, some rising, some falling, over about half a second.
  Because every bar keeps its identity, you can watch a single category and see whether it went
  up or down, and how far relative to its neighbours. The chart has told you about *change*,
  which a redraw could not have done.
- **Mechanism** — interpolate each element's encoded value, keyed by its data identity rather
  than by index.
- **Stack** — D3's data joins with keys, or any charting library that supports keyed updates.
- **Params** — duration (0.4–0.7s); easing (ease-in-out); whether entering and exiting elements
  fade separately from the movement of survivors.
- **Use when** — filters, time selectors, any control that changes the data behind a stable
  chart.
- **Don't use when** — the two datasets share no categories. Then nothing is constant and you
  are animating unrelated shapes into each other.
- **Reduced motion** — instant update.
- **Performance** — proportional to element count; thousands of interpolating points is where
  canvas beats SVG.
- **Gotchas** — **key by data identity, never by array index.** Keyed by index, the bar for
  August becomes the bar for September mid-animation, and the reader's eye follows a bar that
  changed meaning without telling them — the exact failure object constancy exists to prevent.
  Handle enter, update and exit as three distinct treatments.
- **References** — https://chartmogul.com/blog/sweating-details-data-animation ·
  https://blog.pixelfreestudio.com/best-practices-for-animating-data-visualizations/

---

### Chart type morph (`dataviz.type-morph`)

- **One line** — the same data restructures from one chart form into another.
- **What the reader sees** — A bar chart becomes a pie: the bars bend and wrap around a centre,
  each retaining its colour, arriving as the corresponding wedge. Or a scatter plot collapses
  into a histogram, points sliding into their bins and stacking up. Because each element keeps
  its identity throughout, you understand that these are two views of the same thing, and you
  can follow the datum you care about from one encoding to the other.
- **Mechanism** — a staged transition: change one visual variable at a time (position, then
  shape, then scale) rather than all at once, since simultaneous changes are hard to track.
- **Stack** — D3 with keyed joins, or a grammar-of-graphics library with transition support.
- **Params** — stage count (2–3); per-stage duration (0.3–0.5s); ordering (usually position
  first, then form).
- **Use when** — teaching a dataset, or an explorer that offers multiple encodings.
- **Don't use when** — the two encodings are not equivalent. Morphing between charts that
  answer different questions implies an equivalence that does not exist.
- **Reduced motion** — swap directly between the two static charts.
- **Performance** — many simultaneously interpolating elements; the staging that makes it
  readable also makes it cheaper per frame.
- **Gotchas** — staged transitions are demonstrably easier to follow than simultaneous ones —
  this is one of the better-established findings in the animated-transitions literature.
  Minimise occlusion: elements passing over one another during the morph cannot be tracked, and
  the constancy you preserved in the data is lost in the visuals.
- **References** — https://idl.cs.washington.edu/files/2007-AnimatedTransitions-InfoVis.pdf ·
  https://arxiv.org/pdf/2507.16563

---

### Counter to value (`dataviz.counter`)

- **One line** — a headline figure counts to its value.
- **What the reader sees** — The number spins up from zero and decelerates onto its final
  value over a second and a half, so your eye arrives at the figure exactly as it settles. It
  makes a statistic feel arrived-at rather than asserted. It is also the most over-used
  technique in this family, and it borrows credibility that the underlying number may not have
  earned.
- **Mechanism** — tween a proxy value, write the formatted output each frame (see
  `text.counter-odometer`).
- **Stack** — any library, or a short `requestAnimationFrame` loop.
- **Params** — duration (1.2–1.8s); easing (strong ease-out); formatting preserved throughout.
- **Use when** — three or four headline metrics, once per page.
- **Don't use when** — the figure is precise and consequential — a price, a balance, a
  measurement. Watching a real number churn undermines trust in it, and for a moment it
  displays values that are simply false.
- **Reduced motion** — the final value immediately.
- **Performance** — one DOM write per frame per counter.
- **Gotchas** — the true value must be in the markup so that no-JS and reduced-motion readers
  see the truth, and only the *final* value should be announced. Tabular figures, or the row
  reflows on every frame.
- **References** — https://chartmogul.com/blog/sweating-details-data-animation

---

### Sort reorder (`dataviz.sort-reorder`)

- **One line** — re-sorting moves the rows instead of redrawing the order.
- **What the reader sees** — Click "sort by value" and the bars slide past each other into
  their new ranking, each keeping its label and colour, arriving in the new order in about half
  a second. You can watch one category travel from seventh to second, which tells you something
  a static re-render never could: not just the new order, but *how far* things moved.
- **Mechanism** — FLIP or keyed transitions on position only; nothing else changes.
- **Stack** — D3 keyed joins, or a layout-animation library.
- **Params** — duration (0.4–0.6s); easing (ease-in-out — elements start and stop together);
  no stagger, so the reordering reads as one event.
- **Use when** — sortable tables and rankings, leaderboards, comparison charts. In a data
  table specifically, `table.sort-reorder` carries the row-count ceiling that governs it.
- **Don't use when** — the list is long. Twenty rows crossing each other is noise, not
  information.
- **Reduced motion** — apply the new order instantly.
- **Performance** — transform-only movement; cheap.
- **Gotchas** — occlusion is the enemy here: rows swapping past each other in a dense chart
  become untrackable. Consider fading non-relevant rows during the transition to reduce visual
  crossing. The new order must also be announced, since the movement is silent.
- **References** — https://blog.pixelfreestudio.com/best-practices-for-animating-data-visualizations/

---

### Scrubbed time series (`dataviz.time-scrub`)

- **One line** — scroll or drag advances the chart through time.
- **What the reader sees** — As you scroll down the article, the chart beside it advances
  through its years: the line extends, the bars change height, the year label ticks upward,
  all exactly in step with your scrolling. Stop and it stops. It turns a chart into something
  you move through rather than look at, and paired with text that explains each period, it is
  the strongest form of data storytelling on the web.
- **Mechanism** — scroll progress mapped to a time index; the chart re-renders at the
  interpolated state (see `scroll.pin-sequence` for the pinning half).
- **Stack** — a scroll library plus a chart that can render arbitrary intermediate states.
- **Params** — pinning; scroll distance per time step; whether intermediate states interpolate
  or step between real data points.
- **Use when** — data-driven narrative journalism, annual reports, explainers.
- **Don't use when** — the reader needs to compare arbitrary periods. Scrubbing enforces
  sequence; a control panel allows comparison.
- **Reduced motion** — show the final or most relevant state, with controls to change it.
- **Performance** — the chart re-renders every scroll frame; keep the element count low or
  render to canvas.
- **Gotchas** — **interpolating between real data points invents data.** If your series is
  annual, stepping between years is honest and interpolating between them draws values that
  were never measured. Where the intermediate frames are shown as a continuous line, say so.
  Provide a non-scroll control as well, so the content is reachable without the gesture.
- **References** — https://idl.cs.washington.edu/files/2007-AnimatedTransitions-InfoVis.pdf

---

### Highlight focus (`dataviz.highlight-focus`)

- **One line** — one series brightens while the others recede.
- **What the reader sees** — Hover a legend entry and that line stays at full strength while
  every other line fades back to a light grey, over about two hundred milliseconds. The
  highlighted series is suddenly legible in a way it was not amongst a dozen competitors, and
  the others remain visible enough to keep the context. Move away and everything returns.
- **Mechanism** — opacity or colour transitions applied to non-selected series.
- **Stack** — CSS transitions driven by a class, or the chart library's own highlight support.
- **Params** — dim level (0.15–0.3 opacity — enough to keep context, not enough to compete);
  duration (150–250ms).
- **Use when** — multi-series line charts, dense scatter plots, any chart with more than four
  categories.
- **Don't use when** — there are only two series; the contrast does the work already.
- **Reduced motion** — apply the highlight instantly; this is a state change, not travel.
- **Performance** — opacity changes across many elements; group them so one class toggle
  affects the set rather than animating each element separately.
- **Gotchas** — the dimmed state must still meet contrast requirements if any of it must be
  readable. Highlighting must also be reachable by keyboard and touch — hover-only highlighting
  makes a dense chart unusable on a phone.
- **References** — https://skopx.com/resources/data-visualization-best-practices-2026-analyst-guide

---

### Threshold pulse (`dataviz.threshold-pulse`)

- **One line** — a value crossing a limit announces itself.
- **What the reader sees** — A monitoring dashboard, and one figure crosses its threshold: the
  tile's border pulses amber twice, slowly, and settles into a persistent amber state. The
  pulse is what catches your eye from across a room; the persistent state is what tells you the
  condition is still true when you look directly at it. Two decaying pulses, and then it stops
  — a permanently flashing tile is ignored within a minute.
- **Mechanism** — a decaying opacity or scale pulse triggered on state entry, then a static
  changed state.
- **Stack** — CSS keyframes with a finite iteration count, triggered by a class change.
- **Params** — pulse count (2–3, never infinite); period (0.6–1s); the resting alert state.
- **Use when** — monitoring, alerting, live dashboards.
- **Don't use when** — thresholds are crossed frequently. Constant pulsing is noise, and the
  alert loses meaning.
- **Reduced motion** — no pulse; go straight to the alert state, which must be distinguishable
  by more than colour.
- **Performance** — trivial.
- **Gotchas** — never flash more than three times per second — that is a seizure risk, and it
  is a hard limit, not a guideline. The alert must be announced in a live region, and the state
  must be distinguishable without colour.
- **References** — https://www.w3.org/WAI/WCAG22/Understanding/three-flashes-or-below-threshold.html

---

### Map flow (`dataviz.map-flow`)

- **One line** — animated paths show movement between geographic points.
- **What the reader sees** — Arcs spring between cities on a map, each drawing from origin to
  destination, sometimes with a travelling dot following the route. Several routes draw in
  sequence and remain as a network of lines. It reads immediately as *flow* — trade, traffic,
  migration, requests hitting servers — and the drawing order can encode volume or time.
- **Mechanism** — path drawing (`svg.stroke-draw`) plus optional motion-path travellers, on
  projected geographic coordinates.
- **Stack** — a mapping library plus SVG path animation; the projection is the hard part, not
  the motion.
- **Params** — draw duration per arc (0.5–1s); stagger; whether arcs persist or fade.
- **Use when** — origin-destination data, network topology, logistics.
- **Don't use when** — the flows are dense. Fifty animated arcs is a scribble; aggregate first.
- **Reduced motion** — static arcs.
- **Performance** — many stroked paths repainting; batch the draws and cap the count.
- **Gotchas** — arc curvature is a visual convention, not data — make sure nobody reads the
  bulge of the arc as a route or a magnitude. Provide a table equivalent, because an animated
  map is entirely inaccessible without one.
- **References** — https://learnlyai.co.uk/en/course-hub/design/3d-design-animation/motion-graphics/data-visualization-and-information-animation

---

### Live stream update (`dataviz.live-update`)

- **One line** — a real-time chart admits new data without jumping.
- **What the reader sees** — A metrics chart updating every few seconds. Rather than the whole
  plot jumping left each time a point arrives, the line slides smoothly and continuously
  leftward as new data enters at the right edge, so the chart appears to scroll past a fixed
  window. Nothing blinks and nothing jumps; the newest value is always at the same place, which
  is where your eye already is.
- **Mechanism** — the x-axis domain shifts continuously while new points are appended; the
  translation matches the data rate exactly.
- **Stack** — a charting library with streaming support, or a canvas renderer with a ring
  buffer.
- **Params** — window length; update rate; whether the axis slides continuously or steps per
  point.
- **Use when** — monitoring, telemetry, anything genuinely live.
- **Don't use when** — updates are infrequent or irregular. A chart that slides once a minute
  just looks broken; step it instead.
- **Reduced motion** — step the chart on each update rather than sliding continuously.
- **Performance** — a permanently animating chart is a permanently running render loop. Use
  canvas past a few hundred points, and stop entirely when the tab is hidden or the chart is
  off-screen.
- **Gotchas** — the slide rate must match the data rate or the chart drifts out of sync and
  either stalls or jumps to catch up. Do not animate the y-axis domain at the same time as the
  x — a chart rescaling while it scrolls makes trends impossible to read.
- **References** — https://skopx.com/resources/data-visualization-best-practices-2026-analyst-guide

---

## Family notes

**The honesty test.** If removing the animation would change what the reader concludes from
the data, the animation is misleading. Apply it to every effect here before shipping it.

**Object constancy above all.** Key elements by data identity, never by index. The bar for
August must be the bar for August in every frame of every transition, or the reader is
following something that silently changed meaning.

**Stage complex transitions.** Changing position, then shape, then scale is measurably easier
to follow than changing everything at once — and it reduces occlusion, which is what destroys
trackability in practice.

**Easing encodes pacing.** Linear for anything mapped to time; eased for anything mapped to
quantity. An ease-out on a time axis literally misrepresents how fast time passed.

**Never invent intermediate data.** Interpolating between annual figures draws values nobody
measured. Step between real observations, or be explicit that the line is a continuous
estimate.

**A chart's motion is silent.** Every animated view needs a static, accessible equivalent —
a table, a summary, a described trend. This family carries meaning in motion, which means it
carries meaning that a screen reader user, and anyone with reduced motion enabled, will never
receive from the animation itself.
