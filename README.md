# TakeMeter

---

# Unit 5

## What This Does

TakeMeter looks at posts in the `r/badminton` public community forum. Each post is sorted into one of three labels (`pro_tournaments`, `gear_equipment`, `technique_training`). Across the `r/badminton` community forum, users discuss a range of topics from Badminton World Federation (BWF) professional players/tournaments to reviews on badminton equipment and advice on badminton techniques. Based on the given text, TakeMeter classifies and sort posts by labels for the badminton community. 

---

## Label Taxonomy

### `pro_tournaments`

**Definition:** Posts discussing professional BWF players, international tournament results, match threads, or player news. 

**Example 1:** 
> If An Se-young played WD with Kim Ga-eun, what do you think would be their world ranking after a year?

**Example 2:**
> 2019 Kento Momota vs 2021 Lee Zii Jia

### `gear_equipment`

**Definition:** Posts seeking or providing advice, reviews, or specifications regarding badminton equipment, including racquets/rackets, shoes, or shuttlecocks. 

**Example 1:**
> Equipment guide for Doubles players (1) The Racket

**Example 2:**
> Victor Auraspeed 99 Review: no longer a 'speed' racket

### `technique_training`

**Definition:** Posts asking for or sharing guidance on stroke mechanics, footwork, tactics, physical conditioning, or court positioning. 

**Example 1:**
> How do I deal with high serves and clears?

**Example 2:**
> Not a beginner but came back to badminton after break of 4-5 years and want to know the right technique

### The hardest boundary

**Which two labels:** `gear_equipment` and `technique_training`

**The decision rule I used every time:** If a post's primary focus is selecting, comparing, reviewing, or modifying physical equipment (racquets/rackets, shoes, or shuttlecocks), classify it as `gear_equipment`. If the primary focus is on body movement, stroke execution, footwork, tactics, or drills, classify it as `technique_training`, even if specific equipment or racquets/rackets are mentioned in passing.

---

## The Dataset

**Where the posts came from:** Collected directly from the public post feed on `r/badminton` from Reddit 

**How I labelled them:** Labeled 20 posts cold by hand to establish a baseline (`cold`), labeled the remaining 180 posts using Gemini LLM Notebook including definitions for the three labels, decision rule + the source: `r/badminton` (`ai_prelabeled`), then read and correct any issues 

**Counts per label:** 

| Label | Count | Share |
|---|---|---|
| `gear_equipment` | 81 | 40.5% |
| `technique_training` | 75 | 37.5% |
| `pro_tournaments` | 44 | 22.0% |
| **Total** | 200 | 100% |

**Three hard cases**

**1.**
> *The post: How to hold racket handle loosely until shuttle contact point*
>
> *Could have been: `gear_equipment` vs `technique_training`*
>
> *I chose `technique_training` because although the post title mentions "racket handle," the user is actually asking about grip relaxation, hand control, and stroke timing rather than physical equipment.*

**2.**
> *The post: Sanding the frame to make it lighter*
>
> *Could have been: `gear_equipment` vs `technique_training`*
>
> *I chose `gear_equipment` because while "sanding" is an action, it describes a DIY physical modification to alter the badminton racket specifications rather than an on-court playing stroke or movement.*

**3.**
> *The post: What's the correct etiquette regarding breaking racket?*
>
> *Could have been: `gear_equipment` vs `technique_training`*
>
> *I chose `gear_equipment` because breaking a racket involves physical equipment damage rather than intentional badminton stroke or training technique.* 

---

## The Training Run

**Base model:** distilbert-base-uncased

**Settings:** seed 42, 3 epochs, lr 2e-05, batch 16

**Anything I changed from the defaults, and why:** N/A

**Split sizes:** 70% train (139), 15% validation (31), 15% test (30)

- `pro_tournaments` has only 7 examples in the test split. 

`results.json`: 
> device: cpu  (CPU)  
> torch:  2.14.1+cpu

---

## How I Used AI

**Moment 1** 

- *What I asked for:* Brainstorming session for the criterion into Claude - "Here are five acceptance criteria with targets, for a text classifier I have not trained yet. For each target, give me one reason it might turn out too easy and one reason it might turn out too hard. Don't tell me which is more likely and don't suggest different numbers..."
- *What came back:* Feedback on whether it would be too easy or too hard; Better understand what my criterion defines
- *What I changed:* Most criterion were fine, just took into consideration whether I needed to be more specific or not. 

**Moment 2**

