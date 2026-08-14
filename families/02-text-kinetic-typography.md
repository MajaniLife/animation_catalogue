# Text & kinetic typography

Motion applied to letterforms — characters, words, lines — rather than to the boxes that
contain them. This is the signature register of contemporary editorial and agency work,
and the family with the worst accessibility record on the web.

Two things separate it from every other family.

**The splitting problem.** Almost everything here requires breaking a string into
per-character or per-word elements, which means rewriting the DOM of your own copy. That
has three consequences most implementations discover too late: the split must happen
**after fonts have loaded**, or lines are measured against the fallback and break in the
wrong places; the split must be **redone on resize** and its animations rebuilt with it,
or a rewrap orphans the tween and leaves text parked off-screen; and the split **destroys
the text for assistive technology** unless explicitly repaired.

**The accessibility contract.** This is not optional and not subtle. Splitting a word into
character spans causes some screen readers to announce it letter by letter — the text is
literally spelled out. The repair is a two-part pattern: put `aria-label` carrying the
original unsplit string on the container, and `aria-hidden="true"` on every split
fragment inside it. GSAP's SplitText does exactly this with `aria: "auto"`, which is the
default and should never be turned off casually. **The pattern has a real limit**: it
flattens nested semantics, so links, emphasis and headings *inside* split text are lost to
screen readers. If the text contains a link, either don't split it, or split with
`aria: "hidden"` and provide a visually-hidden duplicate of the real markup alongside.

The other rule worth stating once: **animate headlines, never body copy.** A paragraph that
moves while someone is trying to read it is not a design decision, it is an obstruction.
Everything below assumes display type unless it says otherwise.

---

### Character cascade (`text.char-cascade`)

- **One line** — letters arrive one after another in a rapid, directional wave.
- **What the reader sees** — The headline is absent, then its letters appear in quick
  succession — twenty milliseconds apart, so fast that you perceive a sweep rather than
  individual arrivals — each rising or fading into place. Left to right it reads as
  writing; from the centre outward it reads as an unfolding; from the edges inward it
  reads as convergence and feels distinctly cinematic; randomised it feels playful and
  slightly unstable. The whole word lands in well under a second. At display size on a
  short phrase this is the most energetic entrance available to type, and the one that
  most clearly signals that someone made a decision about this page.
- **Mechanism** — split to characters, animate `translateY` or `opacity` per character with
  a small stagger whose origin is configurable (start, centre, edges, random, end).
- **Stack** — GSAP SplitText (free since 2025, rewritten in 3.13); Motion's `splitText`;
  Splitting.js plus any animator.
- **Params** — stagger per char (0.015–0.03s; above 0.05s a long word feels typed rather
  than swept); origin; distance (small — characters travelling far look like confetti).
- **Use when** — one hero headline, one or two words, once per page.
- **Don't use when** — the string is longer than about five words. Per-character work
  scales terribly: a sentence becomes a queue.
- **Reduced motion** — do not split at all; render the text normally at full opacity.
- **Performance** — a 40-character headline is 40 animated elements plus 40 DOM nodes. Fine
  once, expensive if applied to every heading on a long page.
- **Gotchas** — split after `document.fonts.ready`. Rebuild on resize. Keep `aria: "auto"`.
  Never apply `text-wrap: balance` to a split target — it interferes with the split.
  Characters inherit `font-kerning` oddly once wrapped; disabling kerning on the split
  target avoids a visible reflow at the moment of splitting.
- **References** — https://gsap.com/docs/v3/Plugins/SplitText/ ·
  https://css-irl.info/how-to-accessibly-split-text/

---

### Word stagger (`text.word-stagger`)

