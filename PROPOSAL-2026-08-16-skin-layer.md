# Proposal — the skin layer

**Status:** proposal, not adopted. Raised 2026-08-16 out of `work/showcase/`, which was run
against `sondaven.com/en` as a reference.
**Changes:** `ENTRY-SCHEMA.md`, every family file (one new field, mechanically derivable),
`tags/TAG-VOCABULARY.md` §2, `pipeline-2-design/BUILD-GUIDE.md` §2, and the Stage 3 agent.
**Does not change:** any entry's mechanism, any family boundary, the tag model, or the flow.

---

## 1. The gap, stated precisely

Asked whether the catalogue can build a site of `sondaven.com`'s class, the honest answer
splits in two:

**Animation-wise, yes, with headroom.** That site loads GSAP, ScrollTrigger, SplitText,
CustomEase, Flip, Lenis, Barba and Swiper — and **no WebGL, no Three.js, no shader library,
no physics engine.** Every move it makes maps to an existing entry. `INDEX.md`'s decision
tree already has a branch titled *"I want a portfolio that looks like the studios I admire"*
whose prescription is, effect for effect, that site. The catalogue anticipated the genre.

**Style-wise, no — and not because an effect is missing.** The catalogue can tell you *which*
entry to use for a `luxe` hero. It cannot tell you *how mask-rise must behave to read as
luxe rather than corporate*. The mechanism is identical in both. Only the numbers differ, and
the numbers are the one thing no document specifies.

Concretely: `entrance.mask-rise` gives its duration as "0.6–0.9 s" and its stagger as
"0.08–0.12 s". Every entry does this. **The register lives inside those ranges, and choosing
within them is left to taste.** So the catalogue reproduces a look once, by the designer
having good taste on the day, and cannot reproduce it twice — which is the exact failure
the pipeline exists to remove.

## 2. Why `--intensity` cannot close it

`BUILD-GUIDE.md` §2 has one scalar:

```css
--intensity: 1;   /* restrained: .5 | default: 1 | maximal: 1.4 */
```

It multiplies durations *and* distances together. But the registers are not points on one
line — they are opposite shapes:

| | duration | distance | easing | stagger | overshoot |
|---|---|---|---|---|---|
| `luxe` | **long** | **long** | heavy deceleration | **wide** | forbidden |
| `kinetic` | **short** | **long** | linear | **tight** | forbidden |
| `corporate` | short | **short** | symmetric | even | forbidden |
| `playful` | medium | long | spring | tight | **required** |

`luxe` and `kinetic` differ in duration by a factor of three while agreeing on distance.
One number cannot express that. Turning `--intensity` up on a corporate site produces a
corporate site that is louder — never a luxe one. That is the whole gap in one sentence.

`rung:` does not close it either. The intensity ladder in `TAXONOMY.md` varies **which
effects you may use**; it never varies **how a chosen effect is tuned**. Both axes are
needed and only one exists.

## 3. The catalogue already contains the answer, in prose

This is the argument for adopting rather than inventing. `TAG-VOCABULARY.md` §2 has a
column headed **"Typical motion signature"**:

| Tag | Typical motion signature |
|---|---|
| `editorial` | Mask reveals by line, hard easing, no bounce |
| `luxe` | Long durations, heavy deceleration, generous whitespace |
| `brutalist` | Instant cuts, hard clips, no easing curve at all |
| `technical` | Linear or stepped easing, numeric readouts, grid |

Those are parameter specifications written as English. The proposal is not to invent a
mapping — it is to **make the mapping the vocabulary already asserts into something a
build can apply.** If the table is right, it should compile. If it will not compile, the
table was decoration.

## 4. The proposal

### 4a. A skin is a set of deltas over the motion ladder

One file per mood, in a new `skins/` directory beside `catalogue/` and `tags/`. A skin
contains no effects, no selectors, and no mechanism — only multipliers and constraints.

```
skins/
├── README.md          ← what a skin may and may not contain
├── luxe.md            ← the rationale, in prose, with its evidence
├── luxe.json          ← the deltas, generated from the markdown
└── … one pair per mood: 12 in total
```

```jsonc
// skins/luxe.json
{
  "skin": "luxe",
  "duration": 1.8,          // × the ladder
  "distance": 1.6,
  "stagger": 2.0,
  "easing": "decel-heavy",  // names a curve in the easing set, never a bezier here
  "overshoot": "forbidden",
  "granularity": "line",    // ceiling: may split by line, never by character
  "ambient": 1,             // simultaneous continuous loops permitted
  "simultaneity": 1         // attention-directing effects per viewport
}
```

### 4b. The twelve skins

