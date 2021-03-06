Jared Knowles, Lauren Dahlin  
April 7, 2017  

# High School Graduation
*College-Going Pathways*
*R Version*

## Getting Started




<div class="navbar navbar-default navbar-fixed-top" id="logo">
<div class="container">
<img src="..\img\open_sdp_logo_red.png" style="display: block; margin: 0 auto; height: 115px;">
</div>
</div>

### Objective
 
In this guide you will be able to visualize high school graduation rates
by high school, student achievement level before high school, student
race/ethnicity, on-track status after ninth-grade.
 

### Using This Guide

The College-Going Pathways series is a set of guides, code, and sample data about
policy-relevant college-going topics. Browse this and other guides in the series for 
ideas about ways to investigate student pathways through high school and 
college. Each guide includes several analyses in the form of charts together with Stata 
analysis and graphing code to generate each chart.

Once youâ€™ve identified analyses that you want to try to replicate or modify, click the 
"Download" buttons to download Stata code and sample data. You can make changes to the 
charts using the code and sample data, or modify the code to work with your own data. If 
you're familiar with Github, you can click â€œGo to Repositoryâ€ and clone the entire 
College-Going Pathways repository to your own computer. Go to the Participate page to read 
about more ways to engage with the OpenSDP community.

### About the Data

The data visualizations in the College-Going Pathways series use a synthetically 
generated college-going analysis sample data file which has one record per student. Each 
high school student is assigned to a ninth-grade cohort, and each student record includes 
demographic and program participation information, annual GPA and on-track status, high 
school graduation outcomes, and college enrollment information. The Connect guide (coming 
soon) will provide guidance and example code which will help you build a college-going 
analysis file using data from your own school system.

#### Loading the OpenSDP Dataset

This guide takes advantage of the OpenSDP synthetic dataset. 


```r
library(tidyverse) # main suite of R packages to ease data analysis
library(magrittr) # allows for some easier pipelines of data
library(tidyr) #
library(ggplot2) # to plot
library(scales) # to format
library(grid)
library(gridExtra) # to plot
# Read in some R functions that are convenience wrappers
source("../R/functions.R")
pkgTest("devtools")
pkgTest("OpenSDPsynthR")
```



### About the Analyses

High school graduation is a critical step to higher education. Understanding
trends and variations in high school completion rates across schools and student
subgroups is essential. These analyses reveal the extent to which high schools
may differentially influence student trajectories towards high school
completion. After identifying these high schools, you may conduct deeper
analyses on your own to explore what drives these outcomes. 

### Sample Restrictions
One of the most important decisions in running each analysis is 
defining the sample. Each analysis corresponds to a different part of the education 
pipeline and as a result requires different cohorts of students.

If you are using the synthetic data we have provided, the sample restrictions have been 
predefined and are included below. If you run this code using your own agency data, 
change the sample restrictions based on your data. Note that you will have to run these 
sample restrictions at the beginning of your do file so they will feed into the rest of 
your code.



```r
# Read in global variables for sample restriction
# Agency name
agency_name <- "Agency"

# Ninth grade cohorts you can observe persisting to the second year of college
chrt_ninth_begin_persist_yr2 = 2004
chrt_ninth_end_persist_yr2 = 2006

# Ninth grade cohorts you can observe graduating high school on time
chrt_ninth_begin_grad = 2004
chrt_ninth_end_grad = 2006

# Ninth grade cohorts you can observe graduating high school one year late
chrt_ninth_begin_grad_late = 2004
chrt_ninth_end_grad_late = 2006

# High school graduation cohorts you can observe enrolling in college the fall after graduation
chrt_grad_begin = 2008
chrt_grad_end = 2010

# High school graduation cohorts you can observe enrolling in college two years after hs graduation
chrt_grad_begin_delayed = 2008
chrt_grad_end_delayed = 2010

# In RStudio these variables will appear in the Environment pane under "Values"
```

Based on the sample data, you will have three cohorts (sometimes only 
two) for analysis. If you are using your own agency data, you may decide 
to aggregate results for more or fewer cohorts to report your results. This 
decision depends on 1) how much historical data you have available and 
2) what balance to strike between reliability and averaging
away information on recent trends. We suggest you average results for the last 
three cohorts to take advantage of larger sample sizes and improve reliability. 
However, if you have data for more than three cohorts, you may decide to not 
average data out for fear of losing information about trends and recent changes 
in your agency.

