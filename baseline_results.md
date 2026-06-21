# Baseline Results — Groq `llama-3.3-70b-versatile` (zero-shot)

Run in Milestone 4, on the locked test set, **before** any fine-tuning.
Same test set and label definitions that the fine-tuned model will be scored on.

## Numbers

- **Overall accuracy: 0.351** (37/37 responses parseable — 0% unparseable)
- **Macro-F1: 0.25**

| label | precision | recall | F1 | support |
|---|---|---|---|---|
| critique | 0.00 | 0.00 | 0.00 | 13 |
| reaction | 0.41 | 0.71 | 0.52 | 17 |
| recommendation | 0.50 | 0.14 | 0.22 | 7 |
| **accuracy** | | | **0.35** | 37 |
| **macro avg** | 0.30 | 0.28 | **0.25** | 37 |

## Interpretation

The zero-shot model performs barely above chance (3-class random ≈ 0.33 accuracy).
It is essentially unusable on this task, which makes the fine-tuned model's
improvement meaningful.

## Hypothesis (recorded before fine-tuning) — and a correction

- **Original hypothesis:** the baseline would *over-predict `critique`*, confusing
  emotional analysis for reasoned critique.
- **What actually happened (REFUTED):** the baseline *collapses toward the majority
  class `reaction`* (recall 0.71, ~29 of 37 predictions) and is **completely blind to
  `critique`** (precision = recall = F1 = 0 on 13 examples). It does not over-predict
  critique at all — it under-predicts it to zero.
- **Revised hypothesis to test after fine-tuning:** fine-tuning's main job is to
  *recover the `critique` class* and stop the model from dumping everything into
  `reaction`. The `critique` ↔ `reaction` boundary is still the hard one — but the
  error direction is reaction-ward, not critique-ward.
