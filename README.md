# gemma4-agent-track-blind-spots

Code and data for the Gemma 4 Developer Agent Paper Track writeup "What a Code Search Index Cannot See: Coverage,
Chunk Length, Missing Async Code".

## Findings

| Finding | Evidence | Files |
|---|---|---|
| The competition's code index contains no async functions | fastapi task snapshot: 914 async functions in the source, 0 in the code graph; 0 of 375 async-only names | `benchmark/harness_audit/` |
| Index coverage decides lexical versus dense comparisons | BM25 leads by 0.20 to 0.53 with part of the code indexed; equivalent within 0.10 with all of it (6 repositories, 480 questions) | `benchmark/CONFOUND_lines8*.json`, `benchmark/SWEEP_*.json` |
| Coverage accounts for the gap | On questions whose answer was indexed, dense retrieval found it 88 to 92 percent of the time at every index size | `benchmark/CONFOUND_lines8_gold*.json` |
| Chunks past the embedder window hide code | Best-segment scoring on identical chunks fixed 56 questions and broke 2 (p = 1.2e-14) | `benchmark/CONFOUND_words400*.json` |
| The pattern holds on the toolkit's own repository | Same direction, N=34 | `benchmark/kb_selftest/` |
| Why two agent submissions scored 0 | The harness rejects a skill named `repo_navigation` (names must be lowercase kebab-case); the agent never compiled | `benchmark/gemma_agent/selftest_results/` |

## Reproduce

- **Every table and figure:** `benchmark/rebuild_all_tables_and_figures.ipynb`. Runs on a Kaggle CPU notebook and reads only
  committed result files. Also published as the Kaggle notebook `purnimapathak15/cck-results-public`.
- **Retrieval experiments:** `benchmark/confound_sweep.py` and `benchmark/conceptual_fair_metrics.py`, run on Kaggle
  CPU by the `benchmark/*.ipynb experiment notebooks` notebooks. Each experiment notebook records the commit of the code it ran.
- **Harness audit:** `benchmark/harness_audit/async_unique_check.ipynb` with the competition data attached.
- **Index audit tool for any repository or knowledge base:** `benchmark/kb_index_audit.py`
  (`python benchmark/kb_index_audit.py --root <repo> --policy all --policy md-only --chunker lines8`).
- **Agent self-test:** `benchmark/gemma_agent/diag_agent_selftest.ipynb` on a Kaggle L4 GPU machine with the
  competition data, the dataset `metric/gemma-4-developer-agent-wheelhouse` and the model
  `google/gemma-4/Other/gemma-4-31b-it-qat-w4a16-ct/2`. It runs an agent beside the organizers' sample agent on
  published tasks and reports, per task, resolution, patch size, tool calls and every tool error.
- **Submission check:** `python benchmark/gemma_agent/validate_submission.py <dir> [--zip out.zip]`, including the
  kebab-case skill-name check that would have caught the zero scores.

## Layout

```
benchmark/                 experiment code, result files (JSON), notebooks, results log
benchmark/harness_audit/   async-function audit of the competition code graph
benchmark/kb_selftest/     the same experiment and audit on the toolkit's own repository
benchmark/gemma_agent/     agent configurations, validator, self-test harness and its results
research/                  pre-registrations with dated amendments
figures/                   the writeup's figures and card image
```

Agent configurations: `submission_v81_lexical` (a BM25 skill meant to search every file; traces later showed it searched only its own folder, amendment 4), `submission_v81_noskill`,
`submission_v81_cov50` (BM25 over half the files), `submission_v81_noasync` (BM25 blind to async functions),
`submission_v8_*` (earlier corrected arms) and `submission_v7_lexical` (the arm that scored 0, kept as evidence of the
skill-name failure).

## Provenance

These files were taken unchanged from the project repository pupa3066/context-configuration-builder at commit
21e8dcb, except that the results notebook reads this repository. The results log `benchmark/RESULTS_LOG.md` is that
repository's `benchmark/CURRENT_STATE.md`. Early result files from a local run on a single repository (N=80) and
notebooks that reimplemented the chunker are not included; the results log explains why they are invalid.
The results log cites three earlier result files, now included unchanged from commit 21e8dcb: `benchmark/POWERED_RESULTS.json`, `benchmark/CONCEPTUAL_RESULTS.json` and `benchmark/CONCEPTUAL_FAIR_RESULTS.json`. `benchmark/CURRENT_STATE.md` is the file published here as `benchmark/RESULTS_LOG.md`. `benchmark/cck_context.py` is toolkit code, not evidence for the writeup, and stays in the project repository. The agent prompts keep the organizers' sample prompt verbatim, including its punctuation. Arms added after the writeup (G, W, F and the tiered-context study) are on branch feat/small-model-e4b and in the project repository.

