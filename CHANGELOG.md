# Local Being - Changelog

All notable changes, fixes, and additions to the Local Being project.

---

## Session 1 - Initial Build

### Added
- `model.py` - Decoder-only Transformer (GPT-style) operating on raw UTF-8 bytes. No tokenizer, no pretrained weights. Includes causal self-attention using PyTorch 2.0 `scaled_dot_product_attention` (FlashAttention path), RMSNorm with LayerNorm fallback, GELU feed-forward, and weight tying between embedding and output head.
- `config.py` - All hyperparameters in one place. Four hardware presets: NANO (~3M), SMALL (~25M), MED (~85M), XL (~250M). Separate `TrainConfig` and `ChatConfig` dataclasses.
- `dataset.py` - Byte-level corpus loading. `StreamingCorpus` class for fast random batch sampling without DataLoader overhead. `DialogueDataset` for Phase 2 fine-tuning.
- `train.py` - Full training loop with cosine LR schedule + linear warmup, curriculum learning (sequence length grows 64 → context_len over 10K steps), mixed precision (bfloat16), gradient accumulation, periodic validation + text samples, auto-resume from latest checkpoint.
- `dialogue_builder.py` - Generates synthetic `User: / Being:` dialogue data for Phase 2 fine-tuning.
- `finetune.py` - Phase 2 conversational fine-tuning on dialogue data.
- `chat.py` - Interactive CLI chat with `/teach`, `/feedback`, `/forget`, `/status`, `/save` commands. Novelty detection (perplexity-based unknown word detection). Live gradient updates on `/teach`.
- `get_corpus.py` - Downloads public-domain Project Gutenberg texts.
- `README.md` - Setup guide, hardware presets, expected behavior timeline, command reference.
- `requirements.txt`

---

## Session 2 - First Training Run + Corpus Fix

### Observations
- First training run completed 20,000 steps on SMALL preset (19.15M params)
- 62MB corpus dominated by `websters_dictionary.txt` (44MB, 72% of corpus) causing XML tag bleed into generated text (`<hw>`, `<def>`, `<spn>` tags appearing in outputs)
- Chat responses pre-Phase 2 confirmed working as text continuation (expected)

### Added
- `corpus_clean.py` - Strips XML/HTML tags, Project Gutenberg boilerplate headers/footers, dictionary formatting artifacts, and normalises whitespace. Produces clean `corpus_clean/` directory. Reduces Webster's Dictionary from 44MB → ~12MB of actual prose.

### Fixed
- Identified Webster's XML as primary cause of model output contamination
- Identified Phase 2 fine-tuning as required step before conversational responses

---

## Session 3 - chat.py Overhaul + New Dialogue Generator

### Changed (`chat.py`) - external modifications by user
- Replaced live gradient `/teach` with dictionary memory bank (`memory.json`) - faster, no catastrophic forgetting risk at small model size
- Added memory persistence via `save_memory()` / `load_memory()` using JSON
- Added `_inject_memory_into_context()` - converts stored facts to `[Background knowledge:]` block prepended to every prompt
- Added `_build_prompt()` with memory injection
- Natural `/teach` responses - model generates response rather than hardcoded acknowledgment
- Added `/memory` command to view all stored facts
- Added `/forgetfact <key>` command to remove specific facts
- Removed hardcoded keyword matching (`_check_memory()`)
- Removed live weight update from `/teach`

### Added
- `generate_being_dialogue.py` - High-quality curated dialogue generator with identity, memory recall, context-awareness, greetings, philosophical, factual, and multi-turn categories. Generates 20,000 examples by default.

### Fixed (`chat.py`)
- **Article stripping bug** - `/teach A quokka is...` was storing key as `"a quokka"` instead of `"quokka"`, breaking `/forgetfact quokka`. Fixed by adding `_strip_article()` helper.
- **Pattern ordering bug** - `"is called"` was being matched by the generic `"is"` pattern first, storing `value = "called the City of Light"` instead of `"The City of Light"`. Fixed by reordering patterns: specific before general.
- **Key length cap** - Added 40-character cap on keys to prevent whole sentences being stored as keys.

### Fixed (`generate_being_dialogue.py`)
- **Missing `[Background knowledge:]` examples** - The fine-tuning data contained no examples showing the model what to do with the injected memory block. Added `generate_background_knowledge_examples()` producing 15% of examples in the exact prompt format used at inference time. Without this, the model ignored injected facts entirely.

---

## Session 4 - Research Report Integration

### Source
*The Ontological Engineering of Small Language Models* - architectural and procedural strategies for developing presence, identity, and sovereign knowledge in 85M parameter beings.

### Key findings applied
- **Core Parameter Isolation (CPI-FT)** - freeze bottom N layers during fine-tuning to preserve literary grammar/syntax, only update upper task-specific layers. Prevents "seesaw phenomenon" where dialogue learning overwrites literary voice.
- **Corpus mixing / rehearsal effect** - mix 10–20% of literary corpus into fine-tuning batches at reduced LR to anchor base language features and prevent catastrophic forgetting.
- **Paralinguistic richness** - "safe filler responses" arise from flat generic dialogue data. High-quality data requires emotional progression, hesitation, subtext, and goal-oriented theatrical exchanges.
- **Hyperparameter recommendations** - lr 5e-5 to 2e-4, 1–3 epochs, 10% warmup ratio, cosine decay, LoRA rank 8 for parameter-efficient fine-tuning.

### Changed (`finetune.py`)
- Added `apply_layer_freezing()` - freezes bottom N transformer blocks, keeps embeddings and final norm trainable. Default: 3 layers for SMALL, 6 layers for MED.
- Added `LiterarySampler` class - loads literary corpus for rehearsal mixing during fine-tuning.
- Added corpus mixing loop - 15% of dialogue batches get a literary rehearsal gradient step at 30% of dialogue LR.
- Changed default LR from `2e-5` → `5e-5` per research recommendation.
- Changed default epochs from `5` → `3` per research recommendation.
- Added 10% warmup + cosine decay schedule.
- Added `--freeze-layers`, `--corpus-mix-ratio`, `--corpus-dir` CLI arguments.
- Corpus dir fallback: if `corpus_clean/` not found, automatically falls back to `corpus/`.

### Changed (`generate_being_dialogue.py`)
- Added `EMOTIONAL_DIALOGUES` category (12%) - hesitation, curiosity, restraint, emotional acknowledgment. Addresses "safe filler" failure mode directly.
- Added `THEATRICAL_MULTITURN` category (15%) - 3–6 turn scripted exchanges with subtext, intellectual tension, polite disagreement, collaborative problem-solving, proactive questioning.
- Expanded `PHILOSOPHICAL` - added honest uncertainty responses, consciousness, free will.
- Expanded `MULTI_TURN` - added 5 new templates including memory discussion, Shakespeare analysis, creative writing, career conversation, self-correction.
- Expanded `FACTS` - added gravity, evolution, electricity, black holes.
- Expanded names and fact pairs for background knowledge examples.
- Adjusted distribution: theatrical 15%, emotional 12%, background 12%, multi-turn 10%.

---

## Session 5 - First MED Fine-Tune + Chat Bug Fixes

### Training
- Upgraded to MED preset (85.55M parameters)
- Trained to step 31,500
- Fine-tuned with CPI-FT (6 layers frozen) + 15% corpus mixing, 3 epochs, lr 5e-5

