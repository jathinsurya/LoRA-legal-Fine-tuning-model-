# Indian Legal LoRA Fine-Tuning

LoRA fine-tuning of **Qwen2.5-7B** on US legislative bill summarization using [Unsloth](https://github.com/unslothai/unsloth) for 2x faster training on a free Google Colab T4 GPU.

---

## Project Structure

```
Indian-Legal-LoRA-Fine-Tuning/
├── README.md
├── requirements.txt
├── .gitignore
├── notebooks/
│   └── indian_legal_lora_finetuning.ipynb
├── docs/
│   ├── DATASET.md
│   ├── METHODOLOGY.md
│   └── RESULTS.md
└── results/
    └── evaluation_results.md
```

---

## Quick Start

### 1. Install dependencies

```bash
pip install -r requirements.txt
```

### 2. Open the notebook

Open `notebooks/indian_legal_lora_finetuning.ipynb` in Google Colab (free T4 tier) or locally and run all cells.

### 3. Provide the dataset

Upload your `us_train_data_final_OFFICIAL.jsonl` file when prompted by the notebook. The dataset should contain `bill_id`, `title`, `text`, and `summary` fields.

---

## Model

| Setting | Value |
|---|---|
| Base Model | `unsloth/Qwen2.5-7B` |
| Method | LoRA (QLoRA, 4-bit) |
| LoRA Rank | 16 |
| Target Modules | q_proj, k_proj, v_proj, o_proj, gate_proj, up_proj, down_proj |
| Trainable Parameters | ~40.4M (0.53% of total) |

---

## Training Summary

| Metric | Value |
|---|---|
| GPU | NVIDIA Tesla T4 (Free Colab) |
| Training Examples | 500 |
| Epochs | 1 |
| Training Steps | 63 |
| Runtime | ~23.6 minutes |
| Peak VRAM Usage | 9.713 GB / 14.563 GB (66.7%) |

See [`docs/RESULTS.md`](docs/RESULTS.md) for full training metrics and evaluation results.

---

## Saved Artifacts

After training, the LoRA adapter is saved to `legal_lora_model/`:

```
legal_lora_model/
├── adapter_model.safetensors
├── adapter_config.json
├── tokenizer.json
└── tokenizer_config.json
```

Checkpoints are saved to `legal_outputs/checkpoint-25/` and `legal_outputs/checkpoint-50/`.

---

## Limitations

This is an experimental fine-tuning run on a very small dataset (500 examples, 1 epoch). It is **not** production-ready. See [`docs/RESULTS.md`](docs/RESULTS.md) for a full discussion of limitations.

---

## References

- [Unsloth](https://github.com/unslothai/unsloth) — 2x faster LoRA fine-tuning
- [Qwen2.5](https://huggingface.co/Qwen/Qwen2.5-7B) — Base model
- [HuggingFace TRL SFTTrainer](https://huggingface.co/docs/trl/sft_trainer)
- [PEFT](https://github.com/huggingface/peft) — LoRA implementation
