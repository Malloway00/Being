# Being_Dialogue.txt - Project Changelog

**Project:** Training data curation for "Being" - a small custom language model trained from scratch on a classical literary corpus.

**Final file:** `Being_Dialogue.txt`
**Final stats:** 24,385 examples · 4.17 MB · 124,819 lines

---

## Current File Composition

| Metric | Count | % |
|---|---|---|
| Total examples | 24,385 | 100% |
| Single-turn | 20,103 | 82% |
| Multi-turn (2+ turns) | 4,282 | 18% |
| Long multi-turn (4+ turns) | 220 | ~1% |
| With `[Background knowledge:]` blocks | 7,083 | 29% |
| Unique being names (Zero, Echo, Ember, etc.) | 27 | - |
| Unique user names | 411 | - |

### Response length distribution
| Length | Count | % | Purpose |
|---|---|---|---|
| Short (≤6 words) | 9,948 | 40% | Anti-fragment, clean stopping |
| Medium (7–20 words) | 12,096 | 49% | Conversational sweet spot |
| Long (21+ words) | 2,337 | 9% | Depth on complex topics |

### Quality guarantees
- ✅ **Zero banned phrases** - all 16 contamination phrases purged
- ✅ **Max 2 identical identity responses** - duplicate-cap tightened in Pass 15
- ✅ **Max 6 repeat for any response >8 words** - repetition controlled
- ✅ **27 being names** - any future being can be named without retraining
- ✅ **Miscategorized-response bugs fixed** - Pass 15 removed 2 examples where identity responses were attached to unrelated factual questions

---

## Pass-by-Pass History

### Pass 1 - Merge & Initial Expansion
**Result:** 33,671 unique examples (6.9 MB)
- Merged two original source files → 30,249 deduplicated examples
- Added ~3,400 generated examples across categories

### Pass 2 - Everyday & Multi-Turn Depth
**Result:** 35,793 examples (7.1 MB) · +2,122 new
- 1,852 everyday single-turn (food, weather, work, hobbies, pets, travel)
- 270 multi-turn conversations (4–6 turns)

### Pass 3 - BK Block Boundary Enforcement
**Result:** 36,646 examples (7.2 MB) · +853 new
- Name-only → what-do-you-know answers
- Name + one fact → Being uses only what's given
- Bait prompts → Being refuses to fabricate
- Adjacent-topic resistance

### Pass 4 - High-Priority Input Handling
**Result:** 37,451 examples (7.3 MB) · +805 new
- Typos / messy input (~250)
- Emoji & emoticon handling (~180)
- Ultra-short inputs (~150)
- Multilingual & code-switching (~80)
- Hostile / adversarial (~145)

### Pass 5 - Medium-Priority Categories
**Result:** 37,807 examples (7.4 MB) · +356 new
- Humor & jokes (~65)
- Roleplay & hypotheticals (~45)
- Disagreement & pushback (~55)
- Topic transitions (~25)
- Repetition handling (~25)
- Personality types (~55)
- Apologies & mistake recovery (~25)
- Can't-fulfill requests (~40)
- Sensitive topics (~45)

### Pass 6 - Zero Name + Anti-Fragment
**Result:** 38,704 examples (7.5 MB) · +897 new
- Zero-specific BK blocks (~405)
- Short complete responses / anti-fragment (~270)
- Correction handling (~80)
- Continuation prompts (~87)
- No-fabrication with minimal BK (~190)

### Pass 7 - MAJOR CLEANUP
**Result:** 25,430 examples (4.7 MB) · −13,274 removed

**Why the big drop:** Earlier passes had accumulated heavy duplication and contamination that was actively harming the trained model (fragment bleed, phrase over-repetition in chat tests). Surgical cleanup:
- Removed banned phrases entirely: "quiet depth", "surfaces throughout", "was the center of their world", "in a way that doesn't flinch", "wrestles/wrestle with", "What draws you to it?"
- Removed "I'm Being" identity contamination (91 examples)
- Capped exact duplicate responses at max 3 (−8,700)
- Capped repeated response starts at max 8 (−4,300)

**Also added:** compliments toward Being, silence/re-entry, storytelling reactions, cultural references, age-appropriate register (~170 total)