- **One line** — the same idea at word granularity, and the version you can actually ship.
- **What the reader sees** — Words arrive in sequence, each a twentieth of a second behind
  the last, lifting slightly as they fade in. Because the unit is a word rather than a
  letter, the phrase stays readable throughout — you can begin reading before the animation
  finishes, which is the entire practical difference. On a two-line statement it reads as
  measured speech, as if the sentence is being spoken rather than displayed. It is the
  compromise position between the drama of per-character work and the restraint of a line
  reveal, and it is usually the right one.
- **Mechanism** — split to words, per-word stagger on `opacity` and a short `translateY`.
- **Stack** — same as char-cascade; the split type is the only change.
- **Params** — stagger (0.04–0.06s); distance (12–20px); duration per word (0.4–0.6s,
  overlapping).
- **Use when** — statements, pull quotes, subheadings — anything two lines or longer where
  per-character would be intolerable.
- **Don't use when** — body copy. Still body copy.
- **Reduced motion** — unsplit, full opacity.
- **Performance** — an order of magnitude fewer nodes than character splitting. This is the
  one to reach for on a page with several animated headings.
- **Gotchas** — word splitting must preserve the space between words or the phrase reflows
  as a single run-on string mid-animation. Same fonts-ready and resize rules apply.
- **References** — https://motion.dev/docs/split-text

---

### Scramble (`text.scramble`)

- **One line** — characters churn through random glyphs before settling on the real string.
- **What the reader sees** — Where the word should be there is noise: a run of shifting
  characters, changing several times a second, each position cycling through symbols or
  digits. Left to right the noise resolves — the first character locks to its true letter,
  then the next, then the next — until the real word stands where the churn was. It takes
  under a second and it is unmistakably *computational*: terminals, decryption, systems
  coming online. Used on a small label it is a wink. Used on a hero headline it makes the
  first impression of your page a paragraph of gibberish, which is a bolder bet than most
  people realise they are making.
- **Mechanism** — per-character text replacement on a timer, with a moving "resolved"
  boundary; the character set is configurable.
- **Stack** — GSAP ScrambleText (free since 2025); hand-rolled in ~30 lines for a single
  string.
- **Params** — character set (`01`, hex, katakana, the target's own alphabet — this choice
  *is* the aesthetic); duration (0.6–1.2s); reveal delay per character.
- **Use when** — micro-labels, counters, status text, a technical brand register.
- **Don't use when** — the text must be readable on arrival, or the audience is not
  primed for the reference. It is a strong genre signal.
- **Reduced motion** — final string immediately, no churn. This one is non-negotiable:
  rapidly changing text is a legibility and distraction hazard.
- **Performance** — cheap per frame, but it writes to the DOM continuously for its
  duration. Do not run several at once.
- **Gotchas** — scrambling changes text width every frame unless the character set is
  monospaced or the container is fixed-width; otherwise surrounding layout jitters. Screen
  readers may announce the intermediate garbage — mark it `aria-hidden` and expose the
  final string.
- **References** — https://webflow.com/blog/gsap-becomes-free

---

### Typewriter (`text.typewriter`)

- **One line** — text types itself out, character by character, usually with a cursor.
- **What the reader sees** — A blinking block or bar sits alone, then letters appear one at
  a time at a human-ish typing rate, the cursor advancing ahead of them. Sometimes it
  pauses at the end of a phrase, deletes back a few words, and types a different ending —
  the "we build X / Y / Z" pattern. It is the oldest text animation on the web and it
  reads exactly as old as it is: nostalgic at best, filler at worst. Its one genuine
  strength is pacing — it forces the reader to receive the sentence at the author's speed,
  which is occasionally what you want.
- **Mechanism** — progressive substring, or a `steps()` animation on a clipping width for
  the CSS-only monospace variant.
- **Stack** — CSS `steps()` for the single-line monospace case; a few lines of JavaScript
  otherwise.
- **Params** — characters per second (20–40; below 15 is agonising); cursor blink rate
  (~1s); pause at phrase end (1–2s for cycling variants).
