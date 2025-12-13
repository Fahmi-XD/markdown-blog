---
title: "Fine-Tuning LLM Cepat & Hemat Resource dengan Unsloth (Gratis di Google Colab)"
slug: "fine-tuning-llm-unsloth-colab"
date: "2025-12-13"
updated: "2025-12-13"

# — Author Info —
author:
  username: "Fahmi-XD"
  name: "Fahmi XD"
  role: "AI Engineer & Fullstack Developer"
  avatar: "https://avatars.githubusercontent.com/u/205189488?s=400"
  social:
    github: "https://github.com/fahmi-xd"
    website: "https://syntxflow.my.id"

# — Article Meta —
description: "Panduan lengkap fine-tuning LLM menggunakan Unsloth agar lebih cepat, hemat VRAM, dan bisa dijalankan gratis di Google Colab."
tags:
  - LLM Fine-tuning
  - Unsloth
  - LoRA
  - AI Engineering
  - Google Colab
readingTime: 12 # menit

cover:
  image: "https://i.pinimg.com/1200x/95/95/c2/9595c2aeb141c79ff590298e80cb4e4a.jpg"
  alt: "Illustration of efficient LLM fine-tuning"
  blur: true

# — Advance Features —
featured: true
draft: false

seo:
  keywords:
    - Fine tuning LLM
    - Unsloth tutorial
    - LLM hemat VRAM
    - Fine tuning gratis
    - Google Colab LLM
  canonical: "https://syntxflow.com/blog/fine-tuning-llm-unsloth-colab"
---

## Pendahuluan

Fine-tuning **Large Language Model (LLM)** sering dianggap mahal, lambat, dan butuh GPU kelas data center.  
Padahal, dengan **Unsloth**, kita bisa melakukan fine-tuning:

- ⚡ **2–5× lebih cepat**
- 🧠 **Hemat VRAM hingga 70%**
- 💸 **Gratis di Google Colab**
- 🛠️ Tanpa setup ribet

Artikel ini akan membahas **end-to-end workflow** fine-tuning LLM menggunakan **Unsloth + LoRA**, lengkap dengan **dataset, kode, dan notebook siap pakai**.

---

## Apa itu Unsloth?

**Unsloth** adalah framework optimasi fine-tuning LLM yang fokus pada:

- Efficient attention kernels
- Memory-efficient backpropagation
- Optimized LoRA injection
- Support model populer (LLaMA, Mistral, Qwen, Gemma, dll)

> 💡 Unsloth **tidak mengubah arsitektur model**, tapi mengoptimalkan *cara training-nya*.

---

## Kenapa Unsloth Lebih Cepat & Hemat?

### Dibanding pipeline standar (Transformers + PEFT):

| Aspek | Standar | Unsloth |
|-----|--------|--------|
| VRAM Usage | Tinggi | 🔥 Jauh lebih rendah |
| Training Speed | Normal | ⚡ 2–5× |
| Colab Gratis | ❌ Sering OOM | ✅ Aman |
| Setup | Ribet | Simpel |

Unsloth menghindari banyak **intermediate tensor allocation** yang biasanya bikin GPU kehabisan memori.

---

## Spesifikasi Minimum (Realistis)

Kamu bisa mulai hanya dengan:

- ✅ **Google Colab Free**
- ✅ GPU **T4 / P100**
- ✅ RAM 12–16 GB
- ✅ Disk ±15 GB

Model yang ideal:
- `LLaMA-3 8B`
- `Mistral 7B`
- `Qwen 7B`
- bahkan **4-bit quantized**

---

## Setup Google Colab

### 1️⃣ Aktifkan GPU
`Runtime → Change runtime type → GPU`

### 2️⃣ Install Dependency

```bash
pip install unsloth transformers accelerate datasets trl
````

---

## Load Model dengan Unsloth

```python
from unsloth import FastLanguageModel
import torch

model, tokenizer = FastLanguageModel.from_pretrained(
    model_name = "unsloth/llama-3-8b-bnb-4bit",
    max_seq_length = 2048,
    dtype = torch.float16,
    load_in_4bit = True,
)
```

> ⚠️ 4-bit loading ini kunci supaya **tidak OOM di Colab gratis**.

---

## Inject LoRA (Fine-Tuning Aman)

```python
model = FastLanguageModel.get_peft_model(
    model,
    r = 16,
    target_modules = ["q_proj", "k_proj", "v_proj", "o_proj"],
    lora_alpha = 32,
    lora_dropout = 0.05,
    bias = "none",
)
```

📌 Dengan LoRA:

* Model base **tidak berubah**
* Yang di-train cuma adapter kecil
* Bisa merge / reuse kapan saja

---

## Dataset Format (Best Practice)

Gunakan format **instruction-style**:

```json
{
  "instruction": "Jelaskan apa itu AI Persona",
  "input": "",
  "output": "AI Persona adalah..."
}
```

Load dataset:

```python
from datasets import load_dataset

dataset = load_dataset("json", data_files="dataset.json")
```

---

## Training Configuration

```python
from trl import SFTTrainer
from transformers import TrainingArguments

trainer = SFTTrainer(
    model = model,
    tokenizer = tokenizer,
    train_dataset = dataset["train"],
    args = TrainingArguments(
        per_device_train_batch_size = 2,
        gradient_accumulation_steps = 4,
        learning_rate = 2e-4,
        max_steps = 1000,
        fp16 = True,
        logging_steps = 10,
        output_dir = "./outputs",
        report_to = "none",
    ),
)
```

---

## Mulai Fine-Tuning

```python
trainer.train()
```

⏱️ Estimasi:

* Dataset kecil (5k–10k samples): **10–30 menit**
* Colab Free: **aman tanpa OOM**

---

## Save & Export Model

```python
model.save_pretrained("lora-adapter")
tokenizer.save_pretrained("lora-adapter")
```

Atau merge ke model base:

```python
model = model.merge_and_unload()
model.save_pretrained("merged-model")
```

---

## Inference Hasil Fine-Tuning

```python
prompt = "Jelaskan manfaat AI Persona dalam produk digital"

inputs = tokenizer(prompt, return_tensors="pt").to("cuda")
outputs = model.generate(**inputs, max_new_tokens=200)

print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```

---

## Download Notebook Siap Pakai

📓 **Google Colab Notebook (Ready-to-Run)**
👉 **Download:**
[https://github.com/syntxflow/llm-unsloth-colab/blob/main/unsloth_finetune.ipynb](https://github.com/syntxflow/llm-unsloth-colab/blob/main/unsloth_finetune.ipynb)

> Tinggal **Open in Colab → Run All**

---

## Kesalahan Umum (Dan Cara Menghindarinya)

* ❌ Batch size terlalu besar → OOM
* ❌ Tidak pakai 4-bit → GPU jebol
* ❌ Dataset tidak instruction-based
* ❌ Training terlalu lama tanpa eval

---

## Penutup

Dengan **Unsloth**, fine-tuning LLM bukan lagi hal mahal atau eksklusif.
Bahkan dengan **Google Colab gratis**, kamu bisa:

* Melatih AI persona khusus
* Mengadaptasi LLM ke domain bisnis
* Eksperimen tanpa takut boros resource

> 🚀 Ini game changer untuk AI engineer indie & startup.

---

### Referensi

* [https://github.com/unslothai/unsloth](https://github.com/unslothai/unsloth)
* [https://huggingface.co/docs/peft](https://huggingface.co/docs/peft)
* [https://github.com/huggingface/trl](https://github.com/huggingface/trl)

---
