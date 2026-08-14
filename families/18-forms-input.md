# Forms & input

Motion on the surfaces where people give you something. Fields taking focus, labels moving,
errors arriving, steps advancing, submissions resolving.

Forms are the least forgiving place in an interface to animate. Everywhere else a reader is
consuming; here they are *producing*, often reluctantly, and every animation lands on someone
mid-task who is already spending effort. The test for this family is unusually blunt: **does
this movement help someone complete the form, or does it merely decorate the moment they are
being asked to work?**

Three findings shape everything here.

**Top-aligned labels beat clever ones.** Floating labels work for a single well-labelled field
and become error-prone in complex forms; users have a clearer path to completion with
top-aligned labels than with left, right, or animated-into-the-border variants. That is a
design decision the animation cannot rescue.

**Validation motion has an announcement obligation.** Inline validation is fine, and error
messages must be specific and actionable — "Please enter a valid email address", never "Invalid
input" — and announced with `aria-live="polite"` so they reach a screen reader without
interrupting. A red border and a shake say nothing to a substantial share of users.

**Multi-step forms carry a WCAG 2.2 obligation.** Progress must be exposed in *text*, not by
colour or position alone ("Step 2 of 4"), the current step marked with `aria-current="step"`,
focus moved to the new step's heading, and — per 3.3.7 Redundant Entry — values carried forward
so nobody retypes what they already gave you.

Accessibility work here tends to raise conversion, because both are measuring the same thing:
friction removed.

---

### Focus field (`form.focus-field`)

- **One line** — the field acknowledges that it is now the one you are typing in.
- **What the reader sees** — Click or tab into a text field and its border brightens and
  thickens slightly, the label above shifts to the accent colour, and a focus ring appears
  around the whole control — all within about 120 milliseconds. The change is unmistakable
  from across the page, which is the entire point: on a form with fourteen fields, knowing
  where you are is the primary navigational fact, and it must survive being glanced at rather
  than studied.
- **Mechanism** — border, background and outline transitions on `:focus-visible`.
- **Stack** — CSS. No JavaScript.
- **Params** — duration (100–150ms); ring offset (2–3px); whether the label changes colour too.
- **Use when** — every input, select, textarea and custom control.
- **Don't use when** — never. The only wrong version is removing the indicator.
- **Reduced motion** — the state applies instantly. It must still be clearly visible.
- **Performance** — free.
- **Gotchas** — never `outline: none` without an equally visible replacement; this is the most
  common accessibility failure on the web. Use `:focus-visible` so the ring appears for keyboard
  users without decorating every mouse click. The indicator needs contrast against both the
  field and the page background.
- **References** — https://www.a11yproject.com/posts/how-to-write-accessible-forms/ ·
  https://muhammad-haider.medium.com/form-accessibility-labels-errors-focus-states-explained-2026-compliance-guide-553e0a8d971e

---

### Floating label (`form.floating-label`)

- **One line** — the placeholder rises and shrinks to become the label.
- **What the reader sees** — The field shows a single piece of text sitting inside it. Click in,
  and that text travels up and to the left, shrinking as it goes, coming to rest tucked into
  the top border of the field — where it stays as a label while you type. Empty the field and
  it drops back down. It is elegant, it saves vertical space, and it is one of the most
  contested patterns in form design, because the label spends part of its life pretending to be
  a placeholder.
- **Mechanism** — a real `<label>` transitioning `transform: translateY() scale()` on field
  focus or non-empty state, driven by `:focus-within` and `:not(:placeholder-shown)`.
- **Stack** — CSS only, provided the label is a real label above the input in the DOM.
- **Params** — duration (150–200ms); scale (0.75–0.85); easing (ease-out).
- **Use when** — short, simple forms — a login, a single search field, a newsletter signup.
- **Don't use when** — the form is long or complex, or fields need hint text as well as labels.
  Top-aligned static labels measurably outperform this in complex forms, and the hint has
  nowhere to live once the placeholder slot is taken.