- **Use when** — a terminal or code context where the metaphor is literal.
- **Don't use when** — you want to seem current. It is the most dated effect in this file.
- **Reduced motion** — full string immediately, static cursor or none.
- **Performance** — trivial. The DOM writes are the only cost.
- **Gotchas** — the CSS `steps()` version only works in a monospace font and only for a
  single unwrapped line; everything else needs JavaScript. Cycling variants must reserve
  the width of the longest phrase or the layout jumps on every cycle. Announce the final
  text once, not on every keystroke.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/easing-function/steps

---

### Word cycle (`text.word-cycle`)

- **One line** — one word in a sentence swaps repeatedly for alternatives.
- **What the reader sees** — A fixed sentence — "design systems for **studios**" — where
  the last word lifts away and is replaced from below by another: **founders**, then
  **teams**, then back. The rest of the line never moves. Each swap takes a fraction of a
  second and the new word holds for two or three seconds before the next. It is a compact
  way of saying "and also, and also" without a list, and because the surrounding sentence
  is stationary the eye stays exactly where the information changes. The failure is
  jitter: if the words differ in width and the container is not reserved, the whole line
  twitches on every swap.
- **Mechanism** — clipped container; outgoing word translates out while the incoming
  translates in from the opposite edge; loops on a timer.
- **Stack** — CSS keyframes for a fixed list; any library for dynamic ones.
- **Params** — hold duration (2–3s); transition (0.3–0.4s); container width (fixed to the
  longest entry, or animated between measured widths).
- **Use when** — a hero line that must cover several audiences or services.
- **Don't use when** — the alternatives are not genuinely parallel. Cycling three unrelated
  words reads as indecision.
- **Reduced motion** — show one word, static. Or list them separated by slashes — the
  information survives, the motion does not.
- **Performance** — a permanent loop. Small, but it never stops; pause it when off-screen.
- **Gotchas** — auto-playing text that changes every few seconds is a WCAG concern for
  users who read slowly; three seconds minimum, and consider stopping after a full cycle
  rather than looping forever.
- **References** — https://itamde.com/en/kinetic-typography-is-the-web-design-trend-you-cant-ignore/

---

### Scrub highlight (`text.scrub-highlight`)

- **One line** — words brighten from muted to full as the paragraph scrolls through view.
- **What the reader sees** — A block of text sits dimmed, present but recessive. As you
  scroll, its words light to full contrast one after another, the boundary between lit and
  unlit travelling down the paragraph in step with your scrolling. Stop scrolling and the
  effect stops with you; scroll back and it recedes. Because it is tied to scroll position
  rather than a clock, the text illuminates **at your own reading pace**, which produces
  the uncanny impression that the page is following along. It is the most effective way to
  make a long statement feel read rather than displayed, and one of very few text effects
  that genuinely aids attention instead of competing with it.
- **Mechanism** — split to words, animate `color` from muted to full with a stagger, the
  whole tween scrubbed against a scroll range rather than played on a duration.
- **Stack** — GSAP SplitText + ScrollTrigger with `scrub`. CSS `view-timeline` can drive a
  simpler two-tone version where support allows.
- **Params** — dim value (a real muted token, not opacity — see gotchas); scroll range
  (start "top 85%", end "centre 45%"); stagger (small; the range does the pacing).
- **Use when** — a manifesto paragraph, an about statement, a single long quote.
- **Don't use when** — the text must be readable before it is reached. Dimmed text is
  harder to read, and that is a real cost for anyone with low vision.
- **Reduced motion** — all text at full contrast, no dimming.
- **Performance** — colour is not compositor-only, but it is cheap; the real cost is that
  it recalculates on every scroll frame across many word elements. Keep the paragraph
  short.
- **Gotchas** — the dim state must still meet contrast requirements, because for a slow
  reader it is the state they read in. Use a muted colour token rather than `opacity: 0.3`,
  which dims against the background unpredictably. Never combine with a mask-rise on the
  same element; two split-based effects on one node fight over the DOM.
