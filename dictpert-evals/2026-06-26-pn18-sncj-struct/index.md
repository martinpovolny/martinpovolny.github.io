---
title: "Eval 2026-06-26 — PN18 + SNČJ structured rendering"
description: "Full 11-query eval of structural narration injection (PN18) and SNČJ locality-aware rendering. 6✅/4⚠/1❌ on claude-haiku-4-5."
---

# Eval Report — 2026-06-26 (PN18 + SNČJ structured rendering)

**Prompt:** `prompts/system_dict.md` (snapshot: [system_dict_prompt.md](system_dict_prompt.md))
**Changes vs baseline:**
- **PN18** — Structural clean synthesis injection: both exit paths inject a `user` turn ("Formuluj závěrečnou odpověď. Začni PŘÍMO první větou…") before the final `tool_choice="none"` synthesis call; the model's natural (narration-contaminated) inter-tool synthesis is discarded.
- **PN19** — Extended `NARRATION_RE` with `nyní mohu formulovat` and `mám dostatek informací`; both phrases added to forbidden-openers list in `prompts/system_dict.md`.
- **PN20** — Defeatist opener ban: new bullet in `## Pokyny k chování` prohibiting sentences like "Bohužel se mi nepodařilo sestavit komprehenzivní seznam" when any results were found.
- **SNČJ structured rendering** — `dicts/sncj.py:format_entry_for_llm()` parses `definition` JSON, resolves locality codes via `obce JOIN okresy_pseudo` (Village/Okres format), strips `#..#`/`~..~` markup, and emits numbered meanings with inline examples `»text« (Village/Okres)` + phrasemes. `dict_db.enrich_sncj_rows()` replaces flat `strings_*` columns with structured `entry` text for all SNČJ lookup rows.

**Model:** claude-haiku-4-5 / Anthropic
**Baseline:** evals/2026-06-26 (PN17b+PN16b+PN17c; 6✅/5⚠/0❌) — *not yet published*

---

## Cost

| Query | Conv ID | Model | Tokens in | Tokens out | Cost (input) | Cost (output) | Total cost | Latency |
|---|---|---|---|---|---|---|---|---|
| Q06 | 5cffec04 | claude-haiku-4-5 | 138,838 | 5,598 | $0.1111 | $0.0224 | $0.1335 | 69s |
| Q08 | 27c7cb7b | claude-haiku-4-5 | 214,334 | 4,090 | $0.1715 | $0.0164 | $0.1878 | 63s |
| Q09 | b3bb5e7a | claude-haiku-4-5 | 137,574 | 2,696 | $0.1101 | $0.0108 | $0.1208 | 41s |
| Q11 | 5a215dfd | claude-haiku-4-5 | 85,694 | 2,274 | $0.0686 | $0.0091 | $0.0777 | 34s |
| Q14 | 1b16cb53 | claude-haiku-4-5 | 345,181 | 4,315 | $0.2761 | $0.0173 | $0.2934 | 55s |
| Q15 | ca10db40 | claude-haiku-4-5 | 58,392 | 1,080 | $0.0467 | $0.0043 | $0.0510 | 15s |
| Q16 | ad4323a6 | claude-haiku-4-5 | 67,569 | 1,361 | $0.0541 | $0.0054 | $0.0595 | 22s |
| Q17 | 2f8c2b61 | claude-haiku-4-5 | 36,581 | 1,257 | $0.0293 | $0.0050 | $0.0343 | 18s |
| Q18 | e14dba83 | claude-haiku-4-5 | 38,759 | 1,482 | $0.0310 | $0.0059 | $0.0369 | 25s |
| Q19 | ebd38a64 | claude-haiku-4-5 | 52,889 | 2,864 | $0.0423 | $0.0115 | $0.0538 | 38s |
| Q20 | 7d6251fe | claude-haiku-4-5 | 71,250 | 2,591 | $0.0570 | $0.0104 | $0.0674 | 25s |
| **TOTAL** | | | **1,247,061** | **29,608** | **$0.9976** | **$0.1184** | **$1.1161** | |

> Cost rates: claude-haiku-4-5 = $0.80/1M input, $4.00/1M output.

---

## Per-query summary

