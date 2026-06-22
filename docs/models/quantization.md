---
last_verified: 2026-06-14
gardener_sources:
  - https://huggingface.co/docs/transformers/quantization/overview
  - https://arxiv.org/abs/2305.14314
  - https://github.com/TimDettmers/bitsandbytes
---

# Quantization

## What & why

A neural network weight is just a number. In training that number is typically stored
as a 32-bit float (fp32) — 4 bytes per weight. A 7B-parameter model in fp32 needs
28 GB of memory before a single activation is computed. At inference you need at least
two copies of the weights in GPU memory (one for compute, one for KV cache); at
training you need gradients and optimizer states on top.

Quantization represents weights (and sometimes activations) in fewer bits. The
payoffs are direct:

- **Memory**: bits-per-weight halved → model fits on half the hardware.
- **Throughput**: smaller weights → faster memory bandwidth → faster token generation.
- **Cost**: smaller memory footprint → cheaper hosting.

The cost is a small, usually acceptable drop in quality — if you do it right.

## How it actually works

### The dtype landscape

| Format | Bits | Range | Notes |
|---|---|---|---|
| fp32 | 32 | ±3.4×10³⁸ | Training default; full precision |
| fp16 | 16 | ±65504 | Half the memory; but narrow range causes overflow/underflow |
| bf16 | 16 | ±3.4×10³⁸ | Same range as fp32, but less decimal precision |
| int8 | 8 | –128 to 127 | Integer; fast on most hardware |
| NF4 | 4 | (non-uniform) | Matched to normal weight distributions |
| int4 | 4 | –8 to 7 | Uniform; coarser than NF4 |

### bf16 vs fp16 — the one that bites people

Both use 16 bits but split them differently:

- **fp16**: 1 sign + 5 exponent + 10 mantissa bits. Good decimal precision, but the
  narrow exponent means it can only represent values up to ~65,504. Gradient updates
  or activations that exceed this overflow to infinity, causing NaN and training
  collapse. Requires a GradScaler to keep values in range.
- **bf16**: 1 sign + 8 exponent + 7 mantissa bits. Same exponent range as fp32, so
  overflow is essentially never an issue — the same loss scale that works in fp32
  works in bf16. The lower mantissa precision is usually unimportant for neural
  networks because weight updates are noisy anyway.

**The practical rule**: a model that refuses to train stably in fp16 (NaN losses,
loss spikes) often trains fine in bf16. If your hardware supports bf16 (A100, A10,
H100, most modern GPUs), prefer it for training. For inference, fp16 and bf16 are
roughly equivalent in quality.

### NF4 — 4-bit storage that respects weight distributions

Uniform 4-bit quantization maps weights to evenly spaced grid points. But neural
network weights follow an approximately normal (bell-curve) distribution — most
values cluster near zero, with few large values. A uniform grid wastes resolution
where weights are dense (near zero) and under-represents the tails.

**NF4 (NormalFloat 4-bit)** solves this by designing the 16 grid points so that
equal mass of a standard normal falls between consecutive points. This means the grid
is denser near zero, where most weights actually live — maximizing precision where it
matters. NF4 is the quantization format used in QLoRA.

**Double quantization** takes the quantization constants (one per block of weights)
and quantizes *those* too, saving roughly 0.4 bits per parameter on top of the 4-bit
representation itself.

### Post-Training Quantization (PTQ) vs Quantization-Aware Training (QAT)

**PTQ** quantizes a fully-trained model after the fact — no retraining needed. Fast
to apply, but some quality loss is unavoidable because the weights weren't optimized
under the quantized representation. This is what GPTQ, AWQ, and bitsandbytes NF4
all do. Practical for inference.

**QAT** simulates quantization during training, letting the optimizer find weights
that are robust to quantization error. Higher quality results, especially at 4 bits
and below — but requires access to training compute. Used when final model quality is
critical and you can afford the training run.

### The format landscape at a glance

| Format | What it is | Best for |
|---|---|---|
| **GPTQ** | PTQ using second-order weight error minimization; stores in int4/int3 | High-quality inference on GPU |
| **AWQ** | PTQ that protects the small fraction of weights that matter most | Good quality/speed balance on GPU |
| **GGUF** | File format (not a quant algorithm) used by llama.cpp; supports multiple bit widths | CPU inference, local deployment |
| **bitsandbytes NF4** | Per-block NF4 quantization; integrated with HuggingFace | QLoRA training; fast to apply |

## Choosing your format

| Situation | Reach for | Why |
|---|---|---|
| Training with limited GPU memory | **bf16 + NF4 base (QLoRA)** | Stable training, drastically lower memory |
| Inference, max quality, GPU | **fp16 or bf16** | No quality loss |
| Inference, GPU, memory-constrained | **GPTQ or AWQ int4** | ~2× memory savings, <2% typical quality loss |
| Inference on CPU / local laptop | **GGUF** (via llama.cpp) | CPU-optimized, many bit widths |
| Embedding models / encoders | **fp16** | Usually small enough; quantization artifacts compound in embeddings |
| Fine-tuning then deploying | Train with **QLoRA (NF4)**, merge + save as **fp16** | Best of both: low training cost, clean inference |

**The honest take:** for a model that fits in fp16 on your target GPU, don't
quantize for inference — the quality floor is worth the extra memory. Quantize when
you genuinely cannot fit the model otherwise, or when throughput (not quality) is the
binding constraint.

See [Finetuning](finetuning.md) for how NF4 fits into the QLoRA training recipe.

## Interview questions

After this page you should be able to answer:

1. Why does fp16 cause training instability that bf16 avoids? What is the structural difference between the two formats?
2. Explain why NF4 outperforms uniform int4 for neural network weights. What assumption does it exploit?
3. What is the difference between PTQ and QAT? When would you invest in QAT?
4. A colleague wants to quantize an embedding model to int4 to save memory. What concerns would you raise?
5. Walk through the memory savings of QLoRA end-to-end: where does NF4 help, where does LoRA help, and what is left over?

---

*Last verified 2026-06-14 · maintained by the Gardener*
