# Playful, reward & delight

Motion whose purpose is emotional rather than informational. Celebrations, streaks, badges,
easter eggs, mascots, and the small flourishes that make software feel like it was made by
people.

This family needs its distinction stated before anything else, because the two things it
contains are routinely confused:

**Delight is not gamification.** Delight is a well-made moment that reinforces trust and gives a
product character — it asks for nothing. Gamification engineers behaviour, and when it is tuned
to maximise retention rather than to serve the person using it, it stops being design and
becomes manipulation. The same confetti burst can be either, depending entirely on what it is
attached to and what it is trying to make you do next.

**The regulatory position hardened considerably in 2026.** Brazil's **Decree 12,880/2026**
implemented one of the world's most ambitious frameworks against addictive design, banning
specific features by name — infinite scroll, autoplay, **time-based rewards**, excessive
notifications, and exploitation of cognitive vulnerabilities. Commentary notes the prohibitions
may reach ordinary-seeming products: language apps with daily streaks, meditation platforms
rewarding consistency, fitness apps with badges. The EU's Digital Fairness Act (proposal
expected Q4 2026) names addictive design in the same breath as dark patterns. Building streaks
and variable rewards in 2026 is a decision with legal exposure attached, not merely a taste
question.

**And the craft critique is just as blunt.** The most quoted line in the current writing about
this family is worth repeating: **if it is an ordinary thing, it does not need to be met with an
extraordinary response.** Celebrating everything celebrates nothing, and it makes the moments
that genuinely deserve it worthless.

---

### Confetti burst (`play.confetti`)

- **One line** — coloured pieces erupt and fall to mark a completion.
- **What the reader sees** — Finish something significant and the screen erupts: dozens of small
  coloured shapes launched upward from a point, spinning as they arc over and fall, drifting
  slightly sideways as they descend and fading out before they reach the bottom. It lasts two or
  three seconds. Used once, at the right moment, it is genuinely joyful and people remember it.
  Used on every completed task, it becomes an obstruction that people learn to wait out.
- **Mechanism** — a particle system on canvas, with per-piece velocity, gravity, rotation and
  drag; pieces removed once off screen.
- **Stack** — canvas for anything above a few dozen pieces; a small library is fine, a physics
  engine is not needed.
- **Params** — count (50–150 — beyond that it is weather); duration (2–3s); gravity and drag;
  origin (the completed element, not the screen centre).
- **Use when** — a genuinely rare, genuinely significant moment: onboarding complete, a year's
  goal met, a first publish.
- **Don't use when** — the action is routine. This is the single most over-applied effect in
  contemporary product design, and the reason it now often reads as insincere.
- **Reduced motion** — no particles. A static message with a celebratory tone carries the
  meaning without the storm.
- **Performance** — a canvas particle system running for three seconds; cap the count, stop the
  loop when the last piece leaves, and never leave the canvas mounted afterwards.
- **Gotchas** — it must be `pointer-events: none` or it blocks the very control the user is
  reaching for next. Do not fire it over content someone is trying to read. And consider the
  context: a large financial commitment or a medical result is not obviously a moment for
  celebration, and assuming it is can land very badly.
- **References** — https://uxdesign.cc/the-over-confetti-ing-of-digital-experiences-af523745db19 ·
  https://www.uxlift.org/articles/why-confetti-celebrations-backfire-and-how-to-make-them-work/ ·
  https://www.shahimim.com/blog/stop-adding-confetti-to-your-product

---

### Streak flame (`play.streak`)

- **One line** — a running count of consecutive days, animated to feel precious. **Handle with care.**
- **What the reader sees** — A number beside a flame icon, and when you complete today's
  activity the number ticks up while the flame flares briefly and settles brighter than before.
  Miss a day and it is gone — reset to zero, often with a deliberately mournful animation. The
  motion is designed to make the number feel like an asset you own, which is exactly why it
  works and exactly why it is the most criticised mechanic in this file.
- **Mechanism** — a counter increment with a scale pulse and a colour or intensity change on the
  icon; the reset is usually a slower, sadder transition.
