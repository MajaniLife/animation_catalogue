# Motion systems — tokens, specs, handoff

How to codify everything else in this catalogue so that a team produces one coherent motion
language instead of 261 individual decisions.

> **A note on format.** Every other phase in this catalogue is a family file of entries
> conforming to `ENTRY-SCHEMA.md`. This one is not, deliberately. The schema is built around
> **What the reader sees** — a description of an animation — and most of what follows is
> engineering and process practice: naming conventions, spec formats, review checklists,
> versioning. A token naming scheme has no appearance. Forcing these into an animation schema
> would have produced exactly the contorted writing the catalogue criticises, so this is a
> document rather than a family, and `INDEX.md` lists it as such.

---

## 1. Why tokens, and the standardisation gap

A motion system is three or four small numbers applied consistently. Without them, every
component invents its own duration and easing, and the result reads as several designers who
never spoke. With them, changing one value visibly changes the character of the whole product —
which is the test of whether the system is real.

**The state of standardisation, as of 2026.** The W3C Design Tokens Community Group published
the first stable version of the Design Tokens specification in **October 2025** — and that
version **does not cover motion tokens**. Colour, typography and spacing are standardised;
duration, easing, spring configuration and stagger are not.

So the landscape is: no interoperable standard, but strong convergence in practice. **Material
Design 3** ships duration and easing tokens (`md.motion.duration.short1` = 50ms and so on), and
**IBM Carbon**, **Fluent UI** and **Shopify Polaris** all publish motion tokens of their own.
Emerging formats such as MOTION.md exist specifically because the gap leaves code-generating
tools inventing defaults every time.

**What to do about it.** Model motion tokens on the DTCG structure even though motion is out of
scope — same file, same naming discipline, same build pipeline — so that when motion is
specified you are a rename away rather than a rewrite. DTCG is the format with momentum and the
one that survives a tool migration.

---

## 2. The token set

Five groups cover essentially everything in this catalogue.

### Duration

A small scale, not a free-for-all. Every value in the product comes from it.

| Token | Value | Used for |
|---|---|---|
| `duration.instant` | 100ms | Press feedback, hover states |
| `duration.fast` | 200ms | Micro-interactions, toggles, dropdowns |
| `duration.base` | 300ms | Most transitions, panels, layout moves |
| `duration.slow` | 500ms | Entrances, larger reveals |
| `duration.deliberate` | 800ms | Curtains, hero moments, page transitions |

Five steps is enough. If a component needs 250ms specifically, the question is whether the scale
is wrong, not whether to add a sixth value.

### Easing

Fewer than you think. Three curves plus a linear.

| Token | Curve | Used for |
|---|---|---|
| `easing.out` | `cubic-bezier(0.2, 0, 0, 1)` | Things arriving. The default. |
| `easing.in-out` | `cubic-bezier(0.4, 0, 0.2, 1)` | Things moving between states |
| `easing.in` | `cubic-bezier(0.4, 0, 1, 1)` | Things leaving |
| `easing.linear` | `linear` | Scroll-linked, time axes, marquees |

Cubic-bezier notation is the standard expression. The rule that matters more than the numbers:
**entrances decelerate, exits accelerate.** An ease-in on an entrance looks hesitant, and the
mistake is common enough that naming the tokens by *purpose* rather than by curve prevents most
of it.

### Distance

Travel is a token too, and it is the one teams most often forget.

| Token | Value | Used for |
|---|---|---|
| `distance.nudge` | 4px | Hover shifts, magnetic pull |
| `distance.rise` | 24px | Fade-rise entrances |
| `distance.enter` | 100% | Mask-rise, slide-ins (relative) |

Relative units where the element's own size should govern; absolute where it should not.

### Stagger

| Token | Value | Used for |
|---|---|---|
| `stagger.tight` | 40ms | List rows, words |
| `stagger.base` | 80ms | Cards, grid items, lines |
| `stagger.loose` | 120ms | Hero sequences, large sections |

