<<<<<<< HEAD
# Hypothesis Testing Recap
=======
# Hypothesis testing
>>>>>>> main

Hypothesis testing provides a formal framework for determining whether the patterns we observe in data are likely to represent a real effect or simply random variation.

Key question: 

> "If there were actually no effect, how surprising would my data be?"

---

# The Null and Alternative Hypotheses

Every hypothesis test starts with two competing claims.

### Null Hypothesis (H₀)

The null hypothesis represents the status quo or no effect.

Examples:

- A new drug has no effect on blood pressure.
- Smokers and non-smokers have the same average lung capacity.
- A marketing campaign does not change sales.

### Alternative Hypothesis (H₁)

The alternative hypothesis represents the effect we are trying to detect.

Examples:

- The new drug changes blood pressure.
- Smokers and non-smokers have different average lung capacity.
- The campaign changes sales.

---

# An Example

Suppose I want to determine whether a new treatment lowers blood glucose levels.

The hypotheses might be:

$$
H_0: \mu = 100
$$

$$
H_1: \mu < 100
$$

where \( \mu \) is the average blood glucose level after treatment.

I collect data from a sample of patients and calculates a test statistic.

---

# The p-value

The p-value is often the most misunderstood concept in statistics.

A p-value is:

> The probability of observing data at least as extreme as that collected, assuming the null hypothesis is true.

For example:

```text
p = 0.02
```

means:

> If there truly were no treatment effect, we'd only expect to see results this extreme about 2% of the time.

A small p-value suggests that the observed data would be unusual under the null hypothesis.

---

# Statistical Significance

A significance level, denoted by α, is chosen before conducting the test.

A common choice is:

$$
\alpha = 0.05
$$

Decision rule:

- If p < 0.05, reject H₀.
- If p ≥ 0.05, fail to reject H₀.

Notice the wording:

> We do not "accept" the null hypothesis.

Failing to find evidence for an effect is not the same as proving that no effect exists.

---

# Type I and Type II Errors

Every hypothesis test carries the risk of making a mistake.

### Type I Error

Rejecting a true null hypothesis.

A false positive.

Example:

> Concluding that a treatment works when it actually does not.

The probability of a Type I error is controlled by α.

### Type II Error

Failing to reject a false null hypothesis.

A false negative.

Example:

> Concluding that a treatment does not work when it actually does.

The probability of detecting a real effect is known as **statistical power**.

---
