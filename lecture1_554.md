# Lecture 1: Multiple Comparisons

Want to address causality from:

- Randomized studies
  - Can **randomly** allocate control (placebo) and treatment groups
- Non-randomized studies

  - Treatment randomization is impossible
  - Need to apply statistical methods to address it

## Observational vs Experimental Studies

- **Experimental**: The researcher **manipulates** the independent variable and observes the effect on the dependent variable.
  - E.g See if new gym class reduces body fat percentage, so **randomly assign** people to new and old class
- **Observational**: The researcher observes the effect of the independent variable on the dependent variable without manipulating the independent variable.
  - E.g. See if new gym class reduces body fat percentage, so **select** people who signed up for new and old class

## Hypothesis Testing Review

- Recall [552 Hypothesis Testing](https://mds.farrandi.com/block_2/552_stat_inter/552_stat_inter#hypothesis-testing)

- Tolerance to **Type I Error** is $\alpha$
  - $\alpha = P(\text{reject null} | \text{null is true})$
  - Reject $H_0$ when $p < \alpha$

### Why is it Important in Science?

- Easier to publish paper if findings are significant at $\alpha = 0.05$
- By Construction:
  - $H_0$ is "conservative"/ "boring"/ "status quo"
  - $H_a$ is "exciting"/ "new"/ "interesting"
- When $p > \alpha$, we fail to reject $H_0$
  - Does not mean $H_0$ is true
  - Just means we do not have enough evidence to reject it
  - Makes bad headlines: (e.g. "Null Hypothesis Not Rejected")
- **P-Hacking**: Repeatedly testing until $p < \alpha$
  - Inflate Type I Error Rate
  - Does not always mean if $p < \alpha$, the result is significant
    - BE SCEPTICAL OF PAPERS, DO NOT BELIEVE EVERYTHING
  - Need **Bonferroni Correction** to adjust for multiple comparisons

### Example: Finding out Food to Health Relationship

- Have 20 foods to test and 20 body parts to test
- In total get 400 tests (20 foods \* 20 body parts)
- Simulate that the food is NOT related to any body part
- However, when we randomly test, we will get around 40% of the tests significant at $\alpha = 0.05$
  - 162 tests is significant of the 400
- This is because of **Multiple Comparisons**
  - We inflate the Type I Error Rate ($\alpha$ is too high)
  - Need to adjust (decrease) $\alpha$ to account for multiple comparisons

#### Why is this the case?

$$
E_ i = \text{Committing Type I error in the } i \text{th test} \\
P(E_ i) = \alpha.
$$

The probability of NOT committing Type I error in the $i$th test is the following:
$$P\left(E_ i^c\right) = 1 - \alpha$$

Probability of NOT committing Type I error in any of the tests is:
$$P\left(\bigcap_{i=1}^n E_ i^c\right) = \left(1 - \alpha\right)^n$$

Probability of committing at least one Type I error in the tests is:
$$P\left(\bigcup_{i=1}^n E_ i\right) = 1 - P\left(\bigcap_{i=1}^n E_ i^c\right) = 1 - \left(1 - \alpha\right)^n$$

The inflated probability corresponds to committing AT LEAST one Type I error in the $m$ tests.

## Multiple Comparisons

### Bonferroni Correction

- Conservatively guard against p-hacking
- Idea: If we do $m$ tests, then $\alpha$ should be $\frac{\alpha}{m}$

```r
pval # Matrix of p-values

# Bonferroni Correction
pval_corrected = p.adjust(pval, method = "bonferroni")
```

### Bonferroni Guarantee

Let $R_i$ be the event of rejecting $H_0$ when $H_0$ is true for the $i$th test. Then:

$$
P\left(E_ i^c\right) \leq \sum_{i=1}^m P\left(R_i\right)
$$

This leads to the **Family-Wise Error Rate (FWER)**

- The chance that **one or more** of the true null hypotheses is rejected

$$
FWER \leq \sum_{i=1}^m \frac{\alpha}{m} = \alpha
$$

- The Bonferroni Correction guarantees that we **wrongly reject** the null hypothesis with probability less than $\alpha$

- **Drawback: Very Conservative**

$$P(R_1 \; \cup \; R_2) \leq P(R_1) + P(R_2)$$

Works if $R_1$ and $R_2$ are _mutually exclusive_.

- If not, then $P(R_1 \; \cup \; R_2) =  P(R_1) + P(R_2) - P(R_1 \; \cap \; R_2)$

So then the there will be too much correction -> TOO CONSERVATIVE / More Type II Errors

### False Discovery Rate (FDR)

- We cannot just look at p-value and see if study is significant
- Need to find out how much "hunting" they did
- There is a trade-off between **false discoveries** (FP) and **missing real effects** (FN)

$$FDR = \frac{FP}{FP + TP}$$

- **LESS STRICT** than Bonferroni Correction
  - It is a method to control the **Expected Proportion of False Positives** instead of the probability of **one or more** false positives

#### Benjamini-Hochberg (BH) Procedure

- One method to control FDR
- Method:

  1. Specify a _maximum acceptable FDR_ $\alpha$
  2. Order the p-values from smallest to largest
  3. Find the largest $k$ such that $p_{(k)} \leq \frac{k}{m} \alpha$
     - $p_{(k)}$ is the $k$th smallest p-value
  4. Take the $k$ smallest p-values as significant

- Get **BH adjusted p-values** as: $\min \Big\{ \frac{p\text{-value}_i \times m}{i}, \text{BH adjusted } p\text{-value}_{i + 1} \Big\}$

```r
# Benjamini-Hochberg Procedure
pval_corrected = p.adjust(pval, method = "fdr")
```

### When to Use FWER vs. FDR

| FWER                                | FDR                                    |
| ----------------------------------- | -------------------------------------- |
| When there is high confidence to TP | When there is certain proportion of FP |
| Want to be conservative             | Want to be less conservative           |
| Prefer **False Negatives**          | Prefer **False Positives**             |
| `TukeyHSD`                          | `pairwise.prop.test(method="BH")`      |
