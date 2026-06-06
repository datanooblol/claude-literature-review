# Literature Review (Revised)

**Revised:** 2026-06-06
**Status:** draft
**Note:** Expands the original literature review with 11 additional papers (2025). Original citations retained. New papers added to strengthen all three research gaps.

---

## 2. Literature Review

### 2.1 Retrieval-Augmented Generation: Foundations

Retrieval-Augmented Generation (RAG) was introduced by Lewis et al. (2021) to mitigate the limitations of parametric-only knowledge in large language models (LLMs), where factual information is implicitly encoded in model weights and updating knowledge typically requires additional training. Instead of modifying parameters to incorporate new knowledge, RAG augments generation with non-parametric evidence retrieved from an external corpus. The framework couples a dual-encoder dense retriever, which maps queries and documents into a shared embedding space for similarity-based retrieval, with an encoder–decoder Transformer generator (BART). In the sequence formulation, the top-k retrieved documents are concatenated with the input query to condition the decoding process. By combining parametric representations with dynamically retrieved external evidence, RAG improves factual grounding and performance on knowledge-intensive tasks compared to standalone LLMs. Because generation is explicitly conditioned on retrieved documents, retrieval quality serves as a critical determinant of downstream performance.

---

### 2.2 Context Length, Retrieval Quantity, and Diminishing Returns

A core assumption underlying RAG is that more retrieved evidence improves generation quality. However, a growing body of empirical work challenges this assumption, demonstrating that performance gains from additional context are bounded, model-dependent, and subject to diminishing returns.

Jin et al. (2024) reveal that increasing the number of retrieved passages in long-context RAG does not produce monotonic performance gains. While expanding the retrieval set initially improves answer quality through higher recall, performance subsequently deteriorates beyond a threshold. The authors attribute this degradation primarily to the accumulation of "hard negatives" — semantically similar but irrelevant passages that introduce noise into extended contexts. To mitigate this effect, they propose training-free retrieval reordering strategies that exploit positional biases and training-based approaches including supervised intermediate-reasoning fine-tuning. Complementing this, Leng et al. (2024) conduct a large-scale evaluation across 20 LLMs, systematically varying context length from 2,000 to 128,000 tokens. Their results show that only a small subset of state-of-the-art models maintain stable accuracy above 64k tokens, with performance plateauing and declining beyond model-specific limits.

More fundamentally, Du et al. (2025) isolate the effect of context length from retrieval quality itself, using controlled synthetic experiments in which retrieval success is verified by exact match before testing. Even when all distractor tokens are masked from attention — eliminating noise and the "lost-in-the-middle" effect entirely — accuracy still degrades as context grows, falling between 13.9% and 85% relative to short-context baselines across five models and four tasks. The authors attribute this to a positional distribution bias: LLMs trained on short-context data develop positional encodings that degrade when evidence and question are separated by positional distance, regardless of whether intervening tokens are informative. This finding reframes the RAG performance problem: not only does retrieval noise degrade performance, but the sheer volume of context does so independently.

At the benchmark level, Roberts et al. (2025) evaluate 17 frontier LLMs on synthetic UUID key-value retrieval tasks scaled to 630k tokens, introducing a progressive task hierarchy from single needle retrieval to multi-step chaining. Their effective context length metric — expressed in characters to enable fair cross-model comparison — reveals that most models' practical limits lie far below their advertised context windows. For multi-step reasoning tasks at five steps, all models except GPT-4o and LLaMA 3.1 405B achieve zero effective context length. Collectively, these studies establish that context saturation is a structural constraint of current LLMs, independent of retrieval precision.

Vladika and Matthes (2025) extend this analysis to the practical RAG setting, systematically varying retrieved snippet count (k = 0 to 30), corpus size, and retrieval method across eight LLMs on biomedical and encyclopedic long-form QA benchmarks. Their key findings are directly relevant to resource-efficient RAG design: performance improves up to approximately 15 snippets before saturating or declining; domain fit of the LLM matters more than model size for specialized tasks; and for some model-dataset combinations, poorly retrieved snippets actively hurt performance relative to relying on parametric knowledge alone. This last finding implies that the threshold beyond which additional context becomes harmful varies by model architecture — a model-scale dependency that has received limited systematic attention.

