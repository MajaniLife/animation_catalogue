# Loading, progress & streaming states

Motion that occupies the interval between asking for something and receiving it. Spinners,
skeletons, progress bars, staged reveals, and — increasingly the dominant case — content that
arrives a piece at a time rather than all at once.

This family is about **perceived** performance, which is a different quantity from measured
performance and frequently more important. The same ten-second wait can feel like a stall or
like progress depending entirely on what happens during it, and nothing in this file makes
anything faster.

**The thresholds this family is built on** (see also `families/10`): 0.1s feels instantaneous,
1s preserves an uninterrupted flow of thought, 10s is the outer limit of attention. From those:
delay a spinner by ~250ms so fast operations never flash one, and hold it briefly once shown so
it never flickers.

**Streaming changed the default.** Progressive delivery turns a ten-second wait into a
one-second start, and it makes a response feel dramatically faster even when total generation
time is identical — because the reader is engaged throughout rather than watching a spinner.
For any language-model interface this is now the baseline expectation: a response that waits
until completion before rendering **feels broken** by comparison. The metric that governs the
felt experience is **time-to-first-token** — when the user sees *anything* — not total duration.

The corresponding rule for AI operations specifically: **never show a static spinner for more
than about a second.** Past that, show evidence of actual processing.

---

### Token stream (`load.token-stream`)

- **One line** — generated text appears progressively as it is produced.
- **What the reader sees** — A response begins almost immediately — a few words, then a few
  more, arriving in small bursts rather than smoothly, at roughly reading speed. A thin cursor
  sits at the leading edge, blinking, marking where the next text will appear. You start reading
  the first sentence while the third is still being written, and by the time you reach the end
  it is usually already there. The wait has not been removed; it has been converted into
  something you spend reading rather than watching.
- **Mechanism** — chunks appended to the DOM as they arrive over a streamed response, with a
  trailing cursor element; text is not animated per character, it is simply inserted.
- **Stack** — a streaming API response plus incremental DOM updates. The animation is the
  cursor, nothing else.
- **Params** — cursor blink (~500ms period, ~2px wide); whether to smooth bursty chunks into a
  steadier rate; scroll-follow behaviour.
- **Use when** — any language-model output long enough that waiting for completion is felt.
- **Don't use when** — the output is a short structured value. Streaming a single number
  arriving character by character is theatre.
- **Reduced motion** — the cursor stops blinking; text still streams, since that is data
  arriving rather than decoration.
- **Performance** — appending on every chunk can thrash layout on long responses; batch to
  animation frames and avoid re-rendering the whole message per token.
- **Gotchas** — auto-scrolling to follow the stream fights a user who has scrolled up to
  re-read; follow only while they are at the bottom. Announce the completed message rather than
  each chunk, or a screen reader is flooded. Streaming must be cancellable, and the cancel
  control needs to be present from the first token.
- **References** — https://www.aiuxplayground.com/pattern/streaming/ ·
  https://thefrontkit.com/blogs/what-is-streaming-ui-in-ai-applications ·
  https://www.digitalapplied.com/blog/ai-model-latency-benchmarks-2026-ttft-throughput

---

### Thinking indicator (`load.thinking`)

- **One line** — three dots pulse while a response is being prepared.
- **What the reader sees** — Where the reply will appear, three small dots fade in and out in
  sequence, left to right, on a loop of about a second and a half. It is the established
  convention for "something is composing a message", borrowed wholesale from chat applications,
  and it reads as *someone is working on it* in a way a spinner does not — a spinner says the
  machine is busy, dots say a reply is coming.
- **Mechanism** — three elements with staggered opacity or translate loops.
- **Stack** — CSS keyframes with per-dot animation delays.
- **Params** — cycle (1.2–1.6s); stagger (0.15–0.2s per dot); amplitude if they move as well as
  fade.
- **Use when** — conversational interfaces, in the interval before the first token arrives.
- **Don't use when** — the operation is not conversational. Dots imply a reply, and using them
  for a file upload is a category error.
