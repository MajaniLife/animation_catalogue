# Media & image motion

Motion applied to photographs, video and galleries — the surface where an animation decision
is also a **loading** decision, and where getting the second one wrong makes the first one
irrelevant.

Everywhere else in this catalogue, an effect either runs or it does not. Here, the asset may
not have arrived yet, may arrive at an unexpected size, may be several megabytes on a metered
connection, and is frequently the Largest Contentful Paint element the page is being measured
on. A hero image with a beautiful reveal that delays LCP by 400ms has made the page worse.

**The three rules that constrain everything in this file:**

1. **Never lazy-load the LCP image.** The above-the-fold hero must carry `loading="eager"` and
   `fetchpriority="high"`; removing a lazy-load and adding the priority hint has been measured
   at **200–800ms of LCP improvement** on image-heavy pages. Lazy-load everything *below* the
   fold, which helps the LCP image by removing bandwidth contention.
2. **Reserve the space.** `width`/`height` or `aspect-ratio` on every image. An image that
   arrives and pushes the layout is a CLS failure, and no entrance animation redeems it.
3. **Placeholders are for a handful of images, not all of them.** LQIP applies to the hero and
   a few above-the-fold images; applying it to every thumbnail on a long page just ships a
   second set of assets.

---

### Blur-up load (`media.blur-up`)

- **One line** — a tiny blurred version stands in until the real image arrives.
- **What the reader sees** — Where a photograph will be there is already something: a soft,
  colour-accurate smear that has the shape and palette of the final image but none of its
  detail, like a photograph seen through frosted glass. When the real file arrives it
  crossfades in and the blur resolves into detail over a couple of hundred milliseconds. The
  page never has a grey hole in it and never jumps, and on a slow connection the reader can
  tell what the image is going to be before they can see it — which is often enough to decide
  whether to wait.
- **Mechanism** — a 10–20px-wide version of the image, inlined as base64 or preloaded, scaled
  up and blurred by CSS; the full image crossfades over it on load.
- **Stack** — framework image components ship this (Next.js `placeholder="blur"`); otherwise a
  build step to generate the tiny variant.
- **Params** — placeholder width (10–20px); blur radius (10–20px at display size); crossfade
  (200–300ms).
- **Use when** — hero images, above-the-fold editorial photography, anything large on a slow
  connection.
- **Don't use when** — every thumbnail in a grid. The placeholders are payload too.
- **Reduced motion** — crossfade instantly, or show the placeholder until the image simply
  replaces it. There is no travel to remove, so this one is nearly unaffected.
- **Performance** — **the placeholder must be in the initial HTML** (inlined base64, or
  preloaded) or it arrives no sooner than the real image and has bought nothing. Blur on a
  large scaled element is a real cost — keep it to the images that matter.
- **Gotchas** — the placeholder must match the final aspect ratio exactly or the crossfade
  visibly shifts. Do not lazy-load an LCP hero just because it has a nice placeholder — the
  placeholder does not count as the paint.
- **References** — https://cloudinary.com/blog/cloudinary-lqip-and-lazy-loading-best-practices ·
  https://www.tbmatuka.com/blog/blurry-lazy-loading-image-placeholders-lqip ·
  https://unlighthouse.dev/learn-lighthouse/lcp/lcp-lazy-loaded

---

### Ken Burns (`media.ken-burns`)

- **One line** — a still photograph slowly pans and zooms.
- **What the reader sees** — A photograph that is not moving, except that it is: over fifteen
  or twenty seconds it drifts almost imperceptibly across the frame while scaling up by a few
  percent, so at any instant it looks static and after a while the composition has changed.
  It gives a still image the presence of footage without the weight of video, and it is the
  standard treatment for a full-bleed hero that would otherwise be a wall.
- **Mechanism** — a slow `transform: scale()` and `translate()` on an image inside an
  `overflow: hidden` frame, usually alternating direction on a long loop.
- **Stack** — CSS keyframes.
- **Params** — scale range (1.0–1.08; more and the crop visibly changes); duration (15–25s per
  pass); direction.
- **Use when** — a hero image, a section backdrop, a single photograph carrying atmosphere.
- **Don't use when** — the image has a subject that must stay framed, or there is text over it
  whose contrast changes as the image moves.
- **Reduced motion** — static at the best-framed position. This is continuous motion and it
  stops.
- **Performance** — a permanent transform on a large raster. Compositor-friendly, but it keeps
  the page rendering — pause when off-screen.
