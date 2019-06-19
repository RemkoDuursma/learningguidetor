# Functions, lists and loops {#programming}


## Introduction

This chapter demonstrates some building blocks of programming with R. The main purpose here is to allow batch analyses - repeating similar tasks for many subsets of data. To do this, we first have to know how to write our own functions, so that we can apply the custom function to any new subset of data. This approach results in far less, and much more readable code than copy-pasting similar code for different datasets.

We also delve into "lists" in R, a versatile data object that you have worked with already - even if you didn't know it. Understanding lists well in R is the key to more complex analyses, and more readable workflows.

Finally we take a brief look at 'for loops', a basic programming utility that we rarely need in R, since almost all functions are vectorized. Sometimes it is however more convenient to use loops, or makes the code just a little more easy to work with.






**Packages used in this chapter**

We use no new packages in this chapter except the `wrapr` package. All other functionality is included in base R, or commonly used packages `dplyr`, `ggplot2`, `lubridate`, and `lgrdata` for the example data.


## Writing simple functions {#writefunctions}

We have already used many built-in functions throughout this tutorial, but you can become very efficient at complex data tasks when you write your own simple functions. Writing your own functions can help with tasks that are carried out many times, which would otherwise result in a lot of code.

For example, suppose you frequently convert units from pounds to kilograms. It would be useful to have a function that does this, so you don't have to type the conversion factor every time. This is also good practice, as it reduces the probability of making typos.


```r
# This function takes a 'weight' argument and multiplies it with some number 
# to return kilograms.
poundsToKg <- function(weight){
  weight * 0.453592
}
```

We can use this `function` just like any other in R, for example, let's convert 'weight' to kilograms in the weightloss data.

```r
# Read data
library(lgrdata)
data(weightloss)

# Convert weight to kg.
weightloss$Weight <- poundsToKg(weightloss$Weight)
```

Let's write a function for the standard error of the mean, a function that is not built-in in R. 


```r
# Compute the standard error of the mean for a vector
SEmean <- function(x){
  se <- sd(x) / sqrt(length(x))
  return(se)
}
```

Here, the `function` SEmean takes one 'argument' called `x` (i.e., input), which is a numeric vector. The standard error for the mean is calculated in the first line, and stored in an object called `se`, which is then returned as output. We can now use the function on a numeric vector like this:


```r
# A numeric vector
unifvec <- runif(10, 1,2)

# The sample mean
mean(unifvec)
```

```
## [1] 1.540123
```

```r
# Standard error for the mean
SEmean(unifvec)
```

```
## [1] 0.09165783
```


\BeginKnitrBlock{rmdtry}<div class="rmdtry">You can use functions that you defined yourself just like any other function, for example in `summaryBy`. First read in the `SEmean` function defined in the example above, and then use the cereal data to calculate the mean and SE of `rating` by `Manufacturer` (or use data of your choosing).</div>\EndKnitrBlock{rmdtry}


### Functions with many arguments

Functions can also have multiple arguments. The following very simple function takes two numbers, and finds the absolute difference between them, using `abs`.

```r
# Define function
absDiff <- function(num1,num2)abs(num1 - num2)

# Test it with two numbers:
absDiff(5,1)
```

```
## [1] 4
```

```r
# As in many functions in R, you can also give multiple values
# as an argument.
# The following returns the absolute difference between 
# 1 and 3, then 5 and 6, and 9 and 0 (in that order).
absDiff(c(1,5,9), c(3,6,0))
```

```
## [1] 2 1 9
```


### Functions can return many results

What if a function should return not just one result, as in the examples above, but many results? 

For example, this function computes the standard deviation and standard error of a vector, and returns both stored in a vector. Note that we also use the `SEmean` function, which we defined above.  


```r
# An function that computes the SE and SD of a vector
seandsd <- function(x){
  
  seresult <- SEmean(x)
  sdresult <- sd(x)

  # Store results in a vector with names
  vec <- c(seresult, sdresult)
  names(vec) <- c("SE","SD")

return(vec)
}

# Test it:
x <- rnorm(100, mean=20, sd=4)
seandsd(x)
```

```
##        SE        SD 
## 0.4024563 4.0245630
```

### Functions without arguments

Sometimes, a function takes no arguments (input) at all. Consider this very helpful example. 


```r
sayhello <- function()message("Hello!")

sayhello()
```

```
## Hello!
```

We will return to defining our own functions when we look at applying functions many times to sections of a dataframe (Section \@ref(lapply)).


### Wrapper functions {#wrapfunctions}

We often need to write simple functions that adjust one or two arguments to other functions. For example, suppose we often make plots with filled circles, with our favorit color ("dimgrey") :