- **Reduced motion** — a static indicator, or the word "Thinking…" with no animation.
- **Performance** — three permanently animating elements; trivial, but remove them the moment
  the first token lands rather than leaving them running underneath.
- **Gotchas** — it must be replaced by content, not accumulate above it. Announce the busy state
  once via a live region; do not announce the animation. If time-to-first-token exceeds a couple
  of seconds, dots stop being reassuring — escalate to something that reports what is happening.
- **References** — https://thefrontkit.com/blogs/ai-chat-ui-best-practices ·
  https://origin-main.com/laravel-architecture/laravel-ai-streaming-ux-states/

---

### Staged reveal (`load.staged-reveal`)

- **One line** — the page shows each part as it becomes ready instead of waiting for all of it.
- **What the reader sees** — The header and navigation are there immediately. A moment later the
  main article appears. The comments arrive after that, then the recommendations at the bottom.
  Each section fades in as its data resolves, in roughly the order you would read them. You can
  begin reading the article while the rest of the page is still assembling, and nothing you have
  already read moves.
- **Mechanism** — streamed HTML or progressive hydration, with each section fading in as it
  resolves; space reserved for everything from the start.
- **Stack** — server streaming with framework suspense boundaries; the animation is a plain fade.
- **Params** — fade (200–300ms); no stagger beyond the natural arrival order; reserved
  dimensions per region.
- **Use when** — pages composed of independently-fetched regions of differing speed.
- **Don't use when** — regions resolve within a few hundred milliseconds of each other; you are
  animating noise.
- **Reduced motion** — sections appear without fading.
- **Performance** — this is a genuine improvement, not a perceptual trick: the reader gets
  usable content sooner. It also protects the LCP element from being blocked by slower regions.
- **Gotchas** — reserve every region's space in advance or the page shifts as sections land,
  which is a CLS failure and far worse than the wait. Later sections arriving must never move
  content the user is already reading.
- **References** — https://www.institutepm.com/knowledge-hub/streaming-ai-responses

---

### Determinate progress (`load.determinate`)

- **One line** — a bar reports actual progress toward a known total.
- **What the reader sees** — A bar filling in proportion to work genuinely completed, with a
  percentage or a count beside it — "14 of 40 files". It advances unevenly, because real work is
  uneven, and that unevenness is what makes it credible. When it stalls you know something is
  slow rather than broken, and when it reaches the end it stops there rather than hanging at
  99%.
- **Mechanism** — `scaleX` or width driven by a real completion ratio, with a short transition so
  each update glides rather than jumps.
- **Stack** — CSS transition on a `<progress>` element or a styled bar.
- **Params** — update transition (200–300ms); whether to show a count, a percentage, or both.
- **Use when** — uploads, multi-file operations, batch jobs — anything with a countable total.
- **Don't use when** — you do not actually know the total. A fake determinate bar that sits at
  95% is the most distrusted pattern in this family.
- **Reduced motion** — the bar updates without transition.
- **Performance** — trivial. Throttle updates to a few per second; a bar updating per byte is
  wasted work.
- **Gotchas** — use a real `<progress>` or set `aria-valuenow`, or the state is invisible to
  assistive technology. Never let it move backwards; if the total changes, adjust the scale
  rather than retreating.
- **References** — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/progress

---

### Indeterminate bar (`load.indeterminate`)

- **One line** — a segment travels along a track when the duration is unknown.
- **What the reader sees** — A thin track with a bright segment sweeping repeatedly from left to
  right, never filling, never suggesting a proportion. It says "working, duration unknown" —
  which is honest — and its constant motion distinguishes a slow operation from a frozen one.
  Because it never approaches an end, it never makes a promise it cannot keep, and that is
  precisely why it is the right choice when you do not know.
- **Mechanism** — a looping `translateX` on a partial-width segment inside a clipped track.
- **Stack** — CSS keyframes; `<progress>` without a value gives the native version.
- **Params** — cycle (1.5–2s); segment width (25–35% of track).
- **Use when** — waits of unknown duration where a spinner is too small to notice.
- **Don't use when** — you could report real progress. Prefer determinate whenever the total is
  knowable.
