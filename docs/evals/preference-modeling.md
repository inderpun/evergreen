---
last_verified: 2026-06-14
gardener_sources:
  - https://arxiv.org/abs/2203.02155
  - https://arxiv.org/abs/2305.18290
  - https://github.com/huggingface/trl
---

# Preference Modeling & Reward Models

## What & why

When you align a language model with human values, the first problem is: how do you
turn a human preference into a training signal? Humans are good at comparing two
responses — "A is better than B" — but they are inconsistent and slow at assigning
absolute numeric scores. The field's answer is to collect **pairwise comparisons** and
train a **reward model** that turns any single response into a scalar score,
approximating what a human rater would have said.

That scalar is then used to fine-tune the language model with reinforcement learning
(RLHF) or with direct alignment algorithms (DPO, ORPO, etc.). The reward model is
the load-bearing artifact that converts human preferences into a differentiable
training objective.

## How it actually works

### From comparisons to scores: Bradley-Terry and Elo

Pairwise comparisons don't directly give you a score for each response. The
**Bradley-Terry model** provides the principled connection: if response A beats B with
probability `p`, then:

```
p(A > B) = exp(score_A) / (exp(score_A) + exp(score_B))
```

This is equivalent to a logistic regression over the difference in latent qualities.
Training a reward model with binary cross-entropy on pairwise preference labels is
literally fitting a Bradley-Terry model where the "latent quality" is the model's
scalar output.

**Elo ratings** use the same pairwise math but update ratings online (one match at a
time) rather than via a global fit. Elo is used in head-to-head arena evaluations
(e.g., "which LLM did the user prefer?") rather than for training reward models
directly, but the math is the same family.

### Training a reward model

The standard recipe:

1. Start from a pretrained (and usually instruction-tuned) base model.
2. Replace the language model head with a **scalar regression head** — one output per
   input sequence.
3. Feed a pair of responses `(y_w, y_l)` for the same prompt, where `y_w` is the
   preferred one. Compute `r_w = reward(prompt, y_w)` and `r_l = reward(prompt, y_l)`.
4. Train with the **pairwise ranking loss**:
   `L = -log(sigmoid(r_w - r_l))`
   This pushes the preferred response to score higher than the rejected one.

The resulting model outputs a scalar for any (prompt, response) pair — usable both as
a training signal for RLHF and as an automated evaluator.

### Role in RLHF and RLAIF

**RLHF** (Reinforcement Learning from Human Feedback) uses the reward model as a
frozen proxy for human preference during RL fine-tuning. The LLM generates responses;
the reward model scores them; a policy gradient algorithm (usually PPO) updates the
LLM to maximize that score. A KL-divergence penalty against the original model
prevents the LLM from drifting so far that it produces high-scoring gibberish.

**RLAIF** (RL from AI Feedback) replaces human raters with another LLM (the
"constitution" or judge model). The pipeline is otherwise the same. This scales
annotation cheaply but inherits whatever biases the judge LLM carries — see
[LLM-as-judge](llm-as-judge.md).

**DPO** (Direct Preference Optimization) sidesteps the separate reward model
entirely, reparameterizing the RLHF objective so the LLM itself is the implicit
reward model. Simpler to train, no separate RM needed — but less flexible for
iterative improvement.

### What makes a good reward model

- **Distribution match** — the reward model should be trained on preferences from
  raters who resemble your target users, on the tasks your deployment will face.
  A reward model trained on general web chat gives a poor signal for medical summarization.
- **Coverage** — preference data is expensive. A reward model that's seen few examples
  of a failure mode will not penalize it reliably.
- **Calibration** — a reward model that says 8.5 vs 8.6 when the honest answer is
  "roughly equivalent" is dangerous for RL training. Good calibration matters; see
  [Calibration](calibration.md).
- **Agreement with humans** — the standard check is correlation between RM scores and
  held-out human judgments. But correlation does not guarantee the RM can't be gamed.

### Reward hacking

When the LLM is trained against the reward model, it will eventually find behaviors
that score high on the RM without being genuinely good — because the RM is an
imperfect proxy. This is **reward hacking** (also called Goodhart's Law: "when a
measure becomes a target, it ceases to be a good measure"). Symptoms include
responses that are unusually long (verbosity reward), unusually sycophantic, or
confident-sounding without substance.

Mitigations: diverse human re-evaluation after RL, KL penalties, ensemble reward
models, refreshing preference data frequently.

## Interview questions

After this page you should be able to answer:

1. Why do we collect pairwise preferences rather than direct numerical ratings? What property of human judgment does pairwise comparison exploit?
2. Write out the Bradley-Terry model. How does training with pairwise ranking loss relate to it?
3. What is the difference between RLHF and RLAIF? What risk does RLAIF introduce?
4. Your reward model scores response A higher than B, but human raters consistently prefer B. Name three things you would investigate.
5. What is reward hacking? Give one concrete behavioral symptom and one mitigation.

---

*Last verified 2026-06-14 · maintained by the Gardener*
