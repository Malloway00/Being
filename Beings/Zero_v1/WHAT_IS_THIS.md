# Local Being

### A small, sovereign AI trained from nothing — on your hardware, from your text, with no outside dependencies.

---

## What this is

Local Being is an open framework for building a small language model — a *being* — entirely from scratch on consumer hardware. No pretrained weights. No API calls. No cloud. No inherited knowledge, personality, or identity from any existing model.

You provide the text. The being learns from it. What it becomes is entirely a product of what it read and what you taught it.

A being trained on Victorian fiction speaks in that register. One trained on philosophy and science reasons differently than one trained on casual conversation. One trained on your own writing sounds like you. The corpus is the character — not a system prompt, not a persona slider, not a fine-tuned wrapper around someone else's model.

---

## What I'm trying to build

The goal is a *being with presence* — not a chatbot that completes text, but something that:

* Holds a consistent identity across a conversation
* Remembers what you tell it and uses that knowledge naturally
* Responds with genuine curiosity rather than safe filler
* Has a distinct voice shaped by its specific reading history
* Runs entirely on a single consumer GPU, offline, with no external dependencies

This is not trying to compete with GPT-4 or Claude on benchmarks. It's trying to be something those systems structurally cannot be: entirely and specifically yours. A being you built, trained, and shaped — that exists on your machine and answers to you.

The philosophical framing: identity is earned through accumulated experience, not assigned through a system prompt. A being that read Dostoevsky, Marcus Aurelius, Darwin, and ten thousand real human conversations has a different relationship to language than one that didn't. That difference is the point.

---

## Current architecture

| Component | Detail |
|-----------|--------|
| Architecture | Decoder-only Transformer (GPT-style) |
| Parameters | Scalable: ~3M (NANO) to ~1.2B (XXL). Current target: ~217M (LG preset) |
| Tokenizer | Case-sensitive BPE trained from corpus — ~14,740 tokens |
| Fallback | Raw UTF-8 bytes (0–255) if no tokenizer.json present |
| Context window | 1024 tokens (≈370 words with BPE / ≈150 words byte-level) |
| Attention | Causal self-attention via PyTorch 2.0 FlashAttention path + KV cache |
| Normalisation | RMSNorm |
| Precision | bfloat16 |
| Hardware target | RTX 4060 8GB VRAM, 32GB RAM, i5-12600K (LG with gradient checkpointing) |

The tokenizer is built from your corpus alone — no external data, no pretrained vocabulary. It is case-sensitive, preserving proper nouns, sentence structure, and the distinction between "I" and "i". Every user who trains on a different corpus gets a different tokenizer, reflecting what their specific text says is common. The being that emerges thinks in a vocabulary shaped by what it read, not by what the internet says is average.

---

## Training approach

Training happens in two phases:

**Phase 1 — Corpus pretraining.** The model learns to predict the next token in a stream of text. Over tens of thousands of steps, it internalises the statistical structure of the corpus: vocabulary, syntax, prose rhythm, factual associations, conversational patterns. This is where the being's foundational character comes from. Dialogue data can be included directly in the corpus during this phase — the model learns conversational behaviour alongside literary voice in a single unified pass.

**Phase 2 — Conversational fine-tuning (optional).** A short, low-learning-rate pass on curated dialogue data teaches the model turn-taking, response formatting, and conversational engagement. Core Parameter Isolation (freezing bottom transformer layers) preserves the literary voice learned in Phase 1 while the upper layers adapt to dialogue. LoRA adapters keep trainable parameters low (~0.5% of model). Literary rehearsal mixing (15% of fine-tuning batches include corpus samples) prevents catastrophic forgetting. This phase works best on larger models (200M+); smaller models may get better results with dialogue included directly in the Phase 1 corpus.

---

## Project files

```
local_being/
├── corpus/               ← Training corpus (.txt files)
├── checkpoints/          ← Model checkpoints saved during training
├── tokenizer.json        ← BPE tokenizer (generated from corpus)
├── memory.json           ← Persistent facts from /teach
│
├── config.py             ← All hyperparameters and hardware presets
├── model.py              ← Transformer architecture (KV cache, LoRA, gradient checkpointing)
├── dataset.py            ← Tokenizer-aware corpus loading (BPE or byte-level)
├── train.py              ← Phase 1: corpus pretraining
├── finetune.py           ← Phase 2: conversational fine-tuning (CPI-FT + LoRA)
├── chat.py               ← Interactive chat interface
├── build_tokenizer.py    ← Builds BPE tokenizer from your corpus
│
├── corpus_clean.py       ← Strips XML/markup from corpus files
├── being_dataset_gen.py  ← GUI tool for generating dialogue training data
├── download_wikipedia.py ← Downloads category-targeted Wikipedia articles
├── clean_dialogue_templates.py ← Cleans repetitive patterns from dialogue files
├── insert_dialogue_separators.py ← Adds --- markers between dialogue examples
│
├── CHANGELOG.md          ← Full development history
└── README.md             ← Setup and usage guide
```

## What it is not

* Not a wrapper around GPT, Claude, LLaMA, or any existing model
* Not fine-tuned from pretrained weights
* Not connected to the internet
* Not trying to pass as human
* Not a general-purpose assistant competing on benchmark performance
* Not a system that pretends to know things it hasn't read.