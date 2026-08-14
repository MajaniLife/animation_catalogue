# Overlays — modals, drawers, tooltips, popovers

Motion for content that appears *over* the page rather than in it. Dialogs, popovers, tooltips,
sheets, context menus, banners.

Navigation overlays live in `families/17` and the FLIP technique for growing a dialog out of its
trigger is `layout.modal-from-origin`; this file is about the overlay **system** — the top
layer, the backdrop, dismissal, stacking, and the focus choreography that all of them share.

**The platform now does most of this, and that is the headline for 2026.** The `<dialog>`
element and the Popover API both promote content to the **top layer**, which sits above
everything regardless of `z-index` or stacking contexts — the single most common source of
"why is my modal behind the header" bugs. Popover adds light dismiss (outside click and Escape)
for free. Hand-rolled overlays in 2026 are usually reimplementing this, worse.

**The exit-animation problem, and its fix.** Both `<dialog>` and popovers use `display: none`
when closed, so entry animation needs `@starting-style` — and *exit* animation needs two more
things, which is where most implementations fail. First, `transition-behavior: allow-discrete`
so `display` participates in the transition. Second, and less obviously, you must **transition
the `overlay` property too**: by default a closing element is removed from the top layer
instantly, so it gets clipped and obscured mid-animation and the exit is never seen.
Transitioning `overlay` with `allow-discrete` keeps it in the top layer until the animation
finishes.

**The focus contract is the same everywhere here** and is not optional: focus moves in on open,
is trapped for modals, Escape closes, and focus returns to the trigger on close.

---

### Modal dialog (`overlay.modal-dialog`)

- **One line** — a dialog scales up into the centre while the page recedes behind it.
- **What the reader sees** — The page dims and blurs very slightly, and a panel appears in the
  centre, scaling up from about 96% and fading in over roughly 200 milliseconds. It arrives
  fast enough to feel like a response to the click rather than a production. Closing reverses
  it, slightly quicker. Because the background is visibly demoted rather than replaced, you
  keep your sense of where you were — the dialog reads as *on top of* the page rather than as
  a new place.
- **Mechanism** — `scale` and `opacity` on the dialog with `@starting-style` for entry, plus
  `allow-discrete` on `display` and `overlay` for the exit; a backdrop transition alongside.
- **Stack** — the native `<dialog>` element with `showModal()`, which gives the top layer,
  the `::backdrop` pseudo-element, Escape handling and focus containment natively.
- **Params** — duration (180–250ms in, 150–200ms out); scale from (0.95–0.97); backdrop fade
  slightly longer than the panel so it does not finish first.
- **Use when** — confirmations, focused tasks, anything requiring a decision before continuing.
- **Don't use when** — the content is a whole workflow. That is a page, and a dialog you cannot
  see past becomes a prison.
- **Reduced motion** — appear at full scale with a very short fade, or instantly.
- **Performance** — one composited panel plus a backdrop; the blur on the backdrop is the only
  real cost and should be modest.
- **Gotchas** — `showModal()` gives inertness of the background for free; a hand-rolled dialog
  has to add `inert` itself or keyboard users tab straight out behind it. The exit needs the
  `overlay` transition described above or it will simply vanish. Scroll behind must be locked
  and the position restored on close.
- **References** — https://blog.logrocket.com/animating-dialog-popover-elements-css-starting-style/ ·
  https://www.oidaisdes.org/native-dialog-and-popover.en/

---

### Top-layer exit (`overlay.top-layer-exit`)

- **One line** — the technique that makes a closing overlay's animation actually visible.
- **What the reader sees** — The difference between an overlay that fades gracefully away and
  one that simply blinks out of existence. With it, a dismissed dialog shrinks and fades over
  its full exit duration, staying crisply above the page the whole time. Without it, the panel
  disappears instantly, or worse, appears to fall *behind* the header for the last hundred
  milliseconds before vanishing — a glitch that looks like a rendering bug and is in fact a
  specification detail.
