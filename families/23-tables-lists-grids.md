# Tables, lists & data grids

Motion in dense, repetitive, scrollable data. Sorting, filtering, expanding, selecting,
editing, and scrolling through more rows than can exist in the DOM at once.

This family inverts the usual relationship between motion and quantity. Everywhere else an
effect is applied to a handful of elements; here it is potentially applied to thousands, and
the animation that delights on one card becomes unusable on a thousand rows. **Almost every
decision in this file is really a decision about how many elements are allowed to animate at
once.**

**The number that governs everything: about 1,000 rows.** Past roughly that, client-side
rendering begins to stutter and you should already have moved to virtualization or server-side
paging. Virtualization — windowing — renders only the rows in the viewport plus a small buffer,
so a 50,000-row table costs about what a 50-row one does. That single technique is what makes
large tables possible, and it is also what breaks most of the animation in this file, because
rows entering and leaving the DOM constantly is indistinguishable from rows being added and
removed.

**Semantics do the accessibility work here, not ARIA.** Real `<table>`, `<thead>`, `<tbody>`
and `<th scope>` give keyboard and screen-reader support very nearly for free. Virtualized
components, by rendering only part of the dataset, break that — a screen reader cannot perceive
rows that are not in the DOM — which is a genuine trade-off to manage rather than a bug to fix
with attributes.

---

### Sort reorder (`table.sort-reorder`)

- **One line** — re-sorting slides rows to their new positions.
- **What the reader sees** — Click a column header and its sort arrow flips, and the rows move
  — the row that was seventh gliding up to second, the ones it passes sliding down — everything
  arriving in the new order in about a third of a second. You can follow a single row you were
  interested in, which tells you not just the new ranking but how far things moved. On a table
  of twenty rows this is genuinely informative. On a table of five hundred it is chaos, which is
  why the row count decides whether to animate at all.
- **Mechanism** — FLIP across visible rows only (see `layout.flip-reposition`); rows outside the
  viewport are repositioned without animation.
- **Stack** — a layout-animation library, or View Transitions on the table body.
- **Params** — duration (300–400ms); no stagger; **row-count ceiling (~50 visible) above which
  you skip the animation entirely**.
- **Use when** — small to medium tables where individual rows are identifiable.
- **Don't use when** — the table is virtualized, paginated, or long enough that most rows move
  off screen. Animating rows into positions the user cannot see is wasted work.
- **Reduced motion** — the new order applies instantly.
- **Performance** — animate only rows currently in the viewport; a naive implementation animates
  all of them and drops frames on the sort that most needed to feel fast.
- **Gotchas** — the header's sort state needs `aria-sort`, and the new order must be announced;
  the movement is silent. Keyed by row identity, never by index, or rows swap meaning mid-flight.
- **References** — https://www.setproduct.com/blog/data-table-ui-design

---

### Row expand (`table.row-expand`)

- **One line** — a row opens downward to reveal its detail.
- **What the reader sees** — Click a row and it unfolds: a panel grows out beneath it pushing
  the following rows down, its contents fading in once the space exists, and a chevron on the
  row rotates to point down. The row itself stays exactly where it is, so you keep your place in
  the table. Collapse it and everything closes back up. It is the least disruptive way to show
  detail, because the surrounding context never leaves the screen.
- **Mechanism** — a height transition on the detail row (the `0fr → 1fr` grid technique from
  `layout.height-auto`) plus a chevron rotation and a content fade.
- **Stack** — CSS; the grid-rows technique needs no measurement.
- **Params** — expand (250–350ms); chevron rotation (200ms); content fade after the geometry
  settles; single-open or multi-open.
- **Use when** — tabular data with supporting detail: orders, logs, records.
- **Don't use when** — the table is virtualized. Variable row heights and windowing fight each
  other, and the scroll position jumps as rows resize.
- **Reduced motion** — the detail appears instantly.
- **Performance** — a height animation reflows everything below it; on a long table that is the
  whole remaining body. Keep panels small and durations short.