### Giving Feedback on this Guide
 
This guide is an open-source document hosted on Github and generated using the Stata
Webdoc package. We welcome feedback, corrections, additions, and updates. Please
visit the OpenSDP college-going pathways repository to read our contributor guidelines.

## Analyses

### High School Completion Rates by School

**Purpose:** This analysis explores variation in high school completion rates 
across high schools in the system for both on-time and late high school 
graduates.

**Required Analysis File Variables:**

- `sid`
- `chrt_ninth`
- `hs_diploma`
- `ontime_grad`
- `late_grad`
- `first_hs_code`
- `first_hs_name`

**Analysis-Specific Sample Restrictions:** Keep students in ninth
grade cohorts you can observe graduating high school one year late

**Ask Yourself**

- Does the ordering of high school completion rates coincide with beliefs key 
stakeholders have about these high schools?

- Which high schools have the highest and lowest completion rates? Do you know 
why?

**Analytic Technique:** Calculate the proportion of students who complete high 
school by school.



```r
# // Step 1: Keep students in ninth grade cohorts you can observe 
#  graduating high school one year late

plotdf <- filter(cgdata, chrt_ninth >= chrt_ninth_begin_grad_late & 
                   chrt_ninth <= chrt_ninth_end_grad_late) 
```



```r
# // Step 2: Obtain agency level high school and school level graduation 
#  rates

schoolLevel <- bind_rows(
  plotdf %>% group_by(first_hs_name) %>% 
    summarize(ontime_grad = mean(ontime_grad, na.rm=TRUE), 
              late_grad = mean(late_grad, na.rm=TRUE), 
              count = n()), 
  plotdf %>% ungroup %>%  
    summarize(first_hs_name = "Agency AVERAGE",
              ontime_grad = mean(ontime_grad, na.rm=TRUE), 
              late_grad = mean(late_grad, na.rm=TRUE), 
              count = n())
)

#  // Step 3: Reshape the data wide
schoolLevel <- schoolLevel %>% gather(key = outcome, 
                             value = measure, -count, -first_hs_name)
schoolLevel$first_hs_name <- gsub(" High School", "", schoolLevel$first_hs_name)

#  // Step 4: Recode variables for plotting

schoolLevel$outcome[schoolLevel$outcome == "ontime_grad"] <- "On-Time HS Graduate"
schoolLevel$outcome[schoolLevel$outcome == "late_grad"] <- "Graduate in 4+ Years"
# Figure caption
figureCaption <- paste0("Sample: ", chrt_ninth_begin_grad_late -1, "-", 
                        chrt_ninth_begin_grad_late, " to ", 
                        chrt_ninth_end_grad_late -1, "-", chrt_ninth_end_grad_late, 
                        " ", agency_name, " first-time ninth graders. \n", 
                        "All data from ", agency_name, " administrative records.")
```



```r
#  // Step 5: Plot
ggplot(schoolLevel, aes(x = reorder(first_hs_name, measure), y = measure, 
                        group = first_hs_name, fill = outcome)) + 
  geom_bar(aes(fill = outcome), stat = 'identity') + 
  geom_text(aes(label = round(100 * measure, 0)), 
            position = position_stack(vjust = 0.8)) + 
  theme_bw() + theme(panel.grid = element_blank(), axis.ticks.x = element_blank()) +
  scale_y_continuous(limits = c(0, 1), label = percent, 
                    name = "Percent of Ninth Graders") + 
  scale_fill_brewer(name = "", 
                    type = "qual", palette = 7) + 
  theme(axis.text.x = element_text(color = "black", angle = 30, vjust = 0.5), 
        legend.position = c(0.15, 0.825)) +
  labs(title = "High School Graduation Rates by High School", 
       x = "",
       caption = figureCaption)
```

<img src="../figure/C_C1plot-1.png" style="display: block; margin: auto;" />

### High School Completion Rates by Average 8th Grade Achievement

**Purpose:**  This analysis examines the relationship between academic 
achievement at high school entry and high school completion rates. This 
analysis is useful to identify high schools that beat the odds. High schools 
with similar incoming student achievement profiles but different high school 
graduation rates.


**Required Analysis File Variables:**

- `sid`
- `chrt_ninth`
- `test_math_8_std`
- `hs_diploma`
- `first_hs_code`
- `first_hs_name`

