# Page & route transitions

Motion across a navigation boundary. The reader clicks a link and arrives somewhere else,
and this family is about what happens in between.

The job is to **hide a discontinuity**. A hard navigation flashes white, drops the scroll
position and rebuilds the page; a transition covers that seam and, done well, makes the
arrival feel like a continuation rather than a restart. The second job is to buy time
honestly: a well-paced transition occupies the moment when the next document is still
loading.

**The landscape changed, and it changed recently.** For a decade, page transitions were the
main argument for building a single-page application at all — you could not animate between
two documents, so you avoided having two documents. That is no longer true. Cross-document
view transitions are supported in Chromium browsers and Safari 18.2+ as of 2026, with
Firefox in progress, and they are opted into **from CSS** with the `@view-transition` at-rule.
(An earlier `<meta>` tag opt-in circulated during development and is obsolete — guidance
still recommending it is out of date.)

Two events make the cross-document case controllable: **`pageswap`** fires on the outgoing
page immediately before the browser snapshots it, and **`pagereveal`** fires on the incoming
page after initialisation but before first paint. Between them you can set names, capture
state, and decide what the transition should be — including deciding not to have one.

**The timing trap that decides whether any of this works.** A 300ms crossfade feels slow if
the new document has not loaded, because you spend it transitioning into a skeleton. Pairing
transitions with the Speculation Rules API to prerender likely destinations is what closes
that gap — the transition then runs between two already-rendered documents and reads as
instant. Without prerendering, a page transition on a slow connection makes a site feel
slower, not faster.

---

### Auto crossfade (`route.crossfade`)

- **One line** — the browser fades the old page into the new one, in two lines of CSS.
- **What the reader sees** — Click a link and the current page dissolves into the next over
  a couple of hundred milliseconds. No white flash, no jolt — the header and footer, being
  visually identical on both pages, appear to simply stay put while the content between them
  changes. It is the least dramatic transition possible and probably the correct default for
  most sites: it removes the flash of navigation without asking the reader to wait for
  choreography.
- **Mechanism** — `@view-transition { navigation: auto; }` in the CSS of both documents; the
  browser snapshots the old and new views and crossfades the roots.
- **Stack** — platform, no library. Chromium and Safari 18.2+; Firefox in progress, where it
  degrades to an ordinary navigation.
- **Params** — duration and easing via the transition pseudo-elements (default is a short
  crossfade); which elements are named.
- **Use when** — any multi-page site. It is nearly free and strictly better than nothing.
- **Don't use when** — destination pages are slow and unprerendered; you will be crossfading
  into a blank shell.
- **Reduced motion** — skip the transition entirely and navigate normally. A crossfade is
  mild, but a full-viewport dissolve on every click is still motion the user did not ask for.
- **Performance** — the browser snapshots the viewport; on a heavy page that snapshot is the
  cost. The animation itself is compositor work.
- **Gotchas** — **both documents must opt in**, or nothing happens and there is no error to
  tell you. Only same-origin navigations qualify. And elements scrolled out of view are not
  in the old page's snapshot at all — plan around what is actually visible.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@view-transition ·
  https://developer.chrome.com/docs/web-platform/view-transitions/cross-document

---

### Shared element across pages (`route.shared-element`)

- **One line** — the thumbnail you clicked becomes the hero image of the page you land on.
- **What the reader sees** — You click a card in a listing and its image lifts out of the
  grid, growing and travelling as the rest of the page falls away, and lands as the large
  image at the top of the article you have opened. The navigation is invisible: there is no
  moment where the old page ends and the new one begins, only an image moving from one
  context to another. This is the effect that makes a multi-page site feel like a native
  application, and until recently it was effectively impossible without a full SPA.
- **Mechanism** — the same `view-transition-name` on the thumbnail in the outgoing document
  and the hero in the incoming one; the browser matches them and interpolates position and
  size. `pageswap` and `pagereveal` are where you assign those names dynamically.
- **Stack** — platform for the modern route; a client-side router plus FLIP where support or
  architecture requires it.
- **Params** — duration (0.4–0.6s); easing; what the surrounding content does (usually a
  plain crossfade so it does not compete).
- **Use when** — listing to detail, gallery to lightbox, product grid to product page.
- **Don't use when** — the two images differ in aspect ratio, or the destination image is
  large and unloaded — the morph will land on an empty box.
- **Reduced motion** — a plain crossfade with no travel.
- **Performance** — one interpolation. Prerendering the destination matters more here than
  anywhere, because the destination image must be ready at the moment of arrival.