- **Gotchas** — the trigger needs `aria-expanded` and the detail must be associated with its
  row. Expanding a row near the bottom should scroll it into view, or the content opens off
  screen and appears to have done nothing.
- **References** — https://www.setproduct.com/blog/data-table-ui-design

---

### Sticky header (`table.sticky-header`)

- **One line** — column headers stay put while the body scrolls under them.
- **What the reader sees** — Scroll down a long table and the header row does not leave — it
  stays pinned at the top, and as the first body row passes beneath it a shadow appears under
  the header, lifting it visually above the content. Scroll back to the top and the shadow
  fades away. Without the shadow the header looks like part of the table; with it, it clearly
  floats above, and you always know which column you are reading.
- **Mechanism** — `position: sticky` on the header cells, with a shadow transition triggered by
  a sentinel element crossing the top.
- **Stack** — CSS sticky is native; the stuck-state detection uses an IntersectionObserver
  sentinel (there is no `:stuck` selector).
- **Params** — shadow transition (150–200ms); shadow depth (subtle).
- **Use when** — any table taller than the viewport.
- **Don't use when** — never, really. This is a baseline expectation for tabular data.
- **Reduced motion** — sticking is not motion and stays; the shadow can appear instantly.
- **Performance** — native and free; far cheaper than a JavaScript-positioned clone header.
- **Gotchas** — sticky fails silently if any ancestor has `overflow: hidden` or `auto`, which is
  the single most common reason it does not work. With both sticky headers **and** sticky
  columns, the corner cell where the two axes meet needs the highest z-index or it is
  overlapped by one of them.
- **References** — https://mashuktamim.medium.com/building-sticky-headers-and-columns-with-react-window-a-complete-guide-3093f87b0642

---

### Selection & bulk bar (`table.selection-bar`)

- **One line** — selecting rows summons a bar of actions for them.
- **What the reader sees** — Tick a row's checkbox and the row tints slightly. Tick another, and
  a bar slides up from the bottom of the screen: "2 selected", with Delete, Export and Archive.
  Each further selection ticks the count. Clear the selection and the bar slides away. The bar
  appearing is what tells you that selection has a purpose — without it, checkboxes are a
  gesture with no visible consequence.
- **Mechanism** — a row background transition, a count update (`micro.count-change`), and a
  slide-in action bar on the transition from zero to one selection.
- **Stack** — CSS transitions plus selection state.
- **Params** — row tint (100–150ms); bar slide (250ms); count update (150ms).
- **Use when** — any table supporting multi-row operations.
- **Don't use when** — only one row can be acted on at a time; use a row-level menu instead.
- **Reduced motion** — the bar appears without sliding.
- **Performance** — selecting all on a large table must not animate every row; apply the state
  via a parent class rather than per-row transitions.
- **Gotchas** — the count must be announced, and the header checkbox needs an indeterminate
  state for partial selection. Selection must survive sorting and filtering, or people lose
  work invisibly; if it cannot, say so before the operation.
- **References** — https://www.setproduct.com/blog/data-table-ui-design

---

### Inline edit (`table.inline-edit`)

- **One line** — a cell becomes editable in place.
- **What the reader sees** — Double-click a cell and it turns into an input without changing
  size or position — the border appears, the text becomes selectable, the cursor lands at the
  end — over about 120 milliseconds. Type, press Enter, and it reverts to plain text with a
  brief green flash confirming the save. Escape reverts it unchanged. The cell never moves, so
  the table's grid stays absolutely stable while you work down a column.
- **Mechanism** — a border and background transition on the cell, with the input replacing the
  text at identical metrics; a save-confirmation flash on commit.
- **Stack** — CSS transitions plus an editable state; matching the input's font metrics to the
  cell's is the fiddly part.
- **Params** — enter edit (100–150ms); save flash (400–600ms, fading out); revert (instant).
- **Use when** — spreadsheet-like data, admin tables, bulk correction workflows.
- **Don't use when** — the edit needs validation the cell cannot show, or has side effects
  beyond the cell.