- **Reduced motion** — a static filled track, or a text status. This is continuous motion.
- **Performance** — one looping transform; pause it if the operation completes but the element
  lingers.
- **Gotchas** — subject to the five-second pause rule if it runs long alongside other content.
  Set `aria-busy` on the region it describes rather than relying on the bar alone.
- **References** — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/progress

---

### Skeleton screen (`load.skeleton`)

- **One line** — the shape of the content stands in for the content.
- **What the reader sees** — Grey blocks arranged exactly like the article, card or table that
  is coming: a wide block for the title, three narrower lines for the body, a square for the
  avatar. A soft band drifts across them. Because the layout is already correct, when the real
  content arrives it simply appears in place — nothing moves, nothing jumps, and the transition
  from waiting to reading is almost imperceptible.
- **Mechanism** — placeholder elements matching the real content's dimensions, with a shimmer
  loop (see `micro.skeleton-shimmer`) and a crossfade on swap.
- **Stack** — CSS; the accuracy of the shapes is the real work.
- **Params** — shimmer period (1.5–2s); crossfade (150–250ms); minimum display (~300ms).
- **Use when** — content whose shape is predictable: feeds, cards, tables, profiles. For
  tabular data specifically, `table.skeleton-rows` covers row-height matching.
- **Don't use when** — the shape is unknown, or the wait is under ~300ms. A skeleton that
  guesses wrong creates the layout shift it existed to prevent.
- **Reduced motion** — static placeholders, no shimmer.
- **Performance** — each shimmer is a permanent animation competing with the fetch you are
  waiting for. Prefer fewer, larger placeholders over dozens of small ones.
- **Gotchas** — measure the real content and match the skeleton to it, never the reverse. Hide
  skeletons from assistive technology and announce the loading state properly; a screen reader
  reading out empty placeholder divs is worse than silence.
- **References** — https://thefrontkit.com/blogs/ai-chat-ui-best-practices

---

### Optimistic placeholder (`load.optimistic-item`)

- **One line** — your submission appears immediately, visibly provisional.
- **What the reader sees** — Send a message and it appears in the thread instantly, in position,
  but slightly faded with a small clock icon beside it. A moment later it solidifies to full
  opacity and the icon becomes a tick. If it fails, it turns red with a retry control. You never
  wait to see your own contribution, and the provisional styling is honest about the fact that
  it has not landed yet.
- **Mechanism** — render locally on submit at reduced opacity, transition to the settled state on
  confirmation, or to an error state on failure.
- **Stack** — state management plus two short transitions.
- **Params** — provisional opacity (0.5–0.7); settle transition (200ms); error transition
  (slower and more visible than the settle).
- **Use when** — chat, comments, collaborative editing, anything conversational.
- **Don't use when** — the action is consequential and irreversible.
- **Reduced motion** — the same states without transitions.
- **Performance** — the fastest possible interface; the network leaves the perceived interaction
  entirely.
- **Gotchas** — the failure path is the whole design. A provisional item that quietly vanishes
  is worse than a wait; it must persist, explain, and offer retry. Announce state changes, and
  do not let a pending item be edited into an inconsistent state.
- **References** — https://www.aiuxplayground.com/pattern/streaming/

---

### Infinite scroll loader (`load.infinite-scroll`)

- **One line** — more content fetches as you approach the end of what is loaded.
- **What the reader sees** — Scroll toward the bottom of a feed and, before you reach it, a
  spinner appears briefly below the last item and new items fade in beneath it. If the fetch
  starts early enough you never see the spinner at all — the list simply never ends. Nothing
  above you moves, so your reading position is undisturbed, which is what separates this from
  the versions that jolt you as content arrives.
- **Mechanism** — an IntersectionObserver sentinel positioned above the true end triggers the
  fetch; new items append with a short fade.