- *What I asked for:* In Milestone 5, Section 2 showed two parts with [FIX ME] lines. This section checks my data and prints anything that would break the run or quietly ruin it. As it showed that I needed to look over my `labels.csv` data, I asked Claude to debug any issues within my data that it can spot.
- *What came back:* 4 lines in my `labels.csv` data had an extra comma (,) in the text
- *What I changed:* I added quotations (" ") to those 4 lines surrounding the text, so it wouldn't mess up the labels and note within the `labels.csv` data. This is necessary to fix before splitting the data. 

**Pre-labelling disclosure:** I labeled 20 posts manually and had AI (Gemini LLM Notebook) help extract the text from the posts and label the rest following the parameters. Used AI (Gemini LLM Notebook) to extract text and label another 26 posts to fix my thinnest label issue. 

---
# Unit 6

## Baseline vs. Trained

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.867 | 0.733 | -0.133 |
| Macro F1 | 0.859 | 0.711 | -0.148 |
| F1 — `gear_equipment` | 0.923 | 0.733 | -0.190 |
| F1 — `pro_tournaments` | 0.875 | 0.600 | -0.275 |
| F1 — `technique_training` | 0.778 | 0.800 | +0.022 |

**What I predicted before I looked:** My fine-tuned model would outperform the zero-shot baseline model, by 5-10% in overall accuracy: For `pro_tournaments`, I would expect it to score relatively high on both models because BWF players and official event titles are distinct proper nouns that LLMs can easily identify even without fine-tuning. As for `gear_equipment` and `technique_training`, fine-tuning makes a big difference as the zero-shot baseline model will likely get confused by boundary posts mentioning both equipment and technique training.

**What the gap actually means:** My initial prediction was incorrect as the zero-shot baseline model significantly outperformed the fine-tuned model overall (86.7% vs 73.3%), beating it across two out of three labels. This happened because a zero-shot baseline model possesses extensive world knowledge about badminton and when paired with clear label definitions, it categorized posts very effectively. My fine-tuned model is likely overfit given the small dataset of training examples. However, fine-tuning did slightly improve performance on `technique_training`, showing that training data did help the model learn nuance in badminton technique training posts. 

Without the label definitions (bare label names), the zero-shot baseline model doesn't perform as well overall (`baseline_bare_names.json`). The reported baseline used is with the label definitions. 

---

## Run Log — Before

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy at least 70% | ≥ 0.70 | 0.733 | 0.667 | 0.833 | MISSED | 
| 2. Per-label performance must achieve an F1 score of at least 60% (`pro_tournaments`) | ≥ 0.60 | 0.600 | 0.000 | 0.727 | MISSED |
| 3. Label counts should be balanced | 15%-50% | 22.0%-40.5% | 22.0%-40.5% | 22.0%-40.5% | MET |
| 4. Consistency for labels | ≥ 0.80 | 0.933 | 0.933 | 0.933 | MET | 
| 5. Model confidence is at least 85% | ≥ 0.85 | 0.900 | 1.000 | 1.000 | MET |

### Metrics Spread Across Seeds — Before

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.733 | 0.667 | 0.833 | 0.167 |
| Macro F1 | 0.711 | 0.508 | 0.817 | 0.309 |
| F1 — `pro_tournaments` | 0.600 | 0.000 | 0.727 | 0.727 |
| F1 — `gear_equipment` | 0.733 | 0.690 | 0.815 | 0.125 |
| F1 — `technique_training` | 0.800 | 0.833 | 0.909 | 0.109 |

### Confusion matrix

| true \ predicted | pro_tournaments | gear_equipment | technique_training |
|---|---|---|---|
| **pro_tournaments** | 3 | 4 | 0 |
| **gear_equipment** | 0 | 11 | 1 |
| **technique_training** | 0 | 3 | 8 |

**My biggest off-diagonal number, and what it means:** 4 real `pro_tournaments` posts were called `gear_equipment` and 0 real `gear_equipment` posts were called `pro_tournaments`. This one-sided confusion means the model developed a strong bias toward `gear_equipment`. A major contributing factor is likely the class imbalance in training data, with `gear_equipment` being the largest class (81 posts), nearly double the size of `pro_tournaments` (44 posts). As a result, when the model encounters ambiguous vocabulary or brand names mentioned in a `pro_tournament` context, it would more frequently default to the `gear_equipment` label. 

---

