`unnest_wider()` issue
================

``` r
library(tidyverse)

df <- read_rds("pesky_df.rds")

str(df)
```

    ## Classes 'tbl_df', 'tbl' and 'data.frame':    1 obs. of  6 variables:
    ##  $ fips     : chr "45001"
    ##  $ name     : chr "Abbeville"
    ##  $ votes    : int 1957
    ##  $ reporting: int 15
    ##  $ precincts: int 15
    ##  $ results  :List of 12
    ##   ..$ bidenj    : int 1129
    ##   ..$ steyert   : int 312
    ##   ..$ sandersb  : int 286
    ##   ..$ buttigiegp: int 80
    ##   ..$ warrene   : int 60
    ##   ..$ klobuchara: int 42
    ##   ..$ gabbardt  : int 26
    ##   ..$ bookerc   : int 6
    ##   ..$ yanga     : int 6
    ##   ..$ bennetm   : int 5
    ##   ..$ delaneyj  : int 3
    ##   ..$ patrickd  : int 2

`results` is a named list with an integer scalar value for each named
element. This should be useable by `unnest_wider()` to create column
names from the list names.

``` r
df %>% 
  unnest_wider(results)
```

I receive the following error.

> Error in `[[<-.data.frame`(`*tmp*`, col, value = list(bidenj = list(…1
> = 1129L), : replacement has 12 rows, data has 1

This tells me that for some reason `unnest_wider()` was attempting to
make one row per observation.

I tried reproducing this error, but cannot. It is an identical problem
to the documented example except without NAs.

``` r
ex_df <- tibble(
  x = 1:2,
  y = list(c(a = 1, b = 2), c(a = 10, b = 11, c = 12))
)

ex_df %>% unnest_wider(y)
```

    ## # A tibble: 2 x 4
    ##       x     a     b     c
    ##   <int> <dbl> <dbl> <dbl>
    ## 1     1     1     2    NA
    ## 2     2    10    11    12

The solution was to `unlist()` then `list()` the column back together.
When this is done `unnest_wider()` works.

``` r
df %>% 
  mutate(results = list(unlist(results))) %>% 
  unnest_wider(results)
```

    ## # A tibble: 1 x 17
    ##   fips  name  votes reporting precincts bidenj steyert sandersb buttigiegp
    ##   <chr> <chr> <int>     <int>     <int>  <int>   <int>    <int>      <int>
    ## 1 45001 Abbe…  1957        15        15   1129     312      286         80
    ## # … with 8 more variables: warrene <int>, klobuchara <int>,
    ## #   gabbardt <int>, bookerc <int>, yanga <int>, bennetm <int>,
    ## #   delaneyj <int>, patrickd <int>
