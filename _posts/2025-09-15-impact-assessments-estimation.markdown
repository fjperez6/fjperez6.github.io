---
layout: post
title:  "Impact Assessments - estimation pt. 3"
date:   2025-09-14
categories: impact assessments
---
This is part 3 in the series on impact assessments. In this post we discuss the estimation procedure for the treatment effect using difference in differences (DiD)  which is one of the most popular methods for evaluating policy interventions. The data collection for this step is fairly straightforward and consists mainly of collecting two addtional inputs and preparing the data for processing by the DiD package.

### Group-Time Estimator

In this step we rely on an open source package for R called the `did` package by Callaway et. al. This approach functions as an extension of the canonical DiD  consisting of two groups and two time periods. The `did` package uses a “group-time” estimator that is robust to heterogeneity at various levels adding flexibility to help in highlighting treatment effect heterogeneity for the different enrollment groups as well as over time. For example we can see how the effect varies for groups that enrolled earlier in the study compared to later, and we can see how the effect evolves over time for any of the groups. Additionally, inference is carried out using a bootstrap procedure to produce simultaneous inferences that avoids the multiple testing problem and is more suitable for visualizing the uncertainty in the estimate than pointwise estimate. 

What I really found useful in this package is all the different ways of aggregating the effect in addition to all the ways of checking the robustness of the model. This lets as you test the assumptions under various conditions. 

Although identification is handled in the previous step using matching, `did` supports a type of plug-in for specifying a parametric estimator. By using this we can benefit from a doubly-robust estimator that is less reliant on stringent modeling conditions. This can help with allowing the parallel trends assumption to hold after conditioning on the covariates.

### Simulated Datasets - Longitudinal Visits Data

The additional data that we need to collect are the visits and enrollment months to calculate the Per Member Per Month (PMPM) value. We also need to make a few transformations to make the columns that represent the ID, the quarters and the group names into numeric values. We detail these steps in the following sections.

In this example we will once again generate a synthetic dataset of visits. We will encode this simulation with a large effect size to better illustrate the process and the outcomes.

{% highlight r %}
simulate_healthcare_data <- function(df_matched_local, date_start, date_end) {
  # total days in the period
  date_list <- seq(date_start, date_end, by= 'day')
  days_total <- length(date_list)
  df_visits_all <- data.frame(matrix(ncol = 2, nrow = 0))
  colnames(df_visits_all) <- c('id', 'date_visit')

  # daily visits for each id using poison distribution
  # higher mean for control
  # lower mean for in-treatment
  visits_lambda_control <- .2
  visits_lambda_treatment <- .15
  for (i in 1:nrow(df_matched_local)) {
    if (df_matched_local$treatment [i] == 1) {
      date_start_treatment <- df_matched_local$date_enroll[i]
      days_in_control <- as.integer(as.Date(date_start_treatment) - as.Date(date_start))
      days_in_treatment <- days_total - days_in_control

      date_visit_list <- rpois(days_in_control, lambda = visits_lambda_control)
      date_visit_list <- append(date_visit_list
                              , rpois(days_in_treatment, lambda = visits_lambda_treatment))
    } else {
      date_visit_list <- rpois(days_total, lambda = visits_lambda_control)
    }
    df_visit_data <- data.frame(
        id = df_matched_local$id[i]
      , date_visit = date_list
      , date_visit_n = date_visit_list
      ) %>%
      filter(date_visit_n > 0) %>% # exclude days with 0 visits select (id, date_visit)
      select(id, date_visit)

    df_visits_all <- rbind(df_visits_all, df_visit_data)
  }
  return(df_visits_all)
}
{% endhighlight %}

We first take the DF for the matched group of individuals from the previous steps and collect their visits during the study period. These visits will once again be aggregated at the quarterly level. Additionally, we will collect the number of enrollment months in the health care program for each of these quarters. We divide the number of visits by the number of enrollment months to the quarterly visits in PMPM format.

Now we are ready to prepare the DF for processing by the DiD package. The “yname” parameter refers to the dependent variable or `n_visits` in this case. For the “idname” we transform each ID to numeric by dropping the non-numeric prefix. The “tname” parameter refers to the time period and it also needs to be in numeric format. Because we are aggregating at the quarterly level we can simply add the value for the quarter to the end of the value for the year. For example, if a visit occured on 05/17/2023 it would be represented as 20232 and be counted with any other visits that occured in this quarter. Finally, the “gname” parameter refers to the group name and it represents the time period when the individual is enrolled in the program. This also needs to be in numeric format, we can transform it similarly to the “tname” by appending the quarter to the end of the year value. The only difference in the “gname” is that the control group will have a 0 as their value. Finally, I often find it is necessary to set the allow_unbalanced_panel as TRUE, but this would be something to check and see if there are any flaws or gaps in your data also.

![table processed for did](/assets/table-processed-did.png)

### Assessing the Estimates

Fitting the model produces an object that contains all the group-time effects which can be aggregated into a variety of summary parameters. The first aggregate we want to explore is by “group.” This produces the summary measure whose interpretation is the same as that of the ATT for the canonical DiD. This is also what the authors recommend to use as effect estimate. A summary of the aggregate output shows us the ATT estimate along with its confidence bands. We can plot the aggregated estimates to see how the outcomes vary by group although I find this plot is not the most intuitive to share with customers. 

![plot group att](/assets/plot-group-att.png)

The next aggregate I plot is ‘dynamic’ which represents an event plot style analysis that shows the variation of the effect over time. This is the most intuitive plot to share but with the caveat that this aggregate is for a different estimand and is not value we are using as the effect estimate. It this case it helps as robustness check. 

![event study type plot](/assets/event-study-type.png)

A summary of the group-time object shows the ATT for all the groups and all the times. The plot of this object shows all the plots which can be less informative if there are many groups and times involvded. I usually just show the last six groups by plotting the last group of three in one plot and the second to last group of three in another plot.

![group time plot](/assets/plot-group-time.png)
