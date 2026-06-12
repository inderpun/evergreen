# Evergreen

**The GenAI handbook that doesn't go stale.**

The problem with learning generative AI in 2026 isn't scarcity — it's the firehose.
Anthropic, Google, OpenAI, Meta, and AWS ship daily; every explainer you read was
written for a landscape that no longer exists. Docs rot. Tutorials reference
deprecated APIs. "Best practice" posts age out in a quarter.

Evergreen attacks the staleness directly:

- **Concept pages** distill one idea each — what it is, why it exists, how the field
  uses it, and what to say about it in an interview. Every page carries a
  `last_verified` date you can trust.
- **The Stream** is a weekly digest of what actually shipped and why it matters,
  written to be read in five minutes.
- **The Gardener** — an autonomous agent — monitors provider changelogs, library
  releases, and the research feed. When something material changes, it opens a pull
  request against the affected page with the diff and its sources. The
  [PR history](https://github.com/inderpun/evergreen/pulls) *is* the proof this works.

!!! note "Reading order"
    New to GenAI? Go top-to-bottom: **Foundations → Models → RAG → Agents → Evals →
    Production.** Each section's index explains what you'll be able to do after
    reading it. Already building? Jump straight to the concept you need — pages are
    self-contained.

## Page anatomy

Every concept page follows the same contract:

| Section | Promise |
|---|---|
| **What & why** | the idea in plain language, and the problem it exists to solve |
| **How it actually works** | enough mechanism to reason about trade-offs, no fluff |
| **Choosing / using** | the practical decision table for builders |
| **Interview questions** | what you should be able to answer after reading |
| **Last verified** | date + sources the Gardener checked |

---

*Maintained by [Inderpuneet Singh](https://github.com/inderpun) and the Gardener agent.*
