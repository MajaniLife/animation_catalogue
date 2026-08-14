# Ambient & decorative

Continuous motion with no state to report. Marquees, gradient fields, floating shapes, grain,
looping backgrounds — motion that is not triggered, does not resolve, and communicates
nothing except that the page is alive.

Every other family answers a question: what arrived, what changed, what did you press.
Ambient motion answers none, and its justification is atmospheric rather than functional: a
page with a slow drift in the background reads as considered rather than printed. That is a
real effect and a legitimate goal. It is also the family that most reliably damages
performance and accessibility, because unlike everything else here, **it never stops.**

**The legal and ethical floor, stated once.** WCAG 2.2.2 (Level A) requires that moving,
blinking or scrolling content which starts automatically, lasts **more than five seconds**,
and is presented alongside other content, must have a mechanism to pause, stop or hide it —
and once stopped, it must not restart on its own. Auto-updating content has the same
requirement. The exceptions are narrow: motion during a loading phase (the user cannot
interact anyway) and motion that is essential to the activity, such as a timer in a test or
live auction prices. Almost nothing in this file qualifies for those exceptions, so almost
everything here needs a control.

**Three costs to weigh before adding any of it.** A permanent animation prevents the browser
from idling, which on a laptop means the fan and on a phone means battery. It competes for
frames with everything the user actually came for. And it is by definition motion nobody
asked for, which is what `prefers-reduced-motion` exists to switch off — so every entry here
must have a genuinely static form that still looks intentional.

---

### Marquee band (`ambient.marquee`)

- **One line** — a strip of text or logos scrolls sideways, forever.
- **What the reader sees** — A band across the page — client names, services, a repeated
  phrase — sliding steadily in one direction, the content wrapping seamlessly so there is no
  visible start or end. At large type it functions as texture rather than as reading matter;
  you take in the rhythm, and individual words only when one happens to pass your eye. It is
  the most recognisable device in contemporary editorial web design, and it went from
  distinctive to ubiquitous inside about three years.
- **Mechanism** — the track's contents duplicated once, translated by exactly half the total
  width, and wrapped so the loop is seamless.
- **Stack** — CSS keyframes for the simple case; a helper that measures and duplicates for the
  robust one.
- **Params** — speed (60–120px/s; faster becomes unreadable and more agitating); direction;
  gap and separator.
- **Use when** — a band between sections, a client wall, a keyword strip.
- **Don't use when** — the words must be read individually, or there is already other motion
  on screen.
- **Reduced motion** — paused, showing a static row of the content.
- **Performance** — a permanent transform. Cheap per frame, but it never stops — pause it when
  off-screen.
- **Gotchas** — **it needs a pause control**; this is the textbook WCAG 2.2.2 case, and once
  paused it must stay paused. The duplication must be exact or the loop seams visibly at the
  wrap point. Duplicated content should be `aria-hidden` so screen readers do not read the list
  twice.
- **References** — https://www.wcag.com/designers/2-2-2-pause-stop-hide/ ·
  https://www.digitala11y.com/understanding-sc-2-2-2-pause-stop-hide/

---

### Gradient drift (`ambient.gradient-drift`)

- **One line** — a large soft gradient shifts slowly behind the content.
- **What the reader sees** — The background is not flat: two or three broad colour fields
  overlap and move against each other so slowly that at any moment it appears static, yet
  looking away and back the whole thing has changed. Nothing has edges, nothing repeats
  visibly, and the page acquires a sense of depth and atmosphere without any element being
  identifiably responsible. It is the most common "premium" background treatment of the last
  few years and, done gently, the least intrusive thing in this file.
- **Mechanism** — animated radial gradients on a background, or a shader for the smooth
  version; alternatively large blurred shapes drifting behind a frosted layer.
- **Stack** — CSS gradients and transforms; WebGL only if you need genuine fluid motion.
- **Params** — period (20–60s per cycle — this must be slow enough that no motion is
  detectable in a glance); colour contrast between fields (low); blur radius.
- **Use when** — hero backgrounds, dark-theme surfaces, section backdrops behind text.
- **Don't use when** — text sits on it and the contrast varies across the animation.
- **Reduced motion** — a static gradient at a representative frame.
- **Performance** — animating gradient positions repaints a large area continuously; the
  blurred-shapes-behind-a-blur variant is the expensive one. Prefer transforming a few large
  blurred elements over animating gradient stops.
