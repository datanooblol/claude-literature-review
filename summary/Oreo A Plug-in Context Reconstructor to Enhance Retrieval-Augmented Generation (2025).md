# Oreo: A Plug-in Context Reconstructor to Enhance Retrieval-Augmented Generation

**Source:** `raw/Oreo A Plug-in Context Reconstructor to Enhance Retrieval-Augmented Generation (2025).pdf`
**Authors:** Sha Li, Naren Ramakrishnan
**Publisher / Venue:** arXiv
**Year:** 2025
**Summarized:** 2026-06-05

## Research Problem / Objective

Standard RAG systems suffer from several post-retrieval failure modes: (1) semantic dissonance between query, retriever, and generator; (2) noise, redundancy, and distracting content in retrieved chunks misleading the generator; (3) difficulty correlating dependencies across multiple documents for multi-hop reasoning; (4) the "lost-in-the-middle" problem where LLMs neglect middle-positioned evidence; (5) knowledge inconsistencies between the retriever and generator due to independent optimization. The paper proposes a compact, plug-in module to reconstruct retrieved chunks into a concise, query-specific context before generation.

## Methodology

**Oreo (cOntext REcOnstructor)** is a text generation model M_θ inserted between retriever and generator, transforming the "retrieve-then-generate" paradigm into "retrieve-reconstruct-then-generate". Given retrieved chunks D and query q, it produces a refined context c = f_{M_θ}(D, q). T5-small is used as the backbone.

**Three-stage training:**

1. **Supervised Fine-tuning (SFT):** A synthetic gold context dataset is created using Llama3-8B-Instruct to extract key entities, events, and rationale chains from retrieved chunks. Bootstrapping is applied when the model fails to include the ground truth. Data is curated by (a) ground truth alignment and (b) entity/event consistency checks. M_θ is trained with standard next-token prediction (negative log-likelihood loss) on this gold context.

2. **Contrastive Multi-task Learning (CML):** Beam search generates top-n candidate contexts; these are ranked by ROUGE similarity to the gold context and labeled as positive/negative pairs. A pairwise margin-based contrastive loss (combined with SFT loss) encourages Oreo to bring positive candidates closer to the retrieved chunks in embedding space while distancing negative ones. This improves error recognition and generalization.

3. **Reinforcement Learning (RL) Alignment:** PPO is used to align Oreo's output with the downstream generator G's preferences. G serves as the reward model: the reward at each token step is based on ROUGE between G's generated answer and ground truth, weighted by token generation probability and regularized by a KL penalty to prevent the policy from drifting. This stage addresses knowledge inconsistencies across R, Oreo, and G that cannot be fixed by gradient backpropagation from a black-box G.

## Dataset

- **Single-hop QA:** PopQA, NaturalQuestions (NQ), TriviaQA
- **Multi-hop QA:** HotpotQA, 2WikiMultihopQA
- External knowledge source: Wikipedia dump
- Retrievers: Contriver, DPR, BM25 (varied per dataset)
- Downstream generators: FLAN-T5 and OPT-IML (both treated as black boxes)

## Evaluation Metrics

- Exact Match (EM) for single-hop extractive QA (PopQA, NQ, TriviaQA)
- Unigram F1 for multi-hop abstractive QA (HotpotQA, 2WQA)
- Context length reduction and inference latency for efficiency
- Faithfulness and completeness scores (0–10, LLM-as-judge using Qwen2.5-Instruct)

## Findings

- Oreo outperforms all extraction/compression baselines (Selective-Context, LLMLingua, LLMLingua-2, xRAG, CompAct, EXIT, Refiner, CXMI) on average across five datasets and two generators.
- Single-hop EM improvements over full content: +8.8% (PopQA), +23.1% (NQ), +37.5% (TriviaQA); multi-hop F1: +12.7% (HotpotQA), +19.8% (2WQA).
- Oreo compresses context by 84–94% vs. full document content, reducing inference latency by 22.98–43.01%.
- CompAct marginally outperforms Oreo on HotpotQA (+2.26% F1) but is 3× slower.
- Oreo is robust to chunk order shuffling (performance drop ≤ ±0.027), functioning as an implicit reranker and mitigating the "lost-in-the-middle" effect.
- Zero-shot cross-dataset transfer achieves near-in-distribution performance (e.g., PopQA→NQ: 0.4352 vs. 0.4413 in-domain).
- Performance plateau or slight decline occurs beyond 240–270 context tokens, indicating diminishing returns with excessively long reconstructed contexts.
- Oreo achieves highest faithfulness and completeness scores across all datasets per Qwen2.5-Instruct evaluation.

## Conclusion / Discussion

Oreo introduces a "retrieve-reconstruct-then-generate" paradigm that sits between retriever and generator as a lightweight plug-and-play module. Its three-stage training (SFT → contrastive multitask learning → RL alignment) progressively improves context quality, error correction, and generator alignment. The approach is model-agnostic and compatible with arbitrary retrievers, generators, and RAG frameworks without requiring their modification.

## Limitations

- Aggressive compression may omit essential information in complex multi-hop or long-form QA settings.
- Not systematically tested in adversarial retrieval scenarios involving conflicting or deceptive content.
- Evaluations rely on indirect metrics (downstream QA accuracy, LLM judgment), which may introduce bias.

## Future Work

- Adaptive compression strategies that dynamically allocate token budgets based on query complexity.
- Robustness to adversarial or noisy retrieval scenarios.
- More principled evaluation frameworks for compression, informativeness, and faithfulness trade-offs.
- Progress-based RL frameworks with denser intermediate rewards for more stable policy learning.
