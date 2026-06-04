# Sufficient Context: A New Lens on Retrieval Augmented Generation Systems

**Source:** `raw/Sufficient Context A New Lens on Retrieval Augmented Generation Systems.pdf`
**Authors:** Hailey Joren, Jianyi Zhang, Chun-Sung Ferng, Da-Cheng Juan, Ankur Taly, Cyrus Rashtchian
**Publisher / Venue:** ICLR 2025 (International Conference on Learning Representations)
**Year:** 2025
**Summarized:** 2026-06-05

## Research Problem / Objective

Errors in RAG systems are commonly attributed either to retrieval failures (insufficient or irrelevant context) or to LLM failures (inability to use the provided context). However, prior work uses imprecise definitions of "relevant context," conflating topically related with answer-sufficient content. This makes it impossible to cleanly separate retrieval failures from generation failures. The paper introduces a formal notion of *sufficient context* — whether the retrieved content contains enough information to answer the query — and uses it as an analytical lens to study RAG system behavior.

## Methodology

**Sufficient Context Definition:** An instance (Q, C) has sufficient context if and only if there exists a plausible answer A' to question Q given the information in C. This is distinct from entailment (which requires a known answer A) — sufficient context does not presuppose the ground truth answer and allows for incorrect answers in context.

**Sufficient Context Autorater:** An LLM-as-judge (Gemini 1.5 Pro with 1-shot prompt) classifies each (query, context) pair as sufficient or insufficient without needing the ground truth answer. Evaluated on 115 human-annotated instances from PopQA, FreshQA, Natural Questions, and EntityQuestions. Gemini 1.5 Pro (1-shot) achieves 93% accuracy (F1: 0.935), outperforming FLAMe (24B), TRUE-NLI, and Contains GT baselines.

**Analysis:** Using the autorater, the paper stratifies three QA datasets (FreshQA, Musique-Ans, HotpotQA) by sufficient vs. insufficient context. Model responses are classified as Correct / Abstain / Hallucinate using an LLMEval pipeline (Gemini 1.5 Pro zero-shot judge comparing model answer to ground truth).

**Selective Generation Method:** Combines the binary sufficient context signal from FLAMe (efficient, small autorater) with model self-rated confidence (P(True) via sampling 20 responses; P(Correct) via self-assessment for proprietary models) in a logistic regression model trained to predict hallucinations. At inference, a threshold controls the coverage–accuracy trade-off.

**Fine-tuning ablation:** LoRA fine-tuning of Mistral-7B on Musique and HotpotQA with three data mixtures — (1) standard ground truth, (2) 20% random answers replaced with "I don't know," (3) 20% insufficient-context examples replaced with "I don't know."

**Models:** Gemini 1.5 Pro, GPT 4o, Claude 3.5 Sonnet, Gemma 27B, Mistral 3 7B.

## Dataset

- **FreshQA** (452 instances): Time-sensitive QA with oracle URLs as context (77.4% sufficient context).
- **Musique-Ans** (500 instances sampled): Multi-hop QA with 20 fixed supporting snippets (44.6% sufficient context).
- **HotpotQA** (500 instances sampled): Single and multi-hop Wikipedia QA with top-5 retrieved snippets (46.2% sufficient context).

## Evaluation Metrics

% Correct, % Abstain, % Hallucinate (LLMEval classification). Autorater: F1 Score, Accuracy, Precision, Recall. Selective generation: coverage vs. selective accuracy curves.

## Findings

- **Sufficient context rate in standard benchmarks is lower than expected:** Musique has only 44.6% and HotpotQA 46.2% sufficient context with 6k-token contexts, even with oracle or top-5 retrievals.
- **Larger models hallucinate with both sufficient and insufficient context:** With sufficient context, Gemini 1.5 Pro, GPT 4o, and Claude 3.5 all hallucinate (not just abstain) at non-trivial rates (14–17%). With insufficient context, all models hallucinate significantly more than they abstain.
- **RAG reduces abstention:** Claude 3.5 abstention drops from 84.1% (closed book) to 52% (with RAG); Gemini's drops from 100% to 18.6%. Providing any context — even insufficient — increases model confidence and reduces appropriate abstention.
- **SOTA models answer correctly 35–62% of the time even with insufficient context.** This is partially due to closed-book parametric knowledge, Yes/No question structure, limited choices, partial multi-hop fragments, and disambiguating context.
- **Selective generation improves accuracy at most coverage levels:** Combining sufficient context signal with model confidence outperforms confidence alone — gains of >10% for Gemma 27B on HotpotQA at highest accuracy regions, >5% for Gemini at 70% coverage.
- **Fine-tuning with "I don't know" examples does not reliably improve abstention over hallucination.** Fine-tuned models still hallucinate far more than they abstain, and sometimes achieve higher correct rates but at cost of even less appropriate abstention.
- **Entailment ≠ sufficient context:** TRUE-NLI has high precision but lower recall; Contains GT works surprisingly well but requires ground truth. Sufficient context is a more general and harder task.
- **Context length (2k to 6k tokens) modestly improves sufficient context rate; 6k to 10k shows negligible gain.**

## Conclusion / Discussion

The paper introduces sufficient context as a unifying concept for analyzing RAG failures, with a scalable LLM-based autorater (93% accuracy). Key conclusions: (1) even with sufficient context, LLMs hallucinate — RAG cannot be fixed by retrieval improvements alone; (2) models are overconfident when given any context, even insufficient; (3) combining sufficient context signal with model confidence enables controllable, model-agnostic selective generation that improves the precision–coverage trade-off by 2–10%.

## Limitations

- Analysis focuses on QA; summarization tasks may behave differently.
- Does not explore how different retrieval methods affect sufficient context rates.
- Selective generation approach uses FLAMe (24B) as the efficiency-focused autorater, which may introduce overhead when deployed with small LLMs.

## Future Work

- Fine-grained sufficient context scorer (scalar instead of binary) for context ranking.
- Extending sufficient context to multi-modal RAG (visual QA, document QA).
- Iterative retrieval guided by sufficient context signal.
