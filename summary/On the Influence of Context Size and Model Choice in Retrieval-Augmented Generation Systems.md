# On the Influence of Context Size and Model Choice in Retrieval-Augmented Generation Systems

**Source:** `raw/On the Influence of Context Size and Model Choice in Retrieval-Augmented Generation Systems.pdf`
**Authors:** Juraj Vladika, Florian Matthes
**Publisher / Venue:** arXiv (Technical University of Munich)
**Year:** 2025
**Summarized:** 2026-06-05

## Research Problem / Objective

Despite the increasing industrial adoption of RAG systems, there is a lack of systematic studies of their core components — particularly the optimal number of context snippets, the choice of base LLM reader, and the choice of retrieval method. Most existing work uses short factoid QA tasks where one snippet is assumed to contain the answer. This paper fills the gap by studying long-form QA where a good answer must synthesize multiple or all retrieved snippets, across two domains (biomedical and encyclopedic).

## Methodology

**System:** Standard RAG pipeline with a retriever and a reader. Zero-shot prompting throughout. The reader is given a question and a list of k context snippets and asked to generate an answer.

**Experiment 1 — Gold Snippets (Context Size):** Gold snippets provided for k ∈ {0, 1, 3, 5, 10}. Tests how QA performance changes as more context is added.

**Experiment 2 — Closed Retrieval:** Retrieval from a small knowledge base (~8,000 PubMed documents) using semantic search (S-PubMedBERT-MS-MARCO, cosine similarity). k ∈ {1, 3, 5, 10}.

**Experiment 3 — Open Retrieval:** Retrieval from large knowledge bases — PubMed MEDLINE 2022 snapshot (~10.6M abstracts filtered to 10-year window) for BioASQ; Wikipedia API for QuoteSum. Compares BM25 vs. semantic search. k ∈ {1, 3, 5, 10, 15, 20, 30}.

**Models (8 total):**
- Large: GPT-3.5 Turbo, GPT-4o, Mixtral (8x7B), LLaMA 3 (70B)
- Small: Mistral-7B, Gemma 1 (7B), LLaMA 3 (8B), Qwen 1.5 (7B)

All instruction-tuned. Temperature = 0, max 512 output tokens.

## Dataset

- **BioASQ-QA** (Task 10b, 2023 version): 1,130 biomedical summary questions paired with human-selected PubMed evidence snippets and expert-written "ideal answers." Long-form responses summarizing all snippets.
- **QuoteSum**: 805 encyclopedic questions (geography, history, arts, technology) paired with up to 8 Wikipedia passages and human-written semi-extractive answers.

## Evaluation Metrics

- **ROUGE-L** (lexical overlap, longest common subsequence)
- **BERTScore** (semantic similarity via BERT embeddings)
- **Entailment %** (NLI using DeBERTa-v3-tasksource: generated answer as premise, reference answer as hypothesis; higher = generated answer entails reference)
- Additional: METEOR, cosine similarity of sentence embeddings (all-mpnet-base-v2)

## Findings

**Context Size (Gold Snippets):**
- All models show a large jump from 0 to 1 snippet, then steady improvement from 1 to 10 snippets.
- No model shows saturation or decline within 10 gold snippets on these datasets.
- In open retrieval setting (up to 30 snippets), performance peaks around 15–20 snippets and then stagnates or slightly declines at 30, confirming context saturation and "lost in the middle."

**Model Performance:**
- **Biomedical (BioASQ):** Mixtral (8x7B) and Mistral-7B achieve the strongest performance, substantially outperforming GPT-4o and LLaMA 70B on entailment metrics. Qwen 1.5 (7B) is competitive with much larger models. Model size is not the most important factor — domain-specific knowledge matters more.
- **Encyclopedic (QuoteSum):** GPT-4o and LLaMA 3 (70B) perform best. Mixtral underperforms on this task despite leading on BioASQ.
- Gemma (7B) performs poorly on QuoteSum (BERTScore collapses, ROUGE and cosine near-zero at 0 snippets), suggesting poor instruction following without context.

**Closed vs. Open Retrieval:**
- Performance drops substantially from gold snippets to closed retrieval (8k-doc corpus), and further in open retrieval (10M-doc corpus).
- Open retrieval is the most challenging setting: even with 15–20 snippets, models don't reach gold-snippet performance levels.

**BM25 vs. Semantic Search:**
- BM25 yields slightly better final QA performance than semantic search in open retrieval (BioASQ).
- BM25 optimizes for precision (keyword match → relevant documents), while semantic search provides broader but noisier coverage.

**Internal vs. External Knowledge Conflict:**
- In open retrieval, GPT-4o and Mixtral achieve better QA scores with 0 snippets (internal knowledge) than with up to 10 imperfect retrieved snippets. Only after 15+ snippets does RAG surpass internal knowledge. This is because poorly retrieved snippets that don't address the question instruct the model to generate incomplete answers, while unconstrained internal knowledge is often more informative.

## Conclusion / Discussion

In long-form QA RAG, performance improves steadily up to ~15 context snippets but stagnates or declines beyond that (context saturation). Different LLMs excel in different domains — domain-specific knowledge of the model matters more than model size for specialized tasks like biomedical QA. BM25 retrieval outperforms semantic search for precision-critical domains. Open-domain retrieval from large corpora remains highly challenging with significant performance gaps vs. gold context.

## Limitations

- Only two datasets tested; findings may not generalize universally.
- Zero-shot evaluation only; few-shot would likely lead to more uniform model performance.
- Automated metrics (ROUGE, BERTScore, NLI) have known weaknesses; human evaluation was not feasible.
- Models tested reflect June 2024 LLM landscape; many successors have since been released.

## Future Work

- Effects of query expansion methods (HyDE, query rewriting) on RAG performance.
- Evidence re-ranking strategies to improve open-domain retrieval quality.
- Knowledge conflict between internal LLM knowledge and retrieved context.