---

### 2.3 Semantic Mismatch and Pre-Retrieval Query Conditioning

A distinct but related failure mode arises before retrieval begins: when user queries contain domain-specific jargon, abbreviations, or implicit contextual references that standard dense retrievers cannot resolve, the retrieved documents are structurally irrelevant regardless of how they are subsequently processed.

An et al. (2024) demonstrate this in an industrial RAG setting, showing that post-retrieval corrections (as in Corrective RAG or Self-RAG) cannot compensate when the initial retrieval was corrupted at the query stage. Their proposed system, Golden-Retriever, introduces a reflection-based query augmentation pipeline executed before retrieval: an LLM identifies domain-specific jargon and abbreviations, a SQL lookup retrieves definitions from a maintained jargon dictionary, a CoT-based classifier assigns the query a context category, and an augmented query replaces the original. Evaluated on 58 domain-specific questions from Western Digital's industrial knowledge base, this pre-retrieval augmentation yields a 57.3% average improvement over vanilla LLM and 35.0% over standard RAG across three LLMs, with no model training. The result establishes query-side terminological intervention as a high-leverage, low-cost action distinct from both retrieval-side and generation-side improvements.

These findings are particularly relevant for RAG systems deployed in specialized domains, where the nature of terminological conditioning — what information about a term is provided, in what form — determines whether the retriever can locate relevant content at all. The question of whether evidence-style term definitions, explanation-style descriptions, or hybrid approaches best support downstream generation in such settings remains open.

---

### 2.4 Post-Retrieval Context Quality and Sufficiency

Even when retrieval succeeds in returning topically relevant documents, the gap between retrieved content and what the generator can effectively use remains substantial. Recent work has begun to characterize this gap formally and address it through post-retrieval processing.

Joren et al. (2025) introduce the concept of sufficient context — whether retrieved content contains enough information to answer the query, independent of LLM capability — and build a Gemini 1.5 Pro autorater that classifies context sufficiency at 93% accuracy without requiring ground-truth answers. Their analysis across FreshQA, Musique, and HotpotQA reveals a troubling baseline: even frontier models (Gemini 1.5 Pro, GPT-4o, Claude 3.5 Sonnet) hallucinate at 14–17% when context is sufficient, while with insufficient context, models hallucinate far more than they abstain. Providing any context — even insufficient — significantly reduces appropriate abstention, suggesting that models become overconfident when given retrieved text. Their selective generation approach, which combines the sufficiency signal with model self-confidence, improves accuracy by 2–10% at most coverage levels with no training cost. Critically, this work separates retrieval failure from generation failure, demonstrating that fixing retrieval alone cannot eliminate hallucination — the quality and form of the retrieved content as presented to the generator is equally important.

Complementing this diagnostic lens, Li and Ramakrishnan (2025) address post-retrieval noise through a learned reconstruction module. Oreo, a plug-in T5-small model trained via supervised fine-tuning, contrastive multi-task learning, and PPO-based RL alignment to the downstream generator, reconstructs retrieved chunks into a concise, query-specific context before generation. The RL stage addresses a fundamental limitation of prior compression approaches: when the generator is a black box, the retriever, reconstructor, and generator cannot be jointly optimized, and distributional mismatches persist. Across five QA datasets, Oreo achieves up to 37.5% EM improvement on NaturalQuestions and 84–94% context compression, with 23–43% latency reduction. Its robustness to chunk order shuffling suggests it functions as an implicit reranker, mitigating the "lost-in-the-middle" problem as a secondary benefit.

Lin et al. (2025) approach context quality from the efficiency angle, observing that retrieved passages produce block-diagonal attention patterns — near-zero cross-passage attention — due to semantic diversity from deduplication. Their system, REFRAG, replaces retrieved passage tokens with pre-computable encoder embeddings before decoding, reducing input length by a factor of k. At compression rate k=32, REFRAG achieves 30.85× TTFT acceleration over standard LLaMA and 3.75× over the previous SOTA, with matching perplexity. The pre-computability of chunk embeddings is a key practical property: the encoding cost is paid once per document at index time, not per query at inference time.

