---
name: hte
description: >
  Human Token Equivalent (HTE) estimator for written documents. Use this skill whenever
  the user wants to estimate how many cognitive "tokens" a human would have consumed
  to produce a piece of writing — articles, reports, essays, technical docs, blog posts,
  analyses, or any text artifact. Triggers include: "estimate the cognitive cost of this
  document", "how much thinking went into this", "HTE analysis", "how many human tokens",
  "estimate human effort on this article", or any request to quantify the mental work
  behind a written output. The skill produces a structured HTE report with scored signals,
  formula breakdown, and confidence assessment. Use it even when the user doesn't say "HTE"
  — if they're asking about the cognitive or intellectual effort behind a piece of writing,
  this skill applies.
---

# HTE — Human Token Equivalent Estimator (Documents)

This skill estimates the total cognitive "tokens" a human would have consumed to produce
a document, working backward from the document's observable properties.

The model has three components:
- **HTE_output** — the direct output (what was written)
- **HTE_process** — the hidden process (drafts, revisions, thinking-on-paper)
- **HTE_input** — the consumed input (research, reading, background knowledge activation)

All parameters are AI-determined from cognitive science literature. No human scores any rubric.
Read `references/parameters.md` before scoring — it contains the full rubrics, formula, and
empirical anchors.

---

## Workflow

### Step 1 — Acquire the Document

The document may arrive as:
- Direct text in the conversation
- An uploaded file (read it)
- A URL (fetch it)

If none of the above, ask the user to provide the document before proceeding.

### Step 2 — Read the Parameters Reference

Read `references/parameters.md` now. It contains the rubrics for D and B scoring,
the R_type table, reference counting rules, and the full formula. Do not score
from memory — always re-read the reference to ensure consistency across runs.

### Step 3 — Extract Signals

Analyze the document and determine the following. For each signal, record your
reasoning — the reasoning is part of the output.

**L — Output Token Count**
Count words in the document. Apply the token ratio from parameters.md based on
primary language (English ×1.3, Chinese ×0.6, mixed ×0.95).

**T — Document Type**
Classify the document into one of the types in the R_type table. Base this on
the document's *function*, not its format. Explain your classification.

**D — Depth Score (0–5)**
Apply the depth rubric from parameters.md. Score the highest level of reasoning
*consistently sustained*, not just the peak moment. State: the score, the key
evidence from the text that determined it, and any uncertainty.

**B — Breadth Score (0–5)**
Apply the breadth rubric from parameters.md. Count domains by asking whether a
specialist in each field would have needed to review that section. State: the
score, the domains identified, and any uncertainty.

**R — Reference Density**
Count informational dependencies using the counting rules in parameters.md.
List explicit and implicit references separately.

### Step 4 — Calculate HTE

Apply the formula from parameters.md:

```
HTE_output  = L
HTE_process = L × R_type × (1 + 0.20×D + 0.15×B)
HTE_input   = R × 3000 + B × 5000
HTE_total   = HTE_output + HTE_process + HTE_input
```

Show the arithmetic. Do not round intermediate values.

### Step 5 — Assess Confidence

Determine confidence level (High / Medium / Low) per the table in parameters.md.
Identify the primary source of uncertainty if not High.

### Step 6 — Output the Report

Use the report template below exactly. This consistent format enables comparison
across documents and iteration of the model over time.

---

## Report Template

```
# HTE Analysis Report

**Document:** [title or first line, truncated to 60 chars]
**Estimated type:** [type from R_type table]
**Analysis date:** [today's date]

---

## Signal Extraction

| Signal | Value | Reasoning |
|--------|-------|-----------|
| L (output tokens) | X | [word count] words × [ratio] |
| T (document type) | X | [one sentence rationale] |
| D (depth, 0–5) | X | [key evidence: quote or feature] |
| B (breadth, 0–5) | X | [domains identified, listed] |
| R (reference density) | X | [X explicit + X implicit = X total] |

---

## HTE Calculation

| Component | Formula | Tokens |
|-----------|---------|--------|
| HTE_output | L | X |
| HTE_process | L × [R_type] × (1 + 0.20×[D] + 0.15×[B]) | X |
| HTE_input | [R] × 3000 + [B] × 5000 | X |
| **HTE_total** | | **X** |

---

## Interpretation

- **Compression ratio:** [HTE_total / L]× — for every token in the final document,
  approximately [ratio] tokens of human cognitive work went into producing it.
- **Reading-time equivalent:** [HTE_total / (250×1.3)] minutes — how long it would
  take a person to *read* the equivalent cognitive workload at normal reading speed.
- **Relative scale:** [one sentence contextualizing the number — e.g., "comparable
  to producing a well-researched 800-word blog post" or "exceeds typical meeting prep"]

---

## Confidence

**Level:** [High / Medium / Low]
**Primary uncertainty:** [what drove the uncertainty, or "N/A" if High]
**Sensitivity note:** [e.g., "If D were 4 instead of 3, HTE_total would increase by ~15%"]
```

---

## Design Notes (for future iteration)

This skill is v0.1, focused on documents only. Known limitations:

- **Latent tokens excluded by design** — the "shower thought" component (background
  cognition that happens away from the keyboard) is not modeled. This makes HTE
  a lower-bound estimate.
- **Individual variance ignored by design** — all parameters use human population
  averages. An expert writes a technical document with lower HTE than a novice;
  this model does not distinguish.
- **Language support** — currently tuned for English and Chinese. Other languages
  use the mixed ratio as a fallback.
- **No temporal signal** — the model doesn't use time-on-task. If time data is
  available (e.g., document creation timestamps, edit history), note it as a
  supplementary data point but do not incorporate it into the formula until the
  model is extended.

When evaluating outputs, the key question is: does the HTE_total feel intuitively
plausible given the document? The compression ratio is the most human-readable
sanity check — most documents should fall between 5× and 30×. Outliers deserve
scrutiny.