- **Mechanism** — transition `display` **and** `overlay`, both with
  `transition-behavior: allow-discrete`, so the element remains rendered in the top layer until
  the exit animation completes.
- **Stack** — CSS. It applies to any `<dialog>` or popover.
- **Params** — none of its own; it governs whatever exit you have authored.
- **Use when** — every animated top-layer element without exception.
- **Don't use when** — never. The absence of this is a defect, not a choice.
- **Reduced motion** — irrelevant when there is no exit animation to preserve, but harmless.
- **Performance** — free.
- **Gotchas** — `allow-discrete` must come *after* the transition it modifies in the shorthand,
  or it is ignored. This is the single most common reason a popover animates in beautifully and
  disappears instantly on close, and the symptom looks nothing like the cause.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/overlay ·
  https://developer.chrome.com/blog/entry-exit-animations

---

### Backdrop scrim (`overlay.backdrop`)

- **One line** — the page behind darkens to demote itself.
- **What the reader sees** — When a dialog opens, everything behind it dims to perhaps half
  brightness over a couple of hundred milliseconds, sometimes with a slight blur. The content is
  still legible enough to give context but plainly no longer the subject. It is doing real work:
  it separates the layers visually, it signals that the page is not currently interactive, and
  it gives an obvious target for dismissal.
- **Mechanism** — an opacity transition on `::backdrop` or a scrim element; blur via
  `backdrop-filter`.
- **Stack** — `<dialog>`'s native `::backdrop`, which needs the same `@starting-style` and
  `allow-discrete` treatment as the dialog itself.
- **Params** — opacity (0.4–0.6); fade (200–300ms, ending after the panel); blur (0–8px — more
  is expensive and rarely better).
- **Use when** — modal dialogs, drawers, anything requiring a decision.
- **Don't use when** — the overlay is non-modal. A scrim over a tooltip or a dropdown is a
  category error: it says "deal with me" about something that does not need dealing with.
- **Reduced motion** — appears instantly at final opacity.
- **Performance** — `backdrop-filter: blur()` over a full viewport is genuinely expensive,
  particularly on mobile. Use a plain dim unless the blur earns itself.
- **Gotchas** — clicking the scrim should dismiss non-critical dialogs and should *not* dismiss
  destructive confirmations. The scrim must not be the only dismissal route — Escape and a
  visible close control are both required.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/::backdrop

---

### Anchored popover (`overlay.popover-anchored`)

- **One line** — a panel appears attached to the control that opened it, wherever that is.
- **What the reader sees** — Click a control and a small panel appears immediately beside it —
  below if there is room, above if there is not, nudged sideways to stay on screen — with a
  small pointer aimed at the control. It scales in from the edge nearest its anchor, so the
  relationship between the panel and the thing it belongs to is unambiguous. Click anywhere
  else and it is gone.
- **Mechanism** — the Popover API for top-layer placement and light dismiss; CSS anchor
  positioning to attach it to the trigger and flip it when space runs out; a scale-and-fade from
  the anchored edge.
- **Stack** — `popover` attribute plus anchor positioning, with `@starting-style` for the entry.
  This replaces the positioning libraries that used to be mandatory for this pattern.
- **Params** — duration (120–160ms); transform-origin matching the anchor edge; offset from the
  trigger (4–8px).
- **Use when** — dropdowns, filter panels, date pickers, any panel belonging to a control.
- **Don't use when** — the content demands full attention; then it is a dialog.
- **Reduced motion** — appears instantly.
- **Performance** — free, and cheaper than the JavaScript positioning it replaces, which
  recalculated on every scroll and resize.
- **Gotchas** — light dismiss handles outside clicks and Escape, but **do not trap focus** in a
  popover — Tab should move on. The transform-origin must follow the flip: a panel that opened
  upward but scales from its top edge looks detached from its anchor.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/Popover_API/Using ·
  https://frontendmasters.com/blog/menus-toasts-and-more/

---

