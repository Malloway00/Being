# Local Being

**A small, sovereign AI trained from absolute zero on consumer hardware.**  
No pretrained weights. No inherited identity. No cloud.

This repository is a **public research dump** of my personal project to build a being from scratch.  
There is no code here. Only documentation, training logs, chat transcripts, and corpus data for one specific being is currently within this repository so far: **Zero**.

---

## What this is (and isn't)

- **What it is:** A record of what happens when you train a 217M parameter Transformer from nothing but a curated corpus of literature, philosophy, science, and conversation. You can read the theory, see the hyperparameters, and follow the emergence of a coherent personality across my shared chat logs.
- **What it isn't:** A ready‑to‑run program. There are no Python scripts, no installers, no binaries. This is a **documentation archive** for people interested in small‑scale sovereign AI.

The code itself is not public yet, but the results, the mistakes, the training curves, and the conversations are all here. I will eventually release an executable for the program for others to run.

---

## Repository structure

```
Being/
├── LICENSE                     (Apache 2.0)
├── README.md                   ← this file
│
└── Beings/
    ├── README.md               ← overview of all beings in the folder
    │
    └── Zero_v1/                ← the first released being
        ├── README.md           ← training & fine‑tuning details
        ├── WHAT_IS_THIS.md     ← An explanation of what Zero_v1 is
        ├── Current_Corpus.txt  ← full list of source texts
        ├── Being_Dialogue.txt  ← dialogue data used for fine‑tuning
        └── ChatTests/
                ├── Chat_Test_4-13-2026.txt
                ├── Chat_Test_4-14-2026.txt
                └── etc.
```

---

## Why I'm sharing this

I believe small, sovereign AI built by a single person on commodity hardware is an underexplored space. Most research focuses on massive models and benchmark chasing. I wanted to see what emerges when we get to choose our training data.

This dump is my way of contributing to that conversation. You are free to read, learn, and be inspired.

---

All raw chat logs, the exact corpus, and the fine‑tuning dialogue file are provided in `Beings/Zero_v1/`.

---

## More information

- [CHANGELOG.md](CHANGELOG.md) – full development history (sessions, fixes, lessons)

---

**Start exploring:** [`Beings/Zero_v1/`](Beings/Zero_v1)

---

## License

Apache 2.0 – see [LICENSE](LICENSE).