- **Stack** — CSS transitions plus persisted state; the psychology is the substance, not the code.
- **Params** — increment pulse (300–400ms); flame intensity tied to streak length; reset
  treatment.
- **Use when** — the behaviour genuinely benefits the user and they have chosen to build it, with
  **forgiveness built in**: a grace day, a freeze, a repair.
- **Don't use when** — the streak's only purpose is retention. A streak that breaks once and is
  lost forever is a psychological trap, it is the most copied and most abused mechanic in the
  category, and **time-based rewards are named in Brazil's Decree 12,880/2026** — with
  commentary noting the prohibition may reach exactly this pattern in language, meditation and
  fitness apps.
- **Reduced motion** — the number updates without the flare.
- **Performance** — trivial.
- **Gotchas** — the loss animation is where the ethics live: dramatising a broken streak is
  engineered guilt. If you ship this, ship the forgiveness mechanic in the same release, and do
  not send a notification designed to exploit the fear of losing it.
- **References** — https://www.techpolicy.press/brazil-banned-addictive-design-the-crucial-regulatory-choices-are-still-ahead/ ·
  https://nerdsip.com/blog/gamification-gone-wrong-when-streaks-become-the-point

---

### Badge unlock (`play.badge-unlock`)

- **One line** — an achievement is revealed with a moment of ceremony.
- **What the reader sees** — A card slides up with a badge on it, the badge scaling in from small
  with a brief shine sweeping across its surface, its title fading in beneath. It holds for a
  couple of seconds and can be dismissed. It is a small trophy presentation, and its
  effectiveness depends entirely on whether the achievement means anything — a badge for
  completing your profile is a badge for doing paperwork.
- **Mechanism** — a card entrance with a scale-and-shine on the badge (`text.gradient-sweep`
  applied to an image) and a staggered title.
- **Stack** — CSS transitions; a Lottie or Rive asset where the badge itself is illustrated.
- **Params** — entrance (400–500ms); shine sweep (600ms, once); auto-dismiss (3–4s) with manual
  dismiss available.
- **Use when** — milestones that reflect genuine accomplishment or progress the user cares about.
- **Don't use when** — badges are awarded for compliance rather than achievement. People
  distinguish these instantly, and a worthless badge devalues the real ones.
- **Reduced motion** — the card appears without the scale or shine.
- **Performance** — trivial; do not preload every badge asset for badges not yet earned.
- **Gotchas** — it must be dismissible and must not interrupt an in-progress task — arriving
  mid-typing is the classic failure. Announce it politely rather than assertively; an
  achievement is not an alert.
- **References** — https://ui-patterns.com/blog/Psychology-of-rewards-in-web-design

---

### Task completion flourish (`play.task-complete`)

- **One line** — a small, proportionate acknowledgement of finishing something ordinary.
- **What the reader sees** — Tick a to-do item and the checkbox fills with a tick that draws
  itself, the text softens and strikes through, and the row settles down the list into the
  completed section — about four hundred milliseconds in total. There is no confetti and no
  sound. It feels satisfying in the way closing a drawer properly feels satisfying: the
  proportion is right, and the small scale is what allows it to happen fifty times a day without
  becoming tiresome.
- **Mechanism** — a tick stroke-draw (`micro.success-check`), a text-decoration transition, and a
  FLIP reposition into the completed group.
- **Stack** — CSS transitions plus a layout animation.
- **Params** — tick (250–350ms); text transition (200ms); reposition (300ms).
- **Use when** — routine completions: tasks, items, steps. This is the correct default, and the
  one people underuse in favour of larger celebrations.
- **Don't use when** — completions happen in bulk; then animate the summary, not each row.
- **Reduced motion** — the state changes without drawing or repositioning.
- **Performance** — trivial.
- **Gotchas** — the animation must not delay the state change or block the next interaction —
  someone clearing a list is moving fast. Announce completion once, not with a flourish per row.
- **References** — https://nesrinechanguel.substack.com/p/why-delight-is-completely-different

---

### Easter egg (`play.easter-egg`)