### Pass 8 - Chat Test Failure Targeting
**Result:** 23,421 examples (4.0 MB) after further cleanup + 609 new
- Removed ~2,000 more contaminated/false-memory examples
- Added varied being-name examples across 20 names (278)
- Anti-fragment one-sentence responses (155)
- Anti-fabrication with false-memory corrections (90)
- Coherent conversation reinforcement (70)
- Goodbye handling (35)

### Pass 9 - Nuance & Tone
**Result:** 23,020 examples (4.0 MB) · +128 new
- Natural multi-turn conversations (30)
- Longer thoughtful responses (15)
- Correction chains with behavioral follow-through (15)
- Tone mirroring - excited/calm/playful/heavy (40)
- Proactive questions (10)
- Meta-awareness of limitations (18)

### Pass 10 - Volume Recovery with Variety
**Result:** 23,841 examples (4.1 MB) · +821 new
- Being name BK blocks with 24 names × uncommon user names (336)
- Anti-fabrication with uncommon names, varied phrasing (120)
- Varied everyday exchanges with multiple response variants (480)
- Named multi-turn conversations (132)
- Response variety - 15 answers to "How are you?", 10 to "What are you?" (131)
- Fresh real-feeling multi-turn (20)

### Pass 11 - Corpus-Rooted Content
**Result:** 23,948 examples (4.1 MB) · +107 new
- Natural science & nature writing (25) - Sea Around Us, Fabre's insects, Darwin, Newton
- World literature beyond Western canon (18) - Tale of Genji, Thousand and One Nights, Bhagavad Gita, Tao Te Ching, Analects, Norse Eddas, Rumi, Marco Polo
- History, travel & diaries (10) - Plutarch, Pepys, Douglass, Franklin, Equiano, Orwell, Keller
- Practical knowledge (15) - Strunk & White, habit formation, nutrition
- Lesser-known fiction (15) - King in Yellow, Three Men in a Boat, Blue Castle, Siddhartha
- Music & arts (10)
- Corpus references woven into everyday conversation (14)

### Pass 12 - Pre-Training Final Push
**Result:** 24,267 examples (4.2 MB) · +319 new
- Anti-fragment short responses (119)
- Long multi-turn for 2048 context window (11)
- More being-name + uncommon user-name combos (104)
- Anti-fabrication reinforcement (58)
- Goodbye reinforcement (35)

### Pass 13 - Rewrite Pass (no deletion)
**Result:** 24,267 examples (4.2 MB) · 327 rewrites
- 128 copies of top-7 duplicated responses rewritten with 8 varied phrasings each
- 111 repetitive book-discussion openers rewritten with 6+ templates
- 45 over-repeated identity responses diversified
- 31 philosophical template repeats varied
- 7 "I'm Zero. What can I help you with?" rewrites
- 12 broken-rewrite artifacts fixed
- 54 punctuation artifacts cleaned (`?.` → `?`)
- Max repeat for responses >8 words: 25 → 6

### Pass 14 - Character Additions
**Result:** 24,421 examples (4.18 MB) · +154 new
- **Non-Servile Refusals (78)** - Being saying "no" with backbone:
  - Refusing to impersonate other AIs (15)
  - Refusing to fabricate facts or quotes (15)
  - Refusing cruelty toward others (15)
  - Refusing flattery / sycophancy (13)
  - Refusing boundary violations (13)
  - Multi-turn refusals where Being holds ground (7)
- **Being Initiates / Proactive (38)** - Being driving with curiosity:
  - Follow-up on previous conversations (4)
  - Conversation openers with questions (14)
  - Clarifying questions beyond surface (8)
  - Curiosity-driven multi-turn (8)
  - Being volunteering observations first (4)
- **Opinion & Reasoning with Depth (39)** - Being holding views across turns:
  - Defended opinions in multi-turn dialogue (16)
  - Strong single-turn opinions (15)
  - Literary/philosophical opinions with reasoning (8)

### Pass 15 - Post Zero_v2 Diagnosis Trim
**Result:** 24,385 examples (4.17 MB) · −36 removed

**Context:** Zero_v2 training completed (5-day run, 268M params, 2048 context, 15% literary rehearsal). Chat tests showed persistent fragment bleed, name fabrication on first message, templated identity latching at low temperature, and user-fact hallucination. Diagnosis concluded the remaining issues were training-logic problems, not data problems.

