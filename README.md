# 🧠 drill-me

> **`/grill-me` taught Claude what you know. `/drill-me` teaches you what Claude knows.**

A Claude Code plugin that turns Claude into an adaptive tutor with a long-term memory
of *you*. It quizzes before it explains, spaces reviews on a forgetting curve, keeps
you at the difficulty sweet spot, and remembers exactly what you were shaky on —
across sessions, across projects, forever (or until you `rm -rf ~/.drill-me`).

```
/drill-me rust lifetimes
/drill-me this repo's auth flow
/drill-me ~/papers/attention-is-all-you-need.pdf
```

No walls of text. No "Great question! Here are 7 key points." One question at a time,
relentlessly — just in the other direction.

---

## Why

[`/grill-me`](https://github.com/mattpocock/skills) went viral because it inverted the
usual flow: instead of the model guessing what's in your head, it interrogates you
until your knowledge is in *its* head. One question at a time, no mercy.

**drill-me is the mirror image.** The model's knowledge → your head. And it turns out
cognitive science knows exactly how to do that transfer, and it's the opposite of how
chatbots normally behave:

| Chatbots default to | The evidence says |
|---|---|
| Explaining things at you | **Testing you** — retrieval practice beats re-reading, g ≈ 0.61 ([Roediger & Karpicke 2006](https://doi.org/10.1111/j.1467-9280.2006.01693.x)) |
| Everything in one session | **Spacing reviews** on your forgetting curve (FSRS — the algorithm behind Anki) |
| Making it feel easy | **~15% error rate** is the optimal difficulty ([Wilson et al. 2019, *Nature Comms*](https://www.nature.com/articles/s41467-019-12552-4)) |
| Walls of text | **≤150 words per chunk**, working memory holds ~4 items (Sweller, cognitive load theory) |
| Telling you the answer | **Making you guess first** — pretesting boosts retention *even when you guess wrong* |
| "You're absolutely right!" | **Confidence ratings before the reveal** — confident-wrong errors are the most correctable of all (the hypercorrection effect) |

Every rule in drill-me's playbook has a citation. The full receipts are in
[**docs/the-science.md**](docs/the-science.md).

## What a session looks like

```text
You:    /drill-me git rebase

Claude: Before we start — what do you already know about rebase?
        Even fragments are fine.

You:    it's like merge but rewrites history? i always just merge
        because rebase scares me

Claude: That's a real starting point — and "scares me" is fixable.
        Quick guess before I explain anything: when you rebase a
        branch onto main, what do you think actually happens to
        your commits?

You:    they get... moved onto main?

Claude: Close — they get *replayed*. Git takes each of your commits,
        turns it into a patch, and re-applies it on top of main.
        New commits, new hashes. The old ones are abandoned, not moved.

        So — your teammate has branch `foo` checked out, and you
        rebase `foo` and force-push. Why is your teammate now sad?

You:    because their local foo points at commits that don't exist
        anymore?

Claude: ▸ How confident are you in that answer?
          ○ 1–2  Guessing
          ● 3    Fairly sure
          ○ 4–5  Certain

Claude: It's exactly right — and worth more confidence than a 3. ...
```

Next session opens with: *"Last time we covered rebase — before anything new: without
looking, what's the difference between what merge and rebase do to history?"* — because
that card is due today, and the card you flubbed is due first.

## What it can teach

- **Anything Claude knows** — `/drill-me bayesian statistics`, `/drill-me kubernetes networking`
- **Your own codebase** — `/drill-me this repo's payment flow`. It explores the code
  first, then drills you on it, anchoring every question to real `file:line`s. Great for
  onboarding (or for finally understanding the legacy service you inherited).
- **Documents** — point it at a PDF, a file, or a URL and get drilled on *that*.

## Install

```bash
# in Claude Code:
/plugin marketplace add timrichardson/drill-me
/plugin install drill-me@drill-me
```

Then:

```
/drill-me <anything>
/drill-status        # what's due, weak spots, what to study next
```

<details>
<summary>Try it without installing</summary>

```bash
git clone https://github.com/timrichardson/drill-me
claude --plugin-dir ./drill-me
```
</details>

## How the memory works

Everything lives in **`~/.drill-me/`** as plain markdown — readable, editable, yours:

```
~/.drill-me/
├── index.md                  # all topics, what's due when
└── topics/
    ├── git-rebase.md         # one card table per topic
    └── rust-lifetimes.md
```

Each concept is a card with a **stability** (days until you'd forget it) and a **due
date**, updated by a simplified [FSRS](https://github.com/open-spaced-repetition)
scheduler after every answer: recall it when due and the interval roughly doubles;
miss it and it collapses back to a day. Cards you got wrong *while confident* are
flagged and jump the queue. `/drill-status` shows the whole dashboard.

Delete a file to forget a topic. Delete the directory to forget everything. No
database, no sync, no account.

## How it works under the hood

```
drill-me/
├── commands/
│   ├── drill-me.md            # the command — lean, ~60 lines
│   └── drill-status.md        # progress dashboard
├── reference/
│   ├── scheduling.md          # the FSRS-style algorithm + ledger format
│   └── teaching-playbook.md   # session structure, hint ladder, difficulty servo
└── docs/
    └── the-science.md         # citations for every design decision
```

The command stays small (grill-me's spirit); the algorithm and playbook are reference
files it loads at session start. Fork it and tune the playbook to taste — it's all
prose.

## FAQ

**Is this just flashcards?**
The scheduling is Anki-shaped, but the cards write themselves, the "fronts" never
repeat verbatim (varied retrieval transfers better), questions escalate from recall to
transfer problems as you improve, and there's a tutor attached who scaffolds you
through misses instead of just flipping the card.

**Why does it keep asking instead of explaining?**
Because that's the entire trick. Re-reading explanations feels like learning and
measurably isn't. If you want explanations, ask Claude normally — drill-me exists for
the part where it *sticks*.

**It told me a question would "feel harder and that's the point" — is it gaslighting me?**
No — interleaved practice genuinely feels worse and tests ~2× better
(Rohrer & Taylor 2007). Desirable difficulty is the feature.

**Does it work with any topic?**
Anything Claude can be accurate about. For fast-moving or niche material, point it at
the source (file/URL) instead of relying on model knowledge.

## Credits

- [Matt Pocock's **grill-me**](https://github.com/mattpocock/skills) — the inspiration
  and the interaction pattern, inverted.
- Dunlosky et al. (2013), Bjork, Roediger & Karpicke, Wilson et al., and the
  [open-spaced-repetition](https://github.com/open-spaced-repetition) project — the
  actual ideas. See [docs/the-science.md](docs/the-science.md).

MIT © Tim Richardson
