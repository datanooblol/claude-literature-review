# Needle Threading: Can LLMs Follow Threads through Near-Million-Scale Haystacks?

**Source:** `raw/Needle Threading Can LLMs Follow Threads through Near-Million-Scale Haystacks.pdf`
**Authors:** Jonathan Roberts, Kai Han, Samuel Albanie
**Publisher / Venue:** ICLR 2025 (International Conference on Learning Representations)
**Year:** 2025
**Summarized:** 2026-06-05

## Research Problem / Objective

Existing long-context LLM benchmarks have three key limitations: (1) performance saturation — frontier models achieve near-perfect scores on simple needle-in-a-haystack tasks; (2) limited context length — most benchmarks cap at sub-100k tokens, far below frontier model context limits; (3) lack of granular takeaways — aggregating tasks obscures the variables driving performance. The paper addresses these gaps with challenging retrieval tasks up to 630k tokens (LLaMA 3.1 tokenizer), including novel multi-step threading tasks. A secondary finding is that tokenizer differences make cross-model token counts incomparable.

## Methodology

**Synthetic haystack construction:** Key-value pairs of random UUIDs (32-character hexadecimal strings) serialized as JSON. This removes natural language semantics and enables fine-grained control over context length, needle placement, and task difficulty.

**Tasks (five core, one variant):**
1. **Single Needle**: Retrieve the value for one specified key from haystacks at fixed depth percentages (10% increments).
2. **Multiple Needles**: Retrieve values for 2–25 keys, either randomly placed or clustered.
3. **Conditional Needles**: Retrieve all values whose keys contain a special character ('*' or '&'), without being told which keys qualify.
4. **Threading**: Follow a chain of key-value pointers (K→V=K'→V') through the context for 2–25 steps, in forward, backward, or random direction.
5. **Multi-Threading**: Follow multiple simultaneous threads (2–5 threads) of varying lengths.
6. **Branched Threading** (variant): At each thread step, multiple keys equal the previous value; follow the longest chain.

**Natural language ablation:** Threading tasks replicated with historical text (Gibbon's Decline and Fall) using artificially injected linked sentences.

**Models evaluated:** 17 LLMs including GPT-4o/mini, Gemini 1.5 Flash/Pro, Claude 3/3.5 Sonnet/Haiku, LLaMA 3.1 (8B/70B/405B), Mistral Large/Nemo, Jamba 1.5, Reka Core/Flash. Evaluation on up to 630k tokens (LLaMA 3.1 tokenizer). Zero-shot, greedy decoding.

**Effective context length metric:** Contour-based, task-specific metric at 75% accuracy threshold, expressed in characters (model-agnostic) rather than tokens.

## Dataset

Synthetically generated UUID key-value haystacks at 12 sizes from 1k to 630k LLaMA 3.1 tokens. Natural language ablation uses The History of the Decline and Fall of the Roman Empire.

## Evaluation Metrics

Accuracy (%) per task, exact match (via Gemini 1.5 Flash as parser). Effective context length (characters). Tokenization comparison across models.

## Findings

- **Single needle**: Retrieval accuracy decreases at longer context lengths for most models; accuracy is lowest towards the middle of the context (consistent with "lost in the middle"). GPT-4o and Jamba 1.5 Large achieve near-perfect scores throughout.
- **Multiple needles**: Context length has far more impact on performance than the number of needles; stronger models are largely unaffected by needle count.
- **Conditional needles**: Clustered placement significantly outperforms random placement; performance worsens more steeply at shorter lengths than multiple needles.
- **Threading**: Performance drops substantially with both increasing context length and thread length. Forward-travelling threads are easier than backward-travelling threads for all models except Claude 3.5 Sonnet. Many models plateau near zero at longer contexts.
- **Multi-threading**: LLMs are remarkably **thread-safe** — simultaneously following multiple threads (up to 25) does not significantly degrade per-thread accuracy in the synthetic setting. However, multi-threading in natural language text is much harder.
- **Effective context length**: Most models have an effective context limit far shorter than their advertised limit. For threading tasks at 5 steps, all models except GPT-4o (3%) and LLaMA 3.1 405b (1%) show 0% effective context length.
- **Tokenization**: GPT-4o uses ~50 tokens per UUID pair; Gemini 1.5 uses ~75. Gemini 1.5 Flash's 1M token context ≈ 700k GPT-4o tokens. Token counts across models are not directly comparable.
- **Best model by context size**: GPT-4o at 1–2k; Claude 3.5 Sonnet at 2.5–64k; Gemini 1.5 Pro at longer contexts. Closed-source models consistently outperform open-source.

## Conclusion / Discussion

Frontier LLMs can perform simple multi-needle retrieval well at shorter contexts but show clear performance degradation as context grows. Threading (multi-step reasoning retrieval) is much harder, with most models essentially failing at long contexts. The "thread-safe" finding — that models track multiple threads as well as single threads — is a surprising positive capability. The effective context limit metric reveals that advertised context lengths are overstated as usable limits. Tokenization incomparability is an underappreciated issue in comparing models by context length.

## Limitations

- Restricted to synthetic UUID data; does not capture domain-specific LLM behaviours.
- Cost limited the number of experimental repeats for the largest models.
- Some models (Mistral) could not be tested near their context limits due to API restrictions.

## Future Work

Not explicitly stated, but the authors release code and datasets to encourage further long-context understanding research.
