Jared Knowles, Lauren Dahlin  
April 7, 2017  

# College Persistence
*College-Going Pathways*
*R Version*

## Getting Started



<div class="navbar navbar-default navbar-fixed-top" id="logo">
<div class="container">
<img src="..\img\open_sdp_logo_red.png" style="display: block; margin: 0 auto; height: 115px;">
</div>
</div>

### Using this Guide

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

For many high school graduates, college enrollment is just the first of many 
hurdles on the road to postsecondary success. While considerable attention has 
been paid to challenges that surround college preparedness, access, and 
enrollment, only recently has conversation expanded to consider barriers to 
degree completion. These barriers must be understood and addressed at both the 
secondary and postsecondary levels for college attainment rates to increase. In 
the last section of the education pipeline, you examine patterns of persistence 
to the second year of college to identify early indications of student progress 
towards degree attainment.


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

## Analyses

### Persistence Rates to the Second Year of College by High School

**Purpose:** Initial enrollment decisions can dramatically affect higher 
education trajectories and the likelihood of degree attainment. This analysis 
provides a snapshot of persistence to the second year of college by examining
persistence rates across high schools in the system. The analysis illuminates
differences in persistence by level of college first attended (two-year vs. 
four-year). Given another year of sample data, the analysis could also be 
conducted by time of initial entry (seamless vs. delayed enrollment). 

**Required Analysis File Variables:**

- `sid`
- `enrl_1oct_grad_yr1_any`
- `enrl_1oct_grad_yr1_4yr`
- `enrl_1oct_grad_yr1_2yr`
- `enrl_grad_persist_any`
- `enrl_grad_persist_4yr`
- `enrl_grad_persist_2yr`
- `last_hs_code`
- `last_hs_name`
- `enrl_ever_w2_grad_any`

**Analysis-Specific Sample Restrictions:**

- Keep students in high school graduation cohorts you can
observe enrolling in college the fall after graduation
- Keep only graduates who received regular or advanced diplomas
(i.e. exclude students who received SPED diplomas
and other certificates).
- Drop high schools with less than 20 students in the sample.

**Ask Yourself**

- How does college persistence for enrollers at 2-year colleges compare to 
enrollers at 4-year colleges? Given another year of sample data, how does 
college persistence for seamless enrollers compare to delayed enrollers?

**Possible Next Steps or Action Plans:** Consider establishing MOUs with local 
community colleges to obtain detailed data on graduatesâ€™ postsecondary pursuits 
at two-year colleges (Course enrollment and transcript data) allowing agencies 
to explore persistence rates by assignment to remediation coursework.

**Analytic Technique:** Calculate the proportion of students who persist to the 
second year of college by the high school those students first attended.



```r
# // Step 1: Keep students in high school graduation cohorts you can observe 
# enrolling in college the fall after graduation

plotdf <- cgdata %>% filter(chrt_grad >= chrt_grad_begin & 
                              chrt_grad <= chrt_grad_end) %>% 
  select(sid, chrt_grad, enrl_1oct_grad_yr1_2yr, enrl_1oct_grad_yr1_4yr,
         enrl_1oct_grad_yr1_any, enrl_grad_persist_any, 
         enrl_grad_persist_2yr, enrl_grad_persist_4yr, last_hs_name, 
         enrl_ever_w2_grad_any) 

# // Step 2: Rename and recode for simplicity
plotdf$groupVar <- NA
plotdf$groupVar[plotdf$enrl_1oct_grad_yr1_2yr == 1] <- "2-year College"
plotdf$groupVar[plotdf$enrl_1oct_grad_yr1_4yr == 1] <- "4-year College"
```


