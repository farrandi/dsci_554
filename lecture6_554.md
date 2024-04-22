# Lecture 6: Observational Studies

## Observational Studies

- We **do not** have control of the variable of interest.
- **Without randomization, life is harder**. Strategy includes:
  1. Recording potential confounders
  2. Use confounders as part of the analysis
  3. Tempering the causal strength in light of inherent challenges in (i) and (ii)

### Example: Pharmaco-epidemiology

- Response $Y$: binary indicator of disease state. (1: disease, 0: no disease)
- $X$: binary indicator of behavior (1: type A, 0: type B)
  - Type A: more agressive personality, competitive, etc
  - Type B: more laid back personality, relaxed, etc
- Confounders $C_j$ (e.g. age, sex, BMI, cholesterol level, etc.)

</br>

- Since it is binary => binary logistic regression
  - Use log-odds $logit(p) = \log(\frac{p}{1-p})$
  - odds-ratio: $\text{OR} = \frac{\frac{n_{X = 1, Y = 1}/ n}{n_{X = 1,Y = 0} / n}}{\frac{n_{X = 0, Y = 1}/ n}{n_{X = 0,Y = 0} / n}} = \frac{\frac{n_{X = 1, Y = 1}}{n_{X = 1,Y = 0}}}{\frac{n_{X = 0, Y = 1}}{n_{X = 0,Y = 0}}} = \frac{n_{X = 1, Y = 1} \times n_{X = 0,Y = 0}}{n_{X = 1,Y = 0} \times n_{X = 0, Y = 1}}$
    - OR = 1: X does not effect Y
    - OR > 1: X increases the odds of Y
    - OR < 1: X decreases the odds of Y
  - $\text{SE} = \sqrt{\frac{1}{n_{X = 1, Y = 1}} + \frac{1}{n_{X = 1,Y = 0}} + \frac{1}{n_{X = 0, Y = 1}} + \frac{1}{n_{X = 0, Y =0}}}$
- Can also just use binary logistic regression in R

```R
glm(Y ~ X, data = data, family = binomial) |>
  tidy(conf.int = 0.95)
```

#### Adding confounders

- Turn a continous confounder (e.g. age) into discrete categories and add to the model
  - `data |> mutate(age_bins = cut(age, breaks = c(min(age), quantile(age, (1:3) / 4), max(age)), include.lowest = TRUE)`
- By making **stratum-specific inference** with multiple confounders, we aim to infer causality between X and Y
  - However, there will be few observations in each strata (not enough data)
  - **Solution**: use binomial logistic regression (**Overall Model-Based Inference**) with interaction terms

```R
glm(Y ~ X * C1 * C2, data = data, family = binomial) |>
  tidy(conf.int = 0.95)
```

_recall: odds ratio is $exp(\beta)$ where $\beta$ is the coefficient/ estimate_

#### Assumptions for Causal Model-based Inference (binary logistic regression)

1. **Simple/ smooth structure in how the Y-specific log-OR varies across the strata**
   - Check using ANOVA comparing the simple model (all additive terms) and the complex model (with interaction terms of all confounders with each other)
   - `complex_model <- glm(Y ~ X + C1 * C2, data = data, family = binomial)`
   - `anova(simple_model, complex_model, test = "LRT")`
2. **Strength of $(X, Y)$ association is constant across the strata** (i.e. no interaction between X and C)
   - complex model: all simple terms + double interactions of X and confounders
   - `complex_model_c1 <- glm(Y ~ X + C1 + C2 + X:C1, data = data, family = binomial)`
   - `complex_model_c2 <- glm(Y ~ X + C1 + C2 + X:C2, data = data, family = binomial)`
   - Compare all models with simple model using ANOVA
3. **No unmeasured confounders** (All confounders are included in the model)
   - Add new model with unmeasured confounder
   - `new_model <- glm(Y ~ X + C1 + C2 + CU, data = data, family = binomial)` where CU is the unmeasured confounder