## Verdicts and Diagnoses

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy at least 70% | ≥ 0.70 | MISSED | Seeds 42 (73%) and 2024 (83%) met the threshold, but Seed 7 achieved an overall accuracy of 67%. The target must hold at least 70% overall accuracy across all trials, so this counts as a miss. |
| 2 | Per-label performance must achieve an F1 score of at least 60% (`pro_tournaments`) | ≥ 0.60 | MISSED | Every label must achieve an F1 score of at least 60% across all trials. The other two labels `gear_equipment` and `technique_training` achieved an F1 score of at least 60% across all trials. However, `pro_tournaments` did not across all trials. While Seeds 42 and 2024 met the target, on Seed 7 `pro_tournaments` dropped to an F1 of 0.000. |
| 3 | Label counts should be balanced | 15%-50% | MET | The smallest label count was 22.0% and the largest label count was 40.5%, remaining within the 15%-50% boundary. |
| 4 | Consistency for labels | ≥ 0.80 | MET | Achieved an agreement rate of 93% (28/30) against the staff benchmark set. |
| 5 | Model confidence is at least 85% | ≥ 0.85 | MET | High-confidence predictions scored 0.900 (Seed 42), 1.000 (Seed 7), and 1.000 (Seed 2024), beating the 0.85 target on all trials. |

**Diagnoses**
- Criterion 1: Overall accuracy at least 70% 
This miss was driven by dataset instability across data splits rather than a lack of model capacity, as shown by accuracy moving 0.167 across seeds, from 66.7% on Seed 7 up to 83.3% on Seed 2024. As this swing is well over 10 points, it indicates that the dataset is too small or lopsided to produce a stable evaluation, even though Seed 2024 proves it can reach the target. As shown by my agreement report under Pile (a), I tended to label based on surface-level cues rather than evaluating load-bearing intent. This introduced boundary noise across the dataset, causing overall accuracy to dip below the threshold whenever ambiguous, boundary-heavy posts were concentrated into Seed 7's test fold.

- Criterion 2: Per-label performance must achieve an F1 score of at least 60% (`pro_tournaments`)
This miss was likely caused by severe class sparsity combined with an inconsistent decision boundary against `gear_equipment`. `pro_tournaments` had the smallest label count at 22.0% (44 posts) roughly half the size of `gear_equipment`, leaving only 7 examples in the test split. The confusion matrix confirmed this, showing that missed tournament posts were overwhelmingly misclassified as `gear_equipment`. This likely mirrors the surface-level cue identified in my agreement report under Pile (a), where posts discussing tournament matches that mentioned racket brands were reflexively labeled as `gear_equipment`. With only 7 test examples, missing these boundary posts gave the model zero margin for error, causing Seed 7's F1 score to drop to 0.000 (whereas Seed 2024 reached 0.727 on a cleaner split).

---

## Agreement Report

**Agreement rate:** 28 / 30 = 93%

**Disagreements**

**1.**
> *The post: Three of us ran the same session on the same day, different shoes, within 10 seconds of each other over 5 miles. I'm not saying shoes do nothing, but whatever they do was smaller than the noise between three people.*
>
> *Staff said / I said: Staff said `analysis`, I said `hot_take`.*
>
> *My call, and why: The post sits right on the boundary because it contains checkable metrics (3 runners, 10 seconds, 5 miles); however, it comes from an informal run between three friends. Staff credited the observation as tiny, checkable sample. Under Decision Rule #1, a fact must be load-bearing - a casual group run lacking scientific controls or statistical validity cannot dismiss the impact of the type of shoes. Without true experimental rigor, this is essentially an unsupported opinion (`hot_take`) with checkable metrics included.*
>
> *Which pile: (c) The rule is ambiguous here and my reading is defensible.* 

**2.**
> *The post: Times are down about 3% since the shoe rules changed, so obviously the shoes are the whole story and the sport has a problem.*
>
> *Staff said / I said: Staff said `hot_take`, I said `analysis`.*
>
> *My call, and why: The post is ambiguous at first glance because it mentions a specific quantitative metric (3% drop) and a historical event (shoe rules changed), which led me to label this post as `analysis`. Based on Decision Rule #1, a fact must be load-bearing rather than a decorative flourish. The post leaps from an approximate 3% drop straight to an unsupported claim ("so obviously the shoes are the whole story"). Staff is right, and my call was a consistency slip by labeling based on the presence of a number instead of checking whether the number was actually carried the argument.*
>
> *Which pile: (a) The rule covered it and I applied it loosely - a consistency problem.*

