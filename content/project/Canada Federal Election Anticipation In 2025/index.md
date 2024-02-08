---
title: Canada Federal Election Anticipation In 2025
summary: asdasd
tags:
  - Machine Learning
  - Statistics
  - R
  - 
date: '2022-11-18T00:00:00Z'
# # Optional external URL for project (replaces project detail page).
# external_link: ''

image:
  caption: ''
  focal_point: smart
# links:
#   - icon: twitter
#     icon_pack: fab
#     name: Follow
#     url: https://twitter.com/georgecushen
# links:
#   - name:
#     url: ''

url_code: ''
url_pdf: ''
url_slides: ''
url_video: ''

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
slides: ""
type: book
---

``` r
# Loading the packages
knitr::opts_chunk$set(warning = FALSE, message = FALSE)
library(broom)
library(openintro) 
library(tidyverse)
library(cesR)
library(knitr)
library(tibble)
library(ggplot2)
library(ggpubr)
library(flextable)
library(car)
```

## Introduction

After experiencing COVID-19 and continued economic uncertainty in
Canada, it is crucial for Canadians to consider how the 2025 federal
election will affect their life in Canada and how each party will put
Canada on the road to economic recovery\[21\]. The liberal party has won
the 2021 federal election, and the conservative party has the second
highest votes\[21\], so we want to anticipate whether the Liberals
maintain their winning streak or the Conservatives come from behind to
defeat the Liberals in the 2025 federal election.

To help the public have a clear insight into the vote of two parties in
the 2025 federal election, we first develop a research question: ’ which
of the two political parties (Liberal and Conservative) will win the
election based on people’s demographics in Canada? , then we constructed
the statistical models based on the survey dataset about the previous
federal election. Eventually, after analyzing the probability of voting
for a particular party by dividing the entire population into various
cells, we can anticipate the 2025 federal election vote using the entire
population, which is what we call post-stratification in statistics.

The two modelling datasets are the 2019 Canadian Election Study Phone
Survey dataset\[5\] and the 2017 census dataset “General Social Survey
on Social Identity.”\[4\] The 2017 General Social Survey on Social
Identity dataset indicates the demographics of people in Canada. It can
be used to gain inference on statistics of the different kinds of people
living around Canada. The “2019 Canadian Election study Phone Survey
attitudes and opinions of Canadians for the 2019 federal election.

### Hightlight of Hypothesis

Since the Liberal party has won the 2021 federal election, it is
reasonable for us to devised the hypothesis that the Liberal party will
win the 2025 election Specifically, our hypothesis is that the vote of
liberal party will be the higher than others in 2025 election after
conducting statistical prediction.

### Terminologies introduction

``` r
# Generate the table view of important terminologies
all_terms <- data.frame(Term = c("Liberal", "Conservatives", "Census data", "Variable"), 
                        Description = c("Grits, Canada Liberal Party", 
                                        "Tory, PCs, Conservative Party of Canada", 
                                        "Data that studies (social survey) a number of the population", 
                                        "Each variable contains fluctuating value depends on its respondent) (e.g. sex, income, education degree)"
                                        ))
knitr::kable(all_terms, caption = "Terminologies")
```

| Term          | Description                                                                                              |
|:---------|:-------------------------------------------------------------|
| Liberal       | Grits, Canada Liberal Party                                                                              |
| Conservatives | Tory, PCs, Conservative Party of Canada                                                                  |
| Census data   | Data that studies (social survey) a number of the population                                             |
| Variable      | Each variable contains fluctuating value depends on its respondent) (e.g. sex, income, education degree) |

Terminologies

In general, we provided a detailed description of the datasets as well
as data preparation (i.e data cleaning)for future modelling in the
section data. We also gave a comprehensive introduction of the methods
we used for the modelling and poststratification in the methods part.
The results of the model analysis and poststratification prediction were
provided in the results part. Ultimately, we considered the limitation
of our analysis and anticipation

## Data

``` r
# import dataset
census_data <-read_csv("gss_clean.csv")
survey_data <- read_csv("ces2019-phone_clean.csv")
# see the data
head(census_data)
```

    ## # A tibble: 6 × 81
    ##   caseid   age age_first_child age_youngest_child_under_6 total_children
    ##    <dbl> <dbl>           <dbl>                      <dbl>          <dbl>
    ## 1      1  52.7              27                         NA              1
    ## 2      2  51.1              33                         NA              5
    ## 3      3  63.6              40                         NA              5
    ## 4      4  80                56                         NA              1
    ## 5      5  28                NA                         NA              0
    ## 6      6  63                37                         NA              2
    ## # ℹ 76 more variables: age_start_relationship <dbl>,
    ## #   age_at_first_marriage <dbl>, age_at_first_birth <dbl>,
    ## #   distance_between_houses <dbl>, age_youngest_child_returned_work <dbl>,
    ## #   feelings_life <dbl>, sex <chr>, place_birth_canada <chr>,
    ## #   place_birth_father <chr>, place_birth_mother <chr>,
    ## #   place_birth_macro_region <chr>, place_birth_province <chr>,
    ## #   year_arrived_canada <chr>, province <chr>, region <chr>, …

``` r
head(survey_data)
```

    ## # A tibble: 6 × 278
    ##   sample_id survey_end_CES      survey_end_month_CES survey_end_day_CES
    ##       <dbl> <dttm>                             <dbl>              <dbl>
    ## 1        18 2019-09-23 21:48:29                    9                 23
    ## 2        32 2019-09-13 00:02:30                    9                 12
    ## 3        39 2019-09-11 00:02:33                    9                 10
    ## 4        59 2019-10-10 21:20:13                   10                 10
    ## 5        61 2019-09-12 22:28:58                    9                 12
    ## 6        69 2019-09-17 23:56:26                    9                 17
    ## # ℹ 274 more variables: num_attempts_CES <dbl>, interviewer_id_CES <dbl>,
    ## #   interviewer_gender_CES <chr>, language_CES <dbl>, phonetype_CES <dbl>,
    ## #   survey_end_PES <dttm>, survey_end_month_PES <dbl>,
    ## #   survey_end_day_PES <dbl>, num_attempts_PES <dbl>, interviewer_id_PES <dbl>,
    ## #   interviewer_gender_PES <chr>, language_PES <dbl>, phonetype_PES <dbl>,
    ## #   mode_PES <dbl>, phone_type <dbl>, weight_CES <dbl>, weight_PES <dbl>,
    ## #   c1 <dbl>, c2a <dbl>, c3 <dbl>, q1 <dbl>, q2 <dbl>, q3 <dbl>, q4 <dbl>, …

### Dataset

The dataset “Canadian Election Study 2019, Phone Survey”\[5\] was used
for modelling and analysis. The 2019 Canadian Election Study was
conducted to gather Canadians’ attitudes, opinions and characteristics
during and after the 2019 federal election. This data set recorded the
interviews’ responses using phone interviews, and it included 4,021
cases and took survey time, interviewer information and question numbers
as column names. This survey contains 2 components the data set
containing the question numbers and responses; the other is the survey
document with text questions and answer options.

The dataset “Canadian General Social Survey,”\[4\] which Statistics
Canada conducted in 2017, was used to anticipate how the entire
population voted in the 2025 federal election. This was census data
conducted by telephone across ten provinces. This data set provided
respondents’ information and household information, such as age, sex,
education, income, household income, etc. (For the detailed variables,
see Appendix). Since we need to analyze the model and predict the two
parties’ votes based on these two data sets, matching the variables
between survey data and census data was vital and it was described in
the data cleaning.

### Data Cleaning

#### Survey data cleaning

1.  Find the interest of variables and rename these variables into an
    understandable format(See important variables table below)

2.  In order to match the variables and values in the census data, which
    will be helpful for our further analysis, we filtered the step to
    clean the survey data:

    -   We filtered out the value “Other” under the gender variable.
        Since there was only one case from 4,021 cases, it would not
        affect the overall results.
    -   We only wanted to study two major parties that are liberal and
        conservative. Thus, for the variable “Which party you will
        likely vote for”, we filtered out the cases in that people
        responded “Don’t know/ Undecided” and “Refused”. When people who
        choose these responses choose a party to vote for, it also
        affects the total number of votes. So these are also
        indeterminate votes.
    -   For the variable”Household Income”, We filtered out the cases in
        which people responded “Don’t know” and “Refused”.
    -   Lastly, in case there is some error when recording the household
        size response, we only make a limit to choosing households size
        greater than 1.

3.  In order to figure out the proportion of liberal and conservative
    votes in the survey data, we created two new binary variables “vote
    for liberal,” which holds a value of 1 if people chose to vote
    Liberal Party, 0 otherwise; “vote for conservative” which contains a
    value 1 if people decided to vote Conservative Party, 0 otherwise.

4.  As mentioned above, we needed to match the variables and values that
    each variable hold between survey data and census data. (1)We
    divided “household income” into different groups, which are “Less
    than $25,000”, “$25,000 to $49,999”,“$50,000 to $74,999”, “$75,000
    to $99,999”,“$100,000 to $ 124,999” and “$125,000 and more”. (2)We
    put “respondents’ birthplace” into three categories which are “Born
    in Canada”, “Born outside Canada”, and “Don’t know”.(3)We put
    “respondents educational degrees” into 6 categories which are
    corresponded to the categories in census data. The 6 categories are
    “Less than high school diploma or its equivalent”, “High school
    diploma or a high school equivalency certificate”, “College, CEGEP
    or other non-university certificate or di…”, “University certificate
    or diploma below the bachelor’s level”, “Bachelor’s degree
    (e.g. B.A., B.Sc., LL.B.)”, “University certificate, diploma or
    degree above the bachelor’s degree…”.

#### Census dataset cleaning

1.  Because there was no category in the survey data which was
    corresponded to “Trade certificate or diploma” in the census data.
    We filtered out the education degree which was “Trade certificate or
    diploma”.

2.  Since only Canadian residents and permanent residents who are at
    least 18 years old can vote. We filtered out respondents who are
    under 18 years old in both data sets.

3.  Eventually, in order to make the dataset tidy and easy to read, we
    only select the columns that we am interested.

