---
last_verified: 2026-06-14
gardener_sources:
  - https://arxiv.org/abs/2306.05685
  - https://arxiv.org/abs/2404.13076
  - https://github.com/lm-sys/FastChat/tree/main/fastchat/llm_judge
---

# LLM-as-Judge

## What & why

Evaluating language model outputs at scale is hard. Human annotation is expensive,
slow, and noisy. Automated metrics (BLEU, ROUGE) correlate poorly with quality on
open-ended tasks. **LLM-as-judge** uses a capable language model to score, compare,
or critique other models' outputs — acting as an approximation of a human rater.

The approach has become the backbone of automated eval pipelines: picking which
response a user would prefer, rating helpfulness, assessing factual accuracy,
flagging unsafe content. When judge and human judgments align well, it scales to
millions of comparisons. When they don't, it introduces systematic distortions that
are worse than random noise because they are *consistent* — meaning wrong evaluations
reinforce each other.

Understanding where LLM judges work and where they break is the critical skill.

## How it actually works

### Three judging modes

**Pairwise comparison** — give the judge two responses and ask which is better. Maps
directly to preference data for reward model training (see [Preference
modeling](preference-modeling.md)). Easier for judges than absolute scoring; also
directly comparable to human pairwise judgments.

**Single-response scoring** — give the judge one response and ask for a score (e.g.,
1–10 on helpfulness). Faster (no pairing needed), but absolute scales are hard to
calibrate — a 7 from one judge may not equal a 7 from another. Useful for monitoring
and filtering, not for fine-grained ranking.

**Critique and reference** — give the judge a response *and* a reference answer, ask
it to assess correctness or completeness. Reference-based judging is generally more
reliable because the judge has a ground-truth anchor, not just its internal priors.

### The known biases

LLM judges are not neutral. The systematic biases are well-documented:

**Position bias** — the judge tends to prefer whichever response appears first (in
pairwise comparison) or, depending on the judge, the second. This is reproducible and
non-trivial in magnitude — enough to flip verdicts.

**Verbosity bias** — longer responses score higher, even when the extra content is
padding or repetition. This mirrors a known human bias but LLM judges can exhibit it
even more strongly. The effect is large enough that response length is a reasonable
predictor of judge score independent of actual quality.

**Self-preference** — a model asked to judge outputs will often rate outputs from a
model of the same family higher than outputs from competing families. This makes a
model a poor judge of its own outputs.

**Sycophancy / framing bias** — judges can be swayed by confident phrasing,
authoritative tone, or responses that mirror the style of their own training data.

**Coverage over correctness** — judges often reward responses that cover more topics,
even if coverage comes at the cost of accuracy.

### Mitigations

**Position swap** — run the pairwise comparison twice with A/B and then B/A, average
the results. Position bias cancels by construction: if the judge prefers "first" 60%
of the time, each response gets to be "first" once. This is cheap and nearly always
worth doing.

**Calibrated rubrics** — instead of asking "which is better?", give the judge a
detailed scoring rubric with examples of what a 3/5 vs 4/5 answer looks like. Rubrics
reduce variance and make the judge's reasoning auditable.

**Reference answers** — anchor the judge with a known-good response. The judge
assesses "is this response correct relative to the reference?" rather than "does this
response seem good?". Reliably improves agreement with human judgments on factual
tasks.

**Diverse judge ensemble** — use multiple judge models (or multiple judge prompts)
and aggregate. Self-preference and model-family biases partially cancel across judges
from different providers.

**Swap names / anonymize** — in pairwise comparison, assign neutral labels ("Response
A") rather than model names. Prevents self-preference from activating explicitly.

## When judge correlation with humans holds and when it doesn't

| Situation | Judge reliability |
|---|---|
| Open-ended instruction following (helpfulness, clarity) | Good — human raters often also rely on general impression |
| Creative or stylistic tasks with clear rubric | Moderate — rubric helps; "voice" is hard to judge |
| Factual correctness (science, medicine, law) | Poor without reference — judge "knows" wrong things too |
| Detecting subtle safety violations | Unreliable — judges miss dog-whistles and context-dependent harms |
| Long document summarization | Moderate — position/length bias amplified on long inputs |
| Code correctness | Use a real test suite instead — execution is a perfect judge |

The common thread: LLM judges work well when the quality signal is diffuse (overall
quality impression), rubric-based (specific criteria), or style-based — and break down
when factual ground truth or edge-case safety knowledge is required.

## Interview questions

After this page you should be able to answer:

1. Name three systematic biases LLM judges exhibit. For each, describe a concrete mitigation.
2. A team reports their LLM judge achieves 85% agreement with human raters on a helpfulness benchmark. Should you trust it for evaluating factual accuracy? Why or why not?
3. Explain why position-swap eliminates position bias *by construction*, not just approximately.
4. When is "use a different judge model" a better solution than "add a rubric"? When is it worse?
5. A product team proposes using the same model to generate responses and judge them. What problem does this introduce, and how would you address it?

---

*Last verified 2026-06-14 · maintained by the Gardener*