**Analysis-Specific Sample Restrictions:** 

- Keep students in ninth grade cohorts you can observe
graduating high school AND have non-missing eighth grade
math scores.
- Drop any high schools with less than 20 students enrolled in
ninth grade across the cohorts.

**Ask Yourself** 

What might explain differences in high school graduation rates for high schools 
with similar incoming achievement? What might explain differences in incoming
achievement for high schools with similar graduation rates?

**Possible Next Steps or Action Plans:** If substantial variation exists after
controlling for average student achievement at high school entry, think about 
how to share this information across schools. To explore mechanisms that drive
school-level differences in high school completion rates, replicate this 
analysis where the x-axis is a middle school at-risk index (e.g. an index that 
accounts for whether students failed a core class, were chronically absent, and 
other information predictive of student achievement in high school) in place of 
8th grade test scores.

**Analytic Technique:** Bivariate scatterplot of school-level average student 
test scores and high school completion rates.


```r
# // Step 1: Keep students in ninth grade cohorts you can observe graduating 
# high school AND have non-missing eighth grade math scores

plotdf <- filter(cgdata, chrt_ninth >= chrt_ninth_begin_grad & 
                   chrt_ninth <= chrt_ninth_end_grad) %>% 
  filter(!is.na(test_math_8_std))
```



```r
# // Step 2: Obtain agency and school level completion and prior achievement 
#  rates

schoolLevel <- bind_rows(
  plotdf %>% group_by(first_hs_name) %>% 
    summarize(ontime_grad = mean(ontime_grad, na.rm=TRUE), 
              std_score = mean(test_math_8_std, na.rm=TRUE), 
              count = n()), 
  plotdf %>% ungroup %>%  
    summarize(first_hs_name = "Agency AVERAGE",
              ontime_grad = mean(ontime_grad, na.rm=TRUE), 
              std_score = mean(test_math_8_std, na.rm=TRUE), 
              count = n())
  )

# // Step 3: Recode HS Name for display
schoolLevel$first_hs_name <- gsub(" High School", "", schoolLevel$first_hs_name)
# Figure caption
figureCaption <- paste0("Sample: ", chrt_ninth_begin_grad_late -1, "-", 
                        chrt_ninth_begin_grad_late, " to ", 
                        chrt_ninth_end_grad_late -1, "-", chrt_ninth_end_grad_late, 
                        " ", agency_name, 
                        " first-time ninth graders with eigth grade math scores. \n", 
                        "Data from ", agency_name, " administrative records.")
```


```r
# // Step 4: Plot
ggplot(schoolLevel[schoolLevel$first_hs_name != "Agency AVERAGE", ], 
       aes(x = std_score, y = ontime_grad)) + 
  geom_vline(xintercept = as.numeric(schoolLevel[schoolLevel$first_hs_name == 
                                                   "Agency AVERAGE", "std_score"]), 
               linetype = 4, color = I("goldenrod"), size = 1.1) +
  geom_hline(yintercept = as.numeric(schoolLevel[schoolLevel$first_hs_name == 
                                                   "Agency AVERAGE", "ontime_grad"]), 
               linetype = 4, color = I("purple"), size = 1.1) +
  geom_point(size = I(2)) + 
  theme_bw() + theme(panel.grid = element_blank()) +
  coord_cartesian() +
  annotate(geom = "text", x = -.85, y = 0.025, 
           label = "Below average math scores & \n below average graduation rates", 
           size = I(2.5)) +
  annotate(geom = "text", x = .85, y = 0.025, 
           label = "Above average math scores & \n below average graduation rates", 
           size = I(2.5)) +
  annotate(geom = "text", x = .85, y = 0.975, 
           label = "Above average math scores & \n above average graduation rates", 
           size = I(2.5)) +
  annotate(geom = "text", x = -.85, y = 0.975, 
           label = "Below average math scores & \n above average graduation rates", 
           size = I(2.5)) + 
  annotate(geom = "text", x = .205, y = 0.025, 
           label = "Agency Average \n Test Score", 
           size = I(2.5), color = I("goldenrod")) + 
  annotate(geom = "text", x = .85, y = 0.61, 
           label = "Agency Average Graduation Rate", 
           size = I(2.5)) + 
  scale_x_continuous(limits = c(-1, 1), breaks = seq(-1, 1, 0.2)) + 
  scale_y_continuous(limits = c(0, 1), label = percent, 
                     name = "Percent of Ninth Graders", breaks = seq(0, 1, 0.1)) + 
  geom_text(aes(label = first_hs_name), nudge_y = 0.065, vjust = "top", size = I(4), 
            nudge_x = 0.01) +
  labs(title = "High School Graduation Rates by High School", 
       x = "Average 8th Grade Math Standardized Score",
       subtitle = "By Student Achievement Profile Upon High School Entry",
       caption = figureCaption)
```

