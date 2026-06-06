# TITLE

## Introduction

*add this later after finishing conclusion*

---

## Problem Statement

Retrieval-Augmented Generation (RAG) has become a foundational mechanism for grounding Large Language Models (LLMs) in external knowledge. However, existing frameworks continue to face two persistent challenges: semantic misalignment caused by domain-specific terminology and performance instability resulting from excessive contextual integration (An et al., 2024). Although recent agentic approaches introduce advanced augmentation and global context strategies, these methods often prioritize absolute accuracy without systematically evaluating their computational cost. As a result, the relationship between model scale, terminological methodology, and resource efficiency remains insufficiently understood.

---

## Research Statement and Objectives

This study proposes the Assistant Researcher AI (ARAI), a tunable agentic framework designed to optimize the balance between structural coherence and resource efficiency in retrieval-augmented environments. By implementing a dual-flow architecture—comprising an Offline-Indexing Flow (OIF) for sectional stabilization and an Online-Retrieval Flow (ORF) for terminological refinement—this work investigates how different terminological methodologies interact with model scale under constrained resource budgets.

The primary objective is to evaluate the impact of evidence-based, explanation-based, and hybrid methodologies on generation performance (ROUGE-L) and resource efficiency across three LLM architectures.

---

## Research Questions

### Primary Research Question

To what extent does aligning terminological methodology with model scale influence the balance between generation quality and resource efficiency in retrieval-augmented systems?

### Supporting Research Questions

- **Comparative:** How does model scale affect the optimal form of terminological conditioning required to maximize generation performance?
- **Efficiency-Oriented:** Can targeted terminological intervention achieve competitive performance while reducing contextual overhead compared to standard retrieval-augmented approaches?
- **Capacity-Oriented:** At what point does increasing contextual integration cease to produce meaningful performance gains?

---

## Literature Review

*Do it later, find papers from 2025 onward make it at least 10; add background knowledge i.e. BART, LLM*

### Retrieval Augmented Generation (RAG)

Retrieval-Augmented Generation (RAG) was introduced by Lewis et al. (2021) to mitigate the limitations of parametric-only knowledge in large language models (LLMs), where factual information is implicitly encoded in model weights and updating knowledge typically requires additional training. Instead of modifying parameters to incorporate new knowledge, RAG augments generation with non-parametric evidence retrieved from an external corpus. The framework couples a dual-encoder dense retriever, which maps queries and documents into a shared embedding space for similarity-based retrieval, with an encoder–decoder Transformer generator (BART). In the sequence formulation, the top-k retrieved documents are concatenated with the input query to condition the decoding process. By combining parametric representations with dynamically retrieved external evidence, RAG improves factual grounding and performance on knowledge-intensive tasks compared to standalone LLMs. Because generation is explicitly conditioned on retrieved documents, retrieval quality serves as a critical determinant of downstream performance.

### RAG with Long-Context LLMs

Because retrieval quality critically influences downstream generation, recent advances in long-context LLMs have prompted renewed investigation into how expanded context windows interact with retrieval-augmented architectures. Rather than replacing retrieval, long-context models enable RAG systems to incorporate larger volumes of retrieved evidence within a single forward pass, potentially reshaping the trade-offs between retrieval precision, context capacity, and computational efficiency.

Empirical analysis by Jin et al. (2024) reveals that increasing the number of retrieved passages in long-context RAG does not produce monotonic performance gains. While expanding the retrieval set initially improves answer quality through higher recall, performance subsequently deteriorates beyond a threshold. The authors attribute this degradation primarily to the accumulation of "hard negatives" — semantically similar but irrelevant passages that introduce noise into extended contexts. To mitigate this effect, they propose both training-free retrieval reordering strategies that exploit positional biases and training-based approaches, including implicit fine-tuning on noisy inputs and supervised intermediate-reasoning fine-tuning to improve relevance discrimination. These findings indicate that long-context capacity alone does not resolve retrieval-induced noise.

