# Complete Project Walkthrough: Tesla Stock Analysis

## Overview
This project tests two hypotheses about Tesla's stock returns:
1. **Hypothesis 1**: Does Tesla's daily return depend on S&P 500's daily return?
2. **Hypothesis 2**: Do trading volume and market volatility (VIX) significantly contribute to explaining Tesla's returns beyond market movement?

---

## PART 1: DATA CLEANING AND PREPARATION

### Step 1: Loading the Data
**What we did**: Loaded three CSV files:
- `TSLA.csv` (2,956 rows) - Tesla stock data
- `SPY.csv` (8,037 rows) - S&P 500 index data  
- `vix_daily.csv` (8,687 rows) - VIX volatility index data

**Why**: We need all three datasets to test our hypotheses.

**Finding**: Different date formats and column names across datasets.

---

### Step 2: Date Standardization
**What we did**: 
- Converted all dates to `YYYY-MM-DD` format
- Handled timezone issues in SPY data (removed `00:00:00-05:00`)
- Fixed lowercase column names in VIX data (`date` vs `Date`)

**Why**: 
- Dates must match exactly to merge datasets
- SPY had timezone-aware timestamps that would prevent proper merging
- VIX used different column naming conventions

**Method**: 
- Used `pd.to_datetime()` with `utc=True` for SPY (to handle timezone)
- Used `.dt.tz_localize(None)` to remove timezone
- Used `.dt.normalize()` to get date-only format

**Result**: All dates standardized to `YYYY-MM-DD 00:00:00` format.

---

### Step 3: Merging the Data
**What we did**: 
- Selected key columns: `Adj Close` and `Volume` from TSLA, `Close` from SPY, `close` from VIX
- Used **inner join** to merge by date
- Only kept dates where ALL three datasets had data

**Why**: 
- Inner join ensures we only analyze days when we have complete information
- Removes market holidays and non-trading days automatically
- Aligns all data to the same time period

**Result**: 2,956 aligned trading days (2010-06-29 to 2022-03-24)

---

### Step 4: Feature Engineering

#### 4a. Calculating Log Returns
**What we did**: 
- Calculated log returns for TSLA and SPY: `log(Price_today / Price_yesterday)`
- Calculated simple change for VIX: `VIX_today - VIX_yesterday`

**Why log returns?**:
1. **Time-additivity**: Log returns can be added over time (if you have 5% Monday and 3% Tuesday, total is 8% with log returns)
2. **Normalization**: Makes returns comparable across different price levels
3. **Statistical properties**: More symmetric, closer to normal distribution
4. **Better for regression**: Satisfies regression assumptions better than simple returns

**Formula**: 
- `TSLA_Return = ln(TSLA_Adj_Close_t / TSLA_Adj_Close_{t-1})`
- `SPY_Return = ln(SPY_Close_t / SPY_Close_{t-1})`
- `VIX_Change = VIX_Close_t - VIX_Close_{t-1}`

**Result**: Created `TSLA_Return`, `SPY_Return`, and `VIX_Change` variables.

---

#### 4b. Volume Transformation
**What we did**: 
- Checked skewness of volume data
- Found skewness = 2.32 (highly skewed)
- Applied log transformation: `log(Volume)`
- Calculated change: `diff(log(Volume))`

**Why**: 
- Volume data is typically highly skewed (few very high volume days)
- Log transform makes it more normally distributed
- Better for regression analysis

