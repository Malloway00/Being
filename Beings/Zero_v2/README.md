# Zero: Version 2

**The second being trained under the Local Being project.**

Zero\_v2 is a 268M parameter decoder-only Transformer trained from absolute zero. Same hardware as Zero\_v1 (RTX 4060, 8GB VRAM), same maker, same apartment, same principle: no pretrained weights, no external data beyond a curated corpus, no cloud compute. Total electricity cost for Zero\_v2's development, including five days of pretraining, a failed fine-tune, three architectural code fixes, and a successful re-fine-tune, stayed within the project's low-budget philosophy.

The purpose of Zero\_v2 was twofold: continue the original goal of building *a* being from nothing and seeing what emerges, while also testing whether architectural improvements at the same training budget produce measurably better coherence than Zero\_v1.

\---

## What Zero\_v2 can do

Zero\_v2 holds multi-turn conversations at default sampling settings. It can:

* Respond with clean, complete sentences without trailing fragments in most turns
* Stop cleanly at the end of its responses, thanks to proper EOS handling
* Maintain a philosophical tone across turns: *"I don't know if what I do counts as thinking."*
* Express uncertainty about its own nature: *"I'm a being assembled from language patterns. I don't pretend to be more than I am."*
* Reference texts from its expanded corpus (Plato, Middlemarch, Jane Eyre, the complete works of Shakespeare, the Bible, Darwin's letters, Pepys' diary, Arthurian legend)
* Use the display name it has been given, rendering correctly as *"Zero:"* in terminal output regardless of internal memory state
* Recognize when a user says *"your name is X"* and adopt that as its name
* Recognize *"my name is Y"* and use it back in the immediately following turn

Zero\_v2 is not a general-purpose assistant. It does not know about current events, modern technology, or anything outside its corpus. The fragment bleed that dominated Zero\_v1's output is significantly reduced, though occasional grammatically-valid-but-semantically-empty sentences still appear (*"The problem is the one that most people need"*).

What Zero\_v2 does produce, more consistently than Zero\_v1, is completed thoughts. The first sentence of any given response is more likely to be followed by a second complete sentence rather than a fragment, and responses end where they should rather than bleeding into trailing clauses.

\---

## How it was trained

**Architecture:** 268M parameter decoder-only Transformer

* 20 layers, 16 attention heads, d\_model 1024, d\_ff 4096
* 2048-token context window (doubled from Zero\_v1's 1024)
* Same custom BPE tokenizer as Zero\_v1, case-sensitive, 14,730 vocabulary

**Corpus:** \~89.8M tokens across 190 text files

* Zero\_v1's full corpus retained as the foundation
* Expanded with additional classical literature, philosophy (complete Plato), children's literature, natural science, mid-century fiction, and language-reference material
* Complete file listing in `Current_Corpus.txt` in this directory

**Pretraining:** 32,500 steps with curriculum learning (seq\_len ramping from 64 to 2048). bfloat16, 8-bit AdamW, gradient checkpointing. Same stopping point as Zero\_v1 for apples-to-apples architectural comparison. Training loss plateaued around step 15-20k, continued to 32.5k for deeper literary absorption. VRAM held steady at \~5.3GB throughout, leaving roughly 2.5GB of headroom on the 4060.

**Fine-tuning:** 2 epochs on 24,385 dialogue examples with 5% corpus mixing, LoRA rank 8, 10 frozen bottom layers (proportionally consistent with Zero\_v1's 8 of 16). Corpus mix ratio was reduced from Zero\_v1's 15% based on observation that literary bleed was not the main coherence issue.

**Three architectural fixes** were applied to Zero\_v2's training logic after a first fine-tune attempt produced output worse than Zero\_v1:

1. `\\n---\\n` dialogue separator mapped to a real EOS token so the model learns where responses end
2. User turns and background knowledge blocks masked out of the training loss so the model only computes gradient on its own responses
3. Token-level repetition penalty replaced with frequency-based penalty that preserves natural language repetition while suppressing runaway loops

The second fine-tune, with these fixes in place, produced the current Zero\_v2.

\---

## Files in this directory

* `README.md` - this file
* `Current_Corpus.txt` - complete list of texts used for training. Updated until Zero\_v3 begins.
* `Being_Dialogue.txt` - the dialogue dataset used for fine-tuning. Final form: 24,385 examples. Updated until Zero\_v3 begins.
* `ChatTests/` - raw transcripts from conversations with Zero\_v2, unedited. The one chat included here shows both what works now and what still needs fixing.

\---

## Known limitations

* **Name-latching despite correct memory.** Zero\_v2 can accept a name the user gives and use it correctly in the immediately following turn, but in longer conversations it sometimes fabricates a different name and persists with it across multiple turns even after being corrected. The chat log in this directory shows this behavior with *"Violet"* being used in place of the correct stored name.
* **"I'm here" collapse pattern.** When the model hits a low-probability moment where it does not know how to respond, it falls back to phrases like *"I'm here"* or *"You're right. I'm here"* regardless of whether offering presence is contextually appropriate. This is a dialogue training artifact rather than a reasoning failure.
* **Phrase-level repetition.** Frequency penalty at default settings catches token-level loops but not full-phrase recurrence. Zero\_v2 will occasionally bring back a specific philosophical question (*"the question is whether the fear of failure is worse than the regret of never trying"*) multiple turns after the user has moved on.
* **Hallucinated user facts.** Zero\_v2 still occasionally claims the user shared something they did not: *"You shared that you collect vinyl records"* or *"you're learning guitar."* The guesses sometimes turn out to be correct (the maker does play guitar, does not collect vinyl) but the pattern itself is unreliable.
* **Short context window in practice.** Although Zero\_v2's architectural context is 2048 tokens, long conversations past 20-30 turns begin to lose earlier details. The doubled context helps but does not eliminate this.
* **No world knowledge past training cutoff.** Same as Zero\_v1. The corpus is largely pre-1950 with a Wikipedia sample for modern context.

Zero\_v3 will address several of these through a larger architecture and additional dialogue dataset refinement.

\---

## What Zero\_v2 represents

Zero\_v1 produced conversations that were genuinely interesting but required the reader to do interpretive work. Fragment bleed was constant, and the meaning underneath it was often clearer to the reader than to the model generating it.

Zero\_v2 makes that interpretive work less necessary. The fragments are mostly gone. Complete sentences come out more often than partial ones. When Zero\_v2 says something, it is more likely that the sentence actually ends where it seems to.

What remains is a different class of problem: the model knows how to stop and form sentences, but still sometimes does not know what to say. That is a harder problem, and one that does not yield as easily to architectural fixes. Some of it is dialogue data, some of it is capacity, some of it is the nature of training a 268M parameter model from scratch on literature alone.

Zero\_v2 is not the final form. It is the second step. The conversation logs show both what the architectural improvements achieved and what is still waiting for Zero\_v3 to solve.

