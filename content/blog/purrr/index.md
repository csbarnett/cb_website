---
title: "Introduction to the purrr package"
author: "Chris Barnett"
date: 2022-11-15
subtitle: ""
excerpt: "Working with functions and vectors is mandatory for almost any programming language and working with data in R is no exception. The purrr package is the functional programming workhorse of R packages, making working with functions, list and vectors succinct and easier to read."
output:
  html_document:
    toc: yes
    toc_float: yes
---

<style type="text/css">
h1.title { /* Header 4 - and the author and date headers use this too  */
  font-size: 40px;
  font-style: normal;
  font-weight: bold;
  font-family: Tahoma, Verdana, sans-serif;
  color: Black;
}

h4.author { /* Header 4 - and the author and date headers use this too  */
  font-size: 20px;
  font-style: normal;
  font-family: Tahoma, Verdana, sans-serif;
  color: Black;

}
h4.date { /* Header 4 - and the author and date headers use this too  */
  font-size: 20px;
  font-family: Tahoma, Verdana, sans-serif;
  color: Black;
}

h2 {/* Header 2 */
  font-size: 24px;
  font-family: Tahoma, Verdana, sans-serif;
  color: Black;
  background-color: WhiteSmoke;
  text-indent: 5px;
}
</style>

---

![](https://purrr.tidyverse.org/logo.png)
# Introduction

The best resource for the purrr package is likely the [Iteration chapter](https://r4ds.had.co.nz/iteration.html#the-map-functions) in R for Data Science by Garrett Grolemund and Hadley Wickham. Hadley Wickham is one of the authors of the purrr package and he spends a good deal of the chapter clearly explaining how it works. It is definitely recommended further reading. And for those of us in public health there is also a chapter on purrr in [R for Epidemiology](https://www.r4epi.com/using-the-purrr-package.html).

The purpose of this markdown document is to provide a short introduction to purrr, focusing on just a handful of functions.

## What is the purrr package used for?
Answer: The process of looping over a vector, applying a function or change to each element, and saving the results is common in R programming. The purrr package provide tools accomplish this that are more succinct and easier to read.

## Can I just use base R?
Answer: Yes. All of this is absolutely possible with base R, using for() loops or the “apply” functions, such as lapply(), and by()






```r
# Creating a list of character objects (public health regions)
public_health_regions <- list("Anchorage", "Mat-Su", "Interior", "Northern", "Southeast", "Southwest", "Gulf Coast")

# Detecting the string pattern "South" in list with lapply fuction
lapply(public_health_regions,
       function(x){str_detect(string = x, pattern = "South")}
       )
```

```
## [[1]]
## [1] FALSE
## 
## [[2]]
## [1] FALSE
## 
## [[3]]
## [1] FALSE
## 
## [[4]]
## [1] FALSE
## 
## [[5]]
## [1] TRUE
## 
## [[6]]
## [1] TRUE
## 
## [[7]]
## [1] FALSE
```

```r
# Detecting the string pattern "South" in list with purrr:::map fuction
map(public_health_regions,
       function(x){str_detect(string = x, pattern = "South")}
       )
```

```
## [[1]]
## [1] FALSE
## 
## [[2]]
## [1] FALSE
## 
## [[3]]
## [1] FALSE
## 
## [[4]]
## [1] FALSE
## 
## [[5]]
## [1] TRUE
## 
## [[6]]
## [1] TRUE
## 
## [[7]]
## [1] FALSE
```
The code and output of the map() and lapply functions are nearly identical. However, the purrr package comes with a more well defined and **consistent** syntax and a ease of use over base R looping functions. The user interface of the “apply”" functions is not as consistent and all previous looping functions pre-date the pipe %>% operator. 


## What else can the purrr package do?
In addition, the purrr package includes a host of useful functions for looping over vectors and list. The [purrr cheatsheet](https://github.com/rstudio/cheatsheets/blob/main/purrr.pdf) is particully helpful.

```r
# Filtering on list items that contain the string pattern "South"
# Keep() function selects list items that pass a logical test
keep(public_health_regions,
       function(x){str_detect(string = x, pattern = "South")}
       )
```

```
## [[1]]
## [1] "Southeast"
## 
## [[2]]
## [1] "Southwest"
```

```r
# Pluck a single element from a vector or environment 
pluck(.x = public_health_regions, 1)
```

```
## [1] "Anchorage"
```



```r
# Creating a function that converts string object "Mat-Su" 
# to "Matanuska-Susitna"
matsu <- function(x){
  ifelse(x == "Mat-Su", "Matanuska-Susitna", x)
}

# Modifying an element within a list without removing it from the list
modify(.x = public_health_regions, .f = matsu)
```

```
## [[1]]
## [1] "Anchorage"
## 
## [[2]]
## [1] "Matanuska-Susitna"
## 
## [[3]]
## [1] "Interior"
## 
## [[4]]
## [1] "Northern"
## 
## [[5]]
## [1] "Southeast"
## 
## [[6]]
## [1] "Southwest"
## 
## [[7]]
## [1] "Gulf Coast"
```
## The purrr package works seemlessly with piping operators


```r
head(mtcars, 4)
```

```
##                 mpg cyl disp  hp drat    wt  qsec vs am gear carb
## Mazda RX4      21.0   6  160 110 3.90 2.620 16.46  0  1    4    4
## Mazda RX4 Wag  21.0   6  160 110 3.90 2.875 17.02  0  1    4    4
## Datsun 710     22.8   4  108  93 3.85 2.320 18.61  1  1    4    1
## Hornet 4 Drive 21.4   6  258 110 3.08 3.215 19.44  1  0    3    1
```

```r
# How does the number of cylinders change the relationship between
# car miles per gallon and car weight?
mtcars %>%
  split(.$cyl) %>% # Spliting mtcars into 3 cyl dataframes 
  map(~ lm(mpg ~ wt, data = .)) %>% # fitting a linear model on all 3 dataframes
  map(summary) %>% # Obtaining model summaries on all 3 models
  map_dbl("r.squared")
```

```
##         4         6         8 
## 0.5086326 0.4645102 0.4229655
```