```r
# // Step 3: Obtain the agency-level average for persistence and enrollment
agencyData <- plotdf %>% group_by(groupVar) %>%
  summarize(persistCount = sum(enrl_grad_persist_any, na.rm=TRUE),
            totalCount = n()) %>% 
  ungroup %>% 
  mutate(total = sum(persistCount)) %>% 
  mutate(persistRate = persistCount / totalCount, 
         last_hs_name = "Agency AVERAGE")

# // Step 4: Obtain the school-level average for persistence and enrollment
schoolData <- plotdf %>% group_by(groupVar, last_hs_name) %>%
  summarize(persistCount = sum(enrl_grad_persist_any, na.rm=TRUE), 
            totalCount = n()) %>% 
  ungroup %>% group_by(last_hs_name) %>%
  mutate(total = sum(persistCount)) %>% 
  mutate(persistRate = persistCount / totalCount)

# Combine for chart
chartData <- bind_rows(agencyData, schoolData)
# // Step 5: Recode variables for plotting 
chartData$last_hs_name <- gsub(" High School", "", chartData$last_hs_name)

# // STep 6: Filter rows out with missing values or small cell sizes
chartData <- na.omit(chartData)
chartData <- filter(chartData, totalCount > 20)


# // Step 7: Calculate rank for plot order
# Make ranks the same for 2 and 4 year colleges
chartData <- chartData %>% group_by(last_hs_name) %>% 
  mutate(order = mean(persistRate)) %>% 
  ungroup() %>% 
  mutate(order = min_rank(order)) %>% 
  arrange(last_hs_name)

# Convert to a factor and order for ggplot purposes
chartData$groupVar <- factor(chartData$groupVar)
chartData$groupVar <- relevel(chartData$groupVar, ref = "4-year College")
# Figure caption
figureCaption <- paste0("Sample: ", chrt_grad_begin-1, "-", chrt_grad_begin, 
                        " to ",chrt_grad_end-1, "-", chrt_grad_end, " ", 
                        agency_name, " high school graduates. \n", 
                        "Postsecondary enrollment outcomes from NSC matched records. \n", 
                        "All other data from ", agency_name, " administrative records.")
```



```r
# // Step 8: Plot
ggplot(chartData, aes(x = reorder(last_hs_name, -order), 
                                      group = groupVar, 
                      y = persistRate, fill = groupVar, 
                      color = I("black"))) + 
  geom_bar(stat = 'identity', position = 'dodge') +
  geom_text(aes(label = round(persistRate * 100, 0)), 
            position = position_dodge(0.9), vjust = -0.4) +
  scale_y_continuous(limits = c(0, 1), breaks = seq(0, 1, 0.2), 
                     name = "% Of Seamless Enrollers", 
                     expand = c(0,0), label = percent) +
  theme_classic() + 
  guides(fill = guide_legend("", keywidth = 3, nrow = 2)) + 
  theme(axis.text.x = element_text(angle = 30, vjust = 0.5, color = "black"), 
        legend.position = c(0.15, 0.2), axis.ticks.x = element_blank(),
        legend.key = element_rect(color = "black")) +
  scale_fill_brewer(type = "div", palette = 2) + 
  labs(x = "", 
       title = "College Persistence by High School, at Any College", 
       subtitle = "Seamless Enrollers by Type of College", 
       caption = figureCaption)
```

<img src="../figure/E_E1plot-1.png" style="display: block; margin: auto;" />

### Persistence Across Two-Year and Four-Year Colleges

**Purpose:** This analysis provides a snapshot of persistence to the second 
year of college from one type of college to another for different high schools 
in the system. The left analysis charts explores how seamless enrollers in 
4-year colleges either persist at a 4-year or switch to a 2-year. The right 
analysis charts how seamless enrollers in 2-year colleges either persist at a 
2-year or switch to a 4-year.

**Required Analysis File Variables:**

- `sid`
- `enrl_1oct_grad_yr1_any`
- `enrl_1oct_grad_yr1_4yr`
- `enrl_1oct_grad_yr1_2yr`
- `enrl_grad_persist_any`
- `enrl_grad_persist_4yr`
- `enrl_grad_persist_2yr`
- `last_hs_code`
- `last_hs_name`


**Analysis-Specific Sample Restrictions:**

- Keep the three most recent cohorts of graduates for which
persistence in college over four consecutive years can be
reported.
- Keep only graduates enrolled in 4-yr colleges and universities
the fall following high school graduation.
- Keep only graduates for whom cumulative high school GPAs
can be calculated (or obtained from the agency)
- Only include the top six enrolling 4-year colleges
- Only report persistence rates among students falling in each
high school GPA category if the sample includes 25 or more
students

**Ask Yourself**

- How do the rates of persistence or switching differ for seamless enrollers at 
4-year vs. 2-year colleges?