```r
library(lgrdata)
data(howell)
plot(age, height, data=howell, pch=19, col="dimgrey")
```

We could of course *always* specify these arguments, or we can write a function that sets those defaults. It would look like the following, except this function is incomplete, since we have *hardcoded* the other plotting arguments (the dataset, and the x and y variables):


```r
# This function is not how we want it yet!
plot_filled_grey <- function(){
  plot(age, height, data=howell, pch=19, col="dimgrey")
}
```

We would like to be able to call the function via `plot_filled_grey(age, height, data=howell)`, in other words all arguments to our *wrapper function* should be passed to the underlying function. We have a very handy utility in R to do this, the three dots (`...`) :


```r
plot_filled_grey <- function(...){
  plot(..., pch=19, col="dimgrey")
}
```

The function now works as intended. It can be further improved if we realize that the plotting color cannot be changed - it is always "dimgrey". We want the *default* value to be "dimgrey", but with an option to change it. This can be achieved like so,


```r
plot_filled_grey <- function(..., col="dimgrey"){
  plot(..., pch=19, col=col)
}
```

Here, `col=col` sets the color in the call to `plot` with the default value specified in the wrapper function ("dimgrey"), but the user can also change it as usual.

\BeginKnitrBlock{rmdtry}<div class="rmdtry">Take the `plot_filled_grey` function above, test it on some data, and modify it so that the plotting symbol can also be changed, but has a default value of 19.</div>\EndKnitrBlock{rmdtry}



### Wrapper functions to `ggplot2` or `dplyr`

In the previous section we saw how to write wrapper functions, functions that change a few arguments to some other function. These sort of functions are very helpful because we can save a lot of space by reusing a certain template. Suppose we want to make a plot with `ggplot2`, a scatter plot with a loess smoother line. We can achieve this via (result not shown),


```r
data(howell)
library(ggplot2)

ggplot(howell, aes(x = weight, y = height)) +
  geom_point(size = 0.8, col = "dimgrey") +
  stat_smooth(method = "loess", span = 0.7, col="black")
```

We already used quite a bit of code for this simple plot, but imagine you have set various other options, formatting changes, axis limits and so on - you end up with a lot of code for one plot. If we want to reuse the code for another plot, for two other variables from the same dataframe, copy-pasting the code and modifying leads to even more code. Writing wrapper functions for `ggplot2` (or `dplyr`, see below, or many other cases) is more difficult because the arguments in the `ggplot2` code we want to change (height and weight) *are not quoted* - they are variables inside a dataframe (`howell` in our case). We avoid a more technical explanation of this problem, but simply present a solution with the `wrapr` package.

Using `let` from `wrapr`, we can turn our unquoted variables into quoted ones, like so:


```r
library(wrapr)

let(c(xvar = "weight", yvar = "height"), {
  ggplot(howell, aes(x = xvar, y = yvar)) +
  geom_point(size = 0.8, col = "dimgrey")
})
```

The point of placing our plotting code inside `let` is that we can now write our simple wrapper function like before:


```r
# Make a scatter plot with the howell data.
plot_scatter_howell <- function(xcol, ycol){
  let(c(xvar = xcol, yvar = ycol), {
    ggplot(howell, aes(x = xvar, y = yvar)) +
    geom_point(size = 0.8, col = "dimgrey")
  })
}

# The function can be used with quoted names:
plot_scatter("height", "weight")
```

It is important to understand that, in this example, `let` is used *only to turn character arguments into unquoted names*, in this case "height" turns into `height` for use in `ggplot`, and so on. We cannot pass `height` and `weight` as unquoted names, because R would look for those objects directly (from the 'global environment'), rather than as variables in the `howell` dataset.

To also pass the dataset as an argument (making the function more general), we do not have to use `let`, but can immediately set it as an argument (because it is available in the global environment).


```r
# Our function now also takes a dataset as an argument
plot_scatter <- function(xcol, ycol, dataset){
  let(c(xvar = xcol, yvar = ycol), {
    ggplot(dataset, aes(x = xvar, y = yvar)) +
    geom_point(size = 0.8, col = "dimgrey")
  })
}

# We can use it as,
plot_scatter("height", "weight", howell)
```





## Working with lists {#workinglists}

Sofar, we have worked a lot with vectors, with are basically strings of numbers or bits of text. In a vector, each element has to be of the same data type. Lists are a more general and powerful type of vector, where each element of the `list` can be anything at all. This way, lists are a very flexible type of object to store a lot of information that may be in different formats.

Lists can be somewhat daunting for the beginning `R` user, which is why most introductory texts and tutorials skip them altogether. However, with some practice, lists can be mastered from the start. Mastering a few basic skills with lists can really help increase your efficiency in dealing with more complex data analysis tasks.

