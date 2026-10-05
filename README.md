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

Everything below was measured with `python run_eval.py` (3 runs per question, cache off) and logged in `results/run_2026-10-04_1901_before.md` and `results/run_2026-10-04_1903_after.md`. There is no `scorer.py`, so I judged answers by hand against the retrieved chunks.

## Run Log — Before

Retrieval and the gate are deterministic, so criteria 1, 3 and 4 come out the same on every run; only the generated wording changed between runs. Criterion 2 and 5 were checked on each run's actual answers.

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 2/5 | 2/5 | 2/5 | MISSED |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Chunks: ≥4/5 free of unnecessary info AND ≥3/5 within 2-3 sentences | 4 of 5 / 3 of 5 | 3/5 and 22/22 | 3/5 and 22/22 | 3/5 and 22/22 | MISSED |
| 5. ≥3/5 answers give the context of the person who wrote the source | 3 of 5 | 0/5 | 0/5 | 0/5 | MISSED |

How criterion 1 was counted (whole answer needed in the top 5; every question has two facts or one fact I can check): Q1 yes, Q2 yes, Q3 no (exam count missing), Q4 no ("busy at the beginning" missing), Q5 no (rubric timing missing). Q1 and Q2 pass; the three two-part questions fail on their second part.

How criterion 4 was counted: part A = is the top-1 chunk for each question free of text irrelevant to the question. Q1 yes, Q2 yes, Q4 yes, Q3 no (top-1 was `course_biol_160.txt#2`, a different course), Q5 no (top-1 was a workload chunk about reading load, answering neither half) = 3/5. Part B = the 22 distinct chunks retrieved across all five questions each have ≤3 sentences (22/22; 224 of all 231 chunks).

**Real output (from `results/run_2026-10-04_1901_before.md`, produced by `run_eval.py::run_once` → `store.py::search` → `gate.py::check` → `generate.py::answer_from_chunks`)**

Criterion 1 — top 5 retrieved for the Econ 101 question (`store.py::search`):

```
1 0.442 course_biol_160.txt     | Not curved. / Expect 9 to 11 hours a week, the heaviest first-year course by reputation. ...
2 0.477 course_econ_101.txt     | ECON 101 Introduction to Economics / Took this last spring. Format is large lecture, 300 people, with small discussion sections.
3 0.485 course_cs_210_exams.txt | Midterms are curved, the final is not. ...
4 0.488 course_stat_150.txt     | No curve, but the lowest midterm is dropped. ...
5 0.489 course_econ_101.txt     | Curved, and generously. / Expect 4 hours a week outside class. ...
```

The sentence "Assessment: two midterms and a final, all multiple choice." (`course_econ_101.txt#1`) is not in the top 5; it ranked 32nd (distance 0.620).

Criterion 2 — `generate.py::answer_from_chunks`, Q2 run 1:

```
The maximum number of hours you can work in a week during the term is 20 hours.

Source: money_jobs.txt
```

Criterion 3 — `run_eval.py::check_out_of_scope`, cutoff 0.6, refused 5 of 5:

```
| What is the capital of Mongolia? | 0.790 | refused |
| How do I change the oil in a diesel engine? | 0.848 | refused |
| Who won the 1994 World Cup? | 0.818 | refused |
| What is the recommended dosage of ibuprofen for a headache? | 0.798 | refused |
| How do I write a for loop in Rust? | 0.864 | refused |
```

Criterion 4 — the top-1 chunk for the Econ 101 question (`chunker.py::split_documents`):

```
course_biol_160.txt#2: Not curved. / Expect 9 to 11 hours a week, the heaviest first-year course by reputation. / The one piece of advice: the unit tests come fast, roughly every three weeks; falling behind once is very hard to recover from.
```

Criterion 5 — `generate.py::answer_from_chunks`, hist 118 question run 1. The retrieved `course_hist_118.txt` chunk opens "Transferred in last year, so take this with a grain of salt." and the answer never says so:

```
Yes, HIST 118 has no exams (it has two essays and a final project), making it a good fit if you dislike exams (`course_hist_118_exams.txt`). However, the provided documents do not contain information on when the rubrics are released, so I don't have enough information to answer that part of your question.
```