### Tooltip (`overlay.tooltip`)

- **One line** — a small label appears after you rest on something.
- **What the reader sees** — Hover a control and nothing happens for a moment. Rest there and a
  small dark label fades in beside it, having waited long enough to be sure you meant to look.
  Move away and it fades out faster than it arrived. Sweep across a toolbar of ten icons and you
  see no tooltips at all, because none of them was rested on — which is the entire craft of this
  pattern.
- **Mechanism** — a short delay before showing, a shorter one before hiding, with a fade and a
  small offset movement.
- **Stack** — a popover for the top layer and anchor positioning for placement; the timing is
  the substance.
- **Params** — open delay (400–700ms); close delay (~100ms, but longer between adjacent
  tooltips); fade (100–150ms); offset (4–8px of travel toward the anchor).
- **Use when** — icon-only controls, truncated text, supplementary detail.
- **Don't use when** — the content is essential. A tooltip is unavailable on touch and easily
  missed; essential information belongs in the interface.
- **Reduced motion** — appears without the offset movement; keep the delay, which is timing
  rather than motion.
- **Performance** — trivial; use one shared tooltip element rather than one per control.
- **Gotchas** — WCAG requires hover-triggered content to be **dismissible without moving the
  pointer, hoverable** (so you can move onto it without it vanishing), and **persistent** until
  dismissed or invalid. A tooltip that disappears when you try to read it fails all three. It
  must also appear on keyboard focus, not just hover.
- **References** — https://www.w3.org/WAI/WCAG22/Understanding/content-on-hover-or-focus.html

---

### Bottom sheet (`overlay.bottom-sheet`)

- **One line** — a panel rises from the bottom edge on a phone.
- **What the reader sees** — A panel slides up from the bottom of the screen, coming to rest
  part-way up with a grab handle at the top and the page dimmed behind it. It follows your
  finger if you drag it, settling to the nearest resting height on release, and pulling it far
  enough down dismisses it entirely. It is the dominant mobile overlay pattern because it is
  reachable — it appears at the end of the screen where the thumb already is.
- **Mechanism** — a `translateY` entry with a spring settle, plus draggable detents (see
  `gesture.sheet-detents` for the drag behaviour in detail).
- **Stack** — a dialog or popover for semantics, with a drag layer over it.
- **Params** — entry (280–350ms, ease-out); detent positions; dismiss threshold; spring on
  settle.
- **Use when** — mobile detail panels, filters, action lists, share menus.
- **Don't use when** — on desktop, where a sheet anchored to the bottom edge of a wide window is
  simply a strangely-placed dialog.
- **Reduced motion** — appears in place without sliding; dragging still works.
- **Performance** — one transform; trivial.
- **Gotchas** — the scroll-versus-drag conflict is the hard part: inner content scrolls, the
  sheet drags, and the rule that works is that the sheet only moves when the content is already
  at the top and the drag is downward. Respect the safe-area inset at the bottom or controls sit
  under the home indicator.
- **References** — https://frontendmasters.com/blog/menus-toasts-and-more/

---

### Context menu (`overlay.context-menu`)

- **One line** — a menu appears exactly where you invoked it.
- **What the reader sees** — Right-click, or long-press, and a compact list of actions appears
  with its corner at the pointer, scaling up from that exact point over about 120 milliseconds
  so it appears to originate from where you clicked rather than to arrive from elsewhere. If you
  clicked near an edge it flips to stay on screen. Any click elsewhere dismisses it.
- **Mechanism** — a popover positioned at the pointer coordinates with a matching
  transform-origin, flipped when it would overflow the viewport.
- **Stack** — the Popover API for top layer and light dismiss; positioning from the event
  coordinates.
- **Params** — duration (100–140ms — this is a direct response); origin (the click point);
  flip margins.
- **Use when** — file managers, editors, canvases, data tables with per-row actions.
- **Don't use when** — it is the only route to the actions. Right-click is undiscoverable and
  unavailable on touch without a long-press convention.