**Possible Next Steps or Action Plans:** Create individual school-level reports 
for administrators and college counselors to communicate which postsecondary
institutions are associated with greater rates of persistence. Additionally, 
conduct similar analyses that include more detailed institutional information
that may be associated with studentsâ€™ prospects of persisting (e.g. cost of 
tuition and room/board, financial aid, etc.).

**Analytic Technique:** Calculate the proportion of 4-yr college-goers who 
persist through four years of college by the postsecondary institution first 
attended and cumulative high school GPA category.


```r
# // Step 1: Keep students in high school graduation cohorts you can observe 
# enrolling in college the fall after graduation

plotdf <- cgdata %>% filter(chrt_grad >= chrt_grad_begin & 
                              chrt_grad <= chrt_grad_end) %>% 
  select(sid, chrt_grad, enrl_1oct_grad_yr1_2yr, enrl_1oct_grad_yr1_4yr,
         enrl_1oct_grad_yr1_any, enrl_1oct_grad_yr2_2yr, enrl_1oct_grad_yr2_4yr,
         enrl_1oct_grad_yr2_any, enrl_grad_persist_any, 
         enrl_grad_persist_2yr, enrl_grad_persist_4yr, last_hs_name) 
```


```r
# Clean up missing data for binary recoding
plotdf$enrl_grad_persist_4yr <- zeroNA(plotdf$enrl_grad_persist_4yr)
plotdf$enrl_grad_persist_2yr <- zeroNA(plotdf$enrl_grad_persist_2yr)
plotdf$enrl_1oct_grad_yr1_2yr <- zeroNA(plotdf$enrl_1oct_grad_yr1_2yr)
plotdf$enrl_1oct_grad_yr1_4yr <- zeroNA(plotdf$enrl_1oct_grad_yr1_4yr)

# // Step 2: Create binary outcomes for enrollers who switch from 4-yr to 2-yr, 
# or vice versa and recode variables
plotdf$persist_pattern <- "Not persisting"
plotdf$persist_pattern[plotdf$enrl_grad_persist_4yr == 1 &
                         !is.na(plotdf$chrt_grad)] <- "Persisted at 4-Year College"
plotdf$persist_pattern[plotdf$enrl_grad_persist_2yr ==1 &
                         !is.na(plotdf$chrt_grad)] <- "Persisted at 2-Year College"
plotdf$persist_pattern[plotdf$enrl_1oct_grad_yr1_4yr == 1 & 
                        plotdf$enrl_1oct_grad_yr2_2yr == 1 & 
                         !is.na(plotdf$chrt_grad)] <- "Switched to 2-Year College"
plotdf$persist_pattern[plotdf$enrl_1oct_grad_yr1_2yr == 1 & 
                        plotdf$enrl_1oct_grad_yr2_4yr == 1 & 
                         !is.na(plotdf$chrt_grad)] <- "Switched to 4-Year College"

plotdf$groupVar <- NA
plotdf$groupVar[plotdf$enrl_1oct_grad_yr1_2yr == 1] <- "2-year College"
plotdf$groupVar[plotdf$enrl_1oct_grad_yr1_4yr == 1] <- "4-year College"
# Drop NA
plotdf %<>% filter(!is.na(groupVar))
# // Step 3: Obtain agency and school level average for persistence outcomes
chartData <- plotdf %>% 
  group_by(last_hs_name, groupVar, persist_pattern) %>% 
  summarize(tally = n()) %>% # counts the occurrence persist_pattern
  ungroup %>% 
  group_by(last_hs_name, groupVar) %>% # regroup by grouping variable and school
  mutate(denominator = sum(tally)) %>% # sum all levels of persist_pattern
  mutate(persistRate = tally / denominator) %>% # calculate rate
  filter(persist_pattern != "Not persisting") %>%
  mutate(rankRate = sum(persistRate))

agencyData <- plotdf %>%
  group_by(groupVar, persist_pattern) %>% 
  summarize(tally = n(), 
            last_hs_name = "Agency AVERAGE") %>% 
  ungroup %>% 
  group_by(last_hs_name, groupVar) %>% 
  mutate(denominator = sum(tally)) %>% 
  mutate(persistRate = tally / denominator) %>% 
  filter(persist_pattern != "Not persisting") %>%
  mutate(rankRate = sum(persistRate))

chartData <- bind_rows(chartData, agencyData)

# // Step 4: Recode variable names, sort data frame, and code labels for plot
chartData$last_hs_name <- gsub(" High School", "", chartData$last_hs_name)
chartData$last_hs_name <- gsub(" ", "\n", chartData$last_hs_name)
# chartData %<>% filter(persist_pattern != "Not persisting")
chartData %<>% arrange(persist_pattern)
chartData <- as.data.frame(chartData)
chartData$persist_pattern <- factor(as.character(chartData$persist_pattern), 
                                    ordered = TRUE, 
                                    levels = c("Switched to 4-Year College", 
                                               "Switched to 2-Year College", 
                                               "Persisted at 2-Year College", 
                                               "Persisted at 4-Year College"))
```



