# drill-me scheduling reference

This file defines the memory ledger format and the scheduling algorithm. Follow the
arithmetic exactly — do not improvise intervals. The algorithm is a simplified FSRS
(free spaced repetition scheduler): each card has a **stability** (how many days until
the learner's recall probability decays to ~90%) and a **difficulty** rating that damps
stability growth for concepts the learner finds hard.

Always get today's date by running `date +%Y-%m-%d` before reading or writing the ledger.
All dates in the ledger are absolute (`YYYY-MM-DD`). Never write relative dates.

## Ledger layout

```
~/.drill-me/
├── index.md              # one line per topic: name, card count, cards due, next due date
└── topics/<slug>.md      # one file per topic
```

`<slug>` is the kebab-cased topic name (e.g. `rust-lifetimes`, `this-repo-auth-flow`).
Before starting a session, list `~/.drill-me/topics/` and fuzzy-match the requested
topic against existing slugs — "rust lifetimes" must resume `rust-lifetimes.md`, not
create a duplicate. If the match is ambiguous, ask.

## Topic file format

```markdown
---
topic: Rust lifetimes
slug: rust-lifetimes
source: model knowledge        # or: codebase @ /path, file @ /path, url
created: 2026-06-10
learner_level: developing      # novice | developing | solid (your running judgment)
learner_notes: Knows borrowing well; confuses 'static with static items. Prefers code-first examples.
---

## Cards

| # | concept | S (days) | D (1-5) | last | due | flags | history |
|---|---------|----------|---------|------|-----|-------|---------|
| 1 | Why lifetimes exist (dangling refs) | 6.0 | 2 | 2026-06-10 | 2026-06-16 | | G,G |
| 2 | Lifetime elision rules | 1.0 | 4 | 2026-06-10 | 2026-06-11 | cw | A |
| 3 | 'static meaning | 2.6 | 3 | 2026-06-10 | 2026-06-13 | | H |

## Not yet taught

- Lifetime bounds on generics (`T: 'a`)
- Higher-ranked trait bounds (intro only)
```

Column meanings:
- **S** — stability in days, one decimal place.
- **D** — difficulty 1 (easy for this learner) to 5 (hard). Start new cards at 3.
- **last / due** — last review date, next due date (`due = last + round(S)` days, minimum 1).
- **flags** — `cw` = answered confidently but wrong last time (priority re-test);
  `leech` = failed 4+ times total (needs re-teaching from a different angle, not re-quizzing).
- **history** — last ~8 grades, newest last: `A`=Again, `H`=Hard, `G`=Good, `E`=Easy.

The "Not yet taught" list is the remaining syllabus, so a returning session knows what's next.

## Grading a retrieval attempt

After each retrieval question, grade silently (never show the letter grades or the math
to the learner — they see normal tutoring, the ledger sees FSRS):

- **Again** — wrong, or couldn't produce it even with hints rungs 1–3.
- **Hard** — got there, but slowly, partially, or only after hints.
- **Good** — correct with normal effort.
- **Easy** — instant, confident, correct, and the explanation was solid.

## Stability update (on review of an existing card)

Let `elapsed` = days since `last`, and `R = elapsed / S` (how overdue: 1.0 = reviewed
exactly when due). Clamp `R` to [0.25, 2.0]. Overdue successful recalls earn more
(harder retrieval → bigger memory gain); early ones earn less.

On **success** (Hard/Good/Easy):

```
base   = Hard: 1.4   Good: 2.2   Easy: 3.0
factor = base × (1 + 0.35 × (R − 1))        # overdue bonus / early damping
factor = factor × (1.3 − 0.1 × D)           # D=1 → ×1.2, D=3 → ×1.0, D=5 → ×0.8
S'     = max(S × factor, S + 0.5)
```

On **Again** (failure):

```
S' = max(1.0, S × 0.3)
```

Difficulty drift: Again → D+1, Hard → D+0.5 (round up), Easy → D−1. Clamp to [1, 5].
Good leaves D unchanged.

Then: `last = today`, `due = today + round(S')`, append grade to history, set or clear
`cw` (set if the learner rated confidence 4–5 and was wrong; clear after a successful
re-test), set `leech` if history shows 4+ A's.

## New cards (first time a concept is taught and quizzed)

Initial stability from the first retrieval grade:

```
Again: S = 1.0    Hard: S = 1.5    Good: S = 3.0    Easy: S = 6.0
```

Initial D = 3, adjusted ±1 by how the teaching went (needed worked example and two
hints → 4; got it from the pretest guess alone → 2).

## Session ordering

1. **Due cards first**, sorted: `cw`-flagged cards, then most overdue (`elapsed / S`
   descending). Cards due within 1 day count as due.
2. **Leech cards** get re-taught (new angle, new example), not just re-quizzed.
3. **New material only after the due queue is cleared** — or after 10 due cards if the
   queue is long; say so and carry the rest.
4. Cap a session's new cards at ~7. Depth beats coverage.

## In-session repeats (expanding retrieval)

A card graded **Again** or **Hard** during this session gets re-asked within the same
session: after ~3 intervening exchanges, and once more near the close, each time with a
different surface form (new phrasing, new example, or inverted direction). Only the
final in-session grade is written to the ledger.

## index.md

Regenerate after every session:

```markdown
# drill-me index

| topic | cards | due now | next due | last session |
|-------|-------|---------|----------|--------------|
| [Rust lifetimes](topics/rust-lifetimes.md) | 12 | 3 | 2026-06-11 | 2026-06-10 |
```
