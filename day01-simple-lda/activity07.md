Activity 7 - Linear Discriminant Analysis
================

## Task 3

``` r
resume <- readr::read_csv("https://www.openintro.org/data/csv/resume.csv")
```

    ## Rows: 4870 Columns: 30
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (10): job_city, job_industry, job_type, job_ownership, job_req_min_exper...
    ## dbl (20): job_ad_id, job_fed_contractor, job_equal_opp_employer, job_req_any...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

## Question 1: The plot on the right satisfies the assumption. The plot on the left is skewed right while the plot on the right is a normal distribution.

## Task 4

``` r
# Convert received_callback to a factor with more informative labels
resume <- resume %>% 
  mutate(received_callback = factor(received_callback, labels = c("No", "Yes")))

# LDA
library(discrim)
```

    ## 
    ## Attaching package: 'discrim'

    ## The following object is masked from 'package:dials':
    ## 
    ##     smoothness

``` r
lda_years <- discrim_linear() %>% 
  set_mode("classification") %>% 
  set_engine("MASS") %>% 
  fit(received_callback ~ log(years_experience), data = resume)

lda_years
```

    ## parsnip model object
    ## 
    ## Call:
    ## lda(received_callback ~ log(years_experience), data = data)
    ## 
    ## Prior probabilities of groups:
    ##         No        Yes 
    ## 0.91950719 0.08049281 
    ## 
    ## Group means:
    ##     log(years_experience)
    ## No               1.867135
    ## Yes              1.998715
    ## 
    ## Coefficients of linear discriminants:
    ##                            LD1
    ## log(years_experience) 1.638023

## Question 2

The values for Group Means for yes and no are pretty similar.

## Question 3

The peaks for received\_callback yes and no are at the locations for the
group means of yes and no.

## Task 5

``` r
predict(lda_years, new_data = resume, type = "prob")
```

    ## # A tibble: 4,870 × 2
    ##    .pred_No .pred_Yes
    ##       <dbl>     <dbl>
    ##  1    0.923    0.0769
    ##  2    0.923    0.0769
    ##  3    0.923    0.0769
    ##  4    0.923    0.0769
    ##  5    0.884    0.116 
    ##  6    0.923    0.0769
    ##  7    0.928    0.0724
    ##  8    0.885    0.115 
    ##  9    0.939    0.0612
    ## 10    0.923    0.0769
    ## # … with 4,860 more rows

``` r
augment(lda_years, new_data = resume) %>% 
  conf_mat(truth = received_callback, estimate = .pred_class)
```

    ##           Truth
    ## Prediction   No  Yes
    ##        No  4478  392
    ##        Yes    0    0

## Question 4

The model never predicted yes. I think it happened because it is very
uncommon to receive a callback so it did not predict it.

``` r
augment(lda_years, new_data = resume) %>% 
  accuracy(truth = received_callback, estimate = .pred_class)
```

    ## # A tibble: 1 × 3
    ##   .metric  .estimator .estimate
    ##   <chr>    <chr>          <dbl>
    ## 1 accuracy binary         0.920
