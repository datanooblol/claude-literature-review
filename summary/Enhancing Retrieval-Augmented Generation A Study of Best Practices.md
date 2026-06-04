# Enhancing Retrieval-Augmented Generation: A Study of Best Practices

**Source:** `raw/Enhancing Retrieval-Augmented Generation A Study of Best Practices.pdf`
**Authors:** Siran Li, Linus Stenzel, Carsten Eickhoff, Seyed Ali Bahrainian
**Publisher / Venue:** arXiv (University of Tübingen)
**Year:** 2025
**Summarized:** 2026-06-05

## Research Problem / Objective

The best practices for designing RAG systems are not well understood. While many RAG architectures have been proposed, the impact of individual configuration choices — such as LLM size, prompt design, chunk size, knowledge base size, retrieval stride, query expansion, and in-context learning approaches — has not been systematically benchmarked in a controlled, ablation-style study. The paper addresses nine research questions (RQs) about these factors and proposes four novel RAG configurations.

## Methodology

**System Architecture:** Three-component RAG: (1) Query Expansion Module (Flan-T5 small generates N expanded keyword queries); (2) Retrieval Module (FAISS with all-MiniLM-L6-v2 Sentence Transformer, inner product similarity over chunked documents); (3) Text Generation Module (Mistral 7B or 45B instruct).

**Retrieval Process (3 steps):**
1. Retrieve documents D(1) using expanded queries q' and original query q.
2. Re-retrieve final documents D(2) from D(1) using original query q only.
3. (Focus Mode) Split D(2) into sentences S, retrieve most relevant sentences S(1).

**Nine Research Questions (ablation study with 74 total runs):**
1. **LLM Size:** Mistral 7B (Instruct7B) vs. 45B (Mixtral-8x7B Instruct45B).
2. **Prompt Design:** 3 helpful prompts (HelpV1/V2/V3) vs. 2 adversarial prompts (AdversV1/V2).
3. **Document Chunk Size:** 48, 64, 128, 192 tokens (2DocS/M/L/XL).
4. **Knowledge Base Size:** 1K (Level 3 Wikipedia Vital Articles) vs. 10K (Level 4); 2 vs. 5 retrieved docs.
5. **Retrieval Stride:** Context updates every 1, 2, 5 tokens during generation vs. baseline (no updates).
6. **Query Expansion:** Varying initial filter sizes (9/15/21 articles) before focused retrieval.
7. **Contrastive In-Context Learning (novel):** Use evaluation dataset examples (correct and incorrect answers) as knowledge base instead of Wikipedia; ICL1Doc/2Doc (correct examples only) vs. ICL1Doc+/2Doc+ (correct + incorrect/contrastive examples).
8. **Multilingual Knowledge Base:** Random replacement of English docs with French/German; with/without "Answer in English" prompt.
9. **Focus Mode (novel):** Retrieve relevant sentences from documents rather than full chunks (20Doc20S, 40Doc40S, 80Doc80S, 120Doc120S = X documents × X top sentences each).

**Knowledge Base:** Wikipedia Vital Articles (Level 3: ~999 articles; Level 4: ~10,011 articles); multilingual: French and German versions added.

**Base Models:** Mistral-7B-Instruct-v0.2 (baseline); Mixtral-8x7B-Instruct-v0.1 (45B comparison).

## Dataset

- **TruthfulQA** (817 questions, 38 categories): Tests common misconceptions; requires general commonsense.
- **MMLU** (1,824 samples, 57 subjects, 32 examples each): Multiple-choice; tests specialized/professional knowledge.

## Evaluation Metrics

- ROUGE-1, ROUGE-2, ROUGE-L F1 (lexical overlap)
- Embedding Cosine Similarity (ECS; all-MiniLM-L6-v2)
- MAUVE (distribution similarity between generated and reference text)
- FActScore (GPT-3.5-turbo as factuality judge; atomic fact precision)

## Findings

1. **LLM Size:** Larger model (45B) improves TruthfulQA noticeably; gains on MMLU are modest — model size is not the primary lever for specialized tasks.
2. **Prompt Design:** Helpful prompts significantly outperform adversarial ones; HelpV2 and HelpV3 achieve highest scores. Even small wording changes matter.
3. **Chunk Size:** Minimal performance differences across 48–192 tokens; larger chunks (192) slightly better on some metrics but not significantly.
4. **Knowledge Base Size:** No statistically significant benefit from 10K vs. 1K articles or from retrieving 5 vs. 2 documents — quality/relevance matters more than quantity.
5. **Retrieval Stride:** More frequent updates (smaller stride) hurt coherence and ROUGE/ECS/MAUVE — context disruption outweighs freshness; baseline (no updates) is best. Perplexity is unreliable as an evaluation metric here.
6. **Query Expansion:** Marginal improvements only; the most relevant documents are typically found without expansion in these tasks.
7. **Contrastive In-Context Learning (novel, best performer):** ICL1Doc+ (one correct + one incorrect example) achieves highest scores across all metrics and both datasets. On MMLU: ROUGE-L 23.87 vs. 8.91 baseline; ECS 47.12 vs. 29.41. FActScore: 57.00 (TruthfulQA), 74.44 (MMLU) vs. 53.85/63.73 baseline. Benefit is largest on specialized knowledge tasks (MMLU).
8. **Multilingual:** Both multilingual configurations hurt performance — the model struggles to synthesize cross-language information.
9. **Focus Mode (novel, second-best):** Retrieving relevant sentences rather than full chunks improves performance, especially at 80Doc80S (TruthfulQA) and 120Doc120S (MMLU). FActScore 2nd best (65.87 MMLU). Effective for narrowing noise.

## Conclusion / Discussion

Contrastive In-Context Learning RAG is the top performer — providing both correct and incorrect examples in context dramatically improves factuality and relevance, especially for specialized knowledge. Focus Mode (sentence-level retrieval) is the second-best novel contribution. Prompt formulation remains critical. Knowledge base size and chunk size have minimal impact compared to retrieval strategy and in-context learning design. Retrieval stride (frequent context updates) hurts more than helps. These findings provide actionable guidance for RAG system designers.

## Limitations

- Combinations of techniques not tested (e.g., Focus Mode + Contrastive ICL together).
- Model size comparison limited to one pair (7B vs. 45B); deeper analysis across sizes not conducted.
- Multilingual setting only tested English output with French/German context; limited language coverage.

## Future Work

- Dynamically adapting the retrieval module based on prompt/context.
- AutoML techniques to automate retrieval model selection and optimization for specific tasks.
- Combining multiple RAG improvements simultaneously.
