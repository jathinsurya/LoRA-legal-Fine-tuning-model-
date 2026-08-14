# Training and Evaluation Results

> All values in this document are extracted directly from the notebook's cell outputs. No values have been fabricated or inferred.

---

## 1. Training Metrics

### Hardware

| Metric | Value |
|---|---|
| GPU | NVIDIA Tesla T4 |
| Total VRAM | 14.563 GB |
| Platform | Google Colab (Free Tier) |
| CUDA Version | 12.8 |
| Triton Version | 3.6.0 |
| PyTorch Version | 2.11.0+cu128 |
| Unsloth Version | 2026.8.15 |

### Model Scale

| Metric | Value |
|---|---|
| Total Model Parameters | 7,655,986,688 (~7.66B) |
| Trainable LoRA Parameters | 40,370,176 (~40.4M) |
| % Parameters Trained | 0.53% |

### Memory Usage

| Metric | Value |
|---|---|
| GPU Memory Reserved (Before Training) | 7.242 GB |
| Peak Reserved Memory (During Training) | 9.713 GB |
| Peak Memory Used by LoRA Training Only | 2.471 GB |
| Peak Memory as % of Total VRAM | 66.696% |
| LoRA Training Memory as % of Total VRAM | 16.968% |

### Training Configuration

| Metric | Value |
|---|---|
| Training Examples | 500 |
| Number of Epochs | 1 |
| Total Training Steps | 63 |
| Effective Batch Size | 8 (2 per device × 4 gradient accumulation steps) |
| Training Runtime | 1417.13 seconds (~23.62 minutes) |

### Training Loss Log

Loss was recorded every 10 steps:

| Step | Training Loss |
|---|---|
| 10 | 0.9806 |
| 20 | 1.0152 |
| 30 | 1.0143 |
| 40 | 1.0419 |
| 50 | 1.0329 |
| 60 | 0.9969 |

**Observation:** The training loss remained relatively flat throughout training (range: 0.98–1.04), with no consistent downward trend. This is consistent with a very short training run (63 steps) on a small dataset. It may indicate:
- The model requires more training steps to meaningfully adapt
- The effective batch size of 8 introduces significant gradient noise
- The task or prompt format is difficult for the model within this limited training budget

Training loss is a measure of how well the model predicts the training tokens at each step. It is **not equivalent to generation quality, factual accuracy, or task performance.** A lower loss would indicate better memorization of training examples but does not guarantee good generalization to unseen inputs.

### Saved Artifacts

| Artifact | Location |
|---|---|
| LoRA Adapter | `legal_lora_model/adapter_model.safetensors` |
| Adapter Config | `legal_lora_model/adapter_config.json` |
| Tokenizer | `legal_lora_model/tokenizer.json`, `legal_lora_model/tokenizer_config.json` |
| Training Checkpoints | `legal_outputs/checkpoint-25/`, `legal_outputs/checkpoint-50/` |

---

## 2. Evaluation Methodology

### Evaluation Type

Evaluation was **purely qualitative.** No quantitative metrics (ROUGE, BLEU, BERTScore, LLM-judge scores, perplexity) were computed.

### No Baseline Comparison

The base `unsloth/Qwen2.5-7B` model was **not tested** on the same inputs. No direct comparison between base model and fine-tuned model outputs is possible from the notebook data.

### Test 1: Bill Summarization

The model was tested using the same prompt format as training:

```
You are an expert legal AI assistant.


Generate a concise and accurate summary of the following legislative bill.

### Bill Title:
{title}

### Legal Document:
{text}

### Summary:

```

- `max_new_tokens`: 200
- `use_cache`: True
- Decoding: Greedy (default)

### Test 2: Indian Constitutional Law Q&A

A different prompt format was used for this evaluation:

```
Below is an instruction related to Indian law.
Provide a clear and informative legal response.

Instruction:

{question}

Response:

```

- `max_new_tokens`: 256
- `use_cache`: True
- Decoding: Greedy (default)

> **Important:** This prompt format was not used during training. The training data contained US legislative bills, not Indian constitutional law material. These evaluation results test the base model's general legal knowledge, not fine-tuning adaptation.

---

## 3. Evaluation Results

### 3.1 Bill Summarization Test

**Input Bill Title:** Border Hospital Survival and Illegal Immigrant Care Act

**Input Legal Document:**
```
SECTION 1.

This Act establishes a five-year reimbursement program for hospitals
that provide emergency medical care to undocumented immigrants in Arizona.
The Secretary of Health and Human Services will reimburse eligible
providers for emergency services.
```