- **Gotchas** — scale up from 1.0 rather than down to it, or the image is momentarily rendered
  below its natural resolution and looks soft. The image must be oversized relative to its
  frame or the pan reveals an edge.
- **References** — https://www.twicpics.com/blog/optimizing-the-image-element-lcp

---

### Before / after slider (`media.compare-slider`)

- **One line** — a draggable divider wipes between two versions of the same image.
- **What the reader sees** — One photograph with a vertical line down it and a handle on the
  line. Drag the handle and the image on one side is replaced by the other — the same frame,
  retouched or not, before the renovation or after — with the boundary tracking your finger
  exactly. Because both images occupy the identical frame and only the divider moves, the
  comparison is direct: your eye stays on one feature and watches it change.
- **Mechanism** — two stacked images, the top one clipped by a `clip-path: inset()` whose edge
  follows the pointer.
- **Stack** — CSS `clip-path` plus a pointer handler; a range input underneath makes it
  keyboard-operable for free.
- **Params** — handle size; whether it snaps to 50% on release; whether hover alone moves it
  (usually not — deliberate dragging reads better).
- **Use when** — retouching, restoration, product configuration, any genuine A/B of one frame.
- **Don't use when** — the two images are not the same frame. The whole comparison depends on
  everything except the变 subject being identical.
- **Reduced motion** — unaffected; this is direct manipulation, not animation.
- **Performance** — one clipped image; trivial. Both images must load before it is usable, so
  neither should be lazy.
- **Gotchas** — build it on a range input so it works with a keyboard and exposes a value to
  assistive technology; a pointer-only divider is unusable for a significant group. Label
  which side is which, since the divider alone does not say.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/clip-path

---

### Gallery lightbox (`media.lightbox`)

- **One line** — a thumbnail expands into a full-screen viewer and returns to its place.
- **What the reader sees** — Click a thumbnail in a grid; the surrounding page darkens as the
  image itself grows and moves to the centre of the screen at full size, arriving as the
  lightbox view. Arrow through the set and images slide horizontally. Close it and the image
  travels back to the exact thumbnail it came from, so the grid is where you left it and you
  know which one you were looking at. The return journey is what most implementations skip and
  what makes the difference between a viewer and a modal.
- **Mechanism** — a shared-element transition from the thumbnail's rectangle to the viewer's
  (see `layout.shared-element`), with a backdrop fade.
- **Stack** — View Transitions with a name applied to the clicked thumbnail only, or FLIP.
- **Params** — open duration (0.35–0.5s); backdrop fade (slightly longer); inter-image
  transition (0.25s slide or fade).
- **Use when** — photo galleries, portfolios, product imagery.
- **Don't use when** — the full-size images are heavy and unloaded; opening onto a spinner
  wastes the transition.
- **Reduced motion** — the viewer appears with a short fade, no travel.
- **Performance** — preload the neighbouring images once the lightbox opens, not all of them
  up front.
- **Gotchas** — focus must move into the lightbox and return to the originating thumbnail on
  close; it must trap focus and close on Escape. If the thumbnail is a different crop from the
  full image, the morph distorts — match aspect ratios or crossfade instead.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API

---

### Video scrub on scroll (`media.video-scrub`)

- **One line** — scroll position controls video playback time.
- **What the reader sees** — A product rotates, a process completes, a landscape changes
  season — and it does so exactly in step with your scrolling, forward and backward, stopping
  when you stop. It looks like video under your control. Where a frame sequence achieves this
  by decoding hundreds of images, this uses one video file, which is far lighter to download
  and considerably less reliable to scrub.
- **Mechanism** — map scroll progress to `currentTime` on a `<video>` element.
- **Stack** — the platform, plus a scroll library. See `scroll.frame-sequence` for the
  image-sequence alternative.
- **Params** — video duration mapped to scroll distance; keyframe interval in the encode
  (dense keyframes are what make seeking usable); resolution.
- **Use when** — the sequence is long enough that an image sequence would be unaffordable.
- **Don't use when** — smoothness is essential. This is the compromise option, and you should
  know that going in.
- **Reduced motion** — show a representative still frame.
- **Performance** — lighter to download than a frame sequence and worse to scrub: seeking is
  not frame-accurate, decoders stall on rapid seeks, and mobile browsers throttle or refuse
  programmatic seeking. Test on real devices before committing.
- **Gotchas** — encode with a very short keyframe interval or seeking lands on the wrong frame
  and stutters. Must be muted, `playsinline`, and preloaded. iOS in particular has historically
  restricted programmatic playback in ways that break this entirely — have a fallback.
- **References** — https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/currentTime

---

