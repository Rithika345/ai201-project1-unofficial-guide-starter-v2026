# Run log — after-prompt

- Produced by: `run_eval.py::main`
- Retrieval: `store.py::search`, chunks from `chunker.py::split_documents`
- Corpus: `campus_life` (index variant `headers`)
- top-k: 5 · relevance cutoff: 0.6
- Runs per question: 3, caching off
- When: 2026-10-04 19:09

This table is one row per QUESTION. The run log your README asks for is
one row per CRITERION, so aggregate these into it — criterion 1 is how many
of your questions had the answer in the retrieved chunks, and so on.

| Question | Run 1 | Run 2 | Run 3 |
|---|---|---|---|
| What is the wait time at 12:30 PM in ridgeway cafe |   |   |   |
| What is the maximum number of hours we can work in a week during the term? |   |   |   |
| How many exams are there for Econ 101? Is the class curved? |   |   |   |
| Is stat 150 more busy in the beginning or end of the semester? How much time can I expect to spend on this class? |   |   |   |
| I don't like taking exams so is hist 118 a good fit for me? I want to do well so when are the rubrics released? |   |   |   |

> The Run columns are blank because `scorer.py` doesn't exist yet.
> Judge each question yourself by reading the output below, or build
> the scorer first and re-run.

---

## The relevance gate on out-of-corpus questions

Produced by `run_eval.py::check_out_of_scope`, cutoff 0.6. Refused 5 of 5.

Retrieval is deterministic and the gate is a comparison against a
fixed number, so these do not vary between runs — one pass over the
list is the whole measurement.

| Out-of-scope question | Best distance | Gate |
|---|---|---|
| What is the capital of Mongolia? | 0.790 | refused |
| How do I change the oil in a diesel engine? | 0.853 | refused |
| Who won the 1994 World Cup? | 0.827 | refused |
| What is the recommended dosage of ibuprofen for a headache? | 0.833 | refused |
| How do I write a for loop in Rust? | 0.858 | refused |

---

## Real output

This is what the system actually produced. Paste the relevant parts
into your README underneath the table — the rubric asks for real
output as text, not a description of it.

### What is the wait time at 12:30 PM in ridgeway cafe — run 1

- Best distance: 0.1306 (passed the gate)
- Sources retrieved: dining_the_ridgeway_cafe.txt, dining_the_ridgeway_cafe_followup.txt, transit_walking.txt

```
According to `dining_the_ridgeway_cafe.txt` (noting a second-year student's experience), the wait time at 12:30 is 10 to 15 minutes. This is also confirmed in `dining_the_ridgeway_cafe_followup.txt`.
```

### What is the wait time at 12:30 PM in ridgeway cafe — run 2

- Best distance: 0.1306 (passed the gate)
- Sources retrieved: dining_the_ridgeway_cafe.txt, dining_the_ridgeway_cafe_followup.txt, transit_walking.txt

```
According to a second-year student in `dining_the_ridgeway_cafe.txt` (and confirmed in `dining_the_ridgeway_cafe_followup.txt`), the wait time at 12:30 is 10 to 15 minutes.
```

### What is the wait time at 12:30 PM in ridgeway cafe — run 3

- Best distance: 0.1306 (passed the gate)
- Sources retrieved: dining_the_ridgeway_cafe.txt, dining_the_ridgeway_cafe_followup.txt, transit_walking.txt

```
The wait time at 12:30 at The Ridgeway Café is 10 to 15 minutes, according to *dining_the_ridgeway_cafe.txt* (noted by a second-year student) and *dining_the_ridgeway_cafe_followup.txt*.
```

### What is the maximum number of hours we can work in a week during the term? — run 1

- Best distance: 0.3809 (passed the gate)
- Sources retrieved: course_engl_205_workload.txt, course_math_220_workload.txt, course_phys_130_workload.txt, course_stat_150_workload.txt, money_jobs.txt

```
The maximum number of hours you can work in a week during the term is 20 hours (money_jobs.txt).
```

### What is the maximum number of hours we can work in a week during the term? — run 2

- Best distance: 0.3809 (passed the gate)
- Sources retrieved: course_engl_205_workload.txt, course_math_220_workload.txt, course_phys_130_workload.txt, course_stat_150_workload.txt, money_jobs.txt

```
The maximum number of hours you can work on campus in a week during the term is 20 hours (money_jobs.txt).
```

### What is the maximum number of hours we can work in a week during the term? — run 3

- Best distance: 0.3809 (passed the gate)
- Sources retrieved: course_engl_205_workload.txt, course_math_220_workload.txt, course_phys_130_workload.txt, course_stat_150_workload.txt, money_jobs.txt