**Test**: Shapiro-Wilk test (we'll see this later for residuals)
**Result**: Created `TSLA_Volume_Log` and `TSLA_Volume_Change`.

---

### Step 5: Outlier Removal
**What we did**: 
- Calculated z-scores for all regression variables
- Removed rows where ANY variable had |z-score| > 3

**Why**: 
- Extreme outliers can distort regression results
- Z-score > 3 means the value is more than 3 standard deviations from the mean
- These are rare events (should occur < 0.3% of the time if data is normal)

**Method**: 
- Used `scipy.stats.zscore()` to calculate z-scores
- Filtered: `(abs(z_scores) < 3).all(axis=1)`

**Result**: 
- Removed 134 outliers (4.5% of data)
- Final dataset: **2,821 trading days**

---

## PART 2: REGRESSION ANALYSIS

### Hypothesis 1: Tesla vs S&P 500

**Model**: `TSLA_Return = β₀ + β₁(SPY_Return) + ε`

**What this tests**: Is there a linear relationship between Tesla's returns and the market's returns?

**Results** (with robust standard errors):
- **Coefficient (β₁)**: 1.3389
- **Robust SE**: 0.064122
- **t-statistic**: 20.88
- **p-value**: < 0.001
- **R²**: 0.1598 (15.98% of variance explained)

**Interpretation**: 
- For every 1% change in SPY return, Tesla return changes by approximately 1.34%
- Tesla is **more volatile** than the market (β > 1)
- The relationship is **highly significant** (p < 0.001)

**Conclusion**: ✓ Tesla's returns ARE significantly correlated with SPY returns.

---

### Hypothesis 2: Volume and VIX Beyond Market

**Model**: `TSLA_Return = β₀ + β₁(SPY_Return) + β₂(VIX_Change) + β₃(TSLA_Volume_Change) + ε`

**What this tests**: Do volume and VIX add explanatory power beyond just market movement?

**Results** (with robust standard errors):

| Variable | Coefficient | Robust SE | t-stat | p-value | Significant? |
|----------|-------------|-----------|--------|---------|--------------|
| SPY_Return | 1.2024 | 0.1151 | 10.45 | < 0.001 | ✓ Yes |
| VIX_Change | -0.0011 | 0.0007 | -1.62 | 0.106 | ✗ No |
| TSLA_Volume_Change | 0.0048 | 0.0017 | 2.81 | 0.005 | ✓ Yes |

**R²**: 0.1647 (16.47% of variance explained)

**Interpretation**:
- **SPY_Return**: Still highly significant (market matters)
- **VIX_Change**: NOT significant (volatility doesn't add explanatory power)
- **TSLA_Volume_Change**: Significant (trading volume DOES matter beyond market)

**Conclusion**: ✓ **Volume** significantly contributes to explaining Tesla's returns beyond market movement, but **VIX does not**.

---

## PART 3: ASSUMPTION CHECKS

### Why Check Assumptions?
Regression analysis requires certain assumptions to be valid. If assumptions fail, our p-values and confidence intervals may be wrong, making conclusions unreliable.

---

### Assumption 1: Linearity
**What it tests**: Is the relationship between variables actually linear?

**How we tested**:
- Visual inspection: Scatter plots of TSLA_Return vs SPY_Return
- Residual plots: Residuals vs fitted values

**What we look for**:
- Points should follow a straight line pattern
- Residuals should be randomly scattered around zero (no patterns)

**Result**: ✓ **PASSED** - Linear relationship appears reasonable

---

### Assumption 2: Normality of Residuals
**What it tests**: Are the residuals (errors) normally distributed?

**Why it matters**: 
- Needed for valid t-tests and p-values
- Ensures confidence intervals are correct

**How we tested**:
1. **Q-Q Plot**: Compares residual distribution to normal distribution
   - Points should fall along a straight line if normal
2. **Shapiro-Wilk Test**: Statistical test for normality
   - H₀: Residuals are normally distributed
   - p < 0.05 means we reject normality

**Results**:
- **Hypothesis 1**: Shapiro-Wilk p < 0.001 → ✗ **FAILED**
- **Hypothesis 2**: Shapiro-Wilk p < 0.001 → ✗ **FAILED**

**What this means**: Residuals are NOT perfectly normal, but with large samples (n = 2,821), this is less critical. We use robust standard errors to account for this.

---

### Assumption 3: Homoscedasticity (Constant Variance)
**What it tests**: Do residuals have constant variance across all values of X?

**Why it matters**: 
- If variance changes, standard errors are wrong
- P-values become unreliable

**How we tested**:
1. **Visual**: Residuals vs fitted values plot
   - Should show random scatter (homoscedasticity)
   - Funnel shape = heteroscedasticity (bad)
2. **Breusch-Pagan Test**: Statistical test
   - H₀: Constant variance (homoscedasticity)
   - p < 0.05 means we reject (heteroscedasticity detected)

**Results**:
- **Hypothesis 1**: Breusch-Pagan p = 0.891 → ✓ **PASSED**
- **Hypothesis 2**: Breusch-Pagan p < 0.001 → ✗ **FAILED**

**What this means**: 
- Hypothesis 1: Variance is constant (good)
- Hypothesis 2: Variance changes (heteroscedasticity) - we need robust SE

---

### Assumption 4: No Autocorrelation
**What it tests**: Are residuals independent of each other? (No correlation between today's error and yesterday's error)

**Why it matters**: 
- Autocorrelation makes standard errors too small
- P-values become too optimistic (think things are significant when they're not)

**How we tested**:
- **Durbin-Watson Test**: 
  - DW ≈ 2.0: No autocorrelation
  - DW < 1.5: Positive autocorrelation (bad)
  - DW > 2.5: Negative autocorrelation (rare)

**Results**:
- **Hypothesis 1**: DW = 1.985 → ✓ **PASSED**
- **Hypothesis 2**: DW = 1.984 → ✓ **PASSED**

**What this means**: Residuals are independent - no autocorrelation detected.

---

### Assumption 5: No Multicollinearity (Hypothesis 2 only)
**What it tests**: Are the independent variables too highly correlated with each other?

**Why it matters**: 
- If variables are too correlated, we can't tell which one matters
- Coefficients become unstable and unreliable

**How we tested**:
- **Variance Inflation Factor (VIF)**:
  - VIF < 5: Low multicollinearity (good)
  - VIF 5-10: Moderate (caution)
  - VIF > 10: High (problem)

**Results**:
- SPY_Return: VIF = 2.86
- VIX_Change: VIF = 2.87
- TSLA_Volume_Change: VIF = 1.01

**What this means**: ✓ **PASSED** - All VIF < 5, no multicollinearity problem.

---

## PART 4: ROBUST STANDARD ERRORS

### Why We Used Them

**When assumptions fail**, standard errors become unreliable. We used **Newey-West (HAC) Robust Standard Errors** to fix this.

**Newey-West HAC** stands for:
- **H**eteroscedasticity and **A**utocorrelation **C**onsistent
- Accounts for both heteroscedasticity AND autocorrelation
- Valid even when normality fails (asymptotically)

**When we applied them**:
- **Hypothesis 1**: Normality failed → Used robust SE for extra safety
- **Hypothesis 2**: Normality + Heteroscedasticity failed → Definitely needed robust SE

**What they do**:
- Adjust standard errors to account for assumption violations
- Provide correct p-values and confidence intervals
- Make our conclusions reliable

---

## FINAL SUMMARY OF FINDINGS

### Hypothesis 1: Tesla vs Market
✓ **Tesla's daily returns ARE significantly correlated with S&P 500 returns**
- Coefficient: 1.34 (Tesla is 34% more volatile than market)
- Highly significant (p < 0.001 with robust SE)
- R² = 16% (market explains 16% of Tesla's return variance)

### Hypothesis 2: Volume and VIX
✓ **Trading volume DOES significantly contribute beyond market movement**
✗ **VIX (volatility) does NOT significantly contribute**

**Key Findings**:
- **SPY_Return**: Highly significant (β = 1.20, p < 0.001)
- **TSLA_Volume_Change**: Significant (β = 0.0048, p = 0.005)
- **VIX_Change**: NOT significant (β = -0.0011, p = 0.106)

**Interpretation**:
- Market movement (SPY) is the strongest predictor
- Trading volume adds additional explanatory power
- Market volatility (VIX) doesn't help explain Tesla's returns beyond what market movement already explains

---

## WHY ALL THIS MATTERS

1. **Data Quality**: Cleaned and merged data ensures we're comparing apples to apples
2. **Proper Transformations**: Log returns and log volume make data suitable for regression
3. **Outlier Removal**: Prevents extreme events from distorting results
4. **Assumption Checking**: Ensures our statistical tests are valid
5. **Robust Standard Errors**: Makes results reliable even when some assumptions fail
6. **Interpretable Conclusions**: We can confidently say what factors matter and what don't

---

## KEY TAKEAWAYS

1. **Tesla moves with the market** (β = 1.34), but is more volatile
2. **Trading volume matters** - high volume days have different return patterns
3. **VIX doesn't help** - market volatility index doesn't add explanatory power beyond market returns
4. **Results are reliable** - we checked assumptions and used robust methods
5. **16% of variance explained** - there's still 84% unexplained (company-specific factors, news, etc.)

---

## STATISTICAL TESTS REFERENCE

| Test | What It Tests | What We Found |
|------|---------------|---------------|
| **Shapiro-Wilk** | Normality of residuals | Failed (p < 0.001) |
| **Breusch-Pagan** | Constant variance | H1: Passed, H2: Failed |
| **Durbin-Watson** | No autocorrelation | Both passed (DW ≈ 2.0) |
| **VIF** | No multicollinearity | All VIF < 3 (passed) |
| **Z-score** | Outlier detection | Removed 134 outliers |

---

## FINAL VERDICT

✅ **Both hypotheses are interpretable and reliable**
- Used robust standard errors to account for assumption violations
- Strong statistical evidence for our conclusions
- Large sample size (n = 2,821) supports validity
- All critical assumptions either passed or were corrected with robust methods