<img src="../figure/C_C2plot-1.png" style="display: block; margin: auto;" />

### High School Completion Rates by 8th Grade Achievement Quartiles


**Purpose:** This analysis examines variation in completion rates for high 
schools  among students with 8th grade test scores in the same quartile. The 
analysis is useful to explore high school completion rates across schools with 
students in the same quartile or range of achievement. Each high school is 
repeated as a blue bar in each quartile.


**Required Analysis File Variables:**

- `sid`
- `chrt_ninth`
- `test_math_8_std`
- `hs_diploma`
- `first_hs_code`
- `first_hs_name`

**Analysis-Specific Sample Restrictions:**

- Keep students in ninth grade cohorts you can observe
graduating high school AND have non-missing eighth grade
math scores.
- Drop high schools with less than 20 students in each quartile
enrolled in ninth grade across the cohorts.

**Ask Yourself**

- Looking at the average in each quartile (orange bars), how do 8th grade test 
scores relate to high school graduation?
- For each quartile of 8th grade test scores (the blue bars), how do graduation 
rates vary by high school? What is the difference between top and bottom high 
schools in each quartile?

**Possible Next Steps or Action Plans:** Highlight comparison schools to show 
variation across quartiles and explore reasons why students at different 
schools, but with similar academic profiles at high school entry, are more or 
less likely to graduate.

**Analytic Technique:** Calculate the proportion of students, by high school, who 
complete high school and an 8th grade test score quartile for each. 



```r
# // Step 1: Keep students in ninth grade cohorts you can observe graduating 
# high school AND have non-missing eighth grade math scores

plotdf <- filter(cgdata, chrt_ninth >= chrt_ninth_begin_grad & 
                   chrt_ninth <= chrt_ninth_end_grad) %>% 
  filter(!is.na(test_math_8_std))
```


```r
# // Step 2: btain the agency-level and school level high school graduation 
# rates by test score quartile

schoolLevel <- bind_rows(
  plotdf %>% group_by(qrt_8_math, first_hs_name) %>% 
    summarize(ontime_grad = mean(ontime_grad, na.rm=TRUE), 
              count = n()), 
  plotdf %>% ungroup %>% 
    summarize(first_hs_name = "Agency AVERAGE",
              qrt_8_math = 1,
              ontime_grad = mean(ontime_grad, na.rm=TRUE), 
              count = n())
)

# // Step 3: Recode HS Name for display
schoolLevel$first_hs_name <- gsub(" High School", "", schoolLevel$first_hs_name)
```


