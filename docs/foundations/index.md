# Foundations

**The concepts that everything else is built on.**

You can use LLM APIs without this section. You cannot *reason about* them — debug
unexpected outputs, evaluate trade-offs, or make good architecture decisions — without
it. Foundations is the section you return to when something confuses you downstream.

## What this section covers

| Page | Core idea |
|---|---|
| [Tokens & tokenization](tokenization.md) | How text becomes numbers — and why that matters for costs, multilingual behavior, and bugs |
| [Embeddings](embeddings.md) | What it means for meaning to live in a vector, and how similarity search works |
| [Attention & transformers](transformers.md) | The architecture that made modern LLMs possible |
| [Sampling & decoding](sampling.md) | How a model picks the next token — temperature, top-p, and when to use what |
| [Context windows](context-windows.md) | What fits, what gets lost, and how models handle long inputs |

## Suggested reading order

Go top-to-bottom in the table above. Tokenization and embeddings are prerequisites
for almost everything else on the site. Attention/transformers is the one page here
that is more "how it works" than "how I use it" — worth reading once, not
something to memorize.

## After this section you can...

- Estimate token costs for a prompt before running it
- Explain why a model behaves differently in English vs. another language
- Choose between greedy and sampled decoding for a given use case
- Debug why retrieval is returning unexpected results
- Have a confident conversation about architecture trade-offs with anyone who asks

---

*Section maintained by [Inderpuneet Singh](https://github.com/inderpun) and the Gardener agent.*
