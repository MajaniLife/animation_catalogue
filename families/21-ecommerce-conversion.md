# E-commerce & conversion motion

Motion on the surfaces where someone is deciding whether to spend money. Adding to a cart,
choosing a variant, watching a price change, completing a purchase.

This is the only family in the catalogue where the animation has a **commercial incentive
attached**, and that changes how it has to be assessed. Everywhere else the question is whether
motion helps the reader. Here there is a second party with an interest in the outcome, and some
of the best-known techniques in this family exist to pressure rather than to inform.

**The regulatory position, stated plainly, because it now constrains design.** Fake countdown
timers and personalised "only two left" messages are named examples of deceptive patterns. The
EU's **Digital Fairness Act** is expected to be proposed in Q4 2026 and explicitly targets dark
patterns, addictive design, urgency and scarcity claims, confirm-shaming, and items sneaked into
baskets. In the US, the FTC's dedicated dark-patterns rule was finalised in 2024 and **vacated
on procedural grounds in 2025**, but enforcement continues under Section 5 of the FTC Act
against unfair and deceptive practices. So: the specific rule is in flux, the underlying
prohibition is not.

The practical test used throughout this file: **is the motion reporting something true, or
manufacturing a feeling?** A countdown to a genuine deadline informs. A countdown that resets
when you reload manufactures. Both animate identically, and only one of them is defensible.

---

### Add to cart (`commerce.add-to-cart`)

- **One line** — the product visibly travels to the cart.
- **What the reader sees** — Press Add and a small copy of the product image lifts out of the
  page, arcs up toward the cart icon in the header, shrinking as it goes, and disappears into it
  — at which point the cart badge increments with a small bounce. It takes about half a second.
  The point is not decoration: it answers "did that work, and where did it go" in a single
  gesture, which is why carts that merely increment silently generate so many duplicate adds.
- **Mechanism** — a cloned element animated along a curved path (see `svg.motion-path`) from the
  product's position to the cart's, with scale and fade, then a badge update.
- **Stack** — a clone plus a motion path or a two-axis transform; the badge is
  `micro.count-change`.
- **Params** — duration (400–600ms); arc height; end scale (0.2–0.4); badge bounce (150ms).
- **Use when** — the cart is visible on screen and persistent across the site.
- **Don't use when** — adding navigates straight to the cart, or the cart control is off screen
  — an item flying to nowhere is worse than no animation.
- **Reduced motion** — no flight; the badge increments and a confirmation appears.
- **Performance** — one cloned element, transform-only; remove the clone on completion or they
  accumulate.
- **Gotchas** — the button must confirm immediately and locally, before the network resolves
  (see `micro.optimistic`); a flight animation waiting on a server response defeats its own
  purpose. Announce the addition in a live region — the flight is silent. Rapid repeated adds
  need to queue or overlap gracefully rather than stacking clones.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/offset-path

---

### Variant swap (`commerce.variant-swap`)

- **One line** — choosing a colour or size updates the imagery in place.
- **What the reader sees** — Tap a colour swatch and the swatch itself gains a ring, while the
  product image crossfades to the same shot in the new colour — same angle, same crop, same
  framing, so only the colour appears to change. The price and stock line update in place
  beneath it. Nothing reflows and nothing jumps, so you can flick through six colours quickly
  and compare them, which is the entire job.
- **Mechanism** — a crossfade between pre-loaded variant images plus in-place text updates; the
  swatch selection state transitions separately.
- **Stack** — CSS transitions; the discipline is in the photography and the preloading.
- **Params** — crossfade (150–250ms); swatch selection (100ms); preload adjacent variants on
  hover intent.
- **Use when** — products with visual variants: colour, finish, material.
- **Don't use when** — the variants are not photographed identically. A crossfade between
  different angles reads as an error.
- **Reduced motion** — images swap instantly.
- **Performance** — preload variant images on hover or first interaction, not all on page load.
  A twelve-colour product should not ship twelve hero images up front.
- **Gotchas** — the selected state must not rely on colour alone — a ring or checkmark, since
  swatches are literally colour samples and a colour-blind user cannot see which is chosen.
  Announce the change, and update the URL so a variant can be shared.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/transition

---

### Price update (`commerce.price-update`)

- **One line** — the price changes visibly when your selection changes it.
- **What the reader sees** — Choose a larger size and the price ticks upward — the old figure
  sliding up and out while the new one rises in beneath it, over about two hundred milliseconds,
  the row never changing width. Add an option and it happens again. Because the number moves
  rather than being replaced, you register *that it changed* even if you were looking at the
  option list, which matters more here than anywhere else on the page.
- **Mechanism** — a clipped digit swap (`micro.count-change`) with tabular figures; upward for
  increases, downward for decreases.