- **One line** — a hidden reaction for someone who went looking.
- **What the reader sees** — Nothing, for almost everyone. But click the logo seven times, or
  type the Konami code, or find the one link in the footer nobody clicks, and something happens:
  the mascot turns around, the page tilts, a hidden message appears. It is a reward for
  curiosity that costs nothing to the people who never find it, which makes it the only effect
  in this file with genuinely no downside — provided it is truly hidden and truly harmless.
- **Mechanism** — an input sequence or unusual interaction detector, triggering any animation
  you like.
- **Stack** — anything; this is the one place in the catalogue where indulgence is the point.
- **Params** — entirely yours; the only constraint is reversibility.
- **Use when** — a personal site, a developer tool, a product with a human voice.
- **Don't use when** — it could be triggered accidentally, or it changes state the user cares
  about.
- **Reduced motion** — respect it. Even an easter egg should not spin the viewport for someone
  who asked it not to.
- **Performance** — irrelevant if it never loads for most visitors; lazy-load the assets.
- **Gotchas** — it must be dismissible and must not break anything. An easter egg that requires
  a page reload to escape is a bug wearing a costume. Keyboard sequences must not conflict with
  assistive-technology shortcuts.
- **References** — https://nesrinechanguel.substack.com/p/why-delight-is-completely-different

---

### Mascot reaction (`play.mascot`)

- **One line** — a character responds to what you are doing.
- **What the reader sees** — A small illustrated character in the corner of a form. Enter a
  password field and it covers its eyes. Make an error and it looks concerned. Finish and it
  waves. Each reaction is a second or so, and the effect is that the interface appears to be
  paying attention to you — which is disarming in a form, the most joyless surface in software,
  and is why this pattern keeps reappearing.
- **Mechanism** — a state machine over a small animated asset, driven by form or app events.
- **Stack** — Rive is the right tool here — state machines are its purpose — or Lottie with
  named segments.
- **Params** — reaction length (0.8–1.5s); idle loop between reactions; return to neutral.
- **Use when** — sign-up flows, empty states, error pages, products with a deliberately human
  voice.
- **Don't use when** — the product is a serious professional tool used all day. A character
  reacting to your spreadsheet is a novelty that wears out in an afternoon.
- **Reduced motion** — a static character in a neutral pose; keep the illustration, drop the
  animation.
- **Performance** — a runtime-animated asset is heavier than it looks; one mascot, lazily
  loaded, never several.
- **Gotchas** — it must be decorative to assistive technology (`aria-hidden`) — nobody needs a
  screen reader describing a blinking otter. It must never be the only feedback for an error,
  and it must be silent by default.
- **References** — https://uxmag.com/articles/designing-for-dependence-when-ux-turns-tools-into-traps

---

### Variable reward (`play.variable-reward`)

- **One line** — an unpredictable payoff, animated to feel like a slot machine. **This is the
  ethical line.**
- **What the reader sees** — Pull to refresh, open a box, spin a wheel — and the animation
  deliberately delays the outcome: a spin that slows, a box that shakes before opening, a brief
  suspense before the reveal. Sometimes the reward is good, sometimes it is nothing, and the
  variability is the point. The motion exists specifically to stretch the moment of uncertainty,
  because that is where the compulsion lives.
- **Mechanism** — a suspense animation of deliberately variable duration, resolving to a
  randomised outcome.
- **Stack** — any; the technique is trivial and the design decision is not.
- **Params** — suspense duration; outcome distribution — both tuned for compulsion by default,
  which is precisely the problem.
- **Use when** — genuinely gambling-adjacent contexts that are regulated as such, or a game
  where the player has opted into chance as the subject.
- **Don't use when** — in a productivity tool, a social app, a learning product or anything used
  by children. Variable-ratio reinforcement is the mechanic that makes slot machines addictive;
  applying it to a to-do app is not gamification, it is exploiting a cognitive vulnerability —
  language that now appears in regulation rather than only in criticism.
- **Reduced motion** — remove the suspense animation entirely; show the outcome immediately.
  Notably, doing so removes most of the compulsion, which tells you what the animation was for.
- **Performance** — trivial.
- **Gotchas** — Brazil's Decree 12,880/2026 bans exploitation of cognitive vulnerabilities and
  time-based rewards outright; the EU's Digital Fairness Act names addictive design. This entry
  exists in the catalogue so that the pattern can be recognised and refused, not so it can be
  implemented well.