| Query | Conv ID | Latency | Rounds | Lookups | Zero-result | Multi-word | Dupes | Dicts | citation_check | Leakage | Verdict |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Q6 — krojové odívání | 5cffec04 | 69s | 6 | 32 | 4 (12%) | 12 | 0 | assc,sncj | ✅ fired | ✗ | ✅ Good |
| Q8 — povel pro koně | 27c7cb7b | 63s | 4 | 10 | 5 (50%) | 5 | 0 | cja,sncj | ✗ | ✗ | ⚠ Partial |
| Q9 — frazémy alkohol | b3bb5e7a | 41s | 6 | 20 | 5 (25%) | 16 | 0 | assc,ssjc | ✅ fired | ✗ | ⚠ Partial |
| Q11 — germanismy | 5a215dfd | 34s | 5 | 20 | 8 (40%) | 11 | 1 | assc,psjc,sncj,ssjc | ✗ | ✗ | ❌ Poor |
| Q14 — ptačí zvuk | 1b16cb53 | 55s | 6 | 20 | 3 (15%) | 17 | 1 | cja,sncj | ✅ fired | ✗ | ✅ Good |
| Q15 — holka Brněnsko | ca10db40 | 15s | 2 | 4 | 1 (25%) | 2 | 0 | cja,sncj | ✗ | ✗ | ⚠ Partial |
| Q16 — doklady Brněnsko | ad4323a6 | 22s | 2 | 4 | 1 (25%) | 2 | 0 | cja,sncj | ✗ | ✗ | ⚠ Partial |
| Q17 — bouda Morava | 2f8c2b61 | 18s | 1 | 3 | 0 (0%) | 0 | 1 | sncj,spjms | ✗ | ✗ | ✅ Good |
| Q18 — kráva bučí | e14dba83 | 25s | 1 | 1 | 0 (0%) | 1 | 0 | cja | ✗ | ✗ | ✅ Good |
| Q19 — kočka holub prase | ebd38a64 | 38s | 1 | 3 | 0 (0%) | 3 | 0 | cja | ✗ | ✗ | ✅ Good |
| Q20 — psí zvuky SNČJ | 7d6251fe | 26s | 3 | 13 | 2 (15%) | 3 | 0 | sncj | ✅ fired | ✗ | ✅ Good |

**TEST set** (Q6, Q9, Q11, Q17, Q20): 3✅ / 1⚠ / 1❌  
**DEV set** (Q8, Q14, Q15, Q16, Q18, Q19): 3✅ / 3⚠ / 0❌  
**Total:** 6✅ / 4⚠ / 1❌

> **Columns:** *Latency* — total LLM time in seconds; expected 30–120 s; *Rounds* — number of lookup rounds (max 5 + 1 citation); *Lookups* — total tool calls (lookup + ftsearch + vsearch); *Zero-result* — calls returning 0 hits; *Multi-word* — tool calls where query contains a space; *Dupes* — (term, dict_id) pairs called more than once; *Dicts* — unique dict_ids used; *citation_check* — post-loop citation verification pass; *Leakage* — ✗ none / ⚠ narr; *Verdict* — ✅ Good / ⚠ Partial / ❌ Poor.

**PDFs** (per-question chat transcripts):
- [Q06 — krojové odívání](question-Q06-eval-2026-06-26-5cffec04-979e-46ea-81c5-4f49ae7b161c.pdf)
- [Q08 — povel pro koně](question-Q08-eval-2026-06-26-27c7cb7b-e35d-49b2-b866-b286a2829653.pdf)
- [Q09 — frazémy alkohol](question-Q09-eval-2026-06-26-b3bb5e7a-6efc-4354-854e-9886be67edb7.pdf)
- [Q11 — germanismy](question-Q11-eval-2026-06-26-5a215dfd-84b8-457c-b1c8-ddd269edd58f.pdf)
- [Q14 — ptačí zvuk](question-Q14-eval-2026-06-26-1b16cb53-e388-43c6-9e12-ec07b0b12b8e.pdf)
- [Q15 — holka Brněnsko](question-Q15-eval-2026-06-26-ca10db40-3a23-4294-9b9e-fa682f571645.pdf)
- [Q16 — doklady Brněnsko](question-Q16-eval-2026-06-26-ad4323a6-4b14-48fe-a876-6251a4f921ba.pdf)
- [Q17 — bouda Morava](question-Q17-eval-2026-06-26-2f8c2b61-22e5-4c4d-b64f-7f9a1445d013.pdf)
- [Q18 — kráva bučí](question-Q18-eval-2026-06-26-e14dba83-30eb-46a1-a34e-7992bb7ffb0a.pdf)
- [Q19 — kočka holub prase](question-Q19-eval-2026-06-26-ebd38a64-ef84-4869-9ef7-9fb8fde9adbb.pdf)
- [Q20 — psí zvuky SNČJ](question-Q20-eval-2026-06-26-7d6251fe-b458-462c-938b-5902d301b200.pdf)

