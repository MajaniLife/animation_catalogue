# Queue

One session takes the **first unticked line**, researches only that, writes only that
file, commits, and ticks it. Never two phases. Never skipped.

- [x] 00 Foundation — taxonomy, stacks, entry schema, queue
- [x] 01 Entrance & reveal
- [x] 02 Text & kinetic typography
- [x] 03 Scroll-driven
- [x] 04 Pointer, hover & cursor
- [x] 05 Layout & shared-element transitions (FLIP)
- [x] 06 Page & route transitions
- [x] 07 SVG & path animation
- [x] 08 3D & WebGL
- [x] 09 Physics, drag & gesture
- [x] 10 Micro-interaction & feedback
- [x] 11 Data-visualisation motion
- [x] 12 Ambient & decorative
- [x] 13 Cross-cutting — accessibility, performance, orchestration, anti-patterns
- [x] 14 Index & decision tree
- [x] 15 QA & conformance pass

## Expansion — v2

The first sixteen phases covered the motion *primitives*. This expansion covers the
*surfaces* those primitives get applied to — the recurring interface contexts where the
choice of effect is constrained by what the surface is for. Same rules: one session, one
phase, research before writing, own file only.

- [x] 16 Media & image motion
- [x] 17 Navigation & menu systems
- [x] 18 Forms & input
- [x] 19 Loading, progress & streaming states
- [x] 20 Overlays — modals, drawers, tooltips, popovers
- [x] 21 E-commerce & conversion motion
- [x] 22 Onboarding, tours & empty states
- [x] 23 Tables, lists & data grids
- [x] 24 Theme & appearance transitions
- [x] 25 Playful, reward & delight
- [x] 26 Motion systems — tokens, specs, handoff (document, not a family — see file header)
- [x] 27 Index & QA pass for the expansion

## Notes for the next session

- Phases 01–12 write `families/<nn>-<slug>.md`. Minimum 12 entries, aim for 18.
- Open each family file with a paragraph on what unites it and when it gets reached for.
- `STACKS.md` marks unverified claims with ⚠️. Phase 15 must confirm or correct every one
  of them; do not propagate an unverified number into a family file without checking it.
- Phase 15 is the only phase permitted to edit an earlier phase's file, and must log
  every edit in `QA-REPORT.md`.

## Raised by Pipeline 2 — `work/showcase/`, 2026-08-16

Filed rather than improvised around, per Stage 3's rule. The run was benchmarked against
`sondaven.com/en`; the finding was that the catalogue's **effects** cover that class of site
and its **registers** do not. Full argument in
[`PROPOSAL-2026-08-16-skin-layer.md`](PROPOSAL-2026-08-16-skin-layer.md).

- [ ] `G-S4` **The skin layer** — `mood:` → parameter deltas over a base entry, plus
  `Skin surface` / `Skin invariants` on every entry. Structural: not one entry. **Read the
  proposal before taking this line; it is roughly six sessions and it touches all 262 entries.**
- [ ] `G-S1` Display type measured to the viewport as a layout primitive — family 02
- [ ] `G-S2` Preloader → hero handoff as one composite — family 01
- [ ] `G-S3` An element scrubbed continuously across a section boundary — family 03
- [ ] `G-S5` **QA, not a gap.** `INDEX.md`'s conflict matrix bans `ambient.*` × 2+;
  `TAXONOMY.md`'s intensity ladder authorises "multiple simultaneous loops" at the maximal
  rung. Both cannot hold. A maximal design hits this immediately — `work/showcase/` did, and
  resolved it conservatively for itself only. Needs a decision at the catalogue level.

## Raised by Pipeline 2 — `work/adda/`, 2026-08-16 (Stage 8 revision of the title card)

Filed rather than improvised around. All three came out of one region — the opening of the Adda
deck — when the human asked for a typed wordmark and a better lighting treatment.

