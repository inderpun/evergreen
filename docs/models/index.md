# Models

**What the models are, how they learn, and how to make them do what you need.**

This section moves one level below API calls: what distinguishes models from each
other, how finetuning actually changes a model's behavior, and what quantization
trades away. You don't need this to prototype. You need it to decide whether to
finetune, which model to pick, and why your finetuned model regressed on tasks you
didn't change.

## What this section covers

| Page | Core idea |
|---|---|
| [The model landscape](landscape.md) | How to read the model ecosystem — capability tiers, open vs. closed, when size stops mattering |
| [Finetuning (LoRA/QLoRA/PEFT)](finetuning.md) | When finetuning is the right lever, how parameter-efficient methods work, and common failure modes |
| [Quantization](quantization.md) | How models shrink, what gets lost, and how to choose a quantization level |
| [Reasoning & test-time compute](test-time-compute.md) | Why "thinking longer" improves answers, and what it costs |

## Suggested reading order

Landscape first — it gives you the vocabulary for the rest. Then finetuning and
quantization can be read in either order; test-time compute builds on both.

## After this section you can...

- Articulate the real trade-off between open-weight and closed models for a given use
  case
- Decide when finetuning is likely to help and when prompt engineering is the better
  first bet
- Understand what a quantized model is giving up and whether it matters for your task
- Explain why a reasoning-mode call costs more and takes longer, and when that trade
  is worth making

---

*Section maintained by [Inderpuneet Singh](https://github.com/inderpun) and the Gardener agent.*
