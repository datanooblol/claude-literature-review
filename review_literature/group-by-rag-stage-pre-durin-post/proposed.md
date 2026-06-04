# Group Literature Review by RAG Pipeline Stage (Pre / During / Post)

**Created:** 2026-06-05
**Status:** draft

## Core Idea

Organize the 11 papers by where in the RAG pipeline their primary contribution sits: pre-retrieval (document indexing and query preparation), retrieval (the fetch step itself), and post-retrieval (context processing and generation). This grouping is the most actionable for a practitioner — it tells you which stage of your own pipeline to intervene in to address a given failure mode.

Several papers span multiple stages. They are placed at their primary contribution and flagged as cross-stage where relevant.

## Relevant Papers

### Stage 1 — Pre-Retrieval (Indexing & Query Preparation)

- [[Golden-Retriever High-Fidelity Agentic Retrieval Augmented Generation]]
  - Contribution: query-side — resolves jargon and rewrites the query *before* the retrieval call
- [[Reconstructing Context Evaluating Advanced Chunking Strategies for Retrieval-Augmented Generation]]
  - Contribution: index-side — chunking strategy determines the unit that gets embedded and retrieved
- [[Parametric Retrieval Augmented Generation]]
  - Primary contribution: index-side — trains a LoRA adapter per document offline, before any query arrives
  - *(Also spans Stage 3: adapter weight merging happens right before generation)*

### Stage 2 — Retrieval (The Fetch Step)

- [[On the Influence of Context Size and Model Choice in Retrieval-Augmented Generation Systems]]
  - Contribution: retrieval design — how many snippets to retrieve (k), BM25 vs. semantic search, open vs. closed corpus
- [[Reconstructing Context Evaluating Advanced Chunking Strategies for Retrieval-Augmented Generation]]
  - *(Also spans here: rank fusion and reranking are retrieval-time operations)*
- [[Enhancing Retrieval-Augmented Generation A Study of Best Practices]]
  - Contribution: retrieval design — query expansion, retrieval stride, chunk size, KB size; Contrastive ICL
  - *(Also spans Stage 1 for indexing choices and Stage 3 for ICL strategy)*

### Stage 3 — Post-Retrieval (Context Processing & Generation)

- [[Oreo A Plug-in Context Reconstructor to Enhance Retrieval-Augmented Generation (2025)]]
  - Contribution: between retriever and generator — reconstructs retrieved chunks into concise, query-specific context
- [[REFRAG Rethinking RAG based Decoding]]
  - Contribution: compresses retrieved passage tokens into pre-computed encoder embeddings before they reach the decoder
- [[Sufficient Context A New Lens on Retrieval Augmented Generation Systems]]
  - Contribution: scores whether retrieved context is sufficient; gates generation based on sufficiency + confidence
- [[Context Length Alone Hurts LLM Performance Despite Perfect Retrieval]]
  - Contribution: generation-side — shows context length degrades LLM reasoning independently of retrieval; proposes retrieve-then-solve mitigation
- [[Needle Threading Can LLMs Follow Threads through Near-Million-Scale Haystacks]]
  - Contribution: generation-side benchmark — measures the LLM's ability to use long retrieved context
- [[Let's (not) just put things in Context Test-Time Training for Long-Context LLMs]]
  - Contribution: generation-side — adapts query projections at inference time to recover attention signal lost in long context
- [[Parametric Retrieval Augmented Generation]]
  - *(Also spans here: online weight merging into FFN happens before generation)*

## Proposed Approach / Connection

The pipeline-stage grouping is the most practically useful of the three framings (cf. [[methodology-grouping-of-rag-literature/proposed]] and [[group-literature-review-by-research-problem-or-objective/proposed]]) because it maps directly to system design decisions: fix the query, fix the fetch, or fix what the LLM does with what it gets.

The distribution of papers across stages is highly uneven:
- **Stage 1 (pre-retrieval): 2–3 papers** — relatively thin coverage; Golden-Retriever is the only paper focused exclusively here
- **Stage 2 (retrieval): 2–3 papers** — most treat retrieval as a design variable to tune, not a component to redesign
- **Stage 3 (post-retrieval/generation): 7 papers** — the most crowded cluster by far

This concentration reveals where the field has placed its bets: the biggest research investment is in *what happens after retrieval*, not in improving retrieval itself. The implicit assumption seems to be that retrieval is "good enough" and the bottleneck is in the LLM's ability to use what it receives.

Cross-referencing with the problem grouping:
- Stage 1 ≈ Problem 1 (query failures) + part of Problem 2 (retrieval design)
- Stage 2 ≈ Problem 2 (retrieval design)
- Stage 3 ≈ Problem 3 (post-retrieval quality) + Problem 4 (context length barrier)

Parametric RAG is the outlier that doesn't fit cleanly: it operates at both Stage 1 (document preparation) and Stage 3 (weight injection), bypassing Stage 2 (retrieval) almost entirely in its generation step.

## Supporting Evidence

- **Golden-Retriever** (Stage 1): 35.0% gain over standard RAG from query rewriting alone, with no model training.
- **Reconstructing Context** (Stage 1/2): Contextual retrieval beats late chunking on metrics but costs LLM inference per chunk + 20GB VRAM — a Stage 1 indexing decision with Stage 2 consequences.
- **On the Influence** (Stage 2): Poorly retrieved snippets can hurt more than no retrieval at all; BM25 beats semantic search for biomedical domain.
- **Enhancing RAG** (Stage 2): Chunk size, KB size, retrieval stride, and query expansion all have minimal impact — standard Stage 2 tuning levers are mostly inert.
- **Oreo** (Stage 3): 84–94% context compression, 37.5% EM gain — most of Stage 3's latency and noise are addressable with a T5-small module.
- **REFRAG** (Stage 3): 30.85× TTFT speedup by compressing Stage 3 input before decoding.
- **Sufficient Context** (Stage 3): Even with sufficient context at Stage 3 entry, LLMs hallucinate 14–17% of the time.
- **Context Length Alone Hurts** (Stage 3): 13.9–85% accuracy drop at Stage 3 from length alone, even with perfect Stage 2 retrieval.
- **qTTT** (Stage 3): Gradient steps at inference outperform FLOP-matched thinking tokens for long-context Stage 3 processing.

## Open Questions

- Stage 2 papers (On the Influence, Enhancing RAG) find that many retrieval hyperparameters barely matter. Does this mean Stage 2 is already "solved enough," or that the benchmarks used are insensitive to Stage 2 variation?
- If Stage 3 is where most failures occur (per Problem 4 papers), does investing in Stage 1 and Stage 2 improvements give diminishing returns? Or are Stage 1 failures (e.g., query jargon) a separate failure mode that Stage 3 fixes cannot compensate for?
- Parametric RAG bypasses Stage 2 for generation but still uses Stage 2 (BM25) to select which adapters to merge. Is this a genuine Stage 2 bypass or a Stage 2 dependency in disguise?
- Is the Stage 3 concentration in the literature a reflection of where the hard problems are, or a reflection of research incentives (easier to publish a new post-retrieval module)?

## Next Steps

- Build a 3×3 matrix: Stage (1/2/3) × Methodology (empirical/prompting/training) to identify under-studied cells
- Cross-reference with [[group-literature-review-by-research-problem-or-objective/proposed]] to check alignment between stages and problem clusters
- Draft a section using the pipeline-stage framing as the organizing structure — could serve as a tutorial-style survey section
