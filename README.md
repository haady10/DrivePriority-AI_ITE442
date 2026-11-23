# DrivePriority AI

**Evaluating learned and SMART-based hard-drive failure rankings under limited inspection capacity**

ITE442 Data Science and Big Data Analytics, Abu Dhabi University, Fall 2026.
Haady Abdul Jabar V.A, Sulaiman Ahmed, Mohammed Shuhail, Mohammed Zahid and Faheem Mansoor.
Instructor: Dr. Laiali Almazaydeh.

> **The final code and results are on the branch [`phase5/simulation`](https://github.com/haady10/DrivePriority-AI_ITE442/tree/phase5/simulation).**
> `main` stops at the merge of the Phase 5 simulation; the post hoc analysis (amendment A1), the result archives
> and the last fixes were added on `phase5/simulation` afterwards.

## In one paragraph

Disk-failure models are usually judged by classification metrics, but an operator can inspect only a few drives a
day. We replay 2.5 years of the Backblaze fleet day by day with a capacity of 20 inspections per 10,000 drives,
next-day service, cancellations, a 7-day re-inspection cooldown and an expiring queue. A failure counts as caught
in time if a completed inspection came 7 to 30 days before it. With next-day re-inspection and fresh top-K
selection, a random forest caught **26.5%** of 6,271 eligible test failures and a five-indicator SMART count
**11.8%** (**+14.7 points**, 95% interval 13.8 to 15.6), but there the count inspected only one manufacturer.
With a 7-day cooldown, both scores rose by 17.7 to 31.7 points and no difference was established (+0.6 points,
−0.4 to 1.7). Post hoc, with fairer SMART baselines, no service rule gave either score a uniform advantage.
**Predicting which drives may fail is only part of the problem; how limited inspections are scheduled can matter
just as much as the model ranking them.**

**Scope.** This is a retrospective decision-support simulation. Nobody inspected these drives, a simulated
inspection changes nothing on a drive, and "failure" always means the failure flag recorded by Backblaze. The
results describe how inspection slots would have been allocated; they do not show that failures, downtime or
cost are prevented.

## Research questions

- **RQ1 (primary).** Under the same fleet and the same daily inspection capacity, how do the re-inspection rule
  (next day vs 7-day cooldown) and the scheduling rule (fresh top-K vs carry-forward queue) change the difference
  between a SMART-count rule and a machine-learning score, measured as the share of failures inspected 7 to 30
  days in advance?
- **RQ2.** What do alert-persistence rules (confirm, retain, EWMA) cost in timely capture for a steadier list?
- **RQ3.** How do 7-, 14- and 30-day warning horizons compare as predictors and as budgeted rankings?

## Data

Backblaze Drive Stats, Q1 2016 to Q2 2026: 42 quarterly archives, 709,242,263 daily records of 501,437 drives.
After removing solid-state drives: 497,500 hard drives and 28,987 failures. Split by time: training to 2023,
model choice on 2024, test period January 2025 to June 2026 (run once).

Source: Backblaze, Inc., Hard Drive Test Data (Drive Stats),
https://www.backblaze.com/cloud-storage/resources/hard-drive-test-data. The raw data are not redistributed here.

## Pipeline

| Phase | What | Code | Key documents |
|---|---|---|---|
| 1 | Per-quarter cleaning and validation of the 42 archives (four outcomes: passed, passed_with_warnings, requires_review, failed) | `src/drivepriority/data/`, `RUN_C1_2.ps1` | `docs/C1_2_MASTER_GUIDE.md`, `docs/C1_TEAM_README.md` |
| 1b | Central incremental integration into one read-only snapshot | `src/drivepriority/integration/`, `RUN_INTEGRATION.cmd`, `RUN_CENTRAL.cmd` | `docs/CENTRAL_INTEGRATION_GUIDE.md`, `docs/START_HERE_INTEGRATION.txt` |
| 2-3 | Data profile; labels (7/14/30 days), past-only features, date splits, HDD cohort | `src/drivepriority/features/`, `RUN_PHASE3.cmd` | `docs/data_pipeline_runbook.md` |
| 4 | H2O models (LR, GLM, GBM, random forest) and the SMART baseline; protocol fixed with file hashes before the final runs | `src/drivepriority/models/train_h2o.py` | `docs/protocol_frozen_before_test.md` |
| 5 | Full-fleet scoring and the stateful inspection replay (RQ1-RQ3), eight policies, ten tie seeds | `src/drivepriority/simulation/` | `docs/phase5_protocol.md`, `configs/phase5_protocol_v1.json`, `docs/phase5_runbook.md` |
| A1 | Post hoc (after the test results): fairer SMART baselines and manufacturer quotas | `src/drivepriority/posthoc/`, `RUN_POST_HOC.cmd` | `docs/phase5_amendment_A1_fair_baselines_2026-11-20.md`, `docs/phase5_amendment_A1_results_2026-11-20.md` |

Tools: Python, DuckDB, Pandas, PyArrow and Parquet for data processing; H2O 3.46.0.12 for the models; Power BI
for the dashboard.

## Evidence

- Frozen replay protocol: `configs/phase5_protocol_v1.json` (SHA-256 checked by the code).
- Test authorization for the single test replay: `configs/phase5_test_authorization.json`.
- Run record of the test replay, result tables and checks: `docs/phase5_test_run_record/`.
- Every reported number with its file and column: `docs/phase5_claims_and_limitations_2026-11-14.md`.
- Result archives of both replays (result tables, per-event capture results, service logs) with SHA-256
  checksums: `published_results/phase5/`; post hoc outputs: `published_results/phase5_A1/` and
  `published_results/phase5_post_test/` (see `published_results/README.md`).

## Running the checks

```bash
python -m venv .venv && . .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python -m pytest tests -q              # conftest.py puts src/ on the path
python tools/verify_results.py         # checks the result archives (pandas + pyarrow), a few seconds
```

On 23 November 2026 the suite, with the H2O setup test excluded, gave 420 passed and 5 skipped on Linux, and
`tools/verify_results.py` passed for the test period and 2024.

The Windows launchers (`*.cmd`, `*.ps1`) wrap the same Python entry points for the team's lab machines. The H2O
smoke test is opt-in because it needs Java and starts a local H2O cluster: set `DRIVEPRIORITY_RUN_H2O=1` to run it.

## Large files not in this repository

- **Saved H2O models** (12 models, about 5 MB): logistic regression, GLM, GBM and random forest for 7, 14 and 30
  days. They are H2O binary models and load only with `h2o.load_model()` in H2O 3.46.0.12.
- **Daily scores of the whole fleet** (about 1.7 GB) and the **integrated snapshot**.

They are available from the authors on request.

## Licence

No licence has been chosen yet; until one is added, all rights are reserved by the authors.
