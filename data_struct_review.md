Review of R data structures
===========================

Vectors
-------

The simplest data structure in R is a *vector* (more strictly, an atomic vector). A vector is a sequence of elements sharing the same basic data type. We create new vectors and append to existing vectors with `c()`.

``` r
# A numeric vector
vect <- c(4.5, 2, 99, -0.3)
vect <- c(vect, NA)
str(vect)
```

    ##  num [1:5] 4.5 2 99 -0.3 NA

``` r
# A numeric vector with names
vectn <- c(a = 4.5, b = 2, c = 99, d = -0.3)
str(vectn)
```

    ##  Named num [1:4] 4.5 2 99 -0.3
    ##  - attr(*, "names")= chr [1:4] "a" "b" "c" "d"

The `str` command displays the structure of an object. This is similar to the information shown in the *Environment* pane of RStudio.

We generally subset vectors with single brackets `[]`. Recall that there are multiple ways to do so:

-   with a vector of positive integers (positions of elements to retain), e.g. `vect[1:2]`
-   with a vector of negative integers (positions of elements to discard), e.g. `vect[-c(1,4)]`
-   with a logical vector (retain elements where logical vector is *TRUE*), e.g. `vect[vect > 3]`
-   with a character vector (names of elements to retain), e.g. `vectn[c("b", "c")]`

Matrices
--------

Matrices are the two-dimensional generalization of vectors. Each row or column of a matrix is a vector, which means that all entries must be of the same data type. Matrices can be created from scratch with `matrix` or by reshaping a vector with `dim`. In the latter case, note that the matrix is filled one column at a time.

``` r
# Reshaping a vector into a matrix
mat <- 1:6
dim(mat) <- c(3,2)
mat
```

    ##      [,1] [,2]
    ## [1,]    1    4
    ## [2,]    2    5
    ## [3,]    3    6

New rows and columns are appended to a matrix with `rbind` and `cbind`, respectively. Matrix subsetting works the same way as vector subsetting, except that two expressions must be provided, separated by a comma. The first expression subsets the rows and the second subsets the columns. Either expression can be empty, which translates as "keep all the rows and/or columns". R accepts different types of expressions for the two dimensions, e.g.:

``` r
mat[1:2, c(FALSE, TRUE)]
```

    ## [1] 4 5

As we only selected the second column, the output is a vector. This might be undesirable in a script where the following instructions expect a matrix. With one additional argument: `mat[1:2, c(FALSE, TRUE), drop = FALSE]`, R will keep all the dimensions and return a 2 x 1 matrix in this case.

**Q.**: How would you reverse the order of the rows in *mat*?

Since a matrix is simply a vector reshaped in two dimensions, you can still use a single subsetting expression to select individual cells from the matrix. For example, `mat[mat <= 4]` will return the vector `(1, 2, 3, 4)`. If you need to know the position of those elements in the matrix, use the `arr.ind` option of `which`:

``` r
inds <- which(mat <= 4, arr.ind = TRUE)
inds
```

    ##      row col
    ## [1,]   1   1
    ## [2,]   2   1
    ## [3,]   3   1
    ## [4,]   1   2

The final way to pick elements from a matrix is with a two-column matrix of row and column indices, just like the `inds` matrix we created above:

``` r
mat[inds]
```

    ## [1] 1 2 3 4

*Arrays* generalize matrices in *d* = 3 or more dimensions. Although we won't discuss them in this workshop, subsetting arrays works in the same way as matrices (i.e. with *d* comma-separated expressions or a *d*-column matrix).

Lists
-----

Lists are one-dimensional sequences, but unlike vectors, each position in the list can be filled with a different type of R object, including another list. Lists are created with `list`:

``` r
nested <- list(a = list(c(1,2), c("c","d")), b = c(18, 25))
str(nested)
```

    ## List of 2
    ##  $ a:List of 2
    ##   ..$ : num [1:2] 1 2
    ##   ..$ : chr [1:2] "c" "d"
    ##  $ b: num [1:2] 18 25

There are three ways to subset lists:

-   single brackets return a sub-list by element positions or names e.g. `nested[1:2]`, `nested["a"]`;
-   double brackets return a single element from the list e.g. `nested[[2]]`, `nested[["a"]]`;
-   the `$` sign has the same function as double brackets, but does not require quoting names e.g. `nested$a`.

Single brackets always return a list, even if it has a single element. Compare:

``` r
str(nested["b"][1])
```

    ## List of 1
    ##  $ b: num [1:2] 18 25

``` r
str(nested[["b"]][1])
```

    ##  num 18

**Q.**: Which combination of subsets gives the letter "c" from the list *nested*?

Data frames
-----------

Data frames are central to most R scripts. They superficially look like matrices, but are really implemented by R as lists. Creating a data frame with `data.frame` is similar to creating a list, except that each element (also called column, or variable) should have the same length:

``` r
df <- data.frame(a = 1:3, b = c("x","y","z"))
str(df)
```

    ## 'data.frame':    3 obs. of  2 variables:
    ##  $ a: int  1 2 3
    ##  $ b: Factor w/ 3 levels "x","y","z": 1 2 3

Data frames can be subset in the same way as lists: `df[1]` returns a one-column data frame, whereas `df[[1]]` or `df$a` return the contents of a column (a vector). However, they can also be subset with two expressions separated by a comma, like matrices:

``` r
df[2:3, c("a","b")]
```

    ##   a b
    ## 2 2 y
    ## 3 3 z
