# Results — Baseline vs Fine-tuned (final run)

Both models scored on the **same locked test set** (37 examples) after the label
re-audit and re-split. Baseline = Groq `llama-3.3-70b-versatile` zero-shot;
fine-tuned = `distilbert-base-uncased`. Full export in `evaluation_results.json`,
confusion matrix in `confusion_matrix.png`.

## Headline

| metric | baseline | fine-tuned | Δ |
|---|---|---|---|
| accuracy | 0.459 | **0.514** | +0.055 |
| **macro-F1** | 0.31 | **0.45** | **+0.14** |

## Per-class

| label | baseline P/R/F1 | fine-tuned P/R/F1 | support |
|---|---|---|---|
| critique | 0.50 / 0.23 / 0.32 | 0.44 / 0.31 / 0.36 | 13 |
| reaction | 0.48 / 0.82 / 0.61 | 0.54 / 0.76 / 0.63 | 17 |
| recommendation | 0.00 / 0.00 / 0.00 | 0.50 / 0.29 / 0.36 | 7 |

## Verdict against the success criteria (planning.md §6)

- **Relative (beat baseline): MET.** Fine-tuned macro-F1 0.45 > baseline 0.31. The
  biggest wins are the classes the baseline could not handle: `recommendation`
  (0.00 → 0.36) and `critique` (recovered to 0.36 after the label clean-up; an
  earlier run with noisy labels scored 0.00).
- **Absolute (macro-F1 ≥ 0.70, every class ≥ 0.60): NOT met.** macro-F1 0.45;
  `critique` and `recommendation` sit at 0.36. This is an honest ceiling for a
  subjective 3-class task with ~200 examples, a 37-example test set, and residual
  label subjectivity on the `critique`↔`reaction` boundary.

## Hypothesis check

The revised hypothesis (after the baseline refuted the first one) was that
fine-tuning's main job is to **recover the `critique` class** and stop the model
dumping everything into `reaction`. Confirmed: `critique` and `recommendation` both
moved off 0.00, while `reaction` stayed strong. The `critique`↔`reaction` boundary
remains the hardest, consistent with the taxonomy's predicted edge case.

## Training note

Fine-tuned for 8 epochs, lr 5e-5, batch 16, with `load_best_model_at_end` on
validation accuracy (best epoch ≈ 5, val acc 0.556, val loss 0.98). An earlier
3–5 epoch / 2e-5 run underfit (loss stuck near ln 3 ≈ 1.10, collapsed to the
majority class); a 15-epoch run overfit (train loss → 0.004, val loss → 2.34).
