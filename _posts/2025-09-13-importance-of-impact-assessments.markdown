---
layout: post
title:  "Impact Assessments pt. 1 - Intro"
date:   2025-09-12
categories: impact assessments
---
This is the first in a series of posts dedicated to implementing an impact assessment (IA). In this series we will examine the importance of robust evaluation methods, detail the steps involved, and specify tools that help with the implementation. We will also review and interpret results to help us maximize outcomes.

IAs are used in evaluating health system programs because of the focus on measuring a program's true impact on health outcomes through attribution. We isolate a program's effect by statistically "controlling for" other variables that can influence the results. This part is key to reliable evidence-based policy making that informs program improvements and helps drive better health outcomes. Additionally, we use the results of an IA in cost-effectiveness analysis.

The core of our analysis consists of two parts: first the design phase followed by the analysis phase. In the design phase we perform identification of the treatment effect parameters in order to adjust for confounding that can arise when assignment to treatment is non-randomized, like in observational studies such as the one we will cover in the next parts of this series. For this we typically use a statistical technique called "matching" to select a control group with a set of key covariates that are balanced with the treatment group. The plot below shows an example of the outcomes of the mean differences between the groups before and after matching. 

![plot of mean differences before and after matching](/assets/mean-differences-plot.png)

The second phase of the analysis is estimation of the treatment effect parameters. For this we use a popular quasi-experimental comparative study design called difference-in-differences (DiD) that compares the average change in outcomes over time between the treatment and selected control group. The plot below shows an example the outcomes of the treatment effect varying by length of exposure.

![event study type plot](/assets/event-study-type.png)

In the following posts we will walk through the specific design and setup for generating these types of results.