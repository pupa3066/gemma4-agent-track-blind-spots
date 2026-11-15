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

Two agent submissions scored zero along the way. A self-test harness built on the organizers' pipeline showed why: a
skill name with an underscore made every task fail to compile, and the leaderboard reported a completed score of zero.

## How it connects

The experiments, the audit and the agent work use one idea: before trusting a comparison or an index, check what it
can and cannot see. The index audit tool and the checklist in the writeup turn that into steps anyone can run. The
pre-registered plan in `research/` carries the same question to agent task success on the competition leaderboard.

After the writeup was submitted, the organizers' own note confirmed the async finding and narrowed its scope: none of
the scored private tasks touch async code, so the gap matters for the published tasks that teams develop against, not
for leaderboard scores. The plan was amended before any affected result, and a small-model arm was added for the
organizers' small-model prize. Each change is dated in the README.

A gold-patch control then showed that 72 of the 129 published tasks can be scored locally, so every agent comparison
is now made on those tasks. On them, the 31B agent rarely misses the code it needs (3 of 45 failures), while the small
model often does (13 of 43). The writeup's method, ranked search over small chunks of every file, cut the small model's
misses from 13 to 5. Its remaining failures are fixes it writes and submits despite its own failing tests, which is the
next registered test. The retrieval findings are also being tested on real issue texts from 125 competition tasks and
from SWE-bench Lite.