With a rule attached: **cap the total.** A stagger of 80ms across forty items is 3.2 seconds of
arrival. Past about ten items, divide a fixed total rather than multiplying per item.

### Spring

For anything interactive (see `families/09`). Springs have no duration, so they need their own
group.

| Token | Config | Used for |
|---|---|---|
| `spring.snappy` | stiffness 400, damping 30 | Toggles, small controls |
| `spring.default` | stiffness 300, damping 26 | Panels, sheets, drag release |
| `spring.gentle` | stiffness 200, damping 26 | Large surfaces |

Library defaults (commonly around stiffness 100, damping 10) visibly oscillate; every value here
is closer to critically damped on purpose.

---

## 3. Naming

Name by **role**, never by value or appearance.

- `duration.fast` — good. Survives being retuned.
- `duration.200` — bad. The name lies the moment someone changes it to 180.
- `easing.out` — good. Says what it is for.
- `easing.bouncy` — risky. Describes a feeling that a future value may not deliver.

Follow the DTCG-style three-part shape — `category.role.variant` — so motion sits alongside
colour and spacing in the same file and the same build. Component-level aliases
(`button.press.duration → duration.instant`) are worth it in large systems: they let you retune
one component without touching the global scale, and they document intent at the point of use.

---

## 4. Reduced motion belongs in the tokens

The single most valuable structural decision in a motion system: **make the reduced-motion
branch a property of the token layer, not of every component.**

```css
:root {
  --duration-fast: 200ms;
  --duration-base: 300ms;
  --distance-rise: 24px;
}

@media (prefers-reduced-motion: reduce) {
  :root {
    --duration-fast: 1ms;
    --duration-base: 1ms;
    --distance-rise: 0px;
  }
}
```

Every component that reads the tokens is now compliant without knowing the preference exists.
This is how you avoid the usual outcome — a page that is 80% reduced because six components
forgot to check.

Two caveats. First, `1ms` rather than `0` keeps transition-end events firing, so state machines
that depend on them do not stall. Second, **this is not sufficient on its own**: JavaScript-driven
motion, canvas work, autoplaying video and looping ambient effects all need the runtime check
too, and effects that carry meaning need a designed static equivalent rather than a zeroed
duration. The token layer covers CSS, which is most of it, not all of it. This gap — motion
tokens with a proper reduced-motion fallback — is an open request in current tooling, not a
solved problem.

---

## 5. Writing a motion spec

The handoff artefact that actually works is short and answers six questions per interaction:

1. **Trigger** — what starts it (click, scroll position, state change, arrival).
2. **Property** — what animates, named exactly (`transform: translateY`, not "it moves up").
3. **From → to** — start and end values, in tokens where possible.
4. **Duration and easing** — as token names, never raw numbers.
5. **Sequence** — offsets and overlaps for anything with more than one element.
6. **Reduced motion** — the end state, and what replaces the travel.

Anything a developer would otherwise have to guess belongs in these six lines. Anything else is
usually decoration on the document rather than information.

**What prototypes lie about.** Design-tool prototypes are useful for *feel* and misleading about
almost everything else: they run on ideal hardware with assets preloaded, they do not model
interruption, they rarely show the reduced-motion branch, and they never show what happens when
the data is slow or the list has 400 rows. Treat a prototype as a reference for character, and
the spec as the contract.

---

## 6. Reviewing motion in code

A short checklist catches most of what goes wrong across this catalogue:

- **Tokens, not literals.** A raw `0.3s` or `cubic-bezier(...)` in a component is the system
  leaking. Grep for them; it is the fastest audit available.
- **`transform` and `opacity` only**, unless there is a documented reason. Layout properties
  animated per frame are the standard performance defect.
- **Both scroll directions, and refresh.** Every reveal must handle entering from below and
  reconcile on refresh — the single most common "the animation didn't run" bug.
- **Teardown exists.** Listeners removed, tweens killed, observers disconnected, splits
  reverted. In a single-page app the absence of teardown is a slow leak.