- **Reduced motion** — the label sits in its raised position permanently, or moves without
  transition.
- **Performance** — free.
- **Gotchas** — it must be a real `<label>` with a `for` attribute, never a placeholder doing
  double duty — a placeholder-as-label disappears the moment typing starts, which fails anyone
  who looks away mid-field. The raised label is small, so check its contrast at that size.
  Browser autofill often does not fire the events the raised state depends on, leaving the
  label overlapping filled text.
- **References** — https://getbootstrap.com/docs/5.0/forms/floating-labels/ ·
  https://www.lazi-akademie.de/wiki/mediendesign-digitale-medien/ux-patterns/form-design/

---

### Inline validation (`form.inline-validation`)

- **One line** — the field reports whether its contents are acceptable, in place.
- **What the reader sees** — Move out of a field with something malformed and its border turns
  red as a specific message slides into the space beneath it: not "Invalid input" but "Please
  enter a valid email address". Correct it and the message slides away and the border returns.
  The reserved space means nothing below jumps as messages appear and disappear, which matters
  on a form where three fields might be complaining at once.
- **Mechanism** — colour transitions on the field, a height or opacity transition on the message
  region, and a live region announcing the message.
- **Stack** — CSS transitions plus form state; the timing of *when* to validate is the real
  design work.
- **Params** — transition (150–250ms); validate on blur first, then live on subsequent edits;
  reserve the message row's height.
- **Use when** — any field with rules a user can violate.
- **Don't use when** — validating from the first keystroke. Telling someone their email is
  invalid when they have typed two characters is hostile, and it is the most common
  implementation error.
- **Reduced motion** — message appears without sliding; colour and icon unchanged.
- **Performance** — negligible.
- **Gotchas** — announce with `aria-live="polite"` and link the message with `aria-describedby`
  plus `aria-invalid`; colour and motion alone reach nobody using a screen reader. Reserve the
  space or the form jumps under the user's cursor as they work down it.
- **References** — https://www.reform.app/blog/accessible-form-validation-best-practices ·
  https://wcagkit.com/blog/accessible-forms-guide/

---

### Step progression (`form.step-progress`)

- **One line** — a multi-step form advances, and shows how far along you are.
- **What the reader sees** — Complete a step and the current panel slides out to the left while
  the next slides in from the right, taking about 300 milliseconds, and the progress indicator
  above advances — a segment filling, a number ticking from "Step 2 of 4" to "Step 3 of 4". The
  direction encodes travel: going back reverses it, so the form has a spatial model you can hold
  in your head. Nothing you entered is lost when you go back, and that is more reassuring than
  any animation.
- **Mechanism** — a directional slide between panels driven by navigation direction, plus a
  progress bar or step indicator transition.
- **Stack** — any transition library; the state handling matters far more than the motion.
- **Params** — slide duration (250–350ms); direction from travel; progress transition (0.3s).
- **Use when** — checkouts, onboarding, applications, anything long enough to need chunking.
- **Don't use when** — the form is short enough to show at once. Splitting three fields across
  three steps is friction dressed as simplicity.
- **Reduced motion** — panels swap without sliding; the progress indicator updates instantly.
- **Performance** — trivial; both panels exist briefly during the transition.
- **Gotchas** — **move focus to the new step's heading** on advance, or a keyboard user is left
  focused on a button that no longer exists. Expose progress as text, not colour or position
  alone. Mark the current step with `aria-current="step"`. And carry values forward — WCAG 2.2's
  Redundant Entry criterion means re-asking for data the user already gave is a failure, not
  merely rude.
- **References** — https://bati-itao.github.io/learning/esdc-self-paced-web-accessibility-course/module6/multi-step-forms.html ·
  https://www.thewcag.com/examples/forms

---

### Submit resolution (`form.submit-resolve`)