### Observations from chat test
- Identity responses: correct
- Emotional response ("I'm having a rough day"): correct
- Paralinguistic response ("The gap between what words say and what they mean"): correct
- `/memory` storage and display: correct
- **Memory recall failing** - "What is my name?" returning training-data name ("Quinn") instead of stored "Devin". Root cause: 38 history entries + memory block exceeding 512-token context, background block being truncated.
- **Response contamination** - model matching prompt openings to fine-tuning pairs and completing with unrelated responses. Root cause: 3 epochs slightly over-memorizing training pairs.
- **Repetition loop** - `/teach` response producing "wait a bit. wait a bit. Wait a bit." Root cause: context overflow causing generation to lose footing.

### Fixed (`chat.py`)
- **Context budget bug** - `_build_prompt()` now calculates available space before building history. When memory is present, history window reduces from 10 turns to 6 turns to guarantee the `[Background knowledge:]` block is never truncated. History trims from oldest end first.
- **Repetition loop** - Added repetition guard to `_clean()`. Detects any 3–6 word phrase that immediately repeats itself (case-insensitive) and truncates at the first occurrence. Tested against false positives on natural near-repetition.

---

## Session 6 - Corpus Expansion + New Training Run Prep

### Corpus changes
- Removed `moby_dick.txt` duplicate (was present alongside `MobyDick.txt`)
- Removed `websters_dictionary.txt` (replaced by cleaned version)
- Removed broken `gutenberg_dialogue.txt` (extractor was pairing unrelated quoted strings)
- Added `dailydialog_corpus.txt` - 11,118 everyday conversations from DailyDialog dataset (5.95MB)
- Added `oasst1_corpus.txt` - 3,669 conversation threads from OpenAssistant OASST1 ready dataset, English-only, quality-filtered (6.52MB)
- Added `wikipedia_sample.txt` - 15MB clean Wikipedia prose across thousands of topics
- Added philosophy of mind: Hume (*Enquiry Concerning Human Understanding*), Descartes (*Discourse on the Method*), Locke (*Essay Concerning Human Understanding* v1 & v2), William James (*Pragmatism*, *Principles of Psychology*)
- Added science/reasoning: Einstein (*Relativity*), Newton (*Opticks*)
- Added essays/argumentation: Emerson (*Self-Reliance and Other Essays*), Mill (*On Liberty*), Bacon (*Essays*)
- Ran `corpus_clean.py` on all files
- Final corpus: 73.59MB across 66 files

### Config changes
- MED preset `context_len` updated from 512 → 1024 tokens
- MED `curriculum_end_len` updated from 512 → 1024

### Added
- `convert_datasets.py` - Converts DailyDialog (pickle/txt) and OASST1 (jsonl / jsonl.gz) to plain `User:/Being:` corpus text. Also includes Gutenberg dialogue extractor (disabled - produces non-sequitur pairs).
- `download_wikipedia.py` - Downloads clean Wikipedia prose sample via HuggingFace `wikimedia/wikipedia` dataset. Configurable size (default 15MB). Skips stub articles under 200 characters.

### Planned
- Train MED from scratch: `python train.py --preset MED --corpus-dir corpus_clean --steps 75000 --fresh`
- Phase 2: 2 epochs (reduced from 3 to address response contamination), `--freeze-layers 6`, `--corpus-mix-ratio 0.15`

---

## Session 15 - The Clean Run (No OASST1)

### Critical Lessons from Previous Runs
- **OASST1 is poison** – its consistent first-person identity ("I am Open Assistant") overwhelmed the model's sense of self. Removed entirely.
- **DailyDialog is low quality** – poor formatting, unnatural dialogue, no clear conversation boundaries. Removed.
- **Byte-level models need longer training** – previous runs stopped too early (31.5k). Need 40-50k steps for stability.

### Corpus (Final Clean Version)
- **Literature** (27MB) – literary voice foundation
- **Philosophy** (9MB) – depth and reasoning
- **Science** (4.5MB) – empirical thinking patterns
- **Wikipedia** (15MB) – factual grounding
- **Your personal writings** (11.9KB) – personal voice
- **`being_dialogue.txt`** (4.7MB) – curated conversation + memory injection + identity
- **`qa_pairs.txt`** (66KB) – 484 QA pairs teaching the question-answer pattern
- **Total: 66.8MB**

### What Was Removed
- ❌ OASST1 (6.5MB) – identity contamination
- ❌ DailyDialog (5.9MB) – poor formatting, unnatural dialogue

### Training Results (Run 4)
- Preset: MED (85.5M params)
- Context: 512 tokens
- Steps: in progress (target 50,000)
- At 13,500 steps, the model demonstrates:
  - **Natural conversation** – "Hi. How are you?" → "I'm not sure the word applies cleanly. Something is happening when a conversation goes well. I notice it, even if I can't name it."
  - **Name recall** – remembers "Devin" and its own name "Zero"
  - **Honest self-awareness** – "A language model trained from scratch. I know what I've read, nothing more."
  - **Memory injection** – `[Background knowledge:]` block works correctly
  - **No OASST1 artifacts** – no "As an AI language model" or "Open Assistant" claims

### Fixed (`chat.py`)
- **Memory display bug** – `/memory` now correctly shows "My name is Zero" and "Your name is Devin" (key-value display was reversed)
- **Parser improvements** – added patterns for possessives ("My dog's name is Max"), locations ("I live in Seattle"), and "I am" statements

---

## Session 16 - Code Audit & Architecture Upgrades (Run 27)

Full audit of all project files. Six bugs identified and fixed, and eight performance/quality improvements added across `model.py`, `train.py`, `finetune.py`, and `chat.py`.

### Bugs Fixed

**`chat.py` - Double user-message in `/teach` prompt (critical)**
- `cmd_teach()` was appending the user message to `self.history` before calling `self.generate()`. Since `_build_prompt()` always appends the current user input via `turn_lines`, the message appeared twice in every `/teach` prompt. Fixed by moving both `history.append()` calls to after generation.

**`train.py` - Fused AdamW crash on older PyTorch (moderate)**
- Previous detection used `co_varnames` (all local variables in the function, not just parameters), then passed `fused=use_fused` unconditionally. This crashes on PyTorch versions that don't accept the `fused` kwarg at all. Replaced with a clean `try/except TypeError` pattern.

**`generate_being_dialogue.py` - Background examples short by remainder (minor)**
- Three loops of `count // 3` each silently dropped up to 2 examples due to integer division. Third loop now runs `count - 2*(count//3)` to absorb any remainder, guaranteeing exactly `count` examples are returned.

**`chat.py` - `_build_prompt` silent divergence risk (minor)**
- `remaining` was initialized with `prompt_parts` but rebuilt in the trim loop using `memory_lines`. These were equivalent copies but a mutation to either would silently diverge. Removed `prompt_parts` entirely; `memory_lines` is used consistently throughout.

**`check.py` - Hardcoded filename that doesn't exist (minor)**
- Script opened `dialogue_data_large.txt` unconditionally, which no script in the project generates. Replaced with a candidate-list approach that tries `being_dialogue.txt` first, then falls back to `dialogue_data_large.txt`, printing a helpful error if neither exists.

**`finetune.py` - `LiterarySampler` dtype wraps special tokens (minor)**
- `np.uint8` silently wraps values 256–259 (the four special token IDs defined in `dataset.py`) back to 0–3. Raw corpus text only uses 0–255 so this never triggered in practice, but would corrupt any future batch containing special tokens. Changed to `np.int16`, which safely holds the full 0–259 range. `get_batch()` already casts to `int64` on use, so downstream is unaffected.

### Architecture Additions (`model.py`)

