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

## License

Apache License 2.0. See `LICENSE`.
