# TakeMeter — A Discourse-Quality Classifier for r/CDrama

📺 **Video walkthrough:** https://www.youtube.com/watch?v=znXaaotyslQ

TakeMeter is a fine-tuned text classifier that sorts comments from the
[r/CDrama](https://www.reddit.com/r/CDrama/) subreddit (fans of Chinese dramas) into
three discourse types. It was built by defining a label taxonomy, hand-collecting and
annotating 242 real comments, fine-tuning `distilbert-base-uncased`, and comparing it
against a zero-shot Groq `llama-3.3-70b-versatile` baseline on a held-out test set.

> Design notes, edge-case rules, and the full annotation reasoning live in
> [planning.md](planning.md). This README is the standalone final report.

## Labels

| label | one-line definition |
|---|---|
| `critique` | Names a **concrete** aspect of a drama (acting, dubbing, cinematography, plot, pacing, writing, casting) **and judges it**. A "why" helps but isn't required. Generic praise ("masterpiece") does **not** count. |
| `reaction` | An emotional, in-the-moment response — feelings, gushing, jokes, plot venting, vague/global like-dislike — with no concrete aspect and no intent to recommend. |
| `recommendation` | Mainly steers **other people's** viewing — recommend, warn off, or endorse so others watch. Personal "adding to my watchlist" intent does not count. |

**Two examples per label:**
- `critique`: *"The dubbing is out of sync by half a second and the rushed cuts make
  the characters' decisions feel like a summary."* · *"Li Yitong has the range to do
  more serious roles; this seems a lighter drama and the plot is rather obvious."*
- `reaction`: *"I'm still getting distracted by how beautiful Deng Wei is 😭."* ·
  *"Why is Ding Yuxi playing a villain so attractive?"*
- `recommendation`: *"Best series I've watched in years — go watch this one."* ·
  *"Here's our unofficial sub starter package so you find more dramas to watch."*

The boundary is resolved with an ordered decision rule (strip actor-thirst → steer
others? → concrete aspect + judgment? → else reaction). Full rule and edge cases in
[planning.md](planning.md).

**Why this community:** I watch C-dramas daily and know the shows and actors being
discussed, so I can label borderline comments the way an actual participant reads
them. r/CDrama is also a good classification target because the same thread genuinely
mixes all three discourse types, and no single type trivially dominates.

## Data

- **Source:** public comments scraped from r/CDrama via the site's `.json` endpoints
  from a logged-in browser session (the official API path was blocked for low-karma
  accounts). Administrative content (bot/mod posts, airing schedules, link-only
  posts) and stickied mega-threads were filtered out.
- **Labeling:** all 242 examples were **hand-labeled** by me, a daily r/CDrama
  reader, applying the decision rule. No LLM pre-labeling was used.
- **Size & distribution** (`labeled_data.csv`, single un-split file; the notebook
  does the 70/15/15 stratified split):

  | label | count | share |
  |---|---|---|
  | reaction | 111 | 45.9% |
  | critique | 82 | 33.9% |
  | recommendation | 49 | 20.2% |

  No class exceeds 70%; every class clears a 40-example floor. `recommendation` was
  sparse in discussion threads and required a targeted top-up from "what to watch"
  threads.

- **Hard annotation cases** (3 of several, full list in planning.md):
  1. *"…these episodes have broken my heart 💔 just mourning for all these poor
     things…"* — discusses plot but the function is grief, not evaluation →
     **reaction**. (Originally mislabeled `critique`; a check of my own notes caught
     it.)
  2. *"Cracking up at Coconut Boy … acting like those manipulative concubines …
     choosing injury over accountability."* — opens as a joke but evaluates how the
     show handles a trope → **critique**.
  3. *"Adding this to my watchlist immediately, the trailer gave me chills 😍"* —
     personal intent, not steering others → **reaction**.

## Model & Training

- **Base model:** `distilbert-base-uncased` (HuggingFace).
- **Approach:** sequence classification head fine-tuned on the 70% train split,
  validated on 15%, evaluated on a locked 15% test set (37 examples).
- **Key hyperparameter decision:** the default 3 epochs / `2e-5` underfit badly on
  ~169 training examples — loss stayed near `ln 3 ≈ 1.10` and the model collapsed to
  always predicting the majority class. I raised **epochs to 8** and the **learning
  rate to 5e-5** to give the model enough gradient steps to learn, and enabled
  `load_best_model_at_end` on validation accuracy so the exported model is the best
  epoch (≈ epoch 5, val acc 0.556) rather than an over-fit final epoch. (An
  intermediate 15-epoch run confirmed the overfitting risk: train loss → 0.004 while
  val loss ballooned to 2.34.)

## Evaluation Report

Both models scored on the **same 37-example test set**.

### Baseline description

The baseline is **Groq `llama-3.3-70b-versatile`, zero-shot** (no task-specific
training). It received a system prompt containing the same three label definitions
and the ordered decision rule from this README, plus one example per label, and was
instructed to **output only the single lowercase label word**. Each of the 37 test
comments was sent as a separate user message; the notebook parsed the one-word reply
and matched it against the label strings. **0% of responses were unparseable**, so no
prompt revision was needed. This establishes how hard the task is for a strong
general model with no training.

### Overall

| metric | baseline (Groq zero-shot) | fine-tuned (DistilBERT) |
|---|---|---|
| accuracy | 0.459 | **0.514** |
| macro-F1 | 0.31 | **0.45** |

### Per-class (precision / recall / F1)

| label | baseline | fine-tuned | support |
|---|---|---|---|
| critique | 0.50 / 0.23 / 0.32 | 0.44 / 0.31 / 0.36 | 13 |
| reaction | 0.48 / 0.82 / 0.61 | 0.54 / 0.76 / 0.63 | 17 |
| recommendation | 0.00 / 0.00 / 0.00 | 0.50 / 0.29 / 0.36 | 7 |

The fine-tuned model beats the baseline on macro-F1 by **+0.14**, and its biggest
wins are exactly the classes the baseline failed on: `recommendation` (0.00 → 0.36)
and `critique` (the baseline barely recalls it; an earlier noisy-label run scored
0.00 here too).

### Confusion matrix — fine-tuned model (rows = true, cols = predicted)

| true ↓ / pred → | critique | reaction | recommendation |
|---|---|---|---|
| **critique** | 4 | 7 | 2 |
| **reaction** | 4 | 13 | 0 |
| **recommendation** | 1 | 4 | 2 |

(Supplementary image: `confusion_matrix.png`.)

The matrix shows one dominant, **directional** failure: `critique` → `reaction`
(7 of 13 true critiques). Combined with `reaction` → `critique` (4), the
`critique` ↔ `reaction` boundary accounts for **11 of 18 errors**. `recommendation`
is under-recalled, leaking mostly into `reaction` (4 of 7).

### Three analyzed failures

1. **`critique` → `reaction`** — *"Cultural explanation: … The bell symbolizes
   death…"* (pred `reaction`, conf 0.87). This is genuine analysis, but it reads as
   calm lore/plot chatter with no evaluative verb ("the acting is…", "the pacing
   drags"). The model learned that an emotional/plot-narration *tone* signals
   `reaction`, so analytical content delivered in a conversational tone is missed.
   **Boundary, not labeling, problem** — the label is correct; the model hasn't
   learned that critique can be tonally flat.
2. **`reaction` → `critique`** — *"My, oh my Dowager Consort Qin. 10 years of lies
   unravelling… one must never trust powerful people…"* (pred `critique`). Long,
   structured, argument-shaped plot reasoning. The *structure* signals critique even
   though the content is emotional plot reaction. The model overfits to "looks
   reasoned = critique." **Boundary problem.**
3. **`recommendation` → `reaction`** — *"Started Story of Yanxi Palace… thoroughly
   enjoying the harem antics… rare for a costume drama to grab me in 45 mins"*
   (pred `reaction`, conf 0.73). The recommendation here is *implicit* (enthusiasm
   that functions as an endorsement). With only ~34 training recommendations, the
   model only learned explicit "you should watch X" phrasing and reads implicit
   endorsement as enthusiasm. **Data problem** — needs more implicit-recommendation
   examples.

### Sample Classifications (fine-tuned model)

| comment (truncated) | predicted | confidence | correct? |
|---|---|---|---|
| "These episodes have broken my heart 💔 just mourning for all these poor characters." | reaction | ~0.85 | ✅ |
| "The kiss is soooo underwhelming 😂 the angles are so wrong too… director needs to brush up the skills." | critique | 0.59 | ✅ |
| "Cultural explanation: … the bell symbolizes death…" | reaction | 0.87 | ❌ (true critique) |
| "Started Story of Yanxi Palace… thoroughly enjoying the harem antics…" | reaction | 0.73 | ❌ (true recommendation) |

The first row is a **reasonable correct** prediction: the comment is pure emotional
mourning with no concrete craft aspect, which is exactly the `reaction` definition —
and the model is confident (~0.85), as it should be on a clear case.

## Reflection — what the model captured vs. what I intended

I intended `critique` to mean *"evaluates a concrete craft aspect."* What the model
actually learned is closer to *"is written in an analytical-sounding tone"* — it keys
on **surface form (structure, length, argument-shape) rather than on whether a
concrete aspect is named.** That is why flat-toned real critique leaks into
`reaction`, and why long emotional plot-musings leak into `critique`. The model also
captured `reaction` well (the majority, emotionally-marked class) but treats it as a
default bucket, dumping anything ambiguous there. For `recommendation`, it learned
only the *explicit* form and missed implicit endorsement. In short: the model learned
a **tone/structure boundary**, while my labels encode a **function/content boundary** —
and the gap between those two is precisely the `critique` ↔ `reaction` confusion. The
fix is not more epochs but more (and more varied) examples that decouple tone from
function: flat-toned critiques and reasoned-sounding emotional reactions.

## Spec Reflection

- **How the spec helped:** committing to **macro-F1 as the primary metric** in
  planning.md (before training) kept me honest. Accuracy alone (0.51) looks like a
  pass, but macro-F1 (0.45) and the per-class floor exposed that two of three classes
  are weak — which is the real story.
- **Where I diverged:** I set an absolute success bar of macro-F1 ≥ 0.70 with every
  class ≥ 0.60. I did **not** hit it (0.45; critique/recommendation at 0.36). I kept
  the bar rather than lowering it after the fact, and report the shortfall honestly:
  on a subjective 3-class task with ~200 examples and a 37-example test set, the
  meaningful result is the **relative** win over the baseline plus a diagnosable
  failure mode, not a single high score.

## AI Usage

1. **Label stress-testing (design):** I directed Claude to generate boundary-case
   comments and classify them with my decision rule. It surfaced two gaps; I
   **overrode** its instinct on one ("cinematography 😍 those palace shots are
   unreal") — I ruled that naming a *concrete* aspect counts as `critique` even
   without a stated "why", and tightened the rule accordingly.
2. **Failure-pattern analysis (evaluation):** I gave Claude my misclassified
   examples and asked for common themes. It proposed the directional
   `critique` ↔ `reaction` confusion and the "analytical tone vs. concrete content"
   split; I **verified each by re-reading the comments and the confusion matrix**
   before including them, and discarded loose claims that the test set was too small
   to support.
3. **Annotation disclosure:** no LLM pre-labeling was used — all 242 labels are mine
   by hand. Claude also helped debug the training run (diagnosing under- then
   over-fitting from the loss curves) and scripted the data merge/cleanup; I made
   every labeling and hyperparameter decision.

