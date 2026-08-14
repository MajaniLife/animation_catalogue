# Physics, drag & gesture

Motion with momentum and constraint. Springs, inertia, dragging, throwing, collision — the
family where you stop specifying how long something takes and start specifying the forces
acting on it.

**The change of model is the point.** A duration-based animation is a promise: this will take
400ms and follow this curve, whatever happens. A spring is a simulation: it has a current
position and a current velocity, and it moves toward a target under forces. It has no
duration you set, and that is a feature — because at any moment you can change the target and
the motion continues *from the velocity it already had* rather than restarting.

That property, **velocity inheritance**, is what makes this family the correct choice for
anything the user is touching. When someone flicks a sheet and it reverses direction, a
duration animation must stop and start again — a visible break — while a spring simply
absorbs the new target and keeps its momentum. Interruptibility is not a nice extra here; it
is the whole argument.

**The parameters, in the terms that matter.** Stiffness is how hard the spring pulls toward
its target — higher is faster and more aggressive. Damping is the resistance that removes
energy — higher settles sooner and overshoots less. Mass is how much inertia the object has —
higher feels heavier and slower to start. Typical library defaults sit around stiffness 100,
damping 10, mass 1; that combination oscillates visibly, which is why almost every real UI
raises the damping.

The one number to build intuition on is the **damping ratio**: below 1 the spring overshoots
and wobbles (underdamped), at 1 it arrives as fast as possible without overshooting
(critically damped), above 1 it creeps in slowly (overdamped). Most interface motion wants
critically damped or just below.

**When not to use a spring.** Anything choreographed. A hero sequence needs elements to land
at known times relative to each other, and springs have no duration to synchronise. Springs
for interaction, timelines for choreography.

---

### Drag to move (`gesture.drag`)

- **One line** — the element follows the pointer exactly, then obeys physics on release.
- **What the reader sees** — Press on a card and it lifts slightly — a shadow appears, it
  scales up a fraction — and then it tracks your pointer precisely, one-to-one, with no lag
  at all. That exactness is what makes it feel like you are holding it rather than steering
  it. Release and it does not stop dead: it continues in the direction you were moving,
  decelerating, and settles. The lift on grab and the glide on release are the two details
  that separate a draggable object from a div with a mousemove handler.
- **Mechanism** — pointer capture, direct transform updates during the drag (no easing, no
  tween), then velocity-based inertia on release.
- **Stack** — GSAP Draggable with InertiaPlugin (both free since 2025); Motion's drag props;
  or Pointer Events directly.
- **Params** — lift scale (1.02–1.05) and shadow; axis lock; bounds; release behaviour
  (inertia, snap, or immediate stop).
- **Use when** — cards, sliders, sheets, canvas panning, anything the user should feel they
  are physically manipulating.
- **Don't use when** — there is no keyboard alternative. Drag-only interaction is inaccessible
  by definition; provide buttons or keyboard drag.
- **Reduced motion** — the drag itself is direct manipulation and stays. Drop the lift and the
  inertia; release should stop immediately.
- **Performance** — transform writes during pointer move. Use pointer capture so the drag
  survives the pointer leaving the element, and promote the dragged layer once rather than
  per frame.
- **Gotchas** — apply `touch-action` correctly or the browser's own scrolling competes with
  your drag and both feel broken. Never animate the element toward the pointer with an easing
  — during the drag the position must be exact, and any smoothing reads as lag.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/Pointer_events

---

### Throw with inertia (`gesture.throw-inertia`)

- **One line** — release velocity carries the element onward and it coasts to a stop.
- **What the reader sees** — Flick a carousel and it keeps going after your finger leaves,
  slowing gradually and coming to rest — fast flicks travel further, gentle ones barely move.
  The deceleration curve is what your hand expects from a physical object sliding on a
  surface, so nobody has to learn it. When it reaches a boundary it either stops firmly or
  pulls back with resistance, which communicates the edge without a message.