To make a list from scratch, you simply use the `list` function. Here is a list that contains a numeric vector, a character vector, and a dataframe:


```r
mylist <- list(a=1:10, txt=c("hello","world"), dfr=data.frame(x=c(2,3,4),y=c(5,6,7)))
```

### Indexing lists

To extract an element from this list, you may do this by its name ('a','txt' or 'dfr' in this case), or by the element number (1,2,3). For lists, we use a double square bracket for indexing. Consider these examples,


```r
# Extract the dataframe:
mylist[["dfr"]]
```

```
##   x y
## 1 2 5
## 2 3 6
## 3 4 7
```

```r
# Is the same as:
mylist$dfr
```

```
##   x y
## 1 2 5
## 2 3 6
## 3 4 7
```

```r
# Extract the first element:
mylist[[1]]
```

```
##  [1]  1  2  3  4  5  6  7  8  9 10
```

Note that in these examples, the contents of the elements of the list are returned (for 'dfr', a dataframe), but the result itself is not a list anymore. If we select multiple elements, the result should still be a list. To do this, use the single square bracket.

Look at these examples:

```r
# Extract the 'a' vector, result is a vector:
mylist[['a']]
```

```
##  [1]  1  2  3  4  5  6  7  8  9 10
```

```r
# Extract the 'a' vector, result is a list:
mylist['a']
```

```
## $a
##  [1]  1  2  3  4  5  6  7  8  9 10
```

```r
# Extract multiple elements (result is still a list):
mylist[2:3]
```

```
## $txt
## [1] "hello" "world"
## 
## $dfr
##   x y
## 1 2 5
## 2 3 6
## 3 4 7
```


### Converting lists to dataframes or vectors

Although lists are the most flexible way to store data and other objects in larger, more complex, analyses, ultimately you would prefer to output as a dataframe or vector.

Let's look at some examples using `do.call(rbind,...)` and `unlist`.

```r
# A list of dataframes:
dfrlis <- list(data1=data.frame(a=1:3,b=2:4), data2=data.frame(a=9:11,b=15:17))
dfrlis
```

```
## $data1
##   a b
## 1 1 2
## 2 2 3
## 3 3 4
## 
## $data2
##    a  b
## 1  9 15
## 2 10 16
## 3 11 17
```

```r
# Since both dataframes in the list have the same number of columns and names, 
# we can 'successively row-bind' the list like this:
do.call(rbind, dfrlis)
```

```
##          a  b
## data1.1  1  2
## data1.2  2  3
## data1.3  3  4
## data2.1  9 15
## data2.2 10 16
## data2.3 11 17
```

```r
# A list of vectors:
veclis <- list(a=1:3, b=2:4, f=9:11)

# In this case, we can use the 'unlist' function, which will 
# successively combine the three vectors into one:
unlist(veclis)
```

```
## a1 a2 a3 b1 b2 b3 f1 f2 f3 
##  1  2  3  2  3  4  9 10 11
```

In real-world applications, some trial-and-error will be necessary to convert lists to more pretty formats.



### Combining lists

Combining two lists can be achieved using `c()`, like this:


```r
veclis <- list(a=1:3, b=2:4, f=9:11)
qlis <- list(q=17:15)
c(veclis,qlis)
```

```
## $a
## [1] 1 2 3
## 
## $b
## [1] 2 3 4
## 
## $f
## [1]  9 10 11
## 
## $q
## [1] 17 16 15
```

```r
# But be careful when you like to quickly add a vector
# the 'veclis'. You must specify list() like this
veclis <- c(veclis, list(r=3:1))
```


### Extracting output from built-in functions

One reason to gain a better understanding of lists is that many built-in functions return not just single numbers, but a diverse collection of outputs, organized in lists. Think of the linear model function (`lm`), it returns a lot of things at the same time (not just the p-value).

Let's take a closer look at the `lm` output to see if we can extract the adjusted R$^2$. 


```r
# Read data
data(allometry)

# Fit a linear model
lmfit <- lm(height ~ diameter, data=allometry)

# And save the summary statement of the model:
lmfit_summary <- summary(lmfit)

# We already know that simply typing 'summary(lmfit)' will give 
# lots of text output. How to extract numbers from there?
# Let's look at the structure of lmfit:
str(lmfit_summary)
```

