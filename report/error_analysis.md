# Qualitative error analysis

From the 1,514 test sentences we flagged the hardest cases: high-CMI and wrong under the modal
prediction of >=4 of 5 models. This gave 121 candidates; 40 were sampled with seed 42. Of these, 28
were wrong in all five models and 12 in four. Each was read by hand and assigned one of five categories
(`results/qualitative_review_sample_categorized.csv`).

| Category | n | What goes wrong | Example sentence_id |
|---|---:|---|---|
| neutral_lexically_charged | 15 | Neutral text with charged vocabulary; models read the words, not the stance | 43004 |
| sarcasm_irony | 10 | Sentiment flips on tone, not lexical content | 31968 |
| text_quality_noise | 5 | Garbled or repeated tokens break the signal | 12988 |
| entity_domain_knowledge | 5 | Sentiment hinges on a named person/event | 14075 |
| conflicting_signals | 5 | Genuine mix of positive and negative cues, including 2 non-compositional idioms (ids 32282, 4840) | 37438 |

sentence_ids per category:
- neutral_lexically_charged: 2451, 28655, 18529, 37726, 19438, 10057, 30306, 43004, 40336, 15694, 30203, 25815, 21139, 25378, 1410
- sarcasm_irony: 8891, 19441, 31968, 40361, 17949, 14454, 6506, 287, 34739, 33402
- text_quality_noise: 12988, 29917, 5260, 22228, 13513
- entity_domain_knowledge: 44763, 28267, 14075, 13593, 23517
- conflicting_signals: 9033, 37438, 14731, 32282, 4840

## Findings
1. Lexically-charged neutrals are the dominant mode (15/40, 37.5%). Direction splits roughly evenly
   (about 7 pushed positive, 8 negative), which suggests models key on lexical intensity, not stance.
2. Sarcasm/irony is second (10/40, 25%). Together: 25/40 = 62.5%, so the hardest errors are
   semantic/pragmatic rather than tokenization or data-quality problems.
3. 28/40 were wrong in all five models, including HingBERT and HingRoBERTa, so Hinglish-specific
   pretraining does not rescue these cases. This is consistent with the near-zero robustness gap.

## Neutral over-representation
Neutral is 50.0% (20/40) of the hard-error sample versus 37.3% of both the full corpus and the test
split. Of the 20 neutral-true errors, 15 are neutral_lexically_charged, 2 entity_domain_knowledge,
2 sarcasm_irony, 1 text_quality_noise. This corroborates the Phase 2.4 per-class F1 hint that neutral
is the weak class, and names the mechanism.

## Caveats
Two sentences first labelled `mixed_idiom` (non-compositional idioms) were folded into `conflicting_signals` because the category was too thin to stand alone. The sample of 40 is illustrative, not a census.