- **References** — https://builtformars.com/tooltips/variable-rewards ·
  https://www.techpolicy.press/brazil-banned-addictive-design-the-crucial-regulatory-choices-are-still-ahead/ ·
  https://webinale.com/blog-en/generative-ai-ux-slot-machine-design/

---

### Level up (`play.level-up`)

- **One line** — crossing a threshold is marked with escalation.
- **What the reader sees** — A progress bar fills to its end and, instead of stopping, overflows:
  the bar flashes, the level number ticks up with a scale pulse, the bar empties and refills a
  little way into the next level. It takes about a second and it borrows directly from games,
  where the pattern is well understood and the escalation is earned by genuine effort.
- **Mechanism** — a progress bar completion, a numeric increment with emphasis, and a reset to
  the next level's baseline, sequenced on one timeline.
- **Stack** — CSS transitions plus a small timeline.
- **Params** — fill completion (400ms); level tick (300ms); refill (500ms); total under 1.5s.
- **Use when** — genuine skill or contribution progression: a learning platform, a craft tool, a
  community with real standing.
- **Don't use when** — the levels are arbitrary. Levelling up for logging in is a number
  pretending to be an accomplishment, and people see through it quickly.
- **Reduced motion** — the level changes without the fill sequence.
- **Performance** — trivial.
- **Gotchas** — the sequence must not block the interface; someone who levelled up mid-task
  should not have to wait for a celebration to finish. What the level unlocks needs to be
  discoverable — a number that means nothing is worse than no number.
- **References** — https://dotgg.gg/why-gamification-is-everywhere-in-2026/

---

### Error page personality (`play.error-personality`)

- **One line** — the 404 page is the one place to be funny.
- **What the reader sees** — A page that should be a dead end instead has something on it: an
  illustration that reacts to the cursor, a small animation that plays out, a joke that lands.
  The tone acknowledges that something went wrong without being either apologetic or
  saccharine, and there is always a clear route back. It is the highest-value playful moment in
  most products, because the user's expectation at that moment is *low*, and exceeding it costs
  nothing.
- **Mechanism** — any effect from any family; this is a page with no functional requirements
  beyond an exit route.
- **Stack** — whatever suits; keep it light, since the page loads at a moment of failure.
- **Params** — yours entirely.
- **Use when** — 404s, empty search results, offline states, maintenance pages.
- **Don't use when** — the error cost the user something. A playful animation over "your payment
  failed" or "your draft could not be saved" is tone-deaf and reads as not taking it seriously.
- **Reduced motion** — a static illustration with the same voice in the copy.
- **Performance** — keep it small; an error page that loads slowly compounds the failure.
- **Gotchas** — the way back must be obvious and above the fold. Humour must not obscure what
  went wrong or what to do; the joke sits alongside the information, never instead of it.
- **References** — https://nesrinechanguel.substack.com/p/why-delight-is-completely-different

---

### Idle personality (`play.idle-personality`)

- **One line** — the interface has small habits when nothing is happening.
- **What the reader sees** — Leave the page alone for a while and something small happens: an
  illustrated character blinks, a cursor in a demo drifts, a logo shifts its weight. It is never
  demanding, never repeated often enough to notice a pattern, and it stops the moment you
  interact. What it produces is a sense that the page is *inhabited* rather than static —
  present rather than waiting.
- **Mechanism** — an idle timer triggering short, infrequent, non-looping animations at
  irregular intervals.
- **Stack** — a timer plus small assets; irregular timing is what prevents it reading as a loop.
- **Params** — idle threshold (15–30s); animation length (under 1s); interval between (30–90s,
  randomised).
- **Use when** — a brand site, a personal site, a product with a deliberate character.
- **Don't use when** — anywhere someone is reading or working. Peripheral movement while
  concentrating is disruptive, and this family's charm does not exempt it.
- **Reduced motion** — nothing moves.
- **Performance** — negligible if genuinely infrequent, but it keeps the page from idling
  entirely; stop it when the tab is hidden.