- **References** — https://gsapvault.com/blog/how-to-animate-on-scroll

---

### Variable weight (`text.variable-weight`)

- **One line** — the typeface's own axes animate: weight, width, slant.
- **What the reader sees** — The headline thickens. Not a fade or a move — the letterforms
  themselves grow heavier, strokes swelling while the shapes stay in place, as though the
  type is being pressed harder into the page. Reversed, it thins to a hairline. Tied to
  scroll, a heading can bulk up as it approaches the centre of the screen and slim again as
  it leaves. Because the change happens *inside* the glyphs rather than to the box around
  them, it reads as a property of the type rather than as an animation applied to it —
  which is why it feels expensive and why it only works with typefaces that were designed
  for it.
- **Mechanism** — animate `font-variation-settings` axes (`wght`, `wdth`, `slnt`, or
  custom), on a clock or scrubbed against scroll.
- **Stack** — CSS, provided the font is variable. No library needed for the property
  itself.
- **Params** — axis and range (stay inside the font's declared range — clamped values
  outside it snap); duration; whether to animate one axis or several (several at once
  usually reads as mush).
- **Use when** — you have a genuine variable font and a display-size heading.
- **Don't use when** — the font is static. There is no polyfill; you are choosing a
  typeface, not an effect.
- **Reduced motion** — settle at the mid or final weight, no animation.
- **Performance** — **this re-renders glyphs every frame.** It is not a transform. On a
  large headline it is one of the more expensive things in this family, though a single
  variable file replacing four to eight static weights is a net saving of roughly 200–500 KB
  on load — the trade is download cost against render cost.
- **Gotchas** — animate `font-variation-settings` as a whole string and the browser may not
  interpolate cleanly; where supported, prefer the individual registered properties
  (`font-weight`, `font-stretch`) which interpolate natively. Check the axis ranges the
  font actually ships — many advertise axes they implement partially.
- **References** — https://www.theinkorporated.com/insights/future-of-typography/ ·
  https://developer.mozilla.org/en-US/docs/Web/CSS/font-variation-settings

---

### Odometer counter (`text.counter-odometer`)

- **One line** — a number counts up to its value instead of simply being there.
- **What the reader sees** — Where a figure will be, digits spin upward rapidly — 0, then a
  blur of increasing values, decelerating as they approach the real number and landing on
  it precisely. It takes a second and a half, most of which is the slow final approach, so
  the eye arrives at the figure just as it stops. The effect is entirely about **making a
  number feel earned**: the same "550" that would have been ignored as static text is now
  something you watched accumulate. It is a stock trick of statistics sections, and it works
  every time, which is presumably why it is on every statistics section.
- **Mechanism** — tween a proxy value, write the formatted result to the DOM on each frame.
  Never animate text content directly.
- **Stack** — any library, or `requestAnimationFrame` in a dozen lines.
- **Params** — duration (1.2–2s); easing (strong ease-out — the deceleration is what sells
  it); formatting (thousands separators preserved, decimals fixed).
- **Use when** — three or four headline metrics, once on a page. Inside a chart rather than
  as standalone type, see `dataviz.counter` and `svg.counter-arc`.
- **Don't use when** — the number is precise and consequential — a price, a balance, a
  dosage. Watching a real figure churn undermines trust in it.
- **Reduced motion** — final value, immediately. The authored markup should contain the
  final value so this is the natural fallback rather than a special case.
- **Performance** — trivial; one DOM write per frame per counter.
- **Gotchas** — use tabular figures or the number's width changes on every frame and the
  row reflows continuously. Put the real value in the HTML and count *from* zero to it, so
  no-JS and reduced-motion readers get the truth. Announce only the final value.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/font-variant-numeric

---

### Outline fill (`text.outline-fill`)