## Verdicts

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunk contains the answer (4 of 5) | MISSED | 2/5 on all three runs, against a target of 4. Not close: three of five answer chunks were not in the top 5 at all. |
| 2 | Every answer names a source (5 of 5) | MET | All 15 answers (5 questions x 3 runs) cited at least one filename. |
| 3 | Gate stops out-of-corpus questions (4 of 5) | MET | Gate refused 5/5; the closest out-of-scope question (0.790) is well over the 0.6 cutoff, and the worst in-corpus question (0.519) is well under it. |
| 4 | Chunk quality (≥4/5 no unnecessary info, ≥3/5 ≤3 sentences) | MISSED | The sentence-count half is met (22/22), but the "unnecessary information" half got 3/5 against a target of 4, and the criterion needs both. Reading "unnecessary" was a judgement call, so this one is the least certain; I kept my stricter reading. |
| 5 | ≥3/5 answers give the speaker's context | MISSED | 0/5 on all three runs. Not one answer said anything like "a second-year says" or "a transfer student, with a grain of salt". |

I did not revise any criterion. Criterion 4's "unnecessary information" is fuzzy, but I could still score it, so it is a miss, not a broken measurement.

## Diagnoses

**Criterion 1 (retrieval, caused by chunking).** Q3, Q4 and Q5 all ask two things, and in each the chunk with the second answer is a continuation chunk that never names its subject. "Assessment: two midterms and a final, all multiple choice." (Q3), "It's front-loaded — the first month is heavier than the rest" (Q4, `course_stat_150_workload.txt#1`, rank 23) and "...the essay rubric is posted in week 2" (Q5, `course_hist_118.txt#2`, rank 6) contain no course name, so the embedding of "ECON 101 / STAT 150 / HIST 118" can't match them. Chunks that do name another course or are generic ("Curved, and generously", "Not curved") outrank them. One pattern, three misses: chunking by sentence stripped the title off every chunk after the first, and the embedder only sees chunk text. Generation was fine: given the right chunk, the model answered it.

**Criterion 4 (chunking).** Same root cause. A chunk like `course_biol_160.txt#2` ("Not curved. Expect 9 to 11 hours...") has nothing saying it is BIOL 160, so it matches any "curved" question for any course and arrives as noise for the Econ 101 question.

**Criterion 5 (generation, with a chunking contribution).** The prompt in `generate.py::GROUNDING_INSTRUCTION` never asks the model to say who wrote the source, so it doesn't. This is a generation miss, not retrieval: the speaker sentence was in the retrieved chunks for Q1 ("Second-year here.") and Q5 ("Transferred in last year, so take this with a grain of salt.") and was left out of the answer. Only 22 of the 88 documents contain such a sentence, so even a perfect prompt could not reach 3/5 on every set of questions; but it could here, where two or more of my questions' sources have one.

## The Improvement

**What I changed:** In `chunker.py::split_documents`, every chunk after the first in a document now starts with the document's title line (e.g. "ECON 101 Introduction to Economics") before its text, controlled by `config.CHUNK_HEADERS` (env `AI201_CHUNK_HEADERS=1`). I first indexed it as a separate variant (`python app.py --variant headers index`) so the before index stayed untouched for the comparison. Afterwards I switched `CHUNK_HEADERS` on by default and rebuilt the default index, so `python app.py ask` now uses the improved chunking; `AI201_CHUNK_HEADERS=0` rebuilds the old one. Nothing else changed: same model, top-k 5, cutoff 0.6, prompt.

**Why I picked it:** Criterion 1's three misses all came from continuation chunks that don't say which course they are about, and putting the title back into the chunk text is the direct fix for that one mechanism. I chose this over hybrid search because BM25 can't match "Econ 101" to a chunk that doesn't contain the words "Econ 101" either.

### Run Log — After

`python run_eval.py --label after --variant headers` → `results/run_2026-10-04_1903_after.md`.

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 4/5 | 4/5 | 4/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Chunks: ≥4/5 free of unnecessary info AND ≥3/5 within 2-3 sentences | 4 of 5 / 3 of 5 | 5/5 and 24/24 | 5/5 and 24/24 | 5/5 and 24/24 | MET |
| 5. ≥3/5 answers give the context of the person who wrote the source | 3 of 5 | 0/5 | 0/5 | 0/5 | MISSED |

After, the top-1 chunk for every question comes from the document the question is about (Q3 is now `course_econ_101.txt#2`, not BIOL 160). The 24/24 count excludes the added title line when counting sentences.

Real output after (`generate.py::answer_from_chunks`), STAT 150 question run 1, which was the "documents do not contain information regarding busier at the beginning" answer before:

```
STAT 150 is front-loaded, meaning the first month is heavier than the rest. You can expect to spend 5 to 6 hours a week outside of class.
*(Sources: `course_stat_150_workload.txt` and `course_stat_150.txt`)*
```

