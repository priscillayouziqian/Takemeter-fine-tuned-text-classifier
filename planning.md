# TakeMeter — Planning

A fine-tuned text classifier that evaluates discourse quality in **r/CDrama**, an
online community of Chinese-drama fans.

---

## 1. Community

r/CDrama is an English-language community of Chinese-drama fans who discuss
currently-airing and past shows. Its comments range from substantive critique of
acting, dubbing, and writing, to in-the-moment emotional reactions, to
recommendations that steer what others watch. Telling these apart matters to
regulars because the sub's value comes from reasoned critique and useful
recommendations rather than pure hype — a "good take" here gives a reason or helps
you decide what to watch, while a "bad take" is just noise.

**Why I chose it.** I watch C-dramas daily as my regular way to relax and have seen
a large number of shows, so the discussion topics and the actors being talked about
are familiar to me. That domain familiarity is not just convenient — it directly
improves annotation quality: as a community regular I can label borderline comments
the way an actual participant would read them, which is exactly the "grounded in
community norms" standard the task requires.

**Why it's a good fit for classification.** Reading a 40-comment sample confirmed the
discourse is genuinely varied in *kind*, not just topic: the same thread mixes
detailed production critique, pure thirst/emotional reactions, and watchlist
recommendations. All three categories appear frequently, so the labels capture a
real distinction the community itself makes — and no single label trivially
dominates, which keeps the classification task non-trivial.

## 2. Labels

We chose 3 mutually exclusive labels grounded in how this community actually talks.
(The classic NBA-style `analysis / hot_take / reaction` was rejected: confident
evidence-free "hot takes" are rare in a fan community, so that label could not
reach the required ≥20% share.)

#### 🔬 `critique` — a reasoned evaluation of the drama itself
**Definition:** Points to a **concrete, specific aspect** of a drama (acting,
dubbing, cinematography, plot, pacing, production, casting…) and makes an evaluative
judgment about it. A stated "why" strengthens it but is **not required** — naming a
concrete aspect plus a judgment is enough. The point is to judge the work itself,
not to steer others. (Contrast: *generic / global* praise with no concrete aspect —
"masterpiece", "getting interesting" — is `reaction`, not `critique`.)

- **Clear example 1:** *"What I like: visual feast and ensemble cast. What I don't
  like: the dubbing is out of sync by half a second, and there are weird rushed
  cuts — the characters' decisions feel like a summary rather than elaborate
  thought."*
- **Clear example 2:** *"Li Yitong has the range to do more serious roles… this
  seems a lighter drama without the heavy political scheming, plot seems rather
  obvious as well."*
- **Uncertain example (resolved → critique):** *"The cast were already famous when
  this was filmed… wonder if it's more of a low budget feel."* It reads partly as a
  curious question, but it offers a genuine, coherent reason (cast fame rules out
  the "before they were famous" explanation, so the look must come from budget).
  A real reason is present even though it is brief → **critique**.

#### 💗 `reaction` — an emotional, in-the-moment response
**Definition:** Expresses a feeling, thirst/gushing, joke, or immediate impression
with **no concrete reason** and **no intent to recommend**.

- **Clear example 1:** *"I'm still getting distracted by how beautiful Deng Wei is
  😭 he always mesmerises me."*
- **Clear example 2:** *"Why is Ding Yuxi playing a villain so attractive?"*
- **Uncertain example (resolved → reaction):** *"Don't the astronomers inform the
  royals beforehand? Why are they so shocked?… and why the hell she yap so much
  omggg 🙄🔫"* It contains plot questions, but the overall function is venting
  emotion with no production-level reasoning and no recommendation → **reaction**.

#### 📋 `recommendation` — steers what others watch
**Definition:** The comment's main purpose is to influence other people's viewing
choices — recommending, warning off, or sharing a watchlist as a suggestion.

- **Clear example 1:** *"Here's our officially unofficial sub starter package so
  that you might find some more interesting dramas."*
- **Clear example 2:** *"Just finished The Lead. Best series I've watched in years.
  Started Blossoms Shanghai…"* (a strong endorsement that steers others' choices).
- **Uncertain example (resolved → reaction):** *"Bai Jingting and his shoulders…
  Cheng Lei always bringing charm… Love in the Clouds is next on my watchlist."*
  Despite naming titles and a watchlist, it is almost entirely thirst about
  actors' looks; after stripping the appearance-gushing, little substantive
  steering remains → **reaction**.

### Mutual exclusivity — ordered decision procedure

