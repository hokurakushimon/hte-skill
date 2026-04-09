# HTE Model Parameters Reference

This file contains the empirical anchors and scoring rubrics used by the HTE skill.
All parameters are derived from cognitive science literature or established industry data.
No human manual tuning — the AI applies these rubrics consistently as a neutral judge.

---

## 1. Human Cognitive Throughput Anchors

| Parameter | Value | Source / Rationale |
|-----------|-------|--------------------|
| Conscious information bandwidth | ~120 bits/sec | Nørretranders (1998), *The User Illusion* |
| Average adult reading speed (careful reading) | 200–250 words/min | Rayner et al. (2016), Psychological Science in the Public Interest |
| Average adult reading speed (skimming) | 400–600 words/min | Same |
| Dense technical reading speed | 100–150 words/min | Domain expert estimates, consistent with comprehension research |
| Average typing speed (net output) | 40–60 WPM | Dhakal et al. (2018), CHI |
| Token-to-word ratio (English) | ×1.3 | Empirical from OpenAI tokenizer across corpus types |
| Token-to-word ratio (Chinese) | ×0.6 | Chinese characters are denser per token |

## 2. Document Type → Draft Multiplier (R_type)

Draft multiplier represents: (total words produced including drafts) / (final word count).
Derived from writing process research and software engineering productivity studies.

| Document Type | R_type | Basis |
|---------------|--------|-------|
| News / brief / announcement | 1.5 | Low revision, time-pressured writing |
| Social post / short-form | 1.5 | High density editing but short total length |
| Blog post / essay / opinion | 2.5 | Typical blogger revision behavior |
| Internal report / memo | 3.5 | Multiple stakeholder review cycles |
| Technical documentation | 5.0 | Accuracy requirements drive high revision rate |
| Strategic analysis / white paper | 6.0 | Complex argumentation, extensive redrafting |
| Academic / research paper | 8.0 | Peer review, multiple full rewrites common |

Classification is by the document's *function*, not its length.
When ambiguous, use the higher multiplier (conservative estimate).

## 3. Reference / Source Reading Cost

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| avg_source_tokens | 3,000 tokens | Assumes author reads ~2,300 words per cited/referenced source (careful reading of one section or article abstract + key sections) |
| background_reading_factor | 5,000 tokens per breadth unit | Cost to acquire working familiarity with one new domain: ~3,800 words at careful reading pace |

These are *per-reference* and *per-domain* costs respectively.

## 4. Depth Scoring Rubric (D, scale 0–5)

Depth measures how many layers of reasoning and domain knowledge are embedded in the text.
Score based on the highest level consistently sustained throughout the document.

| Score | Label | Criteria |
|-------|-------|----------|
| 0 | Surface | Pure factual description, no analysis. "X happened on Y date." Bullet lists of facts. No causal reasoning. |
| 1 | Light | Basic interpretation. "X happened because of Y." Single-level cause-effect. No domain prerequisite needed. |
| 2 | Moderate | Multi-step reasoning. Reader needs some domain familiarity. Definitions used correctly. Compares tradeoffs. |
| 3 | Substantive | Expert-level vocabulary used precisely. Arguments built on multiple premises. Counterarguments acknowledged. |
| 4 | Deep | Original synthesis of ideas. Requires expert background to evaluate. Nested reasoning chains (A→B→C→D). Quantitative reasoning or formal logic present. |
| 5 | Exceptional | PhD-equivalent or practitioner-expert level. Novel frameworks proposed. Challenges established consensus with evidence. Irreducible complexity—cannot be summarized without loss. |

**Scoring rule:** If the document is inconsistent (shallow intro, deep analysis section), weight toward the deepest sustained section, discounted by proportion of the document it occupies.

## 5. Breadth Scoring Rubric (B, scale 0–5)

Breadth measures how many distinct knowledge domains are meaningfully engaged—not just mentioned.
"Meaningfully engaged" means the author had to know something about the domain to write it correctly.

| Score | Label | Criteria |
|-------|-------|----------|
| 0 | Single domain | One topic, one domain. No cross-references. A technical tutorial on one API. |
| 1 | Near-domain | Single topic with adjacent context. A business analysis that briefly mentions tech constraints. 1–2 domains. |
| 2 | Multi-aspect | Multiple facets of one topic, 2–3 domains. A product analysis covering UX + business + market. |
| 3 | Cross-domain | Synthesis across 3–4 distinct domains. E.g., economic policy + behavioral psychology + political science. |
| 4 | Wide-ranging | 4–5+ domains, each engaged substantively. Connections drawn between domains are non-obvious. |
| 5 | Synthetic | Encyclopedic in scope. Author must hold 5+ distinct domain models simultaneously. Rare—typically book-length thinking compressed into an article. |

**Scoring rule:** Count domains by asking "would a specialist in field X need to have reviewed this section?" Each "yes" counts as one domain. Surface name-drops (e.g., "like in physics…") do not count.

## 6. Reference Density (R) — Counting Rules

R is the count of *informational dependencies* the document has on external sources.
This is not just citation count—it includes implicit references.

**Count as 1 reference each:**
- Explicit citation (numbered, in-text, footnote)
- Named external work, study, or dataset ("the McKinsey report", "the 2023 OpenAI paper")
- Attributed statistic or data point ("70% of users…" — implies a source even if unnamed)
- Named real-world case study used as evidence

**Do NOT count:**
- Generic statements ("research shows" without specificity) — count as 0.3 each, round total
- Examples invented by the author for illustration
- Author's own prior work if context makes this clear

---

## 7. Full HTE Formula

```
L           = word_count × token_ratio
              (use 1.3 for English, 0.6 for Chinese, 0.95 for mixed)

HTE_output  = L

HTE_process = L × R_type × (1 + 0.20×D + 0.15×B)

HTE_input   = R × 3000 + B × 5000

HTE_total   = HTE_output + HTE_process + HTE_input
```

**Compression ratio** = HTE_total / L
**Human reading time equivalent** = HTE_total / (250 × 1.3) minutes
  (how long it would take to *read* the equivalent cognitive work at normal speed)

---

## 8. Uncertainty and Confidence

Report a confidence level for each run:

| Confidence | Condition |
|------------|-----------|
| High | Document type is clear, D and B scores are unambiguous, R is countable |
| Medium | One of D, B, or T is ambiguous; estimate is within ±40% |
| Low | Multiple signals are unclear, document is unusual; estimate is order-of-magnitude only |

Always report the primary uncertainty source (e.g., "D score is uncertain between 3 and 4").
