---
last_verified: 2026-06-14
gardener_sources:
  - https://arxiv.org/abs/2203.11171
  - https://arxiv.org/abs/2408.03314
  - https://huggingface.co/docs/transformers/generation_strategies
---

# Reasoning & Test-Time Compute

## What & why

For most of deep learning's history, the dominant scaling lever was *training-time
compute*: train on more data, with more parameters, for more steps. The resulting
model is then a fixed artifact — every query gets the same model, the same number of
forward passes, and the same wall-clock time to answer.

**Test-time compute** breaks this assumption. Rather than fixing the work done at
inference, you let harder problems use more compute: more samples, longer reasoning
chains, more rounds of checking. The model's effective capability scales with the
inference budget you're willing to spend.

The shift matters because verification is often cheaper than generation. A model that
can reliably check whether an answer is correct can be made much more capable by
generating many candidates and selecting the best one — even if the generator alone
is mediocre.

## How it actually works

### Chain-of-thought

The simplest form: ask the model to reason step-by-step before giving its answer.
This works because transformer forward passes are shallow — each token is generated in
one pass through the network. Spreading reasoning across many tokens effectively
gives the model more "scratch-pad" computation. Prompting with "think step by step"
or providing reasoning examples in context both trigger this behavior in capable
models. The cost is output tokens; the gain is accuracy on multi-step problems.

### Sampling + voting (self-consistency)

Generate `k` independent solutions to the same problem (usually with non-zero
temperature — see [Sampling & decoding](../foundations/sampling.md)) and aggregate by
majority vote on the final answer. This works when:

1. The model generates correct answers *sometimes* — errors are not systematic.
2. There is a compact final answer to vote on (math result, a choice label, a
   verifiable string).

The gains follow a roughly logarithmic curve with `k`: going from 1 to 8 samples
helps a lot; going from 32 to 64 helps less. There is a ceiling where the model's
systematic errors dominate over random ones.

### Verifier-guided search

Instead of voting on outputs, train or use a **verifier** (also called a process
reward model or outcome reward model) that scores candidate solutions. Generate many
candidates; re-rank by verifier score; keep the top. This is more powerful than naive
voting when:

- The answer space is too large for majority vote (e.g., open-ended generation).
- A reliable verifier exists (formal checkers, unit tests, another LLM, a learned
  reward model — see [Preference modeling](../evals/preference-modeling.md)).

The verifier's quality is the ceiling. A weak or gameable verifier leads to
**reward hacking**: the generator finds high-scoring outputs that aren't actually
correct.

Search can also be structured as **beam search**, **Monte Carlo Tree Search (MCTS)**,
or other explicit planning algorithms, with the verifier scoring nodes in the tree.

### Test-time training (TTT) / test-time adaptation

The most aggressive form: actually update the model's weights at inference time based
on information in the input. For a task with a few labeled examples (few-shot context),
TTT fine-tunes a small adapter (often LoRA — see [Finetuning](finetuning.md)) on
those examples before predicting. The "pretrained base → adapted weights → prediction"
pipeline now happens per-input rather than once before deployment.

This is expensive (a mini-training run per query) and its practicality depends heavily
on:

- **Augmentation quality** — if the few examples can be reliably augmented into many
  training pairs, TTT is very powerful. If augmentation is noisy, it overfits fast.
- **Adaptation speed** — small models with LoRA adapters can adapt in seconds to
  minutes; full-scale models cannot.
- **Task structure** — TTT particularly shines on tasks with strong internal
  consistency constraints (e.g., learning the rule that governs a pattern) rather than
  tasks requiring external world knowledge.

## Cost trade-offs

Test-time compute is not free. The practical questions:

| Approach | Relative inference cost | Best when |
|---|---|---|
| Chain-of-thought | 2–10× token cost | Multi-step reasoning; cheap to prompt |
| Self-consistency (k samples) | k× token cost | Cheap verifier (majority vote); errors are random |
| Verifier-guided search | k× generation + verifier cost | Reliable verifier exists; errors are systematic |
| Test-time training | Training run per input | Very hard tasks; few labeled demos available; latency-insensitive |

**The honest take:** for most production tasks, the right answer is chain-of-thought
plus a small `k` of samples with voting. Full TTT is currently a research technique
and a tool for domains where the task structure strongly favors it. The value of any
test-time method degrades if the base model's accuracy on the task is already near
ceiling — more samples won't save a fundamentally wrong approach.

## Interview questions

After this page you should be able to answer:

1. Why does chain-of-thought prompting improve accuracy on multi-step problems? What is the mechanism — is the model "thinking harder"?
2. Self-consistency generates k answers and takes a majority vote. What two conditions must hold for this to outperform a single best-of-one sample?
3. What is a process reward model, and how does it differ from an outcome reward model? When does each make sense?
4. A verifier-guided search system is producing high-scoring outputs that are wrong. What is this called, and what is the root cause?
5. Test-time training adapts model weights per input. Name two settings where it is likely to be worth the cost, and one where it probably isn't.

---

*Last verified 2026-06-14 · maintained by the Gardener*
