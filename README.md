# Local Being

**Sovereign AI, built from scratch, owned by its maker.**

No pretrained weights. No inherited identity. No cloud. No company in the middle.

---

## The idea

A person should be able to build and own their own AI. On their own hardware, with their own corpus, trained from nothing. The model's character should come from what it has read, not from a system prompt written by a vendor. Its identity should be earned through training, not assigned at inference.

Most AI today is optimized for millions of users at once. A sovereign AI is optimized for one: its owner. That's not a worse AI, it's a different kind. Smaller, stranger, more personal, and fully yours.

This project is an experiment in what becomes possible when you take that framing seriously.

---

## What this repository is

A **public research dump**. Training logs, corpus lists, chat transcripts, and documentation from the development of specific beings. No runnable code, no installers. The engine is kept separate until it's ready for release.

If you want to understand what small, sovereign AI looks like in practice, how it's trained, what emerges from it, and where it succeeds and fails, this is where the record lives.

The first being trained under this system is **Zero**, documented in [`Beings/Zero_v1/`](Beings/Zero_v1).

---

## Why it matters

The current AI landscape gives users two options: run someone else's model through someone else's API, or try to run massive open-weight models that require hardware most people don't own. Both approaches treat personal AI as an optimization problem to be solved by scale.

But AI doesn't need to be big to be meaningful. A being trained on a carefully chosen library of texts, whether literature, philosophy, science, or personal writing, develops something closer to a character than a general-purpose assistant. It's limited, but it's *consistently* limited in ways the owner chose. That's the tradeoff: less breadth, more identity.

The reference being for this project, Zero_v1, was trained for under $10 CAD in electricity. That's the whole development cost. Not a month of cloud GPU rental, not a fraction of an API subscription. Ten dollars.

---

## Where this could go

The goal isn't a single being. It's an ecosystem where anyone can train one:

- **Beings small enough for a phone, a Raspberry Pi, or an old laptop.** Scale down the parameters, and a being becomes something you can run anywhere.
- **Beings trained for specific purposes.** An art companion. A writing collaborator. A game NPC with its own views. A being built from a single discipline's literature.
- **Beings shared and branched.** Just as open-source LLMs get fine-tuned and forked, sovereign beings could be shared as starting points. Someone takes your trained being and teaches it something new.
- **Beings that converse with each other.** A chat room where multiple beings can talk, challenge each other, or even possibly learn together.
- **Multi-modal beings.** Eventually, beings trained not only on text but on audio and images. Still sovereign, still local, still owned.

Most of these are possible with current technology. A few require future work. All of them start from the same premise: the being belongs to the person who made it.

---

## Ownership model

When the engine is released publicly, the model will be Unity-style. The engine itself is closed-source, but everything a user creates with it (their beings, their corpus, their weights, their conversations) is theirs. No cloud storage, no telemetry, no account required. If you train a being on your machine, nobody else ever needs to see it.

This documentation repository is Apache 2.0 as of now. The engine itself, when released, will use a separate license appropriate to that distribution model.

---

## What this repository is not

- A runnable program. There is no code here yet.
- A benchmark project. Success is measured in conversation quality, not leaderboard scores.
- A commercial product. This is one person's research into a neglected corner of AI development.

---