- **Gotchas** — the name must be **unique in each document at the moment of the transition**,
  which in a grid means assigning it to the clicked item only, usually in `pageswap`. If the
  clicked thumbnail is scrolled out of view it is not in the snapshot and the morph has
  nothing to start from. Back-navigation should reverse it, which means restoring the name
  on the way back too.
- **References** — https://developer.chrome.com/docs/web-platform/view-transitions/cross-document ·
  https://css-tricks.com/cross-document-view-transitions-part-1/

---

### Curtain wipe (`route.curtain`)

- **One line** — a panel covers the screen, the page changes behind it, the panel leaves.
- **What the reader sees** — Click a link and a solid colour sweeps up over the whole
  viewport, holds for a moment, then sweeps away to reveal an entirely different page,
  already in place. You never see the old page leave or the new one build — the cover does
  all the work. It is the most theatrical transition available and it is the house style of
  agency and fashion sites, where the pause reads as composure. On a content site the same
  pause reads as latency you were made to wait through.
- **Mechanism** — intercept the link click, play the cover animation, navigate when it
  completes, then play the uncover on arrival. State is carried across the boundary in
  `sessionStorage` so the incoming page knows to start covered.
- **Stack** — a page-transition library (Barba, Swup) or hand-rolled; with cross-document
  view transitions you can now express much of it in `pageswap`/`pagereveal` instead.
- **Params** — cover duration (0.4–0.6s); hold (as short as the navigation allows); uncover
  (0.5–0.8s); panel count (one, or several offset for a slatted look).
- **Use when** — a portfolio or brand site where the transition is part of the identity.
- **Don't use when** — people navigate frequently. A one-second toll on every click is
  exhausting by the fifth page.
- **Reduced motion** — navigate directly with no cover.
- **Performance** — the cover itself is trivial; the risk is that it masks a slow load and
  becomes indefinite. Cap the hold and reveal regardless.
- **Gotchas** — the classic failure is a transition that never completes because the
  navigation failed — always resolve the uncover on a timeout as well as on load. Focus must
  move to the new page's start, and the change must be announced; a screen reader user gets
  nothing from the curtain but silence.
- **References** — https://css-tricks.com/cross-document-view-transitions-part-1/

---

### Directional slide (`route.directional-slide`)

- **One line** — pages slide in from the side that matches the direction you travelled.
- **What the reader sees** — Move forward through a sequence and the new page slides in from
  the right, pushing the old one out to the left. Go back and it reverses — the previous page
  returns from the left. The direction encodes the navigation itself, so you always know
  whether you went deeper or came back, which is exactly the spatial model phone operating
  systems trained everyone on.
- **Mechanism** — both snapshots translated in opposite directions; the direction chosen from
  navigation type (back/forward) or from a known hierarchy.
- **Stack** — view transitions with direction-aware pseudo-element animations, or a router
  with a history-aware transition.
- **Params** — distance (full width, or a partial offset for the outgoing page — 20–30% reads
  as depth); duration (0.3–0.4s); easing (ease-out).
- **Use when** — linear flows: onboarding, checkout, documentation with next/previous, mobile
  web apps.
- **Don't use when** — the information architecture is not linear. Sliding between unrelated
  sections implies an order that does not exist.
- **Reduced motion** — crossfade or nothing.
- **Performance** — two full-viewport layers moving; cheap on the compositor, but the
  snapshot cost applies.
- **Gotchas** — you must detect back-navigation to reverse it, and browser back is not the
  only way to go back. Getting the direction wrong is worse than having no direction — it
  actively misleads.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API/Using

---

### Prerender-and-go (`route.prerender-instant`)

- **One line** — the next page is already rendered before you click, so the transition is instant.
- **What the reader sees** — You click and you are simply there. No spinner, no crossfade
  into a skeleton, no perceptible wait — the destination is complete at the moment of
  arrival, including its images. Paired with a shared-element morph, the effect is
  indistinguishable from a native application, and the reason is not that the animation is
  better but that there is nothing to hide.
- **Mechanism** — Speculation Rules prerender likely destinations in the background; the
  navigation then swaps to an already-rendered document, and any view transition runs between
  two live pages.
- **Stack** — platform (Speculation Rules API) plus whatever transition you are using.
- **Params** — eagerness (conservative on hover/pointerdown, moderate, or eager); which URLs
  are eligible.
