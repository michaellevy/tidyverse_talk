tidyverse
================
Michael Levy, Prepared for the Davis R-Users' Group
October 13, 2016

What is the tidyverse?
----------------------

~~Hadleyverse~~

The tidyverse is a suite of R tools that follow a tidy philosophy:

### Tidy data

Put data in data frames

-   Each type of observation gets a data frame
-   Each variable gets a column
-   Each observation gets a row

### Tidy APIs

Functions should be consistent and easily (human) readable

-   Take one step at a time
-   Connect simple steps with the pipe
-   Referential transparency

### Okay but really, what is it?

Suite of ~20 packages that provide consistent, user-friendly, smart-default tools to do most of what most people do in R.

-   Core packages: ggplot2, dplyr, tidyr, readr, purrr, tibble
-   Specialized data manipulation: hms, stringr, lubridate, forcats
-   Data import: DBI, haven, httr, jsonlite, readxl, rvest, xml2
-   Modeling: modelr, broom

`install.packages(tidyverse)` installs all of the above packages.

`library(tidyverse)` attaches only the core packages.

Why tidyverse?
--------------

-   Consistency
    -   e.g. All `stringr` functions take string first
    -   e.g. Many functions take data.frame first -&gt; piping
        -   Faster to write
        -   Easier to read
    -   Tidy data: Imposes good practices
    -   Type specificity
-   You probably use some of it already. Synergize.
-   Implements simple solutions to common problems (e.g. `purrr::transpose`)
-   Smarter defaults
    -   e.g. `utils::write.csv(row.names = FALSE)` = `readr::write_csv()`
-   Runs fast (thanks to `Rcpp`)
-   Interfaces well with other tools (e.g. Spark with `dplyr` via `sparklyr`)

`tibble`
--------

> A modern reimagining of data frames.

``` r
library(tidyverse)
```

``` r
tdf = tibble(x = 1:1e4, y = rnorm(1e4))  # == data_frame(x = 1:1e4, y = rnorm(1e4))
class(tdf)
```

    ## [1] "tbl_df"     "tbl"        "data.frame"

Tibbles print politely.

``` r
tdf
```

    ## # A tibble: 10,000 × 2
    ##        x          y
    ##    <int>      <dbl>
    ## 1      1  1.7307583
    ## 2      2  1.4246209
    ## 3      3  0.2762850
    ## 4      4  1.9267297
    ## 5      5  1.8189041
    ## 6      6  1.1574624
    ## 7      7  0.1248573
    ## 8      8 -0.1066158
    ## 9      9 -0.7412011
    ## 10    10 -0.9383221
    ## # ... with 9,990 more rows

-   Can customize print methods with `print(tdf, n = rows, width = cols)`

-   Set default with `options(tibble.print_max = rows, tibble.width = cols)`

Tibbles have some convenient and consistent defaults that are different from base R data.frames.

#### strings as factors

``` r
dfs = list(
  df = data.frame(abc = letters[1:3], xyz = letters[24:26]),
  tbl = data_frame(abc = letters[1:3], xyz = letters[24:26])
)
sapply(dfs, function(d) class(d$abc))
```

    ##          df         tbl 
    ##    "factor" "character"

#### partial matching of names

``` r
sapply(dfs, function(d) d$a)
```

    ## Warning: Unknown column 'a'

    ## $df
    ## [1] a b c
    ## Levels: a b c
    ## 
    ## $tbl
    ## NULL

#### type consistency

``` r
sapply(dfs, function(d) class(d[, "abc"]))
```

    ## $df
    ## [1] "factor"
    ## 
    ## $tbl
    ## [1] "tbl_df"     "tbl"        "data.frame"

Note that tidyverse import functions (e.g. `readr::read_csv`) default to tibbles and that *this can break existing code*.

#### List-columns!

``` r
tibble(ints = 1:5,
       powers = lapply(1:5, function(x) x^(1:x)))
```

    ## # A tibble: 5 × 2
    ##    ints    powers
    ##   <int>    <list>
    ## 1     1 <dbl [1]>
    ## 2     2 <dbl [2]>
    ## 3     3 <dbl [3]>
    ## 4     4 <dbl [4]>
    ## 5     5 <dbl [5]>

The pipe `%>%`
--------------

Sends the output of the LHS function to the first argument of the RHS function.

``` r
sum(1:8) %>%
  sqrt()
```

    ## [1] 6

`dplyr`
-------

Common data(frame) manipulation tasks.

Four core "verbs": filter, select, arrange, group\_by + summarize, plus many more convenience functions.

