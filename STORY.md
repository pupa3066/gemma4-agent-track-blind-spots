# Story

## Why

A coding agent can only fix what it can find. Most agents find code with a retriever, and teams pick retrievers from
comparisons. In an earlier study for this project, keyword search beat embedding search by a wide margin. The study
had indexed only part of each repository for the embedding search, to fit a small machine, while keyword search saw
everything. The comparison was unfair, and nothing in the numbers showed it.

## What

This repository measures that kind of blind spot. When both retrievers see the whole repository, they are equivalent;
when one sees part of it, it loses exactly on the questions whose answers it never indexed. A second blind spot is
chunk length: an embedding model ignores text past its input window. The same check, applied to the competition's
own code index, found that it contains none of the async functions in fastapi, the repository behind half the
published tasks.

## What happened, in order

- 12 November: the Paper Track writeup, "What a Code Search Index Cannot See", was submitted (tag v1.0-paper). Two
  agent submissions had scored 0.00; the self-test harness showed why (a skill name with an underscore).
- 12 to 13 November: the agent comparison in the writeup turned out to be invalid. The BM25 skill ran outside the
  agent's sandbox and searched its own folder; a correction was posted on 14 November. A grep run inside the sandbox replaced it.
  Leaderboard: 0.17 (arm L) and 0.15 (arm G).
- 14 November: a reference-fix control found that only 72 of the 129 published tasks can be scored locally, as other
  participants had reported earlier; every comparison since uses those 72. Failure analysis: the 31B agent rarely
  misses the file (3 of 45 failures) and mostly writes a wrong fix in the right file (26).
- 14 to 15 November: small-model (E4B) work. Ranked chunk search cut its file misses from 13 to 5 but did not raise
  solved tasks; told the exact files, it still solved only 5 of 46. A larger budget (B) solved 30 against 27 locally,
  then exceeded the scorer's 12-hour limit and got no score.
- 16 November: all results were put under one Holm correction, and analysis rules were fixed for later tests
  (families declared in advance, detectable effect stated first, same-session controls).
- 16 to 18 November: the retrieval findings were rerun on all 300 SWE-bench Lite issues. Coverage and the chunk-length
  effect replicated on real issues.
- 17 to 18 November: arm N was submitted after a pre-submission smoke test and scored 0.17 after about 21 hours in the
  scoring queue, the same as arm L; arm G took the next slot.

## What held

Five results hold after Holm correction over all 27 primary tests: coverage decides lexical versus dense on two
docstring sets; chunks past the embedder window hide code (best-segment scoring fixed 56 questions and broke 2); on
300 real issues, dense retrieval with all code indexed found the changed code for 0.263 of issues against 0.143 with a
quarter of it; and at equal returned text, 8-line chunks beat 400-word chunks for dense retrieval (0.523 vs 0.377),
while BM25 went the other way. The async gap in the competition index is a measured fact, first reported by
another participant and confirmed by the organizers. Details and every test: `research/RESULTS_LEDGER.md`.

## What did not work

None of the 20 registered agent tests detected a difference in solved tasks: grep first, a BM25 skill, windowed reads,
a larger budget, testing before submitting, a rule to create new files, ranked chunk search and a fix check. The
largest net difference was 3 tasks, below what 72 tasks can detect. Ranked chunk search used 15.8 percent more tokens,
against the prediction. A retrieval hit on the problem statement did not predict a solved task. The larger budget
broke the runtime limit. The first distillation adapter for the small model was never completed.

## What we would do differently

Measure where the agent fails before choosing what to change: the failure analysis came two weeks in, and it showed
the levers being tested aimed at a problem the 31B agent did not have. Run the reference-fix control on day one. State the
detectable effect before every test and run both arms in one session. Check that a whole run fits the scorer's time
limit before submitting. Use every daily leaderboard slot from the start, because one score varies by two to three
tasks and the board keeps the best.

## Where things are

- Tag `v1.0-paper`: the writeup exactly as submitted; never moved.
- Branch `main`: everything after the writeup: plans, amendments, results, the ledger and this story.
- Branch `feat/small-model-e4b`: the small-model arms and their results, merged with `main`.
- `research/PREREG_20day_agent_plan.md`: every agent test, registered before its result, with dated amendments.
- `research/PREREG_tiered_context_gemma_harness.md`: the retrieval tests on real issues, including SWE-bench Lite.
- `benchmark/`: code and data for every table; `benchmark/KAGGLE_NOTEBOOKS.md` names each Kaggle notebook and what it
  showed.