- **Mechanism** — sample pointer velocity over the last few frames before release, then
  integrate a decelerating motion from that initial velocity, optionally snapping to the
  nearest valid resting point.
- **Stack** — GSAP InertiaPlugin, Motion's drag momentum, or a hand-rolled friction
  integrator.
- **Params** — friction/deceleration; maximum throw distance; snap targets; boundary
  behaviour (hard stop, or rubber-band and return).
- **Use when** — carousels, long lists, map panning, anything with more content than screen.
- **Don't use when** — precision placement matters. Inertia is the enemy of putting something
  exactly where you meant.
- **Reduced motion** — stop on release with no coast.
- **Performance** — a decaying animation on one element. Cheap.
- **Gotchas** — sample velocity over several recent frames rather than the last one, or a
  momentary pause before release produces a dead throw. Snap points must be resolved from the
  *projected* resting position, not the release position, or the snap feels like a
  correction rather than a destination.
- **References** — https://blog.maximeheckel.com/posts/the-physics-behind-spring-animations/

---

### Spring settle (`physics.spring-settle`)

- **One line** — the element arrives with a small overshoot and settles back.
- **What the reader sees** — A panel slides in, goes very slightly past its resting position,
  and comes back — a movement of a few pixels, over maybe a tenth of a second at the end of
  the transition. You do not consciously see the overshoot; you register that the panel has
  *weight*, that it arrived rather than being placed. Increase the overshoot and it becomes
  bouncy and playful; remove it entirely and the motion becomes precise and slightly clinical.
  This single parameter is most of the difference between an interface that feels friendly and
  one that feels industrial.
- **Mechanism** — a spring solver with a damping ratio just below critical, run per frame
  until velocity and displacement fall under a threshold.
- **Stack** — Motion, React Spring, or any library with a real spring solver. CSS
  `linear()` easing can approximate a spring curve for non-interruptible cases.
- **Params** — stiffness (200–400 for UI); damping (20–30 at those stiffnesses — library
  defaults around 10 are noticeably wobbly); mass (1 unless you want deliberate heaviness).
- **Use when** — panels, modals, toggles, anything appearing in response to a direct action.
- **Don't use when** — several elements must land in a coordinated rhythm; springs finish
  whenever they finish.
- **Reduced motion** — arrive at the target with no overshoot, or instantly.
- **Performance** — a per-frame solve per property. Negligible for UI, and it can run on the
  compositor via CSS `linear()` when interruption is not needed.
- **Gotchas** — springs need a rest threshold or they run forever at imperceptible amplitudes,
  quietly keeping the page awake. Overshoot on a element flush against a boundary makes it
  visibly cross an edge — either clamp it or design the layout with room.
- **References** — https://developer.android.com/develop/ui/views/animations/spring-animation ·
  https://blog.nordcraft.com/physics-based-animations-spring-to-life

---

### Rubber-band edge (`gesture.rubber-band`)

- **One line** — dragging past a limit meets increasing resistance, then snaps back.
- **What the reader sees** — Pull a list beyond its last item and it keeps moving, but less
  and less — you drag an inch and it travels a quarter of one, the resistance growing the
  further you go. Let go and it springs back to the boundary. Nothing tells you that you have
  reached the end; you *feel* it, and you also learn that the app is still responding rather
  than frozen. It is one of the most successfully copied interaction ideas in software.
- **Mechanism** — beyond the boundary, apply the drag delta through a diminishing function so
  displacement approaches an asymptote; on release, spring back to the limit.
- **Stack** — most drag libraries offer it as an option; the function is a few lines.
- **Params** — resistance curve; maximum overscroll (60–120px); return spring (stiff, damped
  — the return should be decisive).
- **Use when** — scrollable regions, carousels, sheets, any bounded drag.
- **Don't use when** — the boundary is not a real limit. Overscroll implies "there is nothing
  more here", so using it where more content could load is misleading.
