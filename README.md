# 🌲 Evergreen

**The living GenAI handbook — docs that don't go stale.**

📖 **Read it: [inderpun.github.io/evergreen](https://inderpun.github.io/evergreen/)**

## The problem

Learning generative AI in 2026 isn't limited by scarcity — it's the firehose. Providers
ship daily; every explainer was written for a landscape that no longer exists. Static
docs rot quietly: deprecated APIs, stale "best practices," numbers that were true last
quarter.

## The approach

1. **Concept pages** — one idea each, distilled: what it is, why it exists, how the
   field uses it, what to be able to answer about it in an interview. Every page
   carries a `last_verified` date in its frontmatter.
2. **The Stream** — weekly digests of what actually shipped and why it matters,
   readable in five minutes.
3. **The Gardener** *(in development)* — an autonomous agent that monitors provider
   changelogs, library releases, and the research feed. When something material
   changes, it opens a pull request against the affected page with the diff and its
   sources. **The PR history of this repo is the live demo.**

## Structure

```
docs/
├── stream/        # weekly digests
├── foundations/   # tokens, embeddings, transformers, sampling, context
├── models/        # landscape, finetuning, quantization, test-time compute
├── rag/           # vector DBs, chunking, retrieval, evaluation
├── agents/        # patterns, MCP, multi-agent, memory
├── evals/         # benchmarks, LLM-as-judge, preference models, calibration
└── production/    # observability, cost, safety
```

Pages marked 🌱 are seedlings — structure committed, content queued.

## Local development

```bash
python3 -m venv .venv && .venv/bin/pip install mkdocs-material
.venv/bin/mkdocs serve   # http://127.0.0.1:8000, hot-reload
```

Deploys automatically to GitHub Pages on push to `main` (`.github/workflows/deploy.yml`).

## License

Content: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — share and adapt
with attribution.