### Hover preview (`media.hover-preview`)

- **One line** — a thumbnail starts playing when you point at it.
- **What the reader sees** — A grid of video thumbnails, static. Move the pointer onto one and
  after a short pause it begins playing silently in place, a few seconds of the content, looping
  or scrubbing through highlights. Move away and it returns to its poster frame. You can survey
  a whole library by moving across it, and nothing plays until you express interest.
- **Mechanism** — a muted, looping, `playsinline` video swapped in over the poster on hover
  intent; often a short pre-generated preview clip rather than the full asset.
- **Stack** — the platform plus `pointer.hover-intent` for the delay.
- **Params** — hover delay (300–500ms — without it, sweeping across a grid starts six videos);
  preview length (3–6s); crossfade (150ms).
- **Use when** — video libraries, course catalogues, media browsers.
- **Don't use when** — on touch, where there is no hover. Provide an explicit play affordance
  instead.
- **Reduced motion** — do not autoplay on hover; require a click.
- **Performance** — the delay is the optimisation: it prevents a grid from fetching a dozen
  videos as the pointer passes. Fetch on intent, not on render.
- **Gotchas** — never play audio on hover. Preview clips should be separately encoded and tiny,
  not the full asset seeked into. Respect `prefers-reduced-data`.
- **References** — https://www.debugbear.com/blog/nextjs-image-optimization

---

### Progressive image reveal (`media.progressive-reveal`)

- **One line** — the image appears as it decodes, top to bottom or coarse to fine.
- **What the reader sees** — Instead of nothing then everything, the photograph resolves: a
  coarse version appears almost immediately and sharpens in a couple of passes as more data
  arrives. On a slow connection you watch it come into focus. It is the oldest image-loading
  behaviour on the web — progressive JPEG — and it remains one of the best, because the
  browser does it for free with no JavaScript and no second asset.
- **Mechanism** — encode as progressive JPEG or interlaced format; the browser handles the
  rest.
- **Stack** — an encoder setting. There is nothing to implement.
- **Params** — number of scans; encoder quality.
- **Use when** — large photographs on connections you do not control.
- **Don't use when** — images are small enough that the whole file arrives in one pass;
  progressive encoding costs a little size for no benefit there.
- **Reduced motion** — unaffected; this is decoding, not animation.
- **Performance** — free, no extra request, no placeholder payload. Often the better answer
  than a blur-up for images that are not the LCP element.
- **Gotchas** — modern formats do not all support progressive rendering the same way, so this
  interacts with your format choice. It is complementary to `media.blur-up`, not an
  alternative — use the placeholder for the hero, progressive encoding for everything else.
- **References** — https://www.tbmatuka.com/blog/blurry-lazy-loading-image-placeholders-lqip

---

### Carousel slide (`media.carousel-slide`)

- **One line** — a horizontal strip of images advances one item at a time.
- **What the reader sees** — Images in a row; press the arrow and the strip slides one position
  left, the next image settling into the frame with a slight deceleration. Drag it and it
  follows your finger, then snaps to the nearest item on release. Position dots below track
  which one you are on. It is the most familiar media pattern on the web, and the version that
  advances by itself is a different entry — `ambient.auto-carousel` — and a different argument.
- **Mechanism** — CSS scroll snap for the modern implementation, with buttons that scroll
  programmatically; or a transform-based track.
- **Stack** — `scroll-snap-type: x mandatory` plus `scroll-behavior`; no library needed for
  most cases.
- **Params** — snap alignment; transition (0.3–0.4s); whether partial next item is visible (it
  should be — a peeking edge is what tells people it scrolls).
- **Use when** — product galleries, related items, image sets too large for a grid.
- **Don't use when** — the content is important. Carousel items after the first are seen by a
  small fraction of visitors.
- **Reduced motion** — jump between items rather than smooth-scrolling.
- **Performance** — native scroll snap is free and gets touch momentum for nothing.
- **Gotchas** — the native scroll container gives you keyboard and screen-reader behaviour that
  a transform-based track has to reimplement badly. Show a peek of the next item; a carousel
  whose edge aligns exactly with the container looks like a static image.
- **References** — https://developer.mozilla.org/en-US/docs/Web/CSS/scroll-snap-type

---

### Image zoom on hover (`media.zoom-inspect`)

- **One line** — pointing at a product image magnifies the region under the cursor.
- **What the reader sees** — Move over a product photograph and the area under your pointer
  appears magnified — either in place, the image scaling under a fixed frame, or in a panel
  beside it showing a crop at full resolution. Move around and the magnified region follows.
  It answers "what does the fabric look like" without a page change, and for anything whose
  detail matters it converts a picture into an inspection.
