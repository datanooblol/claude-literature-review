# Literature Review (v1)

**Version:** 1
**Date:** 2026-06-06
**Status:** draft
**Supersedes:** `literature_review_v0.md`
**Changelog:** See `log/literature_revision.md`

---

## 2. Literature Review

This review examines three interrelated dimensions of retrieval-augmented generation that collectively motivate the present study: the structural limitations of in-context knowledge injection, the empirical reality of context saturation and its implications for efficiency, and the underexplored role of terminological conditioning form as an independent variable. The review builds toward three research gaps that no existing work addresses jointly: the lack of systematic comparison across terminological conditioning strategies, the absence of an explicit efficiency–performance trade-off framework, and the limited examination of model-scale sensitivity to these interventions.

---

### 2.1 RAG Foundations and the Limits of In-Context Injection

Retrieval-Augmented Generation (RAG) was introduced by Lewis et al. (2021) to mitigate the limitations of parametric-only knowledge in large language models (LLMs), where factual information is implicitly encoded in model weights and updating knowledge typically requires additional training. Instead of modifying parameters to incorporate new knowledge, RAG augments generation with non-parametric evidence retrieved from an external corpus. The framework couples a dual-encoder dense retriever — which maps queries and documents into a shared embedding space for similarity-based retrieval — with an encoder–decoder Transformer generator (BART). In the sequence formulation, top-k retrieved documents are concatenated with the input query to condition the decoding process. Because generation is explicitly conditioned on retrieved documents, retrieval quality serves as a critical determinant of downstream performance.

This in-context injection architecture, while effective, contains a structural asymmetry that constrains the depth of knowledge integration. Su et al. (2025) make this asymmetry explicit: large language models store and operationalize knowledge in their feed-forward network (FFN) parameters, whereas in-context retrieval augmentation provides knowledge only through attention-layer key-value computations — a shallower pathway that the model cannot treat as internalized knowledge. To address this, Parametric RAG (P-RAG) trains a LoRA adapter for each document offline, encoding document content into FFN weight perturbations, then merges the adapters of retrieved documents into the LLM's weights before generation with no retrieved text in the input context. On factoid and multi-hop QA benchmarks, P-RAG outperforms all in-context RAG baselines with 29–36% lower inference latency for LLaMA-3-8B, while combining P-RAG with standard in-context RAG achieves the highest overall performance. The practical constraint is storage cost (approximately 4.72 MB per document in LoRA matrices) and model-specificity, making P-RAG most suitable for fixed corpora.

Taken together, these foundational results establish that the mechanism of injection — not just the content — determines how effectively an LLM can use external knowledge. This implies that how terminological information is formatted and delivered to the generator is not merely a prompt engineering detail, but a variable with architectural significance.

---

### 2.2 Context Saturation: Empirical and Theoretical Evidence

A foundational assumption of RAG is that more retrieved evidence improves generation quality. A converging body of empirical and theoretical work demonstrates this assumption is incorrect beyond a task-specific threshold — establishing the concept of context saturation as a structural constraint rather than an incidental engineering failure.

Jin et al. (2024) provide early empirical evidence: in long-context RAG, performance initially improves as the retrieval set expands through higher recall, but deteriorates beyond a threshold due to accumulation of "hard negatives" — semantically similar but irrelevant passages that introduce noise into extended contexts. Leng et al. (2024) extend this observation across 20 commercial and open-source LLMs, systematically varying context length from 2,000 to 128,000 tokens. Their results show that only a small subset of state-of-the-art models maintain stable accuracy above 64k tokens, with performance plateauing and declining beyond model-specific limits — indicating that saturation thresholds are architecturally determined, not universal.

More fundamentally, Du et al. (2025) isolate context length as an independent variable from retrieval quality, using controlled synthetic experiments in which retrieval success is verified by exact match before testing. Even when all distractor tokens are masked from attention — eliminating noise and the "lost-in-the-middle" effect entirely — accuracy still degrades as context grows, falling between 13.9% and 85% relative to short-context baselines across five models and four tasks, with at least 7.9% degradation for open-source models at 30k masked tokens. The authors attribute this to positional distribution bias: LLMs trained on short-context data develop positional encodings that degrade when evidence and question are separated by positional distance, regardless of whether intervening tokens are informative. This reframes the problem: degradation is not only caused by retrieval noise, but by the sheer volume of context independently.