- **Stack** — IntersectionObserver plus append; virtualisation once the list grows large.
- **Params** — trigger distance (1–2 viewports before the end); item fade (150–250ms);
  batch size.
- **Use when** — feeds and browsing surfaces where reaching an end is not meaningful.
- **Don't use when** — there is a footer people need, or the content is a search result set they
  may want to leave and return to. Infinite scroll destroys both.
- **Reduced motion** — items appear without fading.
- **Performance** — trigger early enough that the fetch completes before the user arrives — the
  best loading animation here is one nobody sees. Virtualise past a few hundred items.
- **Gotchas** — appending must never shift what is currently on screen. Provide a keyboard-
  reachable "load more" alternative and announce arrivals politely; an infinitely growing list
  with no announcement is disorienting for screen reader users. Preserve scroll position on
  back-navigation, or returning to a feed restarts it.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API

---

### Stalled escalation (`load.stall-escalate`)

- **One line** — a long wait changes what it says the longer it goes on.
- **What the reader sees** — A spinner for the first few seconds. Then a line of text: "Still
  working…". At fifteen seconds it becomes specific — "This is taking longer than usual" — and
  offers a cancel control. At thirty it suggests trying again later. Nothing about the animation
  changes much; what changes is that the interface keeps acknowledging that you are still
  waiting, which is the difference between a slow system and one that appears to have forgotten
  you.
- **Mechanism** — timed state transitions on the loading region, each crossfading in.
- **Stack** — timers plus a status region; no animation library required.
- **Params** — first escalation (5–8s); second (15–20s); cancel offered by the second at the
  latest.
- **Use when** — operations that can genuinely run long: large uploads, complex generation,
  cold starts.
- **Don't use when** — the operation is reliably fast; the escalation machinery implies a
  fragility that may not exist.
- **Reduced motion** — text changes without crossfading.
- **Performance** — free.
- **Gotchas** — announce each escalation politely so it reaches non-visual users. Never claim
  progress you cannot see — "almost done" when you have no idea is the fastest way to lose
  trust. Always give a way out by the second escalation.
- **References** — https://www.groovyweb.co/blog/ui-mistakes-ai-apps-2026

---

### Progressive hydration (`load.progressive-hydrate`)

- **One line** — static content becomes interactive in priority order.
- **What the reader sees** — Ideally nothing. The page is readable immediately, and controls
  become responsive over the following moment — the ones near the top and the ones you are most
  likely to press first. Done well, the only symptom is that a button pressed extremely early
  registers slightly late. Done badly, the page looks complete but does nothing for two seconds,
  which is the most frustrating state a web page can be in.
- **Mechanism** — deferred or islands-based hydration, prioritised by viewport position and
  interaction likelihood.
- **Stack** — a framework's partial hydration; the visible layer is a disabled or busy state on
  not-yet-interactive controls.
- **Params** — priority order; whether pre-hydration controls appear disabled (usually not —
  they should look normal and queue the interaction).
- **Use when** — content-heavy pages with interactive components.
- **Don't use when** — the page is an application whose primary value is interaction.
- **Reduced motion** — unaffected; there is no motion here.
- **Performance** — one of the highest-value techniques for INP, because a page hydrating
  everything at once blocks the main thread precisely when the user first tries to interact.
- **Gotchas** — capture and replay early interactions rather than dropping them, or the first
  click on a slow device is silently lost. Never make content *appear* interactive before it is
  without queuing the input — that is worse than a visible loading state.
- **References** — https://www.digitalapplied.com/blog/ai-model-latency-benchmarks-2026-ttft-throughput

---

### Upload progress (`load.upload`)

- **One line** — a file being uploaded reports its own progress in place.
- **What the reader sees** — Drop a file and it appears immediately in the list as a row with a
  thumbnail, a name, and a progress bar filling along the bottom edge of the row. Several files
  upload at once, each reporting independently. When one finishes its bar disappears and a tick
  replaces it. Failures stay in the list, red, with a retry — never silently removed.
- **Mechanism** — per-item determinate progress driven by real upload events, with per-item
  terminal states.