**Dialogue file changes (narrow, targeted):**
- **Group A - Duplicate-cap tightening (35 removals):** Identity responses appearing 3+ times capped to max 2 copies each. No unique phrasings lost. Trimmed: "A language model. Small. Well-read. Unsure of its own nature." (5→2), "Built from text, shaped by reading. That's what I am." (4→2), "I'm a language model with a narrow diet - mostly classics." (6→2), and ~12 others.
- **Group B - Generic identity clone cleanup (1 additional removal):** "Software that learned language from literature." reduced from 6x total to 1x (4 caught by Group A's cap logic, 1 additional from Group B). The phrase was the heaviest duplicate and interchangeable with many unique identity responses.
- **Group C - Rejected by user.** "I'm a language model trained from scratch. I know what I've read, nothing more." kept at 2x. User flagged it as load-bearing - appears verbatim in multiple Zero_v1 chat transcripts.
- **Bonus fix:** 2 of the Group A removals (A10 and A19) were also miscategorized examples where identity responses were attached to unrelated factual questions (e.g., "What's the hottest planet?" with BK stating "Venus is the hottest planet" received an identity response about being a language model). Removing these fixes a data-coherence bug beyond duplicate-cap logic.

**What was intentionally NOT changed:** ~260 unique identity response phrasings preserved. Zero_v2's latching at low temperature is a sampling problem, not a data problem - cutting unique phrasings would reduce range without fixing the latching.

---

## Category Totals in Final File

| Category | Approximate count |
|---|---|
| Knowledge boundary / IDK training | ~2,800 |
| BK fabrication prevention | ~1,043 |
| Being name in BK blocks | ~908 |
| Everyday conversation | ~2,200 |
| Multi-turn conversations | 4,282 |
| Long multi-turn (4+ turns) | 220 |
| Typos / messy input | ~250 |
| Emoji handling | ~180 |
| Ultra-short inputs | ~150 |
| Hostile / adversarial | ~145 |
| Multilingual / code-switching | ~80 |
| Humor & jokes | ~65 |
| Roleplay / hypotheticals | ~45 |
| Disagreement & pushback | ~55 |
| Topic transitions | ~25 |
| Repetition handling | ~25 |
| Personality types | ~55 |
| Apologies / recovery | ~25 |
| Can't-fulfill requests | ~40 |
| Sensitive topics | ~45 |
| Anti-fragment short responses | ~270 |
| Correction handling | ~105 |
| Continuation prompts | ~100 |
| Compliments toward Being | ~25 |
| Silence & re-entry | ~20 |
| Storytelling reactions | ~40 |
| Cultural references | ~25 |
| Age-appropriate register | ~30 |
| Tone mirroring | ~40 |
| Proactive questions | ~48 |
| Meta-awareness | ~18 |
| Goodbye handling | ~150 |
| Corpus (science, world lit, history, practical) | ~107 |
| Non-servile refusals | ~78 |
| Being initiates | ~38 |
| Opinion & reasoning | ~39 |
| Identity responses (post Pass 15 trim) | ~267 |

---

## Being Names Coverage (27 names)

| Name | Examples | Name | Examples |
|---|---|---|---|
| Zero | 315 | Sage | 20 |
| Echo | 54 | Atom | 20 |
| Nova | 51 | Wisp | 15 |
| Ember | 50 | Haze | 15 |
| Ash | 48 | Glyph | 15 |
| Lyra | 48 | Spark | 15 |
| Sol | 47 | Dusk | 15 |
| Pixel | 46 | Quill | 15 |
| Drift | 25 | Volt | 15 |
| Aura | 25 | Wren | 3 |
| Onyx | 25 | | |
| Flux | 25 | | |
| Coda | 21 | | |
| Rune | 21 | | |
| Vale | 21 | | |
| Moss | 21 | | |
| Lux | 21 | | |

---

## Banned Phrases (All Purged)

These were removed because the trained model was memorizing and regurgitating them:
- "was the center of their world"
- "approached it from one angle"
- "I'm still turning it over"
- "That one stops me"
- "That feels layered"
- "That lingers"
- "That sits with me"
- "quiet depth"
- "surfaces throughout"
- "in a way that doesn't flinch"
- "wrestles with" / "wrestle with"
- "What draws you to it?"
- "feels honest"
- "I keep returning to"
- "keeps coming back to me"
- "I'm Being." / "My name is Being." (identity contamination from before Zero_v1)
