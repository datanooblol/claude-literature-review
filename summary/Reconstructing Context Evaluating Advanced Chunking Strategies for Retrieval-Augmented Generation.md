# Reconstructing Context: Evaluating Advanced Chunking Strategies for Retrieval-Augmented Generation

**Source:** `raw/Reconstructing Context Evaluating Advanced Chunking Strategies for Retrieval-Augmented Generation.pdf`
**Authors:** Carlo Merola, Jaspinder Singh
**Publisher / Venue:** arXiv; accepted at Second Workshop on Knowledge-Enhanced Information Retrieval, ECIR 2025 (Springer LNCS)
**Year:** 2025
**Summarized:** 2026-06-05

## Research Problem / Objective

Traditional RAG pipelines split documents into fixed-size chunks before embedding, which fragments semantic context: individual chunks may lack information about which document they belong to or what they refer to (e.g., "The company's revenue grew by 3%" without specifying which company). Two advanced chunking techniques introduced in 2024 — late chunking and contextual retrieval — both aim to preserve global document context, but their comparative strengths and trade-offs in an integrated RAG workflow (not just retrieval benchmarks) remain unstudied. The paper evaluates them under the same experimental conditions.

## Methodology

Two research questions guide the study:

**RQ#1 — Early vs. Late Chunking:**
- **Early Chunking**: Segments document into chunks first, then embeds each chunk independently; mean pooling over token embeddings per chunk.
- **Late Chunking**: Embeds the entire document at token level first, then segments the resulting token embeddings into chunks using boundary cues, then mean pools. Preserves full document context without additional training.
- Both are tested with fixed-size (512 chars) and semantic segmentation (Jina Segmenter API) and with dynamic segmenters (simple-qwen-0.5 for structural boundary detection; topic-qwen-0.5 for topic-based segmentation via CoT reasoning).
- Embedding models: Stella-V5 (MTEB rank 5), Jina-V3 (rank 53), Jina-V2 (rank 123), BGE-M3 (rank 211).

**RQ#2 — Traditional Retrieval vs. Contextual Retrieval with Rank Fusion (ContextualRankFusion):**
- **Contextualization**: An LLM (Phi-3.5-mini-instruct, 4-bit quantized) generates a brief context summary for each chunk from the full document; this context is prepended to the chunk before embedding.
- **Rank Fusion**: Dense embeddings + BM25 sparse embeddings are combined with weighted rank fusion (dense:BM25 = 4:1 weight ratio), using Milvus vector database.
- **Reranking**: Jina Reranker V2 Base (cross-encoder) reranks the initial results before passing to LLM.

**Generation**: Phi-3.5-mini-instruct (4-bit) used as the reader for question answering (MsMarco dataset).

## Dataset

- **Retrieval Evaluation:** NFCorpus (medical information retrieval, long average document length). RQ#1 used the full dataset (~5,000 documents, 1,000 queries); RQ#2 used 20% subset (~300 documents, 50 queries) due to GPU memory constraints.
- **Generation Evaluation:** MsMarco (passage-level QA).
- **Hardware:** Nvidia RTX 4090 with 24GB VRAM.

## Evaluation Metrics

Retrieval: NDCG@5, MAP@5, F1@5, NDCG@10, MAP@10, F1@10. Generation: Not formally assessed beyond observing that generation differences were not notable.

## Findings

**Early vs. Late Chunking (RQ#1):**
- Late chunking generally matches or slightly outperforms early chunking in most model/dataset combinations — but not consistently.
- Exception: BGE-M3 on NFCorpus shows early chunking substantially outperforming late chunking (NDCG@5: 0.246 early vs. 0.070 late), suggesting late chunking's effectiveness is embedding-model-dependent.
- On MsMarco with Stella-V5, early chunking again outperforms late (NDCG@5: 0.630 vs. 0.503).
- Dynamic segmenters (topic-qwen) perform well with Jina-V3 but require 2× (simple-qwen) to 4× (topic-qwen) more processing time than fixed/semantic segmentation. They also produce inconsistent wording, reducing reliability.
- Fixed-window chunking is simpler and faster than semantic chunking with comparable performance.

**ContextualRankFusion vs. Traditional Retrieval (RQ#2):**
- Rank Fusion with contextualized chunks (FCC+RFR) consistently outperforms traditional retrieval (FCC+TR): e.g., Jina-V3 NDCG@5 improves from 0.312 to 0.317, NDCG@10 from 0.295 to 0.308.
- The reranking step is crucial — without it, rank fusion does not consistently improve results.
- For the weaker BGE-M3 model, rank fusion provides larger relative gains but absolute performance remains poor.
- Equal weighting of dense and BM25 vectors hurts retrieval; the 4:1 ratio (dense:BM25) is optimal.
- Contextual retrieval is computationally demanding: generating context for long documents requires up to 20GB VRAM and significantly slows indexing.

**Direct Comparison (Table 5, same 20% NFCorpus subset, Jina-V3, fixed-window):**
- ContextualRankFusion outperforms late chunking: NDCG@5 0.317 vs. 0.309; NDCG@10 0.308 vs. 0.294.

## Conclusion / Discussion

Neither late chunking nor contextual retrieval is a definitive solution to the chunking context problem. Contextual retrieval with rank fusion achieves better semantic coherence preservation but incurs high computational cost (LLM inference per chunk, high VRAM for long documents). Late chunking is computationally efficient and broadly applicable but may sacrifice relevance and completeness on certain embedding models/datasets. The choice depends on available resources and document characteristics.

## Limitations

- Small scale evaluation: RQ#2 limited to 50 queries / ~300 documents.
- Only one LLM (Phi-3.5-mini) tested for both context generation and QA.
- Generation quality differences not statistically notable.
- Only two datasets (NFCorpus, MsMarco) used.

## Future Work

Not mentioned in paper.
