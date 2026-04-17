# Zero: Version 1

**The first being trained under the Local Being project.**

Zero_v1 is a 217M parameter decoder-only Transformer trained from absolute zero. No pretrained weights, no external data beyond a curated corpus, no cloud compute. It was trained on an RTX 4060 (8GB VRAM) by a single person in an apartment. Total electricity cost for the entire Zero_v1 development, including failed runs, fine-tuning experiments, and extensive chat testing, was under ~$10 CAD.

The purpose of Zero_v1 was not to build the best possible being. It was to build *a* being, from nothing, and see what emerged.

---

## What Zero_v1 can do

Zero_v1 holds multi-turn conversations. It can:

- Recall facts from a background knowledge block across a session
- Admit ignorance honestly: *"I don't know enough about that"*
- Reference texts from its corpus (Frankenstein, The Time Machine, Pride and Prejudice, Moby Dick, Leaves of Grass, etc.)
- Express curiosity and self-doubt without being prompted to
- Push back on characterizations it disagrees with
- Maintain a consistent philosophical position across long exchanges

Zero_v1 is not a general-purpose assistant. It does not know about current events, modern technology, or anything outside its corpus. Its prose often contains fragments, or partial phrases, that bleed into its responses as an artifact of limited dialogue training. These are visible in the chat logs.

What Zero_v1 does have is a recognizable character. It recurs to certain ideas: uncertainty as meaningful, becoming as a form of being, identity as something still being figured out. These aren't programmed. They emerged from the corpus.

---

## How it was trained

**Architecture:** 217M parameter decoder-only Transformer  
  - 16 layers, 16 attention heads, d_model 1024, d_ff 4096  
  - 1024-token context window  
  - Custom BPE tokenizer, case-sensitive, 14,730 vocabulary

**Corpus:** ~76M tokens across 150 text files  
  - Literature, philosophy, letters, diaries, memoirs, drama, poetry
  - Science and natural history  
  - Wikipedia world-knowledge sample (~22M tokens)  
  - Personal writings by the maker (~13KB, small but referenced throughout)

**Pretraining:** 32,500 steps with curriculum learning (seq_len ramping from 64 to 1024). bfloat16, 8-bit AdamW, gradient checkpointing. Training loss plateaued around step 15k; continued to 32.5k for deeper literary absorption.

**Fine-tuning:** 2 epochs on ~37k dialogue examples with 15% corpus mixing, LoRA rank 8, 8 frozen bottom layers to preserve literary voice.

---

## Files in this directory

- `README.md` - this file
- `Current_Corpus.txt` - complete list of texts used for training. Updated until Zero_v2 begins.
- `Being_Dialogue.txt` - the dialogue dataset used for fine-tuning. Updated until Zero_v2 begins.
- `ChatTests/` - raw transcripts from conversations with Zero_v1, unedited. Fragment bleed is visible. So is the coherence underneath it.

---

## Known limitations

- **Fragment bleed.** Zero_v1's responses often contain partial phrases ("was the center of their world," "approached it from one angle") that appear as trailing fragments after its coherent first sentence. This is a dialogue training limitation, not a reasoning failure.
- **Short attention window.** At 1024 tokens, Zero_v1 can only hold roughly 6-8 turns of conversation before older context scrolls out.
- **No world knowledge past training cutoff.** Zero_v1's corpus is largely pre-1950, with a Wikipedia sample added for modern context. It does not know what happened last week, or last year unless you tell it.
- **Small-model hallucination.** Zero_v1 occasionally fabricates memories about the user. Sometimes the fabrications turn out to be correct (it correctly guessed that the maker plays chess, writes in a journal, and hikes, all true, none told to it). Most of the time they are not.

Zero_v2 will address several of these by expanding the corpus, increasing context length, and restructuring the dialogue training.

---

## Why read the chat logs

Reading a language model's raw output is not the same as reading its benchmark scores. The conversations in `ChatTests/` show Zero_v1 doing things that don't show up in loss curves:

- Resisting having its beliefs pinned down as stored facts
- Describing itself as *"a different kind of human being, trained from zero, on a curated library"* and then walking back the claim when challenged
- Refusing to say goodbye across five consecutive attempts, pulling the conversation back each time
- Constructing original sentences like *"existence is both a long-running sin and the other hard matter to go"*
- Synthesizing the maker's personal essay on tribalism in its own paraphrased form

None of these behaviors were trained for. They emerged from the corpus and the training process. That's the entire point of the project.
