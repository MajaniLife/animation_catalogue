# 3D & WebGL

Motion in a rendered scene rather than in the document. Cameras, geometry, materials, light,
shaders — a separate rendering stack running on the GPU, embedded in a page.

This family does not share a cost model with anything else in the catalogue. Everywhere else
the browser lays out and paints, and your job is to avoid making it do that twice. Here you
own a render loop, and the questions are draw calls, texture memory, fill rate and thermal
budget. An effect that is trivial in CSS can be impossible here, and vice versa.

**What changed by 2026.** WebGPU is now supported across Chrome, Edge, Firefox and Safari
including iOS, with 2–10× gains reported for draw-call-heavy scenes, compute workloads and
complex post-processing. Three.js has a WebGPU renderer, React Three Fiber can select it via
its renderer factory, and TSL (Three Shader Language) compiles the same shader source to both
WGSL and GLSL — which finally makes writing one shader for two backends practical. Compute
shaders are the genuinely new capability: particle systems and physics that were CPU-bound
can move to the GPU.

**The number that governs everything: draw calls.** Aim for **under 100 per frame**. Batching
— instancing, merging geometry, texture atlases — matters far more than triangle count. A
scene with 2 million triangles in 40 draw calls will outrun one with 200,000 triangles in 800.

**The rule for the web specifically.** A 3D scene on a marketing page is competing with the
page's own load, on hardware you do not choose, often on battery. The right question is
almost never "can we render this" but "what does the person on a three-year-old mid-range
phone get". Every entry below assumes a fallback exists.

---

### Scene entrance (`3d.scene-entrance`)

- **One line** — the 3D object arrives rather than simply being there when the canvas loads.
- **What the reader sees** — The canvas area is empty, then the object appears — usually
  fading up while the camera pulls back slightly and the object settles into its resting
  rotation over about a second. The light comes up with it. Because the arrival covers the
  moment the assets finish decoding and the first frame renders, you never see the hard cut
  from blank canvas to fully-lit object, which is what makes an unhandled 3D embed feel like
  a plugin from 2009.
- **Mechanism** — a timeline over camera position, object scale and material opacity, started
  once assets are loaded and the first frame is ready.
- **Stack** — Three.js or React Three Fiber, driven by any animation library or the render
  loop itself.
- **Params** — duration (0.8–1.5s); camera travel (small — a long dolly reads as a title
  sequence); whether light animates with the object.
- **Use when** — any embedded 3D object. This is the baseline courtesy of the family.
- **Don't use when** — the object is the page's primary content and the user came to
  manipulate it; get out of their way.
- **Reduced motion** — the object present at its resting state, lit, immediately.
- **Performance** — the entrance itself is trivial next to the cost of having a scene at all.
- **Gotchas** — start it on the first *rendered* frame, not on the load event, or the
  animation plays behind a blank canvas while shaders compile. Shader compilation stalls are
  the usual cause of a "frozen" first second; compile ahead of time where the renderer
  supports it.
- **References** — https://appscale.blog/en/blog/threejs-production-3d-web-2026-webgpu-realtime-standards

---

### Scroll-driven camera (`3d.scroll-camera`)

- **One line** — scrolling moves the camera through the scene.
- **What the reader sees** — You scroll and the view travels: past an object, around it,
  through an environment. Stop and the camera stops with you. Because the movement is mapped
  to scroll rather than played, you are steering — leaning forward through a product,
  reversing when you scroll back. It is the most common serious use of 3D on marketing sites
  and, when the path is well-chosen, the most effective: each scroll position is a composed
  shot.
- **Mechanism** — scroll progress mapped to a camera position along a path, usually with
  damping so the camera eases rather than snapping to each scroll delta.
- **Stack** — Three.js/R3F plus a scroll library; the camera follows a curve or a set of
  keyframed waypoints.
- **Params** — path; damping (0.05–0.15 — undamped camera motion is nauseating); field of
  view; how many waypoints (fewer and further apart than instinct suggests).
- **Use when** — product tours, spatial narratives, architectural walkthroughs.
- **Don't use when** — the reader needs to scan or search. Scroll-driven camera work makes
  skimming impossible.
- **Reduced motion** — **static camera at a representative position.** Camera movement through
  a 3D space is among the strongest vestibular triggers available in a browser; this branch is
  a safety feature.
- **Performance** — rendering every scroll frame means the scene must hold frame rate during
  the most expensive interaction on the page.
- **Gotchas** — damping and scroll-linking interact badly if both smooth: a smooth-scroll
  library plus a damped camera produces a floaty lag that feels broken. Pick one. Also cap
  the camera's travel per frame, or a fast flick teleports the viewer.
