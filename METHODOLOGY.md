# Methodology

## Overview

This experiment fine-tunes `unsloth/Qwen2.5-7B` (a 7.66B parameter language model) on a legal bill summarization task using LoRA (Low-Rank Adaptation) with 4-bit quantization via Unsloth. The entire pipeline runs on a free Google Colab Tesla T4 GPU.

---

## Environment

| Component | Version / Value |
|---|---|
| Platform | Google Colab (Free Tier) |
| GPU | NVIDIA Tesla T4 |
| Total VRAM | 14.563 GB |
| CUDA Version | 12.8 |
| PyTorch | 2.11.0+cu128 |
| Triton | 3.6.0 |
| Unsloth | 2026.8.15 |
| Transformers | 5.5.0 |
| TRL | 0.24.0 |
| PEFT | 0.19.1 |

---

## Base Model

**Model:** `unsloth/Qwen2.5-7B`

Qwen2.5-7B is a 7.66B parameter decoder-only transformer from Alibaba. It was loaded in **4-bit NF4 quantization** (`load_in_4bit=True`) using bitsandbytes, which significantly reduces VRAM usage with minimal quality loss.

```python
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/Qwen2.5-7B",
    max_seq_length=2048,
    dtype=None,          # Auto-detected: float16 for T4
    load_in_4bit=True,
)
```

---

## LoRA Configuration

LoRA adapters were applied to the attention and MLP projection layers. Only the adapter weights (~40.4M parameters, 0.53% of total) were trained; the base model weights remained frozen.

```python
model = FastLanguageModel.get_peft_model(
    model,
    r=16,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj",
                    "gate_proj", "up_proj", "down_proj"],
    lora_alpha=16,
    lora_dropout=0,
    bias="none",
    use_gradient_checkpointing="unsloth",
    random_state=3407,
    use_rslora=False,
    loftq_config=None,
)
```

| LoRA Hyperparameter | Value |
|---|---|
| Rank (r) | 16 |
| Alpha | 16 |
| Dropout | 0 (optimized for Unsloth) |
| Bias | none |
| Gradient Checkpointing | "unsloth" (30% less VRAM) |

---

## Data Preparation

The dataset was formatted using a structured prompt template:

```
You are an expert legal AI assistant.


Generate a concise and accurate summary of the following legislative bill.

### Bill Title:
{title}

### Legal Document:
{text}

### Summary:
{summary}<EOS>
```

500 examples were selected from the full dataset of 18,949. Each example was tokenized with a max sequence length of 2048 tokens.

---

## Training Configuration

Training used HuggingFace TRL's `SFTTrainer` with the following `TrainingArguments`:

```python
TrainingArguments(
    per_device_train_batch_size=2,
    gradient_accumulation_steps=4,
    num_train_epochs=1,
    learning_rate=2e-4,
    warmup_steps=20,
    fp16=True,            # T4 does not support bfloat16
    logging_steps=10,
    optim="adamw_8bit",
    weight_decay=0.01,
    lr_scheduler_type="linear",
    seed=3407,
    output_dir="legal_outputs",
    report_to="none",
    save_strategy="steps",
    save_steps=25,
    save_total_limit=2,
    save_only_model=True,
)
```

| Training Hyperparameter | Value |
|---|---|
| Per-device batch size | 2 |
| Gradient accumulation steps | 4 |
| Effective batch size | 8 |
| Learning rate | 2e-4 |
| Warmup steps | 20 |
| LR scheduler | Linear decay |
| Optimizer | AdamW (8-bit) |
| Weight decay | 0.01 |
| Precision | FP16 |
| Epochs | 1 |

---

## Inference

Two inference tasks were run after training.

### Task 1: Bill Summarization

The fine-tuned model was tested using the same prompt format as training (without the summary field):

```python
inputs = tokenizer(prompt, return_tensors="pt").to("cuda")
outputs = model.generate(**inputs, max_new_tokens=200, use_cache=True)
```

### Task 2: Indian Constitutional Law Q&A

A different prompt format was used (not seen during training):

```
Below is an instruction related to Indian law.
Provide a clear and informative legal response.

Instruction:
{question}

Response:
```

```python
outputs = model.generate(**inputs, max_new_tokens=256, use_cache=True)
```

Both tasks used greedy decoding (no temperature or top-p sampling).

---

## Model Saving

The LoRA adapter (not the merged model) was saved locally:

```python
model.save_pretrained("legal_lora_model")
tokenizer.save_pretrained("legal_lora_model")
```

Saved artifacts:
- `legal_lora_model/adapter_model.safetensors`
- `legal_lora_model/adapter_config.json`
- `legal_lora_model/tokenizer.json`
- `legal_lora_model/tokenizer_config.json`

Training checkpoints were saved at steps 25 and 50 under `legal_outputs/`.