- **Reduced motion** — no transitions; the save confirmation becomes a static marker.
- **Performance** — trivial per cell, but do not mount inputs for every cell in advance.
- **Gotchas** — the input must match the cell's font, padding and alignment exactly or the
  table twitches on every edit — the most visible failure of this pattern. Keyboard support is
  the feature: Enter commits, Escape reverts, Tab moves to the next cell. Announce saves.
- **References** — https://www.setproduct.com/blog/data-table-ui-design

---

### Column resize (`table.column-resize`)

- **One line** — dragging a column edge changes its width live.
- **What the reader sees** — Move over the boundary between two headers and the cursor becomes a
  resize handle as a thin line appears. Drag, and the column follows your pointer exactly while
  the neighbouring columns adjust, text rewrapping as space changes. Release and it stays. There
  is no easing at all during the drag — the edge is under your finger, and any smoothing would
  read as the table lagging behind you.
- **Mechanism** — direct width application during the drag with no transition; optionally a
  guide line rather than live resizing for very large tables.
- **Stack** — pointer events plus width or grid-template updates.
- **Params** — no easing during drag; minimum column width; whether resizing is live or via a
  guide line.
- **Use when** — data tables where content width varies and users compare columns.
- **Don't use when** — the layout is fixed by design, or on touch, where the handle is too small
  to hit reliably.
- **Reduced motion** — unaffected; this is direct manipulation.
- **Performance** — live resizing reflows the entire table on every pointer move. On large or
  virtualized tables, drag a guide line and apply the width on release instead.
- **Gotchas** — persist widths per user or they are lost on every visit, which makes the feature
  feel pointless. Provide a keyboard path — resize handles are pointer-only otherwise — and
  enforce a minimum width so a column cannot be dragged to invisibility.
- **References** — https://dev.to/zeeshanali0704/frontend-system-design-virtualization-handling-large-data-sets

---

### Virtualized scroll (`table.virtual-scroll`)

- **One line** — only the visible rows exist, and the scroll must not betray that.
- **What the reader sees** — A list of fifty thousand rows that scrolls exactly like a list of
  fifty: smooth, with a scrollbar whose size and position are proportionate to the whole set.
  Scroll fast and there is no blank space, no flicker, no rows arriving late. The entire success
  criterion for this technique is that **the reader never notices it is happening** — every
  visible artefact is a failure.
- **Mechanism** — render only rows in the viewport plus a buffer, absolutely positioned inside a
  spacer of the full scroll height; recycle rows as the window moves.
- **Stack** — TanStack Virtual, react-window, or Virtuoso handle the windowing maths;
  `content-visibility: auto` is a lighter CSS-only option that skips rendering and painting of
  off-screen content without removing it from the DOM.
- **Params** — overscan buffer (3–10 rows — larger on fast scroll); fixed vs measured row
  heights.
- **Use when** — beyond about 1,000 rows, where client-side rendering begins to stutter.
- **Don't use when** — the dataset is small. Virtualization costs you find-in-page, native
  keyboard navigation and simple accessibility for no benefit under a few hundred rows.
- **Reduced motion** — unaffected; there is no animation here to reduce.
- **Performance** — this *is* the performance technique. It also **forbids most of this file**:
  entrance animations on recycled rows fire constantly as rows scroll in, which looks broken and
  wastes frames. Rows must appear plainly.
- **Gotchas** — **the accessibility cost is real**: screen readers and find-in-page cannot
  perceive rows that are not in the DOM. Mitigate with correct `aria-rowcount`/`aria-rowindex`,
  keyboard navigation that scrolls the window rather than relying on tab order, and — where the
  data must be fully searchable — a print or export view that is not virtualized. Variable row
  heights need measurement and are where most jitter originates.