- **Reduced motion** — appears instantly.
- **Performance** — trivial.
- **Gotchas** — the transform-origin must match the invocation point on both axes, including
  after a flip, or the menu appears to leap. Keyboard users need an equivalent — the context
  menu key, or a visible affordance. Long-press on touch must not fight text selection.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/Popover_API/Using

---

### Snackbar with action (`overlay.snackbar`)

- **One line** — a brief message with an undo, anchored to the bottom of the screen.
- **What the reader sees** — Delete something and a dark bar slides up from the bottom edge:
  "Message archived" with an **Undo** beside it. It waits several seconds, then slides away.
  Hovering or focusing it stops the countdown. It differs from a toast (`micro.toast`) in one
  important way — it carries an action, which means its duration is not merely a reading
  allowance but a decision window.
- **Mechanism** — a `translateY` entry from below with a fade, a hold governed by a pausable
  timer, and a faster exit.
- **Stack** — a non-modal popover so it never steals focus; the timer logic is the substance.
- **Params** — entry (200–250ms); hold (5–8s when an action is offered — longer than a plain
  toast); exit (150–200ms).
- **Use when** — reversible destructive actions, background operations completing.
- **Don't use when** — the action is critical. If missing the window causes real loss, ask
  first instead of offering undo.
- **Reduced motion** — appears without sliding.
- **Performance** — trivial.
- **Gotchas** — pause the timer on hover **and** focus, and never auto-dismiss while it holds
  focus. The action must also be reachable by keyboard without hunting — and if the undo window
  expires unnoticed, the user has silently lost the choice, which is why generous timing matters
  here more than anywhere else in this file.
- **References** — https://frontendmasters.com/blog/menus-toasts-and-more/

---

### Destructive confirmation (`overlay.confirm-destructive`)

- **One line** — the dialog that deliberately does not make it easy.
- **What the reader sees** — A dialog that arrives slightly more slowly and squarely than a
  normal one, with no scale flourish. The destructive action is not pre-focused; the safe option
  is. Clicking the backdrop does nothing. If the consequence is severe, the confirm button stays
  disabled until you type the name of the thing being deleted, and only then does it become
  active — a small colour transition that is the only motion in the whole component.
- **Mechanism** — a plain fade with minimal transform, focus deliberately placed on the safe
  action, and backdrop dismissal disabled.
- **Stack** — `<dialog>` with `showModal()`; the restraint is the design.
- **Params** — duration (200ms); no scale, or a very small one; no bounce.
- **Use when** — deletion, cancellation, anything irreversible.
- **Don't use when** — the action is reversible. Offer an undo snackbar instead — it is far
  less friction for a far better outcome.
- **Reduced motion** — instant.
- **Performance** — trivial.
- **Gotchas** — never autofocus the destructive button; a user pressing Enter out of habit
  should not destroy anything. Do not allow light dismiss, because an accidental outside click
  should not cancel a decision the user was making. Playful motion here is actively wrong — the
  tone of the animation is part of the warning.
- **References** — https://www.oidaisdes.org/native-dialog-and-popover.en/

---

### Consent banner (`overlay.consent-banner`)

- **One line** — the compliance panel that appears before anything else.
- **What the reader sees** — A panel slides up from the bottom or fades in over the page,
  offering choices about data. It is the first thing many visitors see, it appears while the
  page is still settling, and it is almost universally resented. The motion decision that
  matters most is restraint: a short fade that does not compete with the page load, and an exit
  that is immediate once a choice is made.
- **Mechanism** — a fade or short slide, deferred until after the page's own first paint.
- **Stack** — a non-modal region; blocking the entire page for a cookie banner is a pattern to
  avoid where the law permits.
- **Params** — entry delay (until after first paint); entry (200–300ms); exit (fast, 150ms).
- **Use when** — legally required consent.
- **Don't use when** — you can avoid needing it. This is the one entry in the catalogue whose
  best implementation is fewer trackers.