Complementing this mechanistic analysis, Leng et al. (2024) conduct a large-scale evaluation across 20 commercial and open-source LLMs, systematically varying total context length from 2,000 to 128,000 tokens (and beyond when supported). Their results show that although retrieving more documents can initially improve performance, only a small subset of state-of-the-art models maintain stable accuracy at extended contexts above 64k tokens. Performance frequently plateaus and declines beyond model-specific limits, with distinct failure modes emerging under long-context conditions.

Collectively, these studies demonstrate that RAG performance under expanded context windows exhibits diminishing returns and model-dependent task saturation thresholds. Increasing context capacity does not guarantee improved outcomes; instead, performance is governed by the balance between evidence coverage and contextual noise, highlighting the need for retrieval strategies that optimize not only recall but also information density and efficiency.

### Parametric RAG and Knowledge Integration Beyond Context Expansion

While prior work has focused on optimizing retrieval quality and managing long-context limitations within input-augmented architectures, Su et al. advance a complementary line of inquiry by questioning the fundamental efficacy of in-context retrieval itself. They argue that expanding the input sequence with multiple retrieved documents not only increases computational overhead and inference latency, but can also degrade performance in tasks requiring complex reasoning. Beyond efficiency concerns, the authors identify a structural limitation of conventional RAG: retrieval-augmented methods inject knowledge at the input level, whereas large language models encode and operationalize knowledge primarily within their parameters. Consequently, input-level augmentation may constrain the depth of knowledge integration achievable during inference.

To address these limitations, Su et al. propose Parametric RAG, a paradigm that embeds external knowledge directly into the model's parameter space rather than its context window. Instead of appending retrieved documents as textual input, the framework converts each document into a lightweight, document-specific parameter update applied to the feed-forward layers of the LLM. This document parameterization is achieved through a two-stage process. First, during an offline indexing phase, documents undergo augmentation via iterative rewriting and question–answer pair generation to promote robust knowledge abstraction. Second, a parameter-efficient adaptation technique (RoLA) encodes the augmented document into low-rank parameter matrices associated with the model's feed-forward networks.

At inference time, the system follows a Retrieve–Update–Generate pipeline. After retrieving the top-k relevant documents, it loads their corresponding low-rank parameter updates and merges them with the frozen base model weights to form a document-conditioned model. Response generation then proceeds without injecting lengthy textual evidence into the context window. By shifting knowledge integration from the input channel to the parametric channel, the framework reduces online computational cost while enabling deeper incorporation of external information.

Empirical results demonstrate that Parametric RAG consistently outperforms conventional in-context RAG across multiple LLM architectures in both effectiveness and efficiency. Moreover, the authors show that parametric integration is complementary to traditional retrieval augmentation, with hybrid configurations yielding further gains. Collectively, these findings suggest that scaling context length alone does not resolve retrieval-induced limitations; instead, architectural strategies that internalize external knowledge within model parameters represent a promising direction for advancing retrieval-augmented systems.

### Semantic Mismatch and Domain-Specific Terminology

A persistent challenge in retrieval-augmented systems arises from semantic mismatch between user queries and domain-specific corpora, particularly when queries contain jargon, abbreviations, or context-dependent terminology. Standard dense retrievers often struggle in these environments because embedding similarity alone may fail to resolve ambiguous or underspecified domain expressions, leading to suboptimal retrieval.

To address this issue, An et al. (2024) proposed Golden-Retriever, an agentic RAG framework that introduces a reflection-based query augmentation stage prior to retrieval. Instead of directly embedding the raw user query, the system first identifies domain-specific jargon and abbreviations, determines their contextual meaning using a predefined taxonomy, and retrieves extended definitions from a domain dictionary. The query is then augmented with these clarifications before being passed to the retriever. By enriching the semantic representation of the query, Golden-Retriever reduces ambiguity and improves alignment with industrial knowledge bases.

Empirical evaluation on a domain-specific question-answer dataset using three open-source LLMs demonstrates significant gains in retrieval and answer accuracy compared to traditional RAG pipelines. These findings suggest that retrieval failures in specialized domains often stem not from insufficient context length, but from semantic under-specification at the query level.