- **One line** — hollow letterforms fill with colour.
- **What the reader sees** — The headline is drawn as outlines only — visible, legible, but
  empty, letters made of thin strokes on the page background. Then colour floods in,
  usually from one side or from below, and the type becomes solid. It is a two-state
  transformation with a clear before and after, and the outline state is genuinely
  attractive in its own right, which sets it apart from effects whose start state is just
  "invisible". On scroll it can be tied to progress so the fill tracks the reader. The
  hazard is contrast: outlined type on a busy background is much harder to read than solid
  type, and that is the state some readers will spend the longest with.
- **Mechanism** — `-webkit-text-stroke` (or `text-stroke`) with a transparent fill, then a
  `background-clip: text` gradient whose position animates, or a clipped duplicate layer.
- **Stack** — CSS. Free.
- **Params** — stroke width (1–2px at display size); fill direction; duration (0.6–1s).
- **Use when** — large display type on a plain background.
- **Don't use when** — the type is under about 40px, where outlines close up and become
  illegible, or over imagery.
- **Reduced motion** — solid filled type from the start.
- **Performance** — `background-clip: text` with an animated gradient repaints the text
  each frame; fine for a headline, not for a page of them.
- **Gotchas** — `-webkit-text-stroke` centres the stroke on the glyph edge, eating into
  the letterform, so outlined type reads visually lighter than the same face solid. Text
  stroke plus `background-clip: text` interact badly in some engines — test both together,
  not separately.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/-webkit-text-stroke

---

### Stroke draw (`text.stroke-draw`)

- **One line** — letterforms draw themselves as if written.
- **What the reader sees** — A line begins at the start of the first letter and traces its
  shape — round the bowl of a *b*, up the stem, across — and the word is written in front
  of you, stroke following stroke, over a second or two. Unlike the typewriter, which
  reveals finished letters, this draws the *path*, so it genuinely resembles handwriting or
  a plotter at work. On a script or monoline face it is beautiful. On a heavy grotesque it
  looks like a stencil being outlined, because the drawn line is a path around a shape
  rather than the shape itself.
- **Mechanism** — the text must be **converted to SVG paths** at build time, then
  `stroke-dashoffset` animated from full to zero on each path.
- **Stack** — SVG plus a draw plugin (GSAP DrawSVG, free since 2025) or plain CSS on
  `stroke-dashoffset`. Requires an asset pipeline step — this is not live text. The general
  technique is `svg.stroke-draw`; for a name specifically, see `svg.signature`.
- **Params** — duration per letter; stagger between letters; stroke width; whether a fill
  appears after the draw completes.
- **Use when** — a logo, a signature, one hero word. Anything short and fixed.
- **Don't use when** — the text is dynamic, translated, or long. Converting to paths means
  it is no longer text.
- **Reduced motion** — show the completed drawing immediately.
- **Performance** — cheap to animate. The cost is payload: outlined type as SVG paths is
  far heavier than the same string as text.
- **Gotchas** — **the element must have a visible stroke and stroke-width or nothing
  renders at all**, which is the classic first-attempt failure. Converted paths are
  invisible to search engines and screen readers; supply the real string as an
  `aria-label` or adjacent visually-hidden text. Complex faces produce enormous path data.
- **References** — https://webflow.com/blog/gsap-becomes-free

---

### Wave (`text.wave`)

- **One line** — characters ride a travelling sine, bobbing in sequence.
- **What the reader sees** — Each letter of the word sits at a slightly different height,
  and those heights move — a smooth undulation passing left to right along the string,
  continuously, like a flag or a line of kelp. Nothing arrives or departs; the word is
  simply alive. At small amplitude it is a gentle shimmer you notice only after a moment.
  At large amplitude it is cartoonish and reads as a children's product or a joke. It is
  ambient rather than an entrance: it never resolves, which means it also never stops
  competing for attention.
- **Mechanism** — per-character `translateY` driven by a phase-shifted sine, either a
  looping keyframe animation with per-character delay or a ticker computing offsets.
