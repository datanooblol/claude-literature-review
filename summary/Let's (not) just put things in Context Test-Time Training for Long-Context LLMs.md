# Let's (not) just put things in Context: Test-Time Training for Long-Context LLMs

**Source:** `raw/Let's (not) just put things in Context Test-Time Training for Long-Context LLMs.pdf`
**Authors:** Rachit Bansal, Aston Zhang, Rishabh Tiwari, Lovish Madaan, Sai Surya Duvvuri, Devvrit Khatri, David Brandfonbrener, David Alvarez-Melis, Prajjwal Bhargava, Mihir Sanjay Kale, Samy Jelassi
**Publisher / Venue:** arXiv
**Year:** 2025
**Summarized:** 2026-06-05

## Research Problem / Objective

Long-context LLMs can consume far more text than they can reliably use. Despite context windows now reaching millions of tokens, models still fail to retrieve relevant information buried in long contexts ("lost in the middle"). Inference-time compute strategies such as chain-of-thought "thinking" tokens show rapid diminishing returns as context length grows. The paper seeks to explain this failure and find a more effective use of inference-time compute for long-context retrieval and reasoning.

## Methodology

**Theoretical analysis — Score Dilution:**  
The authors formalize the failure of static self-attention as *score dilution*: when many distractor tokens are near-tied with the target (needle) in logit space, the softmax attention mass on the target vanishes as context length T grows. They prove a *logarithmic margin requirement*: to guarantee fixed target attention mass against worst-case distractors, the target–distractor logit gap must scale as Ω(log T). They further prove that autoregressively generated "thinking" tokens cannot repair this deficit because each generated token carries at most its own attention mass on the needle.

**Proposed method — Query-Only Test-Time Training (qTTT):**  
qTTT directly addresses score dilution by adapting only the query projection matrices (W_Q) of the attention layers, while keeping keys and values fixed.

Algorithm:
1. *Single prefill*: Perform one full forward pass to compute and cache all Key/Value tensors {K^(ℓ), V^(ℓ)} from the long context. This is an O(T²) operation done once.
2. *Span-sampled query updates*: For N_TTT steps, randomly sample a short span x_s of length k ≪ T from the context; compute the next-token prediction loss using the frozen {K, V} cache; backpropagate and update only {W_Q^(ℓ)}.

Theoretical justification: The gradient ∇_{q_i} ℓ_i = (1/√d_k)(μ_i − k_{j*}), where μ_i is the attention-weighted centroid of all keys. A gradient descent step moves q_i toward the needle key k_{j*} and away from μ_i, provably increasing the target–distractor logit margin (Lemma 3.2).

**FLOP matching:** The rule T_think ≈ 2·N_qTTT·k equates compute between thinking tokens and qTTT, allowing fair comparison.

## Dataset

**Synthetic tasks (controlled analysis):**
- Bug localization in a code repository (OLMo codebase): single-line bug injection, context varies from 5 to 10,000 lines.
- Anomaly detection in banking transaction logs: 25–500 transactions.

**Real-world benchmarks:**
- **LongBench-v2**: 6 subsets — Code Repositories, Long Dialogue History, Long Structured Data, Long In-Context, Multi-Document QA, Single-Document QA.
- **ZeroScrolls**: 8 datasets — GovReport, MuSiQue, NarrativeQA, QASPER, QMSum, QUALITY, SQuALITY, SummScreen-FD.

## Evaluation Metrics

Task-specific accuracy for synthetic tasks; benchmark-defined metrics per subset (EM/F1, ROUGE, multiple-choice accuracy). All comparisons use FLOP-matched inference budgets.

## Findings

- In-context accuracy drops sharply and monotonically as context length increases on synthetic tasks; thinking tokens improve performance for short contexts but show clear diminishing returns and converge to near-baseline at large T.
- qTTT consistently improves performance across all context lengths on synthetic tasks, maintaining significantly higher attention mass on target tokens as T grows.
- On LongBench-v2 (Qwen3-4B), qTTT yields average accuracy 39.6% vs. 33.5% with thinking and 27.0% in-context only. On ZeroScrolls, 32.5% vs. 23.9% (thinking) and 18.4% (in-context).
- Gains are largest on retrieval-driven tasks (Long Dialogue History: 20.5→30.8→43.6; Multi-Document QA: 30.0→40.0→46.0 for in-context/thinking/qTTT) and smallest on summarization tasks.
- qTTT improvements scale with model size and hold for Qwen3-{1.7B, 4B, 8B, 32B}.
- Disabling RoPE (positional encoding ablation) accelerates attention mass collapse, but even with RoPE the decline is substantial, supporting score dilution as the primary failure mode.
- qTTT is competitive with or better than strictly FLOP-matched Best-of-N and Beam Search; SC-N helps when per-sample accuracy is already high but often degrades below 50% accuracy.
- Wall-clock time for qTTT, thinking, and best-of-N is similar; prefill time dominates for long contexts, motivating the frozen KV cache design.
- qTTT is not very sensitive to learning rate in the range [1e-6, 1e-5]; performance degrades at extreme values.

## Conclusion / Discussion

Score dilution in static quadratic self-attention is identified as a core cause of long-context failures. qTTT reallocates inference-time compute from generating more tokens to a small number of query-only gradient updates that provably increase the target–distractor logit margin. This simple shift in how inference-time compute is spent yields consistently large performance improvements across models and benchmarks without any changes to pre-training, architecture, or data. The practical takeaway: for long context, context-specific training of query projections is a better use of inference compute than producing more thinking tokens.

## Limitations

- qTTT requires gradient computation at inference time, which increases implementation complexity compared to standard decoding.
- Gains are task-dependent: summarization tasks see smaller improvements (retrieval is not the bottleneck there).
- The paper evaluates a single operating point on the (k, N_TTT) trade-off; optimal budget scheduling is unexplored.
- Only tested on Qwen3 model family (1.7B–32B); generalization to other architectures is not demonstrated.

## Future Work

- Explore budget schedules across span size and number of TTT steps.
- Extend compute-matched comparison to self-consistency and best-of-n within the same framework.
- Develop simple predictors for when to prefer qTTT over decoding-based scaling.
- Apply qTTT on top of other long-context strategies (e.g., sliding window attention, RAG).