```r
# //  Step 4: Create plot template
#  Load library for arranging multiple plots into one
library(gridExtra); library(grid)

# Create a plot template that you can drop different data elements into
p2 <-  ggplot(schoolLevel[schoolLevel$qrt_8_math == 2 & 
                       schoolLevel$first_hs_name != "Agency AVERAGE", ], 
       aes(x = reorder(first_hs_name, ontime_grad), y = ontime_grad)) +
      geom_hline(yintercept =
                   as.numeric(schoolLevel$ontime_grad[schoolLevel$first_hs_name ==
                                                        "Agency AVERAGE"]),
               linetype = 2, size = I(1.1)) +
  geom_bar(stat = "identity", fill = "lightsteelblue4", color = I("black")) + 
    scale_y_continuous(limits = c(0,1), breaks = seq(0, 1, 0.2), 
                       expand = c(0, 0), label = percent) + 
    theme_bw() + 
    theme(panel.grid = element_blank(),
                       axis.text.x = element_text(angle = 30, color = "black", 
                                                  vjust = 0.5, size = 6),
          axis.ticks = element_blank(),
          axis.text.y = element_blank(), axis.line.y = element_blank(), 
          panel.border = element_blank()) +
    labs(y = "", x = "") + 
    geom_text(aes(label = round(ontime_grad * 100, 0)), vjust = -0.2) +
    expand_limits(y = 0, x = 0)

# Step 5: Create four plots, three using the template above and with the legend, 
# put these in a list

grobList <- list(
  ggplot(schoolLevel[schoolLevel$qrt_8_math == 1 & 
                       schoolLevel$first_hs_name != "Agency AVERAGE", ], 
       aes(x = reorder(first_hs_name, ontime_grad), y = ontime_grad)) + 
        geom_hline(yintercept =
                     schoolLevel$ontime_grad[schoolLevel$first_hs_name ==
                                                          "Agency AVERAGE"],
                   linetype = 2, size = I(1.1)) +
  geom_bar(stat = "identity", fill = "lightsteelblue4", color = I("black")) + 
    scale_y_continuous(limits = c(0,1), breaks = seq(0, 1, 0.2), 
                       expand = c(0, 0), label = percent) + 
    theme_bw() + 
    theme(panel.grid = element_blank(),
                       axis.text.x = element_text(angle = 30, size = 6,
                                                  color = "black", vjust = 0.5),
          axis.line.y = element_line(),
          axis.ticks.x = element_blank(),panel.border = element_blank()) +
    labs(y = "Percent of Ninth Graders", x = "") + 
    annotate(geom = "text", x = 5, 
    y = 0.025 + schoolLevel$ontime_grad[schoolLevel$first_hs_name == "Agency AVERAGE"], 
          label = "Agency Average") +
    geom_text(aes(label = round(ontime_grad * 100, 0)), vjust = -0.2) +
    expand_limits(y = 0, x = 0),
  p2, 
  # Use the %+% argument to pass a different data element to the p2 plot template
  p2 %+% schoolLevel[schoolLevel$qrt_8_math == 3 & 
                       schoolLevel$first_hs_name != "Agency AVERAGE", ],
  p2 %+% schoolLevel[schoolLevel$qrt_8_math == 4 & 
                       schoolLevel$first_hs_name != "Agency AVERAGE", ]
)

# Step 6: Apply a label to the bottom of each plot object
wrap <- mapply(arrangeGrob, grobList, 
               bottom = c("Bottom Quartile", "2nd Quartile", 
                          "3rd Quartile", "Top Quartile"), 
               SIMPLIFY=FALSE)

# Step 7: Draw the plot
grid.arrange(grobs=wrap, nrow=1, 
    top = "On-Time High School Graduation Rates \n by Prior Student Achievement", 
    bottom = textGrob(
      label = figureCaption, 
      gp=gpar(fontsize=10,lineheight=1), just = 1, x = unit(0.99, "npc")))
```

<img src="../figure/C_C3plot-1.png" style="display: block; margin: auto;" />

### Racial Gaps in Completion Overall and by 8th Grade Achievement Quartiles

**Purpose:** This analysis displays an overall graduation gap by race, and 
examines the extent to which this gap is explained by average differences in 
academic achievement between racial sub-groups at high school entry. The 
analysis is useful to diagnose whether racial gaps in high school result from 
persistent academic achievement gaps that emerge in early grades, or if other 
factors unique to the high school experience drive high school completion rate
differences by race.

**Required Analysis File Variables:**

- `sid`
- `chrt_ninth`
- `test_math_8_std`
- `hs_diploma`
- `first_hs_code`
- `first_hs_name`

**Analysis-Specific Sample Restrictions:**

- Keep students in ninth grade cohorts you can observe
graduating high school AND have non-missing eighth grade
math scores.
- Drop any race/ethnic sub-groups with at least 20 students
in each quartile (for the second graph). You may further
restrict the sample to only include students from the most
representative racial/ethnic sub-groups in your agency.

**Ask Yourself**

- How do racial gaps in graduation rates change after prior achievement is 
accounted for? Do these gaps change for different prior achievement quartiles?

**Possible Next Steps or Action Plans:** Repeat analyses for only students that 
qualify for free or reduced price lunch (FRPL) to explore if racial gaps are 
better explained by disparities in prior academic achievement and family 
socioeconomic status.

**Analytic Technique:** Calculate the proportion of students who complete high 
school by race/ethnicity overall, and by race/ethnicity and 8th grade test 
score quartile.


```r
# // Step 1: Keep students in ninth grade cohorts you can observe graduating 
# high school AND have non-missing eighth grade math scores

plotdf <- filter(cgdata, chrt_ninth >= chrt_ninth_begin_grad & 
                   chrt_ninth <= chrt_ninth_end_grad) %>% 
  filter(!is.na(test_math_8_std))

plotdf$race <- as.factor(plotdf$race_ethnicity)
```