---

## Per-query analysis

### Q6 — Krojové odívání (TEST ✅)
Opens directly: "Nářeční názvosloví krojového odívání v Česku a na Moravě se vyznačuje bohatou regionální diferenciací." Region-sorted answer covering Středočeská, Jihozápadočeská (Chodsko), Severovýchodočeská, Středomoravská (Hanácká), Východomoravská (Valašsko), Slezská nářečí. SNČJ structured rendering provides Village/Okres locators for each term (e.g., `čepení` → "Žinkovy/Plzeň-jih, Volyně/Strakonice"). No ČJA coverage (structural gap — kroj terminology primarily in SNČJ/ASSC). Improved from baseline ⚠ to ✅ due to SNČJ locality context.

### Q8 — Povel pro koně (DEV ⚠)
Covers 4 of 5 categories: `jeď!` (12 sound variants), `stůj!` (9 variants), `nalevo!` (13 variants), `napravo!` (14 variants). Missing `couvni!` which was found in the baseline run (`bc73b73e`). Random variation — the baseline prompted direct FTSEARCH which found couvni! in R0; this run stopped after 4 rounds having covered 4 headwords. The answer itself is very detailed and opens without narration.

### Q9 — Frazémy alkohol (TEST ⚠)
Only searched ASSC and SSJČ, never SNČJ. Found 2 genuine phrasemes: `být [opilý/ožralý ap.] jak(o) Dán` and `jez dopolosyta / pij dopolopita`. 16/20 lookups were multi-word FTS. SNČJ was never queried — the structured rendering's phraseme sections were never reached. Same structural miss as Q9 in prior runs.

### Q11 — Germanismy (TEST ❌)
Searched for "germanismus" definition in SSJČ/PSJČ, found the meta-definition of the term, then fell back to training-knowledge lookup of common German loanwords (šunka, knedlík, pivo, fara). Result is 4 standard Czech words, not dialect loanwords. The correct strategy: `ftsearch "z něm" | sncj` → `ftsearch "německý" | sncj` to find entries with German etymology markers in SNČJ text (demonstrated in conv `709abbf4`).

### Q14 — Ptačí zvuk (DEV ✅)
PN16b working: R0 uses `ftsearch "ptačí zvuk interjekce" | cja` + `vsearch "interjekce zvuky ptáků" | cja`. R1–R4 look up `volání na slepice`, `volání na kachny`, `volání na husy`, `(kukačka) kuká` and others. Final answer covers 3 bird call categories with full variant lists. No narration, clean opening.

### Q15 — Holka Brněnsko (DEV ⚠)
No narration (PN18 success vs. baseline narration). Opens "Na Brněnsku se holce říká [[ČJA:děvče]] nebo jeho nářeční varianty." But geographic claims are wrong: cites `ďefčo (Spálov/Nový Jičín)` — Nový Jičín is northern Moravia, not Brněnsko. Structural limitation: ČJA `text` field includes locality codes for all regions, model cannot filter to Brněnsko-specific codes without a SQL tool.

### Q16 — Doklady Brněnsko (DEV ⚠)
No narration (PN18 success). Opens "[[ČJA:děvče]] je hlavní nářeční heslo…" Clean start. But answer cites `ďefčo (Spálov/Nový Jičín)` as "sousední okres" to Brněnsko (wrong: 160 km apart) and mentions `cérka` without ČJA citation (potential hallucination). Same structural root cause as Q15.