``` r
library(ggplot2movies)
str(movies)
```

    ## Classes 'tbl_df', 'tbl' and 'data.frame':    58788 obs. of  24 variables:
    ##  $ title      : chr  "$" "$1000 a Touchdown" "$21 a Day Once a Month" "$40,000" ...
    ##  $ year       : int  1971 1939 1941 1996 1975 2000 2002 2002 1987 1917 ...
    ##  $ length     : int  121 71 7 70 71 91 93 25 97 61 ...
    ##  $ budget     : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ rating     : num  6.4 6 8.2 8.2 3.4 4.3 5.3 6.7 6.6 6 ...
    ##  $ votes      : int  348 20 5 6 17 45 200 24 18 51 ...
    ##  $ r1         : num  4.5 0 0 14.5 24.5 4.5 4.5 4.5 4.5 4.5 ...
    ##  $ r2         : num  4.5 14.5 0 0 4.5 4.5 0 4.5 4.5 0 ...
    ##  $ r3         : num  4.5 4.5 0 0 0 4.5 4.5 4.5 4.5 4.5 ...
    ##  $ r4         : num  4.5 24.5 0 0 14.5 14.5 4.5 4.5 0 4.5 ...
    ##  $ r5         : num  14.5 14.5 0 0 14.5 14.5 24.5 4.5 0 4.5 ...
    ##  $ r6         : num  24.5 14.5 24.5 0 4.5 14.5 24.5 14.5 0 44.5 ...
    ##  $ r7         : num  24.5 14.5 0 0 0 4.5 14.5 14.5 34.5 14.5 ...
    ##  $ r8         : num  14.5 4.5 44.5 0 0 4.5 4.5 14.5 14.5 4.5 ...
    ##  $ r9         : num  4.5 4.5 24.5 34.5 0 14.5 4.5 4.5 4.5 4.5 ...
    ##  $ r10        : num  4.5 14.5 24.5 45.5 24.5 14.5 14.5 14.5 24.5 4.5 ...
    ##  $ mpaa       : chr  "" "" "" "" ...
    ##  $ Action     : int  0 0 0 0 0 0 1 0 0 0 ...
    ##  $ Animation  : int  0 0 1 0 0 0 0 0 0 0 ...
    ##  $ Comedy     : int  1 1 0 1 0 0 0 0 0 0 ...
    ##  $ Drama      : int  1 0 0 0 0 1 1 0 1 0 ...
    ##  $ Documentary: int  0 0 0 0 0 0 0 1 0 0 ...
    ##  $ Romance    : int  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ Short      : int  0 0 1 0 0 0 0 1 0 0 ...

``` r
filter(movies, length > 360)
```

    ## # A tibble: 21 × 24
    ##                                               title  year length  budget
    ##                                               <chr> <int>  <int>   <int>
    ## 1                         Commune (Paris, 1871), La  2000    555      NA
    ## 2                            Cure for Insomnia, The  1987   5220      NA
    ## 3             Ebolusyon ng isang pamilyang pilipino  2004    647      NA
    ## 4                                            Empire  1964    485      NA
    ## 5                                Farmer's Wife, The  1998    390      NA
    ## 6                                     Foolish Wives  1922    384 1100000
    ## 7                                        Four Stars  1967   1100      NA
    ## 8                 Hitler - ein Film aus Deutschland  1978    407      NA
    ## 9                               Imitation of Christ  1967    480      NA
    ## 10 Longest Most Meaningless Movie in the World, The  1970   2880      NA
    ## # ... with 11 more rows, and 20 more variables: rating <dbl>, votes <int>,
    ## #   r1 <dbl>, r2 <dbl>, r3 <dbl>, r4 <dbl>, r5 <dbl>, r6 <dbl>, r7 <dbl>,
    ## #   r8 <dbl>, r9 <dbl>, r10 <dbl>, mpaa <chr>, Action <int>,
    ## #   Animation <int>, Comedy <int>, Drama <int>, Documentary <int>,
    ## #   Romance <int>, Short <int>

``` r
filter(movies, length > 360) %>%
  select(title, rating, votes)
```

    ## # A tibble: 21 × 3
    ##                                               title rating votes
    ##                                               <chr>  <dbl> <int>
    ## 1                         Commune (Paris, 1871), La    7.8    33
    ## 2                            Cure for Insomnia, The    3.8    59
    ## 3             Ebolusyon ng isang pamilyang pilipino    8.4     5
    ## 4                                            Empire    5.5    46
    ## 5                                Farmer's Wife, The    8.5    52
    ## 6                                     Foolish Wives    7.6   191
    ## 7                                        Four Stars    3.0    12
    ## 8                 Hitler - ein Film aus Deutschland    9.0    70
    ## 9                               Imitation of Christ    4.4     5
    ## 10 Longest Most Meaningless Movie in the World, The    6.4    15
    ## # ... with 11 more rows

