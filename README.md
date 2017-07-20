Using dplyr Within Functions
========================================================
author: Nick Salkowski
date: July 20, 2017
autosize: true

dplyr Explained in One Slide
========================================================

- dplyr is an R package for working with data sets that works with the %>% operator
- dplyr functions (or 'verbs'):
  - filter(): choose rows
  - select(): choose columns
  - mutate(): modify columns or create new columns
  - summarize(): calculate summary statistics
  - group_by(): define subsets of interest for summarize()
  
dplyr with functions
========================================================


dplyr functions work well interactively

dplyr functions work well in scripts

But special care must be take to use dplyr within functions

Underscore Functions
======================================================

These functions are deprecated, but they do still work

Each dplyr function had an underscore version that uses standard evaluation, so they work inside functions.  

The underscore version of mutate() is mutate_()

The underscore version of summarize() is summarize_()

etc.

Summary in open code
==========================================

"Normal" Version:

```r
iris %>% summarize(mean_sl = mean(Sepal.Length))
```

```
   mean_sl
1 5.843333
```

.dots
======================================================
The underscore functions accept an argument named `.dots`

`.dots` is a named character vector (or named list)

"Underscore" Version:

```r
my_summary <- c(mean_sl = "mean(Sepal.Length)")
summarize_(iris, 
           .dots = my_summary)
```

```
   mean_sl
1 5.843333
```


setNames()
========================================================
The setNames() function is helpful!


```r
summarize_(iris, 
           .dots = setNames("mean(Sepal.Length)", 
                            "mean_sl"))
```

```
   mean_sl
1 5.843333
```


Underscore-Style Function
========================================================

```r
imeanit <- function(.data, x) {
  summarize_(.data,
             .dots = setNames(
               paste0(
                 "mean(",
                 x,
                 ")"),
               paste0("mean_",
                      x)))
}
imeanit(iris, "Sepal.Length")
```

```
  mean_Sepal.Length
1          5.843333
```

Multiple Inputs
=====================================


```r
iris %>% 
  imeanit(c("Sepal.Length", 
            "Sepal.Width",
            "Petal.Length", 
            "Petal.Width")) %>% 
  t
```

```
                      [,1]
mean_Sepal.Length 5.843333
mean_Sepal.Width  3.057333
mean_Petal.Length 3.758000
mean_Petal.Width  1.199333
```

Midpoint Example
==============================================

```r
iris %>% 
  group_by(Species) %>% 
  imeanit(c("Sepal.Length", 
            "Sepal.Width"))
```

```
# A tibble: 3 x 3
     Species mean_Sepal.Length mean_Sepal.Width
      <fctr>             <dbl>            <dbl>
1     setosa             5.006            3.428
2 versicolor             5.936            2.770
3  virginica             6.588            2.974
```

The NEW way: quosures!
======================================
`quo()` turns its argument into a "quosure", which is a special type of formula:


```r
q <- quo(Sepal.Length)
q
```

```
<quosure: global>
~Sepal.Length
```

Unquosurizing
=========================================
`!!` unquosurizes a quosure:


```r
iris %>% 
  summarize(mean(!!q))
```

```
  mean(Sepal.Length)
1           5.843333
```

enquo(): quosurizing function arguments!
=========================================


```r
ireallymeanit <- function(.data, x) {
  qx <- enquo(x)
  summarize(.data, mean(!!qx))
}
ireallymeanit(iris, Sepal.Length)
```

```
  mean(Sepal.Length)
1           5.843333
```

Naming Stuff (Part 1)
==========================================
Wouldn't it be nice to convert the quosure to a character string?

You could use that to name stuff.


```r
ireallymeanit <- function(.data, x) {
  qx <- enquo(x)
  qx_name <- paste0(
    "mean_",
    quo_name(qx))
  # summarize(.data, mean(!!qx))
  qx_name
}
iris %>% ireallymeanit(Sepal.Length)
```

```
[1] "mean_Sepal.Length"
```

Naming Stuff (Part 2)
======================================
The `:=` operator lets you name stuff:


```r
ireallymeanit <- function(.data, x) {
  qx <- enquo(x)
  qx_name <- paste0(
    "mean_",
    quo_name(qx))
  summarize(.data, !!qx_name := mean(!!qx))
  }
iris %>% ireallymeanit(Sepal.Length)
```

```
  mean_Sepal.Length
1          5.843333
```

Multiple Inputs (Again)
=============================================

```r
ireallymeanit <- function(.data, ...) {
  qx <- quos(...)
  qx_names <- lapply(qx, quo_name)
  qx <- lapply(qx, function(z) quo(mean(!!z)))
  qx <- setNames(qx,
                 paste0("mean_", qx_names))
  summarize(.data, !!! qx)
}
iris %>% ireallymeanit(Sepal.Length, Sepal.Width)
```

```
  mean_Sepal.Length mean_Sepal.Width
1          5.843333         3.057333
```

Final Example
==============================================

```r
iris %>% 
  group_by(Species) %>% 
  ireallymeanit(Sepal.Length, Sepal.Width)
```

```
# A tibble: 3 x 3
     Species mean_Sepal.Length mean_Sepal.Width
      <fctr>             <dbl>            <dbl>
1     setosa             5.006            3.428
2 versicolor             5.936            2.770
3  virginica             6.588            2.974
```

For more info:
=======================================================

`?mutate_`

<https://cran.r-project.org/web/packages/dplyr/vignettes/programming.html>