Derived from `TAG-VOCABULARY.md` §2's own signature column. Multipliers are against
`BUILD-GUIDE.md` §2's ladder, so `1.0` means "the ladder as written".

| skin | dur × | dist × | stagger × | easing | overshoot | split ceiling | ambient |
|---|---|---|---|---|---|---|---|
| `editorial` | 1.2 | 1.0 | 1.6 | decel-hard | forbidden | line | 1 |
| `brutalist` | 0.15 | 1.0 | 0 | none / step | forbidden | block | 0 |
| `playful` | 0.9 | 1.3 | 0.7 | spring | **required** | char | 1 |
| `corporate` | 0.8 | 0.5 | 1.0 | symmetric | forbidden | block | 0 |
| `luxe` | 1.8 | 1.6 | 2.0 | decel-heavy | forbidden | line | 1 |
| `technical` | 1.0 | 0.8 | 1.0 | linear / stepped | forbidden | char | 1 |
| `organic` | 1.5 | 1.2 | 1.4 ± jitter | spring-soft | slight | word | 2 |
| `retro` | 0.6 | 1.0 | 1.0 | `steps(n)` | forbidden | char | 1 |
| `minimal` | 0.7 | **0** | 1.0 | decel | forbidden | block | 0 |
| `maximal` | 1.3 | 1.5 | 0.6 | mixed | permitted | char | 2+ |
| `kinetic` | 0.5 | 1.8 | 0.4 | linear | forbidden | word | 1 continuous |
| `cinematic` | 1.6 | 1.4 | 1.8 | decel-heavy | forbidden | line | 1 |

Read the two extremes against each other. `luxe` and `kinetic` both travel far and differ
by 3.6× in duration and 5× in stagger. That difference is the entire distance between an
investment-grade hotel and a skate brand, and today it is undocumented.

### 4c. The base entry is the guideline — this is the load-bearing half

A skin that could change anything would let `mood:playful` turn `entrance.mask-rise` into
a different effect, and the catalogue would stop meaning anything. So the constraint runs
the other way: **the base entry declares what a skin may touch, and what it may never
touch.** Two new fields in `ENTRY-SCHEMA.md`:

```markdown
- **Skin surface** — `duration` `stagger` `distance` `easing` `granularity`
- **Skin invariants** — the revealing edge stays hard; a soft or feathered edge is
  `entrance.blur-focus`, not this entry. Lines never overlap in flight. The mask is a
  clip, never an opacity ramp.
```

The invariants are the interesting field and they are already latent in every entry —
they are what the **What the reader sees** paragraph is describing when it says *"from
behind a hard edge"*. Writing them down as invariants makes them enforceable instead of
merely descriptive, and it gives the answer to the question the operator actually asked:
*the base animation is the guideline that theme wraps around.* A skin bends the numbers.
The entry owns the shape. If a register needs the shape changed, it needs a **different
entry**, and that is a Pipeline 1 gap, not a Pipeline 2 decision.

Neither field needs research. Both are derivable from text each entry already contains, so
this is a mechanical pass over 262 entries, not 262 new judgements.

### 4d. What this looks like in a build

`BUILD-GUIDE.md` §2 gains one layer. The ladder stays exactly as it is; the skin is applied
over it; `--intensity` survives unchanged as the rung dial, now genuinely orthogonal.

```css
/* tokens.css — the ladder, unchanged */
:root {
  --dur-base: 300ms;  --dur-slow: 500ms;  --dur-scene: 800ms;
  --move-md: 16px;    --stagger: 50ms;    --intensity: 1;
}

/* skins/luxe.css — generated from luxe.json. Deltas only. */
:root[data-skin="luxe"] {
  --skin-dur: 1.8;  --skin-dist: 1.6;  --skin-stagger: 2.0;
  --skin-ease: cubic-bezier(.16, 1, .3, 1);
  --skin-overshoot: 0;      /* effect files multiply their overshoot by this */
  --skin-split: line;
}

/* what an effect actually reads — one place, derived from both */
:root {
  --t-slow:  calc(var(--dur-slow) * var(--intensity) * var(--skin-dur, 1));
  --d-md:    calc(var(--move-md)  * var(--intensity) * var(--skin-dist, 1));
  --st:      calc(var(--stagger)  * var(--skin-stagger, 1));
}
```

Swapping `data-skin="luxe"` for `data-skin="technical"` on one attribute re-registers the
whole site: 1.8 → 1.0 on duration, 2.0 → 1.0 on stagger, heavy deceleration → linear, and
the split ceiling opens from line to character. **Not a different site. The same site, in a
different register** — which is the thing that cannot be done today at any price.