- **Stack** — CSS transitions on a clipped container.
- **Params** — duration (180–250ms); direction encoding the sign of the change; brief colour
  emphasis on increase.
- **Use when** — configurable products, quantity changes, options with price impact.
- **Don't use when** — the price is static. A price that animates on page load looks like it is
  being negotiated.
- **Reduced motion** — the number changes without sliding.
- **Performance** — trivial; tabular figures prevent the row twitching.
- **Gotchas** — never animate the price *downward* to make an increase look softer, and never
  animate slowly enough that someone can begin checkout mid-transition against the old figure.
  The change must be announced — a price that silently changed is the most consequential silent
  update in this file.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/font-variant-numeric

---

### Quantity stepper (`commerce.quantity-stepper`)

- **One line** — plus and minus controls with the number responding between them.
- **What the reader sees** — Press plus and the number ticks up with a small upward slide,
  while the button dips under your press. Hold it down and it repeats, accelerating gently.
  Reach the stock limit and the plus button dims and a short shake or message explains why.
  Reach zero and the minus becomes a remove control. Each state change is small and immediate,
  and together they make a two-button control feel like a physical dial.
- **Mechanism** — a press response on the buttons (`micro.press`), a digit swap on the value,
  and a disabled-state transition at the limits.
- **Stack** — CSS transitions plus a repeat timer for press-and-hold.
- **Params** — press (60–80ms); digit swap (150ms); repeat delay (500ms) then interval
  (100–150ms, accelerating).
- **Use when** — carts, order forms, anything with an adjustable count.
- **Don't use when** — quantities are typically large; provide a text input instead of asking
  someone to press plus forty times.
- **Reduced motion** — the number changes without sliding.
- **Performance** — trivial; debounce the network update, not the visual one.
- **Gotchas** — update the displayed number optimistically and reconcile the subtotal after;
  making every press wait for a server round trip makes the control feel broken. The limit
  message must be announced, not just shown, and the input must remain typeable.
- **References** — https://www.w3.org/WAI/ARIA/apg/patterns/spinbutton/

---

### Cart drawer (`commerce.cart-drawer`)

- **One line** — the cart slides in without leaving the page you were browsing.
- **What the reader sees** — Add something, and a panel slides in from the right showing the
  cart contents with the new item highlighted briefly at the top. The page behind dims but stays
  visible. Close it and you are exactly where you were, still browsing. It removes the
  navigate-to-cart-and-back journey that otherwise interrupts every addition, which is the
  single biggest reason it exists.
- **Mechanism** — a drawer transition (`nav.drawer`) plus a brief highlight on the newly added
  row and a subtotal update.
- **Stack** — a dialog or drawer component; the item highlight is a short background transition.
- **Params** — drawer slide (280–350ms); new-item highlight (fade over 1–1.5s); subtotal update
  (`commerce.price-update`).
- **Use when** — catalogues where people add several items across a browsing session.
- **Don't use when** — every add opens it and cannot be dismissed easily; then it is an
  interruption with a subtotal attached.
- **Reduced motion** — drawer appears without sliding.
- **Performance** — one panel; trivial.
- **Gotchas** — focus moves into the drawer and returns to the add button on close, or a
  keyboard user is stranded. Auto-opening on every add becomes hostile fast — consider opening
  only on the first add of a session. Scroll lock must restore position exactly.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/showModal

---

### Filter result update (`commerce.filter-update`)

- **One line** — applying a filter rearranges the grid rather than reloading it.
- **What the reader sees** — Tick a filter and the product grid responds: non-matching items
  fade and shrink away, the survivors glide into a tighter arrangement, and the result count
  above ticks down to the new number. Products that survive are never re-created — you watch
  them move — so if you were considering one, you can follow it. The count changing is what
  confirms the filter did something even when the visible page looks similar.
- **Mechanism** — a FLIP reflow across surviving items (`layout.flip-filter`) plus a counter
  update.
- **Stack** — a layout-animation library or View Transitions.
- **Params** — move (0.35–0.5s); exit shorter than the move; count update (200ms).
- **Use when** — catalogue filtering that resolves client-side or fast enough to feel immediate.
- **Don't use when** — filtering triggers a full page load; then animate nothing and show a
  loading state instead.
- **Reduced motion** — the grid re-renders instantly.
- **Performance** — the animation must not delay interactivity; a user ticking three filters in
  quick succession should not queue three animations.
- **Gotchas** — announce the new result count in a live region; a silently changed grid tells a
  screen reader user nothing. If a filter yields nothing, the empty state must be reached
  gracefully rather than leaving a blank area where the grid was.
- **References** — https://css-tricks.com/animating-layouts-with-the-flip-technique/

---

### Sticky buy bar (`commerce.sticky-buy-bar`)

