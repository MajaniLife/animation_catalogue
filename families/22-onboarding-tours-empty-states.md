# Onboarding, tours & empty states

Motion in the moments when someone does not yet know how to use the thing. First run, an empty
screen, a new feature, a step they have not taken before.

This family exists to teach, which makes it the one place where an animation's success is
measurable in whether someone can subsequently do something. That is a harder standard than
"does it look considered", and most of what is built here fails it.

**The most important finding is about sequencing, not motion.** Progressive onboarding —
introducing features when they become relevant rather than all at once — is grounded in
cognitive load research: working memory has hard limits, and overloading it blocks learning
outright. A seven-step tour on first launch is not thorough, it is a guarantee that nothing is
retained. Break learning into context-aware steps that arrive at the moment they apply.

**The 2026 position on tours is that they are the last resort, not the first.** Assistive
patterns come first — checklists, inline hints, contextual tooltips, searchable help — and a
tour is used only where those cannot carry the point. Blasting one identical tour at every user
is the pattern being actively moved away from; the direction is adaptive and segmented, shown
only to the people who need it.

**Empty states have become the primary onboarding surface.** The best products treat the empty
screen as the teaching moment rather than a placeholder to apologise in — a human-voiced prompt,
a template or two, and one clear action. The copy discipline that goes with it is worth quoting:
one sentence of value, one of how. Three paragraphs on an empty state is a confession that the
product is not obvious.

---

### Empty state entrance (`onboard.empty-state`)

- **One line** — the screen with nothing in it introduces itself.
- **What the reader sees** — Where a list of documents will eventually be, there is instead a
  small illustration that fades and drifts gently up, a line explaining what goes here, and one
  prominent button. The illustration arrives first, the text a beat later, the action last, over
  about half a second — so the eye is led from *what this is* to *what to do*. It reads as a
  designed destination rather than a failure to load, which is the whole distinction this entry
  is about.
- **Mechanism** — a short staggered entrance (`entrance.fade-rise`) across illustration, copy
  and action.
- **Stack** — CSS transitions; the content decisions matter far more than the animation.
- **Params** — stagger (0.08–0.1s); total under 600ms; illustration drift (8–16px).
- **Use when** — first-run screens, filtered results with no matches, cleared inboxes,
  newly-created workspaces.
- **Don't use when** — the emptiness is an error or a loading state. Those need different
  treatment entirely, and dressing a failure as an empty state hides a problem.
- **Reduced motion** — everything present immediately.
- **Performance** — trivial; keep the illustration light, since it is the first thing rendered
  on a screen with nothing else to do.
- **Gotchas** — one sentence of value and one of how; if you need three paragraphs, the product
  is unclear and the animation is decorating that. Distinguish "you have nothing yet" from "your
  filter matched nothing" — they need different copy and different actions.
- **References** — https://www.72technologies.com/blog/empty-states-as-onboarding-surface ·
  https://www.useronboard.com/onboarding-ux-patterns/empty-states/

---

### Coach mark (`onboard.coach-mark`)

- **One line** — a tooltip points at one control and explains it.
- **What the reader sees** — A small panel appears beside a button, with a pointer aimed
  squarely at it, containing a sentence about what that button does and a Next control. It fades
  and scales in from the direction of the control, so the connection between the explanation and
  the thing being explained is unmistakable. One control, one sentence, one moment.
- **Mechanism** — an anchored popover (`overlay.popover-anchored`) with a directional
  transform-origin.
- **Stack** — the Popover API plus anchor positioning.
- **Params** — appear (150–200ms); origin at the anchored edge; offset (8–12px).
- **Use when** — a single non-obvious control, at the moment it becomes relevant.
- **Don't use when** — chained across six controls on first launch. Coach marks work as part of
  a broader strategy, not as a standalone tour, and a queue of them is the pattern people
  dismiss without reading.
- **Reduced motion** — appears instantly.
- **Performance** — trivial.
- **Gotchas** — focus must move to the coach mark so a screen reader user encounters it at all,
  and Escape must dismiss it permanently rather than re-showing next session. If the anchored
  control is off screen, scroll to it *before* showing the mark — a pointer aimed at nothing is
  worse than silence.
- **References** — https://productfruits.com/blog/how-to-build-perfect-product-tours-in-2026

---

### Spotlight mask (`onboard.spotlight`)

- **One line** — everything dims except the one thing being pointed at.
- **What the reader sees** — The screen darkens, but a hole remains: one control stays at full
  brightness, cut cleanly out of the dimming layer, drawing the eye to it absolutely. Move to
  the next step and the hole travels and resizes to frame the next control, the mask morphing
  rather than blinking, so you can follow where attention is being moved. It is the strongest
  attention-directing device available and the most intrusive.
