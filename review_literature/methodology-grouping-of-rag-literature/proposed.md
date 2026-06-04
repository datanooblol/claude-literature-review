# Methodology Grouping of RAG Literature

**Created:** 2026-06-05
**Status:** draft

## Core Idea

The 11 papers in this corpus can be organized along a methodological axis into three groups: training-based approaches, prompting/inference-time approaches, and empirical/benchmarking studies. This grouping reveals a natural progression — empirical studies diagnose the problems, prompting methods patch them cheaply at inference time, and training-based methods address them at a deeper level.

## Relevant Papers

### Group 1 — Training-based Approaches
- [[Parametric Retrieval Augmented Generation]]
- [[Oreo A Plug-in Context Reconstructor to Enhance Retrieval-Augmented Generation (2025)]]
- [[REFRAG Rethinking RAG based Decoding]]
- [[Let's (not) just put things in Context Test-Time Training for Long-Context LLMs]]

### Group 2 — Prompting / Inference-time Methods
- [[Golden-Retriever High-Fidelity Agentic Retrieval Augmented Generation]]
- [[Sufficient Context A New Lens on Retrieval Augmented Generation Systems]]
- [[Context Length Alone Hurts LLM Performance Despite Perfect Retrieval]]

### Group 3 — Empirical / Benchmarking Studies
- [[Needle Threading Can LLMs Follow Threads through Near-Million-Scale Haystacks]]
- [[Context Length Alone Hurts LLM Performance Despite Perfect Retrieval]]
- [[On the Influence of Context Size and Model Choice in Retrieval-Augmented Generation Systems]]
- [[Reconstructing Context Evaluating Advanced Chunking Strategies for Retrieval-Augmented Generation]]
- [[Enhancing Retrieval-Augmented Generation A Study of Best Practices]]

## Proposed Approach / Connection

The three groups form a problem→patch→solve arc:

- **Group 3 (Empirical)** establishes that context length alone degrades LLM performance, that effective context windows are far shorter than advertised, and that design choices like chunking and context size have measurable but complex effects.
- **Group 2 (Prompting)** responds with inference-time fixes: better query formulation (Golden-Retriever), selective generation based on retrieval sufficiency (Sufficient Context), and evidence recitation before answering (Context Length Alone Hurts).
- **Group 1 (Training)** goes deeper: compressing retrieved content into learned representations (REFRAG, Oreo), injecting knowledge into model parameters (Parametric RAG), or updating query projections at test time (qTTT).

A key tension across groups: **cost vs. depth**. Prompting methods are cheap to deploy but limited by the LLM's fixed weights. Training methods achieve stronger gains but require offline infrastructure. This trade-off is underexplored as an explicit design choice in the literature.

Note: *Context Length Alone Hurts* appears in both Group 2 and Group 3 — its primary contribution is empirical, but it also proposes a prompting-based mitigation.

## Supporting Evidence

- **Parametric RAG**: 29–36% lower inference latency vs. in-context RAG; combining parametric + in-context achieves highest overall performance.
- **Oreo**: Up to 37.5% EM improvement on NQ; 84–94% context compression; 23–43% latency reduction — all from a plug-in T5-small module.
- **REFRAG**: 30.85× TTFT acceleration over LLaMA; 3.75× over previous SOTA at k=32 compression.
- **qTTT**: Up to 14.1% average gain over FLOP-matched thinking tokens on LongBench-v2; retrieval-driven tasks benefit most.
- **Golden-Retriever**: 57.3% avg improvement over Vanilla LLM; 35.0% over Vanilla RAG — with no model training.
- **Sufficient Context**: Even with sufficient context, LLMs hallucinate 14–17% of the time; selective generation improves accuracy 2–10%.
- **Context Length Alone Hurts**: Accuracy drops 13.9–85% from context length alone, even with perfect retrieval and masked distractors.
- **Needle Threading**: Most models' effective context length is far below their advertised limit.
- **On the Influence**: Performance saturates at ~15 snippets; BM25 beats semantic search for precision-critical domains.
- **Reconstructing Context**: Contextual retrieval outperforms late chunking on metrics but at high compute cost.
- **Enhancing RAG**: Contrastive ICL (correct + incorrect examples) is best across all metrics; chunk size has minimal impact.

## Open Questions

- Is the cost/depth trade-off between prompting and training methods made explicit anywhere in the literature, or is it implicit?
- Can training-based methods (Oreo, REFRAG) be combined with inference-time methods (Golden-Retriever, Sufficient Context) without additive latency?
- Does the empirical finding that context length alone hurts motivate the training-based compression approaches (REFRAG, Oreo) — or were they developed independently?
- qTTT uses gradient steps at test time — how does its cost profile compare to the prompting-only group?

## Next Steps

- Read summaries of Oreo and REFRAG side-by-side to compare compression strategies in detail
- Check if any paper explicitly frames its contribution against the cost/depth trade-off
- Consider a follow-up idea document on the compression sub-theme (Oreo, REFRAG, Parametric RAG)
- Draft a narrative paragraph using the problem→patch→solve arc as the organizing spine
