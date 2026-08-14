# Animation Catalogue

A reference catalogue of frontend animations, written so that an entry can be **chosen or
rejected from its description alone** — without opening a demo, a CodePen, or a library's
marketing page.

That constraint is the whole point. Most animation resources are galleries: you have to
watch the thing to know what it is, which means you cannot search them, cannot compare
them, and cannot hand them to someone else as a decision. Here, every entry carries a
paragraph called **What the reader sees**, written the way a director describes a shot.
If that paragraph does its job, the demo is optional.

## Who this is for

- **Someone designing a page** who knows the feeling they want and not the name of it.
  Start at `INDEX.md` and read the decision tree.
- **Someone implementing** an effect they have already chosen. Go to the family file and
  read Mechanism, Params, Performance and Gotchas.
- **An agent** with no visual access at all, deciding what to build. This catalogue is
  written to be legible to a reader who cannot see.

## How to read an entry

Every entry follows `ENTRY-SCHEMA.md` exactly. The fields are ordered by how you actually
use them:

1. **One line** — is this even the right neighbourhood?
2. **What the reader sees** — is this the thing I want? *Read this one properly.*
3. **Mechanism / Stack / Params** — what it costs to build and what I can tune.
4. **Use when / Don't use when** — the honest boundaries.
5. **Reduced motion / Performance / Gotchas** — what it takes to ship it responsibly.
6. **References** — where the claims came from.

## Layout

```
TAXONOMY.md        the classification axes, and the intensity ladder
STACKS.md          what to build these with, and what each stack is bad at
ENTRY-SCHEMA.md    the contract every entry conforms to
QUEUE.md           the work spine — see below
families/          one file per animation family, the bulk of the catalogue
cross-cutting.md   accessibility, performance budgets, orchestration, anti-patterns
INDEX.md           every entry in one table, plus the decision tree
QA-REPORT.md       what the conformance pass changed
```

## How this gets built

`QUEUE.md` is the spine. Each work session takes **the first unticked line**, researches
only that, writes only that file, commits, and ticks it. One session, one phase — never
two, never skipped.

That rule exists because the catalogue is built by agent sessions that share no memory.
The repository is the only thing that persists between them, so the queue is not a
progress bar for humans: it is the mechanism by which a session with no context knows
what to do. Disjoint file ownership per phase is what makes it safe to stack sessions
without them overwriting each other.

## Standing rules

- **Nothing is written from memory.** Every phase does live research and cites what it
  actually fetched. A dead link is deleted, not left decorating the page.
- **Uncertainty is stated, not smoothed over.** A flagged "sources disagree here" is worth
  more than a confident guess, because this file will be read a year from now.
- **No marketing language.** Never *stunning*, *beautiful*, *eye-catching*, *seamless*.
- **Effects that are dated, overused, or bad on touch are labelled as such.** A catalogue
  that only praises is a catalogue nobody can make a decision with.
