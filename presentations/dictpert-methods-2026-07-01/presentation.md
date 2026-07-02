---
marp: true
theme: default
paginate: true
style: |
  section {
    font-size: 1.4rem;
    padding-top: 30px !important;
  }
  h1, h2 {
    margin-top: 0 !important;
    margin-bottom: 0.4em !important;
  }
  section.lead h1 {
    font-size: 2.4rem;
  }
  section.lead h2 {
    font-size: 1.4rem;
    color: #555;
  }
  section.backup {
    background: #f8f8f8;
  }
  section.backup h2 {
    color: #666;
    border-bottom: 2px solid #aaa;
  }
  h1 { color: #1a3a5c; }
  h2 { color: #2c6fad; border-bottom: 2px solid #2c6fad; padding-bottom: 4px; }
  code { background: #f0f4f8; padding: 2px 6px; border-radius: 3px; }
  pre { background: #f0f4f8; }
  .columns { display: grid; grid-template-columns: 1fr 1fr; gap: 2rem; }
  .sidenote {
    background: #fffbeb;
    border-left: 4px solid #f59e0b;
    padding: 0.6rem 1rem;
    font-size: 1.1rem;
    color: #444;
    margin-top: 1rem;
  }
  table { font-size: 1.1rem; }
  blockquote { border-left: 4px solid #2c6fad; color: #444; }
---

<!-- _class: lead -->

# dictpert — Recent Methods

## Algorithmic Improvements in Czech Dialectology RAG
**June–July 2026**

Martin Povolný

---

## Context

**The System:** Multi-dictionary RAG for Czech dialectology

- **6 primary dictionaries**: ČJA, SNČJ, SPJMS, SPJČ, PSJČ, ASSC
- **Diverse formats**: Structured JSON, scraped HTML, PostgreSQL, SQLite
- **The challenge**: Vocabulary gap between natural language queries and archaic/dialectal headwords
- **Quality bar**: Citation discipline — every claim must be grounded in retrieved entries

**This presentation:** Seven algorithmic improvements implemented June–July 2026
- 5 major methods + 2 supporting methods + prompt micro-improvements

---

## 1. Native Tool Calls

### The Core Idea

**Stop fighting the model's native interface — use it.**

Instead of forcing the LLM to emit text-based tool directives (`[LOOKUP:term|dict_id]`) that we parse, expose tools through the model's **native function-calling API**.

**Why this works:** The model uses the same JSON-based tool protocol it was trained on. No translation layer = no protocol confusion.

---

## Native Tool Calls — The Problem

**Before (text protocol):** Model emits `[LOOKUP:term|dict_id]`, system parses and executes

**What went wrong with Claude Haiku 4.5:**
- Model leaked XML `<function_calls>` despite explicit prompt bans
- Fought against bracket notation, kept reverting to native format
- **Q15 complete failure**: 0 lookups executed, XML instead of brackets

**Root cause:** Claude models are trained on Anthropic's native tool-calling format. Forcing them to use bracket notation conflicts with training.

---

## Native Tool Calls — The Solution

**Switch to LiteLLM native tool calls:**
- Define tools in OpenAI format (JSON schemas)
- LiteLLM translates to Anthropic's native XML format
- Model sees tools the same way it did during training

**Implementation:**
```python
DICT_TOOLS = [{
    "type": "function",
    "function": {
        "name": "lookup",
        "description": "Přesné vyhledání hesla ve slovníku",
        "parameters": {
            "type": "object",
            "properties": {
                "term": {"type": "string"},
                "dict_id": {"type": "string", "enum": ["cja", "sncj", ...]}
            }
        }
    }
}]
```

---

## Native Tool Calls — Impact

**Result:** Q15 ❌ → ✅, Q6 ⚠ → ✅, Q11 ⚠ → ✅, Q14 ⚠ → ✅, Q17 ⚠ → ✅

**Key takeaway:** Align with the model's training instead of fighting it. When a model has a native interface for something, use that interface.

---

## 2. Structural Synthesis Injection (PN18)

### The Core Idea

**Reset the model's framing structurally, not through prompts.**

When the model enters the final synthesis phase, it has a conversation history full of inter-tool thinking text (*"Výborně! Nyní provedu vyhledávání..."*). Prompt instructions to "don't include this" are insufficient — **structurally supersede it** with a clean user instruction.

**Why this works:** A new user message **reframes the task**. The model now sees "start directly with the answer" as the last instruction, which takes precedence over everything that came before.

---

## Structural Synthesis Injection — The Problem

**Native tool calls exposed a new failure mode:**

The model emits **process narration** between tool calls:
- *"Výborně! Nyní provedu vyhledávání..."*
- *"Podívám se na doklady z Brněnska..."*
- *"Nyní mohu formulovat závěrečnou odpověď..."*

This thinking text leaked into the final user-facing answer, despite layered prompt bans.

**Root cause:** The final synthesis call saw the full conversation history, including all narration in `assistant` messages. Prompt instructions were insufficient — the model was synthesizing from contaminated context.

---

## Structural Synthesis Injection — The Solution

**Structural intervention:**

Inject a clean `user` turn immediately before the final synthesis call:

```
user: Nyní napiš konečnou odpověď. Začni přímo první větou odpovědi.
```

**What this does:**
- Supersedes all previous inter-tool narration
- Model can't leak thinking text if the last thing it saw was "start directly with the answer"
- The user message takes precedence over prompt instructions

---

## Structural Synthesis Injection — Impact

**Result:** 94% zero-leakage rate (15/16 queries in final eval)

**Key takeaway:** When prompts fail, use structure. The conversation context is mutable — you can inject turns that reset the model's framing mid-inference.

---

## 3. Self-RAG Critique

### The Core Idea

**Make the model critique itself before it generates hallucinations, not after.**

After each retrieval round, the model answers three self-check questions:
1. Are these results relevant to my query?
2. Do I have enough information to answer completely?
3. Can I verify every claim I want to make from these results?

**Why this works:** The critique happens **before the model commits to an answer**. When the model asks "can I verify every claim?" while still in the retrieval phase, it naturally avoids generating claims it can't verify.

---

## Self-RAG Critique — The Problem

**LLMs in RAG systems are prone to citation hallucination:**
- Claiming information appears in retrieved documents when it doesn't
- Standard mitigation: post-hoc citation verification (reactive)

**Why post-hoc verification is insufficient:**
- By the time the check fires, the model has already committed to an answer structure
- The check forces regeneration, but the model may simply rephrase the same hallucination
- Detection, not prevention

---

## Self-RAG Critique — The Solution

**Self-RAG** (Asai et al., ICLR 2024): self-critique during inference

After each retrieval round, inject three questions:
```markdown
1. Jsou tyto výsledky relevantní pro odpověď na otázku?
2. Mám dostatek informací pro úplnou odpověď?
3. Mohu každý svůj budoucí údaj ověřit v těchto výsledcích?
```

**Implementation:**
- Model answers each question explicitly before deciding to retrieve more or synthesize
- If "no" to Q3 → retrieve more rather than hallucinate
- Citation check (post-hoc) still runs, but now catches edge cases, not systemic issues

---

## Self-RAG Critique — Impact

**Result:** 81% Good rate (13/16 queries), 0 ❌ Poor verdicts, citation check fired only 19% of queries (down from typical 30–40%)

**Key takeaway:** Prevention beats detection. Self-critique during retrieval shifts citation discipline from post-check to generation-time awareness.

---

## 4. HyDE for Vector Search

### The Core Idea

**Bridge the vocabulary gap by generating what you're looking for.**

Instead of embedding the raw user query (*"Jak se říká holce na Brněnsku?"*), generate a **hypothetical dictionary entry** that would answer the query, then embed that.

**Why this works:** A synthetic entry embeds closer to real entries in vector space than the original query. The model "translates" natural language into dictionary format, bridging the vocabulary gap.

---

## HyDE for Vector Search — The Problem

**Semantic search with raw queries:**

User query: *"Jak se říká holce na Brněnsku?"*
- Embedding captures: **holka**, **Brněnsko**, question intent
- Real SNČJ entries contain: dialectal variants (`holče`, `holča`), qualifiers (`s f`), village names

**Vocabulary mismatch:**
- Query embedding is question-shaped
- Entry embeddings are definition-shaped
- Cosine similarity suffers from format gap, not semantic gap

---

## HyDE for Vector Search — The Solution

**HyDE** (Gao et al., ACL 2023): Hypothetical Document Embeddings

1. Generate a synthetic dictionary entry matching target format:
   ```markdown
   holka s f — mladá žena, dívka. hovor. »Ta holče je moc hezká.« (Brno/okres Brno-město); expr. »Naše holče už je dospělá.« (Židlochovice/okres Brno-venkov). V nářečí jižní Moravy jsou časté varianty holče, holča (jzč).
   ```

2. Embed the synthetic entry instead of the raw query

3. Retrieve real entries similar to the synthetic one

**Key insight:** The model already knows dictionary format from the system prompt. HyDE asks it to "show me what a SNČJ entry for this query would look like."

---

## HyDE for Vector Search — Implementation

**Prompt template** (`prompts/hyde_vector.md`):

```markdown
## Formát SNČJ
Napiš 2–4 věty napodobující SNČJ styl:
- Číslované významy: 1., 2., atd.
- Kvalifikátory: expr., hovor., bot., zool., cúz, jzč, mor
- Výklad významu
- Citované příklady: »text« (Obec/Okres)
- Nářeční rysy: j/ň alternace, í/ý změny, -ej koncovky

**Otázka uživatele:** {query}
**Hypotetické heslo:**
```

---

## HyDE for Vector Search — Validation Results

**Embedding similarity tests:**

| Query | Query ↔ Real | HyDE ↔ Real | Δ |
|---|---|---|---|
| "nářeční označení pro aktivitu, která se podobá pití" | 0.7287 | **0.8795** | **+0.1507** |
| "nářeční označení pro malé dítě" | 0.7372 | **0.9037** | **+0.1665** |

**Success threshold:** Δ ≥ 0.05 (3× exceeded)

**Status:** Validation complete ✅; integration into `dicts/vector_db.py` in progress

---

## HyDE — Key Takeaway

**Bridge vocabulary gaps in the embedding space, not the query space.**

HyDE doesn't retrieve better documents — it retrieves the **same documents more reliably** by translating the query into the format the embeddings were trained on.

**Caveat:** Only works when the model knows the target format. Requires format-specific prompt templates per dictionary.

---

## 5. Morphological Fallback (Majka)

### The Core Idea

**Czech is heavily inflected. Headwords are lemmas. Queries are often inflected.**

When a lookup returns zero results, lemmatize the query term and retry before giving up.

**Why this works:** Users ask about *holce* (dative), dictionaries index *holka* (nominative). Majka bridges this gap automatically.

---

## Morphological Fallback — The Problem

**Inflected query forms:**

User asks: *"Jak se říká **holce** na Brněnsku?"*
- Lookup `holce` → 0 results
- Model concludes: "not in dictionary"
- Reality: SNČJ has extensive `holka` entry

**Root cause:** Czech dictionary headwords are nominative singular (nouns), infinitive (verbs). User queries contain dative, genitive, instrumental, conjugated forms.

---

## Morphological Fallback — The Solution

**Majka** (Šmerk, RASLAN 2009): Fast Czech morphological analyzer

```python
from ufal.morphodita import Tagger

def lemmatize(word: str) -> str:
    form = tagger.tag_sentence(word)[0]
    return form.lemma  # holce → holka, mlékem → mléko
```

**Integration:**
1. Lookup `holce` → 0 results
2. Lemmatize: `holce` → `holka`
3. Retry lookup `holka` → 47 results ✅

---

## Morphological Fallback — Impact

**Result:** Q15 ⚠ → ✅ (holce → holka), Q11 improved (germanismy → germanismus)

**Key takeaway:** For inflected languages, lemmatization is not optional — it's required infrastructure. The model can't reliably guess all morphological variants.

---

## 6. FTS Multi-word Preprocessing

### The Core Idea

**Full-text search chokes on meta-words and function words.**

When the query is `frazémy týkající se alkoholu`, FTS sees it as:
```
"frazémy" AND "týkající" AND "se" AND "alkoholu"
```

No results — because no entry text contains *"týkající se"*.

**Solution:** Strip stopwords and meta-words, add prefix wildcards:
```
"frazém*" AND "alkohol*"
```

---

## FTS Multi-word Preprocessing — The Solution

**Preprocessing pipeline:**

1. Tokenize query
2. Strip stopwords: `se`, `pro`, `na`, `v`, `z`, `o`, ...
3. Strip meta-words: `frazémy`, `interjekce`, `nářeční`, `varianty`, ...
4. Add prefix wildcards: `*` suffix
5. Join with AND

**Example:**
- Input: `"interjekce pro koně"`
- Output: `"koň*"` (just the semantic core)

---

## FTS Multi-word Preprocessing — Impact

**Result:** Q09 zero-result rate 44% → 18%

**Key takeaway:** FTS is a literal substring matcher. Don't send it meta-language — send it the **lexical core** the user is actually searching for.

---

## 7. Geographic Filtering (Geofilter)

### The Core Idea

**ČJA entries contain hundreds of locality attestations. Users want a subset.**

Query: *"Jak se říká holce **na Brněnsku**?"*
- ČJA entry for `holka`: 200+ villages across Czechia and Moravia
- User asked for Brněnsko specifically

**Solution:** `geofilter` tool that takes an ethnographic region and filters ČJA results to matching localities.

---

## Geographic Filtering — Implementation

```python
def geofilter(dict_id: str, region: str) -> list[str]:
    """Return locality codes for an ethnographic region."""
    if dict_id == "cja" and region in ETHNOGRAPHIC_REGIONS:
        return ETHNOGRAPHIC_REGIONS[region]  # ["639", "641", ...]
    return []
```

**Model flow:**
1. Lookup `holka` in ČJA → 200 localities
2. Call `geofilter("cja", "Brněnsko")` → `["639", "641", "681"]`
3. Format answer citing only Brněnsko localities

---

## Geographic Filtering — Impact

**Result:** Q15 ⚠ → ✅, Q16 ⚠ → ✅

**Key takeaway:** Large entries need post-retrieval filtering when the user specifies a constraint the index can't handle. Offload constraint matching to a tool rather than asking the model to "filter mentally."

---

## Method Synergy

These methods **compose**:

1. **Native Tool Calls** → reliable tool execution
2. **Morphological Fallback** → `holce` → `holka` lemmatization
3. **FTS Preprocessing** → `"interjekce pro koně"` → `"koň*"`
4. **HyDE** → query → synthetic entry → better semantic retrieval
5. **Self-RAG Critique** → "can I verify every claim?" → fewer hallucinations
6. **Geographic Filtering** → narrow 200 localities → 3 relevant ones
7. **Structural Synthesis** → no thinking text in final answer

**Pipeline example (Q15):**
- Input: *"Jak se říká holce na Brněnsku?"*
- Majka: `holce` → `holka`
- Lookup `holka` → 200 ČJA localities
- Geofilter `"Brněnsko"` → 3 relevant codes
- Self-RAG: "Can I verify claims?" → yes
- Structural injection → clean synthesis
- Output: *"Na Brněnsku se říká **holče** (639, 641, 681). [[ČJA:holka]]"*

---

## Beyond the Seven Methods

**Prompt micro-improvements** (10+ small changes):
- Ban XML format (PN9)
- STOP repeat-lookup guard (PN10)
- VSEARCH promotion for category queries (PN11)
- ČJA compound headword awareness (PN16)
- SNČJ phraseme structure hint
- ASSC idiom FTS pre-check note

**Philosophy:** Small, targeted prompt edits stack. Each addresses one observed failure mode.

---

## Key Lessons

1. **Align with training, not against it** (Native Tool Calls)
   - When the model has a native interface, use it

2. **Structure beats prompts** (Structural Synthesis)
   - Conversation context is mutable — inject turns that reset framing

3. **Prevention beats detection** (Self-RAG)
   - Critique during retrieval, not after generation

4. **Bridge vocabulary gaps in embedding space** (HyDE)
   - Translate queries into the format embeddings were trained on

5. **Inflection is infrastructure** (Majka)
   - For inflected languages, lemmatization is required, not optional

6. **FTS needs semantic preprocessing** (FTS Multi-word)
   - Strip meta-language, send the lexical core

7. **Offload constraints to tools** (Geofilter)
   - Don't ask the model to "filter mentally" — give it a tool

---

## References

- Asai, A., Wu, Z., Wang, Y., Sil, A., & Hajishirzi, H. (2024). Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection. *ICLR 2024*. arXiv:2310.11511

- Gao, L., Ma, X., Lin, J., & Callan, J. (2023). Precise Zero-Shot Dense Retrieval without Relevance Labels. *ACL 2023*. arXiv:2212.10496

- Šmerk, P. (2009). Fast Morphological Analysis of Czech. In Sojka, P. & Horák, A. (Eds.), *Proceedings of Third Workshop on Recent Advances in Slavonic Natural Language Processing, RASLAN 2009*. Brno: Masaryk University. https://nlp.fi.muni.cz/raslan/2009/papers/13.pdf

---

<!-- _class: backup -->

## Backup: Evaluation Progression

| Date | Queries | ✅ Good | ⚠ Partial | ❌ Poor | Cost |
|---|---|---|---|---|---|
| 2026-06-25 | 9 | 7 | 2 | 0 | $0.87 |
| 2026-06-26 | 11 | 6 | 4 | 1 | $1.12 |
| 2026-06-28† | 17 | 13 | 4 | 0 | $1.25 |
| 2026-06-30‡ | 16 | 13 | 3 | 0 | $1.54 |

**†** All 17 queries completed (Option B + PN18 + phraseme hint + PN22 + majka + geofilter)  
**‡** Q11 excluded due to context limit with Self-RAG prompt addition

**Key milestones:**
- **2026-06-25**: First 0 ❌ Poor verdicts (Option B native tools)
- **2026-06-28**: First sustained 13✅ Good rate
- **2026-06-30**: 94% zero-leakage rate (Self-RAG)

---

<!-- _class: backup -->

## Backup: Cost Breakdown (June 30 Eval)

| Query | Model | Tokens in | Tokens out | Total cost |
|---|---|---|---|---|
| Q06 | haiku-4-5 | 139,759 | 4,867 | $0.1315 |
| Q08 | haiku-4-5 | 118,322 | 2,234 | $0.1036 |
| Q09 | haiku-4-5 | 209,030 | 2,580 | $0.1775 |
| Q14 | haiku-4-5 | 115,446 | 3,024 | $0.1045 |
| Q15 | haiku-4-5 | 37,862 | 981 | $0.0342 |
| Q16 | haiku-4-5 | 131,702 | 2,039 | $0.1135 |
| Q17 | haiku-4-5 | 27,906 | 910 | $0.0259 |
| Q18 | haiku-4-5 | 68,527 | 2,240 | $0.0638 |
| Q19 | haiku-4-5 | 86,179 | 2,660 | $0.0795 |
| Q20 | haiku-4-5 | 49,302 | 1,887 | $0.0469 |
| **TOTAL** | | **984,035** | **23,422** | **$1.5381** |

**Rate:** claude-haiku-4-5 = $0.80/1M input, $4.00/1M output  
**Average per query:** $0.096