``` r
filter(movies, Animation == 1, votes > 1000) %>%
  select(title, rating) %>%
  arrange(desc(rating))
```

    ## # A tibble: 135 × 2
    ##                                   title rating
    ##                                   <chr>  <dbl>
    ## 1         Sen to Chihiro no kamikakushi    8.6
    ## 2                            Duck Amuck    8.4
    ## 3  Wallace & Gromit: The Wrong Trousers    8.4
    ## 4                          Finding Nemo    8.3
    ## 5                        Hotaru no haka    8.3
    ## 6                      Incredibles, The    8.3
    ## 7                         Mononoke-hime    8.3
    ## 8                    What's Opera, Doc?    8.3
    ## 9                               Vincent    8.2
    ## 10      Wallace & Gromit: A Close Shave    8.2
    ## # ... with 125 more rows

`summarize` makes `aggregate` and `tapply` functionality easier, and the output is always a data frame.

``` r
filter(movies, mpaa != "") %>%
  group_by(year, mpaa) %>%
  summarize(avg_budget = mean(budget, na.rm = TRUE),
            avg_rating = mean(rating, na.rm = TRUE)) %>%
  arrange(desc(year), mpaa)
```

    ## Source: local data frame [128 x 4]
    ## Groups: year [54]
    ## 
    ##     year  mpaa avg_budget avg_rating
    ##    <int> <chr>      <dbl>      <dbl>
    ## 1   2005 NC-17        NaN   6.700000
    ## 2   2005    PG   45857143   5.733333
    ## 3   2005 PG-13   42269333   5.326087
    ## 4   2005     R   24305882   4.595833
    ## 5   2004    PG   45126852   5.847619
    ## 6   2004 PG-13   46288254   6.080180
    ## 7   2004     R   19548519   5.848469
    ## 8   2003    PG   37057692   5.897674
    ## 9   2003 PG-13   46269491   5.949038
    ## 10  2003     R   21915505   5.702273
    ## # ... with 118 more rows

`count` for frequency tables. Note the consistent API and easy readability vs. `table`.

``` r
filter(movies, mpaa != "") %>%
  count(year, mpaa, Animation, sort = TRUE)
```

    ## Source: local data frame [156 x 4]
    ## Groups: year, mpaa [128]
    ## 
    ##     year  mpaa Animation     n
    ##    <int> <chr>     <int> <int>
    ## 1   1999     R         0   366
    ## 2   2001     R         0   355
    ## 3   2002     R         0   343
    ## 4   2000     R         0   341
    ## 5   1998     R         0   335
    ## 6   1997     R         0   325
    ## 7   1996     R         0   310
    ## 8   1995     R         0   293
    ## 9   2003     R         0   264
    ## 10  2004     R         0   196
    ## # ... with 146 more rows

``` r
basetab = with(movies[movies$mpaa != "", ], table(year, mpaa, Animation))
basetab[1:5, , ]
```

    ## , , Animation = 0
    ## 
    ##       mpaa
    ## year   NC-17 PG PG-13 R
    ##   1934     0  1     0 0
    ##   1938     0  1     0 0
    ##   1945     0  0     1 0
    ##   1946     0  1     0 0
    ##   1951     0  2     0 0
    ## 
    ## , , Animation = 1
    ## 
    ##       mpaa
    ## year   NC-17 PG PG-13 R
    ##   1934     0  0     0 0
    ##   1938     0  0     0 0
    ##   1945     0  0     0 0
    ##   1946     0  0     0 0
    ##   1951     0  0     0 0

### joins

`dplyr` also does multi-table joins and can connect to various types of databases.

``` r
t1 = data_frame(alpha = letters[1:6], num = 1:6)
t2 = data_frame(alpha = letters[4:10], num = 4:10)
full_join(t1, t2, by = "alpha", suffix = c("_t1", "_t2"))
```

    ## # A tibble: 10 × 3
    ##    alpha num_t1 num_t2
    ##    <chr>  <int>  <int>
    ## 1      a      1     NA
    ## 2      b      2     NA
    ## 3      c      3     NA
    ## 4      d      4      4
    ## 5      e      5      5
    ## 6      f      6      6
    ## 7      g     NA      7
    ## 8      h     NA      8
    ## 9      i     NA      9
    ## 10     j     NA     10

Super-secret pro-tip: You can `group_by` %&gt;% `mutate` to accomplish a summarize + join

``` r
data_frame(group = sample(letters[1:3], 10, replace = TRUE),
           value = rnorm(10)) %>%
  group_by(group) %>%
  mutate(group_average = mean(value))
```

    ## Source: local data frame [10 x 3]
    ## Groups: group [3]
    ## 
    ##    group       value group_average
    ##    <chr>       <dbl>         <dbl>
    ## 1      b -0.07744649   -0.61076927
    ## 2      c  0.11825771    0.08865786
    ## 3      c  0.58866540    0.08865786
    ## 4      c  0.27584554    0.08865786
    ## 5      b -0.80187845   -0.61076927
    ## 6      c -0.33398635    0.08865786
    ## 7      c -0.20549302    0.08865786
    ## 8      b -0.95298286   -0.61076927
    ## 9      a -0.48253785   -0.68985472
    ## 10     a -0.89717159   -0.68985472

