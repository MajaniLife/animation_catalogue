# Micro-interaction & feedback

Small motion that confirms a state change. The button acknowledging the press, the toggle
sliding, the field shaking, the toast arriving, the spinner covering a wait.

This is the only family in the catalogue that is **not optional**. Everything else can be
removed and the page still works; remove feedback and the interface genuinely breaks —
people press buttons twice, submit forms twice, and conclude the site is broken when it was
merely silent. Motion here is not decoration, it is the answer to "did that work?"

**The timing is the craft, and the numbers are well established.** Nielsen Norman's
thresholds still govern: **0.1 second** is the limit for something to feel instantaneous,
**1 second** for an uninterrupted flow of thought, **10 seconds** for holding attention at
all. From these, working ranges follow: button feedback lands between **100–300ms**, general
feedback animations between **200–500ms**, and a loading indicator should be **delayed by
roughly 250ms** before appearing so fast operations never flash a spinner at all.

The mistake that defines the family is treating these like other animations. An entrance can
take 600ms and feel considered; a button that takes 600ms to acknowledge a press feels
broken. **Feedback should be near-instant and short.** The animation is a receipt, not a
performance.

The second mistake is animating the *response* instead of the *acknowledgement*. The press
must be confirmed immediately, locally, before any network call resolves — otherwise every
interaction inherits the latency of your backend.

---

### Press response (`micro.press`)

- **One line** — the button visibly reacts the instant it is pressed.
- **What the reader sees** — Press down and the button dips: it scales down a couple of
  percent and darkens slightly, immediately, before you have finished the click. Release and
  it returns. There is no wait for anything to happen elsewhere — the surface responds to
  your finger the way a physical key does. It is the single most important animation in an
  interface, and it takes about 80 milliseconds, most of which is the return.
- **Mechanism** — `transform: scale(0.97)` and a background shift on `:active` /
  `pointerdown`, released on pointerup.
- **Stack** — CSS `:active`. No library, no JavaScript.
- **Params** — scale (0.96–0.98); press duration (as close to instant as possible — 50–80ms);
  release (100–150ms, slightly slower than the press).
- **Use when** — every button, every tappable card, everywhere.
- **Don't use when** — never. If an element is pressable it should show that it was pressed.
- **Reduced motion** — a colour or brightness change without the scale. Keep the feedback;
  drop the movement.
- **Performance** — free.
- **Gotchas** — `:active` on touch requires care: iOS historically needs a touch handler
  before `:active` applies. The press state must not be confused with the focus state — they
  are different signals and both must exist. Never delay the press feedback behind a network
  call.
- **References** — https://createbytes.com/insights/microinteractions-ui-best-practices

---

### Toggle slide (`micro.toggle`)

- **One line** — a switch's knob travels between its two positions.
- **What the reader sees** — Tap a switch and the knob slides across its track while the track
  changes colour behind it, over about two hundred milliseconds. The knob often stretches
  slightly as it starts and compresses as it lands — the classic squash-and-stretch that makes
  it feel like a physical object being thrown rather than a graphic being repositioned. When
  it arrives the colour change has completed with it, so the on state is unambiguous.
- **Mechanism** — `translateX` on the knob plus a background colour transition on the track;
  optionally a brief `scaleX` for the stretch.
- **Stack** — CSS transitions on a checkbox's sibling.
- **Params** — duration (150–250ms); easing (ease-out, or a light spring for the stretch);
  stretch amount (1.1× at most).
- **Use when** — binary settings that apply immediately.
- **Don't use when** — the setting requires saving. A switch implies instant effect; if there
  is a Save button, use a checkbox.
- **Reduced motion** — the knob jumps; the colour still changes.
- **Performance** — free.
- **Gotchas** — colour alone must never carry the state, for colour-blind users — position
  does that work, which is why the knob must actually move. The underlying control must be a
  real checkbox or a `role="switch"` with `aria-checked`, or the state is invisible to
  assistive technology.
- **References** — https://millermedia7.com/blog/microinteractions-in-user-experience-design/

---

### Optimistic commit (`micro.optimistic`)

- **One line** — the interface acts as if the request already succeeded.
- **What the reader sees** — Tap the heart and it fills immediately, with a little bounce and
  the count incrementing — no spinner, no delay, no waiting for a server. If the request later
  fails, the heart quietly empties again with a brief message. Because the common case is
  success, the interface feels instantaneous almost always, and pays the cost of a rare
  correction rather than the cost of a universal wait.
- **Mechanism** — apply the state change locally on interaction, fire the request, and roll
  back with an explanation if it fails.
- **Stack** — state management, not animation; the animation is whatever the success state
  looks like.
- **Params** — commit animation (150–300ms); rollback animation (slower and more visible than
  the commit — a silent revert is worse than none).
