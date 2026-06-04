# Golden-Retriever: High-Fidelity Agentic Retrieval Augmented Generation for Industrial Knowledge Base

**Source:** `raw/Golden-Retriever High-Fidelity Agentic Retrieval Augmented Generation.pdf`
**Authors:** Zhiyu An, Xianzhong Ding, Yen-Chun Fu, Cheng-Chung Chu, Yan Li, Wan Du
**Publisher / Venue:** arXiv preprint (arXiv:2408.00798v1 [cs.IR])
**Year:** 2024
**Summarized:** 2026-06-05

## Research Problem / Objective

Traditional RAG frameworks struggle to handle domain-specific queries in industrial knowledge bases for two reasons: (1) LLMs may hallucinate or misinterpret domain-specific jargon and abbreviations present in user questions, leading to poor document retrieval; (2) user questions rarely contain explicit context information necessary for accurate retrieval. Existing post-retrieval methods (Corrective RAG, Self-RAG) cannot compensate when the initial retrieval itself is flawed. The paper proposes Golden-Retriever to resolve ambiguities in user questions *before* retrieval via reflection-based question augmentation.

## Methodology

Golden-Retriever has two phases:

**Offline — LLM-Driven Document Augmentation:**
- Collect company documents (slides, images, tables); apply OCR to extract text
- Split text into chunks (~4,000 tokens for Meta-Llama-3)
- For each chunk, prompt an LLM to generate a domain-expert-perspective summary
- Add augmented summaries to the document database alongside original content

**Online — Reflection-Based Question Augmentation (per query):**
1. **Identify Jargons (§3.2):** Prompt an LLM to extract all jargons and abbreviations from the user's question. Uses LLM (not string matching) for robustness against typos and unknown terms. If the list is empty, skip to retrieval directly.
2. **Identify Context (§3.3):** Prompt the LLM with a predefined list of context categories and few-shot Chain-of-Thought (CoT) examples to classify the question's context (e.g., "Design Question", "Process Question").
3. **Query Jargons (§3.4):** Execute a code-synthesized SQL query (not LLM-generated SQL) against a jargon dictionary to retrieve extended definitions and descriptions for identified terms.
4. **Augment Question (§3.5):** Combine the original question, identified context, and jargon definitions into a structured augmented question that replaces the original for retrieval.
5. **Query Miss Response (§3.6):** If a jargon is not found in the dictionary, return a fallback response instructing the user to check spelling or contact the knowledge base manager; avoid hallucination.

The augmented question is then used as input to the standard RAG retrieval-and-generation pipeline.

## Dataset

- **Question-Answering Dataset:** Multiple-choice questions collected from new-hire training documents at Western Digital Corporation, spanning 6 domains (9–10 questions each, 58 total). Questions are 1–2 sentences long, contain jargon/abbreviations, and have 2–4 answer choices. Evaluated over 5 repeated trials; average scores reported.
- **Abbreviation Identification Dataset:** Synthetic dataset created by inserting randomly generated abbreviations (30 total, drawn from letter-frequency distributions) into manually prepared question templates. Questions contain 1–5 abbreviations each.

## Evaluation Metrics

- **Question-Answering:** Total correct answers out of 58, averaged across 5 trials per method-LLM combination
- **Abbreviation Identification:** Accuracy (%) of correctly identifying *all* abbreviations in a question, broken down by number of abbreviations per question (1–5)

## Findings

**Question-Answering (Total Score / 58):**

| Method | Llama3 | Mistral | Shisa |
|--------|--------|---------|-------|
| Vanilla LLM | 21.2 | 23.0 | 21.0 |
| Vanilla RAG | 27.0 | 27.0 | 22.0 |
| Golden-Retriever | **38.0** | **33.8** | **30.6** |

- Golden-Retriever improves Llama3 by 79.2% over Vanilla LLM and 40.7% over Vanilla RAG
- Averaged across all three LLMs: +57.3% over Vanilla LLM, +35.0% over Vanilla RAG

**Abbreviation Identification Accuracy:**

| Model | 1 abbr | 2 abbr | 3 abbr | 4 abbr | 5 abbr |
|-------|--------|--------|--------|--------|--------|
| Llama3 | 70% | 100% | 90% | 90% | 100% |
| Mistral | 100% | 100% | 100% | 70% | 80% |
| Shisa | 50% | 80% | 60% | 80% | 100% |

Llama3 and Mistral achieve generally high accuracy; Shisa shows lower and more variable performance. Different failure modes observed: Llama3 sometimes adds known expansions, Mistral includes surrounding context as part of the abbreviation, Shisa produces mixed-language outputs.

## Conclusion / Discussion

Golden-Retriever demonstrates that resolving jargon and context ambiguity *before* retrieval substantially outperforms post-retrieval correction approaches. The system avoids fine-tuning drawbacks (Reversal Curse, catastrophic forgetting, retraining costs) and does not require dedicated training datasets for context classification by using an "LLM as backend" approach. The authors argue RAG is superior to fine-tuning for industrial knowledge bases due to dynamic knowledge updates without retraining.

## Limitations

Not explicitly stated as a limitations section. Implicit limitations from the paper:
- Requires a pre-existing, maintained jargon dictionary; system cannot handle jargon absent from the dictionary
- Requires a predefined and curated list of context categories
- Introduces additional latency from multiple sequential LLM calls per query
- Evaluation is narrow: 58 questions from one company (Western Digital), 6 domains, 3 LLMs
- No evaluation of the offline document augmentation step in isolation

## Future Work

Not mentioned in paper.
