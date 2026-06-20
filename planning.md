# TakeMeter — Planning

A fine-tuned text classifier that evaluates discourse quality in **r/CDrama**, an
online community of Chinese-drama fans.

---

## Milestone 1 — Community & Label Taxonomy

### Community

r/CDrama is an English-language community of Chinese-drama fans who discuss
currently-airing and past shows. Its comments range from substantive critique of
acting, dubbing, and writing, to in-the-moment emotional reactions, to
recommendations that steer what others watch. Telling these apart matters to
regulars because the sub's value comes from reasoned critique and useful
recommendations rather than pure hype — a "good take" here gives a reason or helps
you decide what to watch, while a "bad take" is just noise.

### Label taxonomy (3 labels)

We chose 3 mutually exclusive labels grounded in how this community actually talks.
(The classic NBA-style `analysis / hot_take / reaction` was rejected: confident
evidence-free "hot takes" are rare in a fan community, so that label could not
reach the required ≥20% share.)

#### 🔬 `critique` — a reasoned evaluation of the drama itself
**Definition:** Evaluates a *specific aspect* of a drama (acting, dubbing, plot,
pacing, production, casting…) **and gives at least one genuine, understandable
reason**. The point is to judge the work itself, not to steer others.

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

1. Is the remaining content's main purpose to steer others' viewing
   (recommend / warn off / share a watchlist as a suggestion)?   → recommendation

2. Otherwise, does it give a genuine, understandable reason about a specific
   aspect of the drama? (length is a rough guide, ~20 words, NOT a hard cutoff)
                                                                  → critique

3. Otherwise (only emotion / jokes / vague like-dislike with no reason)
                                                                  → reaction
```

Key clarifications baked into the rule:
- Vague "good / interesting" with **no stated reason** → `reaction`, not `critique`
  (e.g. *"I'm liking it so far, story is getting interesting after ep 4-5"*).
- A **reason that evaluates the work** → `critique`; a reason whose point is to
  **make you watch** → `recommendation`.
- Reason quality matters more than word count (a short but coherent reason still
  counts as `critique`).

### Hardest anticipated edge case

The `critique` ↔ `reaction` boundary when an opinion is stated **without** a
reason. Handling: if no genuine, understandable reason about the drama is present,
it is `reaction` regardless of how strongly the opinion is worded.

### Data collection & filtering note

- Data is collected from r/CDrama via a browser-console script that calls Reddit's
  public `.json` endpoints from a logged-in session (the official API path was
  blocked: app creation is gated for low-karma accounts, and unauthenticated
  `.json` requests return HTTP 403).
- A first 40-comment sample was pulled to validate the taxonomy before annotating.
- The `hot` listing is dominated by a pinned monthly mega-thread full of airing
  schedules, bot messages, and mod announcements — **administrative content that is
  not discourse and will be excluded at collection time** (filter out
  AutoModerator/bot authors, mod announcements, pure schedules, link-only posts).
- For the full dataset (~250 comments, to be trimmed to 200+), collection will
  skip stickied mega-threads and pull from real episode-discussion threads.

