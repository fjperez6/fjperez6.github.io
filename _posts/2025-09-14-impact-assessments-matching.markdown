---
layout: post
title:  "Impact Assessments - matching pt. 2"
date:   2025-09-14
categories: impact assessments
---
This is the second post in the series about impact assessments (IA). Here we will discuss how incorporating matching as part of an impact assessment can produce more robust outcomes and enhance the estimation of the effect. We use one-to-one exact matching which is the most intuitive approach and produces the simplest outcomes to interpret.

Statistical matching is a data preprocessing technique that makes treatment and control groups more comparable, mimicking a randomized study. I use the MatchIt package, which helps produce more robust results that depend less on specific modeling assumptions.

### Working Within the Context of Prior Research

In this example we will walk through the matching process on a dataset of synthetic IDs and visit data to show how to format and prepare the data for use with the program. We then interpret and evaluate the outcomes with the tools included in the package. The steps are based on a report by the Center for Health Policy Research (CHPR) at UCLA that provides a very comprehensive evaluation of the Whole Person Care pilot.

In their report, CHPR provides a detailed list of the pre-treatment covariates they used in selecting the control group for the evaluation. I find this list to be a great starting point for inclusion in any health policy evaluation because it lists many of the variables that can be confounders in health plan interventions. ![table of sample variables](/assets/table-sample-variables.png)

Most of the static covariates are straightforward to collect such as the demographics and health status indicators. On the other hand, those that vary based on the enrollment date, such as pre-treatment visits, require a bit more work to collect. Additionally, because most programs don’t have a single enrollment date for enrollees, we must deal with the challenge of variation in treatment timing or rolling enrollment. This means that for treated individuals we use a fixed time period immediately prior to the enrollment date to collect the pre-treatment covariates. And for the potential matches, we calculate pre-treatment covariates for every individual at every time interval in the study period. In the following sections I detail how this plays out in a simulated application. This includes getting the necessary data for the enrollment group as well as for the potential match pool from which we will select a control group. 

### Simulated Datasets - Enrollees

First we prepare the simulated set of individuals. We begin by collecting the static demographic data. This code generates a sample of 500 random IDs along with only three demographic covariates to keep things simple and speedy. We then assign them a start date that is arbitrarily selected within the study period. This information represents the type of data that we typically collect from the program administrators at the beginning of each assessment. The parameters for the distributions are set to resemble the types of populations we typically work with. 

{% highlight r %}
library(tidyverse)
# enrollment list
n_enroll <- 500
list_ids_enroll <- paste0('m-', sample(x = 10000:20000
                                      , size = n_enroll
                                      , replace = FALSE))
list_dates_enroll <- sample(seq(from = DATE_START_STUDY
                              , to = DATE_END_STUDY
                              , by = 'day')
                          , size = n_enroll)
list_ages_enroll <- round(rnorm(n_enroll, mean = 45, sd = 15))
list_male_enroll <- sample(c(1, 0)
                        , n_enroll
                        , replace = TRUE
                        , prob = c(0.60, 0.40))
list_race_ethnicity_enroll <- sample(c('hispanic', 'white', 'black' , 'asian')
                                  , n_enroll
                                  , replace = TRUE
                                  , prob = c(0.55, 0.3, 0.1, 0.05))

df_enroll <- data.frame(id = list_ids_enroll
                      , date_enroll = list_dates_enroll
                      , age = list_ages_enroll
                      , male = list_male_enroll
                      , race_ethnicity = list_race_ethnicity_enroll
                      ) %>%
             mutate(date_quarter_enroll = floor_date(date_enroll, 'quarter')
                  , age = pmin(pmax(age, 18), 95)
                  , treatment = 1)
{% endhighlight %}

The first data transformation we make is for producing the quarters of the enrollment date. I find that aggregating activity by quarters provides the ideal amount of granularity to spot trends without getting lost in the noise when performing the analysis. This table shows a sample of the demographic DF for the simulated enrollees.

![table of simulated enrollees](/assets/table-simulated-enrollees.png)

Next, we collect the data for the visits up to two quarters prior to the start of the study period. This will be formatted in a way that each individual has their enrollment quarter and the count of visits for one and two periods before their enrollment quarter. This step can be tricky to get right because when we aggregate the visits, any quarter with no values is skipped. For instance, notice how there are no 0 values for any of the records of aggregated quarterly visits. Instead the only records we see are those with at least one visit for that time period, Otherwise the time period is skipped. 

![table with missing values](/assets/table-missing-values.png)

If we gathered the prior utilization from this point we would only get the 1 & 2 visits from the prior quarters that have one or more visits, skipping all of the periods with 0 visits. To fix this, and get the appropriate prior visits, we need to use an index consisting of each distinct ID and date quarter in the study period. We start by creating a DF with a cross join of the IDs-quarters to serve as the primary for the join with the visits DF. We now see a 0 for all the time periods that were previously skipped. So far so good.

![table with missing values added](/assets/table-missing-values-added.png)

Next we get the visit values from the actual 1 & 2 prior quarters which is simple with dplyr's group_by() and lag() function. The final step for the enrollee group is combining the prior visit DF to the demographics DF as the primary table for the join. This results in a one-row-per-record format for each id.

![table combined enrollees](/assets/table-combine-enrollees.png)

### Simulated Datasets - Potential Matches

Next, for the pool of potential matches, all the steps are the same for the demographics as well as the prior visits DF. The main difference is when combining the DFs we instead use the prior visits as the primary in order to keep the IDs-quarters index. Formatting this way produces multiple rows per ID, one for each time period. With this we keep the prior visits for each ID at each quarter for the entire study period allowing for a potential match with the enrollees at any point through the study period.

![table potential matches](/assets/table-potential-matches.png)

For the matching steps, first we combine the enrollee and potential match DFs into a single DF that is formatted to be processed by the MatchIt function. We also define the formula which identifies the covariates to be matched on. Finally, we set the specifications and fit the function on the pre-processed data. In this example we use the basic 1-to-1 matching scheme with no ID reuse. This means that each enrollee ID can be matched to only one potential match ID making the treatment and control groups the same size. 

MatchIt supports a large range of additional matching methods that can be rather sophisticated in their design. Full matching, for example, allows us to keep all the members in the potential match group and assign them a weight to balance the groups. This increase in sample size often produces more precise estimates. But the increase in complexity comes with a challenge of interpretation and acceptance by the customers we support.

The key to dealing with the staggered enrollment issue we mentioned earlier is in the specification for the "exact" parameter. By setting this to the enrollment quarter, MatchIt will search through each potential match at each of the quarters in the study period to find the most appropriate match at the enrollee's enrollment quarter. For example, if a specific enrollee has an enrollment date of the first quarter in 2023, MatchIt will search for the ideal match based on demographics as well as prior visits for the first quarter of 2023 for each of the potential matches. 

### Assessing the Matches

The final step in this process is to check the quality of the matches. Matchlt includes tools for this by running a summary of the output object. This prints a table with the means of the covariates of the two groups before the matching and after to show how well the groups were balanced by the tool. A good rule of thumb is to have the standard mean difference to be less than “.1” All of the covariates are less than that so this shows a good balance between the groups.

![table summary matches](/assets/table-summary-matches.png)

In addition to the summary output, MatchIt includes a dot plot with variable names on the y-axis and standardized mean differences on the x-axis. This helps display the covariate with each point representing the standardized mean difference before and after matching. 

![plot of mean differences before and after matching](/assets/mean-differences-plot.png)


