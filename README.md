# Gita-Qwen-0.5B-V2

A parameter-efficient fine-tuning experiment that adapts
`Qwen/Qwen2.5-0.5B-Instruct` for Bhagavad Gita question answering using LoRA.

🔗 **Model:** https://huggingface.co/Rudraksh001/gita-qwen-0.5b-v2

## Overview

Gita-Qwen-0.5B-V2 is a LoRA fine-tune of Qwen2.5-0.5B-Instruct trained on the English subset of the
`JDhruv14/Bhagavad-Gita-QA` dataset.

The goal of the project was to explore how effectively a small language model can be adapted to a
specific knowledge domain using parameter-efficient fine-tuning.

## Model

| Property | Value |
|---|---|
| Base model | Qwen/Qwen2.5-0.5B-Instruct |
| Fine-tuning | LoRA / PEFT |
| LoRA rank | 16 |
| LoRA alpha | 32 |
| LoRA dropout | 0.05 |
| Target modules | q_proj, k_proj, v_proj, o_proj |
| Trainable parameters | 2,162,688 |
| Total parameters | 496,195,456 |
| Trainable percentage | 0.4359% |
| Training examples | 3,500 |
| Training epochs | 5 |
| Maximum sequence length | 256 |
| Learning rate | 2e-4 |
| Scheduler | Cosine |
| Precision | FP16 |
| Compute | Kaggle free GPU |

## Dataset

Training used:

**JDhruv14/Bhagavad-Gita-QA**

https://huggingface.co/datasets/JDhruv14/Bhagavad-Gita-QA

The dataset contains 3,500 English question-answer examples covering 700 Bhagavad Gita verses.

The dataset is released under the MIT License.

## Training

The model was trained using LoRA through PEFT.

The data was split by **verse rather than individual question** to prevent questions from the same verse appearing in both training and evaluation subsets.

- Training verses: 595
- Evaluation verses: 105
- Training examples: 2,975
- Evaluation examples: 525

The best checkpoint was selected using validation loss.

### Best validation result

**Validation loss: 1.8104**

## Evaluation

A fixed 50-question verse-level benchmark was constructed from the evaluation verses.

The same questions were given to:

1. `Qwen/Qwen2.5-0.5B-Instruct`
2. `Gita-Qwen-0.5B-V2`

Generation was performed deterministically with:

- `do_sample=False`
- `max_new_tokens=150`

Both models successfully generated responses for all 50 benchmark questions.

The raw benchmark outputs are included in this repository.

This benchmark is an open-ended generation evaluation and should not be interpreted as an exact-match accuracy score.

## Limitations

This model is an experimental fine-tune.

It can produce relevant Bhagavad Gita-related explanations, but it can also:

- confuse teachings between different verses
- produce generic philosophical responses
- hallucinate verse-specific information
- give confident but incorrect interpretations

The model should therefore not be treated as an authoritative source for scripture, religious instruction, or scholarly interpretation.

For serious study, consult the original Sanskrit text and reliable translations or commentaries.

## Repository Structure

```text
gita-qwen-0.5b-v2/
│
├── README.md
├── LICENSE
│
├── training/
│   └── gita-qwen-0.5b-v2-finetuning.ipynb
│
├── evaluation/
│   ├── gita_v2_benchmark_50.json
│   ├── gita_v2_benchmark_base.json
│   ├── gita_v2_benchmark_v2.json
│   └── gita_v2_benchmark_results.json
│
├── inference/
│   └── inference.py
│
└── requirements.txt