`tidyr`
-------

Latest generation of `reshape`. `gather` to make wide table long, `spread` to make long tables wide.

``` r
who  # Tuberculosis data from the WHO
```

    ## # A tibble: 7,240 × 60
    ##        country  iso2  iso3  year new_sp_m014 new_sp_m1524 new_sp_m2534
    ##          <chr> <chr> <chr> <int>       <int>        <int>        <int>
    ## 1  Afghanistan    AF   AFG  1980          NA           NA           NA
    ## 2  Afghanistan    AF   AFG  1981          NA           NA           NA
    ## 3  Afghanistan    AF   AFG  1982          NA           NA           NA
    ## 4  Afghanistan    AF   AFG  1983          NA           NA           NA
    ## 5  Afghanistan    AF   AFG  1984          NA           NA           NA
    ## 6  Afghanistan    AF   AFG  1985          NA           NA           NA
    ## 7  Afghanistan    AF   AFG  1986          NA           NA           NA
    ## 8  Afghanistan    AF   AFG  1987          NA           NA           NA
    ## 9  Afghanistan    AF   AFG  1988          NA           NA           NA
    ## 10 Afghanistan    AF   AFG  1989          NA           NA           NA
    ## # ... with 7,230 more rows, and 53 more variables: new_sp_m3544 <int>,
    ## #   new_sp_m4554 <int>, new_sp_m5564 <int>, new_sp_m65 <int>,
    ## #   new_sp_f014 <int>, new_sp_f1524 <int>, new_sp_f2534 <int>,
    ## #   new_sp_f3544 <int>, new_sp_f4554 <int>, new_sp_f5564 <int>,
    ## #   new_sp_f65 <int>, new_sn_m014 <int>, new_sn_m1524 <int>,
    ## #   new_sn_m2534 <int>, new_sn_m3544 <int>, new_sn_m4554 <int>,
    ## #   new_sn_m5564 <int>, new_sn_m65 <int>, new_sn_f014 <int>,
    ## #   new_sn_f1524 <int>, new_sn_f2534 <int>, new_sn_f3544 <int>,
    ## #   new_sn_f4554 <int>, new_sn_f5564 <int>, new_sn_f65 <int>,
    ## #   new_ep_m014 <int>, new_ep_m1524 <int>, new_ep_m2534 <int>,
    ## #   new_ep_m3544 <int>, new_ep_m4554 <int>, new_ep_m5564 <int>,
    ## #   new_ep_m65 <int>, new_ep_f014 <int>, new_ep_f1524 <int>,
    ## #   new_ep_f2534 <int>, new_ep_f3544 <int>, new_ep_f4554 <int>,
    ## #   new_ep_f5564 <int>, new_ep_f65 <int>, newrel_m014 <int>,
    ## #   newrel_m1524 <int>, newrel_m2534 <int>, newrel_m3544 <int>,
    ## #   newrel_m4554 <int>, newrel_m5564 <int>, newrel_m65 <int>,
    ## #   newrel_f014 <int>, newrel_f1524 <int>, newrel_f2534 <int>,
    ## #   newrel_f3544 <int>, newrel_f4554 <int>, newrel_f5564 <int>,
    ## #   newrel_f65 <int>

``` r
who %>%
  gather(group, cases, -country, -iso2, -iso3, -year)
```

    ## # A tibble: 405,440 × 6
    ##        country  iso2  iso3  year       group cases
    ##          <chr> <chr> <chr> <int>       <chr> <int>
    ## 1  Afghanistan    AF   AFG  1980 new_sp_m014    NA
    ## 2  Afghanistan    AF   AFG  1981 new_sp_m014    NA
    ## 3  Afghanistan    AF   AFG  1982 new_sp_m014    NA
    ## 4  Afghanistan    AF   AFG  1983 new_sp_m014    NA
    ## 5  Afghanistan    AF   AFG  1984 new_sp_m014    NA
    ## 6  Afghanistan    AF   AFG  1985 new_sp_m014    NA
    ## 7  Afghanistan    AF   AFG  1986 new_sp_m014    NA
    ## 8  Afghanistan    AF   AFG  1987 new_sp_m014    NA
    ## 9  Afghanistan    AF   AFG  1988 new_sp_m014    NA
    ## 10 Afghanistan    AF   AFG  1989 new_sp_m014    NA
    ## # ... with 405,430 more rows

`ggplot2`
---------

If you don't already know and love it, check out [one of](https://d-rug.github.io/blog/2012/ggplot-introduction) [our](https://d-rug.github.io/blog/2013/xtsmarkdown) [previous](https://d-rug.github.io/blog/2013/formatting-plots-for-pubs) [talks](https://d-rug.github.io/blog/2015/ggplot-tutorial-johnston) on ggplot or any of the excellent resources on the internet.