- **Reduced motion** — hard stop at the boundary, no overscroll.
- **Performance** — trivial.
- **Gotchas** — the resistance function must be continuous at the boundary or there is a
  visible hitch as the element crosses it. On the web, native overscroll behaviour may also be
  running — set `overscroll-behavior` or you get two rubber bands fighting.
- **References** — https://www.breakfreegraphics.com/design-blog/building-fluid-interfaces/

---

### Swipe to dismiss (`gesture.swipe-dismiss`)

- **One line** — drag far or fast enough and the element leaves.
- **What the reader sees** — Push a notification sideways. It follows your finger, and as it
  moves it fades and the content behind becomes visible. Let go early and it springs back into
  place; get past a threshold — or flick quickly even a short distance — and it accelerates
  away and is gone. The threshold is never stated, but it is discoverable in one attempt
  because the element's opacity is telling you how close you are the whole time.
- **Mechanism** — a drag with a commit threshold on *either* distance or velocity, an exit
  animation continuing the gesture's direction, and a spring back otherwise.
- **Stack** — a drag library plus presence handling for the removal.
- **Params** — distance threshold (30–50% of width); velocity threshold (so a fast short flick
  also commits); exit duration (0.2s, continuing the direction of travel).
- **Use when** — notifications, cards, mail rows, image galleries.
- **Don't use when** — the action is destructive and unrecoverable. Provide undo, or require a
  deliberate confirmation.
- **Reduced motion** — the element disappears without travel once committed.
- **Performance** — one element; trivial.
- **Gotchas** — supporting *both* thresholds is what makes it feel right; distance-only feels
  sticky, velocity-only fires accidentally. The row must also be dismissible from the
  keyboard and expose the action to assistive technology — a swipe is invisible to a screen
  reader.
- **References** — https://www.breakfreegraphics.com/design-blog/building-fluid-interfaces/

---

### Sheet with detents (`gesture.sheet-detents`)

- **One line** — a panel drags between defined heights and settles into the nearest.
- **What the reader sees** — A sheet rises from the bottom edge and stops at a half-height
  position. Drag it upward and it follows your finger; release and it settles to whichever
  stop is nearest — half, full, or closed — with a spring. Drag it slowly and it goes where
  you put it; flick it and it jumps a whole stop. It reads as a physical panel with detents,
  and it makes a single surface serve peek, browse and full-screen use without any explicit
  controls.
- **Mechanism** — a drag bounded to a set of snap positions; the resting stop is projected
  from position plus velocity, then a spring animates to it.
- **Stack** — a drag library plus a spring; the projection is the interesting part.
- **Params** — detent positions; projection factor (how much velocity counts toward the
  target); spring stiffness and damping.
- **Use when** — mobile detail panels, map overlays, media controls, filter surfaces.
- **Don't use when** — on desktop, where a sheet is usually the wrong pattern.
- **Reduced motion** — jump between detents instantly.
- **Performance** — one element, one spring.
- **Gotchas** — content inside the sheet scrolls, and the sheet itself drags — deciding which
  gesture wins is the hard part. The rule that works: if the inner content is scrolled to the
  top and the drag is downward, the sheet moves; otherwise the content scrolls. Getting this
  wrong is why some sheets feel broken.
- **References** — https://developer.android.com/develop/ui/views/animations/spring-animation

---

### Pinch and zoom (`gesture.pinch-zoom`)

- **One line** — two fingers scale and pan content around their midpoint.
- **What the reader sees** — Place two fingers on an image and spread them; the image grows,
  anchored so the point between your fingers stays under them. Move both fingers and it pans.
  Release and it either stays or, if you have zoomed out beyond the fit, springs back to fill
  the frame. The anchoring is what matters — if the image scales around its own centre instead
  of the pinch midpoint, the content slides away from your fingers and the whole interaction
  feels wrong.