```
## List of 11
##  $ call         : language lm(formula = height ~ diameter, data = allometry)
##  $ terms        :Classes 'terms', 'formula'  language height ~ diameter
##   .. ..- attr(*, "variables")= language list(height, diameter)
##   .. ..- attr(*, "factors")= int [1:2, 1] 0 1
##   .. .. ..- attr(*, "dimnames")=List of 2
##   .. .. .. ..$ : chr [1:2] "height" "diameter"
##   .. .. .. ..$ : chr "diameter"
##   .. ..- attr(*, "term.labels")= chr "diameter"
##   .. ..- attr(*, "order")= int 1
##   .. ..- attr(*, "intercept")= int 1
##   .. ..- attr(*, "response")= int 1
##   .. ..- attr(*, ".Environment")=<environment: R_GlobalEnv> 
##   .. ..- attr(*, "predvars")= language list(height, diameter)
##   .. ..- attr(*, "dataClasses")= Named chr [1:2] "numeric" "numeric"
##   .. .. ..- attr(*, "names")= chr [1:2] "height" "diameter"
##  $ residuals    : Named num [1:63] -8.84 1.8 0.743 2.499 4.37 ...
##   ..- attr(*, "names")= chr [1:63] "1" "2" "3" "4" ...
##  $ coefficients : num [1:2, 1:4] 7.5967 0.5179 1.4731 0.0365 5.157 ...
##   ..- attr(*, "dimnames")=List of 2
##   .. ..$ : chr [1:2] "(Intercept)" "diameter"
##   .. ..$ : chr [1:4] "Estimate" "Std. Error" "t value" "Pr(>|t|)"
##  $ aliased      : Named logi [1:2] FALSE FALSE
##   ..- attr(*, "names")= chr [1:2] "(Intercept)" "diameter"
##  $ sigma        : num 5.55
##  $ df           : int [1:3] 2 61 2
##  $ r.squared    : num 0.768
##  $ adj.r.squared: num 0.764
##  $ fstatistic   : Named num [1:3] 202 1 61
##   ..- attr(*, "names")= chr [1:3] "value" "numdf" "dendf"
##  $ cov.unscaled : num [1:2, 1:2] 7.05e-02 -1.54e-03 -1.54e-03 4.32e-05
##   ..- attr(*, "dimnames")=List of 2
##   .. ..$ : chr [1:2] "(Intercept)" "diameter"
##   .. ..$ : chr [1:2] "(Intercept)" "diameter"
##  - attr(*, "class")= chr "summary.lm"
```

```r
# The output of lm is a list, so we can look at the names of # that list as well:
names(lmfit_summary)
```

```
##  [1] "call"          "terms"         "residuals"     "coefficients" 
##  [5] "aliased"       "sigma"         "df"            "r.squared"    
##  [9] "adj.r.squared" "fstatistic"    "cov.unscaled"
```

So, now we can extract results from the summary of the fitted regression. Also look at the help file `?summary.lm`, in the section 'Values' for a description of the fields contained here.

To extract the adjusted R$^2$, for example:

```r
lmfit_summary[["adj.r.squared"]]
```

```
## [1] 0.7639735
```

```r
# Is the same as:
lmfit_summary$adj.r.squared
```

```
## [1] 0.7639735
```


This sort of analysis will be very useful when we do many regressions, and want to summarize the results in a table.

\BeginKnitrBlock{rmdtry}<div class="rmdtry">Run the code in the above examples, and practice extracting some other elements from the linear regression. Compare the output to the summary of the `lm` fit (that is, compare it to what `summary(lmfit)` shows on screen).</div>\EndKnitrBlock{rmdtry}


### Creating lists from dataframes {#dfrlists}

For more advanced analyses, it is often necessary to repeat a particular analysis many times, for example for sections of a dataframe. 

Using the `allom` data for example, we might want to split the dataframe into three dataframes (one for each species), and repeat some analysis for each of the species. One option is to make three subsets (using `subset`), and repeating the analysis for each of them. But what if we have hundreds of species? 

A more efficient approach is to `split` the dataframe into a list, so that the first element of the list is the dataframe for species 1, the 2nd element species 2, and so on. In case of the allom dataset, the resulting list will have three components. 

Let's look at an example on how to construct a list of dataframes from the allom dataset, one per species:


```r
# Read allom data and make sure 'species' is a factor:
data(allometry)
is.factor(allometry$species)
```

```
## [1] TRUE
```

```r
# The levels of the factor variable 'species'
levels(allometry$species)
```

```
## [1] "PIMO" "PIPO" "PSME"
```

```r
# Now use 'split' to construct a list:
allomsp <- split(allometry, allometry$species)

# The length of the list should be 3, with the names equal to the 
# original factor levels:
length(allomsp)
```

```
## [1] 3
```

```r
names(allomsp)
```

```
## [1] "PIMO" "PIPO" "PSME"
```


\BeginKnitrBlock{rmdtry}<div class="rmdtry">Run the code in the above example, and confirm that `allomsp[[2]]` is identical to taking a subset of `allom` of the second species in the dataset (where 'second' refers to the second level of the factor variable `species`, which you can find out with `levels`).</div>\EndKnitrBlock{rmdtry}