Note that the pipe and consistent API make it easy to combine functions from different packages, and the whole thing is quite readable.

``` r
who %>%
  select(-iso2, -iso3) %>%
  gather(group, cases, -country, -year) %>%
  count(country, year, wt = cases) %>%
  ggplot(aes(x = year, y = n, group = country)) +
  geom_line(size = .2) 
```

![](tidyverse_files/figure-markdown_github/dplyr-tidyr-ggplot-1.png)

`readr`
-------

For reading flat files. Faster than base with smarter defaults.

``` r
bigdf = data_frame(int = 1:1e6, 
                   squares = int^2, 
                   letters = sample(letters, 1e6, replace = TRUE))
```

``` r
system.time(
  write.csv(bigdf, "base-write.csv")
)
```

    ##    user  system elapsed 
    ##   2.579   0.084   2.704

``` r
system.time(
  write_csv(bigdf, "readr-write.csv")
)
```

    ##    user  system elapsed 
    ##   0.819   0.069   0.940

``` r
read.csv("base-write.csv", nrows = 3)
```

    ##   X int squares letters
    ## 1 1   1       1       h
    ## 2 2   2       4       d
    ## 3 3   3       9       m

``` r
read_csv("readr-write.csv", n_max = 3)
```

    ## Parsed with column specification:
    ## cols(
    ##   int = col_integer(),
    ##   squares = col_double(),
    ##   letters = col_character()
    ## )

    ## # A tibble: 3 × 3
    ##     int squares letters
    ##   <int>   <dbl>   <chr>
    ## 1     1       1       h
    ## 2     2       4       d
    ## 3     3       9       m

`broom`
-------

`broom` is a convenient little package to work with model results. Two functions I find useful are `tidy` to extract model results and `augment` to add residuals, predictions, etc. to a data.frame.

``` r
d = data_frame(x = runif(20, 0, 10), 
               y = 2 * x + rnorm(10))
qplot(x, y, data = d)
```

![](tidyverse_files/figure-markdown_github/make%20model%20data-1.png)

### `tidy`

``` r
library(broom)  # Not attached with tidyverse
model = lm(y ~ x, d)
tidy(model)
```

    ##          term    estimate  std.error   statistic      p.value
    ## 1 (Intercept) -0.02212952 0.32815614 -0.06743595 9.469781e-01
    ## 2           x  2.04880361 0.06368259 32.17211622 2.328477e-17

### `augment`

i.e. The function formerly known as `fortify`.

``` r
aug = augment(model)
aug
```

    ##            y         x   .fitted   .se.fit      .resid       .hat
    ## 1   4.365310 2.2167792  4.519616 0.2190258 -0.15430550 0.08531087
    ## 2  10.183616 5.4172613 11.076775 0.1790893 -0.89315878 0.05703654
    ## 3   7.215538 3.2282729  6.591968 0.1843041  0.62357006 0.06040652
    ## 4  16.104626 7.9968916 16.361931 0.2823602 -0.25730454 0.14178182
    ## 5  15.469541 8.0496637 16.470051 0.2850711 -1.00050940 0.14451735
    ## 6   1.777862 0.5917886  1.190329 0.2963871  0.58753261 0.15621841
    ## 7  14.792232 7.4687935 15.279961 0.2560816 -0.48772985 0.11661935
    ## 8   7.947098 3.9047317  7.977899 0.1709766 -0.03080099 0.05198605
    ## 9   6.652823 3.3824637  6.907874 0.1804498 -0.25505137 0.05790640
    ## 10 16.466676 7.2607843 14.853792 0.2462225  1.61288435 0.10781254
    ## 11  9.147981 4.6081148  9.418993 0.1680642 -0.27101132 0.05023009
    ## 12  7.045629 3.8482676  7.862215 0.1717156 -0.81658622 0.05243644
    ## 13 15.521358 7.3811829 15.100465 0.2518912  0.42089305 0.11283397
    ## 14  2.047400 0.9682785  1.961683 0.2769494  0.08571719 0.13640005
    ## 15  1.924992 1.2773890  2.594990 0.2615541 -0.66999792 0.12165693
    ## 16  6.737163 3.0714391  6.270646 0.1886685  0.46651671 0.06330129
    ## 17  1.835538 0.9904465  2.007101 0.2758271 -0.17156311 0.13529686
    ## 18  3.904365 1.8833655  3.836516 0.2332531  0.06784899 0.09675390
    ## 19 16.870169 8.4911367 17.374542 0.3082513 -0.50437308 0.16897542
    ## 20 15.051012 6.5529524 13.403583 0.2154124  1.64742911 0.08251922
    ##       .sigma      .cooksd  .std.resid
    ## 1  0.7706297 2.158758e-03 -0.21515503
    ## 2  0.7386729 4.549928e-02 -1.22655805
    ## 3  0.7556838 2.365691e-02  0.85787128
    ## 4  0.7686765 1.133193e-02 -0.37038676
    ## 5  0.7256519 1.757615e-01 -1.44252197
    ## 6  0.7558680 6.734724e-02  0.85295049
    ## 7  0.7612891 3.160947e-02 -0.69200983
    ## 8  0.7715844 4.879446e-05 -0.04218560
    ## 9  0.7689861 3.773788e-03 -0.35041888
    ## 10 0.6510658 3.132905e-01  2.27709965
    ## 11 0.7686693 3.636518e-03 -0.37083874
    ## 12 0.7443161 3.462616e-02 -1.11867707
    ## 13 0.7639734 2.258173e-02  0.59590381
    ## 14 0.7712982 1.194837e-03  0.12300378
    ## 15 0.7518898 6.294179e-02 -0.95334091
    ## 16 0.7627149 1.396146e-02  0.64279740
    ## 17 0.7703240 4.735711e-03 -0.24603520
    ## 18 0.7714283 4.854302e-04  0.09520224
    ## 19 0.7598647 5.534563e-02 -0.73782239
    ## 20 0.6491487 2.365693e-01  2.29358645

