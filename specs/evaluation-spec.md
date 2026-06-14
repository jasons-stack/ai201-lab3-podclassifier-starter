# Evaluation Spec — Pod Classifier

Complete this spec **before** writing any code for Milestone 3.

Use Plan or Ask mode to think through each blank field. When you're done,
your answers here become the blueprint for `compute_accuracy()` and
`compute_per_class_accuracy()` in `evaluate.py`.

---

## Background: What is evaluation?

After building a classifier, we need to know how well it works. Evaluation answers:
- **Overall:** What fraction of episodes did we classify correctly?
- **Per-class:** Are we better at some labels than others?

Both functions take the same inputs: a list of predicted labels and a list of
ground-truth labels, in the same order.

---

## compute_accuracy(predictions, ground_truth)

### What it does
Returns the fraction of predictions that exactly match the ground truth.

### Inputs

| Parameter | Type | Description |
|---|---|---|
| `predictions` | `list[str]` | Labels predicted by `classify_episode()`, one per episode. |
| `ground_truth` | `list[str]` | The correct labels, in the same order as `predictions`. |

### Output

| Return value | Type | Description |
|---|---|---|
| accuracy | `float` | A value between 0.0 and 1.0. |

---

### Spec fields — fill these in before writing code

**Formula:**

```
[blank — write out the accuracy formula in plain English.
 What counts as "correct"? What do you divide by?]

Count the number of positions where predictions[i] == ground_truth[i], 
then divide by the total number of episodes.
```

---

**Step-by-step logic:**

```
[blank — describe the steps your code will take.
 1. ...
 2. ...
 3. ...]

1. If both lists are empty, return 0.0
2. Count how many predictions exactly match ground_truth at the same index
3. Divide the count by the total number of episodes
4. Return the result as a float
```

---

**Edge case — what if both lists are empty?**

```
[blank — what should the function return? Why?]

Return 0.0 — there are no predictions to evaluate, so accuracy is undefined.
Returning 0.0 is safer than raising an error since the evaluation loop expects a float.
```

---

**Worked example:**

```
predictions  = ["interview", "solo", "panel", "interview"]
ground_truth = ["interview", "solo", "solo",  "narrative"]

[blank — what does compute_accuracy() return for these inputs? Show your work.]

Correct: position 0 (interview==interview), position 1 (solo==solo) = 2 correct
Total: 4
Accuracy: 2/4 = 0.5
```

---

## compute_per_class_accuracy(predictions, ground_truth)

### What it does
Returns accuracy broken down by each label. For each label in `VALID_LABELS`,
reports how many episodes with that ground-truth label were classified correctly.

### Inputs

| Parameter | Type | Description |
|---|---|---|
| `predictions` | `list[str]` | Labels predicted by `classify_episode()`. |
| `ground_truth` | `list[str]` | Correct labels, in the same order. |

### Output

A `dict` keyed by label. Each value is a dict with three keys:

```python
{
    "interview": {"correct": int, "total": int, "accuracy": float},
    "solo":      {"correct": int, "total": int, "accuracy": float},
    "panel":     {"correct": int, "total": int, "accuracy": float},
    "narrative": {"correct": int, "total": int, "accuracy": float},
}
```

---

### Spec fields — fill these in before writing code

**What does "correct" mean for a given class?**

```
[blank — be precise. When does an episode count as correctly classified
 for the "interview" class, for example?]

An episode counts as correct for class X when ground_truth[i] == X 
AND predictions[i] == X. Both must be true.
```

---

**What does "total" mean for a given class?**

```
[blank — is "total" the total number of predictions, or something more specific?]

Total is the number of episodes where ground_truth[i] == X — not the 
total number of predictions.
```

---

**Step-by-step logic:**

```
[blank — describe the steps your code will take.
 1. Initialize ...
 2. Loop over ...
 3. For each pair (predicted, truth) ...
 4. After the loop ...
 5. Return ...]

1. Initialize a dict for each label in VALID_LABELS with correct=0, total=0
2. Loop over each (predicted, truth) pair
3. If truth == label, increment total for that label
4. If truth == label AND predicted == label, increment correct
5. After the loop, compute accuracy = correct/total for each label
6. Return the dict
```

---

**Edge case — what if a class has no examples in ground_truth (total == 0)?**

```
[blank — what should accuracy be set to? Why?
 Hint: look at the docstring in evaluate.py.]

Set accuracy to 0.0 if total == 0. The class had no examples in the test 
set, so accuracy is undefined — 0.0 is a safe default that won't crash 
the report formatter.
```

---

**Worked example:**

```
predictions  = ["interview", "interview", "solo", "panel", "panel"]
ground_truth = ["interview", "solo",      "solo", "panel", "narrative"]

[blank — fill in the per-class results table below]

label       correct  total  accuracy
----------  -------  -----  --------
interview   [1]  [1]  [1.0]
solo        [1]  [2]  [0.5]
panel       [1]  [1]  [1.0]
narrative   [0]  [1]  [0.0]
```

---

## Reflection questions (discuss at the checkpoint)

1. Your overall accuracy might be decent even if one class has very low accuracy.
   Why is per-class accuracy a more informative metric than overall accuracy alone?
Overall accuracy hides class-level failures. A classifier that always 
predicts "interview" would score 25% overall but 0% on solo, panel, and 
narrative. Per-class accuracy reveals whether the model actually learned 
all four boundaries or just got lucky on the dominant class.

2. If `panel` episodes consistently get misclassified as `interview`, what does
   that tell you about your training labels or your prompt?
It means the training examples don't sufficiently distinguish multi-guest 
roundtables from two-person conversations. The model learned "multiple 
people talking" as the signal for both, without capturing that panel 
requires roughly equal standing and no clear host-guest dynamic.

3. You labeled 20 training episodes and evaluated on 20 test episodes (5 per class).
   How might the evaluation results change if you had labeled 100 training episodes?
   What if you had 200 test episodes?
More training examples would likely maintain or improve accuracy by giving 
the model more diverse cases to learn from, especially for edge cases. 
More test episodes (200) would give a more statistically reliable accuracy 
number — with only 5 episodes per class, one misclassification swings 
per-class accuracy by 20 percentage points.