## Our stance, and the evidence for it

**What an index cannot see or read, an agent cannot find.** Coverage and chunk length decide what a retriever returns,
and comparisons that ignore them crown false winners. Inside a coding agent, full-coverage search over small chunks brings
localization close to its ceiling; past that point, task success is bounded by the model's ability to write the fix.

| Claim | Evidence | Status |
|---|---|---|
| Coverage decides retrieval | Docstring questions, two sets of three repositories: dense retrieval rises from 0.45 and 0.58 to 0.89 as its index goes from 1,000 chunks to everything, while BM25, which always searches everything, stays constant; Holm-adjusted p below 1e-11. Dense finds indexed answers 0.88 to 0.92 of the time at every budget | Holds after correction |
| The same on real issues | SWE-bench Lite, django, 114 real issues: dense 0.342 at full coverage vs 0.210 at 25 percent (p = 0.006; 0.059 after Holm over 13 tests) | One shard; the registered test pools all shards, still running |
| Hidden coverage crowns false winners | BM25 led dense by 0.20 to 0.53 when dense saw part of the code; within 0.10 on every metric when both saw all of it (paired equivalence tests). The competition's own code index omits all 914 fastapi async functions, confirmed by the organizers | Holds |
| Chunk length hides code from the embedder | The embedder ignores text past its input window (direct probe); scoring each chunk by its best in-window segment fixes 56 questions and breaks 2 (Holm-adjusted p = 1e-13) | Holds after correction |
| Full-coverage chunked search reaches near-oracle localization in an agent | Small model (E4B), 46 scorable tasks: never finding the right file fell from 13 to 5 with ranked chunk search, and to 0 when told the files; tasks solved 3 and 4 with ranked search vs 5 when told the files (not distinguishable). The 31B agent misses the right file in 3 of 45 failures | Measured; small N |
| Beyond localization, the limit is writing the fix | Told exactly which files to change, E4B still fails 41 of 46 tasks: 39 edit the right file with a wrong fix | Measured |

Not supported, and stated as such: a token saving from full-coverage search (31B: -2.7 and +7.0 percent; small model
+15.8 percent), and more solved tasks from retrieval changes alone (every agent comparison p = 0.62 or higher). Details,
including which tests were registered before their results and the Holm table: `research/RESULTS_LEDGER.md`.

## Updates after the writeup (12 November 2026)

The writeup describes tag v1.0-paper. Work after it is on main, each item registered before its result:

| When (MT) | What | Why |
|---|---|---|
| 15:45 | 16-task self-tests of C50 (6 of 16) and A0 (5 of 16); no difference from L detected | Complete the four registered arms on published tasks |
| 15:55 | Results log: m3 share of gap removed is 57% (27 of 47), not 58% | Exact count; the data files were already correct |
| 16:00 | The organizers' discussion "Note on Code Graphs for Async Functions" (5 October) confirms the async omission and reports that no scored private task touches an async function | The async gap affects published tasks, not leaderboard scores; the writeup cites the note |
| 16:03 | Results notebook reads tag v1.0-paper by default | It always rebuilds the writeup's figures byte for byte |
| 16:08 | Plan amendment 1: A0 becomes a negative control on the leaderboard | Follows from the organizers' note |
| 16:32 to 16:46 | Plan amendments 2 and 3: E4B arms (one change, the model), decision rule, slot schedule, final-pick rule | The organizers' small-model prize for Gemma E2B or E4B |
| 17:00 | Arm L submitted to the leaderboard (ref 58287913) | Day 1 of the 20-day plan |
| 17:03 | E4B self-tests on 16 published tasks: L-E4B 2, N-E4B 1 (31B: 4 and 4); E = L-E4B for day 2 | Outcome of amendments 2 and 3, recorded as registered |
| 17:09 | Defect: in every BM25 arm the skill script ranked its own folder, not the repository (skill scripts run outside the agent's sandbox). The agent comparisons in the writeup's Section 5 and above therefore do not test BM25 or coverage; the retrieval experiments of Sections 3.1 to 3.6 are unaffected. E is withdrawn | Plan amendment 4, recorded before arm L's leaderboard score |
| 19:10 | Corrected localizer (grep inside the sandbox) passes the trace gate: repository paths in 12 and 14 of 16 first outputs. 16 published tasks: G (31B) 5, N 4; G-E4B 1, N-E4B 1; no difference detected | Plan amendment 5; model work continues on branch feat/small-model-e4b |
| 13 Nov 08:52 | All 129 published tasks: G 30 (0.233) vs N 28 (0.217), p = 0.73; on the 21 async-fix tasks 4 vs 3. Arm L scored 0.17 on the leaderboard | Plan amendment 7, recorded as registered |
| 13 Nov 23:13 | Registered fastapi replicate (67 tasks): G 14 vs N 15, p = 1.0; the fastapi lead from the 129-task runs did not replicate. Arm G submitted to the leaderboard | Plan amendment 9, recorded as registered |
| 14 Nov 08:00 | Arm G scored 0.15 on the leaderboard (arm L: 0.17); one replicate each, within reported noise | Leaderboard record |
| 14 Nov 14:12 | Gold-patch control on all 129 published tasks: 72 are scorable locally; recomputed on them, the 31B agent resolves 37 percent, and G vs N stays 27 vs 26 | Amendment 17; files in benchmark/harness_audit/ |
| 14 Nov 17:27 | Ranked chunk search inside the sandbox for the small model (R-E4B), 46 scorable tasks: never finding the right file fell from 13 to 5; tasks solved 3 vs 3; tokens +15.8 percent | Amendment 20; the remaining failures are wrong fixes, targeted by RC-E4B (amendment 21) |
| 14 Nov | Retrieval on real problem statements: 125 competition tasks and all 300 SWE-bench Lite tasks, testing coverage and chunk length on real issues | Registered before results; running |

## Names used in this repository

- **The toolkit**: this work grew out of a context-configuration toolkit (repository pupa3066/context-configuration-builder,
  internal short name CCK). The short name survives only in Kaggle notebook addresses (kaggle.com/code/purnimapathak15/cck-...),
  in the result files of the toolkit's self-application (`benchmark/kb_selftest/*cckself*`, Section 3.6 of the writeup), and in
  file paths inside the frozen tag v1.0-paper. Kaggle addresses cannot be renamed without breaking links and outputs.
- **What each Kaggle notebook did**: `benchmark/KAGGLE_NOTEBOOKS.md` gives every notebook a descriptive name, why it ran and
  what it showed.
- **Notebook files**: on main, the experiment notebooks are named by what they test (for example
  `coverage_sweep_8line_chunks_set1.ipynb`, `chunk_length_best_segment_and_window_probe.ipynb`,
  `rebuild_all_tables_and_figures.ipynb`). The writeup cites tag v1.0-paper, where the earlier file names
  (`cck_kaggle_*.ipynb`) remain unchanged.
- **Agent arms**: L (BM25 skill, which searched its own folder; amendment 4), N (no search step), G (grep over every Python
  file inside the sandbox), W (G plus 40-line reads), C50 and A0 (BM25 over half the files, BM25 skipping async code), and
  "-E4B" for the same arm on the small model.

## References and credits

Public work in the Gemma 4 Developer Agent competitions that this repository cites or uses (Kaggle, 2026):

| Work | Used for |
|---|---|
| Organizers' getting-started notebook, kaggle.com/code/ryanholbrook/getting-started-gemma-4-developer-agent | The self-test harness's install, model-serving and evaluator cells |
| zbigniewkirchner, kaggle.com/code/zbigniewkirchner/gemma-4-agent-run-your-agent-locally | The approach of evaluating agents locally on published tasks |
| honghanhhh, kaggle.com/code/honghanhhh/gemma-4-baseline-lb-0-24, and flexonafft, kaggle.com/code/flexonafft/gemma-4-budget-aware-coding-agent-v2 | The eval_config.yaml budget and sampling settings of the v8 arms |
| yiyu0716, kaggle.com/code/yiyu0716/g4-v19-complete-harness-source | The run_skill_script calling convention for skill scripts |
| zzgtylors, kaggle.com/code/zzgtylors/graphloc-end-to-end-gemma-4-agent-skill | Prior BM25 localization skill; noted that the graph tools miss async functions |
| Prasetya, kaggle.com/code/fabrianivan/gemma-4-swe-agents-three-paradoxes-ca-hgr | Prior audit of the competition code graph |
| Jain, kaggle.com/code/hitarthjain0/gemma-4-agent-what-took-lb-0-12-0-24 | Report that strong public configurations avoid the graph tools |
| Maroof, kaggle.com/code/maroofiums/ast-compressed-code-graphs-for-gemma-4-agents | Evidence that the unit of indexing matters |
| Mobassir, kaggle.com/code/mobassir/why-your-leaderboard-can-t-see-a-3-point-gain | Leaderboard noise |
| Valiyavalappil et al., Paper Track writeup "Action-Masked LoRA and Empirical Failure Modes in Gemma 4 SWE Agents" | Report that literal search beats the graph tools |

Gloaguen et al. (arXiv:2602.11988) on monolithic repository context files.

## Authors

Purnima Pathak and Akshay Upadhyay.

## License

Apache License 2.0. See `LICENSE`.