- **Gotchas** — the text contrast must pass at **every frame**, not just the one you designed
  against. Very slow gradient animation is where colour banding shows on 8-bit displays; add a
  subtle grain to hide it.
- **References** — https://en.gehirngerecht.digital/wcag-criterion/2-2-2-pause-stop-hide/

---

### Floating shapes (`ambient.float`)

- **One line** — decorative objects bob gently and independently.
- **What the reader sees** — Shapes — circles, product cutouts, icons — drift slowly up and
  down on slightly different cycles, so they never align into an obvious pattern. Each moves a
  handful of pixels over several seconds. The effect is of things suspended in something,
  weightless and lightly buoyant, and it makes an otherwise static composition feel like a
  space rather than a diagram.
- **Mechanism** — per-element sine-based `translateY` loops with different periods and phase
  offsets.
- **Stack** — CSS keyframes with a per-element delay and duration; no library needed.
- **Params** — amplitude (4–12px); period (3–6s, deliberately different per element so they
  desynchronise); phase offset.
- **Use when** — illustrated hero compositions, empty states, playful brand pages.
- **Don't use when** — the shapes are near text. Movement in peripheral vision while reading is
  genuinely distracting, and for some readers disabling.
- **Reduced motion** — everything static at its base position.
- **Performance** — one transform per shape, permanently. Keep the count in single figures and
  pause off-screen.
- **Gotchas** — matching periods make the whole group pulse in unison, which reads as a glitch;
  vary them deliberately. Floating elements that overlap interactive targets can intercept
  clicks — set `pointer-events: none`.
- **References** — https://www.boia.org/wcag2/cp/2.2.2

---

### Grain overlay (`ambient.grain`)

- **One line** — a fine noise texture sits over everything, shifting each frame.
- **What the reader sees** — The page has a subtle tooth to it, like film or coated paper, and
  the grain is not static — it flickers minutely, which is precisely what makes it read as
  film rather than as a texture image. At the right opacity you do not see grain at all; you
  see a page that looks slightly analogue and less like flat colour on a screen. It also
  usefully disguises gradient banding.
- **Mechanism** — a tiled noise image or SVG turbulence over the whole viewport, its position
  jittered every few frames.
- **Stack** — a small tiled PNG is the cheap and correct answer; SVG `feTurbulence` is the
  expensive one.
- **Params** — opacity (0.02–0.06); tile size; jitter rate (8–12 fps is enough — a full 60 is
  wasted work).
- **Use when** — dark editorial themes, brand sites with a film or print register.
- **Don't use when** — over text at small sizes; grain reduces effective contrast and makes
  type harder to read.
- **Reduced motion** — static grain, or none. The flicker is the motion, and it is the part
  that some users find unpleasant.
- **Performance** — a full-viewport overlay repainting continuously is more expensive than it
  looks. Use a tiled raster, animate the background position in steps rather than continuously,
  and never use an animated SVG filter for this at full-screen size.
- **Gotchas** — `pointer-events: none` or it swallows every click on the page. Very fine
  high-contrast grain can shimmer unpleasantly on some displays and is a flicker consideration
  — keep the opacity low.
- **References** — https://accessicart.com/wcag-level-a-sc-2-2-2-pause-stop-hide/

---

### Breathing pulse (`ambient.breathe`)

- **One line** — an element scales gently in and out, like breathing.
- **What the reader sees** — A button, a badge or a dot expands and contracts by a couple of
  percent on a slow three-second cycle. It is barely perceptible when you look straight at it
  and quite noticeable in peripheral vision, which is the point: it draws the eye without
  demanding anything. On a record button or a live indicator it reads as *active*; on a call
  to action it reads as an invitation.
- **Mechanism** — a looping `scale` between 1 and about 1.03, ease-in-out.
- **Stack** — CSS keyframes.
- **Params** — amplitude (1.5–4%); period (2.5–4s — faster reads as urgent and, past a point,
  as anxious).
- **Use when** — live indicators, one primary action, a recording state.
- **Don't use when** — more than one element on screen does it. Two competing pulses is a
  waiting room.
- **Reduced motion** — static. If the pulse indicates a live state, that state needs a static
  representation too — a colour and a label.
