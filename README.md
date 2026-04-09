# HTE Skill — Human Token Equivalent Estimator

> **How much human thinking does a document represent?**

HTE (Human Token Equivalent) is a model that estimates the cognitive work a human consumed to produce a written document — by analyzing the document itself, with no human involvement in scoring or parameter setting.

The name is a deliberate parallel to AI token consumption. Just as LLMs measure compute cost in tokens, HTE measures human cognitive cost in an equivalent unit.

---

## The Model

Human cognitive work on a document has three components:

```
HTE_total = HTE_output + HTE_process + HTE_input
```

| Component | What it measures | How it's estimated |
|-----------|-----------------|-------------------|
| **HTE_output** | The final document itself | Direct token count |
| **HTE_process** | Drafts, revisions, thinking-on-paper | Output × draft multiplier (by document type) |
| **HTE_input** | Research, reading, domain knowledge activation | Reference count × reading cost + breadth × background cost |

Parameters are anchored to cognitive science literature (reading speeds, revision ratios, information processing bandwidth) — no manual tuning.

### Signals extracted from the document

The AI scores five signals directly from the text:

| Signal | Description |
|--------|-------------|
| **L** | Output token count |
| **T** | Document type → determines draft multiplier |
| **D** | Depth (0–5): layers of reasoning embedded |
| **B** | Breadth (0–5): distinct knowledge domains engaged |
| **R** | Reference density: informational dependencies on external sources |

### Formula

```
HTE_output  = L
HTE_process = L × R_type × (1 + 0.20×D + 0.15×B)
HTE_input   = R × 3000 + B × 5000
```

Where `R_type` is a per-document-type draft multiplier (1.5× for news briefs → 8× for academic papers).

---

## Example Outputs

Three documents analyzed during development:

| Document | Type | HTE_total | Compression ratio |
|----------|------|-----------|------------------|
| Short analytical blog post (270 words, CN) | Blog | ~19,000 | 117× |
| Technical deep-dive: Transformer attention (390 words, EN) | White paper | ~45,000 | 89× |
| News brief: product announcement (43 chars, CN) | News | ~5,100 | 197×* |
| API specification doc | Technical doc | ~31,000 | 27× |
| Strategic project kickoff (立项文档) | Strategic analysis | ~64,000 | 53× |

*The very high ratio for the news brief is a known model artifact: the fixed background-reading cost (`B × 5000`) dominates extremely short documents. See known limitations below.

---

## What This Is (and Isn't)

**This is useful for:**
- Comparing the cognitive cost of different document types
- Understanding where human effort actually goes (research vs. writing vs. revision)
- Grounding intuitions about "how hard was this to write?"

**This is not:**
- A precise measurement — it's an order-of-magnitude estimate with ~±40% uncertainty
- Individual-specific — all parameters use human population averages
- A complete model of human cognition — latent thinking (shower thoughts, subconscious processing) is not included by design

**The AI as judge principle:** All scoring (D, B, R) is performed by AI using rubrics derived from cognitive science literature. No human manually sets parameters or scores documents. This ensures consistency across runs and removes subjective bias from the evaluation.

---

## Why This Matters

When an AI produces a document, its token cost is observable and logged. When a human produces the same document, the cognitive cost is invisible.

HTE makes that cost visible — and comparable.

It also reveals a structural asymmetry: human cognitive cost is dominated by **HTE_input** (domain knowledge acquisition), while AI cost is dominated by **context injection** (what you put in the prompt). For the same document, humans "pay" before writing; AIs "pay" at inference time.

---

## Status

**v0.1 — Documents only.** This skill is actively being iterated on.

Current scope: written documents (articles, reports, API specs, strategy docs, blog posts).

Planned expansions:
- Code (using commit history as process signal)
- Presentations
- Meeting transcripts / verbal output
- AI vs. Human cost comparison mode

---

## Using the Skill

This is a [Cowork](https://claude.ai) skill for Claude. Install `hte.skill` via the Cowork plugin interface, then invoke it by asking Claude to run an HTE analysis on any document.

**Trigger phrases:**
- "HTE 分析这篇文章"
- "Run an HTE analysis on this document"
- "How much human cognitive work does this represent?"
- "Estimate the human token cost of this"

**What you get back:**

```
# HTE Analysis Report

Signal Extraction table  →  L, T, D, B, R with reasoning
HTE Calculation table    →  output + process + input = total
Interpretation           →  compression ratio, reading-time equivalent, relative scale
Confidence               →  level + primary uncertainty + sensitivity note
```

---

## File Structure

```
hte-skill/
├── SKILL.md                    # Main skill instructions
├── references/
│   └── parameters.md           # Cognitive science anchors, rubrics, formula
└── evals/
    └── evals.json              # Test cases for iterative improvement
```

---

## Background

This project started from a simple question: *if AI thinking costs tokens, how do you count the tokens a human spends thinking?*

Since human cognition is not directly observable (unlike LLMs, humans don't emit their chain-of-thought as text), the model works backwards from outputs — inferring cognitive cost from what the document itself reveals about the thinking that produced it.

The HTE model is designed to improve through use. Each document analyzed is a data point that can refine the scoring rubrics and parameters over time.