- **Mechanism** — track two pointers, derive scale from the distance ratio and translation
  from the midpoint delta, and apply a combined transform with the correct origin.
- **Stack** — Pointer Events with multiple active pointers, or a gesture library.
- **Params** — scale limits (typically 1× to 4×); spring-back on release beyond limits;
  double-tap-to-zoom step.
- **Use when** — images, maps, diagrams, anything with detail worth inspecting.
- **Don't use when** — the browser's native pinch-zoom already serves the purpose. Overriding
  page zoom is an accessibility problem; zoom the *content*, and never disable page zoom.
- **Reduced motion** — direct manipulation stays; drop the spring-back animation.
- **Performance** — transform-only, but a scaled raster resamples — for large images, swap in
  a higher-resolution source once the gesture settles rather than during it.
- **Gotchas** — the transform origin must track the live pinch midpoint, not be set once at
  gesture start. Handle a third pointer arriving mid-gesture without the transform jumping.
  Never `user-scalable=no`.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/Pointer_events

---

### Collision field (`physics.collision`)

- **One line** — real rigid-body simulation: objects fall, stack and knock each other around.
- **What the reader sees** — Shapes drop into a container and pile up, resting against each
  other at whatever angles they land, wobbling slightly as later arrivals hit them. Drag one
  out and the stack settles. Unlike everything else in this family, these objects are aware of
  one another — the motion is emergent rather than authored, and no two runs look the same.
  On a playful landing page it is genuinely engaging; it also cannot be art-directed, which is
  the trade.
- **Mechanism** — a 2D physics engine steps a simulation of bodies with mass, restitution and
  friction; DOM elements or canvas shapes are synced to the body positions each frame.
- **Stack** — Matter.js is the common choice. Note that this is genuinely different from
  "spring physics" — those are easing curves; this is a solver.
- **Params** — gravity; restitution (bounciness); friction; body count.
- **Use when** — a deliberate playful moment: a 404, an about page, a tag cloud.
- **Don't use when** — the objects are functional. Content that moves unpredictably cannot be
  clicked reliably, which is a motor-accessibility problem.
- **Reduced motion** — a static arrangement; do not simulate.
- **Performance** — the simulation is CPU work every frame and scales with body count and
  contact pairs. Fifty bodies is comfortable; five hundred is not. Sleep bodies that come to
  rest.
- **Gotchas** — syncing DOM elements to simulated bodies means writing transforms for every
  body every frame; past a few dozen, render to canvas instead. Resize invalidates the world
  bounds and objects escape — rebuild them.
- **References** — https://blog.maximeheckel.com/posts/the-physics-behind-spring-animations/

---

### Wiggle nudge (`physics.wiggle`)

- **One line** — a short oscillation that decays, drawing attention to one element.
- **What the reader sees** — A field shakes briefly — three or four decreasing oscillations
  over a third of a second — and stops. On an invalid form field it reads unmistakably as
  refusal, and it is understood without a word. On a button ignored for a while it reads as a
  nudge. The decay is essential: a constant-amplitude shake looks like a broken loop, while a
  decaying one looks like a physical reaction.
- **Mechanism** — a damped oscillation on `translateX` or `rotate`, three to five cycles with
  decreasing amplitude.
- **Stack** — CSS keyframes with hand-authored decay, or a wiggle easing from an animation
  library.
- **Params** — cycles (3–5); amplitude (4–10px); duration (0.3–0.5s total).
- **Use when** — form validation failure, an incorrect entry, a nudge toward a primary action.
- **Don't use when** — as decoration. A wiggle means "no" — using it for delight teaches
  people the wrong vocabulary.
- **Reduced motion** — no shake. Use a colour change and, more importantly, a message.
- **Performance** — trivial.
- **Gotchas** — the shake must never be the only error indication; error text and
  `aria-invalid` carry the meaning, and this is a flourish on top. Avoid on large elements —
  a shaking panel is genuinely unpleasant and can be a vestibular trigger.
