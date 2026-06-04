# Context Length Alone Hurts LLM Performance Despite Perfect Retrieval

**Source:** `raw/Context Length Alone Hurts LLM Performance Despite Perfect Retrieval.pdf`
**Authors:** Yufeng Du, Minyang Tian, Srikanth Ronanki, Subendhu Rongali, Sravan Bodapati, Aram Galstyan, Azton Wells, Roy Schwartz, Eliu A Huerta, Hao Peng
**Publisher / Venue:** arXiv
**Year:** 2025
**Summarized:** 2026-06-05

## Research Problem / Objective

Long-context LLM performance gaps are widely attributed to retrieval failures — the model's inability to identify relevant information in long inputs. This paper tests the assumption that perfect retrieval is sufficient for good task performance. It asks: even when a model can perfectly retrieve all relevant evidence (verified by 100% exact match), does it perform as well as on a short-context version of the same problem?

## Methodology

**Synthetic long-context benchmark construction:**
Each problem is split into *evidence* (all information needed to solve the task) and *question* (query + format). Distraction tokens are inserted between them to create the format: `[Evidence] [Distraction Tokens] [Question]`. Evidence is placed at the start (easiest-to-retrieve position per the Lost-in-the-Middle effect). Three types of distractors are tested in increasing order of "cleanliness":
1. **Essay tokens** (Paul Graham Essays): natural language distraction
2. **Whitespace**: minimally informative tokens
3. **Masking**: all distractor tokens masked from attention, model attends only to evidence and question

**Tasks:** Variable Summation (VarSum), GSM8K (math), MMLU (multiple-choice QA), HumanEval (code generation).

**Retrieval measurement:** The model is separately prompted to recite both the evidence and question exactly (evaluated by exact match) to establish whether retrieval was perfect on those specific instances.

**Mitigation strategy (Retrieve-then-Solve):** First prompt the model to recite the retrieved evidence, then use only the recited evidence + question (without the long context) as a new shorter prompt for final answer generation. This converts a long-context task into a short-context one.

**Models tested:** Llama-v3.1-8B-Instruct, Mistral-v0.3-7B-Instruct (open-source), GPT-4o, Claude-3.7-Sonnet, Gemini-2.0 (closed-source).

## Dataset

- **VarSum**: Synthetic variable summation task
- **GSM8K**: Math word problems
- **MMLU**: Massive multitask language understanding (multiple choice)
- **HumanEval**: Code generation
- **RULER** (QA1, QA2): Established long-context benchmark for mitigation evaluation

## Evaluation Metrics

Task accuracy (EM/pass@1/ROUGE-based). Retrieval measured by exact match of recited evidence vs. original. Performance reported as absolute baseline at 0-token context and as deltas at longer lengths.

## Findings

- Across all 5 models and all 4 tasks, accuracy degrades substantially as context length increases (13.9%–85%), even when retrieval is perfect by exact match.
- Open-source models (Llama, Mistral): large drops begin well below 30K tokens; Llama-MMLU drops 24.2% at 30K while retrieval drops only 0%. HumanEval retrieval for Mistral actually improves at 30K while accuracy keeps falling.
- Whitespace distraction: drops persist (up to 48% for Llama-VarSum, 30% for Mistral-GSM8K at 30K whitespace tokens).
- **Masking experiment**: even when all distractors are masked and the model attends only to evidence and question (identical to short-context attention, but with greater positional distance), performance still drops consistently — at least 7.9% for both models at 30K masked tokens, with Llama-HumanEval suffering a 50% drop.
- Closed-source models (GPT-4o, Claude, Gemini) show more robustness but still exhibit substantial, mostly consistent degradation (e.g., Claude-3.5-MMLU: –67.6% at 30K whitespace).
- Evidence position control (placing evidence at the end just before the question) still shows substantial drops (up to 17–20% at 30K whitespace).
- **Retrieve-then-Solve mitigation**: Mistral on GSM8K improves by up to 31.2% (66.7% vs. 35.5% at 26K tokens); GPT-4o on RULER QA2 improves by up to 4% at 32K, consistently across all lengths.

## Conclusion / Discussion

The paper reveals a previously underappreciated limitation: the sheer length of the input alone degrades LLM reasoning performance, independent of retrieval quality and distraction content. This challenges the standard decomposition of long-context capability into "retrieval" + "problem solving," showing that improving retrieval alone does not guarantee better task performance. The authors attribute the masking-condition drop to a training-induced positional distribution bias. The retrieve-then-solve strategy (recite evidence → short context) is a simple, model-agnostic mitigation. These findings also help explain why RAG performance degrades as more documents are added, and why long CoTs can hurt reasoning models.

## Limitations

- Only 2 open-source and 3 closed-source models tested, and 4 tasks.
- Retrieval evaluation not run on closed-source models at RULER (models occasionally refused to recite long inputs).
- Mitigation strategy requires accurate retrieval as a prerequisite, which is hard in real-world settings harder than the synthetic data.

## Future Work

- Holistic long-context evaluation beyond retrieval-focused benchmarks.
- Fine-grained analysis of the underlying mechanisms (positional bias, attention patterns) causing length-induced degradation.
- Model training strategies that independently improve both retrieval and utilization without trade-offs.