``` r
### Data Cleaning Survey data
survey_data <- 
  survey_data %>% 
  filter(q3 != 3, q69 != -8, q69 != -9, q11 != -8, q11 != -9, q11 != -7) %>% 
  mutate(age = 2019-q2,
         sex = case_when(q3 == 1 ~ "Male",
                         q3 == 2 ~ "Female"),
         vote_liberal = ifelse(q11==1, 1, 0), 
         vote_cons = ifelse(q11== 2, 1, 0),
         vote_ndp = ifelse(q11==3, 1, 0),
         place_birth_canada = case_when(q64 %in% c(1,2) ~ "Born in Canada",
                                        q64 %in% c(3:13) ~ "Born outside Canada",
                                        q64 %in% c(-8,-9) ~ "Don't know"),
         hh_size = q71,
         income_family = case_when(q69 < 25000 ~ "Less than $25,000",
                                   25000 <= q69 & q69 <= 49999 ~"$25,000 to $49,999",
                                   50000 <= q69 & q69 <= 74999 ~"$50,000 to $74,999",
                                   75000 <= q69 & q69 <= 99999 ~"$75,000 to $99,999",
                                   100000 <= q69 & q69 <= 124999 ~"$100,000 to $ 124,999",
                                   q69 >= 125000 ~"$125,000 and more"),
         education = case_when(q61 %in% c(1:4)~"Less than high school diploma or its equivalent",
                               q61 %in% c(5)~"High school diploma or a high school equivalency certificate",
                               q61 %in% c(6,7)~"College, CEGEP or other non-university certificate or di...",
                               q61 %in% c(8)~"University certificate or diploma below the bachelor's level",
                               q61 %in% c(9)~"Bachelor's degree (e.g. B.A., B.Sc., LL.B.)",
                               q61 %in% c(10,11)~"University certificate, diploma or degree above the bach..."),
         province = case_when(q4 == 1 ~ "Newfoundland and Labrador", 
                              q4 == 2 ~ "Prince Edward Island",
                              q4 == 3 ~ "Nova Scotia",
                              q4 == 4 ~ "New Brunswick",
                              q4 == 5 ~ "Quebec",
                              q4 == 6 ~ "Ontario",
                              q4 == 7 ~ "Manitoba",
                              q4 == 8 ~ "Saskatchewan",
                              q4 == 9 ~ "Alberta",
                              q4 == 10 ~ "British Columbia"))%>%
  select(age,sex,vote_liberal,vote_cons,vote_ndp,place_birth_canada,hh_size,
         income_family,education,province)%>%filter(hh_size >= 1, age >= 18) 
# remove the negative value of hh size and people who are at least 18 years old

# Remove NA
survey_data <- na.omit(survey_data) 
survey_data <- survey_data %>% mutate(age_group = case_when(age<25 ~ "Youth", age>= 25 & age <65 ~"Adults", age>= 65 ~"Seniors"))
# see the cleaned version
head(survey_data)
```

    ## # A tibble: 6 × 11
    ##     age sex   vote_liberal vote_cons vote_ndp place_birth_canada hh_size
    ##   <dbl> <chr>        <dbl>     <dbl>    <dbl> <chr>                <dbl>
    ## 1    25 Male             1         0        0 Born in Canada           1
    ## 2    19 Male             0         0        0 Born in Canada           5
    ## 3    35 Male             0         0        1 Born in Canada           1
    ## 4    80 Male             0         0        0 Born in Canada           1
    ## 5    20 Male             0         0        0 Born in Canada           1
    ## 6    24 Male             0         0        0 Born in Canada           2
    ## # ℹ 4 more variables: income_family <chr>, education <chr>, province <chr>,
    ## #   age_group <chr>

``` r
# 2017 census data(GSS), age in 2019 = age in 2017 + 2
census_data <- census_data %>% 
  mutate(age=round(age)+2) %>% 
  select(age,sex,place_birth_canada,hh_size,
         province,income_family,education) %>% 
  filter(education != "Trade certificate or diploma")

# Remove NA from census data
census_data <- na.omit(census_data)
# See the cleaned data
head(census_data)
```

    ## # A tibble: 6 × 7
    ##     age sex    place_birth_canada hh_size province    income_family    education
    ##   <dbl> <chr>  <chr>                <dbl> <chr>       <chr>            <chr>    
    ## 1    55 Female Born in Canada           1 Quebec      $25,000 to $49,… High sch…
    ## 2    66 Female Born in Canada           2 Ontario     $75,000 to $99,… Bachelor…
    ## 3    82 Female Born in Canada           2 Alberta     $100,000 to $ 1… High sch…
    ## 4    30 Male   Born in Canada           2 Quebec      $50,000 to $74,… College,…
    ## 5    65 Female Born in Canada           2 Quebec      $50,000 to $74,… High sch…
    ## 6    61 Female Born in Canada           1 Nova Scotia Less than $25,0… Less tha…

### Important variables

``` r
all_variables <- data.frame(Variable = c("q2","q3", "q11", "q11", "q64", "q71", "q69", "q61", "q4"), 
                            Cleaned_Variable = c("age", "sex", "vote_liberal", "vote_cons", "place_birth", "hh_size", "income_family", "education", "province"), 
                            Type = c("Numerical", "Categorical", "Categorical", "Categorical", "Categorical", "numerical", "Numerical", "Categorical", "Categorical"), 
                            Description = c("Numerical value of repondent's age", "Sex of repondent", "If the respondent voted liberal party, then 1. Otherwise, 0", "If the respondent voted conservative party, then 1. Otherwise, 0", "This variable holds three categories which are Born in Canada, Born outside Canada, and Don't know", "Household size", "Categorized income of the respondent", "Respondent's education degree", "Province of the respondent is currently living")
                            )

names(all_variables)[2] <- "Cleaned Variable"
knitr::kable(all_variables, caption = "Important variables")
```

| Variable | Cleaned Variable | Type        | Description                                                                                        |
|:-----|:---------|:------|:-------------------------------------------------|
| q2       | age              | Numerical   | Numerical value of repondent’s age                                                                 |
| q3       | sex              | Categorical | Sex of repondent                                                                                   |
| q11      | vote_liberal     | Categorical | If the respondent voted liberal party, then 1. Otherwise, 0                                        |
| q11      | vote_cons        | Categorical | If the respondent voted conservative party, then 1. Otherwise, 0                                   |
| q64      | place_birth      | Categorical | This variable holds three categories which are Born in Canada, Born outside Canada, and Don’t know |
| q71      | hh_size          | numerical   | Household size                                                                                     |
| q69      | income_family    | Numerical   | Categorized income of the respondent                                                               |
| q61      | education        | Categorical | Respondent’s education degree                                                                      |
| q4       | province         | Categorical | Province of the respondent is currently living                                                     |

Important variables

## Explotory Data Analysis

### Numerical Summary Table

``` r
# average age
#table
table1 <- survey_data %>% group_by(province) %>% 
  filter(vote_liberal==1) %>% summarise(average_age = sum(age)/n(), sd=sd(age))



table2 <- survey_data %>% group_by(province) %>% 
  filter(vote_cons==1) %>% summarise(average_age = sum(age)/n(), sd= sd(age))

mean_sd <- data.frame(Province = table1$province, table1$average_age, table1$sd, table2$average_age, table2$sd)
names(mean_sd)[2] <- 'Average Age(Liberal Party)'
names(mean_sd)[4] <- 'Average Age(Conservative Party)'
names(mean_sd)[3] <- 'Standard Deviation(Liberal Party)'
names(mean_sd)[5] <- 'Standard Deviation(Conservative Party)'
knitr::kable(mean_sd, caption = "Average age of respondents who voted Liberal Party or Conservative Party")
```

| Province                  | Average Age(Liberal Party) | Standard Deviation(Liberal Party) | Average Age(Conservative Party) | Standard Deviation(Conservative Party) |
|:-----------|------------:|---------------:|--------------:|-----------------:|
| Alberta                   |                   45.50000 |                          17.56823 |                        47.94545 |                               15.23687 |
| British Columbia          |                   52.50000 |                          18.30700 |                        51.92969 |                               16.12363 |
| Manitoba                  |                   54.73333 |                          17.72056 |                        53.43038 |                               15.32359 |
| New Brunswick             |                   52.32432 |                          13.99812 |                        54.00000 |                               16.14001 |
| Newfoundland and Labrador |                   54.76190 |                          15.20881 |                        53.20000 |                               13.76252 |
| Nova Scotia               |                   52.06818 |                          17.57958 |                        53.08824 |                               14.56098 |
| Ontario                   |                   53.29703 |                          16.25759 |                        54.88435 |                               16.21876 |
| Prince Edward Island      |                   59.04444 |                          15.16119 |                        55.36364 |                               15.21680 |
| Quebec                    |                   48.45270 |                          15.16231 |                        47.98551 |                               14.83189 |
| Saskatchewan              |                   51.81818 |                          19.47070 |                        48.77228 |                               16.40846 |

Average age of respondents who voted Liberal Party or Conservative Party

**Description:** The average age and standard deviation of Liberal Party
and Conservative Party voters are displayed in the table below. As can
be seen, there is not a significant age gap between the voters of these
two parties in each province.

``` r
#pie chart voters based on sex
plot_da1 <- survey_data %>% filter(vote_cons == 1)

plot_da2 <- survey_data %>% filter(vote_liberal == 1)


a<-ggplot(plot_da2, aes(x="sex", y=vote_liberal, fill=sex))+
  labs(title="The percentage of male and female Liberal Party voters") +
  theme(plot.title = element_text(hjust = 0))+
  geom_bar(width = 1, stat = "identity") +
  coord_polar("y", start=0) + theme_void()+
  theme(plot.title = element_text(hjust = 0.5))


b<-ggplot(plot_da1, aes(x="sex", y=vote_cons, fill=sex))+
  labs(title=" The percentage of male and female Conservative Party voters" ) +
  theme(plot.title = element_text(hjust = 0))+
  geom_bar(width = 1, stat = "identity") +
  coord_polar("y", start=0) + theme_void()+
  theme(plot.title = element_text(hjust = 0.5))

ggarrange(a,b,nrow = 2,ncol = 1)
```

<img src="Assignment2_files/figure-markdown_github/unnamed-chunk-8-1.png" style="display: block; margin: auto;" />

**Description:** This pie chart shows the percentage of male and female
liberal and conservative party voters separately. From the picture, we
can see that among the conservative party voters, the percentage of male
voters is much higher than the female voters. Among the liberal party
voters, there was less different among the sex of the voters.

``` r
# education and vote
ggplot(data = survey_data, aes(x=education, group= vote_liberal, fill=factor(vote_liberal, 
                                                                                      labels = c("not vote for the Liberal Party", 
                                                                                                 "vote for the Liberal Party")))) +
  geom_bar(position = position_dodge(0.9, preserve = "single")) +
  labs(title = "The distribution of Liberal Party voters 
based on education degree", 
       x="Education Degree", y="Number of people", fill= "Vote Liberal") + 
  scale_fill_brewer(palette = "Dark2") + 
    theme_light() + coord_flip()+theme(plot.title = element_text(hjust = 0))
```

<img src="Assignment2_files/figure-markdown_github/unnamed-chunk-9-1.png" style="display: block; margin: auto;" />

**Description:** The plot shows the education demographic for Liberal
Party voters and non Liberal Party voters. From the plot, we can see
respondents with College, CEGEP or other non-university certificate took
the highest proportion on the non Liberal Party voters. Respondents with
bachelor degree took the highest percentage on the total number of votes
for Liberal Party.

``` r
ggplot(data = survey_data, aes(x=education, group= vote_cons, fill=factor(vote_cons, labels = c("not vote for the Conservative Party", 
                                                                                                         "vote for the Conservative Party")))) +
  geom_bar(position = position_dodge(0.9, preserve = "single")) +
  labs(title = "The distribution of Conservative Party voters 
based on education degree", 
       x="Education Degree", y="Number of people", fill= "Vote Conservative") + 
  scale_fill_brewer(palette = "Dark2") + 
    theme_light()+coord_flip()+theme(plot.title = element_text(hjust = 0))
```

<img src="Assignment2_files/figure-markdown_github/unnamed-chunk-10-1.png" style="display: block; margin: auto;" />

