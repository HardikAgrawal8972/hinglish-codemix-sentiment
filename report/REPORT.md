# Hinglish-Specific Pretraining and Robustness to Code-Mixing: A Sentiment Failure Analysis

*All numbers are copied from `results/*.csv`; nothing is recomputed.*

## 1. Abstract
We ask whether pretraining on Hinglish text makes sentiment models more robust as code-mixing
intensity rises. We fine-tune five models (mBERT, MuRIL, IndicBERT, HingBERT, HingRoBERTa; 3 seeds
each) on SentiMix and compare accuracy across tertiles of the Code-Mixing Index (CMI). The
Hinglish-specific models are more accurate overall (HingRoBERTa 0.701, HingBERT 0.696 vs. 0.634-0.661
for the rest), but the high-vs-mid CMI accuracy gap is approximately zero for all five models. A
close reading of the 40 hardest errors shows the failures are semantic and pragmatic rather than
tokenization-related.

## 2. Data and models
SentiMix Hindi-English, 15,131 tweets, custom stratified 80/10/10 split (seed 42), test n = 1,514.
Labels: negative/neutral/positive. Models and Hugging Face IDs as in the README. 15 runs
(5 models x seeds 42-44); predictions aggregated by modal vote per sentence.

## 3. Methods
- **CMI** (Das & Gambäck, 2014), computed once in Phase 0 and carried through unchanged. Bucketed by
  **tertiles** (33rd/67th percentile) using explicit thresholds (not `pd.qcut`, which collapses on the
  mass of monolingual tweets at CMI = 0).
- **Features:** length, switch_density = n_switches / max(1, n_lang_tokens - 1), hi_ratio, caps_ratio,
  oov_rate (wordfreq top-20k English, alphabetic tokens only). Provenance in `data/feature_spec.json`.
- **Robustness gap:** accuracy(low-CMI) minus accuracy(high-CMI), with a block bootstrap
  (2,000 iterations, resampling sentence_ids so all seed-rows travel together).
- **Logistic regression:** correct ~ cmi + switch_density + length + oov_rate; VIF-checked; features
  standardized on train statistics. hi_ratio omitted (correlated with oov_rate by construction).
- **McNemar:** 10 pairwise tests on modal predictions, Holm-corrected.

## 4. Quantitative results
**Accuracy** (3 seeds pooled, test n = 1,514; `results/phase4_analysis.csv`, macro-F1 from `sa_finetune_summary.csv`):

| Model | Type | Accuracy | Macro-F1 (mean ± sd) |
|---|---|---:|---:|
| HingRoBERTa | Hinglish | 0.694 | 0.696 ± 0.006 |
| HingBERT | Hinglish | 0.688 | 0.692 ± 0.004 |
| MuRIL | general | 0.655 | 0.658 ± 0.004 |
| mBERT | general | 0.628 | 0.631 ± 0.011 |
| IndicBERT | general | 0.623 | 0.627 ± 0.003 |

**Robustness gap** (accuracy(low-CMI) − accuracy(high-CMI), 2,000-iteration block bootstrap; `robustness_gap.csv`):

| Model | Gap | 95% CI |
|---|---:|---|
| HingBERT | +0.000 | [−0.053, +0.054] |
| HingRoBERTa | −0.005 | [−0.054, +0.044] |
| MuRIL | −0.005 | [−0.057, +0.045] |
| mBERT | −0.016 | [−0.071, +0.038] |
| IndicBERT | −0.025 | [−0.078, +0.032] |

Every CI includes zero, and the Hinglish-specific models show no smaller gap than the general ones.
The hypothesis that Hinglish pretraining buys robustness to code-mixing is not supported.
Figures: `figures/robustness_gap.png`, `figures/accuracy_by_cmi_bucket.png`.

**Logistic regression** (`correct ~ cmi + switch_density + length + oov_rate`, standardized; see
`logit_coefficients_summary.csv`). Effects are tiny: McFadden pseudo-R² is 0.0005–0.0016 for all models.
Nominally significant terms (uncorrected): length for HingBERT (p = 0.016) and HingRoBERTa (p = 0.020),
cmi for HingRoBERTa (positive, p = 0.017), oov_rate for IndicBERT (negative, p = 0.031). None would be
expected to survive a multiple-comparison correction across 20 tests, and CMI is not a negative
predictor of correctness for any model.

**McNemar** (modal predictions, Holm-corrected, `mcnemar_pairs.csv`): HingBERT and HingRoBERTa each
significantly outperform mBERT, MuRIL and IndicBERT (Holm p from 1.8e-7 to 0.005). HingBERT vs HingRoBERTa
and all general-vs-general pairs are not significant. So Hinglish pretraining improves overall accuracy but,
per the gap analysis, not robustness.

## 5. Error analysis
Sampling: high-CMI (tertile) and wrong under the modal prediction of >=4/5 models gave 121
candidates; 40 sampled (seed 42), of which 28 were wrong in all five models.

| Category | n |
|---|---:|
| neutral_lexically_charged | 15 |
| sarcasm_irony | 10 |
| text_quality_noise | 5 |
| entity_domain_knowledge | 5 |
| conflicting_signals | 3 |
| mixed_idiom | 2 |

Lexically-charged neutrals and sarcasm make up 25/40 (62.5%). Neutral is 50% (20/40) of the sample,
against 37.3% of both the full corpus and the test split. Of the 20 neutral
errors, 15 are lexically charged, which names the mechanism behind neutral being the weak class.
Examples and full sentence_id lists: `report/error_analysis.md`.

## 6. Limitations
- **Feature bug (Phase 3):** LANG_LABELS was `{'Hin','Eng'}` while the data uses `HI`/`EN`, so
  switch_density and hi_ratio were silently constant 0.0 across all 15,131 rows. Fixed and
  regenerated; only post-fix numbers are reported.
- **Analysis bug (Phase 4):** a cell-order error in VIF/standardization cascaded through the
  logistic regression; fixed by consolidating into one cell.
- **Bucket-table inconsistency:** the saved `cmi_bucket_accuracy_bootstrap.csv` (and so `phase4_analysis.csv`'s `acc_mid`/`acc_high` columns) contains only mid and high buckets, with no low bucket, while the gap in `robustness_gap.csv` is computed as low minus high in the notebook. The gap values and CIs above are the notebook's; the per-bucket table should be regenerated for consistency (the notebook caches results to Drive, so delete the cached CSVs and re-run).
- Single dataset; CPU-only analysis; modest model sizes.
- The 40-sentence sample is illustrative, not a census. mixed_idiom (n = 2) is a thin category and
  could fold into conflicting_signals. It is kept separate because its mechanism (non-compositional idiom) differs.

## 7. Reproducibility
Seeds 42/43/44, pinned versions in `requirements.txt`, per-sentence predictions in `preds/`, repo: https://github.com/HardikAgrawal8972/hinglish-codemix-sentiment.