**Generated Summary (actual output from notebook):**
```
The Border Hospital Survival and Illegal Immigrant Care Act establishes a five-year
reimbursement program for hospitals that provide emergency medical care to undocumented
immigrants in Arizona. The Secretary of Health and Human Services will reimburse eligible
providers for emergency services.
```

**Observation:** The generated summary is accurate, well-formatted, and follows the `### Summary:` template. However, the output closely mirrors the input text with minimal abstraction. For this short, simple input document, it is not clear whether the model is generating an abstractive summary or primarily performing extractive copying. The model correctly terminates within the 200-token budget.

---

### 3.2 Indian Constitutional Law Q&A Results

#### Q1: What is Article 21 of the Indian Constitution?

**Generated Response (actual output, truncated at 256 tokens):**
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

**Observation:** The response is factually reasonable. The final word was cut off at the 256-token limit, indicating the model had more to generate. The content is consistent with the actual scope of Article 21.

---

#### Q2: What is judicial review in India?

**Generated Response (actual output, truncated at 256 tokens):**
```
Judicial review in India refers to the power of the judiciary to review and
invalidate laws, executive actions, and administrative decisions that are found to
be unconstitutional or in violation of fundamental rights. This power is derived
from the Constitution of India, which provides for the separation of powers between
the executive, legislative, and judicial branches of government.

The Sup[TRUNCATED]
```

**Observation:** Factually plausible and correctly identifies the constitutional basis. Truncated before completion — the model was likely about to describe the Supreme Court's role. Increasing `max_new_tokens` to 400–512 would allow complete responses.

---

#### Q3: What is the purpose of a writ petition?

**Generated Response (actual output, truncated at 256 tokens):**
```
A writ petition is a legal document that is filed in a court of law to seek a
remedy or relief from a higher court. The purpose of a writ petition is to challenge
the legality of an action or decision taken by a lower court or administrative body,
or to seek a declaration of the lawfulness of a particular action or decision.
The writ petition is typically filed when the petitioner b[TRUNCATED]
```

**Observation:** Accurate description of writ petitions. Truncated mid-sentence.

---

#### Q4: What is the difference between a civil case and a criminal case?

**Generated Response (actual output, truncated at 256 tokens):**
```
A civil case is a legal dispute between two or more parties, where one party is
seeking a remedy or compensation for a wrong or injury caused by the other party.
The parties involved in a civil case are typically individuals or organizations,
and the case is usually brought before a civil court. The purpose of a civil case
is to reso[TRUNCATED]
```

**Observation:** Accurate description of the civil case side of the distinction. Truncated before completing the comparison with criminal cases.

---

## 4. Summary of Observations

| Aspect | Observation |
|---|---|
| Bill Summarization Format | Model correctly follows `### Summary:` template |
| Summary Content | Accurate for the test bill; closely mirrors input text |
| Abstractive vs. Extractive | Appears largely extractive for short inputs |
| Indian Law Q&A Factual Accuracy | Responses appear factually reasonable |
| Response Completeness | All Indian law responses were truncated by `max_new_tokens` limit |
| Hallucination | No obvious hallucinations observed in the limited test set |
| Format Adherence | Indian law prompt format correctly followed despite not being a training format |
| Baseline Comparison | Not available — base model not tested on same inputs |

---

## 5. Limitations of This Evaluation

- **No quantitative metrics.** ROUGE, BLEU, BERTScore, or human evaluation scores are absent.
- **Only 1 bill summarization test.** A single example provides no statistical basis for performance claims.
- **Only 4 Q&A questions tested.** Insufficient for any generalization claims.
- **No base model comparison.** Cannot isolate the effect of fine-tuning.
- **Domain mismatch in Q&A.** Indian constitutional law was not in the training data.
- **Response truncation.** Multiple responses were cut off, preventing assessment of complete answers.
- **Single inference run.** No temperature or sampling — all outputs are greedy deterministic, which may not reflect the model's response diversity.
- **Self-contained evaluation.** Evaluation was performed on the same machine and model state used for training, with no independent test set.

---

## 6. What This Experiment Demonstrates

This experiment successfully demonstrates:

1. The **end-to-end LoRA fine-tuning pipeline** using Unsloth + HF TRL on a resource-constrained GPU
2. The **feasibility of 4-bit quantized training** within 14.563 GB VRAM
3. The **adapter saving and inference workflow** for LoRA-based models
4. The **prompt template adherence** of the fine-tuned model for bill summarization

It does **not** demonstrate:
- That the fine-tuned model outperforms the base model
- Statistically significant improvement on any legal NLP task
- Reliable factual accuracy across a broad range of legal queries
- Production-ready legal summarization capability