**Description:** The plot shows the education demographic for
Conservative Party voters and non Conservative Party voters. From the
plot, we can see respondents with bachelors degree took the highest
percentage on the non Conservative Party voters. Respondents with
College, CEGEP or other non-university certificate took the highest
percentage on the Conservative Party Voters.The educational backgrounds
of the voters and non-voters of the two parties separately are opposite.

``` r
#income and vote
plot5 <- ggplot(data = survey_data, aes(x=income_family, group= vote_liberal, fill=factor(vote_liberal, 
                                                                                      labels = c("not vote for the Liberal Party", 
                                                                                                 "vote for the Liberal Party")))) +
  geom_bar(position = position_dodge(0.9, preserve = "single")) +
  labs(title = "The distribution of Liberal Party voters 
based on household income", 
       x="Household Income", y="Number of people", fill= "Vote Liberal") + 
  scale_fill_brewer(palette = "Dark2") + 
    theme_light() + coord_flip()
  
plot5 <- plot5 + theme(plot.title = element_text(hjust = 0))

plot6 <- ggplot(data = survey_data, aes(x=income_family, group= vote_cons, fill=factor(vote_cons, labels = c("not vote for the Conservative Party", 
                                                                                                         "vote for the Conservative Party")))) +
  geom_bar(position = position_dodge(0.9, preserve = "single")) +
  labs(title = "The distribution of Conservative Party voters 
based on household income", 
       x="Household Income", y="Number of people", fill= "Vote Conservative") + 
  scale_fill_brewer(palette = "Dark2") + 
    theme_light() + coord_flip()
plot6 <- plot6 + theme(plot.title = element_text(hjust = 0))


figure_in <- ggarrange(plot5, plot6,ncol = 1, nrow = 2) 
figure_in
```

![](Assignment2_files/figure-markdown_github/unnamed-chunk-11-1.png)

**Description:** The above plot represents the household income
demographic for Liberal Party and non-Liberal Party voters. According to
the plot, the group “$125,000 and more” took the highest proportion of
non-Liberal Party voters. However, the group “125,000 and more” took the
highest proportion of those who will vote for the Liberal Party.

The below plot represents the household income demographic for
Conservative Party and non-Conservative Party voters. According to the
plot, the group “125,000 and more” took the highest proportion of
non-Conservative Party voters. However, the group “125,000 and more”
took the highest proportion of those who will vote for the Conservative
Party.

## Methods

### Methodology introduction

This analysis aims to predict the overall popular vote of the next
Canadian federal election in 2025(tentatively) using a regression
model\[6\] with post-stratiﬁcation\[9\]. To achieve the goal, we will
build the logistic regression models with all predictors first and then
perform the stepwise model selection by AIC\[\] to confirm the final
model with specific predictors. The reason why we use logistic
regression model is that the outcome variable in the model is binary,
which simply only has 0 represents for not vote for liberal and 1
represents for vote for liberal\[23\].

The stepwise variable selection essentially adds or deletes variables
based on which model has the smallest AIC after each addition/deletion.
The methods will add/delete a predictor based on trying to add/delete
every available predictor at a time, then choose to keep the model
results with the smallest AIC value compared to the model we start with.
Moreover, as forward and backward selection only explores the portion of
all possible models, we use the stepwise selection to account for this
dependence. The stepwise selection process will iterate between forward
and backwards selection until we can not add or delete the predictors
anymore\[23\].

### Model Specifics

#### Model for predicting the liberal party

We would be using a logistic regression model to predict the proportion
of voters who will vote for the liberal party. The model from variable
selection comes with five predictors ,including numerical variable age,
four categorical variables birth place in Canada, province,education
degree, and sex.However, for better comparison and anticipation between
conservatives party and Liberal party, we decided to add the income
family in the final model. Here is the final logistic model with six
predictors we would be using to predict the proportion of voters who
will vote for the liberal party.

-   Where *p* represents the probability of voting for the liberal party
-   *β*<sub>0</sub> represents the intercept of the model
-   *β*<sub>1</sub> represents A one year increase in age is associated
    with a *β*<sub>1</sub> increase in the log-odds of voting for
    liberal party.
-   *β*<sub>2</sub> represents the average difference in log odds of
    voting for liberal party between male and female for a certain age,
    birth place Canada, province and education.
-   *β*<sub>3</sub> represents the average difference in log odds of
    voting for liberal party between people born outside Canada and
    those who born in Canada(baseline) and ,holding other constant
-   *β*<sub>4</sub> represents the average difference in log odds of
    voting for liberal party between people don’t know if they born in
    Canada or not and those who born in Canada(baseline) and ,holding
    other constant
-   *β*<sub>5</sub> represents the the average difference in log odds of
    voting for liberal party between people live in British Columbia and
    people live in Alberta(baseline), holding other constant.
-   *β*<sub>6</sub> represents the the average difference in log odds of
    voting for liberal party between people live in Manitoba and people
    live in Alberta(baseline), holding other constant.
-   *β*<sub>7</sub> represents the the average difference in log odds of
    voting for liberal party between people live in New Brunswick and
    people live in Alberta(baseline), holding other constant.
-   *β*<sub>8</sub> represents the the average difference in log odds of
    voting for liberal party between people live in Newfoundland and
    Labrador and people live in Alberta(baseline), holding other
    constant.
-   *β*<sub>9</sub> represents the the average difference in log odds of
    voting for liberal party between people live in Nova Scotia and
    people live in Alberta(baseline), holding other constant.
-   *β*<sub>10</sub> represents the the average difference in log odds
    of voting for liberal party between people live in Ontario and
    people live in Alberta(baseline), holding other constant.
-   *β*<sub>11</sub> represents the the average difference in log odds
    of voting for liberal party between people live in Prince Edward
    Island and people live in Alberta(baseline), holding other constant.
-   *β*<sub>12</sub> represents the the average difference in log odds
    of voting for liberal party between people live in Quebec and people
    live in Alberta(baseline), holding other constant.
-   *β*<sub>13</sub> represents the the average difference in log odds
    of voting for liberal party between people live in Saskatchewan and
    people live in Alberta(baseline), holding other constant.
-   *β*<sub>14</sub> represents the the average difference in log odds
    of voting for liberal party between people with non-university
    certificate and people with bachelor’s degree(baseline), holding
    other constant
-   *β*<sub>15</sub> represents the the average difference in log odds
    of voting for liberal party between people with High school diploma
    and people with bachelor’s degree(baseline), holding other constant
-   *β*<sub>16</sub> represents the the average difference in log odds
    of voting for liberal party between people with non-university
    certificate and people with bachelor’s degree(baseline), holding
    other constant
-   *β*<sub>17</sub> represents the the average difference in log odds
    of voting for liberal party between people with below the bachelor’s
    and people with bachelor’s degree(baseline), holding other constant
-   *β*<sub>18</sub> represents the the average difference in log odds
    of voting for liberal party between people with degree above the
    bachelor’s and people with bachelor’s degree(baseline), holding
    other constant
-   *β*<sub>19</sub> represents the the average difference in log odds
    of voting for liberal party between income =125,000 and more and
    income= 100,000 to 124,999(baseline), holding other constant.
-   *β*<sub>20</sub> represents the the average difference in log odds
    of voting for liberal party between income =25,000 to 49,999 and
    income= 100,000 to 124,999(baseline), holding other constant.
-   *β*<sub>21</sub> represents the the average difference in log odds
    of voting for liberal party between income =50,000 to 74,999 and
    income= 100,000 to 124,999(baseline), holding other constant.
-   *β*<sub>22</sub> represents the the average difference in log odds
    of voting for liberal party between income =75,000 to 99,999 and
    income= 100,000 to 124,999(baseline), holding other constant.
-   *β*<sub>23</sub> represents the the average difference in log odds
    of voting for liberal party between income =Less than 25,000 and
    income= 100,000 to 124,999(baseline), holding other constant.

``` r
# Liberal GLM fit
full_model <- glm(vote_liberal~age+sex+place_birth_canada+hh_size+province+
                    income_family+education, data= survey_data,family="binomial")
summary(full_model)


# Stepwise Selection based on AIC
sel.var.aic1 <- step(full_model, trace = 0, k = 2, direction = "both") 
sel.var.aic1<-attr(terms(sel.var.aic1), "term.labels")   

# selected model 
model_liberal <- glm(vote_liberal~ age + sex + place_birth_canada + province+ 
                       education + income_family, 
              data= survey_data,family="binomial")

summary(model_liberal)
```

#### Model for predicting the conservatives party

We would be using a logistic regression model\[6\] to predict the
proportion of voters who will vote for the conservatives party. This is
the final model with seven predictors after the variable
selection,including numerical variable age and household size, five
categorical variables birth place in Canada, province,education degree,
sex and family income. However, for better comparison and anticipation
between conservatives party and Liberal party, we decided to delete the
household size in the final model. Here is the final logistic model with
six predictors we would be using to predict the proportion of voters who
will vote for the conservatives party.

-   Where *p* represents the probability of voting for the conservatives
    party
-   *β*<sub>0</sub> represents the intercept of the model
-   *β*<sub>1</sub> represents on average, a one year increase in age is
    associated with a *β*<sub>1</sub> increase in the log-odds of voting
    for conservatives party.
-   *β*<sub>2</sub> represents the average difference in log odds of
    voting for conservatives party between male and
    female(baseline),holding other constant.
-   *β*<sub>3</sub> represents the average difference in log odds of
    voting for conservatives party between people born outside Canada
    and those who born in Canada(baseline) and ,holding other constant
-   *β*<sub>4</sub> represents the average difference in log odds of
    voting for conservatives party between people don’t know if they
    born in Canada or not and those who born in Canada(baseline) and
    ,holding other constant
-   *β*<sub>5</sub> represents the the average difference in log odds of
    voting for conservatives party between income =125,000 and more and
    income= 100,000 to 124,999(baseline), holding other constant.
-   *β*<sub>6</sub> represents the the average difference in log odds of
    voting for conservatives party between income =25,000 to 49,999 and
    income= 100,000 to 124,999(baseline), holding other constant.
-   *β*<sub>7</sub> represents the the average difference in log odds of
    voting for conservatives party between income =50,000 to 74,999 and
    income= 100,000 to 124,999(baseline), holding other constant.
-   *β*<sub>8</sub> represents the the average difference in log odds of
    voting for conservatives party between income =75,000 to 99,999 and
    income= 100,000 to 124,999(baseline), holding other constant.
-   *β*<sub>9</sub> represents the the average difference in log odds of
    voting for conservatives party between income =Less than 25,000 and
    income= 100,000 to 124,999(baseline), holding other constant.
-   *β*<sub>10</sub> represents the the average difference in log odds
    of voting for conservatives party between people live in British
    Columbia and people live in Alberta(baseline), holding other
    constant.
-   *β*<sub>11</sub> represents the the average difference in log odds
    of voting for conservatives party between people live in Manitoba
    and people live in Alberta(baseline), holding other constant.
-   *β*<sub>12</sub> represents the the average difference in log odds
    of voting for conservatives party between people live in New
    Brunswick and people live in Alberta(baseline), holding other
    constant.