Roberts et al. (2025) extend this diagnosis to near-million-scale contexts with a synthetic benchmark of progressively difficult tasks across 17 LLMs at up to 630k tokens. Their effective context length metric reveals that most models' practical limits lie far below their advertised context windows. For multi-step reasoning at five steps, all models except GPT-4o and LLaMA 3.1 405B achieve zero effective context length.

At the theoretical level, Bansal et al. (2025) formalize this phenomenon as score dilution: when many distractor tokens are near-tied with the target in logit space, the softmax attention mass on the target vanishes as context length T grows, with the required logit gap scaling as Ω(log T). They further prove that autoregressively generated thinking tokens cannot repair this deficit — each thinking token contributes at most its own attention mass. Their proposed remedy, query-only test-time training (qTTT), updates only query projection matrices W_Q at inference time while keeping the KV cache frozen, provably increasing the target–distractor logit margin. On LongBench-v2 with Qwen3-4B, qTTT achieves 39.6% accuracy versus 33.5% for FLOP-matched thinking tokens and 27.0% for standard in-context.

Collectively, these results establish that context saturation is a structural constraint of current LLMs, independent of retrieval precision. They also establish a conceptual foundation for the Efficiency Apex: a point exists at which performance gains from additional context become disproportionate to the token cost, and beyond which the system enters a saturation regime. This trade-off between generation quality and context volume remains underexplored as an explicitly optimizable quantity.

---

### 2.3 Terminological Conditioning: Form Over Quantity

If context saturation establishes that *how much* is retrieved is bounded in usefulness, a parallel line of work establishes that *how* the retrieved content is formatted and framed is the primary lever for generation quality — a finding with direct implications for terminological conditioning strategies.

An et al. (2024) demonstrate this in an industrial RAG setting where queries contain domain-specific jargon, abbreviations, and implicit contextual references that standard dense retrievers cannot resolve. Their system, Golden-Retriever, introduces a reflection-based query augmentation pipeline executed before retrieval: an LLM identifies domain-specific jargon and abbreviations, a SQL lookup retrieves definitions from a maintained jargon dictionary, a CoT-based classifier assigns the query a context category, and an augmented query replaces the original. Evaluated on 58 domain-specific questions from Western Digital's industrial knowledge base, this pre-retrieval augmentation yields a 57.3% average improvement over vanilla LLM and 35.0% over standard RAG across three LLMs — with no model training. Crucially, the authors show that post-retrieval corrections (Corrective RAG, Self-RAG) cannot compensate when the initial retrieval is corrupted at the query stage. The form of the query itself — and specifically the form of terminological information embedded in it — determines what gets retrieved and therefore what the generator receives.

Li et al. (2025) provide the most direct evidence that format dominates quantity at the generation stage. In a 74-run ablation across nine RAG design dimensions including chunk size, knowledge base size, retrieval stride, and query expansion, the most striking result is the prevalence of null findings: chunk size, KB size, retrieval stride, and query expansion all have minimal impact on downstream performance. What does matter decisively is the in-context learning strategy. Their novel Contrastive ICL configuration — which provides both correct and incorrect examples in context rather than retrieved documents — achieves the largest improvements across all metrics: ROUGE-L rises from 8.91 to 23.87 and FActScore from 63.73 to 74.44 on MMLU. Their Focus Mode, which retrieves the most relevant sentences from documents rather than full chunks, ranks as the second-best intervention. Together, these results establish that the form and framing of contextual information, not its volume, is the primary lever for generation quality in RAG systems.

Joren et al. (2025) extend this argument by separating retrieval failure from generation failure. They introduce the concept of sufficient context — whether retrieved content contains enough information to answer the query, independent of LLM capability — and build a Gemini 1.5 Pro autorater that classifies context sufficiency at 93% accuracy without requiring ground-truth answers. Their analysis reveals a sobering finding: even when context is sufficient, frontier models (Gemini 1.5 Pro, GPT-4o, Claude 3.5 Sonnet) hallucinate at 14–17%. With insufficient context, models hallucinate far more than they abstain — suggesting they become overconfident when given any retrieved text, regardless of its actual utility. Their selective generation approach, combining the sufficiency signal with model self-confidence, improves accuracy by 2–10% at most coverage levels with no training cost.

These three papers converge on a finding that motivates the central design question of the present study: the form of what is provided to the generator — whether terminological evidence, explanation, or a hybrid — is an independent variable that is both impactful and systematically underexplored. Prior RAG studies treat retrieved content as homogeneous evidence; none have systematically compared the effect of structurally distinct terminological forms on generation quality and resource efficiency in specialized domain settings where terminology is the primary retrieval barrier.