- **Use when** — likes, bookmarks, reorders, toggles, anything low-stakes and reversible.
- **Don't use when** — the action is consequential: payments, deletions, submissions. There,
  the wait is honest and the confirmation must be real.
- **Reduced motion** — same behaviour, no bounce.
- **Performance** — the fastest possible interface, because it removes the network from the
  perceived interaction.
- **Gotchas** — the rollback is the whole risk. It must be visible, explained, and must not
  silently discard other changes the user has made since. Announce both the optimistic state
  and any rollback to assistive technology.
- **References** — https://medium.com/@kaleemkhan/what-is-the-ideal-waiting-time-before-you-show-loading-indicator-cf094528faec

---

### Delayed spinner (`micro.delayed-spinner`)

- **One line** — the loading indicator waits before appearing, so fast responses never show one.
- **What the reader sees** — Click something and, if the response is quick, nothing happens
  except the result — no flash of a spinner, no flicker. If the wait runs longer, a spinner
  fades in and rotates steadily until the content replaces it. What the user notices is that
  fast actions feel completely clean, because they were never told to wait for something that
  did not take any time.
- **Mechanism** — start a ~250ms timer on request; only render the indicator if the request is
  still pending when it fires. Once shown, keep it for a minimum period so it does not flash.
- **Stack** — a timer and a state flag; the spinner itself is CSS.
- **Params** — appearance delay (~250ms); minimum visible duration once shown (300–500ms);
  spinner period (0.8–1.2s per rotation).
- **Use when** — every asynchronous operation between roughly 200ms and 10s.
- **Don't use when** — the operation exceeds ~10 seconds; beyond that a bare spinner stops
  reassuring anyone and you need real progress or a stated estimate.
- **Reduced motion** — a non-spinning indicator: a pulsing dot, or a static "Loading…" with a
  live region. Continuous rotation is exactly the motion some users have asked to avoid.
- **Performance** — one rotating element. Trivial, but if it is rendered permanently and merely
  hidden, it is animating for nothing — mount it on demand.
- **Gotchas** — the minimum-visible-duration rule is the half everyone forgets: without it, a
  request that resolves at 260ms shows a spinner for 10ms, which reads as a glitch. Loading
  state must be announced (`aria-busy`, or a live region), because a spinner is silent.
- **References** — https://www.eleken.co/blog-posts/spinner-ui · https://medium.com/@kaleemkhan/what-is-the-ideal-waiting-time-before-you-show-loading-indicator-cf094528faec

---

### Success check (`micro.success-check`)

- **One line** — a tick draws itself to confirm completion.
- **What the reader sees** — The button's label disappears, a circle fills with green, and a
  checkmark draws itself inside it in two strokes — down, then up — over about three hundred
  milliseconds. It holds for a second and either stays or returns the control to its resting
  state. The drawing is what makes it feel like a verdict rather than an icon swap: you watch
  the confirmation being written.
- **Mechanism** — a stroke-draw on the tick path (see `svg.stroke-draw`), usually with a
  scale-in on the containing circle.
- **Stack** — SVG plus a dash-offset animation; no library needed with `pathLength="1"`.
- **Params** — draw duration (250–400ms); hold before reverting (1–2s); whether the button
  reverts at all.
- **Use when** — form submission, save confirmation, copy-to-clipboard.
- **Don't use when** — the result needs explanation. A tick says "done", not "done, and here
  is what changed".
- **Reduced motion** — the tick appears without drawing.
- **Performance** — trivial.
- **Gotchas** — success must be announced, not only drawn; a screen reader user gets nothing
  from a tick. Do not revert so quickly that someone who glanced away misses it entirely — and
  do not use green alone as the signal.
- **References** — https://www.justinmind.com/web-design/micro-interactions

---

### Toast entry (`micro.toast`)

- **One line** — a notification slides in, holds, and leaves.
- **What the reader sees** — A small panel slides up from a corner, holds long enough to read,
  and then slides out. If another arrives while it is there, the first moves aside to make
  room and the stack stays orderly. If you hover it, the dismissal timer pauses so it does not
  vanish while you are reading it. The entrance and exit are quick — a couple of hundred
  milliseconds — because the toast is a message, not an event.
- **Mechanism** — an entrance transform plus a layout animation for the stack when siblings
  are added or removed (see `layout.list-add-remove`).
- **Stack** — a toast library, or presence handling plus FLIP for the stack.
- **Params** — entrance (200–300ms); hold (4–6s, longer for longer messages); exit
  (150–250ms — faster than entry); maximum simultaneous toasts (3).
- **Use when** — non-blocking confirmations and errors.
- **Don't use when** — the message is critical or requires action. Toasts disappear, and
  anything that must be acknowledged should not.