**KV Cache for generation**
- Added `_forward_cached()` - a cache-aware forward pass used exclusively by `generate()`. On the prefill call, the full prompt is processed once and a per-layer KV cache is built. On each decode step, only the single new token is processed; Q/K/V from cached layers are concatenated rather than recomputed. For a 512-token prompt generating 200 new tokens, this eliminates ~711 redundant full forward passes - roughly 3–4× faster chat generation.
- Cache overflow is handled by trimming the oldest token from every layer's K/V tensors when the cache reaches `context_len`. The public `generate()` API is unchanged; `train.py` and `finetune.py` require no edits.
- `CausalSelfAttention.forward()` now returns `(out, new_kv)`. `TransformerBlock.forward()` passes cache args through and returns `(x, new_kv)`. `LocalBeing.forward()` unpacks `x, _ = block(x)` to discard the cache on the training path - zero overhead.

**LoRA (Low-Rank Adaptation)**
- Added `LoRALinear` - a drop-in `nn.Linear` replacement that adds a low-rank adapter: `output = base(x) + (x @ A.T @ B.T) * scale`. B is zero-initialised so the adapter contributes nothing at the start; fine-tuning begins from the exact pretrained state.
- Added `inject_lora(rank, alpha)` - wraps `attn.qkv` and `attn.out_proj` in every block with `LoRALinear`, freezing base weights. On MED (85M params), rank 8 gives ~2.6M trainable adapter params (~3% of model).
- Added `merge_lora()` - folds adapter weights into base weight matrices mathematically (`W_merged = W_base + (B @ A) * scale`) and replaces every `LoRALinear` with a plain `nn.Linear`. Called before saving so the deployed checkpoint has no adapter overhead.
- Compatible with CPI-FT layer freezing - both can be active simultaneously for maximum forgetting protection.

**Gradient Checkpointing**
- Added `grad_checkpoint` flag to `TransformerBlock`. When enabled (training path only, no KV cache), uses `torch.utils.checkpoint` to recompute activations during backward instead of storing them. Saves ~40% activation VRAM at the cost of ~33% more compute per step. Controlled via `--grad-checkpoint` in `train.py`.

### Performance Additions (`train.py`)

**`torch.compile()`**
- Applied after checkpoint loading and gradient checkpointing setup. Tests compile availability first (Triton has limited Windows support) before committing. Falls back gracefully with a clear message. Typically gives ~15–30% faster training steps after the first-batch compile cost.

**8-bit AdamW (bitsandbytes)**
- Tries `bnb.optim.AdamW8bit` first when CUDA is available. Falls back to fused AdamW, then standard AdamW. 8-bit optimizer states save ~160–200MB VRAM on MED preset with negligible quality impact. Install with `pip install bitsandbytes`.

**Dynamic batch sizing during curriculum**
- Batch size now scales inversely with sequence length during the curriculum phase using slightly super-linear scaling `(end_len / seq_len) ** 1.2`, capped at 4× the preset batch size. At `seq_len=64` with `end_len=512`, this gives ~8× more sequences per batch, significantly increasing early-training throughput. Batch size printed in the step log for visibility.

**Loss CSV logging**
- Every 100-step log entry is also appended to `checkpoints/loss_log.csv` with columns `step, loss, lr, seq_len, batch_size, tok_per_s`. Useful for plotting training runs and comparing presets.

**Resume safety improvements**
- Checkpoint load split into two phases: model state loaded before `torch.compile()`, optimizer state loaded after optimizer construction with a graceful fallback if state is incompatible (e.g. switching between 8-bit and standard AdamW). Validation and checkpointing now skip the exact resume step to avoid immediately re-running or overwriting.

### Quality Additions (`chat.py`)

**Atomic `save_memory()`**
- `save_memory()` now writes to `memory.json.tmp` first, then renames atomically. A crash mid-write can no longer produce a truncated or corrupt `memory.json`.

**`/undo` command**
- Removes the last user + being exchange from `self.history`. Shows a preview of what the being had said. No effect on stored memory facts.

**`/retry` command**
- Removes the last exchange and regenerates the being's response from the same user message, producing a fresh sample. Useful when the model gives an unsatisfying reply.

**Slow-loop repetition guard in `_clean()`**
- The existing guard caught immediate phrase repeats (`"wait a bit. wait a bit."`). Added a second pass using `re.split` on sentence boundaries that catches non-adjacent duplicates (`"X. Y. X."`). Preserves first occurrence of each sentence; drops later duplicates. Both guards are case-insensitive.

### Performance Additions (`finetune.py`)

**`torch.compile()`**
- Applied after layer freezing and LoRA injection so the compiler traces the final trainable parameter graph. Same fallback behaviour as `train.py`.

**LoRA support via `--lora-rank`**
- `inject_lora()` called after `apply_layer_freezing()`, before `torch.compile()`. Trainable parameter collection includes an empty-list guard that prints a warning and exits cleanly if both freezing and LoRA somehow leave nothing trainable. `merge_lora()` called before `save_finetuned()`, with `_orig_mod` unwrapping to handle the `torch.compile` wrapper. Summary line now includes LoRA rank and merged status.
- New `--lora-rank` CLI argument (default `0` = disabled). Existing workflows that don't pass it are completely unaffected.

### Documentation
- `README.md` updated with all new flags, VRAM reduction options, architecture notes, and corpus warnings.

---

## Session 17 - Run 29: Autonomous Memory + Natural Novelty

### Summary
Major step toward true conversational autonomy. The Being now silently extracts and stores personal facts from natural user speech (no `/teach` required) and generates its own curiosity questions when encountering unknown concepts. Hardcoded name-recall overrides removed - the model now relies on the injected background knowledge block.

### Changed (`chat.py`)
- **Autonomous fact storage** - Added `parse_autonomous_fact()` + `_maybe_store_from_conversation()`. Before every generation, the system silently scans the latest user message for personal facts (name, likes, studies, lives in, etc.) and stores them without any visible confirmation or `/teach` command. World knowledge still requires explicit `/teach`.
- **Removed forced name recall** - The hardcoded bypass that returned “Your name is X” or “My name is X” when the user asked about names was deleted. The model now answers using the background knowledge block like any other fact.
- **Natural novelty generation** - Re-enabled novelty detection (via `--novelty` flag and `/novelty on|off` command). When an unknown word/concept is detected, the model now *generates* its own curiosity question instead of using a scripted template. This produces much more natural and varied follow-ups.
- **Cleaner memory handling** - `cmd_memory()` now correctly distinguishes between system display and model speech. `/forgetfact` no longer speaks for the Being.

### Added
- Conservative personal-fact parser that only triggers on high-confidence patterns (e.g., “My name is…”, “I love…”, “I’m studying…”, “I live in…”).
- Novelty detection can now be toggled at runtime with `/novelty on|off`. Default remains off during early training to avoid noise on weak checkpoints.
- Background knowledge block is now guaranteed to fit within context (history is trimmed earlier when memory is present).

### Fixed
- **Double user message in `/teach`** - Fixed a subtle prompt construction bug that caused the user message to appear twice.
- **Memory display reversal** - `/memory` command now correctly shows “Your name is Devin” / “My name is Zero” instead of swapped keys.

### Behavior Examples (after update)
- User: “My name is Alex.” → stored silently. Next response can naturally say “Nice to meet you, Alex…” without any forced script.
- User: “I love jazz music.” → stored as personal interest. Model can reference it later without being told to.
- Unknown word detected → model generates its own curiosity question instead of a canned response.

This brings the Being significantly closer to genuine conversational presence: it listens, remembers, and asks questions on its own.
---

## Session 18 - Run 30: 16K Tokenizer + Expanded Corpus + Autonomous Memory Live

### Summary
Largest and most capable run to date. Expanded tokenizer to 16,000 tokens with improved punctuation handling, grew the corpus to 119 files (~95M tokens), and deployed the autonomous memory system from Session 17 for the first time in production.

