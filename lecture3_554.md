# Lecture 3: Randomization and Blocking

## Foundations of A/B and A/B/n Testing

- **A/B Testing**: randomized experiment between **control** and **treatment** groups
- **A/B/n Testing**: randomized experiment between **control** for more than 2 groups (n is not the same as n samples)
- Recall: randomization makes variables independent of any confounding variables
- Hence we can infer causality instead of just association

### A/B Testing with Continuous Response

#### Common Terminology

- Example: Have two categorical variables: `color`: blue/red and `font_size`: small/ medium/ large to predict `duration` of stay on a website
- **Variables**: `color`, `font_size`
  - 6 possible treatment groups: 3 font sizes \* 2 colors
  - If all 6 are used -> **Full Factorial Design**, specifically **2x3 Factorial Design**
- Each user is **experimental unit**
- Each treatment group involves **replicates** (multiple experimental units)
  - The experimental units are randomly assigned to treatment groups

| Terms               | Definitions                                                                                                                                                                                                                                 |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Stratification      | Process of splitting the population into homogenous subgroups and sampling observational units from the population independently in each subgroup.                                                                                          |
| Factorial Design    | Technique to investigate effects of several variables in one study; experimental units are assigned to all possible combinations of factors.                                                                                                |
| Cofounder           | A variable that is associated with both the explanatory and response variable, which can result in either the false demonstration of an association, or the masking of an actual association between the explanatory and response variable. |
| Blocking            | Grouping experimental units (i.e., in a sample) together based on similarity.                                                                                                                                                               |
| Factor              | Explanatory variable manipulated by the experimenter.                                                                                                                                                                                       |
| Experimental Unit   | The entity/object in the sample that is assigned to a treatment and for which information is collected.                                                                                                                                     |
| Replicate           | Repetition of an experimental treatment.                                                                                                                                                                                                    |
| Balanced Design     | Equal number of experimental units for each treatment group.                                                                                                                                                                                |
| Randomization       | Process of randomly assigning explanatory variable(s) of interest to experimental units (e.g., patients, mice, etc.).                                                                                                                       |
| Treatment           | A combination of factor levels.                                                                                                                                                                                                             |
| A/B Testing         | Statistically comparing a key performance indicator (conversion rate, dwell time, etc.) between two (or more) versions of a webpage/app/add to assess which one performs better.                                                            |
| Observational Study | A study where the researcher does not randomize the primary explanatory variables of interest.                                                                                                                                              |

#### Two-way ANOVA with main effects only

- Typically initial analysis in A/B testing
- main effects: standalone regressors (e.g. `color`, `font_size`)

$$
Y_{i,j,k} = \mu_{i} + \tau_{j} + \varepsilon_{i, j, k}
$$

- For $i$th `font` and $j$th `color`, for the $k$th experimental unit

```R
OLS_ABs_main_effects <- lm(formula = Duration ~ Font + Color, data = data)
anova(OLS_ABs_main_effects)
```

- ANOVA tables go hand-in-hand with randomized experiments

<img src="images/3_anova.png" width="400">

- p-values tests the hypothesis that:
  - $H_0$: $\mu_{1} = \mu_{2} = \mu_{3}$ (no effect of `font`)
  - $H_a$: at least one $\mu_{i}$ is different
- Recall: If p-value < 0.05, we reject the null hypothesis
- `Df` represents the degrees of freedom
  - It is of the formula: `# of levels - 1`

#### Two-way ANOVA with interaction effects

$$
Y_{i,j,k} = \mu_{i} + \tau_{j} + (\mu\tau)_{i,j} + \varepsilon_{i, j, k}
$$

- Adds the interaction term $(\mu\tau)_{i,j}$ to the model
  - Does not indicate they are multiplied

```R
OLS_ABs_interaction <- lm(formula = Duration ~ Font * Color, data = data)
anova(OLS_ABs_interaction)
```

- `Df` for interaction term is the product of `Df1` and `Df2`

#### Post-hoc tests

- Compares all possible pairs among the levels
  - Involves multiple testing corrections
- Wil;l use Tukey's HSD test
  - Keeps the Family-Wise Error Rate (FWER) at $\alpha$ or less
  - Only for ANOVA models

```R
# Need aov: basically lm but for ANOVA
OLS_aov_ABn <- aov(formula = Duration ~ Font * Color, data = data)
TukeyHSD(OLS_aov_ABn, conf.level = 0.95) |> tidy()
```

## Blocking

- **Blocking Factor**: a variable that is not of primary interest (and cannot be controlled) but is known to affect the response variable
  - Consider as grouping factor
  - Examples: time period when A/B testing is conducted, user demographics, etc.
- **“Block what you can, randomize what you cannot.”**
  - **Blocking**: removing the effect of secondary measurable variables from the response
    - This is so our model could get statistical association/ causation between the secondary variable and the response variable
  - **Randomization**: removing the effect of unmeasurable variables

### Blocking Procedure

1. Stratify experimental units into homogenous blocks. Each stratum is a block
2. Randomize the experimental units into the treatment groups within each block

## Randomization and Blocking Recap

Sorted Best to Worst:

1. Model with blocking for blocking factors
2. Normal data with Post-Hoc adjustments for blocking factors
3. Raw model with no blocking
