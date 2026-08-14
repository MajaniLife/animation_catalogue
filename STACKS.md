# Stacks

What to build these animations with. Researched August 2026.

**Verification status.** Facts marked ✅ were confirmed against sources fetched while
writing this file; those sources are listed at the bottom. Items marked ⚠️ are carried
from general knowledge and **have not been verified this pass** — the QA phase must
confirm or correct them. That distinction is deliberate: a catalogue that cannot tell you
which of its numbers were checked is not a reference.

---

## The platform

### CSS transitions and keyframes
**Cost: free · Verified: n/a (platform)**

The floor, and more capable than most reach for. Handles state-change motion, hover,
entrances via `@starting-style`, and keyframed loops. No orchestration primitive worth
the name — sequencing several elements means counting delays by hand, which is where
people give up and reach for a library.

Best at: micro-interaction, hover, anything with two states.
Bad at: sequences, mid-flight interruption, anything needing to read layout.

### CSS scroll-driven animations (`scroll-timeline` / `view-timeline`)
**Cost: free · ✅ Support confirmed**

Scroll-linked animation with no main-thread JavaScript, which is the entire argument for
it: it runs on the compositor and does not stutter when the main thread is busy.

Support as of mid-2026 ✅: Chrome/Edge 115+, Safari 18+ (landed Safari 26, September 2025;
threaded scroll-driven animations in 26.4; progress-accuracy fixes in 26.5, June 2026).
Firefox implements it **behind the `layout.css.scroll-driven-animations.enabled` flag** in
stable as of Firefox 152 (June 2026), on by default in Nightly. Approximately **84% global
support** ✅ — note that several 2026 blog posts describe support as "universal", which the
Firefox flag contradicts. Treat it as progressive enhancement, not a default.

Best at: progress bars, reading indicators, parallax, reveal-on-enter.
Bad at: anything needing a value read back into JavaScript, and Firefox-critical work.

### Web Animations API
**Cost: free · Verified: n/a (platform)**

The imperative sibling of CSS keyframes: real `Animation` objects you can pause, seek,
reverse and compose. The substrate Motion One is built on. Verbose enough that few people
write it directly, but it is the right answer when you need control and no dependency.

### View Transitions API
**Cost: free · ✅ Support confirmed**

Browser-native crossfade and shared-element morphing across a state change or a full
navigation. Removes the historical reason SPAs existed for transition purposes.

Same-document: broadly supported. Cross-document (MPA) ✅: Chrome/Edge 126+, Safari 18.2+
on macOS and iOS. **Firefox has not shipped cross-document**, and is in Interop 2026.
So: excellent progressive enhancement, not a guarantee.

Best at: route transitions on static and multi-page sites, list-to-detail morphs.
Bad at: fine easing control — you get much less choreography than a real timeline gives.

---

## Libraries

### GSAP 3.13
**Cost: moderate · ✅ Licence and plugin status confirmed**

The reference implementation for timeline-based animation. Sequencing, interruption,
overwrite management and a scroll plugin that is still the most capable option available.

The licence question is settled ✅: since Webflow's acquisition (announced April 2025,
effective 30 April 2025) **the entire toolset including the formerly paid Club plugins —
SplitText, MorphSVG, DrawSVG, ScrollTrigger, ScrollSmoother — is free, including for
commercial use**, and all plugins now ship in the main npm package and GitHub repository.
3.13 shipped a complete SplitText rewrite ✅. Any guidance telling you to configure an
`.npmrc` with a GreenSock auth token is obsolete and will break your build.

Weight ⚠️: core is commonly cited around 23–25 KB gzipped with ScrollTrigger adding
roughly 12 KB, but **exact figures were not confirmed this pass**. One comparison source
states Motion One is "one-seventh the size of GSAP" ✅, which implies a much larger figure
and is probably measuring the full bundle rather than the core — an example of why these
numbers need checking rather than quoting.

Best at: choreography, scroll-driven sequences, text splitting, anything where several
things must be coordinated.
Bad at: being small. If you need one fade, this is the wrong tool.

### Motion (formerly Framer Motion)
**Cost: moderate · ✅ Rebrand and size confirmed**

Rebranded from Framer Motion in mid-2025 ✅. Declarative React animation: layout
animations, gesture props, presence handling on unmount. The default in React work for
good reason — the ergonomics are unmatched inside a component tree.

Weight ✅: **30+ KB**, with roughly 3.6 million weekly downloads; core cited at ~18 KB
minified in one comparison, which is a different measurement basis — verify against your
own bundle rather than trusting either.

Best at: React component motion, layout and presence, gesture-driven UI.
Bad at: bundle-sensitive work; non-React contexts.

### Motion One
**Cost: cheap · ✅ Size confirmed**

WAAPI-backed, from the same author. `animate()` is **3.8 KB** ✅ — roughly half of
Anime.js and a fraction of GSAP ✅. Because it delegates to the browser, animations run
off the main thread where the browser allows, which is why it typically outperforms
heavier libraries on low-end mobile ✅.