- **One line** — the purchase control follows you down a long product page.
- **What the reader sees** — Scroll past the main Add to Cart button and a slim bar appears at
  the bottom of the screen carrying the product name, price and a compact Add control. It slides
  up when the original button leaves view and slides away when you scroll back to it, so there is
  never a moment with two identical controls competing. On a long page it means the decision is
  always one tap away.
- **Mechanism** — an IntersectionObserver on the primary button toggling a `translateY` on the
  bar.
- **Stack** — IntersectionObserver plus a CSS transition.
- **Params** — slide (250–300ms); trigger (when the main button is fully out of view); safe-area
  padding at the bottom.
- **Use when** — long product pages, mobile especially.
- **Don't use when** — the page is short enough that the real button is nearly always visible.
- **Reduced motion** — the bar appears without sliding; it should still appear.
- **Performance** — one observer, one transform; trivial.
- **Gotchas** — it must not cover content permanently on small screens, and must respect the
  device safe area. Do not duplicate the button in the accessibility tree without care — two
  identically-labelled Add controls is confusing to navigate by heading or button list.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API

---

### Payment processing (`commerce.payment-processing`)

- **One line** — the pay button becomes a progress state and then an outcome.
- **What the reader sees** — Press Pay and the button label is replaced by a spinner in place;
  the button keeps its exact size and becomes non-interactive. Nothing else on the page changes
  — no navigation, no overlay — so it is obvious that this specific action is in progress. On
  success it becomes a tick and the page moves to confirmation. On failure it returns to its
  resting state with an error above the form, and the form is still filled in.
- **Mechanism** — a state machine on the button (`form.submit-resolve`) with a fixed width and
  a delayed spinner.
- **Stack** — CSS transitions plus payment state; the discipline is in the failure path.
- **Params** — spinner delay (~250ms); success hold before navigation (~800ms); failure return
  (immediate).
- **Use when** — every payment submission.
- **Don't use when** — the payment provider takes over the page; then the handoff is the
  feedback.
- **Reduced motion** — a non-spinning busy state; the outcome states still apply.
- **Performance** — trivial. The perceived duration is entirely the provider's.
- **Gotchas** — this is the highest-stakes button in any interface. It must be impossible to
  double-submit, the busy state must be announced with `aria-busy`, and a failure must **never**
  clear the form. Do not add an artificial success delay for effect — every millisecond here is
  someone waiting to find out whether they have been charged.
- **References** — https://rangle.io/blog/everything-you-need-to-know-about-designing-accessible-forms

---

### Order confirmation (`commerce.order-confirm`)

- **One line** — the moment the purchase is confirmed, marked deliberately.
- **What the reader sees** — The confirmation page arrives and a tick draws itself inside a
  circle, the order number fading in beneath it, then the details below in quick succession over
  about a second. It is the one place in a commerce flow where a slightly generous animation is
  appropriate: the customer has just spent money and is looking for reassurance that it worked,
  and a page that simply appears feels abrupt in a way that undermines confidence.
- **Mechanism** — a stroke-draw tick (`micro.success-check`) plus a short staggered entrance on
  the order details.
- **Stack** — SVG dash-offset plus a modest entrance sequence.
- **Params** — tick draw (400–500ms); detail stagger (0.08s); total under 1.2s.
- **Use when** — order confirmation, subscription activation, booking completion.
- **Don't use when** — it delays the order number appearing. The reference is the reassurance;
  the animation is around it, never in front of it.
- **Reduced motion** — everything present immediately.
- **Performance** — trivial.
- **Gotchas** — the order number and confirmation must be in the DOM and announced immediately,
  not revealed at the end of an animation. Do not use confetti here by default — a large
  purchase can be a stressful commitment, and celebration is a tonal assumption about someone
  else's decision.
- **References** — https://www.justinmind.com/web-design/micro-interactions

---

### Wishlist toggle (`commerce.wishlist-toggle`)

- **One line** — a heart or bookmark fills with a small flourish.
- **What the reader sees** — Tap the outline heart and it fills, scaling up briefly past its
  final size and settling back, sometimes with a tiny burst of particles. It takes under three
  hundred milliseconds and it is the most purely pleasurable micro-interaction in commerce — a
  small reward for a small, reversible, low-stakes action. Tap again and it empties, quietly and
  without the flourish, because undoing does not warrant celebration.
- **Mechanism** — an icon state change with a scale overshoot on activation only
  (`micro.optimistic` for the state handling).
- **Stack** — CSS transitions, or a morph between outline and filled paths.
- **Params** — overshoot (scale 1.2–1.3); duration (250–300ms); the un-toggle plain and faster.
- **Use when** — saving, favouriting, bookmarking — reversible and inconsequential.
- **Don't use when** — the action has a cost or a side effect, such as notifying a seller.
- **Reduced motion** — the icon changes state without the overshoot.
- **Performance** — trivial; particles should be capped and cleaned up.
- **Gotchas** — the control needs `aria-pressed` and an accessible name that states the item,
  not just "favourite". The asymmetry is deliberate — celebrating the removal makes the toggle
  feel indifferent to which state you chose.