Existing research on retrieval-augmented generation has identified two primary performance constraints: degradation under extended context due to retrieval noise, and semantic mismatch caused by domain-specific terminology. While these studies clarify important failure modes, they largely evaluate performance under fixed methodological assumptions and prioritize accuracy as the dominant metric. Less attention has been given to how different terminological conditioning strategies interact with model scale and resource efficiency constraints.

### Summary of Research Gaps

**Lack of Methodological Differentiation in Terminological Conditioning**

Prior RAG studies typically vary retrieval size, context length, or model architecture, but treat retrieved content as homogeneous evidence. Limited work systematically compares alternative terminological methodologies—such as evidence-based retrieval, explanation-based augmentation, or hybrid approaches—to determine how different forms of contextual support influence downstream generation. As a result, it remains unclear which conditioning strategy best balances coherence and efficiency across models of varying scale.

**Insufficient Examination of Efficiency–Performance Trade-offs**

Although recent studies report diminishing returns and task saturation under extended contexts, performance is primarily measured using accuracy-based metrics. The interaction between generation quality and token consumption remains underexplored. In particular, few works quantify how resource reduction (e.g., token budget minimization) affects performance stability. This leaves unresolved the question of whether optimal RAG configurations should be evaluated solely on output quality or on efficiency-adjusted performance.

**Limited Analysis of Model-Scale Sensitivity**

Existing evaluations compare multiple LLMs but rarely analyze how optimal retrieval or conditioning strategies vary as a function of model scale. Larger models may tolerate contextual noise differently from mid-scale architectures, suggesting that retrieval and terminological interventions may not transfer uniformly across models. A systematic investigation into parameter–method alignment across architectures remains absent.

Addressing these gaps requires a framework that explicitly models the relationship between terminological conditioning, model scale, and resource allocation. This study proposes the Assistant Researcher AI (ARAI), a tunable agentic architecture designed to evaluate parameter–method optimization under efficiency constraints.

---

## Problem Formulation and Research Objectives

### Problem Formulation

Retrieval-augmented generation (RAG) systems must balance generation quality with computational resource consumption. While prior studies have examined retrieval size, context length, and semantic alignment independently, limited work has formalized how different terminological conditioning strategies interact with model scale under constrained token budgets.

We frame this as a parameter–method optimization problem:

> Given a model of scale *S* and a terminological conditioning strategy *M*, determine the configuration that maximizes generation performance while minimizing resource utilization.

- *S* represents model architecture and parameter scale (e.g., 11B vs 17B)
- *M* represents terminological methodology (evidence-based retrieval, explanation-based augmentation, or hybrid configuration)
- Performance is evaluated using ROUGE-L
- Resource utilization is quantified through a tokenizer-agnostic word-based proxy

The objective is not to maximize performance in isolation, but to identify the configuration at which performance gains remain proportional to token expenditure before entering a saturation regime.

### Conceptual Definitions

**Token Reduction Rate (TRR)**
Measures the relative change in resource consumption compared to a standardized retrieval-augmented baseline. Negative values indicate efficiency gains; positive values indicate overhead.

$$TRR = \frac{W_{config} - W_{anchor}}{W_{anchor}} \times 100$$

Where *W_anchor* = mean word count of the TF-base configuration (1,457 words).

**Task Saturation**
The regime in which increasing contextual input or methodological complexity fails to produce proportional improvements in evaluation metrics, often resulting in performance plateau or degradation.

**Efficiency Apex**
The configuration at which the ratio between generation performance and resource consumption is maximized prior to the onset of Task Saturation.

### Research Objectives

- Evaluate how evidence-based, explanation-based, and hybrid terminological methodologies influence ROUGE-L performance across varying model scales
- Quantify the trade-off between performance and resource efficiency using TRR
- Identify model-specific Efficiency Apex points to determine optimal parameter–method alignment in retrieval-augmented environments

### Research Questions (Formalized)

**Primary:** To what extent does strategic alignment between terminological methodology and model scale determine the location of the Efficiency Apex before Task Saturation occurs?

**RQ1 (Comparative):** How does model scale influence the optimal terminological conditioning strategy required to maximize ROUGE-L without entering a saturation regime?