-   *β*<sub>13</sub> represents the the average difference in log odds
    of voting for conservatives party between people live in
    Newfoundland and Labrador and people live in Alberta(baseline),
    holding other constant.
-   *β*<sub>14</sub> represents the the average difference in log odds
    of voting for conservatives party between people live in Nova Scotia
    and people live in Alberta(baseline), holding other constant.
-   *β*<sub>15</sub> represents the the average difference in log odds
    of voting for conservatives party between people live in Ontario and
    people live in Alberta(baseline), holding other constant.
-   *β*<sub>16</sub> represents the the average difference in log odds
    of voting for conservatives party between people live in Prince
    Edward Island and people live in Alberta(baseline), holding other
    constant.
-   *β*<sub>17</sub> represents the the average difference in log odds
    of voting for conservatives party between people live in Quebec and
    people live in Alberta(baseline), holding other constant.
-   *β*<sub>18</sub> represents the the average difference in log odds
    of voting for conservatives party between people live in
    Saskatchewan and people live in Alberta(baseline), holding other
    constant.
-   *β*<sub>19</sub> represents the the average difference in log odds
    of voting for conservatives party between people with non-university
    certificate and people with bachelor’s degree(baseline), holding
    other constant
-   *β*<sub>20</sub> represents the the average difference in log odds
    of voting for conservatives party between people with High school
    diploma and people with bachelor’s degree(baseline), holding other
    constant
-   *β*<sub>21</sub> represents the the average difference in log odds
    of voting for conservatives party between people with non-university
    certificate and people with bachelor’s degree(baseline), holding
    other constant
-   *β*<sub>22</sub> represents the the average difference in log odds
    of voting for conservatives party between people with below the
    bachelor’s and people with bachelor’s degree(baseline), holding
    other constant
-   *β*<sub>23</sub> represents the the average difference in log odds
    of voting for conservatives party between people with degree above
    the bachelor’s and people with bachelor’s degree(baseline), holding
    other constant

``` r
# Conservatives
full_model2 <- glm(vote_cons~age+sex+place_birth_canada+province+hh_size+
                    income_family+education, data= survey_data,
                   family="binomial")

summary(full_model2)

# Stepwise Selection based on AIC
sel.var.aic2 <- step(full_model2, trace = 0, k = 2, direction = "both") 
sel.var.aic2<-attr(terms(sel.var.aic2), "term.labels")   

# selected model 
model_cons <- glm(vote_cons~age + sex +place_birth_canada +
                    income_family+province+education, 
              data= survey_data,family="binomial")

summary(model_cons)

vif(model_liberal)
vif(model_cons)
```

#### Model Assumption Check

After performing these two logistic models, we need to check four
assumptions which are “outcome is binary”, “linearity in the logit for
continuous variables”, “absence of multicollinearity” and “lack of
strongly influential outliers” separately.

1.  The outcome means results, which is whether respondents vote for
    Liberal Party or Conservative Party. The outcome has only two
    value”1” or “0”.

2.  In order to check the assumption ”linearity in the logit for
    continuous variables”, we need to use Box-Tidwell to check whether
    there is linearity between logit and age.

Firstly, we need to set the null hypothesis and alternative hypothesis.
Null hypothesis means “there is linearity between logit and variable
age”. While the alternative hypothesis is the opposite side of the null
hypothesis, which there is no linearity between logit and variable age.
When we reject the null hypothesis, we conclude this alternative
hypothesis.

Lastly, we need to set a 5% cutoff given the null hypothesis is true and
we call it level of significance in statistics. The region is named
rejection region and it represents the extreme value. If the value falls
in the region, we say we reject the null hypothesis, otherwise we fail
to reject the null hypothesis.

*H*<sub>0</sub>: linearity between continuous variable(age) and log-odds
*H*<sub>*A*</sub>: non-linearity between continuous variable(age) and
log-odds

``` r
#linearity
boxTidwell(vote_liberal ~ age, data=survey_data)
```

    ##  MLE of lambda Score Statistic (t) Pr(>|t|)
    ##         3.9192              1.3566    0.175
    ## 
    ## iterations =  5

p-value\[23\] is the probability of observing extreme value given the
null hypothesis is true. After performing the Box-Tidewell, we get
p-value 0.175, which is fairly large, meaning that the linearity
assumption seems to be satisfied.

1.  We used variance inflation factor to check whether there is a
    relationship between predictors, which will affect the final
    results. We assessed these two models. And the values are all
    approximately 1, which indicates the absence of multicollinearity.

2.  As can seen from the plots above, there was no influential points.
    This is the final model with five predictors after the variable
    selection,including numerical variable age, four categorical
    variables birth place in Canada, province,education degree, and
    sex.However, for better comparison and anticipation between
    conservatives party and Liberal party, we decided to add the income
    family in the final model. Here is the final logistic model with
    five predictorsweam using to predict the proportion of voters who
    will vote for the liberal party

#### Limitation of method:

The stepwise selection comes at a heavy price although it is quite
helpful to select models. The estimated coefficients from a post-model
selection model will be actually biased estimators, and the selected
model heavily depends on the data. The method usually result larger test
statistic than it should be, leading to reject more than we should(Type
I error). Therefore, we need to validate the model to further determine
if the model is reasonable for prediction.

## Post-Stratification

After fitting the model, we performed the post-stratification\[9\] to
get the final results. Post-stratification analysis can provide us with
representative results and improve the accuracy of the survey estimate.
Post-stratification is a statistical method that estimates the value
(possibility) for each bin. **Note that, the word “bin” in statistics
refers to “category” and “cell”.** Based on the research variables, we
divided the entire population into different cells to conduct the
analysis. Then, we used demographics to “extrapolate” how the entire
population will vote.

Here is the formula of the post-stratification:

$$\hat{y}^{P S}=\frac{\sum N\_{j} \widehat{y}\_{j}}{\sum N\_{j}}$$

The formula demonstrates that we multiply the estimate for each bin by
its proportion to the entire population. Then we add them together to
obtain the final answer.

In our investigation,we divided the census data into different bins
based on the seven predictor variables in our model(age, sex,
birthplace, province, household size, household income, and education).
We applied the model to each bin to predict the estimate. After
completing this, we were able to find the proportion of voters from each
party at each bin. We then multiplied the proportion with the estimate
of each bin. We were able to complete our post stratification analysis
by adding the number at each bin together.

## Results

Here is the value of each parameter that we use in the model.

``` r
as_flextable(model_liberal) %>% autofit() %>% fit_to_width(7.5)
```