- **One line** — the submit button becomes the outcome.
- **What the reader sees** — Press Submit and the label is replaced by a spinner inside the same
  button, which keeps its size and position; the button is now disabled but visibly busy. On
  success the spinner becomes a tick and the label reads "Sent", holding for a moment. On
  failure the button returns to its resting state and an error appears above the form. The
  feedback stays exactly where the action happened, so nobody has to hunt the page for the
  result of what they just did.
- **Mechanism** — a state machine on one element — idle, busy, success, error — with crossfades
  between, and a fixed width so nothing reflows.
- **Stack** — CSS transitions plus form state; `micro.delayed-spinner` governs when the busy
  state appears.
- **Params** — spinner delay (~250ms, so fast submissions never flash); success hold (1.5–2s);
  crossfade (150ms).
- **Use when** — every form submission.
- **Don't use when** — the result is a page navigation that will arrive quickly; then the
  navigation is the feedback.
- **Reduced motion** — states swap without crossfading; a non-spinning busy indicator.
- **Performance** — trivial.
- **Gotchas** — fix the button's width across all states or it resizes at every transition.
  Disable it on submit to prevent double-posting, but keep it focusable and announce the busy
  state with `aria-busy`; a disabled button that silently loses focus strands keyboard users.
  Errors must be announced and focus moved to the message.
- **References** — https://rangle.io/blog/everything-you-need-to-know-about-designing-accessible-forms

---

### Autocomplete dropdown (`form.autocomplete`)

- **One line** — suggestions appear and filter as you type.
- **What the reader sees** — Type two characters into a field and a list drops beneath it,
  scaling very slightly from the top edge. Keep typing and the list contents change — but the
  list itself does not re-animate, it simply holds its position while rows are replaced, so the
  target under your cursor does not run away. Arrow down and a highlight moves between rows.
  Pick one and the list closes as the field fills.
- **Mechanism** — a dropdown entry animation on first open only, with row content updating
  without transition thereafter; a moving highlight for keyboard selection.
- **Stack** — the Popover API or a listbox pattern; the timing discipline is the interesting
  part.
- **Params** — open (120–150ms); **row updates: no animation**; highlight movement (80–120ms).
- **Use when** — search fields, address entry, tag pickers, anything with a known value set.
- **Don't use when** — the list is short enough to show as radio buttons.
- **Reduced motion** — opens instantly.
- **Performance** — the list must keep up with typing; animating rows per keystroke is the
  failure. Animate the highlight, never the rows.
- **Gotchas** — this needs the full combobox pattern — `aria-expanded`, `aria-activedescendant`,
  and results announced as a live region. Animating rows while someone types moves their target
  mid-reach, which is the single worst thing this pattern can do.
- **References** — https://www.w3.org/WAI/ARIA/apg/patterns/combobox/

---

### Character counter (`form.char-counter`)

- **One line** — a live count reacts as the limit approaches.
- **What the reader sees** — A quiet "0/280" under a textarea, ticking up as you type. Nothing
  happens until you are close to the limit, at which point the counter changes colour, and at
  the limit it turns red and stops. The escalation is what makes it useful: for most of the
  time it is unobtrusive information, and it only becomes an alert when the alert is warranted.
- **Mechanism** — text update on input, with colour transitions at defined thresholds.
- **Stack** — CSS transitions plus an input listener.
- **Params** — warning threshold (~85% of limit); colour transition (200ms); whether the count
  shows remaining or used.
- **Use when** — hard limits the user can hit: messages, bios, SMS fields.
- **Don't use when** — the limit is generous enough that nobody reaches it; the counter is then
  just anxiety.
- **Reduced motion** — colour changes without transition.
- **Performance** — one DOM write per keystroke; use tabular figures to stop the row twitching.
- **Gotchas** — announce sparingly, with a live region that only fires near the limit — a count
  announced on every keystroke floods a screen reader. Colour alone must not carry the warning;
  the number is the signal.