Let's look at an example using the `hydro` data. The data contains water levels of a hydrodam in Tasmania, from 2005 to 2011.


```r
# Read hydro data, and convert Date to a proper date class.
data(hydro)

library(lubridate)
library(dplyr)

hydro <- mutate(hydro, 
                Date = dmy(Date),
                year = year(Date))

# Look at the Date range:
range(hydro$Date)
```

```
## [1] "2005-08-08" "2011-08-08"
```

```r
# Let's get rid of the first and last years (2005 and 2011) since they are incomplete
hydro <- filter(hydro, !year %in% c(2005,2011))

# Now split the dataframe by year. This results in a list, where every
# element contains the data for one year:
hydrosp <- split(hydro, hydro$year)

# Properties of this list:
length(hydrosp)
```

```
## [1] 5
```

```r
names(hydrosp)
```

```
## [1] "2006" "2007" "2008" "2009" "2010"
```

To extract one element of the two lists that we created (`allomsp` and `hydrosp`), recall the section on indexing lists.


### Applying functions to lists {#lapply}

We will introduce two basic tools that we use to apply functions to each element of a list: `sapply` and `lapply`. The `lapply` function always returns a list, whereas `sapply` will attempt to `s`implify the result. When the function returns a single value, or a vector, `sapply` can often be used. In practice, try both and see what happens!


#### Using `sapply`

First let's look at some simple examples:


```r
# Let's make a simple list with only numeric vectors (of varying length)
numlis <- list(x=1000, y=c(2.1,0.1,-19), z=c(100,200,100,100))

# For the numeric list, let's get the mean for every element, and count 
# the length of the three vectors.
# Here, sapply takes a list and a function as its two arguments,
# and applies that function to each element of the list.
sapply(numlis, mean)
```

```
##      x      y      z 
## 1000.0   -5.6  125.0
```

```r
sapply(numlis, length)
```

```
## x y z 
## 1 3 4
```


You can of course also define your own functions, and use them here. Let's look at another simple example using the `numlis` object defined above. 

For example,  

```r
# Let's find out if any diameters are duplicated in the allom dataset.
# A function that does this would be the combination of 'any' and 'duplicated',
anydup <- function(vec)any(duplicated(vec))
# This function returns TRUE or FALSE

# Apply this function to numlis (see above):
sapply(numlis, anydup)
```

```
##     x     y     z 
## FALSE FALSE  TRUE
```

```r
# You can also define the function on the fly like this:
sapply(numlis, function(x)any(duplicated(x)))
```

```
##     x     y     z 
## FALSE FALSE  TRUE
```


Now, you can use any function in `sapply` as long as it returns a single number based on the element of the list that you used it on. Consider this example with `strsplit`.


```r
# Recall that the 'strsplit' (string split) function usually returns a list of values. 
# Consider the following example, where the data provider has included the units in 
# the measurements of fish lengths. How do we extract the number bits?
fishlength <- c("120 mm", "240 mm", "159 mm", "201 mm")

# Here is one solution, using strsplit
strsplit(fishlength," ")
```

```
## [[1]]
## [1] "120" "mm" 
## 
## [[2]]
## [1] "240" "mm" 
## 
## [[3]]
## [1] "159" "mm" 
## 
## [[4]]
## [1] "201" "mm"
```

```r
# We see that strsplit returns a list, let's use sapply to extract only 
# the first element (the number)
splitlen <- strsplit(fishlength," ")
sapply(splitlen, function(x)x[1])
```

```
## [1] "120" "240" "159" "201"
```

```r
# Now all you need to do is use 'as.numeric' to convert these bits of text to numbers.
```

The main purpose of splitting dataframes into lists, as we have done above, is so that we can save time with analyses that have to be repeated many times. In the following examples, you must have already produced the objects `hydrosp` and `allomsp` (from examples in the previous section).Both those objects are *lists of dataframes*, that is, each element of the list is a dataframe in itself. Let's look at a few examples with `sapply` first.


```r
# How many observations per species in the allom dataset?
sapply(allomsp, nrow)
```

```
## PIMO PIPO PSME 
##   19   22   22
```

```r
# Here, we applied the 'nrow' function to each separate dataframe.
# (note that there are easier ways to find the number of observations per species!,
# this is just illustrating sapply.)

# Things get more interesting when you define your own functions on the fly:
sapply(allomsp, function(x)range(x$diameter))
```

```
##       PIMO  PIPO  PSME
## [1,]  6.48  4.83  5.33
## [2,] 73.66 70.61 69.85
```