- **Mechanism** — a high-resolution source positioned by `background-position` or a transform,
  driven by the pointer's relative position within the frame.
- **Stack** — CSS plus a pointer handler; the high-resolution asset is the real dependency.
- **Params** — zoom factor (2–3×); lens size; whether magnification appears in place or in a
  side panel.
- **Use when** — product detail pages for textiles, jewellery, print, anything tactile.
- **Don't use when** — you do not have a genuinely higher-resolution source. Scaling the
  displayed image just shows you larger blur.
- **Reduced motion** — unaffected; it tracks the pointer rather than animating.
- **Performance** — the high-resolution image should load on first hover intent, not with the
  page. It can easily be several megabytes.
- **Gotchas** — no equivalent exists on touch, so pair it with pinch-zoom or a tap-to-open
  full-screen view. The magnified panel must not cover the image it is magnifying.
- **References** — https://www.twicpics.com/blog/optimizing-the-image-element-lcp

---

### Aspect-ratio settle (`media.ratio-settle`)

- **One line** — the frame is correct before the image exists, so nothing moves.
- **What the reader sees** — Ideally, nothing at all. The space where an image will be is
  already exactly the right shape, holding a flat tone or a placeholder, and when the image
  arrives it simply fills that space. No reflow, no text jumping down the page, no losing your
  place mid-sentence because a photograph finished downloading. It is not an animation; it is
  the absence of one, and it is included because it prevents more visible motion than any
  effect in this file creates.
- **Mechanism** — `width` and `height` attributes, or a CSS `aspect-ratio`, on every image and
  video.
- **Stack** — HTML attributes. Nothing more.
- **Params** — none.
- **Use when** — always, on every image and embed.
- **Don't use when** — never.
- **Reduced motion** — this *is* motion reduction; unreserved space is unrequested movement.
- **Performance** — directly prevents CLS, one of the metrics the page is judged on.
- **Gotchas** — responsive images still need the intrinsic ratio declared; a `width: 100%`
  image with no dimensions collapses to zero height until it loads. Framework image components
  enforce this, which is most of why they exist.
- **References** — https://www.debugbear.com/blog/nextjs-image-optimization ·
  https://unlighthouse.dev/learn-lighthouse/lcp/lcp-lazy-loaded

---

### Masonry image reveal (`media.grid-reveal`)

- **One line** — a photo grid fills in as images arrive, in a controlled order.
- **What the reader sees** — A grid of photographs that is empty and then is not — images
  appearing in a wave from the top left, each fading up into its reserved space as it finishes
  loading, so the grid populates in reading order rather than in whatever order the network
  happened to deliver. The difference is significant: network order looks random and broken,
  reading order looks like the page is building itself.
- **Mechanism** — reserved cells, a load listener per image, and a queue that reveals in index
  order regardless of completion order — combined with a batched entrance
  (`entrance.batch-stagger`).
- **Stack** — a small queue plus any entrance animation.
- **Params** — stagger (0.06–0.1s); maximum wait before revealing out of order (~1s, so one
  slow image does not hold the grid).
- **Use when** — portfolio grids, galleries, anything where several images land together.
- **Don't use when** — the grid is long and mostly below the fold; below-fold images should
  lazy-load and reveal on scroll instead.
- **Reduced motion** — images appear as they load, no fade, no stagger.
- **Performance** — the ordering queue is free; the images are the cost. Lazy-load everything
  below the fold, and never the LCP image.
- **Gotchas** — the timeout matters: without it, one slow image stalls the whole wave. Reserve
  every cell before any image arrives, or the grid reflows as it fills, which is the exact
  problem the reveal was meant to hide.
- **References** — https://codefrog.app/quality-engineering/performance/lazy-loading

---

## Family notes

**Loading strategy outranks animation.** The LCP image is eager and high-priority; everything
below the fold is lazy; every frame is reserved. Get those wrong and no reveal will save the
page — get them right and a plain fade is enough.

**The placeholder is payload.** LQIP, poster frames and preview clips are all additional
bytes. They earn their place on the hero and the first screen, not on the fortieth thumbnail.

**Continuous media motion needs a stop.** Ken Burns, hover previews and looping video are all
subject to the same five-second rule as anything in `families/12`, and all should pause when
off-screen.

**Touch has no hover.** Zoom-on-hover and hover-preview simply do not exist on a phone. Each
needs a tap-based equivalent, not a degraded version of a pointer interaction.