**RQ2 (Evaluation-Oriented):** Can standalone terminological intervention (FF configuration) achieve superior resource return-on-investment compared to retrieval-augmented configurations (TF/TT) when evaluated through both ROUGE-L and TRR?

**RQ3 (Mechanistic):** At what level of contextual integration does marginal performance gain approach zero or become negative, indicating the onset of Task Saturation?

---

## Methodology

### System Overview

ARAI employs two agentic workflows: the **Offline-Indexing Flow (OIF)** and the **Online-Retrieval Flow (ORF)**.

### Offline-Indexing Flow (OIF)

Three storage components: DocumentMemory (DM), LongTermMemory (LTM), TermMemory (TM).

**Data Preprocessing:** docling library converts PDFs to markdown → sectional segmentation → chunks <10 words merged → metadata extraction → split-overlapping chunking (500 max tokens, 150-token overlap).

**Knowledge Storage:**
- *Enrichment:* ContextEnricher Agent (LLaMA 3.2-11B via AWS Bedrock) generates summaries; stored in LTM as raw / enrich / combo formats; embedded to 1024-dim vectors (BAAI/bge-m3)
- *Term extraction:* TermExtractor Agent (LLaMA4-Maverick-17B) extracts technical terms, acronyms, abbreviations, frameworks, algorithms; stored in DM with document_id

### Online-Retrieval Flow (ORF)

**Query Input:** Router Agent directs to TermManager, ContextRetriever, or Chat Agent.

**Term Retrieval:** TermManager Agent (LLaMA 3.2-11B) detects terms and confidence scores; full-text search on TM returning top 10 matches; filtered by validity criteria.

**Context Retrieval:** Query embedded (BAAI/bge-m3 1024-dim); vector search on LTM (raw_embedding used for evaluation); top 5 contexts reranked (cross-encoder/ms-marco-MiniLM-L6-v2).

**Response Generation:** Three terminological approaches for prompt construction:
- `evidence`: term evidence strings from TM
- `explanation`: term explanations from TM
- `both` (hybrid): evidence + explanation

Chat Agent (LLaMA 3.2-11B / Scout-17B / Maverick-17B) generates final response.

### Dataset

- 18 AI/ML papers (2024–2025)
- 33 manually curated questions: 11 acronym-type ("What does {term} stand for?"), 22 explanation-type ("What is {term}?")

### Evaluation Framework

**Performance:** ROUGE-L (F1, Precision, Recall) — primary metric; ROUGE-1 and ROUGE-2 secondary.

**Resource Efficiency:** TRR (word-count-based, tokenizer-agnostic); anchor = TF-base (1,457 words mean).

**Ablation design:** 36 configurations (3 models × 4 term methods × 3 retrieval settings).

| Variable | Configurations | Notation |
|---|---|---|
| Models (M) | Llama 3.2-11B, Scout-17B, Maverick-17B | L, S, M |
| Term Processing (T) | skip, evidence, explanation, hybrid | base, evid, expl, hyb |
| Context Retrieval (C) | True, False | T, F |
| Reranking (R) | True (only when C=True), False | T, F |

**Baseline anchor:** TF-base (retrieval on, reranking off, term processing skip).

**Infrastructure:** AWS Bedrock (Oregon, us-west-2) + PostgreSQL (DM/LTM/TM) + NVIDIA GTX 4080 (embeddings and reranking).

---

## Finding

*Find examples X, y_true, y_pred*

### All Question Analysis (ROUGE-L F1 — peak values)

| Model | Best F1 | Configuration |
|---|---|---|
| Llama3.2-11B | 0.6959 | TT-evidence |
| Scout-17B | **0.7199** | TF-hybrid |
| Maverick-17B | 0.6802 | TF-explanation |

### Comprehensive Performance Analysis

- **Scout-17B**: most balanced; highest overall F1 (0.7199, TF-hybrid)
- **Llama3.2-11B**: prefers evidence methodology; highest precision (0.7525, FF-evidence)
- **Maverick-17B**: best recall (0.8478, TF-explanation); shows task saturation in TT configs (F1 growth −0.49% to −2.38%)

