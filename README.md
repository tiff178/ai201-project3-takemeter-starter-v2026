# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

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

**Pre-labelling disclosure:** I labeled 20 posts manually and had AI (Gemini LLM Notebook) help extract the text from the posts and label the rest following the parameters. 

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
| 3. Label counts should be balanced | 15% - 50% | 22.0% - 40.5% | 22.0% - 40.5% | 22.0% - 40.5% | MET |
| 4. Consistency for labels  | ≥ 0.80 |  |  |  | [Pending Milestone 3] | 
| 5. Model confidence is at least 85% | ≥ 0.85 | 0.900 | 1.000 | 1.000 | MET |

### Metrics Spread Across Seeds 

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

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**