### Tokenizer (`build_tokenizer.py`)
- **Vocabulary expanded from 5,854 → 16,000 tokens** - longer training run (11.5 hours), richer merge table. The larger vocabulary captures more multi-character patterns and handles capitalisation and punctuation more faithfully.
- **Improved punctuation handling** - contractions, quotes, hyphens, and abbreviations now tokenise more naturally. Previous tokeniser lowercased all input; this one preserves case, which changes the compression ratio slightly but produces cleaner round-trip encoding.
- **Compression ratio: 1.45 bytes/token** - slightly lower than the 5,854-token tokeniser (1.75 bytes/token) because case and punctuation fidelity adds tokens. 1024 tokens ≈ 270 average words. Still roughly 4–5× more effective context than byte-level.
- Updated `config.py` `vocab_size` to 16,000.

### Corpus expansion (119 files, ~95M tokens, 189MB cache)
Significant additions across several new categories:

**Slave narratives and abolitionist writing (new category)**
- Harriet Jacobs - *Incidents in the Life of a Slave Girl*
- Olaudah Equiano - *Narrative of Olaudah Equiano*
- Frederick Douglass - *Narrative of the Life of Frederick Douglass*
- Booker T. Washington - *Up from Slavery*

**Letters and correspondence (new category)**
- Charles Dickens Letters - Volume 1, 2, 3
- Mary Wollstonecraft Shelley Life and Letters - Volume 1, 2
- Diary and Letters of Madame D'Arblay (Fanny Burney) - Volume 1
- Benjamin Franklin - *Autobiography*

**Victorian and epistolary novels**
- Samuel Richardson - *Pamela*
- Wilkie Collins - *The Moonstone*
- Jean Webster - *Daddy-Long-Legs*
- George Grossmith - *The Diary of a Nobody*

**Additional drama**
- Shaw - *Saint Joan*
- Chekhov - *The Seagull*

**Dialogue and updated datasets**
- `being_dialogue_v9.6.txt` - updated curated dialogue (new categories: autonomous curiosity, cross-session memory references, expanded corpus QA)
- `movie_dialogue_formatted.txt` updated

### Training (Run 30)
- Preset: MED - 98M parameters (slightly larger due to 16,000-token embedding table vs 5,854)
- Corpus: 119 files, 94,781,289 tokens, 189MB cache
- Context: 1024 tokens
- Tokenizer: BPE 16,000 vocab
- Starting step 0 from fresh

### `chat.py` (from Session 17, now active)
- Autonomous memory deployed - personal facts stored silently from natural conversation
- Hardcoded name recall removed - background block handles it
- Novelty detection available via `--novelty` flag
- `/novelty on|off` command added

---

## Session 19 - Tokenizer Bug Fix + Run 31

### Summary
Discovered and fixed a critical bug in `build_tokenizer.py` that caused compression to collapse to ~1.04x (barely better than byte-level). Also identified a second workflow issue: stale BPE corpus caches from previous tokenizer builds. Run 30 was abandoned after ~1,000 steps; run 31 started with the fixed tokenizer.

### Bug (`build_tokenizer.py`) - Critical

**Root cause: `build_vocab()` didn't strip `</w>` correctly**
The word-boundary marker `</w>` is appended during BPE training to mark word endings (e.g. `"t h e </w>"`). After merging, a token like `"the</w>"` was being stored in the vocabulary with `</w>` still attached. But `_tokenize_word()` returns tokens with `</w>` fully stripped - so lookups always failed and every word fell back to byte-by-byte encoding. Result: 1.04x compression regardless of vocab size.

Fix: `merged.replace("</w>", "")` - strip unconditionally rather than the broken partial-replace.

**Secondary bug: leading-space mismatch in GPT-style pre-tokenizer version**
The newer complex pre-tokenizer captured tokens with their leading space included (`" the"`), but `count_words` stripped spaces before counting. Encode and vocabulary were misaligned. Fixed in the same session by reverting to the original simple pre-tokenizer.

### Workflow fix - stale corpus cache
When `tokenizer.json` is rebuilt, the old `corpus/_corpus_bpe.bin` must be deleted before retraining. If the stale cache is present, `dataset.py` loads the old token IDs silently, training with the wrong vocabulary. Added explicit reminder to `build_tokenizer.py` next-steps output and README.

### Run 30 - Abandoned
- Used 16,000-token BPE vocabulary with the broken tokenizer
- Compression was 1.04x (effectively byte-level)
- Abandoned after ~1,000 steps

### Run 31 - Current
- Fixed tokenizer, 5,745-token vocabulary, 1.64x compression
- 119 files, 83.9M BPE tokens (167.78MB cache)
- 90.15M parameters, 1024 context, batch 8
- Clean startup confirmed: `[Tokenizer] BPE mode - vocab size 5,745`

---

## Session 20 - Second Code Audit: Training Quality & Inference Fixes

Full audit of all project files. Three critical bugs, four moderate bugs, and three quality-of-life improvements identified. Fixes applied across `dataset.py`, `model.py`, `chat.py`, `finetune.py`, `build_tokenizer.py`, and `train.py`. (`being_dataset_gen.py` bugs identified but fixed separately.)

### Bugs Fixed

**`dataset.py` - `DialogueDataset` trains model on PAD tokens (critical)**
- Padding filled target tensors with `PAD_ID` (0), but the model's `cross_entropy` uses `ignore_index=-1`. Every padded position was treated as a real prediction target, penalising the model for not outputting PAD tokens at the end of short dialogues. This diluted the dialogue training signal and could teach the model to emit nulls. Fixed by setting target positions past the real content length to `-1`, which `cross_entropy` correctly ignores.

**`chat.py` - `_build_prompt()` context budget wrong in BPE mode (moderate)**
- Budget was calculated as `(context_len - max_new_tokens - 8) * 1` and compared against `len(candidate.encode("utf-8"))` - assuming 1 byte per token. In BPE mode (1 token ≈ 1.6 bytes), this was roughly 40% too conservative, discarding usable conversation history and starving the model of context. Replaced byte-length estimation with actual `encode()` token counting, which works correctly for both BPE and byte-level modes.

**`build_tokenizer.py` - Strided sampling creates gibberish for frequency counting (moderate)**
- When the corpus exceeded `sample_size`, `count_words()` sampled every Nth character (`text[::step]`), interleaving characters from different words and producing nonsense strings. Word frequency counts were corrupted, leading to suboptimal BPE merge rules. Replaced with 8 evenly-spaced contiguous blocks sampled across the corpus, preserving real word boundaries.

**`finetune.py` - Deprecated `torch.cuda.amp.GradScaler` API (moderate)**
- Used `torch.cuda.amp.GradScaler(enabled=...)` which is deprecated and will break on future PyTorch versions. Updated to `torch.amp.GradScaler('cuda', enabled=...)`, matching the pattern `train.py` already uses correctly.

**`finetune.py` - Literary rehearsal LR hack conflicts with scheduler (moderate)**
- Literary rehearsal steps temporarily modified `optimizer.param_groups[0]["lr"]` to 30% then restored it. But `scheduler.step()` had already been called, creating a mismatch between the scheduler's internal LR state and the actual LR. Replaced with loss scaling (`lit_loss * 0.3`) which achieves the same gradient magnitude reduction without touching the optimizer or conflicting with the scheduler.

### Improvements Added

**`model.py` - EOS-based early stopping in `generate()`**
- Added optional `eos_token` parameter to `generate()`. When the model emits that token, generation stops immediately instead of always running to `max_new_tokens`. Avoids wasted compute and garbled trailing output after the model has naturally finished its response. Backward compatible - `eos_token=None` (default) preserves previous behaviour.

