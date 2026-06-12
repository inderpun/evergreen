---
last_verified: 2026-06-12
gardener_sources:
  - https://huggingface.co/docs/transformers/generation_strategies
  - https://github.com/huggingface/transformers/blob/main/src/transformers/generation/configuration_utils.py
  - https://arxiv.org/abs/1904.09751
---

# Sampling & Decoding

## What & why

At every step of generation, a language model produces a **probability distribution
over its entire vocabulary** — a score for every token it could write next. Decoding
is the strategy that turns this distribution into a choice. That choice shapes
whether your model is creative or deterministic, verbose or concise, coherent or
repetitive.

The parameters (temperature, top-p, top-k) are levers on that distribution. Most
developers set them once in a config, rarely think about them, and then wonder why
the model behaves unexpectedly. This page explains the mechanism so you can reason
about them instead of guessing.

## How it actually works

### The raw logits

The model outputs **logits** — raw unnormalized scores, one per vocabulary token.
Softmax converts these into probabilities: `p(token) = exp(logit) / sum(exp(all_logits))`.
With a large vocabulary, most probabilities are vanishingly small; the top few tokens
hold most of the mass.

### Greedy decoding

Pick the highest-probability token at every step. Simple, deterministic, fast. The
failure mode: it is myopic. Choosing the locally best token at step *t* may close off
globally better sequences by step *t+5*. Greedy decoding tends to produce repetitive,
generic outputs for open-ended tasks; it is appropriate for tasks with a clearly
correct answer (classification, extraction, code generation with tests).

### Temperature

Temperature is a scalar applied to the logits *before* softmax:
`p(token) = softmax(logit / T)`.

- **T < 1** (e.g., 0.2): sharpens the distribution. High-probability tokens get
  relatively more mass; unlikely tokens get crushed. More deterministic, more
  "obvious" outputs.
- **T = 1**: the raw model distribution — no modification.
- **T > 1** (e.g., 1.5): flattens the distribution. Unlikely tokens get relatively
  more mass. More diverse, more surprising, more likely to be wrong.

**The T=0 myth:** Setting temperature to 0 does not make the model "perfectly
correct" — it makes it greedy. It will still hallucinate; it will just hallucinate
the same thing every time. For tasks like extraction or summarization, low temperature
reduces variance but does not guarantee accuracy. Test against your actual task.

### Top-k sampling

After temperature scaling, keep only the *k* most probable tokens. Zero out the rest.
Renormalize and sample. This prevents the model from ever sampling from the long tail
of implausible tokens.

Problem: *k* is a fixed count regardless of the shape of the distribution. If the
model is very confident, the top 10 tokens might cover 99% of the probability mass
and the 10th is already implausible. If the model is uncertain, the top 10 might
cover only 30%. A fixed *k* handles neither case well.

### Top-p (nucleus) sampling

Instead of a fixed count, keep the smallest set of tokens whose cumulative probability
exceeds *p* (e.g., 0.9). This adapts to the distribution's shape: when the model is
confident, the nucleus is small; when uncertain, it is larger.

Top-p is generally preferred over top-k for open-ended generation. A common default
is p=0.9 to 0.95 with temperature=1.0 or slightly below.

### Combining them

Temperature, top-k, and top-p are typically applied in sequence: temperature scaling
→ top-k truncation → top-p truncation → sample. Many frameworks apply all three;
you can disable any by setting it to its passthrough value (T=1, k=0, p=1).

## Choosing settings for your use case

| Task | Recommended starting point | Reasoning |
|---|---|---|
| Factual Q&A, extraction, classification | Temperature 0–0.3, no top-p | Determinism; "correct" answer is well-defined |
| Code generation | Temperature 0.2–0.5 | Needs some exploration; syntax must be valid |
| Summarization | Temperature 0.3–0.7 | Faithful to source; some paraphrase variation OK |
| Creative writing, brainstorming | Temperature 0.8–1.2, top-p 0.9 | Diversity desired; quality checked by human |
| Chatbot responses | Temperature 0.7–1.0 | Natural variation without going off-the-rails |

These are starting points. Run an eval on your actual task to find what works.

### Common bugs

**Repetition loops.** A model generating near-duplicate sentences repeatedly is often
running at very low temperature (or greedy), not a bug in the model. Fix: raise
temperature, or enable repetition penalty.

**Over-creative factual answers.** A model confidently stating wrong facts with high
confidence can be worsened by high temperature. It was already uncertain; sampling
from a flatter distribution makes it land on wrong answers more. Fix: lower
temperature, add retrieval (RAG), or use a larger model.

**Non-determinism in "deterministic" calls.** Temperature=0 is greedy in most
frameworks, but floating-point arithmetic on GPUs is not perfectly reproducible
across hardware, batch sizes, or library versions. Do not rely on byte-exact
reproducibility across environments.

## Interview questions

After this page you should be able to answer:

1. What does temperature actually do to the model's output distribution? What is the mechanism?
2. A developer sets temperature=0 and says "now the model will always be correct." What is wrong with this reasoning?
3. What is the difference between top-k and top-p sampling? When does top-p behave better than top-k?
4. Your model is generating repetitive output that loops on the same phrases. What decoding settings are likely responsible, and what do you change?
5. You are building a data extraction pipeline. A teammate suggests using temperature=1.0 for "better coverage." Do you agree? Why or why not?

---

*Last verified 2026-06-12 · maintained by the Gardener*