- **References** — https://wcagkit.com/blog/accessible-forms-guide/

---

### Password strength (`form.password-strength`)

- **One line** — a meter fills and changes colour as a password improves.
- **What the reader sees** — Start typing a password and a thin bar beneath the field grows and
  shifts through colour as it does — a short red segment, then longer and amber, then green
  across the full width — with a word beside it: weak, fair, strong. It updates as you type, so
  you can watch adding a word push it up a band, which is a far more effective teacher than a
  list of rules printed above the field.
- **Mechanism** — a `scaleX` on the meter plus a colour transition, both driven by a strength
  score.
- **Stack** — CSS transitions; the scoring library is the substantive dependency.
- **Params** — transition (200–300ms — slow enough to see the change, fast enough to feel live);
  band count (3–4).
- **Use when** — account creation, password changes.
- **Don't use when** — the rules are strict and binary. Then show the rules as a checklist that
  ticks off, which is clearer than a score.
- **Reduced motion** — the meter jumps between states.
- **Performance** — score on a debounce rather than on every keystroke if the scorer is heavy.
- **Gotchas** — the meter must have a text equivalent and a live region; a colour bar tells a
  screen reader nothing. Never block submission on a subjective score, and never send the
  password anywhere to score it.
- **References** — https://accessibility.build/guides/accessible-forms

---

### Field reveal (`form.conditional-field`)

- **One line** — answering one question reveals the fields that depend on it.
- **What the reader sees** — Tick "I have a different billing address" and a group of fields
  expands into existence below it, the space opening as the fields fade in, everything beneath
  moving down smoothly rather than jumping. Untick it and they collapse away. The form appears
  to be responding to you rather than presenting a wall of possibilities you must ignore.
- **Mechanism** — a height or `grid-template-rows` transition on the container (see
  `layout.height-auto`) plus a short fade on the contents.
- **Stack** — CSS, using the `0fr → 1fr` grid technique which needs no measurement.
- **Params** — expand (250–350ms); content fade (starting at ~60% of the expansion); collapse
  faster than expand.
- **Use when** — conditional logic: billing addresses, "other" options, optional detail.
- **Don't use when** — the revealed content is long. A form that grows by a screen is
  disorienting; consider a step instead.
- **Reduced motion** — fields appear instantly.
- **Performance** — a height transition reflows the form below it; keep the group small.
- **Gotchas** — hidden fields must be removed from the tab order and the accessibility tree,
  not merely clipped — a keyboard user tabbing into an invisible field is lost. Announce the
  appearance, and ensure the trigger carries `aria-expanded`. Newly revealed fields should not
  steal focus unless the user explicitly asked to add something.
- **References** — https://www.a11yproject.com/posts/how-to-write-accessible-forms/

---

### OTP entry (`form.otp-entry`)

- **One line** — a row of single-character boxes advances itself as you type.
- **What the reader sees** — Six small boxes. Type a digit and it appears in the first while
  focus jumps to the second, the active box highlighted with a ring that travels along the row
  as you go. Paste a full code and all six fill at once. Delete, and focus steps backwards. When
  the last box is filled the form submits itself. The advancing highlight makes the sequence
  feel like a single field rather than six, which is what it is pretending to be.
- **Mechanism** — focus management between inputs, with a transitioning focus ring; often the
  highlight is one shared element that travels.
- **Stack** — inputs plus focus handling; `autocomplete="one-time-code"` for the platform assist.
- **Params** — highlight transition (100–150ms); auto-submit on completion; paste handling.
- **Use when** — two-factor codes, email verification, PINs.
- **Don't use when** — the code is long or alphanumeric with ambiguous characters.
- **Reduced motion** — the ring jumps between boxes.
- **Performance** — trivial.
- **Gotchas** — handle paste across all boxes, or users pasting a code from their email get one
  character. Set `autocomplete="one-time-code"` and `inputmode="numeric"` so platforms can
  offer the code automatically. Backspace on an empty box must move back, and the whole group
  needs one accessible label rather than six meaningless ones.