**What the pattern in my disagreements tells me:** My disagreements reveal a pattern of reacting to surface-level cues rather than evaluating load-bearing intent—such as assuming the mere presence of a number automatically makes a post `analysis`. When boundaries felt ambiguous, I tended to apply an overly strict evidence threshold. This likely carried over to my badminton dataset, where posts mentioning gear brands or racket specifications may have been reflexively categorized as `gear_equipment`, even when the primary subject was tournaments or technique. 

---

## The Improvement

**What I changed:** I added 26 new, clean posts to the thinnest class (`pro_tournaments`) in `labels.csv`, increasing its total count from 44 posts (22.0%) to 70 posts (31.0%). All added posts focus specifically on BWF World Tour events, tournament brackets, Olympic qualification, match results, and player rankings, without introducing ambiguous equipment keywords. Using Gemini LLM Notebook to extract 26 new, clean posts from `r/badminton` focusing on posts within `pro_tournaments` and reading through each one to confirm it clearly fits the label.

**Which diagnosis pointed at it:** My diagnosis for Criterion 2 identified that having only 44 total posts, left just 7 examples in the 15% held-out test split, giving the model zero margin for error when random seed splits sampled ambiguous posts. Increasing the sample size of `pro_tournaments` stabilizes test evaluation and prevents minority-class metric collapse. Furthermore, Criterion 1 exhibited severe evaluation instability (a 0.167 spread across seeds) because the dataset was too small and lopsided to produce a consistent measure.

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy at least 70% | ≥ 0.70 | 0.882 | 0.912 | 0.912 | MET | 
| 2. Per-label performance must achieve an F1 score of at least 60% (`pro_tournaments`) | ≥ 0.60 | 0.900 | 0.909 | 0.909 | MET |
| 3. Label counts should be balanced | 15%-50% | 31.0%-35.8% | 31.0%-35.8% | 31.0%-35.8% | MET |
| 4. Consistency for labels | ≥ 0.80 | 0.933 | 0.933 | 0.933 | MET | 
| 5. Model confidence is at least 85% | ≥ 0.85 | 0.909 | 1.000 | 1.000 | MET |

### Metrics Spread Across Seeds — After

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.882 | 0.912 | 0.912 | 0.029 |
| Macro F1 | 0.885 | 0.912 | 0.912 | 0.027 |
| F1 — `pro_tournaments` | 0.900 | 0.909 | 0.909 | 0.009 |
| F1 — `gear_equipment` | 0.846 | 0.909 | 0.870 | 0.063 |
| F1 — `technique_training` | 0.909 | 0.917 | 0.957 | 0.047 |

**Did it help, and how do I know:** Yes, adding more examples to my thinnest label (`pro_tournaments`) helped, stabilizing the dataset and allowing both Criterion 1 and Criterion 2 verdicts to be MET across all three seeds. The accuracy spread across seeds dropped to just 0.029 (well under the 10 point threshold) while overall accuracy increased to a consistent 88–91%, demonstrating that the dataset is now stable enough to produce a consistent, reliable measure (Criterion 1 MET). This also eliminated the minority-class collapse on Criterion 2: on Seed 7, the F1 score for `pro_tournaments` surged from 0.000 to 0.909, with an F1 spread of only 0.009 across all three runs (0.900, 0.909, 0.909). By increasing `pro_tournaments` from 44 to 70 posts, the 15% held-out test split grew from 7 to 11 examples, removing the zero margin for error sensitivity that previously allowed a couple of ambiguous boundary posts to zero out performance and ensuring that every label easily cleared the 0.60 F1 floor (with the lowest per-label F1 across all seeds and classes being 0.846).

---

## What's Still Broken

All five criteria were MET after adding more examples to my thinnest label (`pro_tournaments`), so nothing is failing. One weakness is in the criterion itself: setting a 15% minimum (30 posts) was too permissive for a 200-example dataset, as it allowed `pro_tournaments` to pass even while its label counts were small (7 examples in the test split) and caused severe evaluation instability. A stricter criterion requiring between 25%-45% representation and no fewer than 8 examples is necessary to prevent minority-class metric collapse.

**The gap between what I meant my labels to capture and what the model
learned:** While I expected `gear_equipment` and `technique_training` to be my hardest boundary, the confusion matrix shows they separated cleanly (only 1 mutual error in each direction), with the primary lingering error of 2 real `pro_tournaments` posts misclassified as `gear_equipment` (and 0 in reverse). This reveals the model still relies on surface lexical cues rather than underlying intent, reflexively classifying tournament posts as gear equipment instead. 

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**