```r
# // Step 5: Prepare plot for 2-year colleges
p1 <- ggplot(chartData[chartData$groupVar == "2-year College",], 
       aes(x = reorder(last_hs_name, rankRate), 
           y = persistRate, group = persist_pattern, 
           fill = persist_pattern)) + 
  scale_y_continuous(limits = c(0, 1.25), expand = c(0, 0), 
                     label = percent, breaks = seq(0, 1, 0.2)) + 
  geom_bar(stat = 'identity', position = 'stack', 
           color = I("black")) + 
  geom_text(aes(label = round(persistRate * 100, 0)), 
            position = position_stack(vjust = 0.5)) +
  geom_text(aes(label = round(rankRate * 100, 0), y = rankRate), vjust = -0.7) +
  guides(fill = guide_legend("", keywidth = 2, nrow = 2)) +
  scale_fill_brewer(type = "qual", palette = 1) +
  labs(x = "", y = "Percent of Seamless Enrollers") + 
  theme_classic() + theme(axis.text.x = element_text(angle = 30, vjust = 0.2), 
                          axis.ticks.x = element_blank(), 
                          legend.position = c(0.225, 0.925), 
                          plot.caption = element_text(hjust = 0, size = 7)) + 
  labs(subtitle = "Seamless Enrollers at 2-year Colleges", 
       caption = figureCaption)

# // Step 6: Prepare plot for 4-year colleges by replacing data in plot 
# above with 4 year data
p2 <- p1 %+% chartData[chartData$groupVar == "4-year College",] + 
  labs(subtitle = "Seamless Enrollers at 4-year Colleges")

# // Step 7: Print out plots with labels
grid.arrange(grobs= list(p2, p1), nrow=1, 
             top = "College Persistence by High School")
```

<img src="../figure/E_E2plot-1.png" style="display: block; margin: auto;" />

### Top-Enrolling Colleges/Universities of Agency Graduates

**Purpose:** This analysis reports enrollment and persistence rates among 
top-enrolling two- and four- year institutions attended by graduates. This 
analysis illuminates differences in persistence rates to the second year of 
college among top-enrolling postsecondary institutions. Agency staff that 
advise students during their senior year may find this information useful when 
meeting to weigh college options.

**Required Analysis File Variables:**

- `sid`
- `enrl_1oct_grad_yr1_any`
- `enrl_1oct_grad_yr1_4yr`
- `enrl_1oct_grad_yr1_2yr`
- `enrl_grad_persist_any`
- `enrl_grad_persist_4yr`
- `enrl_grad_persist_2yr`
- `first_college_name_any`
- `first_college_name_2yr`
- `first_college_name_4yr`

** Analysis-Specific Sample Restrictions:**

- Keep only the most recent cohort of seamless college-goers
for which persistence to the second year of college can be
reported
- Only include postsecondary institutions with 25 or more
agency graduates attending.

**Ask Yourself**

- What are the top enrolling 4-year and 2-year colleges or universities in your 
agency? What are the persistence rates at those colleges and universities?

**Analytic Technique:** Calculate the proportion of college-goers attending
top-enrolling 2- and 4-year institutions, as well as the proportion of seamless
enrollers who persist to the second year of any college, by the postsecondary
institution graduates first attended.


```r
# // Step 1: Keep students in high school graduation cohorts you can observe 
# enrolling in college the fall after graduation

plotdf <- cgdata %>% filter(chrt_grad >= chrt_grad_begin & 
                              chrt_grad <= chrt_grad_end) %>% 
  select(sid, chrt_grad, enrl_1oct_grad_yr1_2yr, enrl_1oct_grad_yr1_4yr,
         enrl_1oct_grad_yr1_any, enrl_1oct_grad_yr2_2yr, enrl_1oct_grad_yr2_4yr,
         enrl_1oct_grad_yr2_any, enrl_grad_persist_any, 
         enrl_grad_persist_2yr, enrl_grad_persist_4yr, 
         first_college_name_any, first_college_name_2yr, first_college_name_4yr) 

# // Step 2: Indicate the number of institutions you would like listed
num_inst <- 5
```