### Acronym-Related Questions

Near-perfect performance across all models. Scout-17B achieves perfect 1.0 across F1, Precision, Recall (TF-hybrid).

### Explanation-Related Questions

Greater variability; stronger model–method alignment dependency. Llama3.2-11B best F1 (0.6008, TT-evidence); Maverick-17B best recall (0.7716, TF-explanation).

### Token Consumption and Resource Analysis

| Configuration | Methodology | Mean words (±std) | TRR | Peak F1 |
|---|---|---|---|---|
| FF | evidence | 99 ± 38 | −93.21% | 0.6604 (M) |
| FF | hybrid | 223 ± 89 | −84.69% | 0.6434 (L) |
| TF | hybrid | 1675 ± 489 | +14.96% | **0.7199 (S)** |
| TF | base (anchor) | 1457 ± 492 | 0.00% | 0.665 (S) |
| TT | hybrid | 1677 ± 487 | +15.10% | 0.693 (S) |

**Efficiency Apex:** FF-evidence (Llama3.2-11B) — 0.7525 precision at −93.21% token reduction.
**Performance Apex:** TF-hybrid (Scout-17B) — 0.7199 F1 at +14.96% token overhead.
**Task Saturation:** TT configurations for Maverick-17B — negative F1 growth despite maximum token cost.

---

## Discussion

### Parameter-Method Optimization Heuristic

- **Llama3.2-11B (small):** evidence methodology as general stabilizer; explanation for acronym definitional tasks
- **Scout-17B (intermediate):** hybrid methodology; balances precision and recall; highest overall F1
- **Maverick-17B (large):** explanation methodology; specialized in recall; susceptible to task saturation

### The Efficiency Apex

FF-evidence configuration: −93.21% TRR, ±38 words standard deviation (vs. ±486–492 for retrieval configs). Highest resource ROI for precision-constrained deployments.

### Task Saturation and Diminishing Returns

Most evident in Maverick-17B TT-hybrid: +15.10% token cost, −2.38% F1, −3.32% precision. Attributed to contextual redundancy reducing attention signal-to-noise ratio.

### Domain-Specific Task Analysis

- **Acronym tasks:** mature capability; TermManager stabilizes already strong performance
- **Explanation tasks:** highly sensitive to model–method alignment; evidence favors small models, explanation favors large models

### Conclusion: ARAI Tunable Framework

Two operational poles:
- **Efficiency Apex (FF-evidence):** resource-constrained environments, maximum precision, predictable latency
- **Performance Apex (TF-both):** comprehensive accuracy, Scout-17B as ideal balanced performer

---

## Conclusion

*add this later after finishing literature review*

---

## References

Lewis, P., Perez, E., Piktus, A., Petroni, F., Karpukhin, V., Goyal, N., Küttler, H., Lewis, M., Yin, W.-t., Rocktäschel, T., Riedel, S., & Kiela, D. (2021). Retrieval-augmented generation for knowledge-intensive NLP tasks. arXiv. https://doi.org/10.48550/arXiv.2005.11401

Jin, B., Yoon, J., Han, J., & Arık, S. Ö. (2024). Long-context LLMs meet RAG: Overcoming challenges for long inputs in RAG. arXiv. https://doi.org/10.48550/arXiv.2410

Leng, Q., Portes, J., Havens, S., & Zaharia, M. (2024). Long context RAG performance of large language models. arXiv. https://doi.org/10.48550/arXiv.2411.03538

An, Z., Fu, Y. C., Chu, C. C., Li, Y., & Du, W. (2024). Golden-Retriever: High-fidelity agentic retrieval augmented generation for industrial knowledge base. arXiv. https://doi.org/10.48550/arXiv.2408.00798

Su, W., Tang, Y., Ai, Q., Yan, J., Wang, C., Wang, H., Ye, Z., Zhou, Y., & Liu, Y. (2025). Parametric retrieval augmented generation. arXiv. https://doi.org/10.48550/arXiv.2501.15915
