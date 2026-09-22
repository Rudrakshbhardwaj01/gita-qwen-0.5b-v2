# Gita-Qwen-0.5B-V2

A parameter-efficient fine-tuning experiment that adapts [`Qwen/Qwen2.5-0.5B-Instruct`](https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct) for Bhagavad Gita question answering using LoRA.

The model was trained entirely on free Kaggle GPU compute and released as a public LoRA adapter on Hugging Face.

[Model on Hugging Face](https://huggingface.co/Rudraksh001/gita-qwen-0.5b-v2)

---

## Overview

Gita-Qwen-0.5B-V2 explores how effectively a small instruction-tuned language model can be specialized for a narrow knowledge domain using parameter-efficient fine-tuning.

The project uses the English subset of the
[`JDhruv14/Bhagavad-Gita-QA`](https://huggingface.co/datasets/JDhruv14/Bhagavad-Gita-QA) dataset and fine-tunes Qwen2.5-0.5B-Instruct with LoRA.

## Model

| Property | Value |
| --- | --- |
| Base model | Qwen2.5-0.5B-Instruct |
| Fine-tuning | LoRA / PEFT |
| LoRA rank | 16 |
| LoRA alpha | 32 |
| LoRA dropout | 0.05 |
| Target modules | `q_proj`, `k_proj`, `v_proj`, `o_proj` |
| Trainable parameters | 2,162,688 |
| Total parameters | 496,195,456 |
| Trainable percentage | 0.4359% |
| Training examples | 3,500 |
| Epochs | 5 |
| Max sequence length | 256 |
| Learning rate | 2e-4 |
| Scheduler | Cosine |
| Precision | FP16 |
| Compute | Free Kaggle GPU |

## Dataset

Training used the English subset of:

[JDhruv14/Bhagavad-Gita-QA](https://huggingface.co/datasets/JDhruv14/Bhagavad-Gita-QA)

The dataset contains:

- 3,500 question-answer examples
- 700 verses
- 5 questions per verse

The dataset was split at the verse level rather than the individual question level to avoid questions from the same verse appearing in both training and evaluation.

| Split | Verses | Examples |
| --- | --- | --- |
| Training | 595 | 2,975 |
| Evaluation | 105 | 525 |

## Training

The model was trained for 5 epochs using LoRA through Hugging Face PEFT.

The best checkpoint was selected using validation loss.

**Best validation loss: 1.8104**

Training was performed entirely on a free Kaggle GPU environment.

**Compute cost: ₹0 / $0**

## Evaluation

A fixed 50-question verse-level benchmark was evaluated using both the base model and Gita-Qwen-0.5B-V2.

Each benchmark question corresponds to a different verse.

| Model | Questions | Successful generations |
| --- | --- | --- |
| Qwen2.5-0.5B-Instruct | 50 | 50/50 |
| Gita-Qwen-0.5B-V2 | 50 | 50/50 |

Generation was deterministic using:

- `do_sample=False`
- `max_new_tokens=150`

The benchmark is an open-ended evaluation and does not represent an exact accuracy score.

## Example

Prompt:

```text
According to Bhagavad Gita Chapter 2, Verse 47:

What is the central teaching of this verse?
````

The model can be loaded directly using the base Qwen model and the released LoRA adapter.

## Usage

Install the required libraries:

```bash
pip install torch transformers peft
```

Load the model:

```python
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM from peft import PeftModel

BASE_MODEL = "Qwen/Qwen2.5-0.5B-Instruct"
ADAPTER = "Rudraksh001/gita-qwen-0.5b-v2"

tokenizer = AutoTokenizer.from_pretrained(BASE_MODEL)

base_model = AutoModelForCausalLM.from_pretrained( BASE_MODEL, torch_dtype=torch.float16, device_map="auto" )

model = PeftModel.from_pretrained(
    base_model,
    ADAPTER
)

model.eval()

messages = [
    {
        "role": "user",
        "content": (
"According to Bhagavad Gita Chapter 2, Verse 47:\n\n" "What is the central teaching of this verse?"
        )
    }
]

inputs = tokenizer.apply_chat_template(
    messages,
    add_generation_prompt=True,
    return_tensors="pt"
)

input_ids = inputs["input_ids"].to(model.device) attention_mask = inputs["attention_mask"].to(model.device)

with torch.no_grad():
    outputs = model.generate(
        input_ids=input_ids,
        attention_mask=attention_mask,
        max_new_tokens=150,
        do_sample=False,
        pad_token_id=tokenizer.eos_token_id
    )

answer = tokenizer.decode(
    outputs[0][input_ids.shape[-1]:],
    skip_special_tokens=True
)

print(answer)
```

## Limitations

This model is an experimental fine-tune and should not be treated as an authoritative source for the Bhagavad Gita.

It may:

- confuse teachings between verses
- generate generic explanations
- hallucinate verse-specific details
- produce confident but incorrect answers
- reflect limitations or inaccuracies present in its training data

Important information should be independently verified against reliable translations, primary texts, or scholarly sources.

## Project Structure

The model weights are hosted on Hugging Face rather than stored in this repository.

```text
gita-qwen-0.5b-v2/
├── README.md
└── LICENSE
```

## Links

- Hugging Face:
  [[huggingface.co/Rudraksh001/gita-qwen-0.5b-v2](https://huggingface.co/Rudraksh001/gita-qwen-0.5b-v2)](https://huggingface.co/Rudraksh001/gita-qwen-0.5b-v2)
- Dataset:
  [[huggingface.co/datasets/JDhruv14/Bhagavad-Gita-QA](https://huggingface.co/datasets/JDhruv14/Bhagavad-Gita-QA)](https://huggingface.co/datasets/JDhruv14/Bhagavad-Gita-QA)
- Base model:
  [[huggingface.co/Qwen/Qwen2.5-0.5B-Instruct](https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct)](https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct)

## Acknowledgements

This project builds on:

- Qwen2.5-0.5B-Instruct
- Hugging Face Transformers
- Hugging Face PEFT
- PyTorch
- Bhagavad-Gita-QA by `JDhruv14`

## License

This repository is licensed under the Apache License 2.0.

The base model and dataset are separate third-party resources with their own licenses and terms.
