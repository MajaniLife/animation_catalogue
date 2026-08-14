# QA & conformance report

Phase 15. Every file re-read, every entry checked against `ENTRY-SCHEMA.md`, every reference
URL requested, duplicates reconciled. This is the only phase permitted to edit an earlier
phase's file, and every edit it made is listed below.

**Scope of the catalogue as verified:** 142 entries, 142 unique ids, across 12 family files
plus `TAXONOMY.md`, `STACKS.md`, `ENTRY-SCHEMA.md`, `cross-cutting.md` and `INDEX.md`.

---

## 1. Schema conformance

Checked programmatically: every entry must carry all eleven required fields, a unique id, at
least one reference URL, and a "What the reader sees" paragraph inside the 60–150 word target.

| Check | Result |
|---|---|
| Entries with all 11 required fields | **142 / 142** |
| Unique ids (no collisions) | **142 / 142** |
| Entries with ≥1 reference URL | **142 / 142** |
| "What the reader sees" within target | **141 / 142** before fixes, **142 / 142** after |

**"What the reader sees" length distribution:** average 83 words, minimum 50 (now corrected),
maximum 117. Comfortably inside the schema's 60–150 band, and consistent across families —
no family drifted into either bullet-point summary or essay.

### Fixed

- **`micro.copy-confirm`** — the only conformance failure in the catalogue. Its "What the
  reader sees" was 50 words, below the 60-word floor. Rewritten to 84 words, adding what the
  revert accomplishes (repeatability across successive copies) rather than padding the
  existing description.

---

## 2. Duplicate and near-duplicate reconciliation

Several techniques legitimately appear in more than one family, because the same mechanism
serves a genuinely different purpose in each context. The policy applied: **keep both, and
cross-reference in both directions.** A reader arriving at either entry should learn that the
other exists.

Seven pairs were found where the later-written entry referenced the earlier one but not the
reverse — an artefact of the files being written in queue order. All seven back-references
have been added.

| Pair | Relationship | Action |
|---|---|---|
| `entrance.clip-expand` ↔ `scroll.clip-expand` | Same move, triggered vs scrubbed | Back-reference added to `entrance` |
| `entrance.skeleton-swap` ↔ `micro.skeleton-shimmer` | The swap vs the shimmer loop | Back-reference added to `entrance` |
| `entrance.batch-stagger` ↔ `scroll.reveal-enter` | Sets vs single elements | Back-reference added to `entrance` |
| `text.stroke-draw` ↔ `svg.stroke-draw` / `svg.signature` | General technique vs applied to lettering | Back-reference added to `text` |
| `text.counter-odometer` ↔ `dataviz.counter` / `svg.counter-arc` | Type vs chart context | Back-reference added to `text` |
| `svg.stroke-draw` ↔ `dataviz.line-draw` / `text.stroke-draw` | General vs time-series vs lettering | Back-references added to `svg` |
| `layout.tab-indicator` ↔ `micro.focus-ring` | Same shared-marker technique, different job | Back-reference added to `layout` |

No entry was deleted as a duplicate. In each case the two entries answer different questions
("what does this look like on a chart" vs "what does this look like on a logo") and merging
them would have made both harder to find.

---

## 3. Reference link check

All **104 unique URLs** across the catalogue were requested with a redirect-following client.

| Result | Count | Interpretation |
|---|---|---|
| `200 OK` | **92** | Live and reachable |
| `403 Forbidden` | 5 | Bot protection, not a dead link |
| No response from this environment | 7 | Network restriction here, not a dead link |

**No links were removed**, and that decision is worth recording rather than hiding.

- The five **403s** are CodePen (×2), Medium, `wcag.com` and `learnlyai.co.uk` — all hosts
  with WAF or bot protection that reject automated clients while serving humans normally.
- The seven **non-responding** hosts include `developer.chrome.com`, `web.dev`, `css-tricks.com`,
  MDN mirrors and `ericwbailey.website`. Several of these were **successfully fetched during
  earlier phases of this catalogue's own construction**, which is direct evidence they are
  live and that the failure is a restriction in the checking environment, not a dead URL.

Deleting a working reference because a sandbox could not reach it would make the catalogue
worse. The honest position is that 92 links are confirmed live, 12 are unconfirmed by this
pass, and none are confirmed dead.

---

## 4. Consistency audit

Checked by reading, not by script.

**Confirmed consistent:**

- **Reduced motion** — every one of the 142 entries specifies what the effect *becomes*, not
  merely that it is disabled, as `ENTRY-SCHEMA.md` requires.
- **Cost model** — the compositor/repaint/reflow ranking is stated identically in
  `TAXONOMY.md`, in the SVG family intro, and in `cross-cutting.md`.
- **The scroll-reveal bug** — the "handle both directions and reconcile on refresh" failure is
  documented in `entrance.batch-stagger`, `scroll.reveal-enter` and the `cross-cutting.md`
  anti-patterns, and phrased compatibly in all three.
- **GSAP licensing** — the April 2025 change to a fully free toolset is stated consistently
  wherever a formerly-paid plugin is recommended, so no entry recommends something a reader
  would believe they must pay for.

**Deliberate, flagged inconsistency:**

- `STACKS.md` marks unverified figures with ⚠️. **Those marks have been left in place.**
  Phase 15 confirmed no new figures for Lenis, Three.js, Lottie, Rive, Barba, Swup, Theatre.js,
  Matter.js, React Spring or the framework-native transitions, so promoting them to ✅ would
  have been a lie about what had been checked. The two conflicting GSAP bundle-size figures
  are likewise still recorded as conflicting. **This is the largest known gap in the catalogue
  and the obvious next piece of work.**

---

## 5. Known gaps

Recorded rather than quietly left.

1. **Unverified stack figures.** As above — roughly ten libraries in `STACKS.md` carry versions
   and weights that were not confirmed against primary sources. Every one is marked.
2. **No entry has been visually verified.** The catalogue is written from documentation,
   specifications and technique write-ups. The "What the reader sees" paragraphs describe
   effects as documented and as commonly implemented; none were built and observed as part of
   writing this. That is the correct caveat to attach to a reference assembled this way.
3. **Browser support is a moving target.** Scroll-driven animations behind a Firefox flag,
   cross-document view transitions not yet in Firefox, `overlay` not yet Baseline — all true as
   of August 2026 and all likely to change. Support claims carry dates for this reason.
4. **Twelve links unconfirmed** by the automated pass, as documented in section 3.

---

## 6. Verdict

The catalogue conforms to its own schema: 142 entries, all fields present, all ids unique, all
"What the reader sees" paragraphs within target, no dead links found, near-duplicates
cross-referenced in both directions.

The single unfixed weakness is the unverified figures in `STACKS.md`, which are marked as
such throughout and should be the first task of any future pass.