Best at: the 90% case at minimum cost, mobile-first work.
Bad at: complex timelines and interruption logic — that is GSAP's territory.

### Anime.js v4
**Cost: cheap–moderate · ✅ Version confirmed**

v4 shipped in 2024 ✅, roughly **17 KB minified** ✅. Pleasant API, good SVG and stagger
support, genuinely capable timelines. Smaller ecosystem than GSAP and no equivalent of
ScrollTrigger.

### React Spring ⚠️
Physics-based springs for React. You tune tension and friction rather than durations,
which suits gesture-driven and interruptible motion and suits scripted choreography
badly. **Version and weight not verified this pass.**

### Svelte transitions / Vue Transition ⚠️
Built into the frameworks, zero additional dependency, and enough for entrances,
list transitions and presence. **Not verified this pass.**

### Lenis ⚠️
Smooth-scroll normalisation that wraps native scroll rather than replacing it, so sticky
positioning, anchors and accessibility survive. Pairs with GSAP's ScrollTrigger through a
single ticker. Note that smooth-scroll hijacking is itself an accessibility question —
some users find it disorienting, and it should honour reduced motion. **Version not
verified this pass.**

### Locomotive Scroll ⚠️
Older, heavier alternative in the same category, historically transform-based, which is
what breaks `position: sticky`. **Status not verified this pass.**

### Three.js / React Three Fiber / OGL ⚠️
Real 3D. A separate rendering stack with its own budget, its own asset pipeline and its
own failure modes; OGL is the minimal option. **Versions not verified this pass.**

### Lottie / Rive ⚠️
Designer-authored motion played at runtime. Lottie exports from After Effects and is
widely deployed; Rive uses its own editor with a state machine, which makes genuinely
interactive animation possible rather than just playback. Both move the work to a design
tool — the right call for illustration, the wrong one for interface state.
**Versions and weights not verified this pass.**

### Barba.js / Swup ⚠️
Pre-View-Transitions page-transition libraries: intercept navigation, animate out, swap
content, animate in. Still useful where you need choreography the View Transitions API
cannot express, or Firefox parity. **Status not verified this pass.**

### Theatre.js ⚠️
A visual sequencing editor producing a JSON timeline. Suits long, art-directed sequences
that would be miserable to hand-tune in code. **Status not verified this pass.**

### Matter.js ⚠️
2D rigid-body physics for genuine simulation — collisions, stacking, constraints — rather
than the spring easing most "physics" animation means. **Status not verified this pass.**

---

## Decision matrix

| What you want | Reach for first | Why |
|---|---|---|
| One fade or slide on state change | CSS | No dependency justifies it |
| Coordinated multi-element sequence | GSAP | Timelines are the whole product |
| Scroll progress bar, simple parallax | CSS scroll-driven | Compositor, free — but check Firefox |
| Complex pinned scroll sequence | GSAP ScrollTrigger | Nothing else is close |
| React component enter/exit | Motion | Presence handling is the hard part |
| React, bundle-constrained | Motion One | 3.8 KB for the common case ✅ |
| Interruptible gesture motion | Spring-based (React Spring / Motion) | Durations are wrong for gestures |
| Page transitions, modern browsers | View Transitions API | Free, native — Firefox is the gap |
| Page transitions, full support | Barba / Swup + a timeline library | Costs bundle, buys parity |
| Text splitting and per-character work | GSAP SplitText | Free since 2025 ✅, rewritten in 3.13 ✅ |
| Designer-authored illustration | Rive over Lottie | State machines beat playback |
| Real 3D | Three.js, or OGL if minimal | Separate budget entirely |
| Genuine collision physics | Matter.js | Springs are not simulation |

## The honest default

For most sites: **CSS for state changes, one animation library for choreography, nothing
else.** The common failure is not choosing a bad library — it is shipping three that each
animate a different part of the same page, arriving at 90 KB of motion code for effects
that CSS and one library would have covered.

---

## Sources fetched for this file

- https://gsap.com/blog/3-13/ — 3.13 release, SplitText rewrite
- https://webflow.com/blog/gsap-becomes-free — GSAP free including plugins, April 2025
- https://css-tricks.com/gsap-is-now-completely-free-even-for-commercial-use/ — commercial use
- https://motion.dev/blog/should-i-use-framer-motion-or-motion-one — Motion One vs Motion sizing
- https://blog.logrocket.com/exploring-motion-one-framer-motion/ — Motion One 3.8 KB
- https://gsapvault.com/blog/gsap-vs-animejs-vs-motion — Anime.js v4 ~17 KB, Motion core ~18 KB
- https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Scroll-driven_animations — scroll-driven reference
- https://developer.chrome.com/blog/scroll-triggered-animations — scroll-triggered animations
- https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API — View Transitions reference
- https://css-tricks.com/cross-document-view-transitions-part-1/ — cross-document support and gotchas