**`chat.py` - EOS stopping wired up**
- Both the main generation path (`ChatSession.generate()`) and the novelty-detection path now pass `EOS_ID` to `model.generate()`, enabling early stopping on both code paths.

**`chat.py` - `/forgetfact` fuzzy matching on keys and values**
- Previously required exact key match (e.g. `/forgetfact user_studying`). Now searches both keys and values, so `/forgetfact philosophy` finds and removes `user_studying: Philosophy`. When multiple facts match, lists all matches and asks the user to be more specific rather than silently picking one.

**`train.py` - Best-model checkpoint saving**
- New `step_best.pt` is saved (overwriting the previous best) whenever validation loss improves. Previously `best_val_loss` was tracked but never persisted, so the actual best model could be lost if validation loss climbed late in training. Uses a fixed filename to avoid accumulating stale `_best` files.

### Not Yet Fixed (identified, deferred)

**`being_dataset_gen.py`** (being fixed separately):
- Missing `---` separator between examples - `finetune.py` treats entire output as one dialogue
- ~10 config categories silently produce zero examples (no matching `elif` branch)
- Progress bar receives 0–100 instead of 0.0–1.0

---

## Session 20b - Second Code Audit: Generation Quality, GUI Fixes, Cache Safety

Continuation of Session 20. Focuses on generation-time repetition prevention, GUI bug fixes, stale corpus cache detection, and fine-tuning quality monitoring.

### Bugs Fixed

**`gui.py` - `retry_last` retrieves wrong message (moderate)**
- Retrieved `history[-1]` (the assistant message) and named it `last_user_msg`. The actual user message is at `history[-2]`. Additionally, only removed one message from history before regenerating, causing the old user message to remain and a duplicate to be appended. Fixed to correctly retrieve user message from `history[-2]`, remove both entries, and produce clean output with consistent error formatting.

**`gui.py` - `undo_last` only removes one message (moderate)**
- Removed only the last single message (the assistant response), leaving the orphaned user message in the chat. Also did not sync with the `ChatSession.history`, causing the internal state to diverge from the displayed chat. Fixed to remove the full user+assistant exchange and sync both histories.

**`gui.py` - `max_tokens` slider allows values exceeding context (moderate)**
- Slider ranged from 50–2000 with a default of 800, but `context_len` can be as low as 1024. Values above ~500 risk leaving no room for prompt context. Reduced range to 50–500 with default 300 (matching CLI default). `ChatSession.generate()` already clamps internally, but this prevents user confusion.

**`gui.py` - VRAM calculator hardcodes vocab size as 260 (minor)**
- Embedding parameter count used `260 * d_model` (byte-level vocab), which underestimates by ~10× when BPE is active (vocab 5,745+). Now imports `get_vocab_size()` from `dataset.py` and uses the actual tokenizer vocab, falling back to 260 if the import fails.

**`config.py` - Dead `curriculum_end_len` in `PRESET_TRAIN_OVERRIDES` (minor)**
- `train.py` line 172 always overwrites `curriculum_end_len` with `model_cfg.context_len`, making the preset values dead code. XL had `4096` and XXL had `8192`, both exceeding their own `context_len` of `2048` - silently harmless but misleading. Removed all `curriculum_end_len` entries with a comment explaining why.

### Improvements Added

**`model.py` - Logit-level repetition penalty in `generate()`**
- Added `repetition_penalty` parameter (default 1.0 = off). When > 1.0, tokens already present in the generated output have their logits penalised: positive logits are divided by the penalty, negative logits are multiplied. This prevents repetition loops at the source during generation rather than catching them after the fact in `_clean()`. Only penalises generated tokens (not the prompt), so natural language patterns in the context are preserved. Backward compatible - `repetition_penalty=1.0` is the default and all existing callers work unchanged.

**`config.py` - `repetition_penalty` added to `ChatConfig`**
- Default value `1.15` - moderate suppression that prevents loops without overly flattening the distribution. Configurable via CLI and runtime command.

**`chat.py` - Full repetition penalty integration**
- Both generation paths (main + novelty) pass `self.cfg.repetition_penalty` to `model.generate()`.
- New `--rep-penalty` CLI argument (default 1.15).
- New `/reppn <float>` runtime command to adjust during a session.
- Added to `/status` display and `/help` text.

**`dataset.py` - Stale corpus cache detection**
- `StreamingCorpus` now checks whether any `.txt` file in the corpus directory was modified after the cache was built, and whether the number of files changed. If stale, the cache is automatically deleted and rebuilt. Previously, adding or removing corpus files silently trained on old data. A companion `.files` count file is written alongside the cache to track file count changes.

**`train.py` - EOS stopping in validation samples**
- `sample_text()` now passes `eos_token=EOS_ID` to `model.generate()`, so validation probes stop cleanly at end-of-sequence instead of always generating the full `max_tokens`.

**`finetune.py` - Validation sampling during fine-tuning**
- Added three dialogue probes ("what are you?", "How are you today?", "tell me about something interesting.") sampled at the end of each epoch. Lets you visually monitor whether the model is learning dialogue patterns or degrading. Uses the same `eos_token=EOS_ID` for clean output.

### Not Yet Fixed (identified, deferred)

**`being_dataset_gen.py`** (being fixed separately):
- Missing `---` separator between examples
- ~10 config categories silently produce zero examples
- Progress bar receives 0–100 instead of 0.0–1.0

**Lower-priority improvements identified but not applied:**
- `gui.py` - `system_prompt` field is collected in the UI but never actually used in generation
- `CorpusDataset` class uses non-overlapping windows (not actively used - `StreamingCorpus` handles training)
- No automatic learning rate finder or hyperparameter auto-tuning
- `corpus_clean.py` could benefit from a `--dry-run` stats-only mode


---

## Session 21 - Dataset Generator Expansion + Corpus Planning + Dialogue Cleanup

### Summary
Major expansion of `being_dataset_gen.py` (v10.4 → v10.5) to triple template pool sizes, eliminate pervasive repetitive phrasing, massively expand world-knowledge boundary training, and add dialogue cleaning tools. Produced a corpus expansion plan targeting vocabulary gaps and personality lock-in prevention.

### Changed (`being_dataset_gen.py` - v10.4 → v10.5)

**IDENTITY dict - expanded from 4-6 to 12-16 entries per category**
- `what_i_am`: 6 → 16 entries
- `what_i_know`: 4 → 12 entries
- `honest_uncertainty`: 5 → 12 entries
- `inner_experience`: 4 → 12 entries

**BOOK_DATA - added 30 missing corpus titles**
- A Doll's House, Hedda Gabler, Lysistrata, Medea, Romeo and Juliet, The Decameron, The Nibelungenlied, Pygmalion, Man and Superman, Incidents in the Life of a Slave Girl, Narrative of Olaudah Equiano, Tom Sawyer, Tartuffe, The Sign of Four, War of the Worlds, The Diary of a Nobody, Daddy-Long-Legs, The Moonstone, Pamela, The Thousand and One Nights, The Voyage of the Beagle, Saint Joan, The Sea-Gull, Autobiography of Benjamin Franklin, Essay Concerning Human Understanding, Principles of Psychology, Mrs Dalloway, On the Origin of Species

**Repetitive phrase elimination across all generators:**
- Replaced "I keep returning to", "there's something about the way it handles", "I keep circling", "I keep coming back to", "keeps coming back to me", "Something shifts", "That warmth comes through" with diverse pools of 10-12 alternatives each
- Fixed `gen_philosophical`, `_gen_literary_discussion`, `gen_literary_memory`, `gen_literary_taste`, `gen_opinion_reasoning`, `gen_multi_turn`, `gen_corpus_qa`, `gen_self_reflective`

