---
title: "dictpert — Eval Reports"
description: "Evaluation reports for dictpert, a RAG-powered Czech dialectology assistant. Each run tests a set of 11 queries against the 6-dictionary backend using claude-haiku-4-5."
---

# dictpert — Eval Reports

Evaluation reports for [dictpert](https://github.com/martinpovolny/dictpert), a RAG-powered assistant for Czech dialect dictionaries. Each eval runs 11 answerable queries against six dictionary backends (ČJA, SNČJ, SPJMS, SPJČ, PSJČ, ASSC) and scores answers as ✅ Good / ⚠ Partial / ❌ Poor.

**Model:** claude-haiku-4-5 / Anthropic  
**Query set:** 11 answerable queries from Czech dialectology; split into TEST (5 honest-benchmark queries) and DEV (6 queries with prompt examples).

---

## Runs

| Date | Name | Changes | TEST | DEV | Total | Cost |
|---|---|---|---|---|---|---|
| 2026-06-26 | [PN18 + SNČJ structured rendering](2026-06-26-pn18-sncj-struct/) | Structural narration injection (PN18), SNČJ locality rendering (Village/Okres), defeatist opener ban (PN20) | 3✅/1⚠/1❌ | 3✅/3⚠/0❌ | **6✅/4⚠/1❌** | $1.12 |

---

## Eval methodology

Each run:
1. All 11 answerable queries executed via `scripts/test_prompt.py --dict-mode --save`
2. Chat transcripts exported to PDF via Playwright
3. Per-query metrics computed: latency, rounds, lookup counts, zero-result %, narration leakage
4. Verdict assigned: ✅ Good (grounded, complete), ⚠ Partial (gaps or geography errors), ❌ Poor (wrong strategy or hallucination)

**TEST set** (Q6, Q9, Q11, Q17, Q20) — no specific examples from these queries appear in the prompt; used for honest before/after comparisons.  
**DEV set** (Q8, Q14, Q15, Q16, Q18, Q19) — prompt contains specific headword examples drawn from these queries; improvements may reflect direct hint-following.