---

### 2.4 Retrieval Design and Efficiency Feasibility

A complementary set of studies examines retrieval design choices — how many documents to retrieve, from what corpus, and using what indexing strategy — and finds that standard tuning levers are largely inert while pointing toward efficiency as an underutilized optimization target.

Vladika and Matthes (2025) conduct the most systematic study of retrieval design variables, varying the number of retrieved snippets (k = 0 to 30), comparing BM25 against semantic search, and contrasting retrieval from a closed 8,000-document corpus against an open 10-million-document corpus across eight LLMs on biomedical and encyclopedic long-form QA benchmarks. Their findings are largely cautionary: performance improves up to approximately 15 snippets before saturating or declining, establishing a practical retrieval ceiling. More significantly, for some model-dataset combinations, poorly retrieved snippets actively hurt performance relative to relying on the model's parametric knowledge alone — GPT-4o and Mixtral score higher at k=0 than with 1–10 imperfect snippets. This finding complicates the standard assumption that any retrieval is better than none, and implies that the threshold at which additional context becomes harmful is model-dependent.

Merola and Singh (2025) examine retrieval design at the indexing level, comparing late chunking (full-document embedding before segmentation) and contextual retrieval (LLM-generated context prepended to each chunk before embedding) under identical experimental conditions. Neither strategy dominates consistently: contextual retrieval with rank fusion outperforms late chunking on retrieval metrics, but requires LLM inference per chunk and up to 20GB of GPU memory for long documents. The practical implication is that indexing strategy is as much a resource-budget decision as a quality decision — a finding directly relevant to system design under computational constraints.

Li et al. (2025) similarly confirm that standard retrieval tuning levers (chunk size, query expansion, KB size, retrieval stride) have minimal impact compared to generation-stage choices. This null result across retrieval design dimensions suggests that practitioner effort invested in retrieval configuration has limited headroom, and that efficiency gains must be sought at the generation stage.

Two post-retrieval compression systems demonstrate that substantial efficiency gains are achievable without proportional quality loss. Oreo (Li & Ramakrishnan, 2025), a T5-small module trained via supervised fine-tuning, contrastive multi-task learning, and PPO-based RL alignment to the downstream generator, reconstructs retrieved chunks into concise, query-specific contexts before generation. Across five QA datasets, Oreo achieves up to 37.5% EM improvement on NaturalQuestions while compressing context by 84–94% and reducing generation latency by 23–43%. REFRAG (Lin et al., 2025) approaches the same problem architecturally, replacing retrieved passage tokens with pre-computable encoder embeddings before decoding and reducing input length by a factor of k. At compression rate k=32, REFRAG achieves 30.85× TTFT acceleration over standard LLaMA and 3.75× over the previous SOTA with matching perplexity.

These compression results establish that efficiency improvements of an order of magnitude are achievable without proportional performance loss — which implies that the relationship between token consumption and generation quality is not linear. However, neither Oreo nor REFRAG frames this as a systematic trade-off curve: neither characterizes the point at which efficiency gains cease to be proportional to performance cost, nor examines how this trade-off varies across model scales.

---

### 2.5 Model-Scale Sensitivity to Contextual Strategy

A recurring but underexplored observation across the reviewed literature is that the optimal retrieval and conditioning strategy is model-dependent — varying not only with model size but with model identity, architecture, and task domain in ways that size alone does not predict.

Vladika and Matthes (2025) provide the clearest evidence of this at the model-domain level. On biomedical QA, Mixtral-8x7B and Mistral-7B substantially outperform GPT-4o and LLaMA 3 (70B) despite being smaller models, while on encyclopedic QA the ranking reverses. The authors conclude that domain-specific knowledge of the LLM matters more than model size for specialized tasks. Furthermore, the k-value at which performance saturates and the point at which poorly retrieved snippets become harmful both vary across models — demonstrating that optimal retrieval configuration is not transferable across architectures.

Roberts et al. (2025) extend this observation to context utilization capacity, showing that effective context length — the practical limit at which models can reliably retrieve embedded information — varies substantially by model. For multi-step threading at five steps, only GPT-4o and LLaMA 3.1 405B maintain non-zero effective context length, while all other tested models fail entirely. This scale-dependent capacity means that larger models can use context that smaller models cannot — but the relationship between nominal scale (parameter count) and effective context length is non-linear and model-specific.