- **Use when** — a site with a predictable next click: article to article, listing to detail.
  This is the single highest-value item in this family.
- **Don't use when** — destinations have side effects on load (analytics events, one-time
  tokens, anything that must not run speculatively), or bandwidth is a real constraint for
  your audience.
- **Reduced motion** — unaffected; prerendering is not motion.
- **Performance** — it trades bandwidth and memory for latency. Eager prerendering of many
  URLs is a real cost on mobile data.
- **Gotchas** — prerendered pages run their scripts before the user has committed to
  visiting, so anything with a side effect must be gated on the page actually being
  activated. Over-eager rules waste data on pages nobody visits.
- **References** — https://trade-assistance.com/blog/cross-document-view-transitions-mpa-2026/ ·
  https://developer.chrome.com/docs/web-platform/view-transitions/cross-document

---

### Content stagger on arrival (`route.arrival-stagger`)

- **One line** — the new page assembles itself instead of appearing complete.
- **What the reader sees** — The destination arrives and its parts settle in quick sequence:
  the header first, then the title, then the body, then the imagery — each a fraction of a
  second behind the last, over roughly half a second in total. It gives the arrival a shape
  and, more usefully, it puts the title on screen before the images, so you can start reading
  before the page has finished. It is the same idea as a hero sequence, applied to every page
  rather than the first.
- **Mechanism** — an entrance timeline run on arrival, ideally started in `pagereveal` before
  first paint so nothing flashes unstyled.
- **Stack** — any animation library; the trigger is the navigation lifecycle.
- **Params** — total duration (0.4–0.7s; a per-page ceremony must be much shorter than a
  once-per-visit one); stagger (0.06–0.1s); element count (four or five, not everything).
- **Use when** — content sites where each page has a consistent structure.
- **Don't use when** — the reader is navigating rapidly through many pages, or the content is
  reference material people scan.
- **Reduced motion** — everything present immediately.
- **Performance** — competes directly with the destination's own load work. Keep it cheap and
  do not gate the whole page's visibility on it.
- **Gotchas** — never hide content behind an entrance that might not run. If the animation
  fails, the page must still be readable — which means animating from a visible baseline, or
  setting the hidden state only once JavaScript has confirmed it can run.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API/Using

---

### Persistent shell (`route.persistent-shell`)

- **One line** — the frame stays put while only the content region changes.
- **What the reader sees** — Navigate between sections and the header, sidebar and footer do
  not move at all — no flicker, no re-render, no scroll jump. Only the middle of the screen
  changes, usually with a short crossfade. Because the furniture is continuous, the site
  feels like one application you are moving around inside rather than a series of documents.
  It is the least showy entry here and the one most responsible for a site feeling coherent.
- **Mechanism** — in an SPA, the shell is simply never unmounted. Across documents, naming
  the shell elements with matching `view-transition-name` values makes the browser hold them
  in place while the content region transitions.
- **Stack** — a client-side router, or view transitions with named persistent regions.
- **Params** — which regions persist; content transition duration (0.2–0.3s).
- **Use when** — documentation, dashboards, any site with heavy shared navigation.
- **Don't use when** — the shell genuinely differs between sections; forcing continuity on
  something that changes produces a visible morph where you wanted stillness.
- **Reduced motion** — no content crossfade; the persistence itself is not motion and should
  stay.
- **Performance** — the cheapest possible transition, since most of the viewport is not
  animating at all.
- **Gotchas** — persisting the shell means persisting its scroll position and its focus. If
  the sidebar keeps focus after navigation, keyboard users are left in the old context —
  move focus to the new content region and announce the route change in a live region.
- **References** — https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/ui-behaviors/cross-document-transitions.md

---

### Loading bar (`route.loading-bar`)

- **One line** — a thin progress bar at the top reports that navigation is under way.
- **What the reader sees** — Click a link and a slim coloured bar starts crossing the top of
  the window. It moves quickly at first, then slows as it approaches the right edge — never
  quite arriving — and completes with a snap when the page is ready, fading out immediately
  after. It is a lie in the sense that it does not measure real progress, and it is an honest
  one: it tells you the click registered and something is happening, which is the actual
  question the user has.
- **Mechanism** — an asymptotic progress animation started on navigation intent, forced to
  100% on completion.
- **Stack** — a few lines of CSS plus router hooks; several small libraries do exactly this.
- **Params** — start delay (~150ms, so fast navigations never show it at all); approach curve
  (fast to 80%, then crawl); completion (0.2s to full, then fade).
