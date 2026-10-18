# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1. Overall accuracy at least 70%

Overall classification accuracy on the held-out test set must be at least 0.70 (70%). 

**Why this target:** In a 3-class classification problem, random guessing yields ~ 33.3% accuracy. Since `r/badminton` post titles contain strong, distinct domain vocabulary (e.g., "racket", "string", "tension", "footwork", "backhand", "BWF"), a useful fine-tuned model should comfortably clear 70% overall accuracy.

---

## 2. Per-label performance must achieve an F1 score of at least 60%

Every individual label (`pro_tournaments`, `gear_equipment`, and `technique_training`) must achieve an F1 score of at least 0.60 (60%) on the held-out test set. 

**Why this target:** On `r/badminton`, gear and technique posts/questions are very common, while tournament threads fluctuate depending on the BWF calendar. Setting an F1 floor score of 0.60 across all classes ensures the model performs reliably across all three topics without neglecting minority classes.

---

## 3. Label counts should be balanced - no more than 50% (100 posts) or less than 15% (30 posts) for a single label 

No single label may represent more than 50% (100 posts) or less than 15% (30 posts) of the total 200 labeled posts in `labels.csv`. 

**Why this target:** Enforcing 15% (~ 30 posts minimum) guarantees sufficient training examples for `pro_tournaments` without allowing one high-volume category (like `gear_equipment` or `technique_training`) to dominate the training distribution.

---

## 4. Consistency for labels 

Agreement between independent annotators on the 30-sample reference dataset (`agreement.py`) must be at least 0.80 (80%). 

**Why this target:** The decision rule cleanly separates physical equipment choices (`gear_equipment`) from body movement and stroke mechanics (`technique_training`). Achieving >=80% agreement proves the taxonomy definitions and tie-breaker rules are concrete and reproducible.

---

## 5. Model confidence is at least 85%

For predictions made with a model confidence score of >= 0.85, prediction accuracy on those specific instances must be at least 0.85 (85%). 

**Why this target:** If the model is used for automated post tagging or flair assignment on `r/badminton`, high-confidence predictions must be highly accurate. A well-calibrated confidence threshold ensures high-certainty outputs are reliable.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