<div class="tabwid"><style>.cl-39711548{}.cl-396d115a{font-family:'Helvetica';font-size:8pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-396d1164{font-family:'Helvetica';font-size:8pt;font-weight:normal;font-style:italic;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-396e7de2{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-396e7dec{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-396e87d8{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e87d9{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e87da{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e87e2{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e87e3{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e87e4{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e87e5{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e87ec{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e87ed{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e87ee{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e87ef{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e87f6{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e87f7{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8800{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8801{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8802{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e880a{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e880b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e880c{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e880d{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8814{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8815{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8816{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8817{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e881e{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e881f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8820{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8828{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8829{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e882a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e882b{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e882c{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8832{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8833{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8834{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e883c{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e883d{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e883e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8846{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8847{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8848{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8850{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8851{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8852{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8853{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e885a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e885b{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e885c{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e885d{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e885e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e885f{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8864{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8865{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8866{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8867{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8868{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e886e{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(255, 255, 255, 0.00);border-top: 0 solid rgba(255, 255, 255, 0.00);border-left: 0 solid rgba(255, 255, 255, 0.00);border-right: 0 solid rgba(255, 255, 255, 0.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e886f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(255, 255, 255, 0.00);border-top: 0 solid rgba(255, 255, 255, 0.00);border-left: 0 solid rgba(255, 255, 255, 0.00);border-right: 0 solid rgba(255, 255, 255, 0.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8870{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(255, 255, 255, 0.00);border-top: 0 solid rgba(255, 255, 255, 0.00);border-left: 0 solid rgba(255, 255, 255, 0.00);border-right: 0 solid rgba(255, 255, 255, 0.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e8871{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(255, 255, 255, 0.00);border-top: 0 solid rgba(255, 255, 255, 0.00);border-left: 0 solid rgba(255, 255, 255, 0.00);border-right: 0 solid rgba(255, 255, 255, 0.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e88a0{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(255, 255, 255, 0.00);border-top: 0 solid rgba(255, 255, 255, 0.00);border-left: 0 solid rgba(255, 255, 255, 0.00);border-right: 0 solid rgba(255, 255, 255, 0.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e88a1{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(255, 255, 255, 0.00);border-top: 0 solid rgba(255, 255, 255, 0.00);border-left: 0 solid rgba(255, 255, 255, 0.00);border-right: 0 solid rgba(255, 255, 255, 0.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e88a2{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(255, 255, 255, 0.00);border-top: 0 solid rgba(255, 255, 255, 0.00);border-left: 0 solid rgba(255, 255, 255, 0.00);border-right: 0 solid rgba(255, 255, 255, 0.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-396e88aa{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(255, 255, 255, 0.00);border-top: 0 solid rgba(255, 255, 255, 0.00);border-left: 0 solid rgba(255, 255, 255, 0.00);border-right: 0 solid rgba(255, 255, 255, 0.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-39711548'><thead><tr style="overflow-wrap:break-word;"><th class="cl-396e87d8"><p class="cl-396e7de2"><span class="cl-396d115a"></span></p></th><th class="cl-396e87d9"><p class="cl-396e7dec"><span class="cl-396d115a">Estimate</span></p></th><th class="cl-396e87da"><p class="cl-396e7dec"><span class="cl-396d115a">Standard Error</span></p></th><th class="cl-396e87d9"><p class="cl-396e7dec"><span class="cl-396d115a">z value</span></p></th><th class="cl-396e87d9"><p class="cl-396e7dec"><span class="cl-396d115a">Pr(&gt;|z|)</span></p></th><th class="cl-396e87e2"><p class="cl-396e7de2"><span class="cl-396d115a"></span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-396e87e3"><p class="cl-396e7de2"><span class="cl-396d115a">(Intercept)</span></p></td><td class="cl-396e87e4"><p class="cl-396e7dec"><span class="cl-396d115a">-2.000</span></p></td><td class="cl-396e87e5"><p class="cl-396e7dec"><span class="cl-396d115a">0.312</span></p></td><td class="cl-396e87e4"><p class="cl-396e7dec"><span class="cl-396d115a">-6.406</span></p></td><td class="cl-396e87e4"><p class="cl-396e7dec"><span class="cl-396d115a">0.0000</span></p></td><td class="cl-396e87ec"><p class="cl-396e7de2"><span class="cl-396d115a">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e87ed"><p class="cl-396e7de2"><span class="cl-396d115a">age</span></p></td><td class="cl-396e87ee"><p class="cl-396e7dec"><span class="cl-396d115a">0.010</span></p></td><td class="cl-396e87ef"><p class="cl-396e7dec"><span class="cl-396d115a">0.003</span></p></td><td class="cl-396e87ee"><p class="cl-396e7dec"><span class="cl-396d115a">3.177</span></p></td><td class="cl-396e87ee"><p class="cl-396e7dec"><span class="cl-396d115a">0.0015</span></p></td><td class="cl-396e87f6"><p class="cl-396e7de2"><span class="cl-396d115a"> **</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e87f7"><p class="cl-396e7de2"><span class="cl-396d115a">sexMale</span></p></td><td class="cl-396e8800"><p class="cl-396e7dec"><span class="cl-396d115a">-0.156</span></p></td><td class="cl-396e8801"><p class="cl-396e7dec"><span class="cl-396d115a">0.097</span></p></td><td class="cl-396e8800"><p class="cl-396e7dec"><span class="cl-396d115a">-1.613</span></p></td><td class="cl-396e8800"><p class="cl-396e7dec"><span class="cl-396d115a">0.1068</span></p></td><td class="cl-396e8802"><p class="cl-396e7de2"><span class="cl-396d115a">   </span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e880a"><p class="cl-396e7de2"><span class="cl-396d115a">place_birth_canadaBorn outside Canada</span></p></td><td class="cl-396e880b"><p class="cl-396e7dec"><span class="cl-396d115a">0.657</span></p></td><td class="cl-396e880c"><p class="cl-396e7dec"><span class="cl-396d115a">0.129</span></p></td><td class="cl-396e880b"><p class="cl-396e7dec"><span class="cl-396d115a">5.090</span></p></td><td class="cl-396e880b"><p class="cl-396e7dec"><span class="cl-396d115a">0.0000</span></p></td><td class="cl-396e880d"><p class="cl-396e7de2"><span class="cl-396d115a">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e8814"><p class="cl-396e7de2"><span class="cl-396d115a">place_birth_canadaDon't know</span></p></td><td class="cl-396e8815"><p class="cl-396e7dec"><span class="cl-396d115a">12.692</span></p></td><td class="cl-396e8816"><p class="cl-396e7dec"><span class="cl-396d115a">324.744</span></p></td><td class="cl-396e8815"><p class="cl-396e7dec"><span class="cl-396d115a">0.039</span></p></td><td class="cl-396e8815"><p class="cl-396e7dec"><span class="cl-396d115a">0.9688</span></p></td><td class="cl-396e8817"><p class="cl-396e7de2"><span class="cl-396d115a">   </span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e880a"><p class="cl-396e7de2"><span class="cl-396d115a">provinceBritish Columbia</span></p></td><td class="cl-396e880b"><p class="cl-396e7dec"><span class="cl-396d115a">0.655</span></p></td><td class="cl-396e880c"><p class="cl-396e7dec"><span class="cl-396d115a">0.258</span></p></td><td class="cl-396e880b"><p class="cl-396e7dec"><span class="cl-396d115a">2.540</span></p></td><td class="cl-396e880b"><p class="cl-396e7dec"><span class="cl-396d115a">0.0111</span></p></td><td class="cl-396e880d"><p class="cl-396e7de2"><span class="cl-396d115a">  *</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e8814"><p class="cl-396e7de2"><span class="cl-396d115a">provinceManitoba</span></p></td><td class="cl-396e8815"><p class="cl-396e7dec"><span class="cl-396d115a">0.821</span></p></td><td class="cl-396e8816"><p class="cl-396e7dec"><span class="cl-396d115a">0.295</span></p></td><td class="cl-396e8815"><p class="cl-396e7dec"><span class="cl-396d115a">2.788</span></p></td><td class="cl-396e8815"><p class="cl-396e7dec"><span class="cl-396d115a">0.0053</span></p></td><td class="cl-396e8817"><p class="cl-396e7de2"><span class="cl-396d115a"> **</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e881e"><p class="cl-396e7de2"><span class="cl-396d115a">provinceNew Brunswick</span></p></td><td class="cl-396e881f"><p class="cl-396e7dec"><span class="cl-396d115a">1.252</span></p></td><td class="cl-396e8820"><p class="cl-396e7dec"><span class="cl-396d115a">0.316</span></p></td><td class="cl-396e881f"><p class="cl-396e7dec"><span class="cl-396d115a">3.962</span></p></td><td class="cl-396e881f"><p class="cl-396e7dec"><span class="cl-396d115a">0.0001</span></p></td><td class="cl-396e8828"><p class="cl-396e7de2"><span class="cl-396d115a">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e8829"><p class="cl-396e7de2"><span class="cl-396d115a">provinceNewfoundland and Labrador</span></p></td><td class="cl-396e882a"><p class="cl-396e7dec"><span class="cl-396d115a">1.519</span></p></td><td class="cl-396e882b"><p class="cl-396e7dec"><span class="cl-396d115a">0.312</span></p></td><td class="cl-396e882a"><p class="cl-396e7dec"><span class="cl-396d115a">4.868</span></p></td><td class="cl-396e882a"><p class="cl-396e7dec"><span class="cl-396d115a">0.0000</span></p></td><td class="cl-396e882c"><p class="cl-396e7de2"><span class="cl-396d115a">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e880a"><p class="cl-396e7de2"><span class="cl-396d115a">provinceNova Scotia</span></p></td><td class="cl-396e880b"><p class="cl-396e7dec"><span class="cl-396d115a">1.346</span></p></td><td class="cl-396e880c"><p class="cl-396e7dec"><span class="cl-396d115a">0.306</span></p></td><td class="cl-396e880b"><p class="cl-396e7dec"><span class="cl-396d115a">4.405</span></p></td><td class="cl-396e880b"><p class="cl-396e7dec"><span class="cl-396d115a">0.0000</span></p></td><td class="cl-396e880d"><p class="cl-396e7de2"><span class="cl-396d115a">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e880a"><p class="cl-396e7de2"><span class="cl-396d115a">provinceOntario</span></p></td><td class="cl-396e880b"><p class="cl-396e7dec"><span class="cl-396d115a">1.381</span></p></td><td class="cl-396e880c"><p class="cl-396e7dec"><span class="cl-396d115a">0.252</span></p></td><td class="cl-396e880b"><p class="cl-396e7dec"><span class="cl-396d115a">5.474</span></p></td><td class="cl-396e880b"><p class="cl-396e7dec"><span class="cl-396d115a">0.0000</span></p></td><td class="cl-396e880d"><p class="cl-396e7de2"><span class="cl-396d115a">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e8814"><p class="cl-396e7de2"><span class="cl-396d115a">provincePrince Edward Island</span></p></td><td class="cl-396e8815"><p class="cl-396e7dec"><span class="cl-396d115a">1.450</span></p></td><td class="cl-396e8816"><p class="cl-396e7dec"><span class="cl-396d115a">0.308</span></p></td><td class="cl-396e8815"><p class="cl-396e7dec"><span class="cl-396d115a">4.710</span></p></td><td class="cl-396e8815"><p class="cl-396e7dec"><span class="cl-396d115a">0.0000</span></p></td><td class="cl-396e8817"><p class="cl-396e7de2"><span class="cl-396d115a">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e880a"><p class="cl-396e7de2"><span class="cl-396d115a">provinceQuebec</span></p></td><td class="cl-396e880b"><p class="cl-396e7dec"><span class="cl-396d115a">1.193</span></p></td><td class="cl-396e880c"><p class="cl-396e7dec"><span class="cl-396d115a">0.257</span></p></td><td class="cl-396e880b"><p class="cl-396e7dec"><span class="cl-396d115a">4.647</span></p></td><td class="cl-396e880b"><p class="cl-396e7dec"><span class="cl-396d115a">0.0000</span></p></td><td class="cl-396e880d"><p class="cl-396e7de2"><span class="cl-396d115a">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e880a"><p class="cl-396e7de2"><span class="cl-396d115a">provinceSaskatchewan</span></p></td><td class="cl-396e880b"><p class="cl-396e7dec"><span class="cl-396d115a">0.023</span></p></td><td class="cl-396e880c"><p class="cl-396e7dec"><span class="cl-396d115a">0.330</span></p></td><td class="cl-396e880b"><p class="cl-396e7dec"><span class="cl-396d115a">0.068</span></p></td><td class="cl-396e880b"><p class="cl-396e7dec"><span class="cl-396d115a">0.9455</span></p></td><td class="cl-396e880d"><p class="cl-396e7de2"><span class="cl-396d115a">   </span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e8832"><p class="cl-396e7de2"><span class="cl-396d115a">educationCollege, CEGEP or other non-university certificate or di...</span></p></td><td class="cl-396e8833"><p class="cl-396e7dec"><span class="cl-396d115a">-0.545</span></p></td><td class="cl-396e8834"><p class="cl-396e7dec"><span class="cl-396d115a">0.130</span></p></td><td class="cl-396e8833"><p class="cl-396e7dec"><span class="cl-396d115a">-4.181</span></p></td><td class="cl-396e8833"><p class="cl-396e7dec"><span class="cl-396d115a">0.0000</span></p></td><td class="cl-396e883c"><p class="cl-396e7de2"><span class="cl-396d115a">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e883d"><p class="cl-396e7de2"><span class="cl-396d115a">educationHigh school diploma or a high school equivalency certificate</span></p></td><td class="cl-396e883e"><p class="cl-396e7dec"><span class="cl-396d115a">-0.307</span></p></td><td class="cl-396e8846"><p class="cl-396e7dec"><span class="cl-396d115a">0.162</span></p></td><td class="cl-396e883e"><p class="cl-396e7dec"><span class="cl-396d115a">-1.898</span></p></td><td class="cl-396e883e"><p class="cl-396e7dec"><span class="cl-396d115a">0.0576</span></p></td><td class="cl-396e8847"><p class="cl-396e7de2"><span class="cl-396d115a">  .</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e8848"><p class="cl-396e7de2"><span class="cl-396d115a">educationLess than high school diploma or its equivalent</span></p></td><td class="cl-396e8850"><p class="cl-396e7dec"><span class="cl-396d115a">-0.509</span></p></td><td class="cl-396e8851"><p class="cl-396e7dec"><span class="cl-396d115a">0.245</span></p></td><td class="cl-396e8850"><p class="cl-396e7dec"><span class="cl-396d115a">-2.074</span></p></td><td class="cl-396e8850"><p class="cl-396e7dec"><span class="cl-396d115a">0.0381</span></p></td><td class="cl-396e8852"><p class="cl-396e7de2"><span class="cl-396d115a">  *</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e8853"><p class="cl-396e7de2"><span class="cl-396d115a">educationUniversity certificate or diploma below the bachelor's level</span></p></td><td class="cl-396e885a"><p class="cl-396e7dec"><span class="cl-396d115a">-0.174</span></p></td><td class="cl-396e885b"><p class="cl-396e7dec"><span class="cl-396d115a">0.183</span></p></td><td class="cl-396e885a"><p class="cl-396e7dec"><span class="cl-396d115a">-0.950</span></p></td><td class="cl-396e885a"><p class="cl-396e7dec"><span class="cl-396d115a">0.3422</span></p></td><td class="cl-396e885c"><p class="cl-396e7de2"><span class="cl-396d115a">   </span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e883d"><p class="cl-396e7de2"><span class="cl-396d115a">educationUniversity certificate, diploma or degree above the bach...</span></p></td><td class="cl-396e883e"><p class="cl-396e7dec"><span class="cl-396d115a">0.111</span></p></td><td class="cl-396e8846"><p class="cl-396e7dec"><span class="cl-396d115a">0.138</span></p></td><td class="cl-396e883e"><p class="cl-396e7dec"><span class="cl-396d115a">0.804</span></p></td><td class="cl-396e883e"><p class="cl-396e7dec"><span class="cl-396d115a">0.4214</span></p></td><td class="cl-396e8847"><p class="cl-396e7de2"><span class="cl-396d115a">   </span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e885d"><p class="cl-396e7de2"><span class="cl-396d115a">income_family$125,000 and more</span></p></td><td class="cl-396e885e"><p class="cl-396e7dec"><span class="cl-396d115a">0.010</span></p></td><td class="cl-396e885f"><p class="cl-396e7dec"><span class="cl-396d115a">0.161</span></p></td><td class="cl-396e885e"><p class="cl-396e7dec"><span class="cl-396d115a">0.065</span></p></td><td class="cl-396e885e"><p class="cl-396e7dec"><span class="cl-396d115a">0.9482</span></p></td><td class="cl-396e8864"><p class="cl-396e7de2"><span class="cl-396d115a">   </span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e885d"><p class="cl-396e7de2"><span class="cl-396d115a">income_family$25,000 to $49,999</span></p></td><td class="cl-396e885e"><p class="cl-396e7dec"><span class="cl-396d115a">-0.044</span></p></td><td class="cl-396e885f"><p class="cl-396e7dec"><span class="cl-396d115a">0.188</span></p></td><td class="cl-396e885e"><p class="cl-396e7dec"><span class="cl-396d115a">-0.236</span></p></td><td class="cl-396e885e"><p class="cl-396e7dec"><span class="cl-396d115a">0.8134</span></p></td><td class="cl-396e8864"><p class="cl-396e7de2"><span class="cl-396d115a">   </span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e885d"><p class="cl-396e7de2"><span class="cl-396d115a">income_family$50,000 to $74,999</span></p></td><td class="cl-396e885e"><p class="cl-396e7dec"><span class="cl-396d115a">-0.048</span></p></td><td class="cl-396e885f"><p class="cl-396e7dec"><span class="cl-396d115a">0.177</span></p></td><td class="cl-396e885e"><p class="cl-396e7dec"><span class="cl-396d115a">-0.270</span></p></td><td class="cl-396e885e"><p class="cl-396e7dec"><span class="cl-396d115a">0.7872</span></p></td><td class="cl-396e8864"><p class="cl-396e7de2"><span class="cl-396d115a">   </span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e885d"><p class="cl-396e7de2"><span class="cl-396d115a">income_family$75,000 to $99,999</span></p></td><td class="cl-396e885e"><p class="cl-396e7dec"><span class="cl-396d115a">0.085</span></p></td><td class="cl-396e885f"><p class="cl-396e7dec"><span class="cl-396d115a">0.185</span></p></td><td class="cl-396e885e"><p class="cl-396e7dec"><span class="cl-396d115a">0.459</span></p></td><td class="cl-396e885e"><p class="cl-396e7dec"><span class="cl-396d115a">0.6462</span></p></td><td class="cl-396e8864"><p class="cl-396e7de2"><span class="cl-396d115a">   </span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-396e8865"><p class="cl-396e7de2"><span class="cl-396d115a">income_familyLess than $25,000</span></p></td><td class="cl-396e8866"><p class="cl-396e7dec"><span class="cl-396d115a">-0.233</span></p></td><td class="cl-396e8867"><p class="cl-396e7dec"><span class="cl-396d115a">0.202</span></p></td><td class="cl-396e8866"><p class="cl-396e7dec"><span class="cl-396d115a">-1.152</span></p></td><td class="cl-396e8866"><p class="cl-396e7dec"><span class="cl-396d115a">0.2493</span></p></td><td class="cl-396e8868"><p class="cl-396e7de2"><span class="cl-396d115a">   </span></p></td></tr></tbody><tfoot><tr style="overflow-wrap:break-word;"><td  colspan="6"class="cl-396e886e"><p class="cl-396e7dec"><span class="cl-396d1164">Signif. codes: 0 &lt;= '***' &lt; 0.001 &lt; '**' &lt; 0.01 &lt; '*' &lt; 0.05</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  colspan="6"class="cl-396e88a0"><p class="cl-396e7de2"><span class="cl-396d115a"> </span></p></td></tr><tr style="overflow-wrap:break-word;"><td  colspan="6"class="cl-396e88a0"><p class="cl-396e7de2"><span class="cl-396d115a">(Dispersion parameter for binomial family taken to be 1)</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  colspan="6"class="cl-396e88a0"><p class="cl-396e7de2"><span class="cl-396d115a">Null deviance: 2808 on 2212 degrees of freedom</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  colspan="6"class="cl-396e88a0"><p class="cl-396e7de2"><span class="cl-396d115a">Residual deviance: 2625 on 2189 degrees of freedom</span></p></td></tr></tfoot></table></div>

``` r
as_flextable(model_cons) %>% autofit() %>% fit_to_width(7.5)
```

<div class="tabwid"><style>.cl-399b5b50{}.cl-39981706{font-family:'Helvetica';font-size:8pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-39981710{font-family:'Helvetica';font-size:8pt;font-weight:normal;font-style:italic;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-39995602{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-3999560c{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-39995ea4{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995ea5{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995eae{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995eaf{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995eb0{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995eb8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995eb9{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995eba{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995ec2{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995ec3{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995ec4{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995ec5{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995ecc{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995ecd{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995ece{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995ecf{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995ed6{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995ed7{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995ed8{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995ed9{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995eda{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995ee0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995ee1{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995ee2{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995ee3{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995eea{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995eeb{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995eec{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995eed{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995ef4{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995ef5{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995ef6{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995efe{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995eff{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f00{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f01{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f02{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f08{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f09{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f0a{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f0b{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f0c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f0d{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f0e{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f12{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f13{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f14{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f15{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f1c{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f1d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f1e{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f1f{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f20{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f21{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f26{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f27{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f28{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(255, 255, 255, 0.00);border-top: 0 solid rgba(255, 255, 255, 0.00);border-left: 0 solid rgba(255, 255, 255, 0.00);border-right: 0 solid rgba(255, 255, 255, 0.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f29{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(255, 255, 255, 0.00);border-top: 0 solid rgba(255, 255, 255, 0.00);border-left: 0 solid rgba(255, 255, 255, 0.00);border-right: 0 solid rgba(255, 255, 255, 0.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f2a{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(255, 255, 255, 0.00);border-top: 0 solid rgba(255, 255, 255, 0.00);border-left: 0 solid rgba(255, 255, 255, 0.00);border-right: 0 solid rgba(255, 255, 255, 0.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f2b{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(255, 255, 255, 0.00);border-top: 0 solid rgba(255, 255, 255, 0.00);border-left: 0 solid rgba(255, 255, 255, 0.00);border-right: 0 solid rgba(255, 255, 255, 0.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f2c{width:3.607in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(255, 255, 255, 0.00);border-top: 0 solid rgba(255, 255, 255, 0.00);border-left: 0 solid rgba(255, 255, 255, 0.00);border-right: 0 solid rgba(255, 255, 255, 0.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f30{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(255, 255, 255, 0.00);border-top: 0 solid rgba(255, 255, 255, 0.00);border-left: 0 solid rgba(255, 255, 255, 0.00);border-right: 0 solid rgba(255, 255, 255, 0.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f31{width:0.914in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(255, 255, 255, 0.00);border-top: 0 solid rgba(255, 255, 255, 0.00);border-left: 0 solid rgba(255, 255, 255, 0.00);border-right: 0 solid rgba(255, 255, 255, 0.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-39995f32{width:0.4in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(255, 255, 255, 0.00);border-top: 0 solid rgba(255, 255, 255, 0.00);border-left: 0 solid rgba(255, 255, 255, 0.00);border-right: 0 solid rgba(255, 255, 255, 0.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-399b5b50'><thead><tr style="overflow-wrap:break-word;"><th class="cl-39995ea4"><p class="cl-39995602"><span class="cl-39981706"></span></p></th><th class="cl-39995ea5"><p class="cl-3999560c"><span class="cl-39981706">Estimate</span></p></th><th class="cl-39995eae"><p class="cl-3999560c"><span class="cl-39981706">Standard Error</span></p></th><th class="cl-39995ea5"><p class="cl-3999560c"><span class="cl-39981706">z value</span></p></th><th class="cl-39995ea5"><p class="cl-3999560c"><span class="cl-39981706">Pr(&gt;|z|)</span></p></th><th class="cl-39995eaf"><p class="cl-39995602"><span class="cl-39981706"></span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-39995eb0"><p class="cl-39995602"><span class="cl-39981706">(Intercept)</span></p></td><td class="cl-39995eb8"><p class="cl-3999560c"><span class="cl-39981706">-0.244</span></p></td><td class="cl-39995eb9"><p class="cl-3999560c"><span class="cl-39981706">0.282</span></p></td><td class="cl-39995eb8"><p class="cl-3999560c"><span class="cl-39981706">-0.866</span></p></td><td class="cl-39995eb8"><p class="cl-3999560c"><span class="cl-39981706">0.3867</span></p></td><td class="cl-39995eba"><p class="cl-39995602"><span class="cl-39981706">   </span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995ec2"><p class="cl-39995602"><span class="cl-39981706">age</span></p></td><td class="cl-39995ec3"><p class="cl-3999560c"><span class="cl-39981706">0.012</span></p></td><td class="cl-39995ec4"><p class="cl-3999560c"><span class="cl-39981706">0.003</span></p></td><td class="cl-39995ec3"><p class="cl-3999560c"><span class="cl-39981706">3.738</span></p></td><td class="cl-39995ec3"><p class="cl-3999560c"><span class="cl-39981706">0.0002</span></p></td><td class="cl-39995ec5"><p class="cl-39995602"><span class="cl-39981706">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995ecc"><p class="cl-39995602"><span class="cl-39981706">sexMale</span></p></td><td class="cl-39995ecd"><p class="cl-3999560c"><span class="cl-39981706">0.632</span></p></td><td class="cl-39995ece"><p class="cl-3999560c"><span class="cl-39981706">0.103</span></p></td><td class="cl-39995ecd"><p class="cl-3999560c"><span class="cl-39981706">6.110</span></p></td><td class="cl-39995ecd"><p class="cl-3999560c"><span class="cl-39981706">0.0000</span></p></td><td class="cl-39995ecf"><p class="cl-39995602"><span class="cl-39981706">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995ed6"><p class="cl-39995602"><span class="cl-39981706">place_birth_canadaBorn outside Canada</span></p></td><td class="cl-39995ed7"><p class="cl-3999560c"><span class="cl-39981706">-0.190</span></p></td><td class="cl-39995ed8"><p class="cl-3999560c"><span class="cl-39981706">0.146</span></p></td><td class="cl-39995ed7"><p class="cl-3999560c"><span class="cl-39981706">-1.297</span></p></td><td class="cl-39995ed7"><p class="cl-3999560c"><span class="cl-39981706">0.1946</span></p></td><td class="cl-39995ed9"><p class="cl-39995602"><span class="cl-39981706">   </span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995eda"><p class="cl-39995602"><span class="cl-39981706">place_birth_canadaDon't know</span></p></td><td class="cl-39995ee0"><p class="cl-3999560c"><span class="cl-39981706">-10.938</span></p></td><td class="cl-39995ee1"><p class="cl-3999560c"><span class="cl-39981706">324.744</span></p></td><td class="cl-39995ee0"><p class="cl-3999560c"><span class="cl-39981706">-0.034</span></p></td><td class="cl-39995ee0"><p class="cl-3999560c"><span class="cl-39981706">0.9731</span></p></td><td class="cl-39995ee2"><p class="cl-39995602"><span class="cl-39981706">   </span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995ee3"><p class="cl-39995602"><span class="cl-39981706">income_family$125,000 and more</span></p></td><td class="cl-39995eea"><p class="cl-3999560c"><span class="cl-39981706">0.366</span></p></td><td class="cl-39995eeb"><p class="cl-3999560c"><span class="cl-39981706">0.164</span></p></td><td class="cl-39995eea"><p class="cl-3999560c"><span class="cl-39981706">2.226</span></p></td><td class="cl-39995eea"><p class="cl-3999560c"><span class="cl-39981706">0.0260</span></p></td><td class="cl-39995eec"><p class="cl-39995602"><span class="cl-39981706">  *</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995ee3"><p class="cl-39995602"><span class="cl-39981706">income_family$25,000 to $49,999</span></p></td><td class="cl-39995eea"><p class="cl-3999560c"><span class="cl-39981706">-0.248</span></p></td><td class="cl-39995eeb"><p class="cl-3999560c"><span class="cl-39981706">0.195</span></p></td><td class="cl-39995eea"><p class="cl-3999560c"><span class="cl-39981706">-1.277</span></p></td><td class="cl-39995eea"><p class="cl-3999560c"><span class="cl-39981706">0.2015</span></p></td><td class="cl-39995eec"><p class="cl-39995602"><span class="cl-39981706">   </span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995ee3"><p class="cl-39995602"><span class="cl-39981706">income_family$50,000 to $74,999</span></p></td><td class="cl-39995eea"><p class="cl-3999560c"><span class="cl-39981706">-0.151</span></p></td><td class="cl-39995eeb"><p class="cl-3999560c"><span class="cl-39981706">0.182</span></p></td><td class="cl-39995eea"><p class="cl-3999560c"><span class="cl-39981706">-0.832</span></p></td><td class="cl-39995eea"><p class="cl-3999560c"><span class="cl-39981706">0.4052</span></p></td><td class="cl-39995eec"><p class="cl-39995602"><span class="cl-39981706">   </span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995ee3"><p class="cl-39995602"><span class="cl-39981706">income_family$75,000 to $99,999</span></p></td><td class="cl-39995eea"><p class="cl-3999560c"><span class="cl-39981706">-0.132</span></p></td><td class="cl-39995eeb"><p class="cl-3999560c"><span class="cl-39981706">0.195</span></p></td><td class="cl-39995eea"><p class="cl-3999560c"><span class="cl-39981706">-0.679</span></p></td><td class="cl-39995eea"><p class="cl-3999560c"><span class="cl-39981706">0.4974</span></p></td><td class="cl-39995eec"><p class="cl-39995602"><span class="cl-39981706">   </span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995ee3"><p class="cl-39995602"><span class="cl-39981706">income_familyLess than $25,000</span></p></td><td class="cl-39995eea"><p class="cl-3999560c"><span class="cl-39981706">-0.107</span></p></td><td class="cl-39995eeb"><p class="cl-3999560c"><span class="cl-39981706">0.208</span></p></td><td class="cl-39995eea"><p class="cl-3999560c"><span class="cl-39981706">-0.515</span></p></td><td class="cl-39995eea"><p class="cl-3999560c"><span class="cl-39981706">0.6067</span></p></td><td class="cl-39995eec"><p class="cl-39995602"><span class="cl-39981706">   </span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995ed6"><p class="cl-39995602"><span class="cl-39981706">provinceBritish Columbia</span></p></td><td class="cl-39995ed7"><p class="cl-3999560c"><span class="cl-39981706">-1.922</span></p></td><td class="cl-39995ed8"><p class="cl-3999560c"><span class="cl-39981706">0.216</span></p></td><td class="cl-39995ed7"><p class="cl-3999560c"><span class="cl-39981706">-8.895</span></p></td><td class="cl-39995ed7"><p class="cl-3999560c"><span class="cl-39981706">0.0000</span></p></td><td class="cl-39995ed9"><p class="cl-39995602"><span class="cl-39981706">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995eda"><p class="cl-39995602"><span class="cl-39981706">provinceManitoba</span></p></td><td class="cl-39995ee0"><p class="cl-3999560c"><span class="cl-39981706">-1.004</span></p></td><td class="cl-39995ee1"><p class="cl-3999560c"><span class="cl-39981706">0.250</span></p></td><td class="cl-39995ee0"><p class="cl-3999560c"><span class="cl-39981706">-4.015</span></p></td><td class="cl-39995ee0"><p class="cl-3999560c"><span class="cl-39981706">0.0001</span></p></td><td class="cl-39995ee2"><p class="cl-39995602"><span class="cl-39981706">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995eed"><p class="cl-39995602"><span class="cl-39981706">provinceNew Brunswick</span></p></td><td class="cl-39995ef4"><p class="cl-3999560c"><span class="cl-39981706">-1.389</span></p></td><td class="cl-39995ef5"><p class="cl-3999560c"><span class="cl-39981706">0.286</span></p></td><td class="cl-39995ef4"><p class="cl-3999560c"><span class="cl-39981706">-4.862</span></p></td><td class="cl-39995ef4"><p class="cl-3999560c"><span class="cl-39981706">0.0000</span></p></td><td class="cl-39995ef6"><p class="cl-39995602"><span class="cl-39981706">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995efe"><p class="cl-39995602"><span class="cl-39981706">provinceNewfoundland and Labrador</span></p></td><td class="cl-39995eff"><p class="cl-3999560c"><span class="cl-39981706">-1.974</span></p></td><td class="cl-39995f00"><p class="cl-3999560c"><span class="cl-39981706">0.294</span></p></td><td class="cl-39995eff"><p class="cl-3999560c"><span class="cl-39981706">-6.716</span></p></td><td class="cl-39995eff"><p class="cl-3999560c"><span class="cl-39981706">0.0000</span></p></td><td class="cl-39995f01"><p class="cl-39995602"><span class="cl-39981706">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995ed6"><p class="cl-39995602"><span class="cl-39981706">provinceNova Scotia</span></p></td><td class="cl-39995ed7"><p class="cl-3999560c"><span class="cl-39981706">-1.759</span></p></td><td class="cl-39995ed8"><p class="cl-3999560c"><span class="cl-39981706">0.282</span></p></td><td class="cl-39995ed7"><p class="cl-3999560c"><span class="cl-39981706">-6.237</span></p></td><td class="cl-39995ed7"><p class="cl-3999560c"><span class="cl-39981706">0.0000</span></p></td><td class="cl-39995ed9"><p class="cl-39995602"><span class="cl-39981706">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995ed6"><p class="cl-39995602"><span class="cl-39981706">provinceOntario</span></p></td><td class="cl-39995ed7"><p class="cl-3999560c"><span class="cl-39981706">-1.715</span></p></td><td class="cl-39995ed8"><p class="cl-3999560c"><span class="cl-39981706">0.214</span></p></td><td class="cl-39995ed7"><p class="cl-3999560c"><span class="cl-39981706">-8.021</span></p></td><td class="cl-39995ed7"><p class="cl-3999560c"><span class="cl-39981706">0.0000</span></p></td><td class="cl-39995ed9"><p class="cl-39995602"><span class="cl-39981706">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995eda"><p class="cl-39995602"><span class="cl-39981706">provincePrince Edward Island</span></p></td><td class="cl-39995ee0"><p class="cl-3999560c"><span class="cl-39981706">-1.784</span></p></td><td class="cl-39995ee1"><p class="cl-3999560c"><span class="cl-39981706">0.287</span></p></td><td class="cl-39995ee0"><p class="cl-3999560c"><span class="cl-39981706">-6.224</span></p></td><td class="cl-39995ee0"><p class="cl-3999560c"><span class="cl-39981706">0.0000</span></p></td><td class="cl-39995ee2"><p class="cl-39995602"><span class="cl-39981706">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995ed6"><p class="cl-39995602"><span class="cl-39981706">provinceQuebec</span></p></td><td class="cl-39995ed7"><p class="cl-3999560c"><span class="cl-39981706">-2.485</span></p></td><td class="cl-39995ed8"><p class="cl-3999560c"><span class="cl-39981706">0.230</span></p></td><td class="cl-39995ed7"><p class="cl-3999560c"><span class="cl-39981706">-10.784</span></p></td><td class="cl-39995ed7"><p class="cl-3999560c"><span class="cl-39981706">0.0000</span></p></td><td class="cl-39995ed9"><p class="cl-39995602"><span class="cl-39981706">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995ed6"><p class="cl-39995602"><span class="cl-39981706">provinceSaskatchewan</span></p></td><td class="cl-39995ed7"><p class="cl-3999560c"><span class="cl-39981706">-0.364</span></p></td><td class="cl-39995ed8"><p class="cl-3999560c"><span class="cl-39981706">0.255</span></p></td><td class="cl-39995ed7"><p class="cl-3999560c"><span class="cl-39981706">-1.430</span></p></td><td class="cl-39995ed7"><p class="cl-3999560c"><span class="cl-39981706">0.1528</span></p></td><td class="cl-39995ed9"><p class="cl-39995602"><span class="cl-39981706">   </span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995f02"><p class="cl-39995602"><span class="cl-39981706">educationCollege, CEGEP or other non-university certificate or di...</span></p></td><td class="cl-39995f08"><p class="cl-3999560c"><span class="cl-39981706">0.558</span></p></td><td class="cl-39995f09"><p class="cl-3999560c"><span class="cl-39981706">0.132</span></p></td><td class="cl-39995f08"><p class="cl-3999560c"><span class="cl-39981706">4.212</span></p></td><td class="cl-39995f08"><p class="cl-3999560c"><span class="cl-39981706">0.0000</span></p></td><td class="cl-39995f0a"><p class="cl-39995602"><span class="cl-39981706">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995f0b"><p class="cl-39995602"><span class="cl-39981706">educationHigh school diploma or a high school equivalency certificate</span></p></td><td class="cl-39995f0c"><p class="cl-3999560c"><span class="cl-39981706">0.497</span></p></td><td class="cl-39995f0d"><p class="cl-3999560c"><span class="cl-39981706">0.161</span></p></td><td class="cl-39995f0c"><p class="cl-3999560c"><span class="cl-39981706">3.080</span></p></td><td class="cl-39995f0c"><p class="cl-3999560c"><span class="cl-39981706">0.0021</span></p></td><td class="cl-39995f0e"><p class="cl-39995602"><span class="cl-39981706"> **</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995f12"><p class="cl-39995602"><span class="cl-39981706">educationLess than high school diploma or its equivalent</span></p></td><td class="cl-39995f13"><p class="cl-3999560c"><span class="cl-39981706">0.807</span></p></td><td class="cl-39995f14"><p class="cl-3999560c"><span class="cl-39981706">0.234</span></p></td><td class="cl-39995f13"><p class="cl-3999560c"><span class="cl-39981706">3.453</span></p></td><td class="cl-39995f13"><p class="cl-3999560c"><span class="cl-39981706">0.0006</span></p></td><td class="cl-39995f15"><p class="cl-39995602"><span class="cl-39981706">***</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995f1c"><p class="cl-39995602"><span class="cl-39981706">educationUniversity certificate or diploma below the bachelor's level</span></p></td><td class="cl-39995f1d"><p class="cl-3999560c"><span class="cl-39981706">0.237</span></p></td><td class="cl-39995f1e"><p class="cl-3999560c"><span class="cl-39981706">0.188</span></p></td><td class="cl-39995f1d"><p class="cl-3999560c"><span class="cl-39981706">1.264</span></p></td><td class="cl-39995f1d"><p class="cl-3999560c"><span class="cl-39981706">0.2062</span></p></td><td class="cl-39995f1f"><p class="cl-39995602"><span class="cl-39981706">   </span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-39995f20"><p class="cl-39995602"><span class="cl-39981706">educationUniversity certificate, diploma or degree above the bach...</span></p></td><td class="cl-39995f21"><p class="cl-3999560c"><span class="cl-39981706">-0.487</span></p></td><td class="cl-39995f26"><p class="cl-3999560c"><span class="cl-39981706">0.164</span></p></td><td class="cl-39995f21"><p class="cl-3999560c"><span class="cl-39981706">-2.977</span></p></td><td class="cl-39995f21"><p class="cl-3999560c"><span class="cl-39981706">0.0029</span></p></td><td class="cl-39995f27"><p class="cl-39995602"><span class="cl-39981706"> **</span></p></td></tr></tbody><tfoot><tr style="overflow-wrap:break-word;"><td  colspan="6"class="cl-39995f28"><p class="cl-3999560c"><span class="cl-39981710">Signif. codes: 0 &lt;= '***' &lt; 0.001 &lt; '**' &lt; 0.01 &lt; '*' &lt; 0.05</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  colspan="6"class="cl-39995f2c"><p class="cl-39995602"><span class="cl-39981706"> </span></p></td></tr><tr style="overflow-wrap:break-word;"><td  colspan="6"class="cl-39995f2c"><p class="cl-39995602"><span class="cl-39981706">(Dispersion parameter for binomial family taken to be 1)</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  colspan="6"class="cl-39995f2c"><p class="cl-39995602"><span class="cl-39981706">Null deviance: 2857 on 2212 degrees of freedom</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  colspan="6"class="cl-39995f2c"><p class="cl-39995602"><span class="cl-39981706">Residual deviance: 2488 on 2189 degrees of freedom</span></p></td></tr></tfoot></table></div>

When we ran our model on each of the political parties that we were
investigating by post-stratification method, these were the results we
got:

``` r
# Here we will perform the post-stratification calculation
#new data with different bins in census data(I put people in different bins based on predictors(age, sex, place_birth_canada, province, education)),
#Then,we calculate proportion of each cell relative to the whole population(n()/18730)
#Lastly, we use estimate to multiply by the proportion and sum them to get the final answer

#liberal
s_data <- census_data %>% group_by(age, sex, place_birth_canada, province, education, hh_size,
                                   income_family) %>%
  summarise(num_people = n(), cell_prop = n()/nrow(census_data))

s_data$estimate_lib <- model_liberal %>% predict(newdata = s_data, type= "response")

s_data <- s_data %>% mutate(lib_pre_prop = estimate_lib * cell_prop)
r1<-sum(s_data$lib_pre_prop)

#conservative
s_data$estimate_cons <- model_cons %>% predict(newdata = s_data, type= "response")

s_data <- s_data %>% mutate(cons_pre_prop = estimate_cons * cell_prop)

r2<-sum(s_data$cons_pre_prop)
```

``` r
results <- data.frame(Political_Party = c("Liberal Party", "Conservative Party"), 
                    Probability_Winning_Election = c(r1, r2))

names(results)[1] <- "Polical Party"
names(results)[2] <- "Probability of Winning Election"
knitr::kable(results, caption = "Probability of each party winning the election")
```

| Polical Party      | Probability of Winning Election |
|:-------------------|--------------------------------:|
| Liberal Party      |                       0.3468515 |
| Conservative Party |                       0.3563814 |

Probability of each party winning the election

Based on our logistic model, we were able to get the probability of each
political party winning the election in 2025. The logistic model
indicated that there was a 34.7% probability of the Liberal party
winning the election and a 35.6% probability of the Conservative party
winning the election. Based on our findings, our data indicates that the
Conservative party will win the election in 2025.

However, we must acknowledge that the values for probabilities are only
estimates. This is because our model does not take into account the
people who haven’t voted. There is still a possibility for the Liberal
party to win the election if more people who haven’t voted, end up
voting for the Liberal party. Moreover, we used our model only on two of
the leading political parties and did not take into account any other
political parties. Also, we didn’t have control on the data collection.
Thus, there might be some hidden groups that we didn’t take into
account.

## Conclusions

Using a logistic model and post-stratification analysis, we found that
the **Conservative Party has a 35.6%** probability of winning the
election in 2025, and the **Liberal Party has a 34.7%** probability of
winning the election. This indicates that based on our model and
analysis, the Conservative Party will win the election in 2025.

However, we need to address that our analysis has certain limitations
that we need to take into account. Firstly, we don’t have any control on
the survey data ‘2019 phone survey’. This means that we aren’t aware of
how the data was collected and how reliable the data is. Moreover, there
is always a possibility of a hidden group existing in data collection
that we may have not come across. We also did not account for the fact
that certain individuals would have chosen not to answer the survey or
didn’t decide which party they wanted to vote for. Lastly, our model and
analysis was based on only the Liberal party and the Conservative Party.
It does not take into account any other political parties in the
election.

Therefore, while our model predicted that the Conservative Party will
win, there is still a possibility of the Liberal Party or some other
political party winning if we take into account the limitations
addressed above.

In future analyses, we would suggest using other models to ensure
reliability in our analysis and also make sure that we are taking more
factors into account when predicting the election outcome in 2025.

## Bibliography

1.  Grolemund, G. (2014, July 16) *Introduction to R Markdown*. RStudio.
    <https://rmarkdown.rstudio.com/articles_intro.html>. (Last Accessed
    December 1, 2022)

2.  Dekking, F. M., et al. (2005) *A Modern Introduction to Probability
    and Statistics: Understanding why and how.* Springer Science &
    Business Media. (Last Accessed December 1, 2022)

3.  Allaire, J.J., et. el. *References: Introduction to R Markdown*.
    RStudio. <https://rmarkdown.rstudio.com/docs/>. (Last Accessed
    December 1, 2022)

4.  Statistics Canada. (2017). *General Social Survey*.
    <https://sda-artsci-utoronto-ca.myaccess.library.utoronto.ca/sdaweb/analysis/?dataset=gss31>.
    Dataset. (Last Accessed December 1, 2022)

5.  Stephenson, L. B., Harell, A., Rubenson, D., & Loewen, P. J. (2020,
    August 11). *Canadian Election Study, 2019, Phone Survey. Retrieved
    from Election.* Dataset. (Last Accessed December 1, 2022)

6.  Caetano, S. (2020). *Introduction to Logistic Regression*. Lecture.
    (Last Accessed December 1, 2022)

7.  *Statistical Language - What Are Variables? Australian Bureau of
    Statistics.*.
    <https://www.abs.gov.au/websitedbs/D3310114.nsf/home/statistical+language+-+what+are+variables>.(Last
    Accessed December 1, 2022)

8.  Karabiber, F. *Binary variable. Learn Data Science - Tutorials,
    Books, Courses, and More*.
    <https://www.learndatasci.com/glossary/binary-variable/> (Last
    Accessed December 1, 2022)

9.  Caetano, S. (2022). *Multilevel Regression & Poststratification*.
    Lecture. (Last Accessed December 1, 2022)

10. Karabiber, F. *Binary variable. Learn Data Science - Tutorials,
    Books, Courses, and More*.
    <https://www.learndatasci.com/glossary/binary-variable/> (Last
    Accessed December 1, 2022)

11. *knitr::kable function*. R
    Studio.<https://rmarkdown.rstudio.com/lesson-7.html> (Last Accessed
    December 1, 2022)

12. Bolton, L. (2020). *Bar-chart-tips-and-tricks*. GitHub.
    <https://github.com/asidianya/Data-Visualization-/blob/main/Bar-chart-tips-and-tricks.pdf>
    (Last Accessed December 1, 2022)

13. *Create Elegant Data Visualisations Using the Grammar of Graphics*.
    RStudio. <https://ggplot2.tidyverse.org/> (Last Accessed December 1,
    2022)

14. David Robinson, Alex Hayes and Simon Couch (2022). broom: Convert
    Statistical Objects into Tidy Tibbles. R package version 0.8.0.
    [https://CRAN.R-project.org/package=broom](Last%20Accessed%20December%201,%202022)

15. David Gohel (2022). flextable: Functions for Tabular Reporting. R
    package version 0.7.2.
    [https://CRAN.R-project.org/package=flextable](Last%20Accessed%20December%201,%202022)

16. Kirill Müller and Hadley Wickham (2022). tibble: Simple Data Frames.
    R package version 3.1.7.
    [https://CRAN.R-project.org/package=tibble](Last%20Accessed%20December%201,%202022)

17. H. Wickham. ggplot2: Elegant Graphics for Data Analysis.
    Springer-Verlag New York, 2016.(Last Accessed December 1, 2022)

18. Alboukadel Kassambara (2022). ggpubr: ‘ggplot2’ Based Publication
    Ready Plots. R package version 0.5.0.
    <https://CRAN.R-project.org/package=ggpubr>(Last Accessed December
    1, 2022)

19. Yihui Xie (2022). knitr: A General-Purpose Package for Dynamic
    Report Generation in R. Rpackage version 1.39. (Last Accessed
    December 1, 2022)

20. Paul A. Hodgetts and Rohan Alexander (2021). cesR: Access the
    Canadian Election Study Datasets. R package version 0.1.0.
    <https://CRAN.R-project.org/package=cesR> (Last Accessed December 1,
    2022)

21. *2021 post-federal election analysis*. Grant Thornton LLP Canada.
    <https://www.grantthornton.ca/insights/2021-post-federal-election-analysis/>
    (Last Accessed December 1, 2022)

22. John Fox and Sanford Weisberg (2019). *An {R} Companion to Applied
    Regression, Third Edition*. Thousand Oaks CA: Sage.
    <https://socialsciences.mcmaster.ca/jfox/Books/Companion/> (Last
    Accessed December 1, 2022)

23. Khan, K. (2021). *Confidence Intervals and Hypothesis Testing*.
    Lecture. (Last Accessed December 1, 2022)

24. Khan, M. (2022). *Model Selection*. Lecture. (Last Accessed December
    1, 2022)