```r
# Here, we define a function that takes 'x' as an argument:
# sapply will apply this function to each element of the list,
# one at a time. In this case, we get a matrix with ranges of the diameter per species.

# How about the correlation of two variables, separate by species:
sapply(allomsp, function(x)cor(x$diameter, x$height))
```

```
##      PIMO      PIPO      PSME 
## 0.9140428 0.8594689 0.8782781
```

```r
# For hydro, find the number of days that storage was below 235, for each year.
sapply(hydrosp, function(x)sum(x$storage < 235))
```

```
## 2006 2007 2008 2009 2010 
##    0   18    6    0    0
```


#### Using `lapply`


The `lapply` function is much like `sapply`, except it always returns a list.

For example, 

```r
# Get a summary of the hydro dataset by year:
lapply(hydrosp, summary)
```

```
## $`2006`
##       Date               storage           year     
##  Min.   :2006-01-02   Min.   :411.0   Min.   :2006  
##  1st Qu.:2006-04-01   1st Qu.:449.5   1st Qu.:2006  
##  Median :2006-06-29   Median :493.0   Median :2006  
##  Mean   :2006-06-29   Mean   :514.3   Mean   :2006  
##  3rd Qu.:2006-09-26   3rd Qu.:553.8   3rd Qu.:2006  
##  Max.   :2006-12-25   Max.   :744.0   Max.   :2006  
## 
## $`2007`
##       Date               storage           year     
##  Min.   :2007-01-01   Min.   :137.0   Min.   :2007  
##  1st Qu.:2007-04-02   1st Qu.:223.0   1st Qu.:2007  
##  Median :2007-07-02   Median :319.0   Median :2007  
##  Mean   :2007-07-02   Mean   :363.2   Mean   :2007  
##  3rd Qu.:2007-10-01   3rd Qu.:477.0   3rd Qu.:2007  
##  Max.   :2007-12-31   Max.   :683.0   Max.   :2007  
## 
## $`2008`
##       Date               storage           year     
##  Min.   :2008-01-07   Min.   :202.0   Min.   :2008  
##  1st Qu.:2008-04-05   1st Qu.:267.8   1st Qu.:2008  
##  Median :2008-07-03   Median :345.0   Median :2008  
##  Mean   :2008-07-03   Mean   :389.5   Mean   :2008  
##  3rd Qu.:2008-09-30   3rd Qu.:551.8   3rd Qu.:2008  
##  Max.   :2008-12-29   Max.   :637.0   Max.   :2008  
## 
## $`2009`
##       Date               storage           year     
##  Min.   :2009-01-05   Min.   :398.0   Min.   :2009  
##  1st Qu.:2009-04-04   1st Qu.:467.0   1st Qu.:2009  
##  Median :2009-07-02   Median :525.5   Median :2009  
##  Mean   :2009-07-02   Mean   :567.6   Mean   :2009  
##  3rd Qu.:2009-09-29   3rd Qu.:677.8   3rd Qu.:2009  
##  Max.   :2009-12-28   Max.   :791.0   Max.   :2009  
## 
## $`2010`
##       Date               storage           year     
##  Min.   :2010-01-04   Min.   :251.0   Min.   :2010  
##  1st Qu.:2010-04-03   1st Qu.:316.0   1st Qu.:2010  
##  Median :2010-07-01   Median :368.5   Median :2010  
##  Mean   :2010-07-01   Mean   :428.5   Mean   :2010  
##  3rd Qu.:2010-09-28   3rd Qu.:577.5   3rd Qu.:2010  
##  Max.   :2010-12-27   Max.   :649.0   Max.   :2010
```

Suppose you have multiple similar datasets in your working directory, and you want to read all of these into one list, use `lapply` like this (run this example yourself and inspect the results).

```r
# Names of your datasets:
filenames <- c("pupae.csv","pupae.csv","pupae.csv")
# (This toy example will read the same file three times).

# Read all files into one list,
alldata <- lapply(filenames, read.csv)

# Then, if you are sure the datasets have the same number of columns and names,
# use do.call to collapse the list:
dfrall <- do.call(rbind, alldata)
```