A single comment can mix emotion, reasons, and recommendation. To keep labels
mutually exclusive, apply these steps in order and stop at the first "yes":

```
0. First strip out appearance/thirst gushing about actors — it always counts as
   emotion (reaction-flavor), never as a "reason." [contributed by annotation review]

1. Is the remaining content's main purpose to steer OTHER people's viewing
   (recommend / warn off / endorse so others watch)?             → recommendation
   (Personal intent only — "adding this to MY watchlist", "I'll watch this" — is
    NOT steering others; it falls through to step 3.)

2. Otherwise, does it point to a CONCRETE, specific aspect of the drama
   (cinematography, dubbing, an actor's specific performance, pacing of a
    section…) and make an evaluative judgment about it? (an explicit "why"
    strengthens it but is not required)                          → critique

3. Otherwise (only emotion / jokes / vague-or-global like-dislike with no
   concrete aspect)                                              → reaction
```

Key clarifications baked into the rule:
- *Generic / global* praise with **no concrete aspect** → `reaction`, not `critique`
  (e.g. *"masterpiece, peak cinema"* or *"story is getting interesting after ep 4-5"*).
- Naming a **concrete aspect** + a judgment IS enough for `critique`, even briefly
  (e.g. *"the cinematography, those palace shots are unreal"*).
- A judgment that evaluates the work → `critique`; one whose point is to make
  **others** watch → `recommendation`; personal viewing excitement → `reaction`.

## 3. Hard edge cases

The `critique` ↔ `reaction` boundary when an opinion is stated **without** a
reason. Handling: if no genuine, understandable reason about the drama is present,
it is `reaction` regardless of how strongly the opinion is worded. (Two more
documented boundary cases — `recommendation` vs `reaction`, and plot-question vs
`reaction` — appear as the "uncertain example" under each label above.)

**Real cases that gave me pause during annotation (Milestone 3):**

1. *"…these episodes have broken my heart 💔 just mourning for all these poor
   things. Poor stupid Zhou Tianyang and Qin Zheng…"* — `critique` vs `reaction`.
   It discusses plot at length, but the function is grief/emotion about what
   happened, not evaluation of a concrete aspect of the work. **Decided `reaction`.**
   (This one was first mislabeled `critique`; a consistency check against my own
   note caught it and it was corrected.)
2. *"Cracking up at Coconut Boy Zhou Tianyang acting like those manipulative
   concubines who want to bang their head against a pillar the moment their schemes
   get exposed, choosing injury over accountability."* — `reaction` vs `critique`.
   It starts as a laugh ("cracking up"), but the substance evaluates how the show
   handles a recurring trope. **Decided `critique`** (concrete: the writing/acting
   of a trope, with a judgment).
3. *"Adding this to my watchlist immediately, the trailer gave me chills 😍"*
   (boundary probe) — `recommendation` vs `reaction`. Personal viewing intent, not
   steering others. **Decided `reaction`.**

## 4. Data collection plan

**Where.** Comments are collected from r/CDrama via a browser-console script that
calls Reddit's public `.json` endpoints from a logged-in session. (The official API
path was blocked: app creation is gated for low-karma accounts, and unauthenticated
`.json` requests return HTTP 403.) A first 40-comment sample was already pulled to
validate the taxonomy before annotating.

**What is excluded.** The `hot` listing is dominated by a pinned monthly mega-thread
full of airing schedules, bot messages, and mod announcements — administrative
content that is not discourse. It is filtered out at collection time
(AutoModerator/bot authors, mod announcements, pure schedules, link-only posts), and
collection skips stickied mega-threads, pulling from real episode-discussion threads.

**How many per label (Plan B — natural distribution with a floor).** I will
over-collect ~250 comments and annotate down to 200+. I will *not* force an equal
split; the dataset is allowed to reflect the community's natural mix (where
`reaction` is the most common). The hard constraint is a **floor of ≥40 examples
(≥20%) for every label**, including the rarest (`critique` / `recommendation`), so
the model cannot succeed by always predicting the majority class.

**If a label is underrepresented after 200.** If any label falls below the 40-example
floor, I will do targeted top-up collection from threads where that label is dense:
`critique` → review / deep-discussion threads; `recommendation` → "what should I
watch / suggestions" threads; `reaction` → live episode-reaction threads. I keep
collecting that label only until it clears the floor.

**Split.** The annotated set is divided into train / validation / test, with the
class balance preserved across all three splits (stratified) so the test metrics
reflect performance on every label.

## 5. Evaluation metrics

