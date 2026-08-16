# Entry schema

Every catalogue entry conforms to this exactly. No fields omitted, no fields invented,
this order. A phase that deviates gets corrected in the QA pass.

```markdown
### <Name> (`family.id`)

- **One line** — what it is, in a single sentence.
- **What the reader sees** — REQUIRED, 60–150 words. See below.
- **Mechanism** — which properties are animated, over what curve, driven by what.
- **Stack** — which stacks do this well, the plugin or bundle cost, the cheapest route.
- **Params** — the 3–6 knobs that matter, with defaults and the range where it breaks.
- **Use when** — one line; 2–4 clauses separated by semicolons.
- **Don't use when** — one line; 2–4 clauses separated by semicolons.
- **Reduced motion** — what it becomes under `prefers-reduced-motion`.
- **Performance** — compositor-only or layout-touching; per-frame cost; mobile caveat.
- **Gotchas** — the things that cost someone a day.
- **References** — 2–4 URLs actually fetched while writing the entry.
- **Tags** — the facet block, per `tags/TAG-VOCABULARY.md`. Twelve facets, in vocabulary order.
- **Pairs with** — 2–4 entry ids that compose well with this one. Ids only, no prose.
- **Conflicts with** — entry ids that must not share a screen with this one, each with the
  reason in parentheses. Empty is a valid answer; "none found" is not — omit the field only
  if you checked.
```

## The field that matters

**What the reader sees** is the product. Everything else supports it.

Write it so that someone who has never seen the effect can picture the whole thing: what
moves, from where to where, in what order, how fast, how it feels, what it resembles.
No code. No library names. No jargon that only means something to the person who built it.

A reader must be able to **choose or reject this animation from this paragraph alone**.

### Good

> The headline is invisible, then its first line slides up from behind an edge as though
> rising into a window frame — you never see it cross the boundary, only emerge. The
> second line follows about a tenth of a second later, then the third, so the block
> assembles top to bottom in roughly half a second. Each line decelerates hard at the
> end, arriving rather than stopping. The effect is of type being set rather than typed:
> composed, deliberate, faintly editorial. At small sizes it reads as polish; at display
> sizes it reads as the page introducing itself.

### Bad

> A beautiful staggered text reveal that adds a modern, professional feel to your hero
> section. Uses SplitText with a y-offset and a stagger for a smooth entrance.

The second one tells you nothing you could not have guessed from the name. It describes
the *implementation* and the *marketing*, and skips the only thing the reader needs: what
it actually looks like.

## Field notes

**Mechanism** — be specific about properties, because it determines cost: `transform` and
`opacity` are compositor-only; `width`, `top`, `filter` and `clip-path` are not equal in
price. Name the easing family and say what drives the timeline (time, scroll position,
pointer, state change, physics).

**Params** — give the knob, its sane default, and the point past which it stops working.
"Stagger 0.05s; below 0.02s it reads as simultaneous, above 0.15s the last item feels
abandoned" is worth more than a list of prop names.

**Reduced motion** — never just "disabled". Say what *replaces* it. An entrance animation
becomes an instant appearance at final opacity; a scroll-scrubbed effect becomes its end
state; a looping ambient effect becomes a still frame. If the animation carries meaning
that has no static equivalent, say that too — it is a reason not to use it.

**Gotchas** — the specific, expensive things. Fonts not settled before measuring. A
transform on a `position: fixed` ancestor. Split text going invisible to screen readers.
Two libraries writing the same property. Anything you had to learn twice.

**References** — fetched this session, not remembered. If a URL 404s or is paywalled,
drop it and find another. Prefer primary sources: library docs, spec text, the
technique author's own write-up. Listicles and content farms are not references.

**Use when / Don't use when** — one prose line each, clauses separated by semicolons. Not
bullets. This schema asked for "2–4 bullets" until 2026-08-16 while all 263 entries used the
prose line, which meant the reviewer's check 5 was measuring against a format nothing
followed. The entries were right: a semicolon line reads as a sentence and stays inside the
entry's rhythm, where four bullets fragment it. The schema was changed to describe the
catalogue rather than the catalogue reformatted to satisfy the schema.

**Tags / Pairs with / Conflicts with** — required fields, not an appendix. `TAG-VOCABULARY.md`
has always specified the block "appended to each entry after `References`", but this schema
ended at `References` and told the builder "no fields omitted, no fields invented". A builder
obeying its brief therefore shipped no tags, and Stage 4's check 8 — "tags are defensible" —
had nothing to check. That is exactly what happened to `cand-2026-08-16-a`: the reviewer
recorded check 8 as UNREVIEWABLE and specified a tag block itself, so those tags were written
by the agent that merged them and reviewed by nobody.

Tag at build time so the gate can see them. Tagging at integration puts the tags on the wrong
side of the review.

## Id convention

`family.name` in lowercase kebab, e.g. `entrance.mask-rise`, `scroll.clip-expand`,
`pointer.magnetic`. Ids are stable once published — the index and the conflict matrix
reference them, and a rename breaks both.