- **References** — https://www.patterns.dev/vanilla/virtual-lists/ ·
  https://web.dev/articles/virtualize-long-lists-react-window ·
  https://app.studyraid.com/en/read/11538/362764/ensuring-accessibility-in-virtualized-components

---

### Skeleton rows (`table.skeleton-rows`)

- **One line** — placeholder rows hold the table's shape while data loads.
- **What the reader sees** — The table's headers are there, and beneath them a dozen rows of
  grey blocks in the shape of the real columns — a wide block for the name, a short one for the
  date, a pill for the status — shimmering gently. When the data arrives it replaces them in
  place with no movement at all, because the placeholder rows were exactly the right height.
- **Mechanism** — placeholder rows matching the real row height, with a shimmer
  (`micro.skeleton-shimmer`) and a crossfade on swap.
- **Stack** — CSS; the row height accuracy is the entire trick.
- **Params** — row count (fill the viewport, no more); shimmer period (1.5–2s); crossfade
  (150–250ms).
- **Use when** — tables whose column structure is known before the data arrives.
- **Don't use when** — under about 300ms of wait, or when row heights vary unpredictably.
- **Reduced motion** — static placeholders without shimmer.
- **Performance** — each shimmering row is a running animation; a screenful of them competes
  with the fetch. Consider shimmering only a few rows and leaving the rest flat.
- **Gotchas** — placeholder rows must match real row height precisely or the table jumps on
  swap — the exact shift the skeleton existed to prevent. Mark the region `aria-busy` and hide
  the placeholders from assistive technology.
- **References** — https://www.setproduct.com/blog/data-table-ui-design

---

### Live row update (`table.live-update`)

- **One line** — a row's values change while you are looking at the table.
- **What the reader sees** — A monitoring table where one row's status changes: the cell's
  background flashes briefly — a pale tint fading over about a second — and the new value is
  there. Nothing moves, nothing re-sorts, the row stays exactly where it was. The flash is
  enough to catch peripheral attention without pulling you away from the row you were reading,
  and its decay means a table with frequent updates does not become a light show.
- **Mechanism** — a decaying background transition on the changed cell, with the value swapping
  in place.
- **Stack** — CSS transition triggered by a data change.
- **Params** — flash peak (immediate), decay (800ms–1.2s); tint colour by change type.
- **Use when** — dashboards, monitoring, order books, collaborative tables.
- **Don't use when** — many rows change at once. A whole table flashing conveys nothing; batch
  the update and report a count instead.
- **Reduced motion** — the value changes with a static marker rather than a flash.
- **Performance** — trivial per cell; cap the number of simultaneous flashes.
- **Gotchas** — **do not re-sort live.** A table that reorders itself while someone is reading
  it is the most disorienting thing in this file — hold the sort and offer a "new order
  available" control instead. Announce changes sparingly and in aggregate.
- **References** — https://www.setproduct.com/blog/data-table-ui-design

---

### Pagination change (`table.pagination`)

- **One line** — moving between pages of results.
- **What the reader sees** — Press Next and the rows do not slide sideways or fade out
  dramatically — they are simply replaced, with the table's height held steady and the scroll
  position returned to the top of the table rather than the top of the page. The page indicator
  advances. The restraint is deliberate: an animated page change makes the wait feel longer and
  gives no information you did not already have from pressing the button.
- **Mechanism** — a very short crossfade at most; height reserved to prevent collapse during the
  swap.
- **Stack** — a fade, or nothing at all.
- **Params** — crossfade (0–150ms); scroll to the table's top, not the page's.
- **Use when** — server-paged results, which remains the right answer for very large datasets.
- **Don't use when** — you are tempted to slide pages horizontally. It implies a spatial
  relationship between pages that does not survive sorting or filtering.
- **Reduced motion** — instant replacement.
- **Performance** — reserving the table's height across the swap prevents a layout jump that
  otherwise scrolls the page under the user.
- **Gotchas** — move focus to the table or its first row after a page change and announce the
  new page, or keyboard users are left focused on a button with no idea what happened. Preserve
  sort, filter and selection state across pages, or say clearly that selection is per page.