- **References** — https://www.thewcag.com/examples/forms

---

### Range slider (`form.range-slider`)

- **One line** — a handle drags along a track, with the value following it.
- **What the reader sees** — Drag the handle and it tracks your pointer exactly while the filled
  portion of the track grows behind it and a value bubble follows above, updating live. Release
  near a marked value and it settles onto it. Use the arrow keys and it steps precisely. The
  bubble is what makes it usable — without it the handle's position is the only feedback, and
  positions are not numbers.
- **Mechanism** — direct positioning during drag with no easing, a settle animation on release,
  and a value bubble tracking the handle.
- **Stack** — a styled `<input type="range">` keeps keyboard and assistive-technology support
  for free; a custom control has to rebuild all of it.
- **Params** — no easing during drag; settle (150–200ms); bubble offset; snap radius if stepped.
- **Use when** — price filters, volume, any continuous bounded value.
- **Don't use when** — precision matters. Provide a paired number input alongside.
- **Reduced motion** — remove the settle animation; direct dragging stays.
- **Performance** — one transform during drag; do not re-render the whole control per frame.
- **Gotchas** — build on the native input. Its keyboard behaviour, `aria-valuenow` and screen
  reader support are substantial, and hand-built sliders routinely ship without any of it. Touch
  targets must be at least 44px regardless of how thin the handle looks.
- **References** — https://www.w3.org/WAI/ARIA/apg/patterns/slider/

---

### Save state (`form.autosave`)

- **One line** — the form quietly reports that it has kept your work.
- **What the reader sees** — You stop typing, and a moment later a small line near the form
  reads "Saving…", then "Saved" with a tick, fading back to a timestamp — "Saved 2 minutes ago"
  — that quietly updates. Nothing interrupts, nothing needs dismissing, and the anxiety of a
  long form is removed by a piece of text that changes about once a minute.
- **Mechanism** — a debounced save with a three-state indicator, each state crossfading, then
  decaying to a relative timestamp.
- **Stack** — a debounce plus a small state indicator.
- **Params** — debounce (1–3s after typing stops); transitions (150ms); how long "Saved" shows
  before becoming a timestamp (2–3s).
- **Use when** — long forms, editors, anything a user would be distressed to lose.
- **Don't use when** — saving has side effects — publishing, notifying, charging. Autosave must
  be genuinely inconsequential.
- **Reduced motion** — states change without crossfade.
- **Performance** — the debounce is doing the real work; without it you are saving per keystroke.
- **Gotchas** — announce politely and infrequently; a live region firing on every save is
  intolerable. Failures must be *loud* — a silent failed autosave is worse than never having
  claimed to save at all.
- **References** — https://rangle.io/blog/everything-you-need-to-know-about-designing-accessible-forms

---

## Family notes

**The pattern choice outranks the animation.** Top-aligned labels beat animated ones in complex
forms; a specific error message beats a beautiful error animation; carrying values forward beats
any transition between steps.

**Never animate under a typing user.** Autocomplete rows, conditional fields and validation
messages that move while someone is typing shift their target mid-reach. Reserve space, animate
the highlight rather than the list, and keep validation to blur until a field has been corrected
once.

**Every visual signal needs an announced equivalent.** Error colours, strength meters, progress
bars, save indicators and character counts are all silent. `aria-live`, `aria-invalid`,
`aria-describedby`, `aria-current` — this family carries more of that obligation than any other
in the catalogue.

**Build on native controls.** `<input type="range">`, real `<label>` elements, the combobox
pattern, `autocomplete` tokens. Each hand-built replacement starts by discarding keyboard and
screen-reader behaviour that took years to standardise.

**Fast, and shorter on the way out.** 100–300ms throughout. Someone filling a form is working,
and every millisecond of animation is time they are not making progress.
