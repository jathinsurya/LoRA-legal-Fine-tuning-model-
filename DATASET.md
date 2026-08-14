# Dataset Documentation

## Overview

The training dataset consists of US legislative bills paired with human-written summaries. It was supplied as a local JSONL file (`us_train_data_final_OFFICIAL.jsonl`) and loaded directly into the notebook at runtime via Google Colab's file upload interface.

---

## Format

The raw dataset is a JSONL file. Each line is a JSON object with the following fields:

| Field | Type | Description |
|---|---|---|
| `bill_id` | string | Unique identifier for the legislative bill |
| `title` | string | Full title of the bill |
| `text` | string | Full text of the legislative bill |
| `summary` | string | Human-written summary of the bill |
| `text_len` | integer | Character length of the bill text |
| `sum_len` | integer | Character length of the summary |

### Example Record (from notebook output)

**Bill Title:**
> To amend the Public Health Service Act to establish a 5-year pilot program under which health care providers are reimbursed by the Secretary of Health and Human Services for the costs associated with providing emergency medical care to aliens who are not lawfully present in the United States...

**Summary:**
> Border Hospital Survival and Illegal Immigrant Care Act - Amends the Public Health Service Act to direct the Secretary of Health and Human Services to establish a five-year pilot program of health care provider reimbursement for the costs associated with providing emergency medical and ambulance services in Arizona...

---

## Subset Used for Training

The full JSONL file contained **18,949 examples** (as observed during the `map` step). Only the first **500 examples** were selected for this training run via `dataset.select(range(500))` to fit within the Colab free-tier time constraints.

---

## Prompt Template

Each example was formatted into the following prompt template before tokenization:

```
You are an expert legal AI assistant.


Generate a concise and accurate summary of the following legislative bill.

### Bill Title:
{title}

### Legal Document:
{text}

### Summary:
{summary}<|endoftext|>
```

The EOS token (`<|endoftext|>`) was appended to each formatted example to prevent infinite generation during inference.

---

## Domain Notes

- All training data covers **US federal legislative bills** — not Indian law.
- The Indian constitutional law Q&A evaluation (see `RESULTS.md`) used a **different prompt format** and drew on the base model's pretrained knowledge, not the fine-tuned adapter's domain adaptation.
- For a genuinely India-focused legal model, the training data would need to be replaced with Indian legal corpora (e.g., Supreme Court judgements, IPC sections, constitutional articles).

---

## Data Splits

| Split | Size |
|---|---|
| Train | 500 (subset of 18,949) |
| Validation | None |
| Test | None (evaluation was purely qualitative at inference time) |
