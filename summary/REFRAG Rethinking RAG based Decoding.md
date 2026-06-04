# REFRAG: Rethinking RAG based Decoding

**Source:** `raw/REFRAG Rethinking RAG based Decoding.pdf`
**Authors:** Xiaoqiang Lin, Aritra Ghosh, Bryan Kian Hsiang Low, Anshumali Shrivastava, Vijai Mohan
**Publisher / Venue:** arXiv (Meta Superintelligence Labs, National University of Singapore, Rice University)
**Year:** 2025
**Summarized:** 2026-06-05

## Research Problem / Objective

Processing long-context inputs in RAG introduces significant inference latency (TTFT scales quadratically with input length) and KV cache memory overhead. Generic LLM long-context approaches overlook RAG-specific structure: (1) retrieved passages are often uninformative and reused across queries; (2) retrieval encodings are pre-computed but discarded during decoding; (3) retrieved passages are semantically diverse due to deduplication, resulting in block-diagonal (near-zero cross-chunk) attention patterns. The paper argues that most attention computation over the RAG context is unnecessary and proposes a specialized efficient decoding framework.

## Methodology

**REFRAG (REpresentation For RAG)** replaces retrieved passage tokens with compressed chunk embeddings in the decoder input:

**Architecture:**
- A lightweight encoder (RoBERTa-base/large) processes each retrieved passage chunk C_i of k tokens into a single chunk embedding c_i.
- A 2-layer MLP projection maps c_i into the decoder's token embedding space: e^cnk_i = φ(c_i).
- The decoder (LLaMA-2-7B) receives question token embeddings + projected chunk embeddings, reducing input length by factor k.

**Training pipeline (three stages):**
1. **Reconstruction task**: Freeze decoder, train encoder + projection to reconstruct x_{1:s} from chunk embeddings. Curriculum learning is essential — training begins with single-chunk reconstruction and gradually increases chunk count.
2. **Continual Pre-training (CPT)**: Unfreeze decoder and train on next-paragraph prediction: given compressed context x_{1:s}, predict x_{s+1:s+o}. Also uses curriculum learning.
3. **Selective compression via RL**: An RL policy (GRPO-style with PPO clipping) selects which chunks to leave uncompressed. Policy takes chunk embeddings as input and sequentially selects T' chunks to expand. Reward is negative log-perplexity on the output tokens. Outperforms heuristic selection (compress low-perplexity or high-perplexity chunks).

**Inference:** Chunk embeddings are pre-computable and cacheable (embeddings are independent of the query). Only question tokens and chunk embeddings enter the decoder.

## Dataset

**Pre-training:** 20B token subset of Slimpajama (50% ArXiv, 50% Books domains).

**Evaluation (language modeling):** ArXiv, Books (Slimpajama held-out), PG19, Proof-pile.

**RAG downstream evaluation:** 16 QA and multi-choice tasks (MMLU, BoolQ, SIQA, PIQA, KILT datasets: HellaSwag, Winogrande, TQA, FEVER, NQ) from 5 domains. Retrieval corpus: 400M Wikipedia + CommonCrawl passages with DRAGON+ retriever.

**Multi-turn conversation:** TopiOCQA, ORConvQA, QReCC.

**Summarization:** Arxiv and PubMed long document summarization dataset.

## Evaluation Metrics

Log-perplexity on output tokens; Exact Match and F1/Accuracy per dataset; TTFT (time-to-first-token), TTIT (time-to-iterative-token), Throughput (tokens/second); Rouge-{1,2,L} for summarization.

## Findings

- At compression rate k=16, REFRAG achieves 9.3% average log-perplexity improvement over CEPE (previous SOTA) across 4 datasets, while being 16.53× faster in TTFT (cached) and 2.01× faster than CEPE.
- At compression rate k=32, TTFT acceleration reaches 30.85× over LLaMA and 3.75× over CEPE, with log-perplexity matching CEPE.
- REFRAG_8 and REFRAG_16 consistently outperform baselines (CEPE, REPLUG, LLaMA-32K) across standard and extended context lengths (up to 16k context vs. LLaMA's 4k limit).
- In RAG at equal latency (8 passages for REFRAG vs. 1 for LLaMA), REFRAG achieves 1.22% average improvement over 16 RAG tasks (strong retriever) and 1.93% (weak retriever).
- Under equal number of retrieved passages, REFRAG matches LLaMA in strong retriever and outperforms in weak retriever settings (because larger context helps extract more useful information when passages are less relevant).
- RL-based selective compression consistently outperforms heuristic and random selection strategies; REFRAG_16 with RL-selected compression rate of 8 outperforms REFRAG_8 with full compression.
- Multi-turn conversation: REFRAG outperforms LLaMA (4k context limit causes truncation of history) at 10-passage setting on all 3 datasets.
- Summarization: REFRAG_8 achieves best Rouge scores under same latency constraints across Arxiv and PubMed.
- Compression rate 64 is too aggressive, causing significant performance degradation. Rates 8–32 are practical.
- Decoder scale (7B→13B) has much larger impact than encoder scale (RoBERTa-base→large).
- RAG attention patterns are block-diagonal: within-passage attention is far stronger than cross-passage attention, confirming that cross-chunk attention computation is largely redundant.
- Curriculum learning and reconstruction pre-training are both essential (ablations show direct CPT without these stages fails to reduce perplexity even for reconstruction tasks).

## Conclusion / Discussion

RAG contexts have unique structure (block-diagonal sparse attention due to diverse/deduplicated passages) that generic long-context methods ignore. REFRAG exploits this by compressing retrieved passage chunks into pre-computable embeddings, dramatically reducing TTFT and KV cache memory. The "compress anywhere" property (unlike CEPE, which is prefix-only) enables multi-turn conversations and arbitrary context positions. RL-based selective compression adaptively chooses which chunks to keep uncompressed at inference time without recomputing embeddings. REFRAG also extends the effective context window of LLMs (from 4k to 16k+ for LLaMA-2-7B) without modifying its architecture.

## Limitations

- Requires continual pre-training and instruction fine-tuning; cannot be applied off-the-shelf.
- Compression rate limit: rates above 32 cause diminishing performance, with rate 64 showing significant degradation.
- Evaluated primarily on LLaMA-2-7B (baseline comparisons require re-training baselines on different models).

## Future Work

- Integration with complementary methods (compressed attention, StreamingLLM) for further latency reduction.
- Generalizing to larger and different decoder architectures at scale.
- Exploring higher compression rates through improved training strategies.
