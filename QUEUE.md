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

## Opened by the discovery pipeline

- [x] 28 Collaborative & presence motion — opened against gap `G01`. **In progress:** 1 entry of a 12 minimum. Do not treat as complete.
