# The Unofficial Guide

Rithika — campus_life

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

This system answers questions about the `campus_life` corpus — 88 short
student posts and forum-style threads covering course workload and exam
formats, dining hall wait times, housing, campus jobs, financial aid, and
getting around campus. It's built for specific, factual questions with a
real answer, like "how many exams does ECON 101 have?" or "what's the wait
time at the Ridgeway Café at 12:30?" — not open-ended opinion questions.
Every answer names the source document it came from, and questions the
corpus doesn't cover are refused instead of guessed at.

## Chunking Strategy

**Chunk size:** 150 characters, grouped by whole sentences (never split mid-sentence)
**Overlap:** none — chunks don't share any text

My corpus is short posts, not long guides — most documents run around
300-400 characters, so the starter's default of 800 would have swallowed
almost an entire document into a single chunk, defeating the point of
retrieval. I switched to grouping whole sentences up to 150 characters
instead of a fixed character window, because a fixed-character cut can
slice a sentence in half mid-word, which produces unreadable fragments at
retrieval time.

I chose no overlap on purpose: with documents this short, adjacent chunks
from the same file are already topically close, so repeating text between
them felt like wasted retrieval slots rather than added context.

That tradeoff shows up in my own sample chunks. Chunk 2
(`course_biol_160_workload.txt#1`) opens with "That's real time, not
optimistic time" — the "that" refers to a sentence in the *previous*
chunk, which is now gone. Chunk 5
(`housing_innisfree_hall_laundry.txt#2`) is just "Sunday after 6pm you
will wait" — grammatically complete, but it never says what you'd be
waiting for; only the filename tells you it's about laundry. Sentence-only
grouping with no overlap keeps chunks clean and non-redundant, but it can
strand a sentence that depends on something just before it. If I had more
time I'd look at keeping one sentence of overlap between chunks to catch
exactly this case.

## Sample Chunks

<!-- Five chunks, pasted as text. Label each one and name the file it came from
     AND the function that produced it — the grader checks your code against
     what you claim here.

     `python app.py chunks -n 5` prints all three for you. Copy them straight
     across.

     Milestone 3. -->

**Chunk 1** — source: `admin_add_drop_deadline.txt#0` — produced by: `chunker.py::split_documents`

```
On the add/drop deadline

You can add a course through the end of the second week.
```

**Chunk 2** — source: `course_biol_160_workload.txt#1` — produced by: `chunker.py::split_documents`

```
That's real time, not optimistic time.

It's front-loaded — the first month is heavier than the rest, partly because you're learning the format.
```

**Chunk 3** — source: `course_phys_130_exams.txt#0` — produced by: `chunker.py::split_documents`

```
PHYS 130 Mechanics — assessment

Three midterms, no final, plus a lab practical.
```

**Chunk 4** — source: `dining_the_ridgeway_cafe.txt#0` — produced by: `chunker.py::split_documents`

```
The Ridgeway Café

Second-year here. Wait times: 10 to 15 minutes at 12:30, none after 2:00.
```

**Chunk 5** — source: `housing_innisfree_hall_laundry.txt#2` — produced by: `chunker.py::split_documents`

```
Sunday after 6pm you will wait.
```

## Sample Answer

**Question:** What is the wait time at 12:30 PM in ridgeway cafe

**Answer:**

```
The wait time at The Ridgeway Café at 12:30 is 10 to 15 minutes.

Sources: dining_the_ridgeway_cafe.txt and dining_the_ridgeway_cafe_followup.txt
```

**My relevance cutoff:** 0.6 (the starter's default)

I ran all five of my test questions and all five `OUT_OF_SCOPE` questions
through `app.py retrieve` and recorded the best (rank 1) distance for each.
My in-corpus questions landed between 0.131 and 0.519; my out-of-scope
questions landed between 0.790 and 0.864. That's a clean gap from 0.519 to
0.790 with nothing in it, and 0.6 sits comfortably in the middle of that
gap, so I kept the default rather than changing it.

| Question | In corpus? | Best distance |
|---|---|---|
| What is the wait time at 12:30 PM in ridgeway cafe | yes | 0.131 |
| What is the maximum number of hours we can work in a week during the term? | yes | 0.375 |
| How many exams are there for Econ 101? Is the class curved? | yes | 0.442 |
| Is stat 150 more busy at the beginning or end of the semester? | yes | 0.276 |
| I don't like exams — is hist 118 a good fit? When are rubrics released? | yes | 0.519 |
| What is the capital of Mongolia? | no | 0.790 |
| How do I change the oil in a diesel engine? | no | 0.848 |
| Who won the 1994 World Cup? | no | 0.818 |
| What is the recommended dosage of ibuprofen for a headache? | no | 0.798 |
| How do I write a for loop in Rust? | no | 0.864 |

**On top-k:** I kept `TOP_K` at the default of 5 rather than lowering it.
The hist 118 question is a real example of why: the chunk that actually
answers "when are rubrics released" (`course_hist_118_exams.txt`) comes
back at rank 4 with a distance of 0.620 — worse than the overall 0.6
cutoff on its own. If `TOP_K` were 3, that chunk would never make it back
at all, even though the question passes the gate (rank 1 is 0.519). Five
gives the right chunk room to show up even when it isn't the closest
match.

**On the Econ 101 case:** the closest chunk for "How many exams are there
for Econ 101?" (rank 1, distance 0.442) is actually `course_biol_160.txt`
— a different course that just happens to also mention exams and
curving. The real Econ 101 chunk is rank 2 at 0.477. The gate still passes
(0.442 is well under 0.6) and the right chunk is still in the top 5, so
the answer works out — but it's a concrete case of distance not implying
topical correctness, which is part of why a hard word/rank cutoff alone
isn't enough and the grounding instruction below matters too.

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

**1.**
I asked claude to write the chunking function based on my instruction. I choose sentence chunking along with 150 character limit and picked what worked best. 
**2.**
I asked Claude to run the tests so I could compare the results. it first didn't show me the results and said they passed. I had to prompt it to show me the chunks and results. 
<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->
