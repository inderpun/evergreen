# Evals

**How to know if your system is actually working.**

Evaluation is the discipline that separates engineering from guessing. Without evals,
you can't tell whether a prompt change helped, whether a new model is better for your
use case, or whether a feature shipped to production made things worse. This section
covers how to measure LLM system quality — and why the standard tools are harder to
trust than they look.

## What this section covers

| Page | Core idea |
|---|---|
| [Benchmarks & their limits](benchmarks.md) | What public benchmarks measure, what they don't, and how to read them critically |
| [LLM-as-judge](llm-as-judge.md) | Using an LLM to evaluate LLM outputs — the technique, the biases, and when it's appropriate |
| [Preference modeling & reward models](preference-modeling.md) | How RLHF works, what reward models learn, and what they get wrong |
| [Calibration](calibration.md) | Whether a model's confidence reflects its actual accuracy, and why it matters |

## Suggested reading order

Benchmarks first for context, then LLM-as-judge (the most immediately practical for
builders), then preference modeling and calibration in either order.

## After this section you can...

- Critically read a model's benchmark results instead of taking them at face value
- Set up an LLM-as-judge pipeline with documented bias mitigations
- Explain how RLHF works in plain language and name two ways reward models fail
- Define calibration and explain why an overconfident model is a production risk
- Build a minimal eval harness that gives you a number you can track over model
  versions or prompt iterations

---

*Section maintained by [Inderpuneet Singh](https://github.com/inderpun) and the Gardener agent.*