- **References** — https://www.krapton.com/blog/boosting-react-three-fiber-mobile-performance-in-2026-a-deep-dive-d6105c

---

### Object orbit (`3d.idle-orbit`)

- **One line** — the object turns slowly on its own.
- **What the reader sees** — A product rotates gently and continuously, one revolution every
  fifteen or twenty seconds, so slow that at any glance it appears almost still yet is always
  showing you a slightly different face. Light moves across it as it turns. It signals
  immediately that the object is a real three-dimensional thing rather than a photograph, and
  it invites you to touch it without instructing you to.
- **Mechanism** — a constant rotation applied per frame, usually paused on user interaction
  and resumed after a delay.
- **Stack** — a line in the render loop.
- **Params** — speed (0.05–0.2 rad/s); axis (Y almost always); resume delay after interaction
  (2–4s).
- **Use when** — a hero product, a configurator at rest, anything the user is meant to inspect.
- **Don't use when** — the object has a canonical front that must stay facing the reader.
- **Reduced motion** — static at the best-looking angle.
- **Performance** — the rotation is free; the cost is that it keeps the render loop running
  permanently, which on a laptop means the fan and on a phone means the battery. Pause when
  off-screen — this is the single most important optimisation in this family.
- **Gotchas** — a permanently animating canvas prevents the browser from idling, so a page
  with a 3D hero drains battery even when the user has scrolled far past it. An
  IntersectionObserver that stops the loop is not optional.
- **References** — https://www.utsubo.com/blog/threejs-best-practices-100-tips

---

### Shader transition (`3d.shader-transition`)

- **One line** — one image dissolves into another through a custom distortion.
- **What the reader sees** — Two photographs, and the transition between them is not a fade —
  the first image ripples, tears or melts into the second, pixels displaced along a noise
  pattern so the boundary between the two is organic and irregular. It takes under a second
  and it is unmistakably not something CSS could do. On a portfolio slider it is the single
  clearest signal that a site was custom-built.
- **Mechanism** — both textures bound to a fragment shader with a progress uniform; the shader
  mixes them using a displacement or noise map.
- **Stack** — Three.js or OGL with a plane and a custom shader; several small libraries wrap
  the pattern.
- **Params** — displacement map (this choice *is* the effect); intensity; duration (0.6–1s);
  easing on the progress uniform.
- **Use when** — an image slider or gallery that is a centrepiece.
- **Don't use when** — the images are content the user is scanning. This adds a second to
  every look.
- **Reduced motion** — a plain crossfade, or an instant swap.
- **Performance** — two full-resolution textures in memory per transition pair, and the
  transition is fill-rate bound — it is cheaper on a small element than a full-bleed one, the
  opposite of most intuitions.
- **Gotchas** — texture memory is the real constraint on mobile; a gallery of twenty
  full-resolution images will exhaust it. Use compressed textures (KTX2) and dispose of what
  is off-screen. Colour-space handling differs between renderers and is the usual cause of
  images looking washed out inside a shader but correct in an `<img>`.
- **References** — https://www.utsubo.com/blog/webgpu-threejs-migration-guide

---

### Instanced particle field (`3d.particle-field`)

- **One line** — thousands of small objects moving as one system.
- **What the reader sees** — A cloud of points or small shapes drifting in space — swirling
  around a centre, forming and dissolving a shape, reacting to the pointer by parting around
  it. Individually the elements are meaningless; collectively they read as a substance: dust,
  a swarm, a nebula, data. The density is what sells it, which is exactly why the naive
  implementation of one object per particle collapses immediately.
- **Mechanism** — a single instanced mesh or points geometry, with per-instance attributes
  updated either on the CPU or, increasingly, in a compute shader.
- **Stack** — Three.js `InstancedMesh` renders thousands of instances in **one draw call**.
  Under WebGPU, compute shaders move the simulation onto the GPU entirely, which is where the
  large reported gains for particle systems come from.
- **Params** — count (start at 1,000 and measure; 100,000 is achievable instanced, not
  otherwise); size; simulation forces; pointer influence radius.
- **Use when** — an ambient hero, a data-driven visual, one signature moment.
- **Don't use when** — you need it on low-end mobile. This is where phones thermally throttle
  first.
- **Reduced motion** — a static field, or no field.
- **Performance** — instancing is the whole game: **one draw call for the entire system**
  rather than one per particle. Keep total draw calls under 100 per frame across the scene.
  CPU-side per-particle updates become the bottleneck long before the GPU does, which is the
  argument for compute shaders.
