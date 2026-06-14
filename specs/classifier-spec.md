# Classifier Spec — Pod Classifier

Complete this spec **before** writing any code for Milestone 2.

Use Plan or Ask mode to think through each blank field. When you're done,
your answers here become the blueprint for `build_few_shot_prompt()` and
`classify_episode()` in `classifier.py`.

---

## build_few_shot_prompt(labeled_examples, description)

### What it does
Constructs a prompt string for the LLM that includes the task instructions,
all labeled training examples, and the new episode description to classify.

### Inputs

| Parameter | Type | Description |
|---|---|---|
| `labeled_examples` | `list[dict]` | Each dict has `"title"`, `"description"`, `"label"` (and others). These are the examples you labeled in Milestone 1. |
| `description` | `str` | The episode description to classify. |

### Output

| Return value | Type | Description |
|---|---|---|
| prompt | `str` | A complete prompt string ready to send to the LLM. |

---

### Spec fields — fill these in before writing code

**Task instruction (what should the LLM know about the task?):**

```
You are classifying podcast episodes by their format. Classify the episode
into exactly one of these four labels:

- interview: a conversation between a host and one or more guests
- solo: a single host speaking from memory, experience, or opinion — no guests,
  no assembled external sources
- panel: multiple guests with roughly equal speaking time, often debating or
  discussing a topic together
- narrative: a story assembled from external sources — interviews, archival
  audio, reporting — with a clear narrative arc

Return only the label and your reasoning. Do not explain the taxonomy.
```

---

**How should labeled examples be formatted in the prompt?**

```
Each example should include the episode title, a brief excerpt or the full
description, and the correct label. Separate examples with a blank line or
a delimiter like "---". Include all fields that help the model see why the
label was applied — title and description are both useful; other fields
(like episode ID) are not needed.
```

---

**Example block sketch (write one concrete example):**

```
Title: {title}
Description: {description}
Label: {label}
```

---

**How should the new episode (to be classified) be presented?**

```
Present it in the same format as the labeled examples, but omit the Label
line and replace it with an instruction to classify. For example:

Title: {title}
Description: {description}
Label: ?

Then add a line like: "Classify the episode above. Return your answer in
the format below:" followed by the output format you chose.
```

---

**What output format should you request from the LLM?**

```
[blank — you need to parse the response in classify_episode(). What format
makes parsing reliable? Think about: a single label on its own line?
A structured format like "Label: X / Reasoning: Y"? JSON?
What are the tradeoffs?]

Use a structured format that is easy to parse reliably:

Label: <label>
Reasoning: <one or two sentences>

This is easier to parse than JSON (no need to handle malformed JSON) and 
more reliable than a single line (separates label from reasoning cleanly).
Strip and lowercase the label value before validating against VALID_LABELS.

```

---

**Edge cases to handle in the prompt:**

```
[blank — what if labeled_examples is empty? What if the description is very
short? How does your prompt handle these?]

- If labeled_examples is empty, skip the examples section and classify 
  based on the task instruction alone.
- If the description is very short, the prompt still works, no special 
  handling needed. The LLM will do its best with what's there.
```

---

## classify_episode(description, labeled_examples)

### What it does
Classifies a single podcast episode description using the few-shot LLM classifier.
Returns a dict with a label and reasoning.

### Inputs

| Parameter | Type | Description |
|---|---|---|
| `description` | `str` | The episode description to classify. |
| `labeled_examples` | `list[dict]` | Labeled training examples from `load_labeled_examples()`. |

### Output

| Return value | Type | Description |
|---|---|---|
| result | `dict` | Must have keys `"label"` and `"reasoning"`. `"label"` must be one of `VALID_LABELS` or `"unknown"`. |

---

### Spec fields — fill these in before writing code

**Step 1 — Build the prompt:**

```
Call build_few_shot_prompt(labeled_examples, description) and store the
returned string in a variable (e.g., prompt). Pass through both arguments
exactly as received — no modification needed before calling.
```

---

**Step 2 — Send to the LLM:**

```
Call _client.chat.completions.create() with:
  - model: the model name from config (LLM_MODEL)
  - messages: a list with one dict — {"role": "user", "content": prompt}
    (system-design.md shows an optional system message too — either shape works)
  - max_tokens: a reasonable limit (e.g., 200–300) to keep responses concise

Extract the response text from:
  response.choices[0].message.content
```

---

**Step 3 — Parse the response:**

```
[blank — how do you extract the label and reasoning from the LLM's text output?
What string operations or parsing logic do you need?
This depends on the output format you chose in build_few_shot_prompt.]

Split the response text by newlines. Find the line starting with "Label:" 
and extract the value after the colon, strip whitespace and convert to 
lowercase. Find the line starting with "Reasoning:" and extract the rest 
as the reasoning string. If either line is missing, set label to "unknown" 
and reasoning to the full raw response.

```

---

**Step 4 — Validate the label:**

```
[blank — what do you do if the LLM returns a label that isn't in VALID_LABELS?
What should label be set to?]

After parsing, check if the extracted label is in VALID_LABELS. If it is, 
use it. If not, set label to "unknown".

```

---

**Step 5 — Handle errors gracefully:**

```
[blank — what could go wrong? (Network error? Unparseable response?)
What should the function return if something fails?
Hint: the evaluation loop runs 20 calls — one bad response shouldn't crash everything.]

Wrap the entire function in a try/except. If anything fails (network error, 
unparseable response, missing keys), return:
{"label": "unknown", "reasoning": "Classification failed due to an error."}
This ensures one bad call doesn't crash the 20-episode evaluation loop.

```

---

### Return value structure

```python
{
    "label": str,      # one of VALID_LABELS, or "unknown" if invalid/error
    "reasoning": str,  # brief explanation from the LLM
}
```

---

## Notes on label quality

The classifier is only as good as your labels. If your training examples have
inconsistent or ambiguous labels, the LLM will learn the wrong pattern.

Before implementing the classifier, re-read `data/taxonomy.md` and double-check
any labels you're unsure about. Annotation quality is part of the lab.

---

## Implementation Notes

*Fill this in after implementing and testing both functions.*

**Test: what does the raw LLM response look like for one episode?**

```
Episode tested: Dr. Priya Nair on the Science of Sleep Deprivation
Raw response text: Label: interview
Reasoning: The episode features a conversation between a host and Dr. Priya 
Nair, discussing her research and studies on sleep deprivation, which matches 
the format of an interview.
```

**How did you parse the label out of the response?**

```
[describe the string operations — strip, split, lower, etc.]
Split response by newlines, find the line starting with "Label:", extract 
the value after the colon, strip whitespace and convert to lowercase, then 
validate against VALID_LABELS.
```

**Did any episodes return `"unknown"`? If so, why?**

```
[yes / no — if yes, what did the raw response look like?]
No, the Label:/Reasoning: format was returned consistently.
```

**One thing about the output format that surprised you:**

```
The model followed the exact output format requested without deviation, 
making parsing straightforward.

```