- **Mechanism** — a full-screen overlay with a `clip-path` or radial mask cut out around the
  target's rectangle, the cutout animating between targets.
- **Stack** — CSS `clip-path` with values computed from the target's bounding box.
- **Params** — cutout transition (300–400ms); padding around the target (8–12px); dim opacity
  (0.5–0.7).
- **Use when** — one genuinely critical control that users demonstrably fail to find.
- **Don't use when** — for more than two or three steps. The screen being taken away repeatedly
  is exhausting, and it prevents exploration — the thing that actually teaches software.
- **Reduced motion** — the cutout jumps between targets without morphing.
- **Performance** — a full-viewport overlay repainting as the cutout animates; keep the shape
  simple.
- **Gotchas** — recompute the cutout on scroll and resize or it drifts off its target. The
  highlighted control must remain genuinely interactive — a spotlight that blocks clicks on the
  thing it is pointing at is a common and infuriating bug. Always provide a visible skip.
- **References** — https://productfruits.com/blog/how-to-build-perfect-product-tours-in-2026

---

### Tour step advance (`onboard.tour-step`)

- **One line** — the tour moves from one point of interest to the next.
- **What the reader sees** — Press Next and the explanation panel slides slightly in the
  direction of travel and crossfades to the new text, while the spotlight or pointer glides to
  the new target and the progress indicator advances — "2 of 4". The movement between targets
  is what makes it a tour rather than a sequence of unrelated popups: you watch attention move,
  so you know where you have been taken.
- **Mechanism** — a coordinated transition of panel content, anchor position and progress state
  on one timeline.
- **Stack** — a tour library or a small state machine; the coordination is the substance.
- **Params** — travel (300–400ms); content crossfade (150ms, starting once movement is
  underway); total per step under 500ms.
- **Use when** — a genuinely sequential workflow that cannot be discovered by use.
- **Don't use when** — the same tour is shown to everyone regardless of role or prior
  familiarity. Segment it, or expect it to be skipped.
- **Reduced motion** — panel and anchor jump between steps.
- **Performance** — trivial.
- **Gotchas** — the step count must be honest and visible up front; "2 of 4" is a contract that
  people will hold you to. Focus follows each step, progress is announced, and **skip must be
  available at every step** — not only at the first.
- **References** — https://supademo.com/blog/onboarding-ux-best-practices

---

### Checklist progress (`onboard.checklist`)

- **One line** — a setup list ticks itself off as you complete things.
- **What the reader sees** — A short list of setup tasks, each with an empty circle. Complete
  one anywhere in the product and its circle fills with a tick that draws itself, the row's text
  softening to indicate completion, and a progress bar above advancing — "3 of 5". The
  satisfaction is real and slightly mechanical, and it works because the progress is a
  by-product of doing actual work rather than of clicking through an explanation.
- **Mechanism** — a tick stroke-draw (`micro.success-check`) plus a row state transition and a
  determinate progress bar.
- **Stack** — CSS transitions plus persisted completion state.
- **Params** — tick draw (300–400ms); row transition (200ms); bar update (300ms).
- **Use when** — multi-step setup: connect a source, invite a colleague, create a first item.
- **Don't use when** — the tasks are trivial or padded to make the list look substantial. People
  notice.
- **Reduced motion** — ticks appear without drawing.
- **Performance** — trivial.
- **Gotchas** — completion must persist across sessions and reflect work done outside the
  checklist, or it is theatre. The list must be dismissible permanently — a checklist that
  cannot be closed becomes a permanent accusation. Announce each completion politely.
- **References** — https://userguiding.com/blog/user-onboarding-best-practices ·
  https://www.saasui.design/blog/saas-onboarding-flows-that-actually-convert-2026

---

### Hotspot pulse (`onboard.hotspot`)

- **One line** — a small pulsing dot marks something new.
- **What the reader sees** — A small dot sits beside a menu item, expanding and fading outward
  in a slow ring every couple of seconds, like a radar ping. It catches peripheral vision
  without demanding anything: nothing is blocked, nothing must be dismissed, and clicking the
  item reveals what is new and stops the pulse. It is the least intrusive way to say "there is
  something here you have not seen".
- **Mechanism** — a scale-and-fade loop on a pseudo-element ring, on a long period.
- **Stack** — CSS keyframes.
- **Params** — period (2–3s); ring scale (1 → 2.5); dot size (6–8px); **stop after a few
  cycles or on first view of the section**.
- **Use when** — new features, unvisited sections, changed settings.
- **Don't use when** — several are on screen at once. Three pulsing dots is a page that appears
  to be malfunctioning.