HIST 118 run 1, which could not give the rubric timing before:

```
Yes, HIST 118 is a good fit for you because it has no exams; assessment consists of two essays and a final project (course_hist_118.txt). The essay rubric is posted in week 2 (course_hist_118.txt, course_hist_118_exams.txt).
```

Econ 101 run 1, still wrong on the exam count:

```
Based on the provided documents, the number of exams for ECON 101 is not mentioned. However, the documents state that grades are curved, and generously (*course_econ_101.txt* and *course_econ_101_exams.txt*).
```

**Did it help?** Yes, on the criterion it targeted: criterion 1 went from 2/5 to 4/5 on all three runs, and two answers (Q4, Q5) changed from "not in the documents" to correct. Criterion 4 flipped to MET as a side effect, because the same noise (wrong-course chunks) was behind it. Caveat: I'm judging chunk noise by hand, and the "after" 5/5 is on my own reading of the top-1 chunk. The gate also moved slightly (best out-of-scope distance still ≥0.790, best in-corpus ≤0.490, so the 0.6 cutoff is still in a clean gap). Criterion 5 didn't move, as expected, since I didn't touch the prompt. One run per condition is a small sample: the 2→4 jump is two questions.

## What's Still Broken

- **Criterion 5 (still MISSED, 1/5):** a generation problem plus a retrieval one. After the headers fix I made a second, separate change: a line in `generate.py::GROUNDING_INSTRUCTION` telling the model to mention who is speaking when the source says. Re-run (`results/run_2026-10-04_1909_after-prompt.md`, headers index): 1/5 on all three runs. Only the Ridgeway answer changed ("according to a second-year student in `dining_the_ridgeway_cafe.txt`…"). The HIST 118 answer still says nothing about being a transfer student because the chunk holding "Transferred in last year, so take this with a grain of salt" (`course_hist_118.txt#0`) isn't in the top 5 after the headers change, so the model never sees it. Only 22 of 88 documents have such a sentence at all. What I'd do next: carry the speaker sentence into every chunk of that document, or re-scope the criterion (see below). Note this second change is not part of the before/after comparison above, which isolates the title headers.
- **Econ 101 exam count:** still not answered. "Assessment: two midterms and a final" is in `course_econ_101.txt#1`, which isn't in the new top 5 either (the question's other half, "curved", pulls in curve-related chunks from other courses). Splitting two-part questions into two retrievals, or a larger top-k, would likely catch it. I didn't try; criterion 1 now passes at 4/5, but this is the same kind of fragility.
- The gate only checks the single best chunk, so a two-part question passes as soon as one half matches. That's why no refusal happened on any partly-answered question.

## What I'd Do Differently

Criterion 5 should be rewritten: "answers give the context of the person answering" depends on the corpus (only 22 of 88 documents even have such a sentence), so I should have scoped it to "for questions whose source documents mention who the writer is". I'd also tighten criterion 4: "unnecessary information" took judgement, and I'd replace it with something I can count, like "the top-1 chunk for at least 4 of 5 questions comes from the document the question names". Finally, criterion 1 should say "all parts of a multi-part question", since my questions were two-part and the original wording didn't make clear whether half an answer counted.

## How I Used AI (Unit 2)

**1. Finding the pattern in my retrieval misses.** After `run_eval.py` showed criterion 1 at 2/5, I asked Claude to print the top 40 results for the three questions that failed and report where the chunk holding the missing answer ranked. It came back with ranks 32 (Econ 101 exam count), 23 (STAT 150 "front-loaded") and 6 (HIST 118 rubric). I read those chunks and saw that none of them names its course, which is why a question that says "Econ 101" can't match them. That became one diagnosis instead of three separate ones.

**2. The fix.** I asked Claude to add the document title to the top of every chunk after the first, behind a `CHUNK_HEADERS` setting so I could keep the old index to compare against. It wrote the change in `chunker.py` and `config.py`. I ran the full test against both indexes myself and chose to make the new chunking the default only after criterion 1 went from 2/5 to 4/5.

**3. A fix that mostly didn't work.** I had Claude add a line to the grounding prompt telling the model to mention who is speaking. Criterion 5 only went from 0/5 to 1/5, because the chunk with the speaker line for HIST 118 wasn't being retrieved. I reported it as still missed rather than lowering the target.

**Checking the drafts.** Claude drafted the first version of the README tables. I checked the counts against the files in `results/` before keeping them.