- **Setters created once.** No tween allocation inside a pointer or scroll handler.
- **Reduced-motion path verified**, not assumed — with the OS setting actually enabled.
- **Focus behaviour intact** for anything that opens, closes or moves.
- **Nothing meaningful lives only in the animation.** State changes announced, content reachable
  without motion.

---

## 7. Testing motion

Motion is testable, though rarely tested. In rough order of value:

- **Reduced-motion snapshot tests.** Render key screens with the preference forced and assert
  the end state is correct and reachable. This catches the most consequential class of bug for
  the least effort.
- **Interaction tests that do not wait on animation.** If a test needs an arbitrary sleep to
  find a button, real users are waiting too. Treat those sleeps as a design signal.
- **Performance budgets in CI.** INP and CLS thresholds on key journeys; animation regressions
  show up in INP before anyone reports them.
- **Visual regression on end states only.** Mid-animation frames are flaky by nature; assert
  where things land, never where they pass through.
- **A manual pass on the worst device you support**, with the CPU throttled. No automated check
  substitutes for this, and it is where blur, filters and particle effects are exposed.

---

## 8. Versioning and change

Motion tokens are a public interface once components consume them.

- **Retuning a value is a visual breaking change**, even though nothing breaks. Changing
  `duration.base` from 300ms to 200ms alters every transition in the product; it belongs in
  release notes, not in a drive-by commit.
- **Adding a token is cheap; removing one is not.** Deprecate with an alias for at least one
  release rather than deleting.
- **Component aliases absorb churn.** With `button.press.duration` pointing at
  `duration.instant`, a button-specific retune never touches the global scale.
- **Record the intent, not just the value.** The reason `easing.linear` exists for time axes
  (see `dataviz.line-draw`) is a decision that will otherwise be re-litigated annually.

---

## 9. A starter system

Complete, minimal, and enough for most products. Take it, rename it, retune it.

```css
:root {
  /* duration */
  --duration-instant: 100ms;
  --duration-fast:    200ms;
  --duration-base:    300ms;
  --duration-slow:    500ms;

  /* easing — named by purpose */
  --easing-out:    cubic-bezier(0.2, 0, 0, 1);
  --easing-in-out: cubic-bezier(0.4, 0, 0.2, 1);
  --easing-in:     cubic-bezier(0.4, 0, 1, 1);

  /* distance */
  --distance-nudge: 4px;
  --distance-rise:  24px;

  /* stagger */
  --stagger-tight: 40ms;
  --stagger-base:  80ms;
}

@media (prefers-reduced-motion: reduce) {
  :root {
    --duration-instant: 1ms;
    --duration-fast:    1ms;
    --duration-base:    1ms;
    --duration-slow:    1ms;
    --distance-nudge:   0px;
    --distance-rise:    0px;
    --stagger-tight:    0ms;
    --stagger-base:     0ms;
  }
}
```

Eleven values. If the product feels incoherent with these applied consistently, the problem is
not the token set.

---

## 10. The test that the system is real

Change `--easing-out` and `--duration-base` and reload.

**The character of the entire product should change — coherently, in one move.** If some parts
change and others do not, something is hardcoding a value it should be reading, and that
component is not in the system regardless of what the documentation says.

This is the only reliable audit, it takes thirty seconds, and it is worth running before every
release that touches motion.

---

## Sources

- https://www.w3.org/community/design-tokens/ — Design Tokens Community Group
- https://tasteprofile.io/blog/w3c-dtcg-design-tokens-practical-guide — DTCG stable version, October 2025, and its scope
- https://m3.material.io/styles/motion/easing-and-duration/tokens-specs — Material 3 duration and easing tokens
- https://carbondesignsystem.com/elements/motion/overview/ — IBM Carbon motion
- https://hopper.workleap.design/tokens/core/motion — a published core motion token set
- https://www.designmd.co/blog/motion-md-format — the motion gap in code-generation formats
- https://github.com/google-labs-code/design.md/issues/47 — motion tokens with reduced-motion fallback as an open request
- https://wwnorton.github.io/design-system/docs/foundations/motion/ — motion foundations in a published system