- **Reduced motion** — appear and disappear without sliding.
- **Performance** — negligible.
- **Gotchas** — pause the timer on hover *and* on focus, or keyboard users cannot read a long
  message. Toasts need a live region — polite for confirmations, assertive for errors — and
  must be dismissible. Auto-dismissal is a WCAG timing consideration, not just a design
  preference.
- **References** — https://createbytes.com/insights/microinteractions-ui-best-practices

---

### Field validation (`micro.field-validation`)

- **One line** — a field shows, in place, whether what you typed is acceptable.
- **What the reader sees** — Leave an email field with something malformed and its border
  turns red as a message appears beneath it, the field shaking briefly. Correct it and the
  message slides away as the border returns to normal, sometimes with a small tick appearing
  at the right edge. The correction is acknowledged as clearly as the error was — which is the
  half that most forms get wrong, leaving the error styling until submission.
- **Mechanism** — a colour transition on the field, a height or fade transition on the
  message, and optionally a short decaying shake (see `physics.wiggle`).
- **Stack** — CSS transitions; the timing of *when* to validate is the real design decision.
- **Params** — transition (150–250ms); shake only on submit-time failure, not while typing;
  validate on blur, then live on subsequent edits.
- **Use when** — any form with rules the user can violate.
- **Don't use when** — validating on every keystroke from the first character. Telling someone
  their email is invalid when they have typed two letters is hostile.
- **Reduced motion** — no shake; keep colour, icon and message.
- **Performance** — negligible; note that animating the message's height reflows the form
  below it, which is why reserving the space is kinder.
- **Gotchas** — colour must never be the only error signal; the message text is the signal and
  needs `aria-describedby` plus `aria-invalid`. Reserve space for the message or the whole
  form jumps as errors appear and disappear.
- **References** — https://millermedia7.com/blog/microinteractions-in-user-experience-design/

---

### Copy confirmation (`micro.copy-confirm`)

- **One line** — the copy button briefly becomes a confirmation.
- **What the reader sees** — Click the copy icon and it changes to a tick, with the label
  swapping to "Copied" — for about a second and a half — before reverting to its resting
  state. Nothing else moves: no toast slides in from a corner, no dialog appears, nothing
  demands dismissal. The feedback happens exactly where you clicked, which is exactly where
  you are already looking, and it costs you no attention to receive it. The revert is what
  makes it repeatable — the button returns to its normal state ready to be used again, so
  copying three things in a row produces three identical confirmations rather than an
  accumulating pile of notifications.
- **Mechanism** — a state swap with a short crossfade or icon morph, on a timer.
- **Stack** — CSS transition plus a timeout.
- **Params** — swap duration (150ms); hold (1.2–2s); revert (150ms).
- **Use when** — copy buttons, code blocks, share links, API keys.
- **Don't use when** — the action needs a persistent record.
- **Reduced motion** — instant swap, no crossfade.
- **Performance** — free.
- **Gotchas** — announce it in a live region; the visual swap is silent. Handle rapid repeat
  clicks by resetting the timer rather than queueing several reverts. Keep the button's width
  fixed across both states or the layout twitches on every copy.
- **References** — https://www.justinmind.com/web-design/micro-interactions

---

### Ripple (`micro.ripple`)

- **One line** — a circular wave expands from the exact point of contact.
- **What the reader sees** — Tap a surface and a translucent circle grows from precisely where
  your finger landed, spreading across the element and fading as it goes, over about half a
  second. Because it starts at the contact point rather than the centre, it confirms not just
  that you tapped but *where* — which on a large touch target is genuinely useful information.
  It is strongly associated with one design system, and using it outside that context reads as
  borrowed.
- **Mechanism** — a circle element positioned at the contact coordinates, scaled up and faded
  out, clipped by the parent's overflow.
- **Stack** — CSS plus a pointer handler for the origin.
- **Params** — duration (400–600ms); opacity (0.1–0.2); whether it completes after release.
- **Use when** — large touch targets, list rows, cards on touch-first interfaces.
- **Don't use when** — your visual language is not Material. It carries that association.
- **Reduced motion** — a plain background flash with no expansion.
- **Performance** — one transform and opacity per tap; clean up the element afterwards or they
  accumulate.
- **Gotchas** — the ripple must not delay the actual action; start both together. On a
  keyboard-activated button there is no contact point — fall back to the centre.
- **References** — https://bricxlabs.com/blogs/micro-interactions-2025-examples

---

### Focus ring transition (`micro.focus-ring`)

- **One line** — the keyboard focus indicator moves rather than teleporting.
- **What the reader sees** — Tab through a form and the focus outline travels from field to
  field, resizing as it goes to fit each one, arriving in about 150 milliseconds. Instead of a
  ring blinking out in one place and appearing in another, you follow a single object across
  the interface, which makes it far easier to keep track of where you are — particularly in a
  dense layout.