``` r
ggplot(aug, aes(x = x)) +
  geom_point(aes(y = y, color = .resid)) + 
  geom_line(aes(y = .fitted)) +
  viridis::scale_color_viridis() +
  theme(legend.position = c(0, 1), legend.justification = c(0, 1))
```

![](tidyverse_files/figure-markdown_github/plot%20resid-1.png)

``` r
ggplot(aug, aes(.fitted, .resid, size = .cooksd)) + 
  geom_point()
```

![](tidyverse_files/figure-markdown_github/plot%20cooksd-1.png)

`purrr`
-------

`purrr` is kind of like `dplyr` for lists. It helps you repeatedly apply functions. Like the rest of the tidyverse, nothing you can't do in base R, but `purrr` makes the API consistent, encourages type specificity, and provides some nice shortcuts and speed ups.

``` r
df = data_frame(fun = rep(c(lapply, map), 2),
                n = rep(c(1e5, 1e7), each = 2),
                comp_time = map2(fun, n, ~system.time(.x(1:.y, sqrt))))
df$comp_time
```

    ## [[1]]
    ##    user  system elapsed 
    ##   0.042   0.003   0.046 
    ## 
    ## [[2]]
    ##    user  system elapsed 
    ##   0.055   0.001   0.059 
    ## 
    ## [[3]]
    ##    user  system elapsed 
    ##  12.366   0.218  12.612 
    ## 
    ## [[4]]
    ##    user  system elapsed 
    ##   8.572   0.117   8.742

### `map`

Vanilla `map` is a slightly improved version of `lapply`. Do a function on each item in a list.

``` r
map(1:4, log)
```

    ## [[1]]
    ## [1] 0
    ## 
    ## [[2]]
    ## [1] 0.6931472
    ## 
    ## [[3]]
    ## [1] 1.098612
    ## 
    ## [[4]]
    ## [1] 1.386294

Can supply additional arguments as with `(x)apply`

``` r
map(1:4, log, base = 2)
```

    ## [[1]]
    ## [1] 0
    ## 
    ## [[2]]
    ## [1] 1
    ## 
    ## [[3]]
    ## [1] 1.584963
    ## 
    ## [[4]]
    ## [1] 2

Can compose anonymous functions like `(x)apply`, either the old way or with a new formula shorthand.

``` r
map(1:4, ~ log(4, base = .x))  # == map(1:4, function(x) log(4, base = x))
```

    ## [[1]]
    ## [1] Inf
    ## 
    ## [[2]]
    ## [1] 2
    ## 
    ## [[3]]
    ## [1] 1.26186
    ## 
    ## [[4]]
    ## [1] 1

`map` always returns a list. `map_xxx` type-specifies the output type and simplifies the list to a vector.

``` r
map_dbl(1:4, log, base = 2)
```

    ## [1] 0.000000 1.000000 1.584963 2.000000

And throws an error if any output isn't of the expected type (which is a good thing!).

``` r
map_int(1:4, log, base = 2)
```

    ## Error: Can't coerce element 1 from a double to a integer

`map2` is like `mapply` -- apply a function over two lists in parallel. `map_n` generalizes to any number of lists.

``` r
fwd = 1:10
bck = 10:1
map2_dbl(fwd, bck, `^`)
```

    ##  [1]     1   512  6561 16384 15625  7776  2401   512    81    10

`map_if` tests each element on a function and if true applies the second function, if false returns the original element.

``` r
data_frame(ints = 1:5, lets = letters[1:5], sqrts = ints^.5) %>%
  map_if(is.numeric, ~ .x^2) 
```

    ## $ints
    ## [1]  1  4  9 16 25
    ## 
    ## $lets
    ## [1] "a" "b" "c" "d" "e"
    ## 
    ## $sqrts
    ## [1] 1 2 3 4 5