- **References** — https://dev.to/zeeshanali0704/frontend-system-design-virtualization-handling-large-data-sets

---

### Horizontal scroll affordance (`table.scroll-shadow`)

- **One line** — a shadow at the edge shows there are more columns.
- **What the reader sees** — A wide table on a narrow screen. At the right edge a soft shadow
  sits over the content, implying that it continues beyond the visible area; scroll sideways and
  that shadow fades while a matching one appears on the left. When you reach either end its
  shadow disappears entirely. It is a two-pixel gradient doing the job of a scrollbar that
  modern platforms hide by default.
- **Mechanism** — gradient overlays at each edge whose opacity is tied to scroll position, or
  the CSS `background-attachment: local` shadow technique.
- **Stack** — CSS, with an optional scroll listener for precise fading.
- **Params** — shadow width (16–32px); fade (150ms); tied to scroll offset at each end.
- **Use when** — any horizontally scrollable table or region, which on mobile is most of them.
- **Don't use when** — the overflow is obvious from a partially visible column, which is a
  better affordance where the layout allows it.
- **Reduced motion** — the shadows can appear and disappear without transition.
- **Performance** — trivial; the CSS-only version costs nothing at all.
- **Gotchas** — hidden scrollbars mean many users have no idea a table scrolls sideways; this is
  a genuine discoverability problem, not decoration. Keyboard users must be able to scroll the
  container — give it `tabindex="0"` and an accessible name, or the columns off screen are
  unreachable.
- **References** — https://www.setproduct.com/blog/data-table-ui-design

---

### Grouped section collapse (`table.group-collapse`)

- **One line** — grouped rows fold away under their heading.
- **What the reader sees** — A table grouped by category, each group under a heading with a
  count and a chevron. Click a heading and its rows collapse upward into it, the chevron
  rotating, the rows below sliding up to take the space — over about a quarter of a second.
  Collapse three groups and a hundred-row table becomes a readable summary of five headings you
  can expand one at a time.
- **Mechanism** — a height or grid-rows collapse on the group's row container plus a chevron
  rotation.
- **Stack** — CSS transitions; sticky group headers pair naturally with this.
- **Params** — collapse (200–300ms); chevron (200ms); collapse faster than expand.
- **Use when** — naturally grouped data: by date, owner, category, status.
- **Don't use when** — the table is virtualized, where variable container heights and windowing
  conflict.
- **Reduced motion** — groups collapse instantly.
- **Performance** — collapsing a group with hundreds of rows animates a very large height
  change; consider hiding immediately and animating only the neighbouring shift.
- **Gotchas** — the heading needs `aria-expanded` and its row count should be visible when
  collapsed, or the summary hides how much is inside. Persist collapsed state across sorting and
  reloading, or the table resets every time someone interacts with it.
- **References** — https://mashuktamim.medium.com/building-sticky-headers-and-columns-with-react-window-a-complete-guide-3093f87b0642

---

## Family notes

**Row count decides everything.** Under ~50 visible rows, animate freely. Toward 1,000, animate
only what is in the viewport. Beyond that, virtualize — and accept that virtualization forbids
most of this file.

**Virtualization is a trade, not a free win.** It makes 50,000 rows feel like 50, and it costs
you find-in-page, simple screen-reader access and native keyboard traversal. Manage that
explicitly with row-count semantics and a non-virtualized export path; do not pretend it is
solved.

**Semantic markup carries the accessibility.** `<table>`, `<thead>`, `<th scope>` give you most
of it for free. Every div-based grid starts by discarding that and rebuilding it worse.

**Never move data under a reading user.** No live re-sorting, no rows shifting as updates
arrive, no page jumps on expand. Hold the change and offer it as a control.

**Stability beats delight here.** A table's job is comparison, and comparison requires things to
stay where they were. The best-behaved table in this file is the one whose grid never twitches.