---

### 2.5 Parametric Knowledge Integration

A more architecturally radical response to in-context retrieval limitations is to bypass the input context altogether and inject retrieved knowledge directly into model parameters.

Su et al. (2025) argue that in-context injection creates a structural asymmetry: while LLMs store and operationalize knowledge in their feed-forward network parameters, retrieval augmentation provides knowledge only through attention-layer KV computations — a shallower integration pathway. Parametric RAG addresses this by training a LoRA adapter for each document offline, encoding document content into FFN weight perturbations, then merging the adapters of retrieved documents into the LLM's weights before generation, leaving the input context empty. On factoid and multi-hop QA benchmarks, P-RAG outperforms all in-context RAG baselines with 29–36% lower inference latency for LLaMA-3-8B, while combining P-RAG with standard in-context RAG achieves the highest overall performance. The primary practical constraint is storage cost (approximately 4.72 MB per document in LoRA matrices) and model specificity, making the approach most suitable for fixed, slowly-changing corpora at scale.

---

### 2.6 Retrieval Design: Chunking, Indexing, and Best Practices

A parallel line of work examines the design choices made at indexing and retrieval time, asking whether standard RAG design conventions are well-calibrated.

Merola and Singh (2025) compare two advanced chunking strategies — late chunking (full-document embedding before segmentation) and contextual retrieval (LLM-generated context prepended to each chunk) — under identical experimental conditions on NFCorpus and MsMarco. Neither dominates consistently: contextual retrieval with rank fusion outperforms late chunking on retrieval metrics, but requires LLM inference per chunk and up to 20GB of GPU memory for long documents, while late chunking is efficient but embedding-model-dependent. The practical implication is that indexing strategy is as much a resource-budget decision as a quality decision — a finding directly relevant to system design under computational constraints.

Li et al. (2025) conduct a 74-run ablation across nine RAG design dimensions including chunk size, knowledge base size, retrieval stride, and query expansion, using Mistral 7B and 45B on TruthfulQA and MMLU. Their most striking findings are the null results: chunk size, KB size, retrieval stride, and query expansion all have minimal impact on downstream performance. What does matter is the in-context learning strategy at generation time. Their novel Contrastive ICL configuration — which provides both correct and incorrect examples in context — achieves the largest improvements across all metrics, with ROUGE-L improving from 8.91 to 23.87 and FActScore from 63.73 to 74.44 on MMLU. This finding has a direct implication for terminological conditioning: the format and framing of contextual information, not its quantity, is the primary lever for generation quality improvement. Furthermore, sentence-level retrieval (Focus Mode), which retrieves the most relevant sentences from documents rather than full chunks, ranks as the second-best intervention — suggesting that precision of the terminological unit matters more than volume.

---

### 2.7 Summary of Research Gaps

The literature reviewed above converges on three observations that collectively motivate the present study.

**Gap 1 — Lack of methodological differentiation in terminological conditioning.** Prior RAG studies vary retrieval size, context length, or model architecture, but treat retrieved content as homogeneous evidence. The work of Li et al. (2025) demonstrates that the format of contextual information — whether evidence-based, explanation-based, or contrastive — is the most impactful lever for generation quality. However, this comparison has not been systematically extended to specialized domain settings where terminological precision is the primary failure mode. It remains unclear which conditioning strategy best balances generation coherence and resource efficiency when model-specific jargon is the limiting factor.

**Gap 2 — Insufficient examination of efficiency–performance trade-offs.** While Du et al. (2025), Roberts et al. (2025), and Vladika and Matthes (2025) establish that context beyond a threshold degrades or fails to improve performance, performance is primarily measured using accuracy-based metrics. The interaction between generation quality and token consumption — specifically, the point at which efficiency gains from context reduction remain proportional to performance cost — remains underexplored. Oreo (Li & Ramakrishnan, 2025) and REFRAG (Lin et al., 2025) demonstrate that aggressive context compression is feasible, but neither frames this as a systematic trade-off between token budget and performance stability across model scales.

