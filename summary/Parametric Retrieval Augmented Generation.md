# Parametric Retrieval Augmented Generation

**Source:** `raw/Parametric Retrieval Augmented Generation.pdf`
**Authors:** Weihang Su, Yichen Tang, Qingyao Ai, Junxi Yan, Changyue Wang, Hongning Wang, Ziyi Ye, Yujia Zhou, Yiqun Liu
**Publisher / Venue:** arXiv (under conference review)
**Year:** 2025
**Summarized:** 2026-06-05

## Research Problem / Objective

All existing RAG methods inject external knowledge by appending retrieved passages directly to the LLM's input context (in-context knowledge injection). This paradigm has two core limitations: (1) long contexts increase computational overhead and degrade performance on complex reasoning tasks; (2) in-context injection only affects the online computation of KV pairs in attention layers, not the model's parameters where knowledge is actually stored — meaning LLMs can never use external knowledge as effectively as their internal knowledge. The paper asks whether external knowledge can be injected into LLM parameters effectively, efficiently, and flexibly at inference time.

## Methodology

The paper proposes **Parametric RAG (P-RAG)**, which injects external documents directly into the Feed-Forward Network (FFN) parameters of the LLM rather than into the input context.

**Offline Document Parameterization (two steps):**
1. *Document Augmentation*: Each document is rewritten once (diverse phrasing) and m QA pairs are generated from it, producing an augmented dataset D_i of (document, question, answer) triples.
2. *Parametric Document Encoding*: The augmented dataset is used to train a LoRA adapter (low-rank matrices A and B targeting FFN weight matrices) specific to that document. Training uses standard next-token prediction loss over the full concatenated (document, question, answer) sequence.

**Online Inference — Retrieve-Update-Generate (RUG) pipeline:**
1. *Retrieve*: BM25 retrieves top-k documents for the query.
2. *Update*: The low-rank matrices of retrieved documents are merged (summed with a scaling factor α) and applied to the LLM's FFN weights: W' = W + ΔW_merge.
3. *Generate*: The updated LLM answers the query from the original prompt only (no documents in context).

P-RAG can also be combined with standard in-context RAG ("Combine Both" baseline).

## Dataset

- **2WikiMultihopQA (2WQA)**: Multi-hop reasoning across Wikipedia passages (4 sub-task categories).
- **HotpotQA (HQA)**: Multi-hop reasoning, 2 categories (Bridge, Compare).
- **PopQA (PQA)**: Factual QA.
- **ComplexWebQuestions (CWQ)**: Multi-step web-based questions.

External corpus: Wikipedia dump from the DPR dataset. First 300 questions per sub-dataset used.

## Evaluation Metrics

F1 score on question-answering tasks (accounts for partially correct answers). Inference time (seconds per question) for efficiency analysis.

## Findings

- P-RAG outperforms all in-context RAG baselines (Standard RAG, DA-RAG, FLARE, DRAGIN) on most benchmarks and models (Qwen2.5-1.5B, LLaMA3.2-1B, LLaMA-3-8B).
- The performance gap over baselines is larger for bigger models (LLaMA-8B > LLaMA-1B), indicating larger models benefit more from parametric injection.
- Combining P-RAG with standard in-context RAG ("Combine Both") achieves the highest overall performance, showing the two paradigms are complementary.
- DA-RAG (augmented documents in context) consistently underperforms P-RAG, confirming that gains come from parametric injection itself, not document augmentation.
- P-RAG reduces inference time by 29–36% compared to Standard RAG on LLaMA-8B; multi-round methods (FLARE, DRAGIN) are 4–5× slower.
- Warm-up LoRA initialization (pre-trained on 600 task QA pairs) consistently outperforms random initialization.
- Both document rewriting and QA pair generation are necessary; removing either degrades performance, with QA generation being the more critical component.
- The choice of augmentation model (smaller, same-size, or larger LLM) has minimal impact on final performance.

## Conclusion / Discussion

Parametric RAG offers a new knowledge-injection paradigm that embeds external documents into LLM parameters via LoRA adapters trained offline. It reduces online computational cost (no long context), deepens knowledge integration (parametric space vs. attention computation), and is complementary to in-context methods. When the system serves many queries over a fixed corpus, the offline preprocessing cost becomes negligible compared to inference savings.

## Limitations

- Offline parameterization is computationally intensive; each document requires LoRA training.
- Storage cost is ~4.72 MB per document (LoRA matrices at 16-bit precision), substantially more than raw text.
- Parametric representations are tied to specific LLMs and cannot generalize across models.
- Current LoRA merge/load implementation has latency overhead (~0.32s) beyond the theoretical minimum.

## Future Work

- Improve computational and storage efficiency of the parameterization process for scalability.
- Develop universal, model-agnostic parametric representations.
- Extend information parameterization beyond RAG (e.g., parameterizing agent profiles and configurations in LLM-based agents).