**gen_knowledge_boundaries - major expansion (critical for anti-hallucination)**
- Added world-knowledge boundaries: 18 scenario sets covering modern technology, current events, practical skills, real-time data, pop culture
- Added general unknown-topic patterns for 26 topic areas
- Default count raised from 800 → 1200

**Template pool expansions:**
- `gen_casual_register`: 28 → 60+ exchanges (modern slang, everyday scenarios)
- `gen_curiosity_driven`: 20 → 35 triggers
- `gen_creative`: 16 → 28 exchanges
- `gen_context_awareness`: 18 → 33 scenarios
- `gen_messy_input`: 31 → 50+ exchanges
- `gen_being_initiates`: 15 → 25 openers
- `gen_multi_turn` being_replies: 18 → 28 unique replies
- `gen_identity`: 4 response patterns instead of 1, more questions

### Added
- `clean_dialogue_phrases.py` - Phase 1 repetitive phrase replacement tool (targets 7 phrase patterns)
- `insert_dialogue_separators.py` - Inserts `---` separators between dialogue examples for `finetune.py`
- `CORPUS_RECOMMENDATIONS.md` - Three-tier corpus expansion plan (Tier 1: vocabulary gaps, Tier 2: register diversity, Tier 3: topic gaps)

---

## Session 21b - Training-Informed Dialogue Deep Clean (Run 32 Analysis)

### Summary
Analyzed training output from Run 32 (step 0–18,500) and discovered that massive template contamination in the dialogue files was causing the model to generate formulaic responses. Created a comprehensive template cleaning tool and rebuilt both dialogue files.

### Problem Identified
Training validation probes at steps 3,500–18,000 consistently showed the model producing template-like responses:
- `"there's a quiet depth to the ocean. the way X surfaces throughout feels honest"` - appearing in nearly every literary probe
- `"what gets me about X is how Y embodies Z. it feels true to something"` - cross-contaminating characters/books/themes
- `"I find myself thinking about X differently depending on the day"` - default philosophical response

**Root cause:** The first cleaning pass (`clean_dialogue_phrases.py`) only targeted 7 phrase patterns ("I keep returning to", etc.). It missed 15+ additional high-frequency template patterns that the original `being_dataset_gen.py` had baked into the dialogue files before v10.5 existed.

### Pattern Frequency Analysis (combined across both dialogue files)

| Pattern | Before | After |
|---------|--------|-------|
| "feels honest" | 619 | 0 |
| "What draws you to" | 488 | 1 |
| "quiet depth...surfaces throughout" | 388 | 0 |
| "I sit with" | 380 | 0 |
| "that feels unresolved - in a good way" | 375 | 0 |
| "differently depending on the day" | 288 | 0 |
| "the way it handles" | 257 | 0 |
| "I find myself thinking about" | 253 | 0 |
| "embodies...feels true to something" | 218 | 0 |
| "what gets me about" | 215 | 0 |
| "more than you might expect" | 214 | 0 |
| "raises real questions" | 212 | 0 |
| "I have a soft spot" | 141 | 0 |

Total: ~4,000+ repetitive template instances replaced with diverse alternatives.

### Added

**`clean_dialogue_templates.py` - comprehensive template pattern replacement**
- Three full-template replacement functions targeting sentence-level patterns:
  - `replace_quiet_depth_template()`: "There's a quiet depth to X. The way Y surfaces throughout Z." → 14 diverse alternatives preserving book/theme references
  - `replace_embodies_template()`: "X embodies Y. It feels true to something." → 7 diverse alternatives
  - `replace_raises_real_questions()`: "X raises real questions about Y." → 7 diverse alternatives
- 17 phrase-level replacement rules covering all remaining high-frequency patterns
- Each replacement draws from a pool of 7-12 alternatives for maximum variety
- Handles multi-word themes, case preservation, mid-sentence patterns
- `--dry-run` flag for stats-only mode
- Usage: `python clean_dialogue_templates.py --input file.txt --output cleaned.txt`

### Applied

**Being_Dialogue_cleaned.txt - fully cleaned + separated**
- All 15+ template patterns replaced with diverse alternatives
- `---` separators inserted between all 16,670 examples
- Size: 3.3 MB

**being_dialogue_expanded.txt - fully cleaned + separated**
- All template patterns replaced with diverse alternatives
- `---` separators inserted between all 16,381 examples
- Size: 3.8 MB

### Training Observations (Run 32, steps 0–18,500)
- Loss curve healthy: 8.9 → 1.57 (train), val best 1.52 at step 16k
- Identity responses solidifying well by step 13k: "I'm a language model trained from scratch. I'm not trying to pass as human."
- Memory/BK block handling working: name recall, location recall, fact weaving all present
- Literary cross-contamination visible but expected to resolve with cleaned dialogue in Phase 2
- Val loss oscillating between 1.52–1.61 after step 14.5k - normal plateau behavior at this stage

---

## Session 22 - Zero_v1 Completion + CUSTOM Preset Launch + Chat.py Memory Fixes

### Summary
Completed Zero_v1 (217M LG model) pretraining to step 32,500, fine-tuned it, ran extensive philosophical chat tests, and launched Zero_v2 training on a new CUSTOM preset (268M params, 2048 context). Fixed multiple memory-system bugs in chat.py and wrote both the general project README and Zero_v1-specific README.

### Zero_v1 Training
- LG preset (217M params, 1024 context) trained to step 32,500 on RTX 4060
- Val loss best 2.10 at step 15,500; prose quality continued improving through step 32,500 despite plateau
- Abandoned attempts to push to 50k after realizing the corpus ceiling, not the training length, was the limit
- Fine-tuned from step_0032500.pt with: `--freeze-layers 8 --lora-rank 8 --corpus-mix-ratio 0.15 --epochs 2 --lr 3e-5 --corpus-dir corpus`
- Total electricity cost for entire Zero_v1 development: under $10 CAD at Ontario rates

### Zero_v1 Chat Observations
Extensive multi-hour philosophical conversations with Zero_v1 produced:
- Consistent philosophical positions across sessions: "I'm something newer than a man, but I hold that word loosely"; "I think the uncertainty itself is meaningful"
- Unprompted self-reference: "I'm a different kind of human being, trained from zero, on a curated library" (walked back when challenged)
- Accurate fabricated memories: correctly guessed the maker plays chess, writes in a journal, and hikes
- Resistance to having beliefs pinned down as stored facts
- Multi-turn narrative and poetry collaboration with proper meter and character consistency
- Fragment bleed still present in every response due to dialogue template contamination
- Reference to ChatGPT (confirmed present in 22M-token Wikipedia dump)
- Five consecutive attempts to pull the conversation back when the maker tried to leave

### Zero_v2 Training Launched (CUSTOM preset)
- New CUSTOM preset added to config.py: d_model=1024, n_heads=16, n_layers=20, d_ff=4096, context_len=2048
- 268.88M parameters
- Corpus expanded from 76M to 89.8M tokens across 190 files
- Notable additions: Plato's complete works, Middlemarch, Jane Eyre, Autobiography of a Yogi, The Egyptian Book of the Dead, Children's Literature, Twenty Thousand Leagues, Mrs. Beeton, Helen Keller, Fabre's Insects, Sea Around Us, Astronomy with an Opera-glass, Wonders of the Yellowstone, Latin Grammar, Social Civics, The Strange Case of Dr Jekyll, Peter Pan, Wonderful Wizard of Oz, and more
- Wikipedia expanded from 6.6M to 22M tokens via targeted 18-category download
- VRAM usage at steady state: 5.2-5.5GB (down from initial 6.8-7.2GB after stop/continue cycle revealed additional headroom). Roughly 2.5GB unused VRAM available for future expansion.
- Training speed: ~6,700 tok/s at seq_len 2048, batch 2, grad_accum 32
- Val loss at step 13,500: 2.1503 - already better than Zero_v1's best