- **Gotchas** — dispose of geometries, materials and textures explicitly on teardown; GPU
  resources are not garbage collected with the JavaScript objects, and this is the standard
  memory leak in single-page apps with 3D. Points at large sizes are fill-rate bound and will
  tank on mobile regardless of count.
- **References** — https://www.utsubo.com/blog/threejs-best-practices-100-tips ·
  https://www.utsubo.com/blog/threejs-2026-what-changed

---

### Mesh distortion on hover (`3d.mesh-hover-distort`)

- **One line** — an image plane deforms under the pointer.
- **What the reader sees** — Move over a project thumbnail and the image bulges toward the
  cursor as though the surface were elastic, ripples trailing slightly behind the pointer,
  settling back when you leave. The image is still perfectly readable — the distortion is
  gentle — but the flat rectangle now behaves like a membrane. It is the WebGL equivalent of
  the magnetic hover, and it belongs to the same design vocabulary.
- **Mechanism** — the image is a textured plane with a subdivided grid; a vertex shader
  displaces vertices by distance from the pointer, damped over time.
- **Stack** — Three.js or OGL; OGL is the sensible choice when this is the only 3D on the
  page, since Three is a large dependency for one effect.
- **Params** — subdivision (16–32 segments; more is smoother and costlier); strength; damping
  (the trailing settle is most of the character).
- **Use when** — a portfolio grid on a site already committed to a WebGL layer.
- **Don't use when** — it would be the only reason to load a 3D library. The bundle cost of a
  renderer for one hover effect is very hard to justify.
- **Reduced motion** — flat, undistorted plane.
- **Performance** — one plane per image, each with its own texture; a grid of twelve means
  twelve textures resident. Batch into an atlas or load progressively.
- **Gotchas** — matching the WebGL plane to the DOM position of the element it replaces
  requires syncing on scroll and resize, and any drift is immediately visible. Text over a
  distorting plane must stay in the DOM — rendering type into WebGL loses selection,
  accessibility and crispness.
- **References** — https://www.pkgpulse.com/guides/threejs-vs-react-three-fiber-vs-babylonjs-3d-webgl-2026

---

### Material state change (`3d.material-morph`)

- **One line** — the object's finish changes: colour, metalness, roughness.
- **What the reader sees** — Click a swatch and the product changes material — matte black
  becomes brushed steel — not by swapping images but by the surface itself changing, with
  reflections and highlights re-forming across it as the properties interpolate over half a
  second. Because the light and geometry are unchanged, the object stays absolutely
  continuous; only its material identity moves. For a configurator this is the entire selling
  point of using 3D at all.
- **Mechanism** — interpolate material uniforms (base colour, metalness, roughness) rather
  than swapping the material instance.
- **Stack** — any renderer; the animation is over numeric uniforms.
- **Params** — duration (0.4–0.6s); which properties change together; whether the environment
  map changes too.
- **Use when** — product configurators, customisation interfaces.
- **Don't use when** — the change also implies different geometry; then it is a model swap
  and needs a different treatment.
- **Reduced motion** — instant material change.
- **Performance** — uniform interpolation is nearly free. Swapping *materials* is not — it can
  trigger shader recompilation and a visible stall, which is exactly why you interpolate
  properties instead.
- **Gotchas** — shader recompilation on first use of a new material variant causes a hitch;
  warm them during load. Colour interpolation in the wrong colour space produces muddy
  intermediate values — interpolate in linear space, not sRGB.
- **References** — https://www.utsubo.com/blog/threejs-2026-what-changed

---

### Post-processing bloom & grade (`3d.post-processing`)

- **One line** — a full-screen pass adds glow, grain, aberration or colour grading.
- **What the reader sees** — Bright parts of the scene bleed light into their surroundings;
  edges carry a faint colour fringe; a fine grain sits over everything. The scene stops
  looking like a render and starts looking photographed. Animating the pass — bloom swelling
  at a moment of emphasis, grade shifting between sections — is what gives a 3D sequence
  cinematic rhythm.
- **Mechanism** — the rendered frame is drawn to a texture and passed through additional
  fragment shader passes before display.
- **Stack** — an effect composer in Three.js; WebGPU's compute path makes complex chains
  substantially cheaper.
- **Params** — bloom threshold and intensity; grain amount; chromatic aberration strength;
  number of passes.
- **Use when** — a scene that is the centrepiece and has the budget.
- **Don't use when** — targeting mid-range mobile. Post-processing is fill-rate bound and
  scales with screen area, so it is worst on exactly the devices with the highest pixel
  ratios.
- **Reduced motion** — keep static grading; do not animate the passes.
- **Performance** — each pass is another full-screen render. Two passes on a retina display
  is a lot of pixels; cap the device pixel ratio (1.5–2) rather than rendering at native 3×.
