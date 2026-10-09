# Does Hinglish-specific pretraining buy robustness to code-mixing?

A failure analysis of five transformer models on Hindi-English (Hinglish) code-mixed sentiment
analysis (SentiMix, SemEval-2020 Task 9).

**Short answer:** Hinglish-specific models (HingBERT, HingRoBERTa) are more accurate overall, but
**not more robust** to high code-mixing: the accuracy gap between low- and high-CMI sentences is
about 0 for every model (all within ±0.025, all bootstrap CIs include zero). The hardest errors are
semantic/pragmatic (charged-but-neutral text, sarcasm), not tokenization.

## Setup
- 5 models x 3 seeds (42, 43, 44) = 15 fine-tuning runs: mBERT, MuRIL, IndicBERT, HingBERT, HingRoBERTa
- Dataset: 15,131 tweets, stratified 80/10/10 split (seed 42); test = 1,514 sentences
- Config: 3 epochs, lr 2e-5, batch 16/32, weight decay 0.01, warmup 100, max_len 128, fp16, best by macro-F1

## Results summary
| Model | Modal accuracy (test) |
|---|---:|
| HingRoBERTa | 0.701 |
| HingBERT | 0.696 |
| MuRIL | 0.661 |
| IndicBERT | 0.635 |
| mBERT | 0.634 |

(Per-row accuracy pooled over 3 seeds is in the report; source: `results/phase4_analysis.csv`. The table above uses modal-vote accuracy.)

Qualitative: 40 hardest errors (high-CMI and wrong in >=4/5 models) fall into six modes; lexically
charged neutrals (15) and sarcasm (10) account for 62.5%. See `report/REPORT.md`.

## Repo layout
```
notebooks/   one canonical notebook per phase (run in order, Google Colab)
preds/       per-sentence predictions (all_predictions_long.csv)
results/     analysis CSVs + hand-categorized error table
figures/     accuracy_by_cmi_bucket.png, robustness_gap.png, pointbiserial_heatmap.png
data/        data README + feature_spec.json (raw data not redistributed)
report/      REPORT.md
```

## Reproducing
1. Obtain SentiMix via LinCE (see `data/README.md`).
2. `pip install -r requirements.txt`; IndicBERT is gated: set `HF_TOKEN` and accept the model terms.
3. Run notebooks in order: phase1 -> phase2_1 -> phase2_2 (sanity) -> phase2_3 -> phase2_4 -> phase3 -> phase4 -> phase5.

## Licences
Code: MIT. Dataset and model licences are those of their original authors.
