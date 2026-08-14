# Evaluation Results

> Raw inference outputs from the fine-tuned Qwen2.5-7B LoRA model. All outputs are exactly as produced by the notebook — no values have been fabricated or edited.

---

## Test 1: Bill Summarization

**Prompt format:** Training format (`### Summary:` template)
**Decoding:** Greedy (`max_new_tokens=200`, `use_cache=True`)

### Input

**Bill Title:**
```
Border Hospital Survival and Illegal Immigrant Care Act
```

**Legal Document (excerpt used):**
```
SECTION 1.

This Act establishes a five-year reimbursement program for hospitals
that provide emergency medical care to undocumented immigrants in Arizona.
The Secretary of Health and Human Services will reimburse eligible
providers for emergency services.
```

### Output

```
The Border Hospital Survival and Illegal Immigrant Care Act establishes a five-year
reimbursement program for hospitals that provide emergency medical care to undocumented
immigrants in Arizona. The Secretary of Health and Human Services will reimburse eligible
providers for emergency services.
```

**Notes:**
- Output is accurate and well-formatted.
- The response closely mirrors the input text; it is largely extractive rather than abstractive for this short input.
- Generation terminated within the 200-token budget.

---

## Test 2: Indian Constitutional Law Q&A

**Prompt format:** Instruction-following format (not seen during training)
**Decoding:** Greedy (`max_new_tokens=256`, `use_cache=True`)

> **Important:** This prompt format and domain (Indian constitutional law) were not part of the training data. These outputs reflect the base model's pretrained legal knowledge, not the effect of fine-tuning.

---

### Q1: What is Article 21 of the Indian Constitution?

**Output (truncated at 256 tokens):**
```
Article 21 of the Indian Constitution is a fundamental right that guarantees the
right to life and personal liberty to all individuals within the territory of India.
It states that no person shall be deprived of his life or personal liberty except
according to procedure established by law. This means that the state cannot take
away a person's life or liberty without following a fair and just legal process.

The right to life under Article 21 includes the right to live with dignity and the
right to be free from arbitrary deprivation of life. It also includes the right to
personal liberty, which means that a person cannot be d[TRUNCATED]
```

**Notes:** Factually reasonable. Final word cut off at token limit.

---

### Q2: What is judicial review in India?

**Output (truncated at 256 tokens):**
```
Judicial review in India refers to the power of the judiciary to review and
invalidate laws, executive actions, and administrative decisions that are found to
be unconstitutional or in violation of fundamental rights. This power is derived
from the Constitution of India, which provides for the separation of powers between
the executive, legislative, and judicial branches of government.

The Sup[TRUNCATED]
```

**Notes:** Factually plausible. Truncated before completing the description of the Supreme Court's role. Recommend `max_new_tokens=400` for complete responses.

---

### Q3: What is the purpose of a writ petition?

**Output (truncated at 256 tokens):**
```
A writ petition is a legal document that is filed in a court of law to seek a
remedy or relief from a higher court. The purpose of a writ petition is to challenge
the legality of an action or decision taken by a lower court or administrative body,
or to seek a declaration of the lawfulness of a particular action or decision.
The writ petition is typically filed when the petitioner b[TRUNCATED]
```

**Notes:** Accurate description. Truncated mid-sentence.

---

### Q4: What is the difference between a civil case and a criminal case?

**Output (truncated at 256 tokens):**
```
A civil case is a legal dispute between two or more parties, where one party is
seeking a remedy or compensation for a wrong or injury caused by the other party.
The parties involved in a civil case are typically individuals or organizations,
and the case is usually brought before a civil court. The purpose of a civil case
is to reso[TRUNCATED]
```

**Notes:** Accurate for the civil side. Truncated before the criminal case comparison could be completed.

---

## Summary Table

| Test | Input Domain | Prompt Format Seen in Training? | Factual Accuracy | Completeness |
|---|---|---|---|---|
| Bill Summarization | US Legislative | Yes | Accurate | Complete |
| Article 21 Q&A | Indian Constitutional | No | Reasonable | Truncated |
| Judicial Review Q&A | Indian Constitutional | No | Reasonable | Truncated |
| Writ Petition Q&A | Indian Constitutional | No | Accurate | Truncated |
| Civil vs Criminal Q&A | Indian Constitutional | No | Partial (civil side only) | Truncated |

---

## Recommendations

- Increase `max_new_tokens` to 400–512 for the Q&A task to allow complete responses.
- Add quantitative metrics (ROUGE-L, BERTScore) for the summarization task.
- Test the base model (`unsloth/Qwen2.5-7B` without the adapter) on the same inputs to isolate the fine-tuning effect.
- Expand the evaluation set beyond 1 bill and 4 Q&A questions before drawing any performance conclusions.
