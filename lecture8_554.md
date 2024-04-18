# Lecture 8: Ordinal Regressors

## Matched Case-Control Studies

- All our previous sampling schemes used a **binary logistic regression** model
- Recall we checked that CC is the best compared to the other 2 designs (CS and CO) in terms of power when $Y=1$ is rare

### Alternative Data Collection

- Using CC is acceptable because:
  - We assumed $Y|X, C_1, ..., C_p$
  - Create artificial population by "cloning subjects"
    - This is done using the estimated model from (previous reperesentative sample) + induced random noise
    - AKA Proxy Ground Truth
- CC is useful when %Y=1 is rare and sampling is costly

### CC Matching

- Need to have artificial population to be representative of the true population
- Then will sample to get a sample size of $n$
- Record the confounders of interest as **strata**
- Sample $n/2$ cases then $n/2$ controls
  - Keep Case:Control ratio = 1
  - Match **exactly** on the confounder to the case counterpart
- **Important**: Cannot fit Binary Logistic Regression model since we have matched pairs
  - Can get a sparse data problem
  - Use **McNemar's Test** instead
- CC-matched will show a smaller average bias compared to CC-unmatched
  - Power is the same once $n$ increases

|            | Control $X=0$ | Control $X=1$ |
| ---------- | ------------- | ------------- |
| Case $X=0$ | $n_{00}$      | $n_{01}$      |
| Case $X=1$ | $n_{10}$      | $n_{11}$      |

- $n_{00}$ and $n_{11}$ are the **concordant pairs**
- $n_{01}$ and $n_{10}$ are the **discordant pairs**
- Estimator of the **odds ratio** is based on discordant pairs: $OR = \frac{n_{10}}{n_{01}}$
  - $OR = 1$ implies no association
  - $OR > 1$ implies positive association
  - $OR < 1$ implies negative association

#### McNemar's Test

- $H_0: log(OR) = 0$, $H_a: log(OR) \neq 0$
- $log(OR) = log{n_{10}} - log{n_{01}}$
- It is approximately normally distributed with an SE of:
  - $SE = \sqrt{\frac{1}{n_{10}} + \frac{1}{n_{01}}}$
- Test statistic: $Z = \frac{log(OR)}{SE}$

## Ordinal Regressors

- The numerical confounding strata we have been using are ordinal

### Example

- $Y$ is continuous, $X$ is ordinal

1. Need to convert the variable $X$ to ordinal

```R
data$X_ord <- ordered(data$X, levels=c("low", "medium", "high"))
```

2. Fit a **one-way analysis of variance (ANOVA)** model

- R uses **polynomial contrasts** in ordered type factors
  - Elements of vectors sum to 0
  - Roughly, if have `k` levels, then `k-1` polynomials
  - E.g. `l=4`, we will have linear, quadratic, cubic contrasts
  - Use `contr.poly(l)` to get the contrasts when `l` levels
    - Gives design matrix for the contrasts
    - Cols sum to 0
    - Rows are the contrasts and are orthogonal

```R
OLS <- lm(Y ~ X_ord, data=data)

# Get contrasts
OLS |> model.matrix()
```

3. Hypothesis test:
   - $H_0$: there is no GIVEN trend in the ordered data
   - $H_1$: there is a GIVEN trend in the ordered data
   - GIVEN will be replaced with linear, quadratic, cubic, etc

```R
plot <- eda_plot +
    geom_smooth(aes(x=unclass(X_ord), color=1),
        formula=y~x, method="lm", se=FALSE) +
    geom_smooth(aes(x=unclass(X_ord), color=2),
        formula=y~poly(x, 2), method="lm", se=FALSE) +
    geom_smooth(aes(x=unclass(X_ord), color=3),
        formula=y~poly(x, 3), method="lm", se=FALSE)
```

#### Successive Difference Constrasts

- Alternative to make inferential interpretations more straightforward
- Want to answer whether differences exist between ordered levels

```R
options(contrasts = c("contr.treatment", "contr.sdif"))

OLS_sdif <- lm(Y ~ X_ord, data=data) |> tidy()
```

- Interpretation is very straightforward
