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

- **Every table and figure:** `benchmark/cck_kaggle_results_public.ipynb`. Runs on a Kaggle CPU notebook and reads only
  committed result files. Also published as the Kaggle notebook `purnimapathak15/cck-results-public`.
- **Retrieval experiments:** `benchmark/confound_sweep.py` and `benchmark/conceptual_fair_metrics.py`, run on Kaggle
  CPU by the `benchmark/cck_kaggle_*.ipynb` notebooks. Each experiment notebook records the commit of the code it ran.
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

Agent configurations: `submission_v81_lexical` (BM25 skill over every file), `submission_v81_noskill`,
`submission_v81_cov50` (BM25 over half the files), `submission_v81_noasync` (BM25 blind to async functions),
`submission_v8_*` (earlier corrected arms) and `submission_v7_lexical` (the arm that scored 0, kept as evidence of the
skill-name failure).

## Provenance

These files were taken unchanged from the project repository pupa3066/context-configuration-builder at commit
21e8dcb, except that the results notebook reads this repository. The results log `benchmark/RESULTS_LOG.md` is that
repository's `benchmark/CURRENT_STATE.md`. Early result files from a local run on a single repository (N=80) and
notebooks that reimplemented the chunker are not included; the results log explains why they are invalid.

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