- **Gotchas** — regular intervals make it read as a loop and become irritating within minutes;
  randomise. And "idle" and "reading carefully" look identical to a timer, which is why this
  belongs on brand surfaces rather than in tools.
- **References** — https://uxdesign.cc/the-over-confetti-ing-of-digital-experiences-af523745db19

---

### Haptic pairing (`play.haptic-pairing`)

- **One line** — a physical tap accompanies the visual response.
- **What the reader sees** — Or rather, feels: a toggle flips and a small tick registers in the
  hand at the same instant. A drag reaches a snap point and there is a faint bump. The visual
  and the tactile arrive together, and the interaction gains a solidity that neither alone
  produces — it stops being a picture of a switch and becomes something closer to a switch.
- **Mechanism** — the Vibration API on supporting platforms, fired at the same moment as the
  visual state change; native apps have far richer haptic vocabularies than the web.
- **Stack** — `navigator.vibrate` where available; capability is limited and inconsistent on the
  web.
- **Params** — duration (10–30ms — a tick, never a buzz); fired **with** the visual, not before
  or after.
- **Use when** — toggles, snap points, confirmations on touch devices, where the platform
  supports it well.
- **Don't use when** — the pattern is long or repeated. A buzz is an alarm; a tick is feedback.
- **Reduced motion** — haptics are not motion, but pair the decision with the user's system
  settings, and honour any in-app haptics preference.
- **Performance** — free.
- **Gotchas** — web support is patchy and iOS Safari has historically not supported the
  Vibration API at all, so this must be a genuine enhancement rather than part of the feedback
  contract. Overuse is worse than absence: a device buzzing on every tap gets the whole feature
  switched off at the OS level.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/Vibration_API

---

### Surprise reveal (`play.surprise-reveal`)

- **One line** — something is concealed and then uncovered by the user.
- **What the reader sees** — A panel that must be scratched, swiped or torn to reveal what is
  underneath — the surface eroding under your finger to show the result beneath, with the reveal
  tracking your gesture exactly. Because you are performing the uncovering, the moment of
  discovery belongs to you rather than being presented to you, which is a meaningfully different
  feeling from a card that simply flips itself over.
- **Mechanism** — a mask or canvas erased along the pointer path, revealing the layer beneath;
  auto-completes once enough is cleared.
- **Stack** — canvas compositing for scratch effects; `clip-path` for simpler swipe reveals.
- **Params** — reveal radius; auto-complete threshold (50–70% cleared); the reveal beneath
  pre-rendered so it never lags the gesture.
- **Use when** — a genuine one-off moment: a result, a gift, a personalised summary.
- **Don't use when** — the content is needed quickly, or on a professional surface. It is
  friction chosen for effect, which is only acceptable when the effect is the point.
- **Reduced motion** — reveal the content immediately with a plain fade.
- **Performance** — canvas erasing per pointer move; throttle to animation frames.
- **Gotchas** — **always provide a skip or auto-reveal**; a gesture-gated result is unreachable
  for many users, and if the concealed content matters it must have a non-gestural route. If
  this is dressing up a random outcome, see `play.variable-reward` before shipping it.
- **References** — https://www.appcues.com/blog/variable-rewards

---

## Family notes

**Delight and gamification are different things.** Delight gives a product character and asks
nothing. Gamification engineers behaviour, and tuned for retention rather than for the user it
crosses into manipulation. Know which one you are shipping.

**Proportion is the whole craft.** If it is an ordinary thing, it does not need an extraordinary
response. Celebrating everything celebrates nothing — and it spends the currency you would need
for the moment that actually matters.

**The regulatory ground moved in 2026.** Brazil's Decree 12,880/2026 bans time-based rewards,
infinite scroll, autoplay and exploitation of cognitive vulnerabilities by name, with commentary
suggesting it reaches everyday streak and badge mechanics; the EU's Digital Fairness Act names
addictive design. Variable-ratio rewards are slot-machine mechanics regardless of what they are
attached to.

**Build the forgiveness in the same release as the streak.** Grace days, freezes, repairs. A
mechanic that punishes a single missed day is engineered guilt, not motivation.

**Everything here is optional to the user.** Dismissible, skippable, and silent under
reduced motion. Delight that cannot be declined is not delight.
