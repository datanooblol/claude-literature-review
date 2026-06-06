# Literature Review Version History

| Version | Date | Summary of Changes |
|---------|------|--------------------|
| v0 | 2026-06-06 | First integrated revision — thematic organization; adds 10 new 2025 papers to original 5 citations |
| v1 | 2026-06-06 | Argument-driven rewrite — restructured to build sequentially toward all three research gaps (terminological conditioning, efficiency–performance trade-off, model-scale sensitivity) |

---

## v0 — 2026-06-06

**File:** `original_idea/literature_review_v0.md`

**Sections:**
- 2.1 RAG Foundations
- 2.2 Context Length, Retrieval Quantity, and Diminishing Returns
- 2.3 Semantic Mismatch and Pre-Retrieval Query Conditioning
- 2.4 Post-Retrieval Context Quality and Sufficiency
- 2.5 Parametric Knowledge Integration
- 2.6 Retrieval Design: Chunking, Indexing, and Best Practices
- 2.7 Summary of Research Gaps

**Papers cited:** 15 total — Lewis (2021), Jin (2024), Leng (2024), An (2024), Su (2025), Du (2025), Roberts (2025), Vladika & Matthes (2025), Joren et al. (2025), Li & Ramakrishnan / Oreo (2025), Lin et al. / REFRAG (2025), Merola & Singh (2025), Li et al. / Enhancing RAG (2025), Bansal et al. / qTTT (2025)

**Key changes vs. original paper placeholder:**
- Original paper had only 4 sections (RAG, Long-Context RAG, Parametric RAG, Semantic Mismatch) citing 5 papers
- v0 expanded to 6 thematic sections and 15 citations by integrating all 11 new summary/ papers
- Research gaps updated to cite specific 2025 papers as supporting evidence

**Rationale:** Initial integration pass — organize new papers thematically to prove coverage. Not yet structured to argue toward the three research gaps in a linear, reader-facing way.

---

## v1 — 2026-06-06

**File:** `original_idea/literature_review_v1.md`

**Sections:**
- 2.1 RAG Foundations and the Limits of In-Context Injection
- 2.2 Context Saturation: Empirical and Theoretical Evidence
- 2.3 Terminological Conditioning: Form Over Quantity
- 2.4 Retrieval Design and Efficiency Feasibility
- 2.5 Model-Scale Sensitivity to Contextual Strategy
- 2.6 Summary of Research Gaps

**Papers cited:** Same 15 as v0; placement and emphasis shifted.

**Key changes vs. v0:**
- Parametric RAG (Su et al.) moved from standalone section 2.5 to 2.1 as a structural contrast, establishing early that in-context injection has architectural limits
- New dedicated Section 2.3 makes terminological conditioning form (evidence/explanation/hybrid) the central thesis-building section, supported by Li et al. Enhancing RAG (Contrastive ICL) and Joren et al.
- New dedicated Section 2.5 covers model-scale sensitivity (Vladika, Roberts, qTTT) — previously scattered across multiple sections
- Section 2.4 merges retrieval design + efficiency feasibility (Oreo, REFRAG) to show efficiency gains are achievable, framed around TRR/Efficiency Apex
- Each section now closes with an explicit "this establishes X" sentence that advances the gap argument
- Research gaps (2.6) now directly cite the sections above rather than restating the literature independently

**Rationale:** Argument-driven restructure. A reader reaching 2.6 should already feel all three gaps without needing them explained — the sections build the case sequentially.
