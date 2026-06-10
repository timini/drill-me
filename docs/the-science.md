# The science behind drill-me

drill-me is not a chatbot persona — every rule in its playbook is lifted from the
cognitive-science literature on how humans actually form durable memories. This page
is the receipts. The two load-bearing techniques (retrieval practice and spaced
repetition) are the only ones rated **high utility** in the most-cited review of
learning techniques ever written (Dunlosky et al., 2013); everything else here is a
documented multiplier on top of them.

## 1. Retrieval practice (the testing effect)

**The finding.** Being tested on material beats re-reading it — not by a little.
Roediger & Karpicke (2006) had students either re-study a passage or take practice
tests on it; a week later, the tested group recalled far more, even though the
re-readers *felt* more confident. A meta-analysis of 217 effects (Adesope et al.,
2017) puts the advantage at g ≈ 0.61 over restudying. The act of pulling something
out of memory strengthens the memory itself; re-reading just creates fluency — the
warm, false glow of recognition.

**In drill-me.** The tutor never explains what you might be able to produce. Every
session opens with recall of the previous one ("tell me everything you remember"),
every chunk of teaching is immediately followed by a question, and free recall is
preferred over multiple choice (generation beats recognition).

> Roediger & Karpicke (2006), *Psychological Science* 17(3). Adesope, Trevisan &
> Sundararajan (2017), *Review of Educational Research* 87(3).

## 2. Spaced repetition with decay (distributed practice)

**The finding.** Memory decays on a curve (Ebbinghaus, 1885), and the optimal moment
to review is just before you'd forget — a hard, barely-successful retrieval
strengthens memory far more than an easy, immediate one. Modern schedulers model this
explicitly: each fact has a *stability* (how long until recall probability decays to
~90%) that grows multiplicatively with each successful, well-timed review. SM-2
(SuperMemo, 1987) and FSRS (the open-source scheduler now powering Anki) are the
canonical algorithms.

**In drill-me.** Every concept you're taught becomes a card in `~/.drill-me/` with a
stability and a due date, updated by a simplified FSRS after every answer: succeed
when due → interval roughly doubles; fail → it collapses back to a day. Next session
opens with whatever has decayed most. The math lives in
[`reference/scheduling.md`](../reference/scheduling.md).

> Dunlosky, Rawson, Marsh, Nathan & Willingham (2013), *Psychological Science in the
> Public Interest* 14(1). FSRS: github.com/open-spaced-repetition.

## 3. The 85% rule (desirable difficulty)

**The finding.** Robert Bjork's "desirable difficulties" program showed that
conditions which make practice feel harder — spacing, variation, testing — produce
better long-term retention, while fluent practice produces illusions of mastery.
Wilson, Shenhav, Straccia & Cohen (2019) put a number on it: for a broad class of
learning systems, learning is fastest when the learner errs about 15% of the time.

**In drill-me.** The tutor tracks your rolling success rate and runs a servo around
~85%: cruise above 90% and questions get harder (transfer problems, combined
concepts, no cues); sink below 70% and it scaffolds down (smaller chunks, worked
examples, multiple choice). You should feel slightly stretched the whole time. That's
the point.

> Wilson et al. (2019), "The Eighty Five Percent Rule for optimal learning," *Nature
> Communications* 10:4646. Bjork & Bjork (2011), "Making things hard on yourself,
> but in a good way."

## 4. The pretesting effect

**The finding.** Guessing the answer *before* being taught improves retention of the
subsequent teaching — even when the guess is wrong (Richland, Kornell & Kao, 2009;
reviewed in Pan & Carpenter, 2023, across 60+ studies). A failed retrieval attempt
primes attention for exactly the information that resolves it.

**In drill-me.** Every new concept opens with "before I explain — take a guess,"
and the explanation that follows directly answers the question you just guessed at.

> Richland, Kornell & Kao (2009), *JEP: Applied* 15(3). Pan & Carpenter (2023),
> *Educational Psychology Review* 35.

## 5. Confidence ratings & the hypercorrection effect

**The finding.** People are systematically miscalibrated about what they know — and,
counterintuitively, errors made with *high* confidence are the most likely to be
permanently corrected after feedback (Butterfield & Metcalfe, 2001; Metcalfe & Finn,
2011). The metacognitive jolt of "I was *sure*" makes the correction stick — if it's
followed up with another test.

**In drill-me.** You rate your confidence 1–5 before the answer is revealed.
Confident-and-wrong answers get flagged in the ledger (`cw`) and are guaranteed a
priority re-test later in the session and at the top of the next one.

> Butterfield & Metcalfe (2001), *JEP: Learning, Memory & Cognition* 27(6).

## 6. Interleaving

**The finding.** Mixing problem types (A B A C B…) instead of blocking them
(AAA BBB…) hurts practice performance and roughly *doubles to triples* delayed test
performance — Rohrer & Taylor (2007) found d = 1.34; a classroom RCT (Rohrer et al.,
2014) found 72% vs 38%. Mixing forces you to *choose* the right approach, not just
execute the one the block implies.

**In drill-me.** Once two or more concepts are live, the tutor stops blocking and
mixes old questions among new ones, including "which of these applies here?"
discrimination questions. It will warn you that this feels worse. It is supposed to.

> Rohrer & Taylor (2007), *Instructional Science* 35. Rohrer, Dedrick & Stershic
> (2015), *JEP: Applied*.

## 7. Cognitive load (why no walls of text)

**The finding.** Working memory holds roughly four chunks of novel information
(Cowan, 2001; Sweller's cognitive load theory). Instruction fails when verbosity and
redundancy crowd out the processing that builds schemas. Novices learn more from
worked examples than from unguided problem-solving — but for advanced learners the
effect *reverses* (Kalyuga's expertise-reversal effect), so support must fade.

**In drill-me.** Hard caps: one question per message, one idea per chunk, ~150 words
of explanation max before you're asked to do something. Novices get worked examples
that fade into completion problems and then full problems; learners who've
demonstrated competence skip straight to problems.

> Sweller (1988), *Cognitive Science* 12. Kalyuga et al. (2003), *Educational
> Psychologist* 38(1).

## 8. Elaborative interrogation, self-explanation & teach-back

**The finding.** Generating answers to "why is that true?" and explaining material in
your own words both rated *moderate utility* in Dunlosky et al. (2013) — reliable
gains, especially when the learner generates the explanation rather than receiving
it. The Feynman-style teach-back exploits the same mechanism plus the "protégé
effect": even *expecting* to teach improves learning (Nestojko et al., 2014).

**In drill-me.** Why-probes are sprinkled through every session, and each session
closes with a teach-back — you explain the hardest concept to the tutor, who plays a
confused student and pokes at exactly what you glossed over. Then *you* write the
session summary; the tutor only marks the gaps.

> Dunlosky et al. (2013). Chi et al. (1994) on self-explanation. Nestojko et al.
> (2014), *Memory & Cognition* 42.

## What drill-me deliberately avoids

The same Dunlosky review rated these **low utility**, and drill-me's playbook bans
them as defaults: summarizing at the learner (you write the summaries), highlighting,
and above all **re-reading** — the tutor never re-presents material you've seen; if
you forgot it, that's a retrieval opportunity.