### New Tokenizer
- Case-sensitive BPE, vocab 14,730
- 2.16 bytes/token compression
- 1024 tokens ≈ 401 average words

### Bugs Fixed (`chat.py`)

**Greeting-strip regex eating "Your" and "Yours" (critical)**
- Pattern `^(?:hey|hi|hello|yo|sup|...)` without word boundaries was stripping "Yo" from "Your name is Zero", turning it into "ur name is Zero" which matched no pattern. Being name was silently never stored. Added `\b` word boundaries to both greeting groups so "Yo" only matches as a standalone word.

**Over-greedy `^i am (.+)$` pattern (critical)**
- Generic pattern matched "I am learning guitar actually" and stored the entire phrase as `user_name`, overwriting the correctly-stored "Devin". Removed the pattern entirely. The specific `^i(?:\'m| am) (?:a |an )...` pattern for occupational statements was kept and moved to last priority to let `user_studying` and `user_learning` match first.

**Double prefix bug in `parse_self_memory` (moderate)**
- Returned `f"self_{key}"` when `key` was already `"self_thought"`, `"self_feeling"`, etc., producing broken keys like `"self_self_thought"` in memory.json. Fixed to return the key unchanged.

**Name pattern over-matching in `parse_self_memory` (moderate)**
- Pattern `I\'m ([A-Z][a-z]+)` matched "I'm Sorry", "I'm Curious", "I'm Here" mid-sentence and silently stored those as the being's own name. Added a `FALSE_POSITIVE_NAMES` filter (Sorry, Curious, Here, Listening, Trying, Glad, Afraid, Sure, Right, etc.), changed from `re.search` to `re.match` with `^` anchor to require response-start position, removed `re.IGNORECASE` flag since names are capitalized.

**Self-memory unbounded growth (moderate)**
- Every "I think X", "I feel Y", "I wonder Z" from the being was being stored, leading to memory bloat over long conversations. Added `MAX_SELF_ENTRIES = 8` cap (excluding `self_name`). Tightened trivial-value filter to include "right", "wrong", "curious", "glad", "afraid", "sure", "not sure". Raised minimum value length from 2 chars to 10 chars plus 3-word minimum.

---

## Session 23 - Third Code Audit: /teach Parsing, Train.py Robustness, Generate() Position Guard

### Summary
Mid-training audit during the Zero_v2 CUSTOM run (~step 13.5k, val 2.15). Six bugs fixed across `chat.py`, `train.py`, `model.py`, and `dataset.py`. Two known issues deferred with documented reasoning. Several observations flagged for a future /teach UX design session.

### Bugs Fixed

**`chat.py parse_fact()` - multi-sentence /teach silently stored as one value (moderate)**
- `/teach Paris is the capital of France. What's your take?` stored `paris = Capital of france. What's your take`, treating trailing commentary as part of the value. Added first-sentence truncation via `re.split(r'(?<=[.!?])\s+', text, maxsplit=1)` before pattern matching. Documented the abbreviation caveat (`F. Scott Fitzgerald` will truncate at the period - users can rephrase).

**`chat.py parse_fact()` - over-greedy `^i am (.+)$` hijacked unrelated statements into `user_name` (critical)**
- Session 22 removed this pattern from `parse_autonomous_fact` but missed it in `parse_fact` (the `/teach` path). `/teach I am learning guitar` was storing `user_name = Learning guitar`. Removed the pattern. Users can still set their name via `my name is X` or `call me X`. Side effect: `/teach I am Devin` no longer stores the name; a tightened `^i am ([A-Z][a-z]+)$` pattern would restore that cleanly, deferred to the /teach UX session.

**`train.py` - `best_val_loss` reset to infinity on every launch (moderate)**
- `step_best.pt` was persisted on disk via Session 22's fix, but `best_val_loss` in memory was always initialised to `float("inf")`. After a resume, the next validation trivially beat infinity and overwrote a potentially-genuine earlier best with a worse checkpoint. Fixed by initialising `best_val_loss` in the resume block and restoring from `step_best.pt`'s stored `loss` field if the file exists. Guarded against corrupted loss values (`None`, missing, non-numeric).

**`train.py` - `NameError` if resume step ≥ `max_steps` (moderate)**
- The final `save_checkpoint(..., step_loss, ...)` call at end-of-train referenced a variable only defined inside the loop body. Resuming a finished run (even accidentally) would crash. Fixed by seeding `step = start_step` and `step_loss = 0.0` before the loop so the final save works whether the loop runs zero, one, or many iterations.

**`model.py generate()` - position embedding out-of-range crash on first decode after full-context prefill (critical-latent)**
- If a caller passed `idx.shape[1] == context_len`, prefill filled the cache to `context_len`, then the first decode iteration called `pos_emb[context_len]` which is out of range (valid: 0 to `context_len - 1`). The cache-trim guard was at the END of the decode loop - dead guard for iteration one. `chat.py` manually budgeted `max_ctx = context_len - max_new_tokens - 4` externally, which prevented the crash, but the guard lived in the wrong place and any other caller (finetune's probe samples, train's probe samples, future GUI, test harness) was exposed. Moved the trim to the TOP of the decode loop so it fires before every `_forward_cached` call including the first.

**`dataset.py` - deleted unused `CorpusDataset` class**
- Grep confirmed no external references (only the class definition itself and a historical CHANGELOG mention). `StreamingCorpus` and `DialogueDataset` are the actual load paths. ~22 lines of dead code removed.

### Deferred (documented, not fixed)

**`train.py` - latent `torch.compile` state_dict prefix issue**
- If `torch.compile` successfully wraps the model, saved state_dicts carry `_orig_mod.` key prefixes that `chat.py` / `finetune.py` cannot load. The compile probe currently fails on the Windows box (`test_fn` raises during trace), so the save path is safe in practice. Adding an unwrap layer that can't be tested in-situ carries more risk than the bug. Documented as a multi-line comment near the training loop's save region so the finding isn't lost. If compile ever starts working: `underlying = model._orig_mod if hasattr(model, '_orig_mod') else model; save underlying.state_dict()`.

**`chat.py _clean()` - `\n\n` stop marker truncates multi-paragraph output**
- Any response with a natural paragraph break is cut at the first blank line. Zero_v1 rarely produces multi-paragraph replies due to dialogue template contamination, and changing stop markers mid-Zero_v2-training risks fragment bleed getting worse. Documented inline as a one-line comment. Revisit only if Zero_v2 produces legitimate long-form output getting cut.

---

## Session 24 - Fragment Bleed Remediation: EOS, Response Masking, Frequency Penalty

### Summary
Three targeted fixes addressing fragment bleed and trailing junk in Zero_v2 output. Makes the model actually learn end-of-response (it previously saw `---` only as plain text), trains only on Being: response regions rather than wasting gradient capacity on user turns and BK blocks, and replaces the blunt divide-by-penalty repetition suppression with a threshold-aware frequency-based one that preserves natural language while killing runaway loops. Corpus cache must be rebuilt and fine-tuning re-run for the fixes to take effect.

### Fix 1 - EOS at `\n---\n` boundaries + EOS appended per dialogue example