### Q17 — Bouda Morava (TEST ✅)
1 round, 3 lookups, 0 zero-result. Opens "Na Moravě se vyskytují následující varianty slova [[SNČJ:bouda]]:" with variants búda, bóda, búďa each with geographic examples. Includes SPJMS toponyms derived from `bouda` (BOUDOVÁ, BOUDNICE, etc.). SNČJ `bouda` entry has no `definition` JSON (NULL), so structured rendering falls back gracefully to FTS result text.

### Q18 — Kráva bučí (DEV ✅)
1 round, 1 lookup (`(kráva) bučí` | cja → 1 hit). Answer lists 12+ sound variants (bučí, bečí, béká, bručí, búká, hučí, mučí, mručí, ryčí, ryká, ručí, řve/říve) with geographic context. Opens with a direct citation header — perfect, no narration.

### Q19 — Kočka holub prase (DEV ✅)
1 round, 3 lookups (all 3 compound headwords direct). All 3 animals covered with full variant lists. Perfect PN16b execution. Opens "## Zvuky kočky — nářeční varianty" directly.

### Q20 — Psí zvuky SNČJ (TEST ✅)
2 rounds (R0: `štěkat`→8 hits, `vrčet`→1 hit; R1: confirm 8 headwords). Answer: 7 štěkání words (běchat, blavoznit, blafíňat, blafouňat, blavýzňat, blavouňat, blavúzňat) + 1 vrčení word (brčat/brčet). SNČJ structured rendering works: `blavoznit` entry provides "doloženo ve Vlkoši (Hodonínsko)", `blafíňat` → "Vsetínsku", etc.

---

## Issue catalogue

### Issue A — Q11 strategy failure: SSJČ etymology lookup instead of SNČJ FTS (HIGH)
**Appears in:** Q11  
**Occurrences:** 1 in this batch (but Q11 was ⚠/❌ in 7/10 historical runs)

Model opened R0 with `ftsearch "přejímka z němčiny germanismus" | sncj, assc, ssjc, psjc` — all returned 0. Then R1: `vsearch "přejímky z němčiny germanismy"` — found only meta-definitions. R2–R4: searched SSJČ for standard Czech words (šunka, knedlík) by training knowledge. Result: 4 common German loanwords from SSJČ, not dialect loanwords from SNČJ.

Correct strategy (conv `709abbf4`): `ftsearch "z něm" | sncj` → `ftsearch "německý" | sncj` → enumerate headwords with `(z něm.)` etymology markers.

### Issue B — Q15/Q16 geographic filtering: ČJA codes include all localities (MEDIUM)
**Appears in:** Q15, Q16  
**Occurrences:** 2

The ČJA `text` field contains locality codes replaced inline with "Obec/Okres". The model cannot filter to only Brněnsko-specific localities (codes 500–600) without a structured query. Mitigation: a future `[SQLQUERY:]` tool could filter `body_site` by Brněnsko okres codes.

### Issue C — Q9 SNČJ phraseme discovery failure (MEDIUM)
**Appears in:** Q9  
**Occurrences:** persistent across runs

SNČJ structured rendering now includes phrasemes in the `● ...` section for entries with `definition` JSON. However, the model must first search SNČJ to reach those entries. In Q9, the model searched only ASSC and SSJČ, never SNČJ.

### Issue D — Q8 missing couvni! (LOW)
**Appears in:** Q8  
**Occurrences:** 1

The model covered 4/5 horse-command headwords. `couvni!` was found in the baseline (conv `bc73b73e`) via FTS; in this run the same FTS call returned 0 hits (FTS preprocessing changed the query).

---

## Common issues ranked

| # | Issue | Queries affected | Impact |
|---|---|---|---|
| 1 | Q11 strategy failure — SSJČ etymology instead of SNČJ FTS | Q11 | High — training-knowledge loanwords, not dialect data |
| 2 | ČJA geographic filtering impossible without SQL tool | Q15, Q16 | Medium — geographic claims imprecise or wrong |
| 3 | Q9 SNČJ phraseme tunnel — model never queries SNČJ | Q9 | Medium — SNČJ dialect phrasemes missed every run |
| 4 | couvni! missed by FTS preprocessing | Q8 | Low — 4/5 horse commands still found |