- **Mechanism** — a single shared indicator element positioned to the focused element's
  rectangle via transform and scale (the same technique as `layout.tab-indicator`).
- **Stack** — a focus listener plus a positioned element; the native `:focus-visible` outline
  is the fallback and must remain for anyone this fails for.
- **Params** — duration (120–180ms — any slower and it lags behind fast tabbing); easing
  (ease-out).
- **Use when** — dense interfaces, custom design systems where the native outline is already
  being restyled.
- **Don't use when** — you would be replacing a perfectly good native outline for aesthetics.
  The native one is more robust in more situations.
- **Reduced motion** — the ring jumps instantly. It must still be clearly visible.
- **Performance** — one element, transform-only.
- **Gotchas** — this is the highest-stakes entry in the file: if the shared indicator ever
  fails to appear, keyboard users are lost entirely. Keep `:focus-visible` styling underneath
  as a guaranteed baseline, never remove the outline outright, and ensure the indicator meets
  contrast requirements against every background it crosses.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/:focus-visible

---

### Count change (`micro.count-change`)

- **One line** — a number visibly ticks when it changes.
- **What the reader sees** — A cart badge showing 2 becomes 3: the old digit slides up and out
  while the new one rises into its place, in about two hundred milliseconds, and the badge
  pulses very slightly. It is small enough to miss if you are looking elsewhere and impossible
  to miss if you are looking at it, which is exactly the right weighting for a background
  fact that just changed.
- **Mechanism** — a clipped container with the old and new digits translating in sequence; a
  brief scale pulse on the container.
- **Stack** — CSS transitions plus a DOM swap; the same clipped-swap pattern as
  `pointer.stack-swap`.
- **Params** — slide duration (150–250ms); direction (up for increments, down for decrements
  — this small consistency is what makes it readable); pulse scale (1.1 at most).
- **Use when** — cart counts, notification badges, live totals.
- **Don't use when** — values change several times a second; a constantly ticking number is
  unreadable.
- **Reduced motion** — the number changes with no slide; keep any colour or weight change.
- **Performance** — trivial.
- **Gotchas** — use tabular figures so the badge does not resize between 9 and 10. Announce
  the change politely, and rate-limit the announcement or a fast-updating count floods a
  screen reader.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/font-variant-numeric

---

### Skeleton shimmer (`micro.skeleton-shimmer`)

- **One line** — placeholder blocks pulse to say "still loading" rather than "broken".
- **What the reader sees** — Grey blocks in the shape of the content, with a soft band of
  lighter grey travelling slowly across them every second and a half. The movement is what
  distinguishes a loading state from a rendering failure — a static grey box could be either,
  while a moving one is unambiguously in progress. When the data arrives the blocks crossfade
  into the real content at exactly the same size, so nothing jumps.
- **Mechanism** — a looping gradient translation across the placeholder; the swap is a short
  crossfade (see `entrance.skeleton-swap`).
- **Stack** — CSS keyframes on a gradient background.
- **Params** — period (1.5–2s; faster reads as anxious); contrast (low — the shimmer should be
  barely there); crossfade on swap (150–250ms).
- **Use when** — content whose shape is known before its data arrives.
- **Don't use when** — the wait is under about 300ms; use the delayed-spinner rule and show
  nothing.
- **Reduced motion** — static placeholders with no shimmer.
- **Performance** — a permanently running animation per placeholder. Thirty skeletons is
  thirty loops competing with the fetch you are waiting for — a real cost, and an argument for
  fewer, larger placeholders.
- **Gotchas** — the skeleton must match the real content's dimensions or the swap causes the
  layout shift it existed to prevent. Hide skeletons from assistive technology and announce the
  loading state properly instead.
- **References** — https://www.eleken.co/blog-posts/spinner-ui

---

## Family notes

**Learn the three thresholds.** 0.1s feels instantaneous, 1s keeps a flow of thought intact,
10s is the limit of attention. Every timing decision in this file descends from those.

**Feedback is short.** 100–300ms for a press, 200–500ms for most confirmations. If it takes
longer than half a second, it is no longer feedback — it is an animation the user is waiting
through.

**Acknowledge locally, then resolve.** Confirm the press immediately from the client. Never
let an interaction's perceived responsiveness depend on a network round trip.

**Delay the spinner, then hold it.** ~250ms before showing, then a minimum visible period once
shown. Both halves matter; each without the other produces a flicker.

**Motion is never the only signal.** Every entry here has an assistive-technology obligation
attached — live regions, `aria-invalid`, `aria-busy`, `role="switch"`. A tick that draws and a
toast that slides are both completely silent to a screen reader, and this is the one family
where the information genuinely matters.