- [ ] `G-A1` **An illuminant ramp.** Nothing in 23 families has *a light source arriving in a
  scene* as its subject. The four nearest each miss on a different axis: `ambient.gradient-drift`
  is a permanent loop with no beginning, `pointer.spotlight` is pointer-driven and dies on touch,
  `onboard.spotlight` subtracts light for a tour step rather than adding it, and
  `3d.post-processing` brings a WebGL stack. Missing: a one-shot, **terminating** light with a
  stated off-frame origin, a falloff, a strike-and-settle curve, and everything it reaches
  responding on one timeline — ending, so a sequence can chain off it. Every dark-register site
  with a hero moment improvises this out of `background-clip: text`, which is exactly how the
  Adda deck arrived at a treatment its own client called visually poor. **Interim:**
  `ambient.gradient-drift` run once as a ramp. First sighting.
- [ ] `G-A2` **No conflict-matrix row for split text × `layout.shared-element` on one node.** The
  matrix covers split × split (a DOM-rewrite collision). It does not cover split × FLIP handoff,
  where the failure is quieter: the FLIP measures a container of per-character fragments instead
  of a text node and computes a wrong scale, and `aria-hidden` fragments ride the transition into
  the destination element. Not hypothetical — `work/adda/` had already recorded one wrong FLIP
  scale from measuring the wrong box. Proposed row: *`text.*` (split-based) × `layout.shared-element`
  (same node) — the split must be reverted before the handoff.*
- [ ] `G-A3` **`text.typewriter` and `text.char-cascade` overlap silently, and short strings fall
  through the gap.** `text.typewriter` carries a blanket "most dated effect in this file" and a
  Params set written for sentences (20–40 cps), with nothing for a 2–5 character wordmark — where
  that rate lands below the threshold at which motion is perceived at all. Meanwhile
  `text.char-cascade` documents the typed reading inside a parenthetical on its stagger line,
  where nobody choosing between the two will find it. Catalogue hygiene, not a missing
  capability, but it is why a designer asked for "typed" needed a paragraph to reach an answer
  the catalogue already contained.

Round 2 of the same revision added four more:

- [ ] `G-A4` **The ground that parts.** `load.title-card`'s Mechanism field names "two halves
  seamed on the type line" as one of its three exit shapes, and no entry anywhere describes it,
  prices it, or states its reduced-motion form. The nearest, `entrance.split-part`, is a
  concealment device — "panels removed" under reduced motion — and is the wrong thing. What is
  missing is a reversible, scroll-bound parting of a page's **own** ground.
- [ ] `G-A5` **The matrix reasons about behaviour and is written against ids.** `ambient.*` × 2+
  states its reason as "every ambient loop is a permanent cost; one per page". A design citing two
  `ambient.*` ids and running zero loops satisfies the reason and violates the letter, and there is
  no way to record that. `work/adda/` dropped `ambient.idle-nudge` on the letter while running no
  loops at all. Same class as `G-S5`; third sighting of the class.
- [ ] `G-A6` **No entry for an invitation on an untimed hold.** `ambient.idle-nudge` is the nearest
  and is written for onboarding, capped at two firings. A loading card that waits for the reader by
  design may wait longer than two nudges, and the entry has no guidance beyond "never indefinitely".
- [ ] `G-A1` **is a second sighting and is worse than first recorded.** `work/adda/` now runs an
  `ambient.*` entry *with its ambient switched off* — a field whose only motion is a one-way
  arrival — because the catalogue's only large soft light is defined as a loop. The entry needed is
  a light with an arrival **and** a steady state.
- `G-A2` **narrows.** Still live for a shared-element node whose split must be reverted before the
  FLIP measures its container; it does not apply to split text that travels with its surface. The
  gap is about shared-element nodes specifically, not about split text near a boundary.

## Opened by the discovery pipeline

- [x] 28 Collaborative & presence motion — opened against gap `G01`. **In progress:** 1 entry of a 12 minimum. Do not treat as complete.