- **References** — https://educationalvoice.co.uk/spring-launch-animation/

---

### Pull to refresh (`gesture.pull-refresh`)

- **One line** — dragging down past the top arms a refresh, releasing triggers it.
- **What the reader sees** — At the top of a list, pull down. The content follows with
  resistance and an indicator appears in the space you have opened, rotating in proportion to
  how far you have pulled. Past a threshold the indicator changes state to say it will fire.
  Release and the list springs back to a position that keeps the spinner visible while the
  request runs, then closes when data arrives. Every stage of the gesture reports what will
  happen if you let go now, which is why it works without instructions.
- **Mechanism** — overscroll drag mapped to indicator progress, a commit threshold on release,
  a held position during the request, and a spring back on completion.
- **Stack** — hand-rolled on Pointer Events, or a component library. Native
  `overscroll-behavior` must be managed so the browser's own gesture does not interfere.
- **Params** — trigger distance (60–80px); resistance; held position during load; minimum
  spinner time (~400ms, so a fast response does not flash).
- **Use when** — a mobile list whose content changes and where a manual refresh is meaningful.
- **Don't use when** — content updates automatically, or on desktop where a button is clearer.
- **Reduced motion** — the indicator appears without rotation; the list still moves, since
  that is direct manipulation.
- **Performance** — trivial.
- **Gotchas** — must only arm when the list is already scrolled fully to the top, or it hijacks
  ordinary scrolling. The minimum display time is what prevents a jarring flash on a fast
  network. Provide a non-gesture refresh control too.
- **References** — https://www.breakfreegraphics.com/design-blog/building-fluid-interfaces/

---

### Magnetic snap (`gesture.snap-points`)

- **One line** — a dragged element locks onto valid positions as it passes them.
- **What the reader sees** — Drag a slider handle and it moves freely, but near each marked
  value it pulls in slightly and holds, requiring a little more movement to leave again. You
  feel the stops. On a canvas, a dragged block aligns to a grid the same way — it snaps into
  line just before you would have placed it there. It makes precise placement possible without
  demanding precision from the user, which is the entire point.
- **Mechanism** — during the drag, bias the position toward the nearest snap point within a
  radius; on release, spring to the resolved point.
- **Stack** — any drag implementation plus a snap resolver; CSS scroll-snap for the scrolling
  case.
- **Params** — snap radius (10–20px); strength of the in-drag bias; whether snapping applies
  during the drag or only on release.
- **Use when** — sliders with discrete values, canvas editors, timeline scrubbers, carousels.
- **Don't use when** — the value is genuinely continuous. Snapping a colour picker or a volume
  control removes precision people need.
- **Reduced motion** — snap without a spring.
- **Performance** — trivial.
- **Gotchas** — biasing during the drag means the element no longer tracks the pointer exactly,
  which is a real trade; keep the bias small or apply it only on release. Snap points closer
  together than the radius fight each other and produce sticky, unpredictable movement.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/scroll-snap-type

---

## Family notes

**Springs for interaction, durations for choreography.** If the user is touching it, or can
interrupt it, use a spring — velocity inheritance is why it will feel right. If several
elements must land in a rhythm, use a timeline.

**Raise the damping.** Common library defaults (around stiffness 100, damping 10) visibly
oscillate. UI motion generally wants stiffness 200–400 with damping 20–30 — critically damped
or just under.

**Exactness during, physics after.** While a pointer is down, the element must follow it
one-to-one; any easing reads as lag. Physics belongs to the moment of release.

**Set a rest threshold.** A spring without one never formally finishes, and keeps the page
rendering at amplitudes nobody can see.

**Every gesture needs a non-gesture path.** Drag, swipe, pinch and pull are invisible to
keyboard and screen reader users and unavailable to many people with motor impairments. The
gesture is an accelerator for people who can use it, never the only route to the action.
