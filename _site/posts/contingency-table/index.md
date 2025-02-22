# Using table() in R
Rick Dean
2025-02-02

## Introduction

The following are notes and R script from the R-bloggers article [R
Function of the Day:
table](https://www.r-bloggers.com/2009/09/r-function-of-the-day-table/)
by erikr.

> The table function *table()* is very basic, but essential, function to
> master while performing interactive data analyses. It simply creates
> tabular results of categorical variables (cross-tabulation). However,
> when combined with the powers of logical expressions in R, you can
> gain even more insights into your data, including identifying
> potential problems.

*table()* computes a cross tabulation of counts of each combination of
factor levels

## Setup

``` r
set.seed(456723)

library(data.table)
library(kableExtra)

clinical_dt <- data.table(
  patient = 1:100,
  age = stats::rnorm(100, mean = 60, sd = 6),
  weight = stats::rnorm(100, mean = 140, sd = 20),
  treatment = gl(2,50,
                 labels = c('treatment','control')),
  center = sample(paste("Center", LETTERS[1:5]),100, replace = TRUE)
)

# Set some weight to NA (missing):
is.na(clinical_dt$weight) <- sample(x = 1:100, size = 20)
summary(clinical_dt)
```

        patient            age            weight           treatment 
     Min.   :  1.00   Min.   :48.58   Min.   : 98.93   treatment:50  
     1st Qu.: 25.75   1st Qu.:58.00   1st Qu.:127.59   control  :50  
     Median : 50.50   Median :61.02   Median :141.65                 
     Mean   : 50.50   Mean   :61.65   Mean   :142.03                 
     3rd Qu.: 75.25   3rd Qu.:64.77   3rd Qu.:155.14                 
     Max.   :100.00   Max.   :79.36   Max.   :184.42                 
                                      NA's   :20                     
        center         
     Length:100        
     Class :character  
     Mode  :character  
                       
                       
                       
                       

## How many subjects in each center (margin)

``` r
(center_count_tbl <- table(clinical_dt$center))
```


    Center A Center B Center C Center D Center E 
          23       16       20       22       19 

## How many subjects in each treatment (margin)

``` r
(treatment_count_tbl <- table(clinical_dt$treatment))
```


    treatment   control 
           50        50 

## Cross tab of treatment vs centers

``` r
(treatment_center_tbl <- table(
  clinical_dt$treatment,
  clinical_dt$center,
  dnn = c("Treatment", "Center")
))
```

               Center
    Treatment   Center A Center B Center C Center D Center E
      treatment        9        7       16       11        7
      control         14        9        4       11       12

## tables can be converted to *data.table*

``` r
treatment_center_dt <- data.table::as.data.table(treatment_center_tbl)
kableExtra::kable(treatment_center_dt)
```

| Treatment | Center   |   N |
|:----------|:---------|----:|
| treatment | Center A |   9 |
| control   | Center A |  14 |
| treatment | Center B |   7 |
| control   | Center B |   9 |
| treatment | Center C |  16 |
| control   | Center C |   4 |
| treatment | Center D |  11 |
| control   | Center D |  11 |
| treatment | Center E |   7 |
| control   | Center E |  12 |

## Using logical operators with *table()*

### We want to know how many subjects are under the age of 60 in a clinical trial

``` r
(under_60_tbl <- table(
  clinical_dt$age < 60,
  dnn = "Age<60"
))
```

    Age<60
    FALSE  TRUE 
       63    37 

### We want to know how many subjects are under the age of 60 across treatments in a clinical trial

``` r
(treatments_under_60_tbl <- table(
  clinical_dt$treatment,
  clinical_dt$age < 60,
  dnn = c("Treatment", "Age<60")))
```

               Age<60
    Treatment   FALSE TRUE
      treatment    33   17
      control      30   20

### Count the subjects under 60 and weight greater than 160 along with NA’s

``` r
(under_60_greater_160_tbl <- table(
  clinical_dt$age < 60,
  clinical_dt$weight > 160,
  useNA = "always",
  dnn = c("Age<60","Wt>160")
))
```

           Wt>160
    Age<60  FALSE TRUE <NA>
      FALSE    37   11   15
      TRUE     26    6    5
      <NA>      0    0    0

### Which center has the most subjects with a missing value for weight in the clinical trial?

``` r
(center_with_missing_tbl <- table(
  clinical_dt$center,
  is.na(clinical_dt$weight),
  dnn = c("Center", "NA Weight")
))
```

              NA Weight
    Center     FALSE TRUE
      Center A    18    5
      Center B    12    4
      Center C    17    3
      Center D    18    4
      Center E    15    4

### Sort the center_with_missing table

Note the use of array index for “dnn”

``` r
(center_with_missing_sorted_tbl <- sort(table(
  clinical_dt$center,
  is.na(clinical_dt$weight),
  dnn = "Center")[,2],
  decreasing = TRUE
))
```

    Center A Center B Center D Center E Center C 
           5        4        4        4        3 
