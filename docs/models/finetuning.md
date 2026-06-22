---
last_verified: 2026-06-14
gardener_sources:
  - https://huggingface.co/docs/peft/index
  - https://arxiv.org/abs/2106.09685
  - https://arxiv.org/abs/2305.14314
---

# Finetuning (LoRA / QLoRA / PEFT)

## What & why

A pretrained language model encodes broad world knowledge but knows nothing about your
task, your domain's vocabulary, or the tone your users expect. Finetuning shifts the
model's weights toward your target distribution. The question is *how much* of the
model you actually need to move.

**Full finetuning** updates every parameter. For a 7–70B model that means storing
weights, gradients, and optimizer states (Adam needs 2× the weight count) — easily
tens to hundreds of GB of GPU memory. It is the right tool only when you have abundant
data, abundant compute, and strong evidence that the pretrained representation is
genuinely inadequate for your task.

**PEFT** (Parameter-Efficient Fine-Tuning) is the umbrella term for approaches that
update a small fraction of parameters while keeping most weights frozen. The most
widely adopted PEFT method is **LoRA**.

## How it actually works

### LoRA — Low-Rank Adaptation

The core insight: task adaptation does not require changing every element of a large
weight matrix. It can be approximated by a low-rank update. LoRA freezes the original
weight matrix `W` and learns a pair of small matrices whose product is added at
forward-pass time:

```
W' = W + (α / r) · B · A
```

where `A` has shape `(r, d_in)` and `B` has shape `(d_out, r)`, with `r` much
smaller than `d`. The scaling factor `α / r` keeps the update magnitude stable as
you vary `r`.

What this buys you:

- **Trainable parameter count collapses** — for a 9B model, LoRA adapters on
  attention projections might total ~20–50M parameters, versus 9B for full finetuning.
  Gradient and optimizer memory shrink proportionally.
- **The base model is untouched** — adapters are small files (~50–200 MB) that can be
  swapped per task, per user, or per deployment, while sharing one base model in memory.
- **No inference overhead at merge time** — once trained, `B·A` can be added directly
  into `W`, yielding a standard model with zero extra latency.

**Key hyperparameters:**

| Knob | What it controls | Typical range |
|---|---|---|
| `r` (rank) | Adapter capacity | 8–64; start at 16 |
| `α` (alpha) | Effective learning rate scaling | Often set to `2r` or equal to `r` |
| Target modules | Which layers get adapters | Attention projections are standard; MLP layers add capacity |
| Dropout | Regularization inside adapters | 0.05–0.1 for small datasets |

Higher rank gives more expressive adapters at the cost of more parameters and higher
overfitting risk. For most classification and instruction-following tasks, `r=16–32`
is the reliable starting point.

### QLoRA — Quantized LoRA

LoRA solves the *gradient and optimizer* memory problem. The frozen base weights still
occupy full precision — ~18 GB for a 9B model in fp16. **QLoRA** solves this by
storing the frozen weights in **4-bit NF4** (NormalFloat 4): a quantization grid
specifically matched to the approximately normal distribution of neural network weights.
During the forward pass, each block dequantizes on-the-fly to bf16 for the matrix
multiply; gradients flow through the dequantized values into the full-precision LoRA
adapters, which are never quantized.

A typical memory comparison for a ~9B model:

| Component | Full finetune (fp16) | QLoRA |
|---|---|---|
| Weights | ~18 GB | ~5 GB (NF4 + double quant) |
| Gradients | ~18 GB | ~0.1 GB (LoRA layers only) |
| Optimizer states | ~36 GB | ~0.2 GB |
| Activations | ~5+ GB | ~2–4 GB (with gradient checkpointing) |
| **Total** | **~75+ GB** | **~8–12 GB** |

**Double quantization** takes this further: it quantizes the quantization constants
themselves, saving another ~0.4 bits per parameter. **Paged optimizers** spill
optimizer states to CPU RAM during memory spikes, acting as a safety valve.

**Gradient checkpointing** (recompute activations during the backward pass rather than
storing them) trades ~20–30% training speed for large activation memory savings —
almost always worth it at this scale.

See [Quantization](quantization.md) for a deeper treatment of NF4, bf16, and the
dtype landscape.

## Finetuning vs. the alternatives

Before reaching for finetuning, it is worth asking whether the behavior change you
want is even a job for weight updates.

| Goal | Likely better path |
|---|---|
| Answer questions about a static knowledge base | [RAG](../rag/vector-databases.md) — no training, easy to update |
| Change output format, tone, or task framing | Few-shot prompting or system prompt first |
| Teach genuinely new skills or domain-specific reasoning | Finetuning on curated data |
| Inject recent facts the model doesn't know | RAG or a hybrid |
| Reduce output verbosity / style mismatch at scale | PEFT is cheapest; DPO if you have preference data |
| Hard-coded behavior the model resists (e.g., safety override) | This is exactly what RLHF / preference finetuning is for |

**The honest take:** instruction-tuned models with good prompts handle a surprising
share of tasks. Finetuning earns its cost when you have (a) hundreds to thousands of
high-quality labeled examples, (b) a measurable quality gap from prompting, and
(c) a stable enough task that the training data won't immediately go stale.

## Interview questions

After this page you should be able to answer:

1. Explain the LoRA update `W' = W + (α/r)·BA`. Why is the product `B·A` low-rank, and what does that assume about task adaptation?
2. A colleague proposes full finetuning a 70B model for a classification task with 5,000 labeled examples. What are the practical and statistical concerns?
3. What does QLoRA add over plain LoRA? Walk through where each reduces memory.
4. When would you choose RAG over finetuning, even if you have labeled training data?
5. You train a LoRA adapter and notice it overfits quickly. What three knobs do you reach for first?

---

*Last verified 2026-06-14 · maintained by the Gardener*
