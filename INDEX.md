# Index

261 entries across twenty-two families, plus two cross-cutting documents. Cost is bundle *and* runtime: **free** (CSS/platform only),
**cheap** (a few KB or a library already present), **moderate** (a dedicated library or plugin),
**heavy** (its own rendering stack).

Jump to the [decision tree](#decision-tree) if you know the feeling you want but not the name,
or the [conflict matrix](#conflict-matrix) before combining two effects.

---

## All entries

### Entrance & reveal — [`families/01`](families/01-entrance-reveal.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `entrance.fade-rise` | CSS | free | The default: fades up a short distance into place |
| `entrance.mask-rise` | SplitText | moderate | Text rises from behind a hard edge, line by line |
| `entrance.clip-wipe` | CSS | free | Uncovered by a moving edge rather than faded in |
| `entrance.clip-expand` | ScrollTrigger | moderate | Frame opens outward while the image scales down to meet it |
| `entrance.scale-settle` | CSS | free | Arrives fractionally too large or small and resolves |
| `entrance.blur-focus` | CSS | free | Resolves from out-of-focus to sharp |
| `entrance.batch-stagger` | ScrollTrigger | moderate | A grid reveals as one coordinated wave |
| `entrance.list-cascade` | CSS | free | List items arrive in reading order |
| `entrance.curtain` | CSS | free | A solid panel slides away to expose content |
| `entrance.split-part` | CSS | free | Two halves separate to reveal what is between |
| `entrance.rule-draw` | CSS | free | Hairline rules extend as sections arrive |
| `entrance.flip-3d` | CSS | free | Rotates in from an angle, as if hinged |
| `entrance.skeleton-swap` | CSS | free | Placeholders become the content they stood in for |
| `entrance.starting-style` | CSS | free | Native entry animation from `display: none` |
| `entrance.hero-sequence` | timeline lib | moderate | The composite: a first screen with a scripted order |

### Text & kinetic typography — [`families/02`](families/02-text-kinetic-typography.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `text.char-cascade` | SplitText | moderate | Letters arrive in a rapid directional wave |
| `text.word-stagger` | SplitText | moderate | Same at word granularity — the shippable version |
| `text.scramble` | ScrambleText | moderate | Characters churn through glyphs before settling |
| `text.typewriter` | CSS/JS | free | Types itself out, character by character |
| `text.word-cycle` | CSS | free | One word in a sentence swaps for alternatives |
| `text.scrub-highlight` | SplitText + ST | moderate | Words brighten as the paragraph scrolls through view |
| `text.variable-weight` | CSS | free | The typeface's own axes animate |
| `text.counter-odometer` | any | cheap | A number counts up to its value |
| `text.outline-fill` | CSS | free | Hollow letterforms fill with colour |
| `text.stroke-draw` | DrawSVG | moderate | Letterforms draw themselves as if written |
| `text.wave` | CSS | free | Characters ride a travelling sine |
| `text.gradient-sweep` | CSS | free | A band of light travels across the type |
| `text.physics-fall` | Physics2D | moderate | Letters detach and fall out of the layout |
| `text.letter-magnet` | any | cheap | Individual characters lean toward the cursor |
| `text.line-flip-3d` | CSS | free | Lines rotate in on a horizontal hinge |

### Scroll-driven — [`families/03`](families/03-scroll-driven.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `scroll.parallax-layer` | CSS/ST | free–moderate | Layers move at different rates, implying depth |
| `scroll.progress-bar` | CSS | free | A thin bar fills as you move through the page |
| `scroll.pin-sequence` | ScrollTrigger | moderate | A section sticks while its contents advance |
| `scroll.horizontal-rail` | ScrollTrigger | moderate | Vertical scrolling drives horizontal movement |
| `scroll.clip-expand` | ScrollTrigger | moderate | An image opens out of a slot as it enters |
| `scroll.sticky-stack` | CSS | free | Cards pile up, each sticking as the next slides over |
| `scroll.velocity-skew` | ScrollTrigger | moderate | Content leans in proportion to scroll speed |
| `scroll.snap-sections` | CSS | free | Scrolling settles onto section boundaries |
| `scroll.marquee-velocity` | ScrollTrigger | moderate | A scrolling band reacts to scroll speed and direction |
| `scroll.frame-sequence` | canvas + ST | heavy | Scroll scrubs a frame-by-frame image sequence |
| `scroll.theme-shift` | ScrollTrigger | cheap | The palette changes between sections |
| `scroll.reveal-enter` | CSS/IO/ST | free–moderate | The workhorse: animate in once on first view |
| `scroll.smooth-normalise` | Lenis | cheap | Scrolling is interpolated rather than immediate |

### Pointer, hover & cursor — [`families/04`](families/04-pointer-hover-cursor.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `pointer.underline-draw` | CSS | free | A rule sweeps in under a link and out the way you left |
| `pointer.magnetic` | any | cheap | An element leans toward the cursor |
| `pointer.stack-swap` | CSS | free | A label slides out while an identical copy slides in |
| `pointer.fill-wipe` | CSS + JS | free | Colour floods from the edge the pointer entered |
| `pointer.cursor-dot` | any | cheap | A custom cursor object with a lagged companion |
| `pointer.cursor-state` | any | cheap | The cursor changes shape to say what is under it |
| `pointer.image-follow` | any | cheap | A preview image trails the cursor over a link list |
| `pointer.tilt-3d` | CSS + JS | free | A card tips in perspective, tracking the cursor |
| `pointer.repulsion` | any | cheap | Many small elements scatter away from the cursor |
| `pointer.spotlight` | CSS | free | A soft light follows the cursor across a surface |
| `pointer.skew-move` | any | cheap | An element leans in proportion to pointer speed |
| `pointer.hover-intent` | JS | free | The effect waits to be sure you meant it |

### Layout & shared-element — [`families/05`](families/05-layout-shared-element.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `layout.flip-reposition` | FLIP | cheap | Items slide to new places when order changes |
| `layout.flip-filter` | FLIP | cheap | Filtering moves the survivors instead of redrawing |
| `layout.shared-element` | View Transitions | free | A thumbnail becomes the hero of the view it opens |
| `layout.expand-in-place` | FLIP | cheap | A card grows into a panel without leaving position |
| `layout.height-auto` | CSS | free | The accordion problem: animating to an unknown height |
| `layout.view-transition-auto` | platform | free | The browser crossfades the view in one call |
| `layout.drag-reorder` | drag + FLIP | moderate | Dragging makes neighbours move out of the way live |
| `layout.tab-indicator` | CSS + JS | free | A marker slides between tabs, resizing as it goes |
| `layout.modal-from-origin` | FLIP | cheap | A dialog grows out of the control that opened it |
| `layout.list-add-remove` | FLIP | cheap | Adding or deleting makes neighbours move, not jump |
| `layout.masonry-settle` | FLIP | cheap | An irregular grid reflows as items load |
| `layout.split-view` | CSS/FLIP | cheap | A side panel opens and main content compresses |

### Page & route — [`families/06`](families/06-page-route-transitions.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `route.crossfade` | platform | free | The browser fades old page into new, in two lines |
| `route.shared-element` | platform | free | The thumbnail you clicked becomes the next page's hero |
| `route.curtain` | Barba/Swup | moderate | A panel covers, the page changes, the panel leaves |
| `route.directional-slide` | platform | free | Pages slide in from the direction you travelled |
| `route.prerender-instant` | Speculation Rules | free | The next page is rendered before you click |
| `route.arrival-stagger` | any | cheap | The new page assembles instead of appearing complete |
| `route.persistent-shell` | router/VT | free | The frame stays put while only content changes |
| `route.loading-bar` | CSS | free | A thin bar reports that navigation is under way |
| `route.scroll-restore` | platform | free | Back returns you exactly where you were |
| `route.exit` | platform | free | The page you are leaving acknowledges the click |

### SVG & path — [`families/07`](families/07-svg-path.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `svg.stroke-draw` | CSS | free | A stroke reveals itself, as if being drawn |
| `svg.morph` | MorphSVG | moderate | One shape becomes another by interpolating geometry |
| `svg.motion-path` | CSS | free | An element travels along a defined route |
| `svg.dash-flow` | CSS | free | A dashed line's dashes travel along it |
| `svg.icon-state` | CSS | free | An icon animates between its meanings |
| `svg.progress-ring` | CSS | free | A circular stroke fills to show a proportion |
| `svg.filter-distort` | SVG filters | heavy | Filters warp, blur or fracture content over time |
| `svg.clip-reveal` | CSS | free | Content revealed through a moving vector shape |
| `svg.signature` | CSS | free | A signature writes itself |
| `svg.diagram-build` | timeline lib | moderate | A diagram assembles in the order you should read it |
| `svg.counter-arc` | CSS + JS | free | An arc sweeps to a value while a number counts |

### 3D & WebGL — [`families/08`](families/08-3d-webgl.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `3d.scene-entrance` | Three/R3F | heavy | The object arrives rather than simply being there |
| `3d.scroll-camera` | Three + ST | heavy | Scrolling moves the camera through the scene |
| `3d.idle-orbit` | Three/R3F | heavy | The object turns slowly on its own |
| `3d.shader-transition` | Three/OGL | heavy | One image dissolves into another through distortion |
| `3d.particle-field` | Three/R3F | heavy | Thousands of small objects moving as one system |
| `3d.mesh-hover-distort` | OGL/Three | heavy | An image plane deforms under the pointer |
| `3d.material-morph` | Three/R3F | heavy | The object's finish changes: colour, metalness |
| `3d.post-processing` | Three/R3F | heavy | A full-screen pass adds glow, grain, grade |
| `3d.exploded-assembly` | Three/R3F | heavy | Components fly apart to show construction |
| `3d.fallback` | image | free | What everyone who cannot run the scene gets |

### Physics, drag & gesture — [`families/09`](families/09-physics-drag-gesture.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `gesture.drag` | Draggable/Motion | moderate | Follows the pointer exactly, then obeys physics |
| `gesture.throw-inertia` | Inertia | moderate | Release velocity carries it onward to a stop |
| `physics.spring-settle` | spring lib | cheap | Arrives with a small overshoot and settles |
| `gesture.rubber-band` | drag lib | cheap | Dragging past a limit meets increasing resistance |
| `gesture.swipe-dismiss` | drag lib | cheap | Drag far or fast enough and the element leaves |
| `gesture.sheet-detents` | drag + spring | moderate | A panel drags between defined heights |
| `gesture.pinch-zoom` | Pointer Events | cheap | Two fingers scale and pan around their midpoint |
| `physics.collision` | Matter.js | heavy | Real rigid-body simulation: fall, stack, knock |
| `physics.wiggle` | CSS | free | A short decaying oscillation draws attention |
| `gesture.pull-refresh` | JS | cheap | Dragging past the top arms a refresh |
| `gesture.snap-points` | drag lib | cheap | A dragged element locks onto valid positions |

### Micro-interaction & feedback — [`families/10`](families/10-micro-interaction-feedback.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `micro.press` | CSS | free | The button visibly reacts the instant it is pressed |
| `micro.toggle` | CSS | free | A switch's knob travels between its two positions |
| `micro.optimistic` | state | free | The interface acts as if the request already succeeded |
| `micro.delayed-spinner` | CSS + timer | free | The indicator waits, so fast responses show nothing |
| `micro.success-check` | CSS | free | A tick draws itself to confirm completion |
| `micro.toast` | toast lib | cheap | A notification slides in, holds, and leaves |
| `micro.field-validation` | CSS | free | A field shows in place whether input is acceptable |
| `micro.copy-confirm` | CSS | free | The copy button briefly becomes a confirmation |
| `micro.ripple` | CSS + JS | free | A circular wave expands from the point of contact |
| `micro.focus-ring` | CSS + JS | free | The focus indicator moves rather than teleporting |
| `micro.count-change` | CSS | free | A number visibly ticks when it changes |
| `micro.skeleton-shimmer` | CSS | free | Placeholders pulse to say "loading", not "broken" |

### Data-visualisation — [`families/11`](families/11-data-visualisation-motion.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `dataviz.bar-grow` | CSS/D3 | free–cheap | Bars rise from the baseline to their values |
| `dataviz.line-draw` | CSS | free | A time series draws itself left to right |
| `dataviz.value-transition` | D3 keyed | cheap | The chart moves between two datasets |
| `dataviz.type-morph` | D3 | moderate | The same data restructures into another chart form |
| `dataviz.counter` | any | cheap | A headline figure counts to its value |
| `dataviz.sort-reorder` | FLIP/D3 | cheap | Re-sorting moves the rows instead of redrawing |
| `dataviz.time-scrub` | ST + chart | moderate | Scroll advances the chart through time |
| `dataviz.highlight-focus` | CSS | free | One series brightens while the others recede |
| `dataviz.threshold-pulse` | CSS | free | A value crossing a limit announces itself |
| `dataviz.map-flow` | SVG | cheap | Animated paths show movement between places |
| `dataviz.live-update` | canvas | cheap | A real-time chart admits new data without jumping |

### Ambient & decorative — [`families/12`](families/12-ambient-decorative.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `ambient.marquee` | CSS | free | A strip of text or logos scrolls sideways, forever |
| `ambient.gradient-drift` | CSS | free | A large soft gradient shifts slowly behind content |
| `ambient.float` | CSS | free | Decorative objects bob gently and independently |
| `ambient.grain` | CSS | free | A fine noise texture shifts over everything |
| `ambient.breathe` | CSS | free | An element scales gently in and out |
| `ambient.aurora` | WebGL/CSS | cheap–heavy | Coloured light appears to move through a field |
| `ambient.auto-carousel` | carousel lib | cheap | Slides advance on a timer without being asked |
| `ambient.cursor-trail` | canvas | cheap | Something follows the pointer and lingers |
| `ambient.video-loop` | platform | heavy | A muted video plays behind the content, forever |
| `ambient.idle-nudge` | JS | free | After inactivity, something moves to re-engage |

### Media & image — [`families/16`](families/16-media-image.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `media.blur-up` | framework/build | cheap | A tiny blurred version stands in until the real image arrives |
| `media.ken-burns` | CSS | free | A still photograph slowly pans and zooms |
| `media.compare-slider` | CSS | free | A draggable divider wipes between two versions |
| `media.lightbox` | View Transitions | free | A thumbnail expands into a viewer and returns to its place |
| `media.video-scrub` | platform + ST | moderate | Scroll position controls video playback time |
| `media.hover-preview` | platform | cheap | A thumbnail starts playing when you point at it |
| `media.progressive-reveal` | encoder | free | The image appears as it decodes, coarse to fine |
| `media.carousel-slide` | CSS scroll-snap | free | A horizontal strip advances one item at a time |
| `media.zoom-inspect` | CSS + JS | cheap | Pointing at a product image magnifies that region |
| `media.ratio-settle` | HTML | free | The frame is correct before the image exists |
| `media.grid-reveal` | ST + queue | cheap | A photo grid fills in reading order, not network order |

### Navigation & menus — [`families/17`](families/17-navigation-menus.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `nav.hide-on-scroll` | JS + CSS | free | The header retracts scrolling down, returns scrolling up |
| `nav.overlay-menu` | timeline lib | cheap | A panel covers the viewport, its links arriving in sequence |
| `nav.dropdown` | Popover API | free | A small panel appears beneath its trigger |
| `nav.mega-menu` | CSS + intent | cheap | A full-width panel of grouped links expands |
| `nav.drawer` | dialog/CSS | free | A panel slides in from an edge over the content |
| `nav.command-palette` | dialog | cheap | A search-driven launcher appears on a keystroke |
| `nav.active-indicator` | CSS + JS | free | A marker travels to the current nav item |
| `nav.breadcrumb-collapse` | CSS | free | A long trail folds into an ellipsis |
| `nav.tab-bar` | CSS | free | Mobile tab icons animate as sections change |
| `nav.scroll-spy` | IO | free | The sidebar marks which section you are reading |
| `nav.skip-link` | CSS | free | A hidden link appears on focus to jump past navigation |
| `nav.sticky-subnav` | CSS sticky | free | A secondary bar detaches and pins as you scroll past |

### Forms & input — [`families/18`](families/18-forms-input.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `form.focus-field` | CSS | free | The field acknowledges that it is the one you are typing in |
| `form.floating-label` | CSS | free | The placeholder rises and shrinks to become the label |
| `form.inline-validation` | CSS | free | The field reports in place whether its contents are acceptable |
| `form.step-progress` | any | cheap | A multi-step form advances, showing how far along you are |
| `form.submit-resolve` | CSS | free | The submit button becomes the outcome |
| `form.autocomplete` | Popover/listbox | cheap | Suggestions appear and filter as you type |
| `form.char-counter` | CSS | free | A live count reacts as the limit approaches |
| `form.password-strength` | CSS | free | A meter fills and changes colour as a password improves |
| `form.conditional-field` | CSS | free | Answering one question reveals dependent fields |
| `form.otp-entry` | JS | free | A row of single-character boxes advances itself |
| `form.range-slider` | native input | free | A handle drags along a track, value following |
| `form.autosave` | debounce | free | The form quietly reports that it has kept your work |

### Loading, progress & streaming — [`families/19`](families/19-loading-progress-streaming.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `load.token-stream` | streaming API | cheap | Generated text appears progressively as it is produced |
| `load.thinking` | CSS | free | Three dots pulse while a response is prepared |
| `load.staged-reveal` | server streaming | free | Each part shows as it becomes ready |
| `load.determinate` | CSS | free | A bar reports actual progress toward a known total |
| `load.indeterminate` | CSS | free | A segment travels when the duration is unknown |
| `load.skeleton` | CSS | free | The shape of the content stands in for the content |
| `load.optimistic-item` | state | free | Your submission appears immediately, visibly provisional |
| `load.infinite-scroll` | IO | cheap | More content fetches as you approach the end |
| `load.stall-escalate` | timers | free | A long wait changes what it says the longer it runs |
| `load.progressive-hydrate` | framework | free | Static content becomes interactive in priority order |
| `load.upload` | progress events | free | A file being uploaded reports its own progress in place |
| `load.refresh-pulse` | CSS | free | Visible data indicates quietly that it is updating |

### Overlays — [`families/20`](families/20-overlays.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `overlay.modal-dialog` | `<dialog>` | free | A dialog scales up while the page recedes |
| `overlay.top-layer-exit` | CSS | free | The technique that makes a closing overlay's exit visible |
| `overlay.backdrop` | `::backdrop` | free | The page behind darkens to demote itself |
| `overlay.popover-anchored` | Popover + anchor | free | A panel appears attached to the control that opened it |
| `overlay.tooltip` | Popover | free | A small label appears after you rest on something |
| `overlay.bottom-sheet` | dialog + drag | cheap | A panel rises from the bottom edge on a phone |
| `overlay.context-menu` | Popover | free | A menu appears exactly where you invoked it |
| `overlay.snackbar` | Popover | cheap | A brief message with an undo, anchored to the bottom |
| `overlay.confirm-destructive` | `<dialog>` | free | The dialog that deliberately does not make it easy |
| `overlay.consent-banner` | CSS | free | The compliance panel that appears before anything else |
| `overlay.nested-stack` | top layer | free | An overlay opened from another, without losing the first |
| `overlay.focus-return` | JS | free | Closing an overlay puts you back exactly where you were |

### E-commerce & conversion — [`families/21`](families/21-ecommerce-conversion.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `commerce.add-to-cart` | motion path | cheap | The product visibly travels to the cart |
| `commerce.variant-swap` | CSS | free | Choosing a colour or size updates imagery in place |
| `commerce.price-update` | CSS | free | The price changes visibly when your selection changes it |
| `commerce.quantity-stepper` | CSS + timer | free | Plus and minus with the number responding between |
| `commerce.cart-drawer` | dialog | free | The cart slides in without leaving the page |
| `commerce.filter-update` | FLIP | cheap | Applying a filter rearranges rather than reloads |
| `commerce.sticky-buy-bar` | IO + CSS | free | The purchase control follows you down a long page |
| `commerce.payment-processing` | CSS | free | The pay button becomes a progress state, then an outcome |
| `commerce.order-confirm` | SVG | free | The moment of confirmation, marked deliberately |
| `commerce.wishlist-toggle` | CSS | free | A heart or bookmark fills with a small flourish |
| `commerce.urgency-timer` | timer | free | (!) Countdown or scarcity pressure — see the entry |
| `commerce.trust-reveal` | CSS | free | Reassurance appears at the moment doubt does |

### Onboarding & empty states — [`families/22`](families/22-onboarding-tours-empty-states.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `onboard.empty-state` | CSS | free | The screen with nothing in it introduces itself |
| `onboard.coach-mark` | Popover | free | A tooltip points at one control and explains it |
| `onboard.spotlight` | clip-path | free | Everything dims except the one thing being pointed at |
| `onboard.tour-step` | tour lib | cheap | The tour moves from one point of interest to the next |
| `onboard.checklist` | CSS | free | A setup list ticks itself off as you complete things |
| `onboard.hotspot` | CSS | free | A small pulsing dot marks something new |
| `onboard.sample-data` | components | cheap | The empty screen fills with example content |
| `onboard.contextual-hint` | CSS + state | free | A short explanation the first time you reach something |
| `onboard.progressive-reveal` | state | free | Advanced controls appear once the basics are in use |
| `onboard.demo-loop` | video/Lottie | cheap | A short looping animation shows the feature in use |
| `onboard.completion` | CSS | free | Finishing setup is marked once, then never again |
| `onboard.skip` | CSS | free | The visible, permanent way out of being taught |

### Tables, lists & grids — [`families/23`](families/23-tables-lists-grids.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `table.sort-reorder` | FLIP | cheap | Re-sorting slides rows to their new positions |
| `table.row-expand` | CSS | free | A row opens downward to reveal its detail |
| `table.sticky-header` | CSS sticky | free | Column headers stay put while the body scrolls |
| `table.selection-bar` | CSS | free | Selecting rows summons a bar of actions |
| `table.inline-edit` | CSS | free | A cell becomes editable in place |
| `table.column-resize` | pointer events | free | Dragging a column edge changes its width live |
| `table.virtual-scroll` | TanStack/react-window | moderate | Only the visible rows exist, invisibly |
| `table.skeleton-rows` | CSS | free | Placeholder rows hold the table's shape while loading |
| `table.live-update` | CSS | free | A row's values change while you are looking |
| `table.pagination` | CSS | free | Moving between pages, deliberately undramatic |
| `table.scroll-shadow` | CSS | free | A shadow at the edge shows there are more columns |
| `table.group-collapse` | CSS | free | Grouped rows fold away under their heading |

### Theme & appearance — [`families/24`](families/24-theme-appearance.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `theme.mode-switch` | CSS vars | free | Light becomes dark, everywhere, at once |
| `theme.no-flash` | inline script | free | The correct theme applied before the first pixel |
| `theme.circular-reveal` | View Transitions | free | The new theme expands outward from the toggle |
| `theme.toggle-icon` | SVG | free | The sun becomes a moon |
| `theme.system-sync` | media query | free | The interface follows the OS, and changes when it does |
| `theme.accent-change` | `@property` | free | The user picks a colour and the interface adopts it |
| `theme.density-switch` | CSS vars | free | The interface tightens or relaxes its spacing |
| `theme.text-scale` | rem sizing | free | The user makes everything bigger and the layout copes |
| `theme.contrast-mode` | media query | free | The interface switches to maximum legibility |
| `theme.preview-swatch` | scoped vars | free | Hovering a theme shows it before committing |
| `theme.seasonal` | token layer | free | A temporary skin over the normal appearance |
| `theme.media-swap` | `<picture>` | free | Images and illustrations change with the theme |

### Playful, reward & delight — [`families/25`](families/25-playful-reward-delight.md)

| id | Stack | Cost | One line |
|---|---|---|---|
| `play.confetti` | canvas | cheap | Coloured pieces erupt and fall to mark a completion |
| `play.streak` | CSS + state | free | (!) A running count of consecutive days — see the entry |
| `play.badge-unlock` | CSS/Lottie | cheap | An achievement revealed with a moment of ceremony |
| `play.task-complete` | CSS + FLIP | free | A small, proportionate acknowledgement of finishing |
| `play.easter-egg` | any | free | A hidden reaction for someone who went looking |
| `play.mascot` | Rive/Lottie | moderate | A character responds to what you are doing |
| `play.variable-reward` | any | free | (!) Slot-machine mechanics — documented to be refused |
| `play.level-up` | CSS | free | Crossing a threshold is marked with escalation |
| `play.error-personality` | any | free | The 404 page is the one place to be funny |
| `play.idle-personality` | timers | free | The interface has small habits when nothing happens |
| `play.haptic-pairing` | Vibration API | free | A physical tap accompanies the visual response |
| `play.surprise-reveal` | canvas | cheap | Something concealed, uncovered by the user |

### Motion systems — [`motion-systems.md`](motion-systems.md)

Not a family file. Tokens, naming, specs, review, testing and versioning — the layer that makes
the entries above behave as one language. Written as a document because its content is practice
rather than animation; see the note in its header.


---

## Decision tree

Start from the feeling, not the technique.

### "I want the page to feel calm and expensive"

Slow, few, coordinated. → `entrance.mask-rise` on headings · `entrance.fade-rise` for
everything else · `scroll.reveal-enter` with a gentle threshold · `pointer.underline-draw` ·
one `ambient.gradient-drift`. Share one easing family. **Resist** adding a second ambient
effect — that is the move that turns expensive into busy.

### "I want it to feel fast and responsive"

Short durations and immediate acknowledgement. → `micro.press` everywhere ·
`micro.optimistic` for reversible actions · `micro.delayed-spinner` so fast paths show nothing
· `route.prerender-instant` · `physics.spring-settle` on panels. Nothing over 300ms.

### "I want the product to feel physical"

Springs and direct manipulation. → `gesture.drag` with `gesture.throw-inertia` ·
`gesture.rubber-band` at limits · `physics.spring-settle` on every arrival ·
`pointer.magnetic` on primary actions · `micro.toggle` with a stretch.

### "I want to draw the eye to one number"

→ `text.counter-odometer` or `dataviz.counter` · `svg.counter-arc` if a proportion is implied ·
`ambient.breathe` on the container if it must be found from across a room. One per screen.

### "I want to explain how something works"

Sequence controlled by the reader. → `svg.diagram-build` scrubbed with `dataviz.time-scrub` ·
`scroll.pin-sequence` for staged steps · `3d.exploded-assembly` if it is a physical object ·
`entrance.rule-draw` to structure the sections.

### "I want it to feel like an app, not a website"

Continuity across changes. → `layout.shared-element` · `route.persistent-shell` ·
`route.crossfade` as the baseline · `layout.list-add-remove` · `micro.focus-ring`.

### "I want the type to be the whole design"

→ `text.char-cascade` on one hero only · `text.mask-rise` for everything else ·
`text.scrub-highlight` on the statement paragraph · `ambient.marquee` as a band ·
`text.variable-weight` if the typeface supports it. One per screen.

### "I want a portfolio that looks like the studios I admire"

The house style, honestly labelled: → `entrance.hero-sequence` · `entrance.curtain` as a
preloader exit · `scroll.marquee-velocity` · `pointer.stack-swap` on project titles ·
`pointer.cursor-dot` (native cursor kept) · `scroll.clip-expand` on imagery ·
`pointer.image-follow` on the index. Recognisable — and recognisably a genre.

### "I want to show data honestly"

→ `dataviz.bar-grow` on entry · `dataviz.value-transition` keyed by identity ·
`dataviz.highlight-focus` for dense charts · `dataviz.sort-reorder`. Read the honesty test in
[`cross-cutting.md`](cross-cutting.md) first.

### "I have a performance budget of nearly zero"

Everything free and compositor-only: → `entrance.fade-rise` · `scroll.progress-bar` (CSS
scroll-timeline) · `pointer.underline-draw` · `micro.press` · `layout.view-transition-auto` ·
`route.crossfade`. A complete, polished motion system for 0 KB.

### "I need it to work for everyone, first"

→ `micro.press` · `micro.focus-ring` · `micro.field-validation` · `micro.delayed-spinner` ·
`route.scroll-restore` · `3d.fallback` if 3D is involved. This list is the one that is not
optional.

### "I am building a data-heavy tool"

Stability over delight. -> `table.sticky-header` for orientation · `table.virtual-scroll` past
1,000 rows · `table.live-update` without re-sorting · `form.inline-validation` ·
`micro.focus-ring` · `load.skeleton`. Read the row-count ceiling in `families/23` first.

### "I am building an AI or streaming interface"

-> `load.token-stream` as the baseline · `load.thinking` before the first token ·
`load.stall-escalate` past a few seconds · `micro.optimistic` for reversible actions.
Time-to-first-anything is the metric, not total duration.

### "I want to be sure I am not manipulating anyone"

Read these before shipping: `commerce.urgency-timer` · `play.streak` ·
`play.variable-reward` · `ambient.auto-carousel` · `overlay.consent-banner`. Each documents
what the 2026 regulatory position is and why the pattern is on the list.

---

## Conflict matrix

Pairs the catalogue advises against combining, with the reason.

| A | B | Why |
|---|---|---|
| `scroll.smooth-normalise` | `scroll.snap-sections` | Two systems deciding where the page rests; oscillation |
| `scroll.smooth-normalise` | `3d.scroll-camera` (damped) | Two layers of smoothing; floaty lag that reads as broken |
| `scroll.velocity-skew` | `pointer.skew-move` | Compounding skews; visual mush and softened text |
| `scroll.marquee-velocity` | hover-pause on the same marquee | Competing timescale writers unless one explicitly overwrites |
| `text.mask-rise` | `text.char-cascade` (same element) | Two splits on one node; the second destroys the first |
| `text.scrub-highlight` | any other split effect (same element) | Same DOM-rewrite collision |
| `scroll.parallax-layer` | `scroll.clip-expand` (same element) | Double transform; positions drift |
| `scroll.parallax-layer` | `pointer.magnetic` (same node) | Both write translate; last writer wins |
| `scroll.pin-sequence` | nested `scroll.pin-sequence` | Parent and child triggers fight over the same scroll |
| `scroll.horizontal-rail` | pin or snap on the container animation | Not available on containerAnimation triggers |
| `layout.*` (sticky) | transform-based smooth scroll | A transformed wrapper breaks `position: sticky` |
| `route.crossfade` | `route.curtain` | Two page-transition owners |
| `ambient.cursor-trail` | `pointer.cursor-dot` | Two cursor objects; both obscure what is beneath |
| `ambient.*` × 2+ | each other | Every ambient loop is a permanent cost; one per page |
| any `pointer.*` | touch-only context | No pointer exists; gate on `(any-hover: hover)` |
| `micro.ripple` | non-Material design language | Carries a strong system association |
| `dataviz.*` transitions | index-keyed data joins | Elements silently change meaning; the constancy failure |

---

## How to read the cost column

**free** — CSS or platform API only. No bundle cost, and usually compositor-only at runtime.
**cheap** — a few KB, or a library the page already loads for other reasons.
**moderate** — a dedicated library or plugin, roughly 10–40 KB gzipped.
**heavy** — brings its own rendering stack, asset pipeline or continuous main-thread work.

A free effect can still be expensive at runtime — `entrance.blur-focus` and `svg.filter-distort`
are both free to download and among the most expensive things here to render. Check the
**Performance** field on the entry, not just this column.