- **Reduced motion** — a static dot. The marker stays; the pulse stops.
- **Performance** — a permanent animation per hotspot; cap the count and stop them once seen.
- **Gotchas** — it must expire. A dot that pulses forever because the user never clicked it
  becomes furniture and then irritation. The dot alone is meaningless to a screen reader — the
  item needs an accessible "new" label.
- **References** — https://netcorecloud.com/blog/user-onboarding-best-practices/

---

### Sample data preview (`onboard.sample-data`)

- **One line** — the empty screen fills with example content so you can see the point.
- **What the reader sees** — An empty dashboard, and a control offering to show sample data.
  Press it and the charts populate — bars growing, a table filling in — with a clear banner
  marking it as an example. Press again and it drains away just as visibly. Instead of imagining
  what the product looks like when it is working, you are looking at it, which is a far shorter
  path to understanding than any tooltip.
- **Mechanism** — the normal data-entrance animations (`dataviz.bar-grow`,
  `entrance.batch-stagger`) applied to placeholder content, with a persistent sample marker.
- **Stack** — whatever the real rendering uses; the point is that it is the real component.
- **Params** — populate (0.6–0.8s); clear (faster, 0.3s); banner always visible while active.
- **Use when** — dashboards, analytics, anything whose value is invisible until populated.
- **Don't use when** — the sample cannot be told apart from real data. That is a data-integrity
  problem, not a design flourish.
- **Reduced motion** — data appears without entrance animations.
- **Performance** — same as the real components; ship the sample dataset small.
- **Gotchas** — the sample marking must be unmistakable and permanent while active, in text as
  well as visually. Never let sample data enter an export, a report or a share link — that is
  the failure mode that turns a helpful feature into a serious incident.
- **References** — https://www.72technologies.com/blog/empty-states-as-onboarding-surface

---

### Contextual hint (`onboard.contextual-hint`)

- **One line** — a short explanation appears the first time you reach something.
- **What the reader sees** — You open a panel you have never opened before, and a single line of
  helper text fades in beneath its heading, explaining what it is for. Use the panel again
  tomorrow and the line is gone. It never blocks anything, never needs dismissing, and it arrives
  precisely when the question it answers has just formed — which is why this pattern outperforms
  a tour delivered before the question existed.
- **Mechanism** — a fade on first render of a surface, gated by persisted per-user state.
- **Stack** — a fade plus state; the targeting logic is the real work.
- **Params** — fade (200–300ms); shown once or twice, then retired.
- **Use when** — non-obvious surfaces reached organically. This is the assistive pattern the
  2026 guidance says to reach for **before** a tour.
- **Don't use when** — the interface would be clear with better labels. A hint compensating for
  a bad label is a patch on a design problem.
- **Reduced motion** — appears without fading.
- **Performance** — trivial.
- **Gotchas** — persist "seen" state server-side where possible; per-device state means the hint
  reappears on every new browser and stops feeling contextual. Do not steal focus — this is
  supplementary text, not an interruption.
- **References** — https://formbricks.com/blog/user-onboarding-best-practices ·
  https://userpilot.com/blog/saas-ux-design/

---

### Progressive feature reveal (`onboard.progressive-reveal`)

- **One line** — advanced controls appear only once the basics are in use.
- **What the reader sees** — The interface starts simple — three controls where the mature
  product has fifteen. As you use it, more appear, sliding into the toolbar as they become
  relevant: after your third document, the templates control; after inviting someone, the
  permissions control. Each arrival is small and accompanied by a brief hint. The product grows
  to match you rather than presenting its full complexity on day one.
- **Mechanism** — controls entering with a short fade-and-scale as their unlock conditions are
  met, usually paired with `onboard.contextual-hint`.
- **Stack** — feature-state logic plus a modest entrance.
- **Params** — entrance (250–350ms); one reveal at a time; a hint accompanying the first
  appearance.
- **Use when** — feature-rich tools with a genuine beginner-to-expert gradient.
- **Don't use when** — users will be confused by a moving interface, or need to find a control
  they have heard about and cannot see. Hidden functionality is a support burden.
- **Reduced motion** — controls appear without transition.
- **Performance** — trivial.
- **Gotchas** — this is directly grounded in the working-memory constraint that makes
  front-loaded tours fail — but it must be discoverable in reverse: always provide a way to see
  everything for someone who is looking for a specific feature. An interface that rearranges
  itself between sessions is disorienting for anyone relying on spatial memory.
- **References** — https://userguiding.com/blog/user-onboarding-best-practices ·
  https://www.toptal.com/designers/product-design/guide-to-onboarding-ux

---