```r
# // Step 3: Calculate the number and % of students enrolled in each college 
# the fall after graduation, and the number and % of students persisting, by 
# college type

chart4year <- bind_rows(
  plotdf %>% group_by(first_college_name_4yr) %>% 
  summarize(enrolled = sum(enrl_1oct_grad_yr1_4yr, na.rm=TRUE), 
            persisted = sum(enrl_grad_persist_4yr, na.rm=TRUE)) %>% 
  ungroup %>% 
  mutate(total_enrolled = sum(enrolled)) %>% 
  mutate(perEnroll = round(100 * enrolled/total_enrolled, 1), 
         perPersist = round(100 * persisted/enrolled, 1)),
  plotdf %>% 
  summarize(enrolled = sum(enrl_1oct_grad_yr1_4yr, na.rm=TRUE), 
            persisted = sum(enrl_grad_persist_4yr, na.rm=TRUE), 
            first_college_name_4yr = "All 4-Year Colleges") %>% 
  ungroup %>% 
  mutate(total_enrolled = sum(enrolled)) %>% 
  mutate(perEnroll = round(100 * enrolled/total_enrolled, 1), 
         perPersist = round(100 * persisted/enrolled, 1))
)


chart2year <- bind_rows(plotdf %>% group_by(first_college_name_2yr) %>% 
  summarize(enrolled = sum(enrl_1oct_grad_yr1_2yr, na.rm=TRUE), 
            persisted = sum(enrl_grad_persist_2yr, na.rm=TRUE)) %>% 
  ungroup %>% 
  mutate(total_enrolled = sum(enrolled)) %>% 
  mutate(perEnroll = round(100 * enrolled/total_enrolled, 1), 
         perPersist = round(100 * persisted/enrolled, 1)),
  plotdf %>% 
  summarize(enrolled = sum(enrl_1oct_grad_yr1_2yr, na.rm=TRUE), 
            persisted = sum(enrl_grad_persist_2yr, na.rm=TRUE), 
            first_college_name_2yr = "All 2-Year Colleges") %>% 
  ungroup %>% 
  mutate(total_enrolled = sum(enrolled)) %>% 
  mutate(perEnroll = round(100 * enrolled/total_enrolled, 1), 
         perPersist = round(100 * persisted/enrolled, 1))
)
```


```r
# // Step 4: Create tables
chart4year %>% arrange(-enrolled) %>% 
  select(first_college_name_4yr, enrolled, perEnroll, persisted, perPersist) %>%
  head(num_inst) %>%
    kable(., col.names = c("Name", "Number Enrolled", 
                         "% Enrolled", "Number Persisted", 
                         "% Persisted"))
```



Name                                   Number Enrolled   % Enrolled   Number Persisted   % Persisted
------------------------------------  ----------------  -----------  -----------------  ------------
All 4-Year Colleges                               2162        100.0               2696         124.7
University of North Carolina at ...                955         44.2               1172         122.7
The University of Tampa                            442         20.4                573         129.6
Pennsylvania State University-Wo...                318         14.7                410         128.9
Kenyon College                                     121          5.6                144         119.0

```r
chart2year %>% arrange(-enrolled) %>% 
  select(first_college_name_2yr, enrolled, perEnroll, persisted, perPersist) %>%
  head(num_inst) %>%
  kable(., col.names = c("Name", "Number Enrolled", 
                         "% Enrolled", "Number Persisted", 
                         "% Persisted"))
```



Name                                   Number Enrolled   % Enrolled   Number Persisted   % Persisted
------------------------------------  ----------------  -----------  -----------------  ------------
All 2-Year Colleges                               5417        100.0               6647         122.7
Mt San Jacinto Community College...                925         17.1               1125         121.6
Owens Community College                            921         17.0               1098         119.2
Cape Fear Community College                        580         10.7                729         125.7
Bristol Community College                          576         10.6                706         122.6
