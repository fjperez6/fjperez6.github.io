---
layout: post
title:  "The Importance of Impact Assessments pt. 1"
date:   2025-09-13 20:09:05 -0700
categories: impact assessments
---
This is the first in a series of posts dedicated to impact assessments (IA). We will examine the importance of robust evaluation methods, detail the steps involved, and identify tools that help with the implementation. We will also review and interpret results to help maximize outcomes.

IAs are used in evaluating health system programs through causal inference methods that measure a program's true impact on health outcomes. The focus is on attribution by statistically "controlling for" other variables that can influence the result in order to isolate a program's effect. This part is key to reliable evidence-based policy making that informs program improvements and helps drive better health outcomes. Additionally, the results of an IA can be used in cost-effectiveness analysis.

The core of our analysis consists of two parts, first a design phase followed by an analysis phase. In the design phase, we perform identification of the treatment effect parameters in order to adjust for confounding that can arise when assignment to treatment is non-randomized, like in observational studies such as the one we will cover in the next parts of this series. For this we typically use a statistical technique called "matching" to select a control group with a set of key covariates that are balanced with the treatment group. The plot below shows the outcomes of the mean differences between the groups before and after matching. 

![plot of mean differences before and after matching](/assets/mean-differences-plot.png)

The second phase of the analysis is estimation of the treatment effect parameters through a commonly used quasi-experimental comparative study design called difference-in-differences (DiD) that compares the average change in outcomes between the treatment and selected control group. The plot below shows the outcomes of the treatment effect varying by length of exposure.

![event study type plot](/assets/event-study-type.png)
