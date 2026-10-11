# Data

## Source
SentiMix Hindi-English (SemEval-2020 Task 9), obtained via LinCE (`sa_hineng`), CoNLL format.
Labels: `{negative: 0, neutral: 1, positive: 2}`. Language tags: `HI` / `EN` / `OTHER` (uppercase).

## Redistribution
**Raw tweet text is NOT included in this repo.** SentiMix/LinCE data comes from Twitter and its
terms may permit sharing tweet IDs only; the licence terms were not confirmed, so this repo takes the conservative route:
- Default (safe): ship `sentence_id`s + split assignment + derived CSVs only; users rebuild
  `sa_master.parquet` from their own copy of LinCE with `notebooks/phase2_1_build_master.ipynb`.
- `results/qualitative_review_sample_categorized.csv` has the tweet text column removed. Look sentences up by `sentence_id` in your own copy of the dataset.

## Split
Custom stratified 80/10/10 train/val/test, seed 42 (SentiMix shipped only train+val).
Test split = 1,514 sentences; predictions exist for test only. 15,131 sentences total.

## Join gotcha
`sa_master.parquet` stores `sentence_id` as string; CSVs in `preds/` and `features/` round-trip it as
int64. Cast both to `str` before any merge.

## Files
- `feature_spec.json` -- feature definitions and vocab provenance (copy from `features/` on Drive).