### Inline demo loop (`onboard.demo-loop`)

- **One line** — a short looping animation shows the feature being used.
- **What the reader sees** — Inside the empty state or hint panel, a small silent clip plays on
  a loop: a cursor selecting text, a menu opening, an item being dragged into place. Three or
  four seconds, then it repeats. It shows the interaction rather than describing it, which for
  anything gestural — dragging, resizing, multi-select — is dramatically more effective than a
  sentence attempting the same job.
- **Mechanism** — a looping video or Lottie animation, muted and inline, ideally paused until in
  view.
- **Stack** — a short encoded clip; Lottie or Rive where the asset must scale or theme.
- **Params** — length (3–5s); loop with a visible pause between cycles; play only when visible.
- **Use when** — gestural or multi-step interactions that prose describes badly.
- **Don't use when** — a static annotated image would do. A loop is continuous motion and
  carries that cost.
- **Reduced motion** — a static representative frame with a play control.
- **Performance** — a looping video is heavy for a hint; keep it tiny, and never autoplay
  several on one screen.
- **Gotchas** — subject to the five-second pause rule; provide a control if it runs long
  alongside other content. It needs a text description too — a silent loop teaches nothing to a
  screen reader user, and gestural instructions are exactly what those users most need in words.
- **References** — https://supademo.com/blog/onboarding-ux-best-practices

---

### Completion moment (`onboard.completion`)

- **One line** — finishing setup is marked, once, and then never again.
- **What the reader sees** — The last checklist item ticks and the panel does something it has
  not done before: the progress bar completes, the list collapses, and a short message
  acknowledges that setup is done — perhaps a brief flourish — before the whole panel fades out
  of the interface permanently. The disappearance is the reward. The product now looks like the
  product rather than like a product being set up.
- **Mechanism** — a terminal state on the checklist, then a collapse-and-fade removal of the
  onboarding surface.
- **Stack** — the checklist components plus a removal transition.
- **Params** — flourish (under 1s); message hold (2–3s); collapse (400ms).
- **Use when** — the end of a genuine multi-step setup.
- **Don't use when** — setup is not actually complete. Declaring victory early is worse than not
  marking it.
- **Reduced motion** — the panel is removed without animation; keep the message.
- **Performance** — trivial.
- **Gotchas** — it must never return. A completed onboarding surface that reappears after an
  update is a small betrayal. Announce completion, and make sure any content the panel occupied
  reflows sensibly once it is gone.
- **References** — https://www.saasui.design/blog/saas-onboarding-flows-that-actually-convert-2026

---

### Skip affordance (`onboard.skip`)

- **One line** — the visible, permanent way out of being taught.
- **What the reader sees** — In the corner of every tour step, coach mark and checklist, a plain
  "Skip" or dismiss control — never hidden, never greyed, never appearing only on the last step.
  Press it and the entire onboarding layer fades away in about 200 milliseconds and does not
  come back. For an experienced user migrating from a competitor, this is the single most
  valuable control in this whole file.
- **Mechanism** — a fade-out of the onboarding layer, plus persisted dismissal state.
- **Stack** — trivial; the discipline is that it exists on every step.
- **Params** — fade (200ms); dismissal persisted per user, not per device.
- **Use when** — every onboarding surface without exception.
- **Don't use when** — never. An unskippable tour is the most reliably resented pattern in this
  family.
- **Reduced motion** — the layer is removed instantly.
- **Performance** — free.
- **Gotchas** — it must be reachable by keyboard as the first or last stop in the tour's focus
  order, and honoured permanently across sessions and devices. Confirm-shaming the skip control
  — "No thanks, I'll struggle" — is a named deceptive pattern, not a clever piece of copy.
- **References** — https://productfruits.com/blog/how-to-build-perfect-product-tours-in-2026

---

## Family notes

**Assistive patterns before tours.** Empty states, checklists, contextual hints and inline
demos first; a spotlight tour only where those genuinely cannot carry the point. The 2026
direction is adaptive and segmented — showing the right thing to the users who need it, not the
same tour to everyone.

**Working memory is the constraint.** Front-loading everything on first launch does not teach
faster, it prevents learning. Introduce each thing at the moment it becomes relevant.

**The empty state is the best onboarding surface you have.** It appears exactly when the user is
ready to do the thing, it blocks nothing, and it costs no extra layer. One sentence of value, one
of how.

**Everything here must expire.** Hints shown once, hotspots that stop pulsing, checklists that
dismiss, tours that never return. Onboarding that does not know when it is finished becomes the
product's most annoying feature.

**Skip is mandatory and permanent.** On every step, keyboard-reachable, persisted per user. And
never shame the person taking it.