- **Reduced motion** — appears without sliding.
- **Performance** — it must not delay LCP or shift the layout. Overlay it rather than inserting
  it into flow, and never let it be the largest contentful paint element.
- **Gotchas** — reject must be as easy as accept, both visually and in click count; motion that
  makes accept prominent and reject recessive is a dark pattern with regulatory consequences.
  Focus must move into it if it is modal, and it must be dismissible by keyboard.
- **References** — https://www.w3.org/WAI/WCAG22/Understanding/content-on-hover-or-focus.html

---

### Nested overlay stack (`overlay.nested-stack`)

- **One line** — an overlay opened from another overlay, without losing the first.
- **What the reader sees** — A dialog is open; you press a control inside it and a second panel
  appears above it, while the first recedes slightly — scaling down a fraction and dimming — so
  the depth order is visible. Dismiss the second and the first returns to full presence. You
  always know how many layers deep you are and what pressing Escape will do, which is the
  failure mode this treatment exists to prevent.
- **Mechanism** — each layer scaled and dimmed as another opens above it; the top layer's
  natural stacking handles ordering.
- **Stack** — nested `<dialog>` elements or popovers; the top layer stacks them in open order
  automatically, which is a substantial part of why the platform version is worth using.
- **Params** — recede scale (0.96–0.98); dim (0.5–0.7 opacity); transition matching the incoming
  panel.
- **Use when** — a picker inside a form inside a dialog; genuinely unavoidable nesting.
- **Don't use when** — you can avoid the second layer. Three deep is a sign the flow needs
  rethinking, not better motion.
- **Reduced motion** — layers change state without transitions.
- **Performance** — each layer is another composited panel; keep the depth small.
- **Gotchas** — Escape must close **only the topmost** layer, and focus must return to the layer
  beneath rather than to the page. Scroll locking must not be released until the last layer
  closes — a very common bug that unlocks the page while a dialog is still open.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/Popover_API/Using

---

### Focus return (`overlay.focus-return`)

- **One line** — closing an overlay puts you back exactly where you were.
- **What the reader sees** — For a mouse user, nothing at all. For a keyboard user, everything:
  you press a button, deal with the dialog, close it, and the focus ring is back on the button
  you pressed — not at the top of the page, not lost on `<body>`. Continue tabbing and you carry
  on from where you left off. It is not an animation, and it is included because it is the
  single most-skipped requirement of every entry in this file.
- **Mechanism** — record the active element before opening; restore focus to it on close, after
  the exit animation has completed.
- **Stack** — `<dialog>` does this natively in modern browsers; hand-rolled overlays must do it
  explicitly.
- **Params** — restore timing (after the exit transition, so focus does not land on something
  mid-animation).
- **Use when** — every overlay in this file, without exception.
- **Don't use when** — never.
- **Reduced motion** — unaffected.
- **Performance** — free.
- **Gotchas** — if the trigger no longer exists — the row was deleted, the item was archived —
  focus must go somewhere sensible and be announced, not vanish to the document body. Restoring
  focus before the exit animation ends can scroll the page to an element still moving.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/showModal

---

## Family notes

**Use the platform.** `<dialog>` and the Popover API give the top layer, `::backdrop`, light
dismiss, Escape handling, focus containment and stacking order — all of it hard to reimplement
and all of it commonly reimplemented badly.

**Exit animation needs three things**: `@starting-style` for entry, `allow-discrete` on
`display`, and a transition on `overlay`. Miss the third and your overlay is clipped or vanishes
on close, and the symptom will not point you at the cause.

**Modal traps, non-modal does not.** Dialogs contain focus because there is nothing else to
reach. Popovers, tooltips and menus must let Tab move on.

**Overlay motion is fast.** 120–250ms across this whole family. An overlay is a response to a
click, not a scene change.

**Tone is part of the message.** A springy, playful entrance on a delete confirmation actively
undermines it. The most restrained animation in this file is on the most consequential dialog,
and that is deliberate.
