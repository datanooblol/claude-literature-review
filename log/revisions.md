# Paper Section Revision History

---

## literature_review

| Version | Date | Summary of Changes |
|---------|------|--------------------|
| v0 | 2026-06-06 | First integrated revision — thematic organization; adds 10 new 2025 papers to original 5 citations |
| v1 | 2026-06-06 | Argument-driven rewrite — restructured to build sequentially toward all three research gaps |

### v0 — 2026-06-06

**File:** `original_idea/literature_review_v0.md`
**Sections:** 2.1 RAG Foundations · 2.2 Context Length and Diminishing Returns · 2.3 Semantic Mismatch · 2.4 Post-Retrieval Context Quality · 2.5 Parametric Knowledge Integration · 2.6 Retrieval Design · 2.7 Research Gaps
**Papers cited:** 14 unique (Lewis 2021, Jin 2024, Leng 2024, An 2024, Su 2025, Du 2025, Roberts 2025, Vladika 2025, Joren 2025, Li/Oreo 2025, Lin/REFRAG 2025, Merola 2025, Li/Enhancing RAG 2025, Bansal/qTTT 2025)
**Key changes:** Original paper had 4 sections citing 5 papers. v0 expanded to 6 thematic sections and 14 citations by integrating all 11 summary/ papers. Research gaps updated to cite specific 2025 papers.
**Rationale:** Initial integration pass — organize new papers thematically to prove coverage. Not yet structured to argue toward the three research gaps in a linear way.

### v1 — 2026-06-06

**File:** `original_idea/literature_review_v1.md`
**Sections:** 2.1 RAG Foundations and Limits of In-Context Injection · 2.2 Context Saturation: Empirical and Theoretical Evidence · 2.3 Terminological Conditioning: Form Over Quantity · 2.4 Retrieval Design and Efficiency Feasibility · 2.5 Model-Scale Sensitivity to Contextual Strategy · 2.6 Research Gaps
**Papers cited:** Same 14 as v0; placement and emphasis shifted.
**Key changes:** Parametric RAG moved to 2.1 as structural contrast. New Section 2.3 makes terminological conditioning form the central thesis-building section (Li et al. Contrastive ICL + Joren et al.). New Section 2.5 dedicates space to model-scale sensitivity (previously scattered). Section 2.4 merges retrieval design and efficiency feasibility. Each section closes with an explicit "this establishes X" sentence advancing the gap argument.
**Rationale:** Argument-driven restructure so the reader feels all three gaps before section 2.6 names them.

---

## research_objectives

| Version | Date | Summary of Changes |
|---------|------|--------------------|
| v0 | 2026-06-06 | Verbatim extract from first_draft.md — baseline before any revision |
| v1 | 2026-06-06 | Strengthened gap statements with empirical and theoretical anchors from literature_review_v1 |

### v0 — 2026-06-06

**File:** `original_idea/research_objectives_v0.md`
**Sections:** Research Statement and Objectives · Research Questions · Summary of Research Gaps · Problem Formulation · Conceptual Definitions (TRR, Task Saturation, Efficiency Apex) · Formal Research Objectives · Formalized RQs
**Key changes:** None — verbatim copy from first_draft.md. Preserved as permanent baseline.
**Rationale:** Establish a clean v0 before any revision so all changes are tracked.

### v1 — 2026-06-06

**File:** `original_idea/research_objectives_v1.md`
**Sections:** Same structure as v0.
**Key changes:**
- Gap 1: Added Li et al. (2025) Contrastive ICL as direct empirical precedent; reframed as extending a known high-impact finding into a specialized terminological setting
- Gap 2: Added mechanistic grounding — Du et al. positional bias and Bansal et al. score dilution explain why Task Saturation exists; Oreo/REFRAG compression results establish the Efficiency Apex is reachable
- Gap 3: Added Vladika domain-fit > model-size nuance; reframed as model-scale *and architecture* sensitivity, not purely parameter count
- Problem formulation: Added note that S (model scale) is confounded with model training distribution in the 3-model comparison
- Conceptual definitions, RQs, and formal objectives: Unchanged
- Added ROUGE-L limitation note referencing Joren et al. hallucination finding
**Rationale:** Strengthen the defensibility of all three gaps using specific citations from literature_review_v1 without changing the experimental design or metrics.
