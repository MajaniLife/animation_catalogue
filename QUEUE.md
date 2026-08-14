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
- [ ] 20 Overlays — modals, drawers, tooltips, popovers
- [ ] 21 E-commerce & conversion motion
- [ ] 22 Onboarding, tours & empty states
- [ ] 23 Tables, lists & data grids
- [ ] 24 Theme & appearance transitions
- [ ] 25 Playful, reward & delight
- [ ] 26 Motion systems — tokens, specs, handoff
- [ ] 27 Index & QA pass for the expansion

## Notes for the next session

- Phases 01–12 write `families/<nn>-<slug>.md`. Minimum 12 entries, aim for 18.
- Open each family file with a paragraph on what unites it and when it gets reached for.
- `STACKS.md` marks unverified claims with ⚠️. Phase 15 must confirm or correct every one
  of them; do not propagate an unverified number into a family file without checking it.
- Phase 15 is the only phase permitted to edit an earlier phase's file, and must log
  every edit in `QA-REPORT.md`.