- **Stack** — CSS with a per-character delay custom property; any library for the computed
  version.
- **Params** — amplitude (2–6px at display size); wavelength (the delay between adjacent
  characters); period (1.5–3s).
- **Use when** — a playful brand, a single decorative word, a 404 page.
- **Don't use when** — anywhere serious, or anywhere the word must be read quickly.
- **Reduced motion** — static, flat. Continuous motion is the exact category
  `prefers-reduced-motion` exists for.
- **Performance** — a permanent animation on every character; a 20-letter word is 20
  perpetual tweens. Pause when off-screen.
- **Gotchas** — vertically offset characters change the line's optical baseline, which can
  cause the surrounding layout to look misaligned even though nothing moved.
- **References** — https://studiomeyer.io/en/blog/kinetische-typografie

---

### Gradient sweep (`text.gradient-sweep`)

- **One line** — a band of light travels across the type.
- **What the reader sees** — The headline is a flat colour, and a brighter band — a
  highlight, like light raking across a metal sign — moves across it from one side to the
  other, taking a second or two, then repeats after a pause. The letters do not move; only
  the colour inside them changes. It is the cheapest way to make static type feel alive,
  and depending on the palette it reads either as premium (subtle, low contrast, slow) or
  as a discount banner (high contrast, fast, looping tightly). The gap between those two is
  entirely in the timing and the contrast, not in the technique.
- **Mechanism** — a linear-gradient background on the text with `background-clip: text` and
  a transparent fill; the `background-position` animates.
- **Stack** — CSS. Free.
- **Params** — band width (narrow reads as a specular highlight, wide as a colour shift);
  travel duration (1.5–3s); pause between passes (2s+; a continuous loop with no rest is
  what tips it into "sale banner").
- **Use when** — a logotype, one heading, a loading state.
- **Don't use when** — the text is long or the palette is already saturated.
- **Reduced motion** — static gradient or flat colour.
- **Performance** — repaints the text area every frame while running. Small for a heading,
  and it should not run permanently.
- **Gotchas** — `background-clip: text` needs a transparent `color` to show through, which
  means that if the background fails to load or a forced-colours mode is active, the text
  can render invisible. Provide a fallback colour and test in high-contrast mode.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/background-clip

---

### Physics fall (`text.physics-fall`)

- **One line** — the letters detach and fall out of the layout.
- **What the reader sees** — The headline is sitting normally, and then it lets go.
  Characters drop, each with slightly different velocity and spin, tumbling out of the
  bottom of the viewport as though the line had been holding them up and stopped. Some
  drift, some fall straight, and the whole word is gone in under a second. It is startling
  in a way almost nothing else in this family is, because it violates the basic contract
  that text stays where it is put. Precisely for that reason it works exactly once per
  site, as a deliberate punchline or an exit.
- **Mechanism** — split to characters, then a 2D physics tween per character with velocity,
  gravity and angular velocity; usually triggered by scroll or a click.
- **Stack** — GSAP Physics2D (free since 2025), or a real physics engine if the characters
  must collide.
- **Params** — velocity spread; gravity; angular velocity; trigger point.
- **Use when** — an easter egg, a 404, a deliberate exit gag.
- **Don't use when** — the text is content the reader needs. Once it has fallen it is gone.
- **Reduced motion** — the text simply remains, or fades. Do not fall.
- **Performance** — dozens of independently animated elements with rotation; the heaviest
  entry in this file. Kill the tweens once characters leave the viewport.
- **Gotchas** — falling characters extend the document's scrollable area unless the
  container clips them, producing a mysterious scrollbar. Restore the DOM properly
  afterwards or the text is permanently destroyed for anyone who scrolls back.
- **References** — https://webflow.com/blog/gsap-becomes-free

---

### Letter magnet (`text.letter-magnet`)

