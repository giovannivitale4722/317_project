# Understanding Robust Standard Errors

## The Problem: What Happens When Assumptions Fail?

### Standard OLS Assumptions
When we run regression, we assume:
1. **Normality**: Residuals are normally distributed
2. **Homoscedasticity**: Variance is constant
3. **No Autocorrelation**: Residuals are independent

### What Goes Wrong?
When these assumptions fail, **standard errors become wrong**, which means:
- **P-values are incorrect** (we might think something is significant when it's not, or vice versa)
- **Confidence intervals are wrong** (they're too narrow or too wide)
- **t-statistics are unreliable**

---

## What Are Standard Errors?

### Standard Error = Uncertainty in Our Estimate

Think of it like this:
- **Coefficient** (β = 1.34) = Our best guess of the relationship
- **Standard Error** (SE = 0.064) = How uncertain we are about that guess

**Small SE** = We're confident in our estimate
**Large SE** = We're uncertain

### How Standard Errors Are Calculated

**Standard OLS Formula**:
```
SE = sqrt(Variance of residuals / (n × Variance of X))
```

This formula **assumes**:
- Residuals are normally distributed
- Variance is constant
- No autocorrelation

**If these assumptions fail, this formula gives wrong answers!**

---

## What Are Robust Standard Errors?

### Definition
**Robust Standard Errors** are standard errors that are **correct even when assumptions fail**.

They're called "robust" because they're **robust to assumption violations**.

### How They Work

Instead of using the simple formula that assumes everything is perfect, robust SE use a more sophisticated formula that:

1. **Accounts for heteroscedasticity** (varying variance)
2. **Accounts for autocorrelation** (correlated errors)
3. **Works asymptotically** (gets better with larger samples, even without normality)

### The Math (Simplified)

**Standard OLS SE**:
```
SE = sqrt(σ² / (n × Var(X)))
```
(Assumes constant variance σ²)

**Robust SE (Heteroscedasticity-Consistent)**:
```
SE = sqrt(Σ(residuals² × X²) / (n × Var(X))²)
```
(Allows variance to vary across observations)

**Newey-West HAC (Heteroscedasticity and Autocorrelation Consistent)**:
```
SE = sqrt(Σ(residuals² × X²) + 2×Σ(correlations × residuals × X))
```
(Accounts for BOTH heteroscedasticity AND autocorrelation)

---

## How Do Robust SE Account for Non-Normality?

### The Key Insight: Asymptotic Theory

**Central Limit Theorem (CLT)**:
- Even if data isn't normal, **sample means become normal as sample size increases**
- With large samples (n = 2,821), the distribution of our coefficient estimate approaches normal
- Robust SE use this asymptotic property

### Why Large Samples Help

**Small Sample (n = 30)**:
- If residuals aren't normal → t-tests are unreliable
- Need normality assumption to be true

**Large Sample (n = 2,821)**:
- Even if residuals aren't normal → t-tests are still valid (asymptotically)
- CLT kicks in → coefficient estimates become approximately normal
- Robust SE work because they rely on asymptotic (large sample) properties

### What "Asymptotic" Means

**Asymptotic** = "as sample size approaches infinity"

- We don't need perfect normality
- We just need the sample to be large enough
- With n = 2,821, we're in the "large sample" territory
- Robust SE are valid because they use asymptotic theory

---

## Visual Analogy

### Standard SE (Assumes Perfect Conditions)
Imagine measuring with a ruler that assumes:
- Everything is measured the same way
- No measurement errors
- Perfect conditions

**If conditions aren't perfect → ruler gives wrong measurements**

### Robust SE (Adapts to Conditions)
Imagine a smart ruler that:
- Adjusts for different measurement conditions
- Accounts for measurement errors
- Works even when conditions aren't perfect

**Even when conditions aren't perfect → ruler still gives correct measurements**

---

## Why Robust SE Work for Non-Normality

### 1. They Don't Rely on Normality
- Standard SE formula assumes normal residuals
- Robust SE formula doesn't make this assumption
- They use a different mathematical approach

### 2. They Use Asymptotic Theory
- With large samples, distributions converge to normal (CLT)
- Robust SE leverage this convergence
- They're valid "in the limit" (as n → ∞)

### 3. They're Consistent Estimators
- **Consistent** = Gets better as sample size increases
- Even if residuals aren't normal, robust SE converge to the correct value
- With n = 2,821, we're close enough to the limit

---

## Real Example from Your Data

### Hypothesis 1 Results

**Standard OLS** (assuming normality):
- Coefficient: 1.3389
- Standard Error: 0.0578
- t-statistic: 23.15
- p-value: < 0.001

**Robust SE** (not assuming normality):
- Coefficient: 1.3389 (same!)
- Robust SE: 0.0641 (slightly larger)
- t-statistic: 20.88 (slightly smaller, but still huge)
- p-value: < 0.001 (still highly significant)

**What Changed?**
- Coefficient stayed the same (it's still our best estimate)
- Standard error got slightly larger (we're being more conservative)
- Still highly significant (conclusion unchanged)

**Why the SE Got Larger?**
- Robust SE account for the fact that residuals aren't perfectly normal
- They're more conservative (wider confidence intervals)
- This makes our results MORE reliable, not less

---

## The Bottom Line

### Why We Use Robust SE When Normality Fails

1. **Standard SE assume normality** → If normality fails, they're wrong
2. **Robust SE don't assume normality** → They work asymptotically
3. **Large samples (n = 2,821)** → Asymptotic theory applies
4. **Result**: Robust SE give correct p-values even without normality

### What "Account For" Means

When we say robust SE "account for" non-normality, we mean:

1. **They don't require normality** to be valid
2. **They use asymptotic theory** (CLT) instead
3. **With large samples**, they converge to the correct standard errors
4. **They give reliable p-values** even when residuals aren't normal

---

## Summary Table

| Aspect | Standard SE | Robust SE |
|--------|-------------|-----------|
| **Assumes normality?** | Yes | No |
| **Assumes constant variance?** | Yes | No |
| **Assumes no autocorrelation?** | Yes | No (HAC version) |
| **Works with large samples?** | Only if assumptions hold | Yes (asymptotically) |
| **Works with small samples?** | Only if assumptions hold | Less reliable |
| **Your case (n = 2,821)** | Unreliable (normality failed) | Reliable (asymptotic theory applies) |

---

## Key Takeaway

**Robust Standard Errors** are like a safety net:
- They work even when assumptions fail
- They use asymptotic theory (large sample properties)
- With n = 2,821, they're valid and reliable
- They make our conclusions trustworthy even without perfect normality

**In your case**: Normality failed, but robust SE account for this by using asymptotic theory that doesn't require normality, making your results still interpretable and reliable!