- **References** — https://bricxlabs.com/blogs/micro-interactions-2025-examples

---

### Countdown & scarcity (`commerce.urgency-timer`)

- **One line** — a timer or stock counter creating time pressure. **Handle with care.**
- **What the reader sees** — A clock ticking down beside the buy button, or a line reading "only
  2 left in stock", sometimes with the number decrementing while you watch. The motion is
  trivial — digits changing — and the effect is entirely psychological: it converts deliberation
  into haste. Where the deadline is real (a sale that genuinely ends, stock that is genuinely
  low), it is useful information delivered clearly. Where it is not, it is a deceptive pattern
  that regulators name specifically.
- **Mechanism** — a ticking numeric display, or a stock figure updated from real inventory.
- **Stack** — a timer; the honesty is entirely in the data behind it.
- **Params** — update interval (1s for timers); precision (do not show seconds for a deadline
  days away — false precision is itself a pressure technique).
- **Use when** — the deadline or stock level is **real, verifiable, and identical for every
  visitor**.
- **Don't use when** — the timer resets on reload, the count is personalised per visitor, or
  the "deadline" is generated rather than genuine. Fake countdowns and arbitrary "only two left"
  messages are named examples of deceptive practice: the EU's Digital Fairness Act (proposal
  expected Q4 2026) explicitly targets urgency and scarcity claims, and the FTC continues to
  enforce against them under Section 5 despite its dedicated rule being vacated in 2025.
- **Reduced motion** — show the deadline as static text rather than a live ticking clock.
- **Performance** — trivial, though a per-second interval on many list items is wasteful.
- **Gotchas** — an expiring timer must actually change something when it expires; a countdown
  that hits zero and leaves the offer unchanged is self-evidently a fiction. Announce time
  pressure accessibly, and give enough time — WCAG has requirements about timing that apply to
  any purchase flow with a limit.
- **References** — https://pandectes.io/blog/dark-patterns-in-2026-what-the-ftcs-new-rules-mean/ ·
  https://digitalfairnessact.com/ ·
  https://usercentrics.com/knowledge-hub/dark-patterns-and-how-they-affect-consent/

---

### Trust reveal (`commerce.trust-reveal`)

- **One line** — reassurance appears at the moment doubt does.
- **What the reader sees** — As you reach the payment step, a small line fades in beneath the
  card fields: the security wording, the returns window, the delivery estimate. It was not there
  a moment ago and it is not shouting now — it simply arrives where the hesitation happens.
  Nothing is being sold; something is being answered. The timing is what makes it work, and it
  is one of very few conversion techniques in this file that improves the customer's position
  as well as the seller's.
- **Mechanism** — a scroll- or step-triggered fade on contextual reassurance content.
- **Stack** — `scroll.reveal-enter` or a step transition; the content decision is the substance.
- **Params** — fade (200–300ms); triggered by reaching the relevant field or step.
- **Use when** — checkout, high-consideration purchases, first-time customers.
- **Don't use when** — the claims are not true, or the reassurance is generic badge clutter.
- **Reduced motion** — content present without fading.
- **Performance** — trivial.
- **Gotchas** — do not hide material terms behind an animation or a reveal — delivery cost,
  return conditions and total price must be plainly available, not discovered. Late-appearing
  cost information is itself a deceptive pattern, and animating its arrival does not soften that.
- **References** — https://www.arthurcox.com/knowledge/eu-digital-fairness-fitness-check-shines-light-on-deceptive-patterns/

---

## Family notes

**The honesty test.** Is the motion reporting something true, or manufacturing a feeling? A real
deadline animated clearly is information. A generated one animated identically is a deceptive
pattern, and the identical implementation is exactly why this family needs the test.

**Know the regulatory direction.** Urgency and scarcity claims, confirm-shaming, sneaking items
into baskets and ambiguous choice presentation are all named in the EU's Digital Fairness Act
consultation, with a proposal expected Q4 2026; the FTC continues enforcing under Section 5.
Designing these patterns in 2026 is accumulating debt.

**Confirm locally, always.** Add-to-cart, quantity, wishlist and price all need optimistic local
confirmation. Nothing in a purchase flow should feel like it is waiting for permission.

**Silent changes are the danger here.** A price that changes, a filter that empties a grid, a
stock limit reached — each is a fact the customer needs and each is invisible to a screen reader
unless announced. This family carries a financial consequence for getting that wrong.

**Restraint scales with stakes.** A wishlist heart can overshoot and sparkle. A delete
confirmation and a pay button should not. The most consequential controls in this file have the
least motion, deliberately.