- **Gotchas** — capping DPR is the highest-value single optimisation in this family and the
  most commonly skipped. Bloom on a scene with pure-white UI elements makes text glow and
  become unreadable; exclude the UI layer from the pass.
- **References** — https://appscale.blog/en/blog/threejs-production-3d-web-2026-webgpu-realtime-standards

---

### Model assembly (`3d.exploded-assembly`)

- **One line** — components fly apart to show construction, then reassemble.
- **What the reader sees** — A finished product separates: the case lifts away, the internals
  spread outward along their assembly axes, each part labelled as it settles into its exploded
  position. Reverse it and the object rebuilds itself. It is the clearest possible way to show
  how something is made, and it is the reason engineering and hardware companies commission
  3D in the first place.
- **Mechanism** — per-part position offsets along authored axes, sequenced or scrubbed against
  scroll.
- **Stack** — the model must be authored with separable parts and sensible origins; the
  animation itself is simple translation.
- **Params** — explode distance per part; sequence order (usually outside-in); duration
  (1.5–3s for a full explode); scroll-scrubbed or played.
- **Use when** — hardware, machinery, anything whose interior is the story.
- **Don't use when** — the model is a single mesh. Retrofitting separability is a modelling
  job, not a code one.
- **Reduced motion** — a static exploded view, or a static assembled view with labels.
- **Performance** — same geometry as the assembled model; part count drives draw calls, so
  merge parts that always move together.
- **Gotchas** — labels must be DOM elements positioned over the canvas rather than 3D text,
  or they are unreadable and inaccessible. Part origins from CAD exports are frequently at the
  world origin rather than at each part's centre, which makes everything rotate wrongly — fix
  it in the asset.
- **References** — https://altersquare.io/blog/three-js-vs-webgpu-2026-large-scale-construction-viewers

---

### Fallback and progressive enhancement (`3d.fallback`)

- **One line** — what everyone who cannot run the scene gets instead.
- **What the reader sees** — On a capable machine: the full interactive scene. On a phone with
  limited memory, an older browser, or with reduced motion enabled: a still render of the same
  object at the same angle with the same lighting, occupying exactly the same space in the
  layout. The page does not shift, nothing is missing, and nobody is told they are getting a
  lesser version — because visually they largely are not.
- **Mechanism** — capability detection plus a pre-rendered image (or short video) at the same
  dimensions; the 3D layer replaces it once it is ready.
- **Stack** — a poster image exported from the same scene, so lighting and framing match
  exactly.
- **Params** — what triggers the fallback (no WebGL/WebGPU context, low device memory, reduced
  motion, save-data); whether the fallback is a still or a short loop.
- **Use when** — always. Every entry in this family needs one.
- **Don't use when** — never.
- **Reduced motion** — this *is* the reduced-motion branch, and the argument for building it
  first.
- **Performance** — the fallback is the fast path; on constrained devices it is dramatically
  better than the thing it replaces.
- **Gotchas** — export the poster from the actual scene at the actual camera position, or the
  swap from image to canvas visibly jumps. Reserve the space with an aspect ratio so the
  canvas arriving does not shift the layout. And treat "the WebGL context was lost" as a real
  runtime case — it happens on mobile under memory pressure — by falling back at that point
  too, rather than leaving a blank rectangle.
- **References** — https://www.krapton.com/blog/boosting-react-three-fiber-mobile-performance-in-2026-a-deep-dive-d6105c

---

## Family notes

**Draw calls, not triangles.** Under 100 per frame is the target. Instancing, merged geometry
and texture atlases move the needle; reducing polygon counts usually does not.

**Cap the device pixel ratio.** Rendering a post-processed scene at 3× on a phone is the
fastest way to melt it. 1.5–2 is almost always visually indistinguishable and can halve the
work.

**Dispose explicitly.** Geometries, materials, textures and render targets hold GPU memory
that is not released when the JavaScript object is collected. In a single-page app this is the
standard leak, and it ends in a lost context.

**Pause the loop when off-screen.** A permanently rendering canvas keeps the machine awake
whether or not anyone is looking at it. This one line of IntersectionObserver code is worth
more than most micro-optimisations in this file.

**WebGPU is available across major browsers now**, with the largest wins on draw-call-heavy
scenes, compute workloads and post-processing — but a WebGL path is still the safe floor, and
TSL exists precisely so one shader source can serve both.

**Keep type in the DOM.** Text rendered into WebGL loses selection, search, translation and
accessibility, and looks softer. Position HTML over the canvas instead.