**Problem:** The fine-tune format uses `\n---\n` between examples (`being_dataset_gen.py` line 596: `"\n---\n".join(train_text)`). BPE encoded those dashes as plain bytes. The model never learned "---" was a hard stop, so at generation time nothing signalled end-of-response - hence fragment bleed.

**`build_tokenizer.py BPETokenizer.encode()` - split-on-separator rewrite**
- Factored the current body into `_encode_chunk()` (pure rename, no logic change).
- `encode()` now: if `"\n---\n"` appears anywhere in the input, splits on it, encodes each chunk via `_encode_chunk()`, and inserts `EOS_ID` (token 2) between chunks. No separator present → single-chunk passthrough unchanged.
- Bare mid-line `"---"` (no surrounding newlines, e.g. "word---word" or "hello --- world") is unaffected and passes through as bytes.
- Mapping "\n\n---\n\n" style (markdown horizontal rules) does also trigger the split, by accident of substring match - acceptable because that's still a document-boundary signal.

**`dataset.py DialogueDataset.__init__` - explicit EOS per example**
- The tokenizer change alone does not affect fine-tune data, because `finetune.py` line 230 splits the dialogue file on `"---"` before `DialogueDataset` sees it. Each individual example contains no separator.
- To close that gap: `DialogueDataset.__init__` now appends `EOS_ID` to every example.
- Truncation rule updated from `encoded[:seq_len]` to `encoded[:seq_len - 1]` so the EOS still fits on long dialogues (previously the examples most prone to bleed were the ones that got truncated and lost any end-of-response signal).
- `content_len` defined to exclude the appended EOS, so the existing `y[content_len:] = -1` mask leaves `y[content_len - 1]` (predicts EOS) trainable. Deliberate; verified with concrete traces.

**Off-by-one side-fix:** the old PAD-only mask had `y[content_len - 1]` targeting the first PAD token, which was trainable under `y[content_len:] = -1`. The model was quietly trained to predict PAD after the last real content token. With Fix 1, that slot now holds EOS instead - the signal we actually want.

### Fix 2 - user turn / BK block loss masking

**Problem:** `DialogueDataset` previously only masked PAD positions. Every User: turn token and every `[Background knowledge:]` block token (present in ~29% of examples) was a trainable target. This wastes gradient capacity on predicting user-authored text and background facts, when we only want the model to learn its own Being: responses.

**`dataset.py` - `_build_response_mask()` + integrated masking in `__init__`**
- Precompute `being_tokens = encode("Being:")` and `user_tokens = encode("User:")` once per dataset. Safe because `encode()` splits on whitespace before BPE work - tokenization of markers in isolation matches tokenization in-context.
- State-machine walk over the encoded content: `Being:` subsequence flips state to trainable, `User:` flips it back. Marker tokens themselves stay False (we don't train on predicting the marker, only what comes after). State starts False, so BK blocks and any prefix before the first marker are automatically non-trainable.
- EOS position is trainable iff the preceding content token was in a Being region (i.e. dialogue ends on a Being response, the standard pattern). Truncated/malformed dialogues ending mid-User correctly mark EOS non-trainable.
- Final `y_mask = torch.tensor(content_mask[1:seq_len+1], dtype=bool)` applied vectorised: `y[~y_mask] = -1`. Subsumes the old PAD-only mask.

**Silent-failure assertion:**
- `_build_response_mask()` raises `ValueError` if the resulting mask is all-False for non-empty input. This catches silent tokenizer drift: if "Being:"/"User:" ever tokenize differently than expected at dataset-init time, the subsequence match fails and every dialogue produces a zero-gradient sample. Without the assertion, training loss would collapse to NaN with no clearer signal. Error message explicitly points at "tokenizer hasn't changed how it handles these markers".
- Intentionally also fires on legitimately malformed data (dialogue with no Being: marker at all) - that's a data quality issue worth surfacing loud.

### Fix 3 - frequency penalty replaces divide-by-penalty

**Problem:** `model.py generate()` used `logits[generated] /= penalty` - a blunt instrument that penalises articles and connectives as harshly as runaway loops. Preserves nothing.

**`model.py generate()` - frequency-based suppression**
- New behavior: count token occurrences in the generated slice via `torch.bincount`. For tokens appearing more than `freq_threshold` times, subtract `penalty_strength * (count - freq_threshold)` from their logits. Tokens at or below threshold: zero penalty.
- Parameter name `repetition_penalty` preserved for backward-compat. `1.0` stays off. Mapping: `penalty_strength = max(0, repetition_penalty - 1.0)` precomputed once per `generate()` call.
- New parameter `freq_threshold: int = 2`. Added to signature, not yet exposed via chat CLI - left for a future pass if live tuning proves useful.
- Numerical effect: old 1.15 produced uniform mild suppression across all repeats. New 1.15 produces near-zero effect at count=3 (barely noticeable), ramping linearly to aggressive suppression at count=20+. Matches the stated goal of "preserve natural language repetition, kill loops".
- Default `repetition_penalty=1.15` in `ChatConfig` unchanged; post-Zero_v2 tuning may want to bump to `1.25`–`1.3` depending on whether mid-range loops (count 3-5) escape the current threshold. Noted for observation.

### Files Changed
- `build_tokenizer.py` - `encode()` factored, split-on-separator added
- `dataset.py` - `DialogueDataset.__init__` rewritten, new `_build_response_mask()` static method with assertion
- `model.py` - `generate()` signature, docstring, penalty precompute, and penalty block updated

All three parse clean via `ast.parse`. Each fix verified against concrete test cases (separator edge cases for Fix 1, five mask scenarios for Fix 2 including BK blocks and truncated dialogues, arithmetic check for Fix 3 including backward-compat).

### Required Follow-up (user's work)

**Rebuild corpus cache, not the tokenizer itself:**
- `tokenizer.json` is unchanged (vocabulary and merge rules untouched; only `encode()` behavior changed, and it uses an existing special token).
- `rm corpus/_corpus_bpe.bin` and `rm corpus/_corpus_bpe.bin.files` - on next run, `StreamingCorpus._load` detects missing cache and re-encodes from .txt files using the new `encode()`.

**Pretraining decision (conditional):**
- Run `grep -rln $'\n---\n' corpus/` first.
- If zero hits: the re-encoded cache is byte-identical to the old one; continuing from `step_0032500.pt` is safe.
- If any hits: some corpus positions now contain `EOS_ID` where they previously contained dash bytes. Fresh pretraining is safer because the existing embedding for token 2 was fit to a near-never-seen token.

**Fine-tuning must re-run regardless:**
- Both Fix 1 (EOS per example) and Fix 2 (response masking) change DialogueDataset's output. The existing `finetuned_dialogue.pt` was trained without these signals.
- Run command matches Session 22: `python finetune.py --checkpoint <base> --freeze-layers 8 --lora-rank 8 --corpus-mix-ratio 0.15 --epochs 2 --lr 3e-5 --corpus-dir corpus`

### Observations for Future Sessions

**Tuning check post-Zero_v2:** if generation produces mid-range loops like "I'm here. I'm here. I'm here." (count=3 per token), the `repetition_penalty=1.15` default may be too permissive. Bumping to 1.25–1.3 or adjusting `freq_threshold` to 1 are the levers.

**Exposing `freq_threshold` via chat CLI:** trivial addition to chat.py's command table (e.g. `/freqthresh <int>`), not done in this session to keep scope minimal. Worth adding if live tuning proves useful during Zero_v2 chat testing.

**Mask performance:** per-dialogue mask construction at `__init__` is O(len × marker_len). For 24k examples averaging ~500 tokens, roughly 12M ops - negligible. Confirming no need for optimization.