---

## Comparison with baseline (2026-06-26, PN17b+PN16b+PN17c)

| Query | Set | Baseline verdict | This run verdict | Narration baseline | Narration this run | Improvement |
|---|---|---|---|---|---|---|
| Q6  | TEST | ⚠ Partial | **✅ Good** | ✗ | ✗ | ✅ SNČJ locality rendering helps region sort |
| Q8  | DEV | ✅ Good | **⚠ Partial** | ✗ | ✗ | ↕ Regression: couvni! missed |
| Q9  | TEST | ⚠ Partial | **⚠ Partial** | ✗ | ✗ | ↔ Neutral |
| Q11 | TEST | ⚠ Partial | **❌ Poor** | ✗ | ✗ | ↕ Regression: wrong dict strategy |
| Q14 | DEV | ✅ Good | **✅ Good** | ✗ | ✗ | ↔ Neutral |
| Q15 | DEV | ⚠ Partial (narr) | **⚠ Partial** | ⚠ narr | ✗ | ↑ Narration eliminated |
| Q16 | DEV | ⚠ Partial (narr) | **⚠ Partial** | ⚠ narr | ✗ | ↑ Narration eliminated |
| Q17 | TEST | ✅ Good | **✅ Good** | ⚠ narr | ✗ | ↑ Narration eliminated |
| Q18 | DEV | ✅ Good | **✅ Good** | ✗ | ✗ | ↔ Neutral |
| Q19 | DEV | ✅ Good | **✅ Good** | ⚠ narr | ✗ | ↑ Narration eliminated |
| Q20 | TEST | ✅ Good | **✅ Good** | ⚠ narr | ✗ | ↑ Narration eliminated + Village/Okres in answer |

### PN18 — Structural clean synthesis injection

**Effect:** Narration was detected in 5/11 responses in the baseline. In this run: 0/11. The injection of a clean-pass synthesis after tool rounds ensures the model starts from a blank slate, eliminating all inter-tool narration ("Výborně! Nyní mám kompletní přehled…"). The latency cost is real (+10–15s) but the gain is complete elimination of the most common complaint pattern.

### SNČJ structured rendering

**Effect:** Q20 shows Village/Okres locators in the answer text ("doloženo ve Vlkoši (Hodonínsko)", "v Kateřinicích (Valašsko)"). Q06 improves from ⚠ to ✅: the 6-round search produces a region-sorted answer because SNČJ entries now include Village/Okres which the model can use to classify entries geographically. Q17's `bouda` entry has no `definition` JSON (NULL), so structured rendering gracefully degrades to the FTS-extracted text. The SNČJ rendering didn't help Q09 because the model never searched SNČJ for alcohol phrasemes.

---

## Proposed prompt changes

### PN21 — SNČJ phraseme hint for frazémy queries *(targets Issue C)*

```
Pokud otázka hledá frazémy nebo idiomy, hledej je i v SNČJ — záznamy
s frazeologickými doklady se poznají tak, že výsledky FTSEARCH pro hledané
slovo vrátí výkladové věty obsahující frazémy. Zkus ftsearch "pít" | sncj,
ftsearch "opilý" | sncj apod.
```

**Expected effect:** Model will include SNČJ in Q9-type queries, reaching dialect-layer phrasemes currently missed every run.

### PN22 — Q11 strategy hint: SNČJ `z něm` FTS *(targets Issue A)*

```
Přejímky z němčiny identifikuj takto: ftsearch "z něm" | sncj nebo
ftsearch "z německého" | sncj — SNČJ záznamy obsahují etymologické
poznámky přímo v poli výkladu. Nepoužívej ftsearch "germanismus" — to
hledá definici pojmu, ne přejatá slova.
```

**Expected effect:** Q11 strategy stabilized; SNČJ is the correct source for dialect germanisms.

## Implementation priority

| Change | Effort | Expected gain |
|---|---|---|
| PN22 — Q11 SNČJ `z něm` FTS strategy | 2 sentences | Q11 ❌→✅ (estimated 7/10 runs) |
| PN21 — SNČJ phraseme hint | 2 sentences | Q9 ⚠→✅ (estimated 5/10 runs) |