```
The maximum number of hours you can work in a week during the term is 20 hours. 

Source: `money_jobs.txt`
```

### How many exams are there for Econ 101? Is the class curved? — run 1

- Best distance: 0.2606 (passed the gate)
- Sources retrieved: course_cs_210.txt, course_cs_210_exams.txt, course_econ_101.txt, course_econ_101_exams.txt

```
Based on the provided documents, the number of exams for Econ 101 is not mentioned. However, the class is curved, and generously (*course_econ_101.txt* and *course_econ_101_exams.txt*).
```

### How many exams are there for Econ 101? Is the class curved? — run 2

- Best distance: 0.2606 (passed the gate)
- Sources retrieved: course_cs_210.txt, course_cs_210_exams.txt, course_econ_101.txt, course_econ_101_exams.txt

```
Based on the provided documents, it is not mentioned how many exams there are for Econ 101 (`course_econ_101.txt` and `course_econ_101_exams.txt`). However, the class is curved, and generously, according to both `course_econ_101.txt` and `course_econ_101_exams.txt`.
```

### How many exams are there for Econ 101? Is the class curved? — run 3

- Best distance: 0.2606 (passed the gate)
- Sources retrieved: course_cs_210.txt, course_cs_210_exams.txt, course_econ_101.txt, course_econ_101_exams.txt

```
Based on the provided documents, the number of exams for Econ 101 is not mentioned. However, the class is curved, and generously (*course_econ_101.txt* and *course_econ_101_exams.txt*).
```

### Is stat 150 more busy in the beginning or end of the semester? How much time can I expect to spend on this class? — run 1

- Best distance: 0.2758 (passed the gate)
- Sources retrieved: course_biol_160_workload.txt, course_stat_150.txt, course_stat_150_workload.txt

```
STAT 150 is front-loaded, meaning the first month is heavier than the rest of the semester (from `course_stat_150_workload.txt`). You can expect to spend 5 to 6 hours a week outside of class (from `course_stat_150.txt` and `course_stat_150_workload.txt`).
```

### Is stat 150 more busy in the beginning or end of the semester? How much time can I expect to spend on this class? — run 2

- Best distance: 0.2758 (passed the gate)
- Sources retrieved: course_biol_160_workload.txt, course_stat_150.txt, course_stat_150_workload.txt

```
STAT 150 is front-loaded, meaning the first month is heavier than the rest of the semester (from *course_stat_150_workload.txt*). You can expect to spend 5 to 6 hours a week outside of class (from *course_stat_150.txt* and *course_stat_150_workload.txt*).
```

### Is stat 150 more busy in the beginning or end of the semester? How much time can I expect to spend on this class? — run 3

- Best distance: 0.2758 (passed the gate)
- Sources retrieved: course_biol_160_workload.txt, course_stat_150.txt, course_stat_150_workload.txt

```
STAT 150 is front-loaded, meaning the first month is heavier than the rest of the semester (from *course_stat_150_workload.txt*). You can expect to spend 5 to 6 hours a week outside of class (from *course_stat_150.txt* and *course_stat_150_workload.txt*).
```

### I don't like taking exams so is hist 118 a good fit for me? I want to do well so when are the rubrics released? — run 1

- Best distance: 0.4902 (passed the gate)
- Sources retrieved: course_hist_118.txt, course_hist_118_exams.txt, course_hist_118_workload.txt

```
Yes, HIST 118 is a good fit for you because it has no exams, as stated in `course_hist_118.txt`. The essay rubrics are released in week 2, according to `course_hist_118.txt` and `course_hist_118_exams.txt`.
```

### I don't like taking exams so is hist 118 a good fit for me? I want to do well so when are the rubrics released? — run 2

- Best distance: 0.4902 (passed the gate)
- Sources retrieved: course_hist_118.txt, course_hist_118_exams.txt, course_hist_118_workload.txt

```
Yes, HIST 118 is a good fit for you because there are no exams; the assessment consists of two essays and a final project (course_hist_118.txt). The essay rubric is posted in week 2 (course_hist_118.txt, course_hist_118_exams.txt).
```

### I don't like taking exams so is hist 118 a good fit for me? I want to do well so when are the rubrics released? — run 3

- Best distance: 0.4902 (passed the gate)
- Sources retrieved: course_hist_118.txt, course_hist_118_exams.txt, course_hist_118_workload.txt

```
Yes, HIST 118 is a good fit for you because there are no exams; assessment consists of two essays and a final project (course_hist_118.txt). The essay rubric is released in week 2 (course_hist_118.txt, course_hist_118_exams.txt).
```