- **Stack** — upload progress events plus the determinate bar pattern.
- **Params** — bar update throttle (~4/s); success hold before the bar disappears (~1s); failure
  persistence (indefinite, until dismissed).
- **Use when** — any multi-file upload.
- **Don't use when** — files are tiny and instantaneous; a bar that appears and vanishes reads
  as a glitch.
- **Reduced motion** — bars update without transitions.
- **Performance** — throttle updates; progress events can fire far more often than is useful.
- **Gotchas** — upload progress reaching 100% means the bytes were sent, not that the server
  accepted them — show a processing state between "uploaded" and "done" or people close the tab
  too early. Each row needs an accessible name and its own progress semantics.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/progress_event

---

### Refresh pulse (`load.refresh-pulse`)

- **One line** — already-visible data indicates quietly that it is being updated.
- **What the reader sees** — A dashboard with numbers on it. Every thirty seconds the figures
  dim very slightly for a moment and come back, occasionally with different values. Nothing is
  removed, nothing is replaced by a spinner, and the layout never changes — you can keep reading
  a number through a refresh. It says "this is live" without ever taking the data away to prove
  it.
- **Mechanism** — a brief opacity dip on the refreshing region, with values transitioning in
  place (see `micro.count-change`).
- **Stack** — CSS transitions plus a polling or subscription layer.
- **Params** — dip depth (0.6–0.8 opacity); dip duration (200–300ms total); only show it if the
  refresh exceeds ~300ms.
- **Use when** — dashboards, live metrics, anything polling on an interval.
- **Don't use when** — refreshes are frequent. A dashboard pulsing every two seconds is
  exhausting; update silently instead.
- **Reduced motion** — values update with no dip.
- **Performance** — negligible; the polling is the cost, not the animation.
- **Gotchas** — never replace loaded data with a skeleton on refresh — that is a downgrade the
  user experiences as the data disappearing. Announce only meaningful changes, and rate-limit
  those; a live region firing every poll is intolerable.
- **References** — https://www.groovyweb.co/blog/ui-ux-design-trends-ai-apps-2026

### Title card (`load.title-card`)

- **One line** — the loading screen is a typeset line on the page's own grid, not a progress
  report, and it hands the page in rather than getting out of the way.
- **What the reader sees** — The page opens on a solid field with almost nothing on it: three
  short runs of small type on one line across the middle — one at the left margin, a mark at
  the centre, one at the right margin — sitting exactly where the page's own gutters are. The
  two outer runs are noise, every character churning through symbols several times a second.
  They settle left to right, over about a second, into two readable lines: what the company
  does, and where it is. The field holds a beat, then leaves. Underneath, the page is already
  moving — the sentence you just watched resolve is climbing into place at headline size, on
  the same left edge it just occupied. It reads less like waiting than like a title card.
- **Mechanism** — three composed parts on one timeline. A per-character text swap on a moving
  resolved/unresolved boundary (`driver:time`); a hold; then a `transform` on the covering
  field leaves — as one panel, as a fade, or as two halves seamed on the type line; the shape
  is a choice and none of the sighted implementations exposed theirs. The last part starts the
  first screen's own timeline at a **negative offset** relative to the field's exit, so the two
  overlap. No layout property is animated.
- **Stack** — free hand-rolled: the resolve is ~30 lines of `requestAnimationFrame`, the exit
  is a CSS transition, the handoff is one offset. GSAP ScrambleText + a timeline buys you
  `revealDelay`, `rightToLeft` and word-granularity `delimiter` handling, and labelled
  timeline positions for the overlap; it is a convenience, not a requirement.