### Putting `map` to work

Split the movies data frame by mpaa rating, fit a linear model to each data frame, and organize the model results in a data frame.

``` r
movies %>% 
  filter(mpaa != "") %>%
  split(.$mpaa) %>%
  map(~ lm(rating ~ budget, data = .)) %>%
  map_df(tidy, .id = "mpaa-rating") %>%
  arrange(term)
```

    ##   mpaa-rating        term      estimate    std.error   statistic
    ## 1       NC-17 (Intercept)  6.505809e+00 3.124604e-01  20.8212250
    ## 2          PG (Intercept)  5.768036e+00 1.368008e-01  42.1637635
    ## 3       PG-13 (Intercept)  5.749256e+00 7.809921e-02  73.6147735
    ## 4           R (Intercept)  5.814014e+00 5.238965e-02 110.9764024
    ## 5       NC-17      budget -6.046148e-08 1.835425e-08  -3.2941404
    ## 6          PG      budget  2.028426e-09 2.745805e-09   0.7387365
    ## 7       PG-13      budget  3.493136e-09 1.443405e-09   2.4200674
    ## 8           R      budget  7.732453e-09 1.658841e-09   4.6613582
    ##         p.value
    ## 1  4.732587e-06
    ## 2 1.856102e-104
    ## 3 8.291365e-280
    ## 4  0.000000e+00
    ## 5  2.161461e-02
    ## 6  4.608918e-01
    ## 7  1.585419e-02
    ## 8  3.540387e-06

List-columns make it easier to organize complex datasets. Can `map` over list-columns right in `data_frame`/`tibble` creation. And if you later want to calculate something else, everything is nicely organized in the data frame.

``` r
d = 
  data_frame(
    dist = c("normal", "poisson", "chi-square"),
    funs = list(rnorm, rpois, rchisq),
    samples = map(funs, ~.(100, 5)),
    mean = map_dbl(samples, mean),
    var = map_dbl(samples, var)
  )
d$median = map_dbl(d$samples, median)
d
```

    ## # A tibble: 3 × 6
    ##         dist   funs     samples     mean        var   median
    ##        <chr> <list>      <list>    <dbl>      <dbl>    <dbl>
    ## 1     normal  <fun> <dbl [100]> 4.897684  0.9952718 4.910766
    ## 2    poisson  <fun> <int [100]> 4.990000  4.1716162 5.000000
    ## 3 chi-square  <fun> <dbl [100]> 5.466018 12.3804613 4.923235

Let's see if we can really make this purrr... Fit a linear model of diamond price by every combination of two predictors in the dataset and see which two predict best.

``` r
train = sample(nrow(diamonds), floor(nrow(diamonds) * .67))
setdiff(names(diamonds), "price") %>%
  combn(2, paste, collapse = " + ") %>%
  structure(., names = .) %>%
  map(~ formula(paste("price ~ ", .x))) %>%
  map(lm, data = diamonds[train, ]) %>%
  map_df(augment, newdata = diamonds[-train, ], .id = "predictors") %>%
  group_by(predictors) %>%
  summarize(rmse = sqrt(mean((price - .fitted)^2))) %>%
  arrange(rmse)
```

    ## # A tibble: 36 × 2
    ##         predictors     rmse
    ##              <chr>    <dbl>
    ## 1  carat + clarity 1296.010
    ## 2    carat + color 1474.577
    ## 3      carat + cut 1518.669
    ## 4        carat + x 1530.131
    ## 5        carat + y 1545.970
    ## 6    carat + depth 1546.579
    ## 7    carat + table 1549.821
    ## 8        carat + z 1557.959
    ## 9      clarity + x 1672.964
    ## 10     clarity + y 1689.942
    ## # ... with 26 more rows

### Type-stability

We have seen that we can use map\_lgl to ensure we get a logical vector, map\_chr to ensure we get a character vector back, etc. Type stability is like a little built-in unit test. You make sure you're getting what you think you are, even in the middle of a pipeline or function. Here are two more type-stable function implemented in `purrr`.

#### `flatten`

Like `unlist` but can specify output type, and never recurses.

``` r
map(-1:3, ~.x ^ seq(-.5, .5, .5)) %>%
  flatten_dbl()
```

    ##  [1]       NaN 1.0000000       NaN       Inf 1.0000000 0.0000000 1.0000000
    ##  [8] 1.0000000 1.0000000 0.7071068 1.0000000 1.4142136 0.5773503 1.0000000
    ## [15] 1.7320508

#### `safely`

``` r
junk = list(letters, 1:20, median)
map(junk, ~ log(.x))
```

    ## Error in log(.x): non-numeric argument to mathematical function

