---
last_verified: 2026-06-14
gardener_sources:
  - https://arxiv.org/abs/1706.04599
  - https://arxiv.org/abs/1706.07806
  - https://scikit-learn.org/stable/modules/calibration.html
---

# Calibration

## What & why

A model is **calibrated** if its stated confidence matches how often it is actually
correct. A well-calibrated model that says "I'm 80% confident" should be right about
80% of the time across all such statements. A model that says "90% confident" and is
right 60% of the time is **overconfident** — and dangerously so, because downstream
users and systems trust the number.

Calibration is distinct from accuracy. A model can be highly accurate on average but
poorly calibrated (its correct answers come with 99% confidence and its wrong answers
come with 97% — the confidence doesn't discriminate). Conversely, a modest-accuracy
model can be well-calibrated if its confidence tracks its actual error rate.

This matters because log loss (the most common metric for classification with
probabilities) directly punishes calibration: being confidently wrong is penalized
far more than being unsure and wrong. A calibrated model that says 55/25/20 where the
correct class is in the minority is far safer than one that says 90/5/5 and turns out
to be wrong.

## How it actually works

### Log loss and the calibration penalty

**Log loss** (cross-entropy) for a single example is `-log(p_correct)`, where
`p_correct` is the probability the model assigns to the true class. The key property:

- If you're wrong, being *confidently* wrong costs much more than being unsure.
  `–log(0.05) ≈ 3.0` vs `–log(0.45) ≈ 0.8`.
- Being right with low confidence is still penalized, but mildly.
  `–log(0.55) ≈ 0.6` vs `–log(0.95) ≈ 0.05`.

A model that hedges — spreading probability mass rather than being highly peaked —
will often have *better* log loss than a more accurate but overconfident one. This is
why calibration is the metric that matters for any downstream use of probabilities
(risk systems, decision support, medical diagnostics, ensemble inputs).

### Reliability diagrams and ECE

A **reliability diagram** visualizes calibration: bin predictions by confidence level
(e.g., 0–10%, 10–20%, …), then plot the actual accuracy within each bin. A perfectly
calibrated model's points all lie on the diagonal. Points above the diagonal mean the
model is underconfident (it says 40% but is right 70% of the time). Points below mean
overconfidence.

**Expected Calibration Error (ECE)** converts this diagram into a scalar: the
weighted average gap between confidence and accuracy across bins, where the weight is
the fraction of examples in each bin. Lower ECE is better; ECE = 0 is perfect
calibration. ECE has known flaws (bin-width sensitivity, ignores within-bin variance),
but it is the most widely reported single-number calibration metric.

### Temperature scaling

**Temperature scaling** is a post-hoc calibration technique applied *after* training.
The model's logits (pre-softmax scores) are divided by a temperature scalar `T`
before the softmax:

```
p(class) = softmax(logits / T)
```

- `T > 1` softens the distribution — the model becomes less confident. Fixes
  overconfidence (the common case for modern large networks and finetuned models).
- `T < 1` sharpens the distribution — rarely needed, but addresses underconfidence.

`T` is fit by minimizing log loss on a held-out calibration set, leaving the model
weights unchanged. Temperature scaling uses a single parameter, is virtually never
overfit, runs in seconds, and typically closes 60–80% of the calibration gap with no
accuracy penalty. It is almost always worth doing when you care about probabilities.

More expressive variants exist (vector scaling per class, Platt scaling, isotonic
regression), but they require more calibration data and are more prone to overfitting.
For most classification applications, temperature scaling is the right first tool.

### Neural networks are overconfident by default

Modern neural networks trained with softmax cross-entropy tend to be overconfident —
especially large, heavily regularized models. This is partly structural: the softmax
output can be driven arbitrarily close to 1 by scaling up logits, and gradient
descent will do exactly this if it reduces training loss. Label smoothing during
training (replacing hard 0/1 targets with, say, 0.05/0.95) prevents the model from
driving logits to extreme values, improving calibration from the start.

## When calibration matters, and when it doesn't

| Situation | Calibration importance |
|---|---|
| Probability used as a risk/decision input | Critical — downstream system trusts the number |
| Output fed as a feature into an ensemble | High — poorly calibrated inputs hurt ensemble |
| Log loss is the evaluation metric | High by definition |
| Ranking outputs by confidence | Low — rank order is preserved under monotone transforms |
| Returning a hard label, no probabilities surfaced | Low — accuracy is the metric |
| Medical, legal, financial risk scoring | Critical — overconfidence has direct harm potential |

The short version: if a downstream system or user is going to *act on* the probability
as a probability, calibrate. If you only care about which class is highest, calibration
is secondary.

See [LLM-as-judge](llm-as-judge.md) for how calibration interacts with LLM judge
reliability, and [Preference modeling](preference-modeling.md) for why calibration
matters when using reward models as training signals.

## Interview questions

After this page you should be able to answer:

1. Distinguish accuracy from calibration. Give an example where a model can be highly accurate but poorly calibrated.
2. Why does log loss punish confident wrong answers so much more than uncertain wrong answers? Show with a calculation.
3. Describe what a reliability diagram shows. What does it mean when points fall below the diagonal?
4. How does temperature scaling work? Why does it almost never hurt accuracy?
5. A classification model is being used to rank items by predicted probability. A colleague proposes temperature scaling to improve deployment. Do you agree it's necessary? Why or why not?

---

*Last verified 2026-06-14 · maintained by the Gardener*