- **Use when** — navigations that can exceed about 300ms, which is most navigations on real
  networks.
- **Don't use when** — pages are prerendered and arrive instantly. A bar that flashes on
  every click is noise.
- **Reduced motion** — keep it; progress indication is feedback, not decoration. It can be
  made non-animated (stepped) if preferred.
- **Performance** — one transform. Free.
- **Gotchas** — the delay before showing is what stops it flickering on fast navigations, and
  it is the part most implementations skip. It must be announced to assistive technology as a
  busy state, not left as a purely visual signal.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API

---

### Scroll restoration handoff (`route.scroll-restore`)

- **One line** — going back returns you to exactly where you were, without a visible jump.
- **What the reader sees** — You scroll deep into a listing, open an item, then press back —
  and you are returned precisely to the row you left, already in position, with no flash of
  the top of the page and no scramble as the browser catches up. It is not really an
  animation; what you perceive is the *absence* of a jolt. Combined with a shared-element
  morph in reverse, back-navigation stops feeling like a reload and starts feeling like
  stepping back.
- **Mechanism** — the browser's own scroll restoration where it can be trusted; otherwise
  record the position on `pageswap` and restore it before first paint in `pagereveal`.
- **Stack** — platform, plus a small amount of state handling in the navigation lifecycle.
- **Params** — whether restoration is manual or automatic; what to do when the content has
  changed length since you left.
- **Use when** — always, on any site with long listings. This is a correctness feature that
  happens to live in this family.
- **Don't use when** — never; the only question is who does the restoring.
- **Reduced motion** — unaffected. Restore instantly rather than smooth-scrolling to
  position — an animated scroll to a deep offset is precisely the vestibular problem.
- **Performance** — free, and it prevents the far more expensive experience of a user
  re-scrolling.
- **Gotchas** — restoring before the content has laid out puts you at the wrong offset;
  restore after layout but before paint. A transition that animates the page while the scroll
  is being restored produces a visible slide from the top. Smooth-scroll libraries must be
  told about programmatic restoration or they animate to it.
- **References** — https://css-tricks.com/cross-document-view-transitions-part-1/

---

### Exit animation (`route.exit`)

- **One line** — the page you are leaving acknowledges that you are leaving.
- **What the reader sees** — Click a link and the current page does something brief before it
  goes: content lifts slightly and fades, or the section you clicked stays while everything
  around it recedes. It lasts a fifth of a second, and its purpose is to make the click feel
  answered — the page confirms the request before the network does. Without it, there is a
  dead interval between clicking and anything happening, and dead intervals are where users
  click again.
- **Mechanism** — an animation on the outgoing view, either in `pageswap` before the snapshot
  or as an old-view pseudo-element animation.
- **Stack** — view transitions, or a router's before-navigate hook.
- **Params** — duration (0.15–0.25s — this is inserted *before* a wait, so it must be short);
  what moves (little, and quickly).
- **Use when** — navigations with any meaningful latency.
- **Don't use when** — the destination is prerendered and instant. Then an exit animation is
  a delay you have added by hand.
- **Reduced motion** — none; navigate directly.
- **Performance** — negligible, but it is time added to every navigation. Budget it against
  the load it is covering.
- **Gotchas** — never block navigation on the exit animation completing; start the request
  immediately and let the animation run alongside. Exit animations that must finish first are
  how sites end up feeling slower than they are.
- **References** — https://developer.chrome.com/docs/web-platform/view-transitions/cross-document

---

## Family notes

**Prerender first, animate second.** The best page transition on an unprepared page still
transitions into a spinner. Speculation Rules plus a plain crossfade beats elaborate
choreography over a cold load, every time.

**Both documents must opt in**, the transition only applies to same-origin navigations, and
`view-transition-name` must be unique per document at the moment it runs. All three fail
silently, which makes this family unusually annoying to debug: the symptom is always "nothing
happened".

**Only what is on screen exists.** The outgoing snapshot contains the visible viewport, not
the whole document. Any element you intend to morph must be in view when the navigation
starts.

**A transition is not a navigation announcement.** Screen reader users get nothing from a
curtain wipe. Route changes need focus moved to the new content and the change announced —
that obligation is independent of, and more important than, anything in this file.

**Budget the whole interaction.** Exit animation, network, entrance animation and any
arrival stagger all add up. If the total exceeds roughly half a second on a warm connection,
the transition is no longer covering latency — it is the latency.