-   `safely` "catches" errors and always "succeeds".
-   `try` does the same, but either returns the value or a try-error object.
-   `safely` is type-stable. It always returns a length-two list with one object NULL.

``` r
safe = map(junk, ~ safely(log)(.x))  # Note the different syntax from try(log(.x)). `safely(log)` creates a new function.
safe
```

    ## [[1]]
    ## [[1]]$result
    ## NULL
    ## 
    ## [[1]]$error
    ## <simpleError in .f(...): non-numeric argument to mathematical function>
    ## 
    ## 
    ## [[2]]
    ## [[2]]$result
    ##  [1] 0.0000000 0.6931472 1.0986123 1.3862944 1.6094379 1.7917595 1.9459101
    ##  [8] 2.0794415 2.1972246 2.3025851 2.3978953 2.4849066 2.5649494 2.6390573
    ## [15] 2.7080502 2.7725887 2.8332133 2.8903718 2.9444390 2.9957323
    ## 
    ## [[2]]$error
    ## NULL
    ## 
    ## 
    ## [[3]]
    ## [[3]]$result
    ## NULL
    ## 
    ## [[3]]$error
    ## <simpleError in .f(...): non-numeric argument to mathematical function>

#### `transpose` a list!

Now we could conveniently move on where the function succeeded, particularly using `map_if`. To get that logical vector for the `map_if` test, we can use the `transpose` function, which inverts a list.

``` r
transpose(safe)
```

    ## $result
    ## $result[[1]]
    ## NULL
    ## 
    ## $result[[2]]
    ##  [1] 0.0000000 0.6931472 1.0986123 1.3862944 1.6094379 1.7917595 1.9459101
    ##  [8] 2.0794415 2.1972246 2.3025851 2.3978953 2.4849066 2.5649494 2.6390573
    ## [15] 2.7080502 2.7725887 2.8332133 2.8903718 2.9444390 2.9957323
    ## 
    ## $result[[3]]
    ## NULL
    ## 
    ## 
    ## $error
    ## $error[[1]]
    ## <simpleError in .f(...): non-numeric argument to mathematical function>
    ## 
    ## $error[[2]]
    ## NULL
    ## 
    ## $error[[3]]
    ## <simpleError in .f(...): non-numeric argument to mathematical function>

``` r
map_if(transpose(safe)$result, ~!is.null(.x), median)
```

    ## [[1]]
    ## NULL
    ## 
    ## [[2]]
    ## [1] 2.35024
    ## 
    ## [[3]]
    ## NULL

`stringr`
---------

All your string manipulation and regex functions with a consistent API.

``` r
library(stringr)  # not attached with tidyverse
fishes <- c("one fish", "two fish", "red fish", "blue fish")
str_detect(fishes, "two")
```

    ## [1] FALSE  TRUE FALSE FALSE

``` r
str_replace_all(fishes, "fish", "banana")
```

    ## [1] "one banana"  "two banana"  "red banana"  "blue banana"

``` r
str_extract(fishes, "[a-z]\\s")
```

    ## [1] "e " "o " "d " "e "

Let's put that string manipulation engine to work. Remember the annoying column names in the WHO data? They look like this new\_sp\_m014, new\_sp\_m1524, new\_sp\_m2534, where "new" or "new\_" doesn't mean anything, the following 2-3 letters indicate the test used, the following letter indicates the gender, and the final 2-4 numbers indicates the age-class. A string-handling challenge if ever there was one. Let's separate it out and plot the cases by year, gender, age-class, and test-method.

``` r
who %>%
  select(-iso2, -iso3) %>%
  gather(group, cases, -country, -year) %>%
  mutate(group = str_replace(group, "new_*", ""),
         method = str_extract(group, "[a-z]+"),
         gender = str_sub(str_extract(group, "_[a-z]"), 2, 2),
         age = str_extract(group, "[0-9]+"),
         age = ifelse(str_length(age) > 2,
                      str_c(str_sub(age, 1, -3), str_sub(age, -2, -1), sep = "-"),
                      str_c(age, "+"))) %>%
  group_by(year, gender, age, method) %>%
  summarize(total_cases = sum(cases, na.rm = TRUE)) %>%
  ggplot(aes(x = year, y = total_cases, linetype = gender)) +
  geom_line() +
  facet_grid(method ~ age,
             labeller = labeller(.rows = label_both, .cols = label_both)) +
  scale_y_log10() +
  theme_light() +
  theme(axis.text.x = element_text(angle = 45, vjust = 1, hjust = 1))
```

![](tidyverse_files/figure-markdown_github/unnamed-chunk-6-1.png)

Post-talk debugging improvisation
---------------------------------

``` r
pipe_stopifnot = function(df, test){
  stopifnot(test)
  return(df)
}
```

``` r
print_and_go = function(df, what_to_print) {
  cat(what_to_print)
  return(df)
}
```