Bansal et al. (2025) provide a theoretical account of why this scale sensitivity exists. Score dilution — the vanishing of attention mass on target tokens as context length grows — is a function of the logit gap between target and distractor tokens in the attention mechanism. This gap depends on the model's internal representations, which vary across architectures and training regimes independently of scale. Their qTTT experiments show that improvements from query projection adaptation scale with model size across Qwen3 variants (1.7B to 32B), but whether this scaling relationship holds across architecturally distinct families is not established.

Li et al. (2025) add a complementary finding: larger models (45B vs. 7B) improve performance on general-domain tasks (TruthfulQA) more noticeably than on specialized tasks (MMLU), suggesting that the marginal benefit of scale interacts with task domain. If the benefit of scale is task-dependent, it follows that the benefit of a specific terminological conditioning strategy — which is itself task-aligned — may also depend on model scale in a non-obvious way.

Together, these results establish that model scale is not a proxy for context utilization capacity, and that optimal conditioning strategy cannot be derived from scale alone. No existing study directly examines how the optimal form of terminological conditioning (evidence vs. explanation vs. hybrid) varies as a function of model scale under a constrained resource budget — the question at the core of the present study.

---

### 2.6 Summary of Research Gaps

The preceding review converges on three observations that collectively motivate the ARAI framework.

**Gap 1 — Lack of methodological differentiation in terminological conditioning.**
Prior RAG studies treat retrieved content as homogeneous evidence, varying retrieval size, context length, or model architecture but not the structural form of the terminological information provided. Li et al. (2025) demonstrate that format is the most impactful variable in RAG configuration — Contrastive ICL, which alters the form rather than the quantity of contextual information, produces the largest performance gains across all tested conditions. Golden-Retriever (An et al., 2024) similarly shows that the form of terminological augmentation at the query stage determines retrieval quality in specialized domains. Yet no study systematically compares evidence-based, explanation-based, and hybrid terminological forms as the primary independent variable in a controlled, domain-specific setting where terminology is the principal retrieval barrier. It remains unclear which conditioning strategy best balances generation coherence and resource efficiency across models of varying scale.

**Gap 2 — Insufficient examination of efficiency–performance trade-offs.**
Du et al. (2025), Roberts et al. (2025), and Vladika and Matthes (2025) establish that context saturation is real and model-dependent: performance degrades or plateaus beyond a threshold regardless of retrieval quality. Oreo (Li & Ramakrishnan, 2025) and REFRAG (Lin et al., 2025) demonstrate that aggressive context compression — 84–94% reduction — is feasible without proportional performance loss. These findings together imply that an Efficiency Apex exists: a configuration at which performance per token is maximized before the onset of Task Saturation. However, no study formalizes this trade-off as an explicitly optimizable quantity. Performance continues to be measured primarily using accuracy-based metrics (EM, F1, ROUGE-L), without a companion metric that jointly captures token consumption and output quality. It remains unresolved whether the Efficiency Apex shifts as a function of terminological conditioning strategy or model scale.

**Gap 3 — Limited analysis of model-scale sensitivity to terminological intervention.**
Vladika and Matthes (2025) show that optimal model choice and retrieval configuration are domain-dependent and do not scale predictably with parameter count. Li et al. (2025) find that larger models benefit more from scale on general tasks but less on specialized ones. Roberts et al. (2025) and Bansal et al. (2025) establish that effective context utilization capacity is model-specific and architecturally determined. Yet no study examines how the optimal form of terminological conditioning — evidence vs. explanation vs. hybrid — varies as a function of model scale under a constrained resource budget. Whether smaller models benefit differently from targeted terminological evidence than larger models from richer explanatory context represents an architectural alignment question the existing literature does not answer.

Addressing these three gaps requires a framework that explicitly models the relationship between terminological conditioning strategy, model scale, and resource allocation — treating these three dimensions as jointly optimizable rather than independently tunable. This study proposes the Assistant Researcher AI (ARAI), a tunable agentic architecture designed to evaluate parameter–method optimization under efficiency constraints.

---

## References

An, Z., Ding, X., Fu, Y. C., Chu, C. C., Li, Y., & Du, W. (2024). Golden-Retriever: High-fidelity agentic retrieval augmented generation for industrial knowledge base. *arXiv*. https://doi.org/10.48550/arXiv.2408.00798

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