**Gap 3 — Limited analysis of model-scale sensitivity to terminological intervention.** Vladika and Matthes (2025) show that domain-specific knowledge of the LLM matters more than model size, and that biomedical and encyclopedic tasks favor different architectures. Li et al. (2025) note that larger models improve TruthfulQA gains but show modest MMLU improvement. Yet no study systematically examines how the optimal form of terminological conditioning — evidence vs. explanation vs. hybrid — varies as a function of model scale under constrained resource budgets. Whether smaller models benefit differently from targeted terminological evidence than larger models from richer explanatory context represents an architectural alignment question that the existing literature does not answer.

Addressing these gaps requires a framework that explicitly models the relationship between terminological conditioning strategy, model scale, and resource allocation — treating these three dimensions as jointly optimizable rather than independently tunable. This study proposes the Assistant Researcher AI (ARAI), a tunable agentic architecture designed to evaluate parameter–method optimization under efficiency constraints.

---

## References

An, Z., Fu, Y. C., Chu, C. C., Li, Y., & Du, W. (2024). Golden-Retriever: High-fidelity agentic retrieval augmented generation for industrial knowledge base. *arXiv*. https://doi.org/10.48550/arXiv.2408.00798

Bansal, R., Zhang, A., Tiwari, R., Madaan, L., Duvvuri, S. S., Khatri, D., Brandfonbrener, D., Alvarez-Melis, D., Bhargava, P., Kale, M. S., & Jelassi, S. (2025). Let's (not) just put things in context: Test-time training for long-context LLMs. *arXiv*.

Du, Y., Tian, M., Ronanki, S., Rongali, S., Bodapati, S., Galstyan, A., Wells, A., Schwartz, R., Huerta, E. A., & Peng, H. (2025). Context length alone hurts LLM performance despite perfect retrieval. *arXiv*.

Jin, B., Yoon, J., Han, J., & Arık, S. Ö. (2024). Long-context LLMs meet RAG: Overcoming challenges for long inputs in RAG. *arXiv*. https://doi.org/10.48550/arXiv.2410

Joren, H., Zhang, J., Ferng, C.-S., Juan, D.-C., Taly, A., & Rashtchian, C. (2025). Sufficient context: A new lens on retrieval augmented generation systems. *ICLR 2025*.

Leng, Q., Portes, J., Havens, S., & Zaharia, M. (2024). Long context RAG performance of large language models. *arXiv*. https://doi.org/10.48550/arXiv.2411.03538

Lewis, P., Perez, E., Piktus, A., Petroni, F., Karpukhin, V., Goyal, N., Küttler, H., Lewis, M., Yin, W.-t., Rocktäschel, T., Riedel, S., & Kiela, D. (2021). Retrieval-augmented generation for knowledge-intensive NLP tasks. *arXiv*. https://doi.org/10.48550/arXiv.2005.11401

Li, S., & Ramakrishnan, N. (2025). Oreo: A plug-in context reconstructor to enhance retrieval-augmented generation. *arXiv*.

Li, S., Stenzel, L., Eickhoff, C., & Bahrainian, S. A. (2025). Enhancing retrieval-augmented generation: A study of best practices. *arXiv*.

Lin, X., Ghosh, A., Low, B. K. H., Shrivastava, A., & Mohan, V. (2025). REFRAG: Rethinking RAG based decoding. *arXiv*.

Merola, C., & Singh, J. (2025). Reconstructing context: Evaluating advanced chunking strategies for retrieval-augmented generation. *arXiv / ECIR 2025 Workshop*.

Roberts, J., Han, K., & Albanie, S. (2025). Needle threading: Can LLMs follow threads through near-million-scale haystacks? *ICLR 2025*.

Su, W., Tang, Y., Ai, Q., Yan, J., Wang, C., Wang, H., Ye, Z., Zhou, Y., & Liu, Y. (2025). Parametric retrieval augmented generation. *arXiv*. https://doi.org/10.48550/arXiv.2501.15915

Vladika, J., & Matthes, F. (2025). On the influence of context size and model choice in retrieval-augmented generation systems. *arXiv*.
