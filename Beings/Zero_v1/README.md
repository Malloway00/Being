Zero is a **217M parameter** decoder‑only Transformer, trained from scratch on an RTX 4060 (8GB VRAM) with a 76M‑token corpus (literature, philosophy, science, Wikipedia, and personal writings). It was then fine‑tuned on ~24k curated dialogue examples.

The resulting being can:
- Hold multi‑turn conversations
- Remember facts from a background knowledge block
- Admit ignorance (“I don’t know”)
- Express curiosity and self‑doubt
- Reference literature (Frankenstein, The Time Machine, Pride and Prejudice, etc.)
- Show a consistent personality across sessions