- **Performance** — one transform, permanent. Negligible individually; pause off-screen.
- **Gotchas** — a pulse that means "live" or "recording" is carrying information, so it needs a
  text equivalent. Anything that pulses faster than about three times a second is a flicker
  hazard.
- **References** — https://www.wcag.com/designers/2-2-2-pause-stop-hide/

---

### Aurora / mesh field (`ambient.aurora`)

- **One line** — coloured light appears to move through a soft field.
- **What the reader sees** — Ribbons of colour drift and fold across a dark background,
  brightening where they overlap, like an aurora or ink in water. Nothing has a defined edge
  and nothing repeats; the movement is slow and organic and never resolves into a pattern you
  can predict. It is the most atmospheric background treatment available and the most
  expensive one in this file to do properly.
- **Mechanism** — a fragment shader over noise; the cheaper CSS approximation uses several
  large, heavily blurred, differently coloured shapes drifting behind a blur.
- **Stack** — WebGL/OGL for the real version; CSS blur for the approximation, which is often
  indistinguishable at low contrast.
- **Params** — noise scale and speed; colour palette (2–3 colours; more becomes muddy); blur
  radius in the CSS version.
- **Use when** — a hero backdrop on a site where atmosphere is the brief.
- **Don't use when** — you would be loading a WebGL renderer solely for this, or the audience
  is mobile-heavy.
- **Reduced motion** — a static frame — and a still aurora is genuinely beautiful, so this
  branch costs nothing visually.
- **Performance** — the shader version is fill-rate bound and scales with screen area, which
  makes it worst on high-DPR phones. The CSS version's large blurs are also expensive; both
  need a device-pixel-ratio cap or a size limit.
- **Gotchas** — the same contrast-at-every-frame rule as gradient drift. If it is the only
  reason WebGL is on the page, weigh a static export of the same shader instead — often nobody
  can tell.
- **References** — https://en.gehirngerecht.digital/wcag-criterion/2-2-2-pause-stop-hide/

---

### Auto carousel (`ambient.auto-carousel`)

- **One line** — slides advance on a timer without being asked.
- **What the reader sees** — A hero panel showing one message, which after five seconds slides
  aside for the next, then the next, cycling indefinitely. Dots below show the position. It is
  included here not as a recommendation but because it is everywhere, and because it is the
  clearest example in this catalogue of motion that serves the organisation rather than the
  reader — it exists because several teams wanted the hero slot.
- **Mechanism** — a timed slide or fade between panels, usually with pause-on-hover.
- **Stack** — a carousel library, or scroll-snap plus a timer.
- **Params** — interval (6–8s minimum — the common 3–4s is well below reading speed for many
  people); transition (0.4–0.6s); pause on hover, focus and interaction.
- **Use when** — genuinely rarely. If the content matters, put it on the page.
- **Don't use when** — the panels contain content people need, or there are more than three.
- **Reduced motion** — do not auto-advance at all; show the first panel with manual controls.
- **Performance** — negligible; the cost here is attention, not frames.
- **Gotchas** — auto-advancing content is squarely WCAG 2.2.2: it needs a pause control, and
  once stopped it must not restart. It must pause on hover *and* on focus, or a keyboard user
  loses the panel they are reading. Slides moving out from under a reader mid-sentence is the
  single most common complaint about this pattern, and the reason many teams have removed it.
- **References** — https://testparty.ai/blog/wcag-2-2-2-pause-stop-hide-2025-guide ·
  https://www.wcag.com/designers/2-2-2-pause-stop-hide/

---

### Cursor trail (`ambient.cursor-trail`)

- **One line** — something follows the pointer and lingers behind it.
- **What the reader sees** — A trail of fading shapes, a smear of colour, or a chain of dots
  follows the cursor around the page, catching up when you stop. It is playful and immediately
  noticeable, and it is decoration with no informational content whatsoever — the page is
  drawing on itself as you move. It works on a personal site or an experiment and is very hard
  to justify anywhere with a task in it.
- **Mechanism** — a pool of elements or a canvas layer, positioned along a lagged history of
  pointer positions.
- **Stack** — canvas for anything dense; a handful of DOM elements otherwise.
- **Params** — trail length; fade duration; lag between segments.
- **Use when** — personal sites, experiments, 404 pages.
- **Don't use when** — anywhere with a task. It competes with the interface for attention and
  can obscure what is under the cursor.
