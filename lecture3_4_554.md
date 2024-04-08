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

- Done if any factor is significant
- Compares all possible pairs among the levels
  - Involves multiple testing corrections
- Will use Tukey's HSD test
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

<img src="images/3_blocking_count.png" width="500">

_Strategy A: Just Randomize, Strategy B: Randomize and Block_

### Blocking Procedure

1. Stratify experimental units into homogenous blocks. Each stratum is a block
2. Randomize the experimental units into the treatment groups within each block

### Randomization and Blocking Recap

Sorted Best to Worst:

1. Model with blocking for blocking factors + Post-Hoc adjustments
2. Normal data with Post-Hoc adjustments for blocking factors (Treat blocking as a covariate)
   - **covariate**: a variable that is possibly predictive of the response variable
3. Raw model with no blocking nor Post-Hoc adjustments

### Increasing Sample Size

- Increasing sample size would:
  - Increase the **accuracy** of the estimates
  - Increase the **power** of the test
    - **Power**: probability of rejecting the null hypothesis when it is false (predict correctly)
      - High power means our A/B test is robust enough to detect and estimate experimental treatment effects
    - In class had example: Need 10x the sample size for raw to perform equal to covariate/ blocking
- But this model would not capture the right systematic component of the population, including the **stratum effect** (demographic categories)
- Using covariate and blocking would still be more precise
- Need to consider that increasing sample size is also expensive

#### Power in Sample Size Computations

- Recall that sample size computations involve playing around with three concepts:
  1. **effect size**: how much we want the experimental treatment to differ from the control treatment in terms of the mean response
  2. **significance level** $\alpha$
     - Lower = more strict (less prone to Type I error/ false positive)
  3. **power** of the test $1 - \beta = 1 - P(\text{Type II error})$,
     - Typically 0.8
     - larger = less prone to Type II error

<img src="images/0_stat_chart.png" width="500">

| Decision      | True Condition Positive        | True Condition Negative       |
| ------------- | ------------------------------ | ----------------------------- |
| Test Positive | Correct (True Positive)        | Type I Error (False Positive) |
| Test Negative | Type II Error (False Negative) | Correct (True Negative)       |

_see more in [my statistical inference notes](https://mds.farrandi.com/block_2/552_stat_inter/552_stat_inter.html#visual-representation-of-errors)_

##### Example case

- Lets say we want to see if changeing website design from "X" to "Y" will increase the average time spent on the website.
- We know:
  - Current average is between 30s to 2 min
  - We want to see a 3% increase in time spent
- Calculation:
  - $\mu_x = 0.5 + \frac{2-0.5}{2} = 1.25 \text{ min}$
  - Using 95% CI, $Z_i = \frac{Y_i - \mu_A}{\sigma} \sim \mathcal{N}(0, 1)$
    - $1.96 = \frac{2 - \mu_A}{\sigma}$
    - $\sigma = \frac{2 - 1.25}{1.96} = 0.383$
  - $\delta = 0.03 \times 1.25 = 0.0375$ (desired difference)
    </br>
- Use `pwr.t.test` function in R to calculate sample size needed
  - will give `n` (sample size) needed for **each** group

```R
pwr.t.test(
  d = 0.0375 / 0.383, # delta / sigma
  sig.level = 0.05,
  power = 0.8,
  type = "two.sample",
  alternative = "greater" # we want to see an increase
)
```

### Block Homogeneity and Power

- We want to ensure that the blocks are **homogeneous**:
  - As **much** variation in $\mu$ as possible is **across** the blocks
  - As **little** variation in $\mu$ as possible is **within** the blocks
- If not homogenous: blocking can be worse than raw
- The blocking setup needs to be carefully planned before running the experiment.
