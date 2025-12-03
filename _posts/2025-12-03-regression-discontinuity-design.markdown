---
layout: post
title:  "Regression Discontinuity Design"
date:   2025-12-03
categories: impact assessments
---
This post presents an impact assessment evaluating the effectiveness of a predictive analytic product designed to classify participant non-adherence to prescribed medication. The objective of this product was to support and optimize a targeted medication adherence outreach campaign by using a structured risk classification system to stratify participants into distinct intervention arms: pharmacist-led or technician-led outreach groups.

In the absence of random assignment, we employed a quasi-experimental research design, specifically a regression discontinuity design, that allows us to observe a type of natural experiment and identify the treatment effect as the discontinuity of outcomes between groups near the decision cutoff threshold. The underlying rationale is the assumption of local randomization: participants immediately proximate to the assignment cutoff threshold are expected to be statistically similar, on average, which enables a robust comparison between the technician and pharmacist intervention groups.

Our evaluation addressed the following primary research questions:
- R1) Did the predictive risk score accurately categorize participants based on their Proportion of Days Covered (PDC), utilized as a proxy measure for adherence?
- R2) Did the risk score-based assignment strategy improve the efficiency and targeting of campaign resources?

### Methodology
The predictive analytic risk score employed a risk classification system with three ordinal tiers: 1 (low), 2 (medium), and 3 (high). Classification was derived from historical medication adherence pass/fail rates and various healthcare data points.

The model generated a distinct set of three risk scores for each participant, corresponding to three different target medications. Assignment to an intervention arm (pharmacist-led vs. technician-led) was determined by these scores: participants exhibiting two or more risk scores of '3' (high risk) were assigned to the pharmacist group; all others were assigned to the technician group.

### Simulated Datasets - Longitudinal PDC Data
The data utilized for this example is simulated difference in PDC rates between the final and initial observations for participants. The validity of the simulated treatment effect is visually confirmed through a graphical analysis that specifically displays the difference in PDC rates for the technician (red line) and pharmacist (blue line) intervention groups.

Risk scores range from 1 (indicating low risk across all medications) to 10 (indicating high risk across all medications). The true treatment effect is made visible by the clear discontinuity in outcomes precisely at the decision cutoff threshold (an aggregate risk score of 7).

{% highlight r %}
# Load necessary libraries
library(ggplot2)
library(dplyr)
# Set a seed for reproducibility
set.seed(123)

# Define parameters
N <- 1500         # Number of observations
cutoff <- 7       # The cutoff point for treatment
beta_0_t <- 90    # Intercept technician group
beta_0_p <- 82    # Intercept pharmacist group
beta_1_t <- -1    # Slope of the running variable relationship technician group
beta_1_p <- .2    # Slope of the running variable relationship pharm group
noise_sd <- 3     # Standard deviation of the error term

# 1. Generate the running variable (assignment variable)
# Let's assume the running variable (X) are normally distributed integers
X <- round(rnorm(n = N, mean = 5, sd = 2))

# 2. Determine treatment assignment (D) based on the fuzzy cutoff
# D is 1 if X is > cutoff, 0 if X < cutoff, 
# and equal chance of 1|0 at cutoff
D <- case_when (
         X > cutoff ~ 1
       , X < cutoff ~ 0
       , .default = round(runif(X))
     )

# 3. Model the outcome variable (Y)
# We add a random noise term (epsilon)
epsilon <- rnorm(N, mean = 0, sd = noise_sd)

Y <- case_when (
         D == 1 ~ beta_0_p + beta_1_p * X + epsilon
       , .default = beta_0_t + beta_1_t * X + epsilon
     )

# 4. Create a data frame
rdd_data <- data.frame(X = X, D = D, Y = Y) %>%
  filter (X >= 1
        & X <= 10)
{% endhighlight %}

### Statistical Analysis
To evaluate the intervention's effect, we employed statistical process control (SPC) methodology to monitor outcome variations over time and identify signals of significant change points. Graphical analysis was utilized to visualize and compare outcome differences between participants assigned to the distinct intervention arms. To facilitate this graphical comparison, two separate polynomial curves were fitted to estimate the expected outcomes for each group (e.g., pharmacist-led vs. technician-led) and emphasize the divergence in trends between the cohorts.

![plot processed for rdd](/assets/plot-rdd.png)


{% highlight r %}
# 5. Visualize the data to check for the discontinuity
rdd_data %>%
  ggplot(aes (x = X, y = Y, color = factor (D))) +
  geom_point (alpha = 0.3) +
  geom_smooth (method = 'gam'
             , formula = y ~ s(x, bs = 'cs' , k=3)) +
  geom_vline(xintercept = cutoff, linetype = 'dashed', color = 'black') +
  labs(title = 'Simulated RDD Data with Treatment Effect'
     , x = 'Risk Score'
     , y = 'PDC Rate'
     , color = 'Treatment') +
  theme_minimal() +
  scale_x_continuous(labels = scales::label_number (accuracy = 1))      
{% endhighlight %}

### Results
For R1 we find that the analytic product accurately categorized participants by risk evidenced by the plot showing those with higher predicted risk scores had worse outcomes in the outreach campaign. As expected, medication adherence rates declined for participants with higher predicted risk scores.

For R2 we find that the risk score-based assignment effectively optimizes the utilization of resources evidenced through graphical analysis that shows that the pharmacist group (blue) achieved outcomes that were comparable to or better than those of the technician group (red) as the risk scores increased. These results are notable because the pharmacist group was assigned the highest-risk participants, while the technician group was assigned the lowest-risk participants.

### Limitations
A major challenge we encountered is that the PDC measures are meant to be used in an annual scale. This limits the ability to make interpretations at a weekly scale for ongoing learning rather than an end of the year "pre-post" evaluation. We note that this evaluation does not include information about engagement such as total contacts or other program processes.

Notably, combining all participants based on total weeks in the program does not allow for comparing outcomes of early and later entrants into the pharmacy program. Also, SPC allows for testing the difference between technician and pharmacist groups, but this was not part of the evaluation.
