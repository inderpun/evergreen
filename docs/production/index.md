# Production

**Shipping LLM systems that actually run in the real world.**

Getting a demo to work is not the hard part. Production means: latency you can
predict, costs you can forecast, failures you can debug, and outputs you can trust
enough to put your name on. This section covers the engineering concerns that only
appear when real users hit real systems.

## What this section covers

| Page | Core idea |
|---|---|
| [Observability & tracing](observability.md) | How to instrument LLM calls so you can debug failures and track regressions |
| [Cost engineering](cost.md) | The levers that actually move your inference bill, and how to reason about them |
| [Safety & guardrails](safety.md) | Input/output controls, prompt injection, and the honest state of the art |

## Suggested reading order

Observability first — you need it before you can understand anything else that's
happening. Cost and Safety can be read in either order; most teams need both before
calling a system production-ready.

## After this section you can...

- Instrument an LLM pipeline with traces you can query when something breaks
- Estimate and bound the inference cost of a system before you ship it
- Name the primary attack surfaces in an LLM pipeline and what defends each
- Describe the difference between hard guardrails (deterministic) and soft guardrails
  (model-based) and when each is appropriate
- Make a checklist that a new LLM feature should pass before going to production

---

*Section maintained by [Inderpuneet Singh](https://github.com/inderpun) and the Gardener agent.*