\BeginKnitrBlock{rmdtry}<div class="rmdtry">Recall the use of `dir` to list files, and even to find files that match a specific pattern (see Section \@ref(fileswd). Read all CSV files in your working directory (or elsewhere) into a single list, and count the number of rows for each dataframe.</div>\EndKnitrBlock{rmdtry}

Finally, we can use `lapply` to do all sorts of complex analyses that return any kind of object. The use of `lapply` with lists ensures that we can organize even large amounts of data in this way.

Let's do a simple linear regression of log(leafarea) on log(diameter) for the Allometry dataset, by species:


```r
# Run the linear regression on each element of the list, store in a new object:
lmresults <- lapply(allomsp, function(x)lm(log10(leafarea) ~ log10(diameter), data=x))

# Now, lmresults is itself a list (where each element is an object as returned by lm)
# We can extract the coefficients like this:
sapply(lmresults, coef)
```

```
##                       PIMO       PIPO       PSME
## (Intercept)     -0.3570268 -0.7368336 -0.3135996
## log10(diameter)  1.5408859  1.6427773  1.4841361
```

```r
# This shows the intercept and slope by species.
# Also look at (results not shown):
# lapply(lmresults, summary) 

# Get R2 for each model. First write a function that extracts it.
getR2 <- function(x)summary(x)$adj.r.squared
sapply(lmresults, getR2)
```

```
##      PIMO      PIPO      PSME 
## 0.8738252 0.8441844 0.6983126
```

\BeginKnitrBlock{rmdtry}<div class="rmdtry">Try to fully understand the difference between `sapply` and `lapply` by using `lapply` in some of the examples where we used `sapply` (and vice versa).</div>\EndKnitrBlock{rmdtry}


## Loops {#simpleloops}

Loops can be useful when we need to repeat certain analyses many times, and it is difficult to achieve this with `lapply` or `sapply`. To understand how a `for` loop works, look at this example:


```r
for(i in 1:5){
  message(i)
}
```

```
## 1
```

```
## 2
```

```
## 3
```

```
## 4
```

```
## 5
```

Here, the bit of code between {} is executed five times, and the object `i` has the values 1,2,3,4 and 5, in that order. Instead of just printing `i` as we have done above, we can also index a vector with this object:


```r
# make a vector
myvec <- round(runif(5),1)

for(i in 1:length(myvec)){
  message("Element ", i, " of the vector is: ", myvec[i])
}
```

```
## Element 1 of the vector is: 0.3
```

```
## Element 2 of the vector is: 0.3
```

```
## Element 3 of the vector is: 0.6
```

```
## Element 4 of the vector is: 0.4
```

```
## Element 5 of the vector is: 0.4
```

Note that this is only a toy example: the same result can be achieved by simply typing `myvec`.

Now let's look at a useful application of a `for` loop: producing multiple plots in a `pdf`, using the `allomsp` object we created earlier.

This bit of code produces a `pdf` in your current working directory. If you can't find it, recall that you can use `getwd`() to get the current working directory.


```r
# Open a pdf to send the plots to:
pdf("Allom plot by species.pdf", onefile=TRUE)
for(i in 1:3){
  with(allomsp[[i]],
       plot(diameter, leafarea, pch=15, xlim=c(0,80), ylim=c(0,450),
            main=levels(allom$species)[i]))
}
# Close the pdf (important!)
dev.off()
```

Here, we create three plots (`i` goes from 1 to 3), every time using a different element of the list `allomsp`. First, `i` will have the value 1, so that we end up using the dataframe `allomsp[[1]]`, the first element of the list. And so on. Take a look at the resulting PDF to understand how the code works.

*Note:* On windows (with Adobe reader) If the pdf (`Allom plot by species.pdf`) is open, the above will fail. If you try this anyway, close the pdf and try again. You may have to run the command `dev.off`() another time to make sure the device is ready.


Another way to achieve the same result is to avoid splitting the dataframe into a list first, and simply take subsets on the fly. Consider this template (make your own working example based on any dataset).

We assume here you have a dataframe called 'dataset' with a factor 'species', for which you want to create separate plots of Y vs. X.


```r
pdf("somefilename.pdf", onefile=TRUE)
for(lev in levels(dataset$species)){
  
  with(subset(dataset, species==lev),
       plot(X,Y, 
            main=as.character(lev)))
}
dev.off()
```


## Exercises



### Writing functions


1.  Write a function that adds two numbers, and divides the result by 2.



2.  You learned in Section \@ref(workingtext) that you can take subset of a string using the `substr` function. First, using that function to extract the first 2 characters of a bit of text. Then, write a function called `firstTwoChars` that extracts the first two characters of any bit of text.




3.  Write a function that checks if there are any missing values in a vector (using `is.na` and `any`). The function should return `TRUE` if there are missing values, and `FALSE` if not.




4.  Improve the function so that it tells you which of the values are missing, if any (*Hint:*use the `which` function). You can use `message` to write messages to the console.


5.  The function `readline` can be used to ask for data to be typed in. First, figure out how to use `readline` by reading the corresponding help file. Then, construct a function called `getAge` that asks the user to type his/her age. (*Hint:* check the examples in the `readline` help page).



6. **Hard**. Look at the calculations for a confidence interval of the mean in the example in Section \@ref(inference). Write a function that returns the confidence interval for a vector. The function should have two inputs: the vector, and the desired 'alpha'.




7. **hard** Recall the functions `head` and `tail`. Write a function called `middle` that shows a few rows around (approx.) the 'middle' of the dataset. *Hint:* use `nrow`, `print`, and possibly `floor`.






### Working with lists


First read the following list:

```
veclist <- list(x=1:5, y=2:6, z=3:7)
```





1.  Using `sapply`, check that all elements of the list are vectors of the same length. Also calculate the sum of each element. 



2.  Add an element to the list called 'norms' that is a vector of 10 numbers drawn from the standard normal distribution (recall Section \@ref(distributions)).



3.  Using the `pupae` data, use a $t$-test to find if PupalWeight varies with temperature treatment, separate for the two CO$_2$ treatments (so, do two $t$-tests). You **must** use `split` and `lapply`.



4.  For this exercise use the `coweeta` data - a dataset with measurements of tree size. Split the data by `species`, to produce a list called `coweeta_sp`. Keep only those species that have at least 10 observations. (*Hint:* first count the number of observations per species, save that as a vector, find which are at least 10, and use that to subscript the list.) If you don't know how to do this last step, skip it and continue to the next item.



5.  Using the split Coweeta data, perform a linear regression of `log10(biomass)` on `log10(height)`, separately by species. Use `lapply`.




### Functions for histograms


First run this code to produce two vectors.

```
x <- rnorm(100)
y <- x + rnorm(100)
```


1. Run a linear regression y = f(x), save the resulting object. Look at the structure of this object, and note the names of the elements. Extract the residuals and make a histogram.



2. **Hard**. From the previous question, write a function that takes an `lm` object as an argument, and plots a histogram of the residuals.






### Using functions to make many plots



1.  Read the cereals data. Create a subset of data where the `Manufacturer` has at least two observations (use `table` to find out which you want to keep first). Don't forget to drop the empty factor level you may have created!



2.  Make a single PDF with six plots, with a scatter plot between potassium and fiber for each of the six (or seven?) Manufacturers. (*Hint:* ook at the template for producing a PDF with multiple pages at the bottom of Section \@ref(simpleloops).







### Monthly weather plots


1.  For the HFE weather dataset (`hfemet2008`) that makes a scatter plot between PAR (a measure of light intensity) and VPD (a measure of the dryness of air).



1.  Then, split the dataset by month (recall Section \@ref(dfrlists)), and make twelve such scatter plots. Save the result in a single PDF, or on one page with 12 small figures.






<!-- ### The Central limit theorem -->

<!-- The 'central limit theorem' (CLT) forms the backbone of inferential statistics. This theorem states (informally) that if you draw samples (of *n* units) from a population, the mean of these samples follows a normal distribution. This is true regardless of the underlying distribution you sample from.  -->

<!-- In this exercise, you will apply a simple simulation study to test the CLT, and to make histograms and quantile-quantile plots.  -->



<!-- 1. Draw 200 samples of size 10 from a uniform distribution. Use the `runif` function to sample from the uniform distribution, and the `replicate` function to repeat this many times. -->
<!-- ```{r } -->
<!-- unisamples <- replicate(200, runif(10)) -->
<!-- ``` -->

<!-- 1.  Compute the sample mean for each of the 200 samples in~\ref{ite:draw200}. Use `apply` or `colMeans` to calculate column-wise means of a matrix (note: `replicate` will return a matrix, if used correctly). -->
<!-- ```{r } -->
<!-- colMeans(unisamples) -->
<!-- ``` -->

<!-- 1.  Draw a histogram of the 200 sample means, using `hist`. Also draw a normal quantile-quantile plot, using `qqnorm`.  -->
<!-- ```{r } -->
<!-- hist(colMeans(unisamples)) -->
<!-- qqnorm(colMeans(unisamples)) -->
<!-- ``` -->


<!-- 1.  On the histogram, add a normal curve using the `dnorm` function. Note: to do this, plot the histogram with the argument `freq=FALSE`, so that the histogram draws the probability density, not the frequency.  -->
<!-- ```{r } -->
<!-- X <- colMeans(unisamples) -->
<!-- hist(X, freq=FALSE) -->
<!-- curve(dnorm(x, mean=mean(X), sd=sd(X)), add=T) -->
<!-- ``` -->

<!-- \item \hard Write a function that does all of the above, and call it `PlotCLT`. -->
<!-- ```{r } -->
<!-- plotCLT <- function(n1=200, n2=10){ -->
<!--   unisamples <- replicate(n1, runif(n2)) -->
<!--   X <- colMeans(unisamples) -->
<!--   hist(X, freq=FALSE) -->
<!--   curve(dnorm(x, mean=mean(X), sd=sd(X)), add=T) -->
<!-- } -->
<!-- ``` -->