### 4e. What changes in the two pipelines

**Pipeline 2, Stage 3.** The designer already picks `mood:` tags. It now picks **one skin**
alongside the rung, and every element choice inherits it. The JUSTIFY line gains a cheap,
falsifiable clause: *"at `skin:luxe` this runs 1.6 s with a 0.16 s stagger."* A human at
Stage 4 can disagree with a number.

**Pipeline 2, Stage 4.** The review gains a second dial. Today the only thing a client can
ask for is "less". They can now ask for "colder", and it is an edit.

**Pipeline 2, Stage 6.** One more fidelity check: does the built site read the skin's
tokens, or did the transcriber write numbers? — which the `showcase` review found it doing
in the GSAP half of the site.

**Pipeline 1.** Gains a real acquisition signal. Counting entries per skin-compatibility
shows which registers the catalogue is thin in, the same way counting `era:` shows the
platform-native skew today.

## 5. The other four gaps this run found

The skin layer is `G-S4` and it is the structural one. The rest are ordinary entries.

| id | Gap | Shape |
|---|---|---|
| `G-S1` | **Display type measured to the viewport as a layout primitive.** The reference site's most distinctive move is type that *is* the grid. Family 02 covers type in motion and never type as structure | one entry, family 02 |
| `G-S2` | **Preloader → hero handoff as one composite.** `entrance.curtain` and `load.determinate` both exist; the *join* — where the counter's completion is the hero timeline's start — is where these go wrong, and it is documented nowhere | one entry, family 01 |
| `G-S3` | **An element scrubbed continuously across a section boundary.** `scroll.pin-sequence` owns one section. Nothing describes continuity between two | one entry, family 03 |
| `G-S5` | **Catalogue defect, not a gap.** `INDEX.md`'s conflict matrix forbids `ambient.*` × 2+ outright; `TAXONOMY.md`'s intensity ladder lists "multiple simultaneous loops" as the ambient behaviour *of the maximal rung*. Both cannot hold. A design at maximal hits this immediately | QA pass |

Note that the skin table in §4b resolves `G-S5` as a side effect: `ambient` becomes a
per-skin number, so `minimal: 0` and `maximal: 2+` are both stated and neither contradicts
the other. The matrix line becomes "more than the skin permits", which is checkable.

## 6. Cost

| Work | Size |
|---|---|
| `skins/` — 12 markdown + 12 JSON, generated | one session |
| Two new fields on 262 entries | mechanical; derivable from existing text. Two to three sessions |
| `ENTRY-SCHEMA.md`, `TAG-VOCABULARY.md` §2, `BUILD-GUIDE.md` §2 | one session |
| Stage 3 and Stage 6 agent briefs | half a session |
| A worked re-skin of `work/showcase/` as proof | one session |

Roughly six sessions. The entry pass is the bulk and it is the least interesting work, which
is a reason to do it with a script and review the diff rather than by hand.

**Nothing is invalidated.** No entry changes mechanism, no family moves, no existing site
breaks — `--skin-*` all default to `1` via `var(--skin-dur, 1)`, so an un-skinned build is
byte-identical in behaviour to today's.

## 7. What I would push back on, or want decided

**1. Twelve skins is probably four skins and eight aliases.** `luxe`, `cinematic` and
`editorial` differ by less than their ranges. I have written twelve to match the mood facet
one-for-one, which is tidy and possibly false. *Alternative:* four base skins
(`composed`, `blunt`, `warm`, `precise`) with the twelve moods as named presets over them.
Fewer things to keep honest; loses the direct tie to a facet a designer already uses.

**2. Skins should probably compose, and I have made them exclusive.** A site is often two
registers — `technical` chrome around `cinematic` content. One skin per site is the same
simplification the intensity ladder makes, with the same weakness. *Alternative:* skin per
region, declared in the design doc. That is a real increase in what Stage 4 must review.

**3. The multipliers in §4b are asserted, not measured.** They are read off the vocabulary's
own signature column and they are defensible, not derived. *Alternative:* measure them —
take five sites per register, instrument the durations, and publish the distribution. That
is a Pipeline 1 discovery job and it would turn this table from taste into data. It is also
the single highest-value thing in this document if the catalogue is meant to be a reference
rather than a house style.

**4. `overshoot: required` on `playful` is the only field that forces rather than permits.**
It is right — a playful register with no overshoot is a corporate site with bright colours —
but a *required* field in a delta layer is a smell, and I would rather be told to soften it
than soften it myself.

My recommendation is what is written above. The one I would most like overruled on is (3):
if these numbers get measured instead of asserted, the skin layer stops being a convention
and starts being a finding.