- **Reduced motion** — remove entirely.
- **Performance** — runs on every pointer event, all the time. Canvas past a few segments;
  cap the pool rather than creating elements per move.
- **Gotchas** — `pointer-events: none` throughout, and remove it on touch where there is no
  pointer. It obscures content under the cursor, which is a problem for screen-magnifier users
  specifically — the same objection as custom cursors.
- **References** — https://www.boia.org/wcag2/cp/2.2.2

---

### Looping background video (`ambient.video-loop`)

- **One line** — a muted video plays behind the content, forever.
- **What the reader sees** — Footage — a workshop, a coastline, an abstract texture — playing
  silently behind the hero text, looping every ten or twenty seconds. It carries atmosphere no
  still image can, and it is also the single heaviest element on most pages that use it: a
  multi-megabyte download and a continuously decoding video, competing with everything else
  for bandwidth and battery.
- **Mechanism** — an autoplaying, muted, looping, inline video with a poster image.
- **Stack** — the platform. The work is in the encoding, not the code.
- **Params** — duration (8–15s); resolution (cap at 1080p or below — it sits behind text);
  bitrate; poster frame.
- **Use when** — the footage genuinely carries the brand and the budget allows.
- **Don't use when** — a still frame would say the same thing, which is more often than
  anyone admits.
- **Reduced motion** — show the poster frame instead of playing. This is explicitly what the
  media query is for, and it is trivial to implement.
- **Performance** — the heaviest ambient option. Continuous decode drains battery, and it
  competes with LCP. Serve modern codecs, cap resolution, and never autoplay on a metered
  connection (`prefers-reduced-data`).
- **Gotchas** — must be muted to autoplay at all, and must be `playsinline` or iOS opens it
  fullscreen. Text over video needs a scrim to hold contrast across every frame. Provide a
  pause control: it is moving content lasting more than five seconds, and the loading exception
  does not apply.
- **References** — https://aaardvarkaccessibility.com/wcag-plain-english/2-2-2-pause-stop-hide/

---

### Idle animation (`ambient.idle-nudge`)

- **One line** — after a period of inactivity, something moves to re-engage.
- **What the reader sees** — Nothing happens for thirty seconds, and then the primary button
  gives a small bounce, or an illustration waves, or a hint slides in. It stops as soon as you
  move. It is designed to catch the eye of someone who has stopped, and it is the only entry
  in this family that is *conditional* rather than continuous — which makes it much easier to
  justify.
- **Mechanism** — an inactivity timer reset by any input; on fire, play a brief animation
  once, not a loop.
- **Stack** — a timer plus any short animation.
- **Params** — idle threshold (20–45s); animation length (under 1s); maximum repetitions (once
  or twice — never indefinitely).
- **Use when** — onboarding, a stuck step in a flow, a kiosk display.
- **Don't use when** — the user may be reading. Being nudged while concentrating is
  patronising, and "idle" and "reading carefully" look identical to a timer.
- **Reduced motion** — no nudge, or a static highlight.
- **Performance** — free; nothing runs until it fires.
- **Gotchas** — cap the repetitions. A nudge that repeats every thirty seconds forever is
  hostile, and once it has fired twice it has said everything it can say.
- **References** — https://www.digitala11y.com/understanding-sc-2-2-2-pause-stop-hide/

---

## Family notes

**Five seconds is the line.** Moving content that starts automatically, runs longer than five
seconds and sits alongside other content needs a pause, stop or hide mechanism — and must not
restart itself once stopped. The exceptions (loading indicators, genuinely essential motion
like timers and live auction prices) cover very little of this file.

**Pause when off-screen.** One IntersectionObserver that stops ambient loops the user cannot
see is worth more than every micro-optimisation in this catalogue combined. Without it, a
marquee at the top of the page is still consuming frames when the reader is at the footer.

**One ambient effect per page.** Grain plus a gradient drift plus floating shapes plus a
marquee is not atmosphere, it is noise — and four permanent animations is four permanent
costs.

**The static version has to look intentional.** Everything here disappears under
`prefers-reduced-motion`, so design the still frame deliberately rather than accepting
whatever the animation happens to leave behind.

**Ambient motion is the first thing to cut.** When the performance budget is exceeded, this
family is where you look first — it is, by definition, the motion carrying no information.