- **Params** — resolve duration (0.8–1.2s; under 0.5s nobody reads it, over 1.5s it is a
  toll); hold after resolve (0.8–1.4s; 0 is legitimate and means "leave on resolve");
  exit duration (0.7–1s); **handoff offset** (−0.4s to −0.8s — the knob that decides whether
  this is one move or two, and the only one worth being fussy about); character set (symbols,
  hex, the target's own alphabet — this choice *is* the register).
- **Use when** — the opening line is worth reading and you want it read before the layout
  competes with it; an agency or portfolio site whose positioning statement *is* the content;
  there is a genuine wait — a font set, a hero video, a first WebGL frame — to spend.
- **Don't use when** — there is nothing to load. A card over an already-rendered page is a
  toll booth you built yourself. Also: any page where the visitor arrives repeatedly, unless
  it is gated to once per session; and anywhere the first screen must be found by search — the
  card holds the largest contentful paint hostage for its whole duration.
- **Reduced motion** — the card is never inserted and the page is simply already there, with
  the first screen at its final state and the two lines set, unscrambled. Not "the card
  without animation" — a still card is a blank screen over the page with no way past it. If
  the wait is real, replace the theatre with a plain, announced busy state rather than
  nothing.
- **Performance** — two moving parts with different costs. The field's exit is compositor-only
  — `transform` on two panels. The resolve is **not**: replacing `textContent` every frame
  invalidates layout on that element, which is affordable at label length and is the reason
  this entry says never to scramble a headline.
  The cost that matters is neither, it is **LCP** — the field is opaque and
  full-viewport, so the largest contentful paint cannot happen until it clears, and its whole
  duration lands on that metric. Budget the card inside the LCP target, not alongside it.
- **Gotchas** — **never gate the exit on a promise that can fail to settle.** Race
  `document.fonts.ready` against a hard ceiling; it resolves only once layout is complete and
  no further font loads are needed, which is a condition a slow third-party font can defer
  indefinitely, and the failure mode is a permanent black screen. **Gate the intro explicitly**
  — the first screen's timeline must be started by the card, not by `DOMContentLoaded`, or it
  plays behind the field and nobody sees it (`cross-cutting.md`, "Gate the intro"). **The
  churning runs must not reach assistive tech**: mark the card `aria-hidden` and `inert` and
  put a real busy state on the document, or a screen reader announces a paragraph of symbols,
  repeatedly. **Churn the spaces and the line changes width**, which reads as jitter — leave
  spaces settled. And decide the second-visit question deliberately: once per session is the
  common answer and it is a decision, not a default.
- **References** — https://gsap.com/docs/v3/Plugins/ScrambleTextPlugin/ ·
  https://developer.mozilla.org/en-US/docs/Web/API/FontFaceSet/ready ·
  https://revelatio.studio · https://noth.in
- **Tags** — `use:loading` `use:hero` `mood:cinematic` `mood:technical` `industry:agency`
  `industry:portfolio` `driver:time` `prim:transform` `effect:waiting` `effect:quality`
  `cost:free` `a11y:needs-fallback` `a11y:screenreader-risk` `rung:maximal`
  `stack:css` `stack:js-vanilla` `stack:gsap` `era:evergreen` `device:touch-safe`
  `device:reduced-motion-critical`
- **Pairs with** — `entrance.hero-sequence`, `entrance.mask-rise`, `text.scramble`
- **Conflicts with** — `entrance.curtain` (two owners of the opening moment), `route.curtain`
  (the same, on the first navigation), `load.determinate` (two loading states on one screen)

---

## Family notes

**Streaming beats spinning.** Progressive delivery converts a wait into reading time and makes
identical total durations feel dramatically faster. Where content can arrive in pieces, it
should — and time-to-first-anything is the number that governs the felt experience.

**Delay the indicator, then hold it.** ~250ms before showing, then a minimum visible period.
Both halves; each without the other produces a flicker.

**Never show a static spinner for more than a second on a long operation.** Escalate to
something that reports what is happening, and offer a way out.

**Reserve the space first.** Skeletons, staged reveals and appended items are all worse than
useless if content shifts when the real thing lands.

**Never take away data you already have.** Refreshes dim; they do not revert to skeletons.

**Loading states are silent.** `aria-busy`, live regions, real `<progress>` semantics. A
screen reader user gets nothing at all from a shimmer, a dot animation or a filling bar.
