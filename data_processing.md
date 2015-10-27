Data processing in R
====================

[Intro...]

### Example data

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

### Vector operations

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

### Multiple comparisons with *%in%* and *match*

Using logical subsetting, we can extract all observations of a particular species from the *surveys* data frame, e.g. `surveys[surveys$species_id == "NL", ]`. What if we want to extract all rows corresponding to a specific subset of species? We use the `%in%` operator:

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

### Matrix operations

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

-   For a single-line function that is only used once, we can streamline the above code by creating an anonymous (unnamed) function directly in the `apply` call: `apply(counts_prop, 1, function(x) any(x > 0.5))`.
-   `scale` is a useful alternative to `sweep` when you want to center (subtract a value) and scale (divide by a value) each column of a matrix. By default, `scale` substracts the mean and divides by the standard deviation of each column.
-   Although we focused on matrices, `apply` and `sweep` also work with arrays of three or more dimensions.

### Exercises

-   What is the minimum and maximum number of species found in a single plot?
-   Which species are present in five different plots or less?
