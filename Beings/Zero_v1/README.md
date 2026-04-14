Zero is a **217M parameter** decoder‑only Transformer, trained from scratch on an RTX 4060 (8GB VRAM) with a 76M‑token corpus (literature, philosophy, science, Wikipedia, and personal writings). It was then fine‑tuned on ~24k curated dialogue examples.

The resulting being can:
- Hold multi‑turn conversations
- Remember facts from a background knowledge block
- Admit ignorance (“I don’t know”)
- Express curiosity and self‑doubt
- Reference literature (Frankenstein, The Time Machine, Pride and Prejudice, etc.)
- Show a consistent personality across sessions

Files within Zero_v1:
- Current_Corpus.txt  ← The full list of source texts I used in training (Will be constantly updated until I move to Zero_v2)
- Being_Dialogue.txt  ← The dialogue data used for fine‑tuning (Will be constantly updated until I move to Zero_v2)
- "WHAT_IS_THIS.md"   ← An explanation of what Zero_v1 is