- **One line** — individual characters lean toward the cursor.
- **What the reader sees** — As the pointer moves across a headline, the letters nearest it
  shift a few pixels toward it and the ones further away stay put, so a soft bulge follows
  the cursor along the word. Move away and the letters ease back to their true positions
  with a slight lag. Nothing is clickable and nothing changes state — it is pure
  responsiveness, the type acknowledging that someone is there. On a big display headline
  it is genuinely delightful and takes about fifteen lines of code; on anything smaller
  the displacement is invisible and you have paid the splitting cost for nothing.
- **Mechanism** — split to characters; on pointer move, compute each character's distance
  from the cursor and apply a falloff-scaled translation via pre-created setters.
- **Stack** — any library with a `quickTo`-style setter, or raw transforms.
- **Params** — radius (150–300px); strength (0.1–0.3 of the distance); return duration
  (0.4–0.6s — the lag is most of the charm).
- **Use when** — one large headline on a pointer-capable device.
- **Don't use when** — touch is the primary input; there is no cursor and the effect simply
  does not exist. It must never be the only affordance.
- **Reduced motion** — no displacement.
- **Performance** — one handler for the whole word, setters created once. **Never create a
  tween inside the pointer handler** — that allocates an object every frame and is the
  standard way this effect becomes janky.
- **Gotchas** — gate on `(hover: hover)` so touch devices skip it entirely. Cache character
  positions and recompute on resize, not per frame; reading layout every pointer event is
  the other standard way to make it janky.
- **References** — https://motion.dev/docs/split-text

---

### Line flip (`text.line-flip-3d`)

- **One line** — lines of type rotate in on a horizontal hinge.
- **What the reader sees** — Each line of the headline is tipped backward, its top edge
  away from you, and swings down into the flat plane one after another — like the split-flap
  boards in old railway stations, or a stack of cards being turned face up. Because the
  rotation is in real perspective, the line is briefly foreshortened and its type
  compresses before resolving. With three lines and a tenth-second offset it reads as
  mechanical and precise. It is the most *physical* of the text entrances and the one that
  most obviously belongs to a particular era of motion design.
- **Mechanism** — split to lines, `perspective` on the container, per-line
  `rotateX(-90deg)`→0 with a stagger.
- **Stack** — CSS plus a line-splitting utility.
- **Params** — angle (start at -60° to -90°); perspective (600–1000px); stagger (0.08–0.12s).
- **Use when** — a headline that should feel constructed or mechanical.
- **Don't use when** — legibility during the animation matters; rotated type is briefly
  unreadable and anti-aliasing shifts noticeably mid-rotation.
- **Reduced motion** — flat and present.
- **Performance** — 3D transforms promote each line to its own layer; fine for three lines.
- **Gotchas** — the same containing-block trap as any 3D transform: a transformed ancestor
  becomes the containing block for `position: fixed` descendants. Rotating text renders
  through a different rasterisation path in some engines, so type can look subtly heavier
  during the animation than at rest.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/transform-function/rotateX

---

## Family notes

**The splitting checklist**, which applies to two-thirds of this file and is the source of
almost every bug in it:

1. Split **after** `document.fonts.ready`.
2. Re-split on resize, and **rebuild the animation inside the same callback** — a re-split
   orphans the old tween and text ends up stranded.
3. Split only the granularity you animate. Splitting into lines, words *and* characters
   when you only animate words triples your node count for nothing.
4. Keep the ARIA repair: `aria-label` on the container, `aria-hidden` on the fragments.
5. If the text contains links or emphasis, do not split it — or split it hidden and supply
   a visually-hidden copy of the real markup.
6. Revert the split on teardown. Leaving a page full of character spans behind is a slow
   leak in a single-page app.

**Choose the coarsest unit that achieves the effect.** Lines over words, words over
characters, every time. Per-character animation is the most expensive, least accessible and
most overused technique in this family, and in most cases the difference is invisible at
reading distance.

**One text effect per screen.** These are all loud. Two at once do not read as
sophisticated; they read as a demo.
