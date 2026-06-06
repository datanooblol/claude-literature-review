# Next Steps

*Last updated: 2026-06-06*

---

## Current State

| Artifact | File | Status |
|---|---|---|
| Paper summaries (11 papers) | `summary/` | Done |
| Synthesis sections (3 groupings) | `review_literature/` | Done |
| Literature review | `original_idea/literature_review_v1.md` | Done |
| Research objectives | `original_idea/research_objectives_v1.md` | Done |
| Methodology | `first_draft.md` (inline) | Evaluated, not yet revised |
| Finding | `first_draft.md` (inline) | Evaluated, not yet revised |
| Discussion | `first_draft.md` (inline) | Evaluated, not yet revised |
| Conclusion | `first_draft.md` (inline) | **Placeholder — not written** |
| Introduction | `first_draft.md` (inline) | **Placeholder — write last** |
| Final assembled paper | — | **Not yet created** |

---

## Priority Queue

### 1. Revise discussion → then write conclusion
Do in this order — conclusion summarizes the discussion, so revising discussion first prevents drift.

**Discussion revision (`discussion_v0.md` → `discussion_v1.md`):**
- Extract verbatim from `first_draft.md` as v0 baseline
- v1 changes:
  - Close the gap loop in each subsection (state which gap the finding addresses, what it implies for future work)
  - Replace Task Saturation attribution ("contextual redundancy...") with mechanistic grounding from Du et al. (2025) positional distribution bias + Bansal et al. (2025) score dilution
  - Add comparison of FF-evidence Efficiency Apex (−93.21% TRR) to Oreo (84–94% compression) and REFRAG (30.85× TTFT) — note ARAI achieves comparable compression through conditioning form rather than a post-retrieval module

**Conclusion (`conclusion_v0.md` → `conclusion_v1.md`):**
- v0 = extract the placeholder from `first_draft.md`
- v1 = full written conclusion using this five-element structure:
  1. Restate contribution: ARAI as first systematic comparison of terminological conditioning form under explicit efficiency constraints across model scales
  2. Finding 1 → Gap 1: conditioning form is model-scale dependent (evidence/hybrid/explanation maps to 11B/17B-Scout/17B-Maverick); consistent with Li et al. (2025) format-over-quantity result
  3. Finding 2 → Gap 2: FF-evidence achieves −93.21% TRR at competitive precision — Efficiency Apex is reachable through conditioning form alone, without post-retrieval compression modules
  4. Finding 3 → Gap 3: same parameter scale (17B) produces different optimal methods across architectures — scale sensitivity reflects training distribution + architecture, not parameter count alone (Vladika et al., 2025)
  5. Limitations + future work: small dataset (33 questions, single domain), ROUGE-L does not capture hallucination (Joren et al., 2025), scale and architecture confounded; future: cross-domain testing, hallucination metric, larger dataset

---

### 2. Revise methodology and finding — requires user input first

**Methodology (`methodology_v0.md` → `methodology_v1.md`):**
Needs answers to two questions before proceeding:
- Were any statistical significance tests applied to the 36-configuration results? If yes, which test and what threshold?
- What does "raw_embedding used for evaluation" mean exactly? Does evaluation bypass the cross-encoder reranker, or does it use the un-enriched (raw) chunk embeddings before enrichment?

Planned v1 changes once answered:
- Add statistical testing note (or explicitly acknowledge absence as a limitation)
- Add scale-architecture confound caveat for the 3-model comparison (11B vs. 17B Scout vs. 17B Maverick conflates parameter count with model family and training distribution)
- Clarify the raw_embedding sentence

**Finding (`finding_v0.md` → `finding_v1.md`):**
Needs user to provide:
- Qualitative examples: one example of `(question, y_true, y_pred)` per conditioning method for explanation-type questions, to replace the `*Find examples X, y_true, y_pred*` placeholder
- Full breakdown table for explanation-type questions only (evidence vs. explanation vs. hybrid × 3 models) — this is the head-to-head that most directly answers RQ1 and should be the centerpiece of the findings

Planned v1 changes once provided:
- Reframe Task Saturation mechanism (replace "contextual redundancy" with Du et al. + Bansal et al. grounding)
- Add Efficiency Apex comparison to Oreo/REFRAG with mechanism contrast
- Foreground explanation-task head-to-head as the primary finding (move before all-question analysis)

---

### 3. Write the introduction — last

Write only after conclusion is done. The introduction should:
- State the problem (semantic mismatch + efficiency–performance trade-off in domain-specific RAG)
- Preview the three gaps
- Describe ARAI briefly
- End with paper roadmap (section-by-section)

---

### 4. Assemble final_draft.md

Once all sections are versioned and revised, assemble a complete paper:

```
original_idea/final_draft.md
```

Assembly order:
1. Introduction (v1, to be written)
2. Literature Review → use `literature_review_v1.md`
3. Research Objectives → use `research_objectives_v1.md`
4. Methodology → use `methodology_v1.md` (pending)
5. Finding → use `finding_v1.md` (pending)
6. Discussion → use `discussion_v1.md` (pending)
7. Conclusion → use `conclusion_v1.md` (pending)
8. References → merge all unique citations from versioned sections

Add `final_draft` as a new artifact in `log/revisions.md` when created.

---

## Decisions Already Made

- Versioning convention: `original_idea/<section>_vN.md`; v0 = verbatim from first_draft.md
- Log: `log/revisions.md` tracks all versioned artifacts
- No section is ever overwritten — all vN files are permanent
- Conclusion is written before introduction
- ROUGE-L limitation (Joren et al. hallucination finding) acknowledged in metric scope note, not a reason to change the metric
- Scale-architecture confound (Scout vs. Maverick both 17B) is a limitation to acknowledge, not a redesign trigger