Accuracy alone is misleading here. Under Plan B the classes are imbalanced
(`reaction` is the majority), so a lazy model that always predicts `reaction` could
score ~50% accuracy while getting **0%** on `critique` and `recommendation` — useless
but superficially "okay." Accuracy is dominated by the largest class. The metrics
below are chosen specifically to defeat that illusion.

- **Macro-averaged F1 — primary metric.** F1 (the harmonic mean of precision and
  recall) is computed per class, then averaged with **equal weight per class**
  regardless of class size. Because the rare classes count just as much as the
  majority, a model that ignores `critique`/`recommendation` is heavily penalized.
  This directly matches the imbalanced, every-label-matters nature of the task, so
  it is the headline number rather than accuracy.
- **Per-class precision / recall / F1.** Reported for all three labels. Precision
  asks "of the comments predicted X, how many really are X?" (over-reporting); recall
  asks "of the true X comments, how many were caught?" (under-reporting). These
  expose *which* label the model is weak on — expected to be the rarer/harder ones.
- **Confusion matrix.** Shows, for each true label, what the model predicted instead.
  It pinpoints *where* errors concentrate; the taxonomy predicts the
  `critique` ↔ `reaction` boundary will be the main source of confusion, and the
  matrix will confirm or refute that (feeding the Milestone-4 failure analysis).
- **Overall accuracy.** Still reported for both models (required, and a familiar
  reference point), but explicitly treated as secondary to macro-F1.

Both the fine-tuned model and the Groq zero-shot baseline are scored on the **same
test set** with this identical metric suite, so the comparison is apples-to-apples.

## 6. Definition of success

Success is defined on two levels, both objectively checkable on the held-out test set:

- **Relative (did fine-tuning actually help?).** The fine-tuned model's macro-F1
  must **beat the Groq `llama-3.3-70b-versatile` zero-shot baseline's macro-F1** on
  the same test set. If it doesn't, fine-tuning added no value.
- **Absolute (is it good enough to deploy?).** **macro-F1 ≥ 0.70** *and* **every
  per-class F1 ≥ 0.60**. The per-class floor matters for a real community tool: a
  model that averages well but scores, say, 0.2 on `critique` is blind to the most
  valuable kind of comment and isn't deployable, so no single label may be left
  behind.

This is a subjective task where even two human annotators disagree on borderline
cases, so 0.70 is a realistic, meaningful bar rather than a low one. A result
**above ~0.95 would be treated as a red flag** (likely train/test leakage or labels
that are too easy) and investigated, not celebrated.

## 7. AI Tool Plan

There is no implementation code to generate in this project, so AI tools are used at
the three points where they genuinely help a classification/annotation task.

### Label stress-testing — DONE (before annotating)

I asked an AI to generate 8 comments sitting on the label boundaries and tried to
classify each with the decision procedure. Six resolved cleanly; two exposed gaps in
the definitions, which I then tightened:

- *"Adding this to my watchlist immediately, the trailer gave me chills 😍"* — looked
  like `recommendation` but is only **personal** viewing intent, not steering others.
  → Tightened step 1: `recommendation` must steer **other** people; personal
  watchlist intent falls through to `reaction`.
- *"The cinematography 😍 those palace shots are unreal"* — names a **concrete** aspect
  with a judgment but states no full "why". Ruling: this still counts as `critique`.
  → Tightened step 2: pointing to a **concrete, specific** aspect + a judgment is
  enough; an explicit "why" is not required. The discriminator vs `reaction` is
  **concrete vs generic** ("cinematography/palace shots" = concrete → `critique`;
  "masterpiece" / "story getting interesting" = generic → `reaction`). This is
  consistent with the earlier edge cases (e.g. row-21 stays `reaction`).

### Annotation assistance — decision: hand-label all 200 myself

I will **not** use an LLM to pre-label. As a community regular I am the domain
expert, hand-labeling all ~200 examples forces me to internalize the boundaries, and
it keeps the dataset cleanest with the simplest disclosure story (no pre-labeled
subset to track or risk of anchoring on an LLM's mistakes).

### Failure analysis — planned for Milestone 4

After evaluation I will give the AI the full list of **wrong predictions** and ask it
to identify a **systematic** error pattern (e.g. "short comments are misclassified",
"sarcasm is read as `critique`", "the `critique` ↔ `reaction` boundary dominates the
errors"). I will then **verify every proposed pattern by hand** against the actual
comments before writing it up, since an LLM can hallucinate a pattern that isn't
really there.

