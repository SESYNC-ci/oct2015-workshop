Data processing in R
====================

This tutorial introduces some of the functions provided by R to efficiently organize, transform and summarize data contained in each of its basic data structures: vectors, matrices, lists and data frames.

Example data
------------

For this tutorial, we will use example data from the [Portal teaching dataset](http://github.com/weecology/portal-teachingdb), which is derived from a long-term monitoring dataset of a desert ecosystem near Portal, AZ. We first load the relevant files that were previously saved in the SESYNC *public-data* shared folder:

``` r
plots <- read.csv("/nfs/public-data/portal-teachingdb/plots.csv", na.strings = "")
species <- read.csv("/nfs/public-data/portal-teachingdb/species.csv", na.strings = "")
surveys <- read.csv("/nfs/public-data/portal-teachingdb/surveys.csv", na.strings = "")
head(surveys)
```

    ##   record_id month day year plot_id species_id sex hindfoot_length weight
    ## 1         1     7  16 1977       2         NL   M              32     NA
    ## 2         2     7  16 1977       3         NL   M              33     NA
    ## 3         3     7  16 1977       2         DM   F              37     NA
    ## 4         4     7  16 1977       7         DM   M              36     NA
    ## 5         5     7  16 1977       3         DM   M              35     NA
    ## 6         6     7  16 1977       1         PF   M              14     NA

Vector operations
-----------------

An important feature of R is that most base functions naturally work on arbitrary long vectors. For example, arithmetic operators, logical operators, and math functions like `log()`, `cos()`, etc. operate in parallel on each element of their input vector(s).

``` r
c(1, 2) + c(20, 15)
sqrt(c(9, 16))
1:10 / 2
```

    ## [1] 21 17
    ## [1] 3 4
    ##  [1] 0.5 1.0 1.5 2.0 2.5 3.0 3.5 4.0 4.5 5.0

When a function requires multiple vectors and one of them is shorter than the other, the default behavior of R is to *recycle* elements from the shorter vector. The last example above is interpreted by R as `1:10 / rep(2, 10)` (the function `rep(x, n)` produces a vector with *n* repeats of the vector *x*). Besides such simple cases involving a vector and a single number, it is best to not rely on the recycling behavior and make sure that calculations operate on vector of the same length. This produces more transparent and less error-prone code.

To modify a vector based on some condition applied to each of its elements, R provides a parallel version of `if`, the `ifelse` function. `ifelse(cond, a, b)` works like this: for each element of the logical vector *cond* that is *TRUE*, return the corresponding element from *a*; when it's *FALSE*, return the corresponding element from *b*. For example, we can use `ifelse` to add "C" to the id of control plots and "E" to the id of the non-control (experimental) plot. Note how the `paste0` function also works in parallel.

``` r
new_plot_id <- paste0(plots$plot_id, ifelse(plots$plot_type == "Control", "C", "E"))
head(cbind(plots, new_plot_id))
```

    ##   plot_id                 plot_type new_plot_id
    ## 1       1         Spectab exclosure          1E
    ## 2       2                   Control          2C
    ## 3       3  Long-term Krat Exclosure          3E
    ## 4       4                   Control          4C
    ## 5       5          Rodent Exclosure          5E
    ## 6       6 Short-term Krat Exclosure          6E

**Q.**: Is there any *recycling* of values in the example above? (Yes, "Control", "C" and "E" are implicitly repeated to match the length of `plot$plot_type`.)

Not all vector functions operate in parallel. Summary functions such as `sum`, `mean`, `min` and `max` take a vector input and return a single number (but note the existence of `pmin` and `pmax`, parallel version of `min` and `max`). The logical summary functions [`any`, `all`] return a single TRUE/FALSE value based on whether [any, all] of the input vector's elements are TRUE.

Multiple comparisons with *%in%* and *match*
--------------------------------------------

Using logical subsetting, we can extract all observations of a particular species from the *surveys* data frame, e.g. `surveys[which(surveys$species_id == "NL"), ]`. What if we want to extract all rows corresponding to a specific subset of species? We use the `%in%` operator:

``` r
survey_set <- surveys[surveys$species_id %in% c("NL", "DS", "PF"), ]
head(survey_set)
```

    ##    record_id month day year plot_id species_id sex hindfoot_length weight
    ## 1          1     7  16 1977       2         NL   M              32     NA
    ## 2          2     7  16 1977       3         NL   M              33     NA
    ## 6          6     7  16 1977       1         PF   M              14     NA
    ## 10        10     7  16 1977       6         PF   F              20     NA
    ## 11        11     7  16 1977       5         DS   F              53     NA
    ## 17        17     7  16 1977       3         DS   F              48     NA

Alternatively, we may want to find the genus of each of those species from the *species* data frame. This can be done quickly with the *match* function:

``` r
species$genus[match(c("NL", "DS", "PF"), species$species_id)] 
```

    ## [1] Neotoma     Dipodomys   Perognathus
    ## 30 Levels: Ammodramus Ammospermophilus Amphispiza Baiomys ... Zonotrichia

Note that whereas `x %in% y` returns a logical vector indicating if each element of *x* can be found in *y*, `match(x, y)` returns the index of the first occurrence of each element of `x` within `y`.

**Q.**: What would be the problem with writing `species$genus[species$species_id %in% c("NL", "DS", "PF")` in the last example? (It would have returned the same three names, but in the order in which they appear in *species*, rather than the order of the codes we provided.)

Matrix operations
-----------------

We first use the `table` function to produce a contingency table of the number of observations of each species in each plot:

``` r
counts_mat <- table(surveys$species_id, surveys$plot_id)
counts_mat[1:5, 1:5]
```

    ##     
    ##      1  2  3 4 5
    ##   AB 7 14 10 3 2
    ##   AH 7  7  2 2 3
    ##   AS 0  0  0 0 0
    ##   BA 1  1 19 0 4
    ##   CB 0  0  1 1 1

As you can see, the output is just a matrix with rows labelled by *species\_id* and columns labelled by *plot\_id*. Vector summary functions (`sum`, `mean`, etc.) aggregate over the entire matrix:

``` r
sum(counts_mat)
```

    ## [1] 34786

**Q.**: Why is this number different from the total number of observations in *surveys*? (Hint: Check for unknown species.)

R also provides matrix-specific summary functions: `rowSums`, `colSums`, `rowMeans` and `colMeans`. In our case, `rowSums(counts_mat)` returns a named vector of total counts by species.

``` r
rowSums(counts_mat)[1:5]
```

    ##  AB  AH  AS  BA  CB 
    ## 303 437   2  46  50

More generally, the `apply` function provides a way to apply any function to each row or each column of a matrix. For example, `rowSums(counts_mat)` is equivalent to `apply(counts_mat, 1, sum)`. The second argument is set to "1" to apply over rows, "2" to apply over columns.

Let's say we want to find species for which the majority of observations occur in a single plot. As a first step, we use the `sweep` function to calculate the proportion of each species' total observations that is found in each plot:

``` r
counts_prop <- sweep(counts_mat, 1, rowSums(counts_mat), "/")
```

This particular `sweep` function call translates as: divide (fourth argument: "/") each row (second argument: "1") of *counts\_mat* (first argument) by the sum of that row (third argument).

The next step of the problem is to determine, for each species (row), whether any plot has a proportion of over 0.5. We create a function that tests whether any element of a vector is greater than 0.5, then `apply` it to every row:

``` r
has_major <- function(x) {
    any(x > 0.5)
}
major_plot <- apply(counts_prop, 1, has_major)
names(which(major_plot)) 
```

    ## [1] "CS" "CT" "CU" "CV" "PU" "SC" "SO" "ST"

**Notes**:

-   For a single line function that is only used once, we can streamline the above code by creating an anonymous (unnamed) function directly in the `apply` call: `apply(counts_prop, 1, function(x) any(x > 0.5))`.
-   `scale` is a useful alternative to `sweep` when you want to center (subtract a value) and scale (divide by a value) each column of a matrix. By default, `scale` substracts the mean and divides by the standard deviation of each column.
-   Although we focused on matrices, `apply` and `sweep` also work with arrays of three or more dimensions.

### Exercises

-   What is the minimum and maximum number of species found in a single plot?
-   Which species are present in five different plots or less?

Data frame operations
---------------------

Now let's say you want to compute a summary statistic over more specific subsets of your data, you can use the `aggregate` to define the subsets and the summary function. For example, say you wanted to know the mean hindfoot length for each species, you can create a formula where y variables (numeric data) are split by x grouping variables. You can also specify multiple y variables (e.g., hindfoot length and weight) and or multiple x (e.g., species and sex).

``` r
hindfoot_spp <- aggregate(data = surveys, hindfoot_length ~ species_id, FUN = mean)
head(hindfoot_spp)
```

    ##   species_id hindfoot_length
    ## 1         AH        33.00000
    ## 2         BA        13.00000
    ## 3         DM        35.98235
    ## 4         DO        35.60755
    ## 5         DS        49.94887
    ## 6         NL        32.29423

``` r
lw_spp_sex <- aggregate(data = surveys, cbind(hindfoot_length, weight) ~ species_id + sex, FUN = mean)
head(lw_spp_sex)
```

    ##   species_id sex hindfoot_length    weight
    ## 1         BA   F        13.16129   9.16129
    ## 2         DM   F        35.71942  41.57354
    ## 3         DO   F        35.47213  48.54098
    ## 4         DS   F        49.60878 117.39224
    ## 5         NL   F        32.01320 154.36964
    ## 6         OL   F        20.28211  30.78440

Now if you want to sort the data.frame by species name, you can use the `order` function.

``` r
lw_spp_sex <- lw_spp_sex[order(lw_spp_sex$species_id),]
head(lw_spp_sex)
```

    ##    species_id sex hindfoot_length    weight
    ## 1          BA   F        13.16129  9.161290
    ## 23         BA   M        12.64286  7.357143
    ## 2          DM   F        35.71942 41.573536
    ## 24         DM   M        36.19674 44.324344
    ## 3          DO   F        35.47213 48.540984
    ## 25         DO   M        35.67771 49.121019

If you are only interested in observations for which all variables were recorded (there is no missing data). You can use `complete.cases` to return the indices of rows with no NAs and then subset the original data.frame for these rows.

``` r
surveys_complete <- surveys[complete.cases(surveys),]
head(surveys_complete)
```

    ##    record_id month day year plot_id species_id sex hindfoot_length weight
    ## 63        63     8  19 1977       3         DM   M              35     40
    ## 64        64     8  19 1977       7         DM   M              37     48
    ## 65        65     8  19 1977       4         DM   F              34     29
    ## 66        66     8  19 1977       4         DM   F              35     46
    ## 67        67     8  19 1977       7         DM   M              35     36
    ## 68        68     8  19 1977       8         DO   F              32     52

**Q.**: How many incomplete cases were in the original surveys dataset?

The `unique` function is also useful when you want to remove duplicate elements/rows or summarize the unique combinations of variables

``` r
sampling_dates <- unique(surveys[,c("month", "day", "year")])
head(sampling_dates)
```

    ##     month day year
    ## 1       7  16 1977
    ## 20      7  17 1977
    ## 40      7  18 1977
    ## 63      8  19 1977
    ## 83      8  20 1977
    ## 114     8  21 1977

The `merge` function lets you combine data.frames based on a common column. For example, say you wanted to combine the information on plot type from the plots data.frame with the raw surveys data, you can merge the two columns based on the `plot_id` column. The common column must either have the same name in both data.frames or you can specific the unique names in each with the `by.x` and `by.y` arguments.

``` r
merged_surveys <- merge(surveys, plots, by = "plot_id")
head(merged_surveys)
```

    ##   plot_id record_id month day year species_id  sex hindfoot_length weight
    ## 1       1     30429     3   4 2000         PP    F              22     17
    ## 2       1     33950     5  15 2002         NL    M              30    205
    ## 3       1      5669     3  30 1982         DM    M              36     46
    ## 4       1      2947     5  17 1980         DS    M              48    102
    ## 5       1      9891     1  20 1985         AH <NA>              NA     NA
    ## 6       1     31375     9  30 2000         DO    M              33     50
    ##           plot_type
    ## 1 Spectab exclosure
    ## 2 Spectab exclosure
    ## 3 Spectab exclosure
    ## 4 Spectab exclosure
    ## 5 Spectab exclosure
    ## 6 Spectab exclosure

Finally, there are some helpful functions in the `reshape2` package for massaging data.frames in different ways. The `melt` function allows you to put each unique id-variable combination in its own row. The `cast` functions (`dcast` for data.frames in `reshape2`) lets you move the melted data into any shape you like. Say we wanted to use the tabulated count data for each species\*plot combination, but wanted each count in its own row, we can use the melt function to accomplish this. Using the `dcast` function, we can "put the data back".

``` r
library(reshape2)
counts_melt <- melt(counts_mat)
colnames(counts_melt) <- c("species_id", "plot_id", "count")
head(counts_melt)
```

    ##   species_id plot_id count
    ## 1         AB       1     7
    ## 2         AH       1     7
    ## 3         AS       1     0
    ## 4         BA       1     1
    ## 5         CB       1     0
    ## 6         CM       1     0

``` r
counts_cast <- dcast(counts_melt, species_id ~ plot_id)
```

    ## Using count as value column: use value.var to override.

``` r
counts_cast[1:5,1:5]
```

    ##   species_id 1  2  3 4
    ## 1         AB 7 14 10 3
    ## 2         AH 7  7  2 2
    ## 3         AS 0  0  0 0
    ## 4         BA 1  1 19 0
    ## 5         CB 0  0  1 1

### Exercise

-   Which three genera have the largest mean hindfoot lengths?

Additional resources
--------------------

-   The [tidyr](https://cran.r-project.org/web/packages/tidyr/index.html) and [dplyr](https://cran.r-project.org/web/packages/dplyr/index.html) packages provide functions that streamline many common processing steps for data frames (e.g. reshape, sort, aggregate, join). Also check out our [tutorial](https://github.com/SESYNC-ci/CSI-2015/blob/master/Lessons/R/tidyr_dplyr.md) that focuses on these two packages.