```r
# // Step 2: Obtain average on-time completion by race for agency
plotOne <- plotdf %>% group_by(race) %>% 
  summarize(ontimeGrad = mean(ontime_grad, na.rm = TRUE), 
            N = n()) %>% ungroup %>% 
  filter(N > 100)

# // Step 3: Obtain average on-time completion by race for agency by 
# math score quartile
plotTwo <- plotdf %>% group_by(race, qrt_8_math) %>% 
  summarize(ontimeGrad = mean(ontime_grad, na.rm=TRUE), 
            N = n()) %>% ungroup %>% 
  filter(race %in% c("Black", "Asian", "Hispanic", "White"))

# // Step 4: Make labels
plotTwo$qrt_label <- NA
plotTwo$qrt_label[plotTwo$qrt_8_math == 1] <- "Bottom Quartile"
plotTwo$qrt_label[plotTwo$qrt_8_math == 2] <- "2nd Quartile"
plotTwo$qrt_label[plotTwo$qrt_8_math == 3] <- "3rd Quartile"
plotTwo$qrt_label[plotTwo$qrt_8_math == 4] <- "Top Quartile"

plotTwo$qrt_label <- factor(plotTwo$qrt_label, 
                            ordered = TRUE, 
                            levels = c("Bottom Quartile", 
                                       "2nd Quartile", 
                                       "3rd Quartile", 
                                       "Top Quartile"))
```


```r
# // Step 5: Plot
ggplot(plotOne, aes( x= reorder(race, -N), y = ontimeGrad, fill = race)) + 
  geom_bar(stat = "identity", color = I("black")) + 
  scale_fill_brewer(type = "qual", palette = 4, guide = "none") + 
  geom_text(aes(label = round(ontimeGrad*100, 0)), vjust = -0.4) + 
  theme_bw() + theme(panel.grid = element_blank(), panel.border = element_blank(),
                     axis.line = element_line()) + 
  scale_y_continuous(limits = c(0, 1), expand = c(0, 0), 
                     breaks = seq(0, 1, 0.2), name = "Percent of Ninth Graders", 
                     label = percent) + 
  labs(x = "", title = "On-Time High School Graduation Rates",
       subtitle = "by Race and Eigth Grade Math Quartile", 
       caption = figureCaption)
```

<img src="../figure/C_C4plot-1.png" style="display: block; margin: auto;" />

```r
ggplot(plotTwo, aes( x = qrt_label, 
                     group= reorder(race, -N), y = ontimeGrad, fill = race)) + 
  geom_bar(stat = "identity", color = I("black"), position = "dodge") + 
  scale_fill_brewer(type = "qual", palette = 4) + 
  guides(fill = guide_legend(nrow=1, title = "", keywidth = 2)) +
  geom_text(aes(label = round(ontimeGrad*100, 0)), position = position_dodge(0.9), vjust = -0.3) + 
  theme_bw() + theme(panel.grid = element_blank(), panel.border = element_blank(),
                     axis.line = element_line(), legend.position = "top") + 
  scale_y_continuous(limits = c(0, 1), expand = c(0, 0), 
                     breaks = seq(0, 1, 0.2), name = "Percent of Ninth Graders", 
                     label = percent) + 
  labs(x = "", title = "On-Time High School Graduation Rates",
       subtitle = "by Race and Eigth Grade Math Quartile", 
       caption = figureCaption)
```

<img src="../figure/C_C4plot-2.png" style="display: block; margin: auto;" />

### Enrollment Outcome in Year Four By On-Track Status At the End of Ninth Grade


**Purpose:** This analysis explores how strongly student performance in ninth grade 
predicts high school graduation three years later. Building upon our analysis 
of the relationship between student performance in ninth and tenth grade, the 
analysis assesses the utility of using course-level performance data early in 
studentsâ€™ high school careers to assess risk of non-completion, and target 
students in need of academic and/or socio-emotional support.


**Required Analysis File Variables:**

- `sid`
- `chrt_ninth`
- `ontrack_grad_hs_sample*`
- `ontrack_endyr1*`
- `cum_gpa_yr1*`
- `status_after_yr4*`

