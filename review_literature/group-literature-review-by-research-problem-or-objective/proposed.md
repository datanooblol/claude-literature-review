# Group Literature Review by Research Problem or Objective

**Created:** 2026-06-05
**Status:** draft

## Core Idea

Organize the 11 papers by the specific failure mode or gap in RAG systems that each paper targets, rather than by the methodology used to address it. This grouping reveals which parts of the RAG pipeline are most contested in the literature, and surfaces papers that are directly comparable because they attack the same problem from different angles.

Five problem clusters emerge: query-side failures before retrieval, retrieval design choices, post-retrieval context quality, context length as a fundamental barrier, and knowledge integration beyond in-context injection.

## Relevant Papers

### Problem 1 — Query-Side Failures (pre-retrieval)
- [[Golden-Retriever High-Fidelity Agentic Retrieval Augmented Generation]]
  - *Problem:* LLMs misinterpret domain jargon and abbreviations in user queries, causing poor retrieval that post-retrieval fixes cannot correct.

### Problem 2 — Retrieval Design: What and How to Retrieve
- [[Reconstructing Context Evaluating Advanced Chunking Strategies for Retrieval-Augmented Generation]]
  - *Problem:* Fixed-size chunking fragments semantic context; late chunking vs. contextual retrieval trade-offs unstudied in full pipelines.
- [[Enhancing Retrieval-Augmented Generation A Study of Best Practices]]
  - *Problem:* Impact of individual RAG design choices (chunk size, KB size, ICL strategy, etc.) not systematically benchmarked.
- [[On the Influence of Context Size and Model Choice in Retrieval-Augmented Generation Systems]]
  - *Problem:* No systematic study of how context size, LLM choice, and retrieval method jointly affect RAG on long-form QA.

### Problem 3 — Post-Retrieval Context Quality
- [[Oreo A Plug-in Context Reconstructor to Enhance Retrieval-Augmented Generation (2025)]]
  - *Problem:* Retrieved chunks contain noise, redundancy, and scattered evidence; retriever and generator are misaligned due to independent optimization.
- [[Sufficient Context A New Lens on Retrieval Augmented Generation Systems]]
  - *Problem:* No formal definition of "sufficient context" — impossible to cleanly separate retrieval failures from generation failures.
- [[REFRAG Rethinking RAG based Decoding]]
  - *Problem:* Retrieved passages dominate context length causing high TTFT latency; RAG-specific block-diagonal attention structure not exploited.

### Problem 4 — Context Length as a Fundamental Barrier
- [[Context Length Alone Hurts LLM Performance Despite Perfect Retrieval]]
  - *Problem:* Even with perfect retrieval, input length alone degrades LLM performance — the role of length independent of retrieval is unexplored.
- [[Needle Threading Can LLMs Follow Threads through Near-Million-Scale Haystacks]]
  - *Problem:* Existing benchmarks saturate too quickly and cap at sub-100k contexts — true effective limits of frontier LLMs unknown.
- [[Let's (not) just put things in Context Test-Time Training for Long-Context LLMs]]
  - *Problem:* Long-context LLMs fail to retrieve relevant information in long inputs; thinking tokens show rapid diminishing returns as context grows.

### Problem 5 — Knowledge Integration Beyond In-Context Injection
- [[Parametric Retrieval Augmented Generation]]
  - *Problem:* All RAG methods inject knowledge via input context, but LLMs store knowledge in parameters — causing a fundamental inefficiency and utilization gap.

## Proposed Approach / Connection

This grouping is most useful for *comparing solutions to the same problem*. Problems 3 and 4 are the most contested clusters:

- **Problem 3** has three papers attacking post-retrieval quality from three different angles: Oreo reconstructs context via a learned module, Sufficient Context adds a formal diagnostic layer, and REFRAG compresses context at the architecture level. These three are directly comparable.
- **Problem 4** has three papers converging on the same empirical insight (length hurts, regardless of retrieval) but via different evidence: synthetic controlled experiments (Context Length Alone Hurts), near-million-token benchmarks (Needle Threading), and theoretical analysis of score dilution (qTTT).

Problem 5 (Parametric RAG) stands alone — it is the only paper that challenges the foundational assumption of in-context knowledge injection rather than optimizing within it.

Cross-referencing with the methodology grouping ([[methodology-grouping-of-rag-literature/proposed]]):
- Problem 4 maps almost entirely onto the empirical/benchmarking group
- Problem 3 maps across prompting methods (Sufficient Context) and training methods (Oreo, REFRAG)
- Problem 2 maps onto empirical/ablation studies
- Problems 1 and 5 are each a singleton in the methodology grouping

The problem-based grouping adds value over the methodology grouping by revealing *what the field agrees is broken* versus *how different research traditions choose to fix it*.

## Supporting Evidence

- **Golden-Retriever** (Problem 1): 35% gain over standard RAG from query rewriting alone — confirms pre-retrieval failure is a distinct and fixable failure mode.
- **Reconstructing Context** (Problem 2): Neither late chunking nor contextual retrieval dominates; the right choice depends on available compute, not quality alone.
- **Enhancing RAG** (Problem 2): Chunk size, KB size, retrieval stride, and query expansion all have minimal impact — the field has been tuning the wrong knobs.
- **On the Influence** (Problem 2): Poorly retrieved snippets can hurt worse than no retrieval at all (GPT-4o, Mixtral at low k); domain fit of the LLM matters more than model size.
- **Oreo** (Problem 3): 84–94% context compression with up to 37.5% EM gain — most retrieved content is noise.
- **Sufficient Context** (Problem 3): 14–17% hallucination rate even when context is sufficient — generation failure is independent of retrieval failure.
- **REFRAG** (Problem 3): Block-diagonal attention confirmed empirically; 30.85× TTFT acceleration by exploiting it.
- **Context Length Alone Hurts** (Problem 4): 13.9–85% accuracy drop from length alone, even with masked distractors.
- **Needle Threading** (Problem 4): Effective context length far below advertised for all but 2 models on threading tasks.
- **qTTT** (Problem 4): Score dilution provably scales as Ω(log T); query-only gradient steps at inference outperform FLOP-matched thinking tokens.
- **Parametric RAG** (Problem 5): Parametric injection reduces inference time 29–36% while outperforming in-context RAG on most benchmarks.

## Open Questions

- Problem 4 has the most convergent evidence — does the field treat this as settled, or are there papers pushing back?
- Is Problem 5 (Parametric RAG) truly isolated, or are there other papers outside this corpus that also challenge in-context injection as the default paradigm?
- Problems 3 and 4 overlap: if context length hurts even with perfect retrieval (Problem 4), then compressing context (Problem 3) may be addressing a symptom rather than the root cause. Does any paper make this connection explicitly?
- Problem 2 (retrieval design) papers find minimal effects for many hyperparameters — does that mean the field should shift focus entirely to Problems 3 and 4?

## Next Steps

- Cross-reference this grouping with [[methodology-grouping-of-rag-literature/proposed]] to see which cells of a problem × methodology matrix are empty (potential research gaps)
- Draft a section using Problem 4 as the spine — it has the strongest convergent evidence and the clearest narrative
- Consider whether Problem 5 deserves its own standalone idea document given its architectural distinctiveness
