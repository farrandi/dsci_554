# Lecture 7: Observational Data - Different Sampling Schemes

## Sampling Schemes in Observational Data

- Must Explore how to select our sample **before** executing the study
- Analysis must also account for the sampling scheme

### Sampling Assesment via the Ground Truth

- **Ground Truth**: checking the results of the study in terms of its estimation accuracy against the **real-world**
- Ground truth is hard to get/ not available in many cases
  - Frequentist paradigm: we **do not** have access to true population parameters
  - Need to reply on sample to estimate the population parameters

#### Simulation: Proxy for Ground Truth

- A better simulation technique than Monte Carlo
- **Core Idea**:
  - **Use a relevant sample dataset**: assumes previous sample data is representative of the population
  - **Fit a model** with `Y ~ X + C_j` (where $C_j$ are a determined set of confounders)
  - **Simulate a proxy ground truth** by generating new data from the model
- Still use the 3 assumptions for the causal model-based inference

- Steps:
  1.  Get a dataset and generate multiple rows of data (break continuous variables into quartile bins)
  2.  Fit a model with the data
  3.  Simulate a proxy ground truth by generating new data from the model

### Sampling Schemes using Proxy Ground Truth

- Three Sampling Schemes:
  1. **Cross-Sectional Sampling**
  2. **Case-Control Sampling**
  3. **Cohort Sampling**
- These schemes will imply different **temporalities**

#### Cross-Sectional (CS) Sampling Scheme

- **Contemporaneous**: all data is collected at the same time
  - Grab a simple random sample of size n from the population
  - Similar to an instantaneous snapshot of all study variables
- **Ideal** in **early research stages** to get a sense of the data
  - Fast to run

```R
set.seed(123) # for reproducibility

CS_sample_index <- sample(1:n, size = n_sample, replace = FALSE)
CS_data <- data[CS_sample_index, ]
```

#### Case-Control (CC) Sampling Scheme

- **Retrospective**: data is collected after the event of interest has occurred
  - Sample into cases `Y=1` and controls `Y=0` (Equal sample sizes for both)
  - Then ask the subjects "have you been exposed to `X` in the past?"
- **Ideal** where outcome `Y=1` is rare
  - Not have have a lot of cases of `Y=1` so recruit a lot of patients with `Y=1` for study
- It will **oversample** $Y=1$ cases and **undersample** $Y=0$ controls
  - Leads to a second statistical inquiry: "Is it a winning strategy to oversample cases?"
    - Do **modified Power Analysis** using Case:Control ratio
    - If ratio = 1, then control = case
    - If ratio > 1, then case > control
    - If ratio < 1, then case < control
  - According to SE behaviors, in populations when $Y=1$ is rare, get **more precise** estimates by oversampling cases and undersampling controls

```R
set.seed(123) # for reproducibility

CC_sample_index <- c(sample((1:n)[data$Y == 1], size = n_sample/2, replace = FALSE),
                      sample((1:n)[data$Y == 0], size = n_sample/2, replace = FALSE))
CC_data <- data[CC_sample_index, ]
```

#### Cohort (CO) Sampling Scheme

- **Prospective**: data is collected over time
  - Sample into exposed `X=1` and unexposed `X=0` (Equal sample sizes for both)
  - Then follow the subjects over time to see if they develop the disease `Y=1`
- `Y` is assumed as the recorded outcome at the end of the study
- **Ideal** for when exposure `X=1` is rare
  - Not have have a lot of cases of `X=1` so recruit a lot of patients with `X=1` for study

```R
set.seed(123) # for reproducibility

CO_sample_index <- c(sample((1:n)[data$X == 1], size = n_sample/2, replace = FALSE),
                         sample((1:n)[data$X == 0], size = n_sample/2, replace = FALSE))
CO_data <- data[C0_sample_index, ]
```

### Sampling Scheme Assesment

- Need to run multiple simulations to get a sense of the variability of the estimates
- Use function `sim_study` to run the simulation

```R
sim_study <- function(pop_data, n, alpha, log_OR, num_replicates) {
  res <- list(NULL) # Setting up matrix with metrics
  res[[1]] <- res[[2]] <- res[[3]] <- matrix(NA, num_replicates, 3)

  suppressMessages(for (lp in 1:num_replicates) { # Otherwise, we get "Waiting for profiling to be done..."
    # Obtaining samples by scheme
    # CS
    CS_sampled_subjects <- sample(1:nrow(pop_data), size = n, replace = F)
    CS_sample <- pop_data[CS_sampled_subjects, ]
    # CC
    CC_sampled_subjects <- c(
      sample((1:nrow(pop_data))[pop_data$chd69 == "0"],
        size = n / 2, replace = F
      ),
      sample((1:nrow(pop_data))[pop_data$chd69 == "1"],
        size = n / 2, replace = F
      )
    )
    CC_sample <- pop_data[CC_sampled_subjects, ]
    # CO
    CO_sampled_subjects <- c(
      sample((1:nrow(pop_data))[pop_data$dibpat == "Type B"],
        size = n / 2, replace = F
      ),
      sample((1:nrow(pop_data))[pop_data$dibpat == "Type A"],
        size = n / 2, replace = F
      )
    )
    CO_sample <- pop_data[CO_sampled_subjects, ]

    # Do the three analyses
    # CS
    CS_bin_log_model <- glm(chd69 ~ dibpat + age_bins + smoke + bmi_bins + chol_bins,
      family = "binomial", data = CS_sample
    )
    # CC
    CC_bin_log_model <- glm(chd69 ~ dibpat + age_bins + smoke + bmi_bins + chol_bins,
      family = "binomial", data = CC_sample
    )
    # CO
    CO_bin_log_model <- glm(chd69 ~ dibpat + age_bins + smoke + bmi_bins + chol_bins,
      family = "binomial", data = CO_sample
    )

    # and the takeaways
    res[[1]][lp, ] <- c(coef(CS_bin_log_model)["dibpatType A"], confint(CS_bin_log_model)["dibpatType A", ])
    res[[2]][lp, ] <- c(coef(CC_bin_log_model)["dibpatType A"], confint(CC_bin_log_model)["dibpatType A", ])
    res[[3]][lp, ] <- c(coef(CO_bin_log_model)["dibpatType A"], confint(CO_bin_log_model)["dibpatType A", ])
  })

  # Summaries
  BIAS <- sapply(
    res,
    function(mat) {
      mean(mat[, 1]) - log_OR
    }
  )
  vrnc <- sapply(res, function(mat) {
    var(mat[, 1])
  })
  CVRG <- sapply(res,
    function(mat, trg) {
      mean((mat[, 2] < trg) & (trg < mat[, 3]))
    },
    trg = log_OR
  )
  PWR <- sapply(res, function(mat) {
    mean(mat[, 2] > 0)
  })
  RMSE <- sqrt(BIAS^2 + vrnc)

  opt <- cbind(BIAS, RMSE, CVRG, PWR)
  rownames(opt) <- c("Cross-Sectional (CS)", "Case-Control (CC)", "Cohort (CO)")

  return(opt)
}
```

- Function inputs:
  - `pop_data`: the dataset where we will sample from
  - `n`: sample size
  - `alpha`: significance level $\alpha$
  - `log_OR`: true log odds ratio
  - `num_replicates`: number of simulations to run
- The function `sim_study` will return a matrix with the following metrics:
  - **BIAS**: difference between the estimated and true log odds ratio
  - **RMSE**: root mean squared error
  - **CVRG**: coverage of the 95% confidence interval (as a proportion of `num_replicates`)
  - **PWR**: power of the test