**Analysis-Specific Sample Restrictions:**
- Keep students in ninth grade cohorts you can observe
graduating high school AND are part of the on-track sample
(attended the first semester of ninth grade and never transferred
into or out of the system).

**Ask Yourself**
- How does on-track status at the end of ninth grade relate to high school 
completion status at the end of four years?

**Possible Next Steps or Action Plans:** Repeat analyses for only students that 
qualify for free or reduced price lunch (FRPL) to explore whether racial gaps 
are better explained by disparities in prior academic achievement and family
socioeconomic status.

**Analytic Technique:** Calculate the proportion of students who graduate high 
school within four years, dropout, remain enrolled in high school for a fifth 
year, etc. based on on-track status upon completion of ninth grade.


```r
# // Step 1: Keep students in ninth grade cohorts you can observe graduating 
# high school AND have non-missing eighth grade math scores AND are part of 
# the on-track sample
plotdf <- filter(cgdata, chrt_ninth >= chrt_ninth_begin_grad & 
                   chrt_ninth <= chrt_ninth_end_grad)  %>% 
  filter(!is.na(cum_gpa_yr1)) %>% filter(!is.na(status_after_yr4)) %>%
  filter(ontrack_sample == 1)

# // Step 2: Recode status variables
plotdf$statusVar <- plotdf$status_after_yr4

# // Step 3: Generate on-track indicators that take into account students' 
# GPA upon completion of their first year in high school
plotdf$ontrackStatus <- NA
plotdf$ontrackStatus[plotdf$ontrack_endyr1 == 0] <- "Off-Track to Graduate"
plotdf$ontrackStatus[plotdf$ontrack_endyr1 == 1 & plotdf$cum_gpa_yr1 < 3 &
                       !is.na(plotdf$cum_gpa_yr1)] <- "On-Track, GPA < 3.0"
plotdf$ontrackStatus[plotdf$ontrack_endyr1 == 1 & plotdf$cum_gpa_yr1 >= 3 &
                       !is.na(plotdf$cum_gpa_yr1)] <- "On-Track, GPA >= 3.0"
```


```r
# // Step 4: Create average outcomes by on-track status at the end of ninth grade
plotOne <- plotdf %>% group_by(ontrackStatus, statusVar) %>% 
  summarize(count = n()) %>% ungroup %>%
  group_by(ontrackStatus) %>% 
  mutate(sum = sum(count))

# // Step 5: Recode negative values for dropped out and disappeared

plotOne$count[plotOne$statusVar == "Dropped Out"] <- 
  -plotOne$count[plotOne$statusVar == "Dropped Out"]
plotOne$count[plotOne$statusVar == "Disappeared"] <- 
  -plotOne$count[plotOne$statusVar == "Disappeared"]  
plotOne$statusVar <- ordered(plotOne$statusVar, 
                             c("Graduated On-Time", "Enrolled, Not Graduated", 
                               "Disappeared", "Dropped Out"))

# Figure caption
figureCaption <- paste0("Sample: ", chrt_ninth_begin_grad_late -1, "-", 
                        chrt_ninth_begin_grad_late, " to ", 
                        chrt_ninth_end_grad_late -1, "-", chrt_ninth_end_grad_late, 
                        " ", agency_name, " first-time ninth graders. \n",
                        "Students who transferred into or out of the agency are excluded.\n",
                        "All data from ", agency_name, " administrative records.")
```


```r
ggplot(plotOne, aes(x = ontrackStatus, y = count/sum, fill = statusVar, 
                    group = statusVar)) + 
  geom_bar(stat="identity") + 
  geom_text(aes(label = round((count/sum) * 100, digits = 0))) + 
  scale_fill_brewer(type = "div", palette=7, direction = -1) + 
  geom_hline(yintercept = 0, size =1.1) + 
  scale_y_continuous(limits = c(-0.6, 1), label = percent, 
                     breaks = seq(-0.6, 1, 0.2), name = "Percent of Students") + 
  labs(x = "Ninth Grade On-Track Status", fill = "Status After Year Four", 
       title = "Enrollment Status After Four Years in High School", 
       subtitle = "By Course Credits and GPA after First Year of High School", 
       caption =figureCaption) + 
  theme_classic() + theme(legend.position = c(0.8, 0.8),
                          axis.text = element_text(color = "black")) 
```

<img src="../figure/C_C5plot-1.png" style="display: block; margin: auto;" />

#### *This guide was originally created by the Strategic Data Project.*
