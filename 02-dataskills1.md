# Data skills - Part 1 {#dataskills1}




## Introduction

Analysing data is much more than applying the right statistics at the right time. A lot of effort and time is spent on reading, filtering, reshaping, bending and twisting your data before you can actually use the data for visualization and analysis.

In this chapter we learn many skills for working with data in R. All of these skills can be seen as mandatory skills before you learn new statistical techniques or fancy new visualizations. We first look at reading datasets into *dataframes*, filtering data when certain conditions are met, and learn in detail about the most important data types that can be stored in dataframes.

We will also look at various ways to summarize data, from simple summaries of what is contained in the original data, to more complex tables of statistics by grouping variables. Finally we will look at merging (joining) dataframes by one or more key-variables, and reshaping datasets (from long to wide, and back). 


**Packages used in this chapter**

The examples will generally let you know which packages are used, but for your convenience, here is a complete list of the packages used. 

For plotting, we often use `ggplot2` and `ggthemes` (these are usually omitted from the examples):

Otherwise:

- `lgrdata` (for the example datasets, see \@ref(exampledata))
- `lubridate` (for dates and times)
- `dplyr` (for various data skills)
- `padr` (for aggregating timeseries data)
- `doBy` (for `summaryBy`, handy for making summary tables)
- `Hmisc` (for `contents` and `describe`, both summarize dataframes)
- `tidyr` (for reshaping dataframes)
- `reshape2` (for reshaping dataframes)
- `data.table` (for `fread`, fast reading of dataframes)
- `readxl` (for reading Excel files)
- `stringr` (for working with text)






## Generating data

### Sequences of numbers {#sequences}

Let's look at a few ways to generate sequences of numbers that we can use in the examples and exercises. There are also a number of real-world situations where you want to use these functions.

First, as we saw already, we can use `c`() to 'concatenate' (link together) a series of numbers. We can also combine existing vectors in this way, for example:


```r
a <- c(1,2,3)
b <- c(4,5,6)
c(a,b)
```

```
## [1] 1 2 3 4 5 6
```

We can generate sequences of numbers using `:`, `seq` and `rep`, like so:


```r
# Sequences of integer numbers using the ":" operator:
1:10   # Numbers 1 through 10
```

```
##  [1]  1  2  3  4  5  6  7  8  9 10
```

```r
5:-5   # From 5 to -5, backwards
```

```
##  [1]  5  4  3  2  1  0 -1 -2 -3 -4 -5
```

```r
# Examples using seq()
seq(from=10, to=100, by=10)
```

```
##  [1]  10  20  30  40  50  60  70  80  90 100
```

```r
seq(from=23, by=2, length=12)
```

```
##  [1] 23 25 27 29 31 33 35 37 39 41 43 45
```

```r
# Replicate numbers:
rep(2, times = 10)
```

```
##  [1] 2 2 2 2 2 2 2 2 2 2
```

```r
rep(c(4,5), each=3)
```

```
## [1] 4 4 4 5 5 5
```

The `rep` function works with any type of vector. For example, character vectors:


```r
# Simple replication
rep("a", times = 3)
```

```
## [1] "a" "a" "a"
```

```r
# Repeat elements of a vector
rep(c("E. tereticornis","E. saligna"), each=3)
```

```
## [1] "E. tereticornis" "E. tereticornis" "E. tereticornis" "E. saligna"     
## [5] "E. saligna"      "E. saligna"
```


### Random numbers {#randomnumbers}

We can draw random numbers using the `runif` function. The `runif` function draws from a uniform distribution, meaning there is an equal probability of any number being chosen. 


```r
# Ten random numbers between 0 and 1
runif(10)
```

```
##  [1] 0.2879627 0.7966966 0.9356509 0.4866498 0.6955631 0.6192641 0.7236583
##  [8] 0.4341522 0.9433122 0.1882243
```

```r
# Five random numbers between 100 and 1000
runif(5, 100, 1000)
```

```
## [1] 375.0443 223.8260 902.0179 687.1368 679.7013
```

\BeginKnitrBlock{rmdtry}<div class="rmdtry">The `runif` function is part of a much larger class of functions, each of which returns
numbers from a different probability distribution. Inspect the help pages of the functions `rnorm`
for the normal distribution, and `rexp` for the exponential distribution. Try generating some data from a normal distribution with a mean of 100, and a standard deviation of 10.</div>\EndKnitrBlock{rmdtry}


Next, we will `sample` numbers from an existing vector.


```r
numbers <- 1:15
sample(numbers, size=20, replace=TRUE)
```

```
##  [1] 10  1  2  5 10  2  6 13 15  4  4 14 12 11 11  5 11  6  6  2
```

This command samples 20 numbers from the `numbers` vector, with replacement.



## Reading data {#readingdata}

There are many ways to read data into R, but we are going to keep things simple and show only a couple of options to read data from text files, and from databases (hosted online). 

Throughout this book we focus on 'rectangular data', that is, data that can be organized in a table with columns and rows. A data table like this, with individual observations in rows, and various data fields in columns, is called a *data frame*.

We first show a few options to read text files into dataframes in R, including comma, tab-delimited, and JSON formats, and how to read data from Excel. Reading data from remote (No)SQL databases is included in Chapter \@ref(webservices).


### Reading CSV files {#readcsv}

A very common text format for data is 'Comma-Separated Values', or CSV. You can use `read.csv` to read these files. Suppose you have the file `Allometry.csv` in your working directory, you can read in the file into a dataframe with, 


```r
allometry <- read.csv("Allometry.csv")
```

Here, `read.csv` assumes that your system uses a point ('.') to separate digits, but in many parts of the world the default is a comma - in which case values are separated by ';'. In this case, use `read.csv2` instead of `read.csv`.

Make sure you fully understand the concept of a working directory (see Section \@ref(fileswd)) before continuing.

If the file is stored elsewhere, you can specify the entire path (this is known as an *absolute* path).


```r
allometry <- read.csv("c:/projects/data/Allometry.csv")
```

It is generally *not recommended* to use absolute paths, because the script will then depend on an exact location of a datafile. But sometimes you want to refer to very large files or databases that are stored in a central location.

If the file is stored in a sub-directory of your working directory, you can specify the *relative* path.


```r
allometry <- read.csv("data/Allometry.csv")
```

The latter option is probably useful to keep your data files separate from your scripts and outputs in your working directory. 

<!--We will not discuss how to organize your files here, but return to this important topic in Chapter \@ref(projectman).-->

The function `read.csv` has many options, let's look at some of them. We can skip a number of rows from being read, and only read a fixed number of rows. For example, use this command to read rows 10-15, skipping the header line (which is in the first line of the file) and the next 9 lines. \emph{Note:} you have to skip 10 rows to read rows 10-15, because the header line (which is ignored) counts as a row in the text file! 


```r
allomsmall <- read.csv("Allometry.csv", skip=10, nrows=5, header=FALSE)
```


### Reading large CSV files

We used the built-in `read.csv` function above, but note that for large files, it is rather slow. The best alternative is to use `fread` from the `data.table` package. The `data.table` package is an excellent resource if you need to do data manipulations on large files (though the syntax takes some getting used to).


```r
library(data.table)
allom <- fread("Allometry.csv")
```

Of course, you could use `fread` even for small files, but I generally recommend to keep dependencies to a minimum. In other words, use built-in ('base') functions when you can, to develop more robust and reproducable code.


### Reading Tab-delimited text files {#tabdelimtext}

Sometimes, data files are provided as text files that are TAB-delimited. To read these files, use the following command:


```r
mydata <- read.table("sometabdelimdata.txt", header=TRUE)
```

In fact, `read.table` is the more general function - `read.csv` is a specific case for comma-delimited files. When using `read.table`, you must specify whether a header (i.e., a row with column names) is present in the dataset (unlike `read.csv`, it is the default to not read the header). If you have a text file with some other delimiter, for example `;`, use the `sep` argument:


```r
mydata <- read.table("somedelimdata.txt", header=TRUE, sep=";")
```


### Including data in a script {#datainscript}

You can also write the dataset in a text file, and read it as in the following examples. This is useful if you have (found) a small dataset that you typed in by hand, or for making reproducible code snippets that include the dataset. 


```r
read.table(header=TRUE, text="
a b
1 2
3 4
")
```

```
##   a b
## 1 1 2
## 2 3 4
```



### JSON

A very popular format for data, especially on the web, is JSON - a text-based format that can represent not just rectangular data (dataframes), but any complex nested data structure.

We can use the `jsonlite` package to read JSON data, and convert it to a dataframe if the data allows it, as in the following example.


```r
# Fake data can be read fro this site.
# Visit the link to see what the original data looks like.
url <- "https://jsonplaceholder.typicode.com/posts/1/comments"

# Package to read JSON; it is very fast.
library(jsonlite)

# fromJSON reads and simplifies the data into a dataframe (if possible)
comment_data <- fromJSON(url)

# We end up with a dataframe.
Hmisc::contents(comment_data)
```

```
## 
## Data frame:comment_data	500 observations and 5 variables    Maximum # NAs:0
## 
## 
##          Storage
## postId   integer
## id       integer
## name   character
## email  character
## body   character
```


### (No)SQL databases

We show examples of how to obtain data from SQL and NoSQL databases in Chapter \@ref(webservices).


### Excel spreadsheets

Excel spreadsheets are *not* a recommended way to store data, but often you don't make that choice yourself. If you do need to read an XLS or XLSX file, the `readxl` package works very well. *Note*: avoid older implementations like the `xlsx` package and `read.xls` in the `gtools` package, which are less reliable.


```r
library(readxl)
mydata <- read_excel("mspreadsheet.xlsx", sheet = 2)
```

Here, `sheet` will specify the sheet by number, alternatively you can refer to the sheet by name (e.g., `sheet = 'rawdata'`).



### Proprietary formats

Many statistical software packages store data in their own format, not just text files. For data from **SPSS**, **SAS** or **Stata**, we recommend the `haven` package for reading the data into dataframes, and the `foreign` package provides further support for Minitab, **Systat**, and **Weka.** 

### Web services

Nowadays many data are available via a 'RESTful' API, which is now by far the most common way to download publicly available (open) data (and many other services as well). We discuss reading data from a REST service, as well as setting up our REST service in Chapter \@ref(webservices).



## Working with dataframes {#dataframes}

As mentioned, this book focuses heavily on dataframes, because this is the object you will use most of the time in data analysis. The following sections provide a brief introduction, but we will see many examples using dataframes throughout this manual. 

Like matrices, dataframes have two dimensions: they contain both columns and rows. Unlike matrices, each column can hold a different type of data. This is very useful for keeping track of different types of information about data points. For example, you might use one column to hold height measurements, and another to hold the matching species IDs. When you read in a file using `read.csv`, the data is automatically stored in a dataframe. Dataframes can also be created from scratch using the function `data.frame` (see Section \@ref(vecstodfr)), but usually we start with data read from a file.



\BeginKnitrBlock{rmdinfo}<div class="rmdinfo">**What is a tibble?**
Sometimes you will notice that some functions return objects that are very similar to
dataframes, but are actually called 'tibbles' (for example, `read_excel` returns one). A tibble is a newer format for dataframes, and is internally very nearly the same. You can pretty much always assume that dataframes and tibbles behave in the same way.</div>\EndKnitrBlock{rmdinfo}



### Convert vectors into a dataframe {#vecstodfr}

Suppose you have two or more vectors (of the same length), and you want to include these in a new dataframe. You can use the function `data.frame`. Here is a simple example:


```r
vec1 <- c(9,10,1,2,45)
vec2 <- 1:5

data.frame(x = vec1, y = vec2)
```

```
##    x y
## 1  9 1
## 2 10 2
## 3  1 3
## 4  2 4
## 5 45 5
```

Here, we made a dataframe with  columns named `x` and `y`. *Note*: take care to ensure that the vectors have the same length, otherwise it won't work!

\BeginKnitrBlock{rmdtry}<div class="rmdtry">Modify the previous example so that the two vectors are *not* the same length. Then, attempt to combine them in a dataframe and inspect the resulting error message.</div>\EndKnitrBlock{rmdtry}


### Variables in the dataframe {#vecdataframes}

We read the `allometry` data (make sure the `lgrdata` package is loaded) to practice basic dataframe skills.


```r
library(lgrdata)
data(allometry)
```

After reading the dataframe, it is good practice to always quickly inspect the dataframe to see if anything went wrong. I routinely look at the first few rows with `head`. Then, to check the types of variables in the dataframe, use the `str` function (short for 'structure'). This function is useful for other objects as well, to view in detail what the object contains.


```r
head(allometry)
```

```
##   species diameter height   leafarea branchmass
## 1    PSME    54.61  27.04 338.485622  410.24638
## 2    PSME    34.80  27.42 122.157864   83.65030
## 3    PSME    24.89  21.23   3.958274    3.51270
## 4    PSME    28.70  24.96  86.350653   73.13027
## 5    PSME    34.80  29.99  63.350906   62.39044
## 6    PSME    37.85  28.07  61.372765   53.86594
```

```r
str(allometry)
```

```
## 'data.frame':	63 obs. of  5 variables:
##  $ species   : Factor w/ 3 levels "PIMO","PIPO",..: 3 3 3 3 3 3 3 3 3 3 ...
##  $ diameter  : num  54.6 34.8 24.9 28.7 34.8 ...
##  $ height    : num  27 27.4 21.2 25 30 ...
##  $ leafarea  : num  338.49 122.16 3.96 86.35 63.35 ...
##  $ branchmass: num  410.25 83.65 3.51 73.13 62.39 ...
```

Individual variables in a dataframe can be extracted using the dollar `$` sign.
Let's print all the tree diameters here, after rounding to one decimal point:


```r
round(allometry$diameter,1)
```

```
##  [1] 54.6 34.8 24.9 28.7 34.8 37.9 22.6 39.4 39.9 26.2 43.7 69.8 44.5 56.6
## [15] 54.6  5.3  6.1  7.4  8.3 13.5 51.3 22.4 69.6 58.4 33.3 44.2 30.5 27.4
## [29] 43.2 38.9 52.6 20.8 24.1 24.9 46.0 35.0 23.9 60.2 12.4  4.8 70.6 11.4
## [43] 11.9 60.2 60.7 70.6 57.7 43.1 18.3 43.4 18.5 12.9 37.9 26.9 38.6  6.5
## [57] 31.8 73.7 28.2 61.5 51.6 18.3  8.4
```

It is also straightforward to add new variables to a dataframe. Let's convert the tree diameter to inches, and add it to the dataframe as a new variable:


```r
allometry$diameterInch <- allometry$diameter / 2.54
```

Instead of using the `$`-notation every time (which can result in lengthy, messy code, especially when your variable names are long) you can use `with` to indicate where the variables are stored. Let's add a new variable called `volindex`, a volume index defined as the square of tree diameter times height:


```r
allometry$volindex <- with(allometry, diameter^2 * height)
```

The `with` function allows for more readable code, while at the same time making sure that the variables `diameter` and `height` are read from the dataframe `allometry`. 

An even better approach is to use `mutate` from the `dplyr` package (similar to base R's `transform`, but with a useful advantage).


```r
library(dplyr)
allometry <- mutate(allometry, 
                diameterInch = diameter / 2.54,
                volindex = diameterInch^2 * height)
```

Where `mutate` adds new variables to the dataframe, and (unlike `transform`) it is able to use variables used just before in the same call (note `diameterInch` was created in the same call).

\BeginKnitrBlock{rmdtry}<div class="rmdtry">The above examples are important - many times throughout this book you will be required to perform similar operations. As an exercise, also add the ratio of height to diameter to the dataframe.</div>\EndKnitrBlock{rmdtry}

A simple summary of the dataframe can be printed with the `summary` function; where we use indexing (`[,1:3]`) to use the first three columns of `allometry` only to save space.


```r
summary(allometry[,1:3])
```

```
##  species      diameter         height     
##  PIMO:19   Min.   : 4.83   Min.   : 3.57  
##  PIPO:22   1st Qu.:21.59   1st Qu.:21.26  
##  PSME:22   Median :34.80   Median :28.40  
##            Mean   :35.56   Mean   :26.01  
##            3rd Qu.:51.44   3rd Qu.:33.93  
##            Max.   :73.66   Max.   :44.99
```

For the numeric variables, the minimum, 1st quantile, median, mean, 3rd quantile, and maximum values are printed. For so-called 'factor' variables (i.e., categorical variables), a simple table is printed (in this case, for the `species` variable). We will come back to factors in Section sec:workingfactors. If the variables have missing values, the number of missing values is printed as well (see Section \@ref(workingmissing)).

<!--If you are using an `rmarkdown` document - see Section \@ref(XXX) on how to make R output pretty when the document is converted to HTML.
-->

To see how many rows and columns your dataframe contains (handy for double-checking you read the data correctly), use `nrow` and `ncol`. Alternatively, `dim` gives the 'dimension' of the dataframe (rows x columns).


```r
nrow(allometry)
```

```
## [1] 63
```

```r
ncol(allometry)
```

```
## [1] 7
```


### Changing column names in dataframes {#namesdataframe}

To access the names of a dataframe as a vector, use the `names` function. You can also use this to change the names. Consider this example:


```r
# read names:
names(allometry)
```

```
## [1] "species"      "diameter"     "height"       "leafarea"    
## [5] "branchmass"   "diameterInch" "volindex"
```

```r
# rename all (make sure vector is same length as number of columns!)
names(allometry) <- c("spec","diam","ht","leafarea","branchm")
```

We can also change some of the names, using simple indexing (see Section \@ref(vectorindexing)).


```r
# rename Second one to 'Diam'
names(allometry)[2] <- "Diam"

# rename 1st and 2nd:
names(allometry)[1:2] <- c("SP","D")
```

Better yet is to use `rename` from the `dplyr` package, which makes sure you change the right column names (indexing as above can be dangerous if the order of columns has changed!).


```r
library(dplyr)

# The names on the right are the original names, on the left are the new ones.
allometry <- rename(allometry, 
                       spec = species,
                       diam = diameter)
```


## Extracting data

### Vectors {#vectorindexing}

Let's look at reordering or taking subsets of a vector, or `indexing` as it is commonly called. This is an important skill to learn, so we will look at several examples.

Let's define two `numeric vectors`:


```r
nums1 <- c(1,4,2,8,11,100,8)
nums2 <- c(3.3,8.1,2.5,9.8,21.2,13.8,0.9)
```

Individual elements of a vector can be extracted using square brackets, `[ ]`. For example, to extract the first and then the fifth element of a vector:


```r
nums1[1]
```

```
## [1] 1
```

```r
nums1[5]
```

```
## [1] 11
```

You can also use another object to do the indexing, as long as it contains a integer number. For example,


```r
# Get last element:
nelements <- length(nums1)
nums1[nelements]
```

```
## [1] 8
```

This last example extracts the last element of a vector. To do this, we first found the length of the vector, and used that to *index* the vector to extract the last element.

We can also select multiple elements, by *indexing* the vector with another vector. Recall how to construct sequences of numbers, explained in Section \@ref(sequences).


```r
# Select the first 3:
nums1[1:3]
```

```
## [1] 1 4 2
```

```r
# Select a few elements of a vector:
selectthese <- c(1,5,2)
nums1[selectthese]
```

```
## [1]  1 11  4
```

```r
# Select every other element:
everyother <- seq(1,7,by=2)
nums1[everyother]
```

```
## [1]  1  2 11  8
```

```r
# Select five random elements:
ranels <- sample(1:length(nums2), 5)
nums2[ranels]
```

```
## [1]  3.3  8.1 21.2  0.9  9.8
```

```r
# Remove the first element:
nums1[-1]
```

```
## [1]   4   2   8  11 100   8
```

```r
# Remove the first and last element:
nums1[-c(1, length(nums1))]
```

```
## [1]   4   2   8  11 100
```

Next, we can look at selecting elements of a vector based on the values in that vector. Suppose we want to make a new vector, based on vector `nums2` but only where the value within certain bounds. We can use simple logical statements to index a vector.


```r
# Subset of nums2, where value is at least 10 :
nums2[nums2 > 10]
```

```
## [1] 21.2 13.8
```

```r
# Subset of nums2, where value is between 5 and 10:
nums2[nums2 > 5 & nums2 < 10]
```

```
## [1] 8.1 9.8
```

```r
# Subset of nums2, where value is smaller than 1, or larger than 20:
nums2[nums2 < 1 | nums2 > 20]
```

```
## [1] 21.2  0.9
```

```r
# Subset of nums1, where value is exactly 8:
nums1[nums1 == 8]
```

```
## [1] 8 8
```

```r
# Subset nums1 where number is NOT equal to 100
nums1[nums1 != 100]
```

```
## [1]  1  4  2  8 11  8
```

```r
# Subset of nums1, where value is one of 1,4 or 11:
nums1[nums1 %in% c(1,4,11)]
```

```
## [1]  1  4 11
```

```r
# Subset of nums1, where value is NOT 1,4 or 11:
nums1[!(nums1 %in% c(1,4,11))]
```

```
## [1]   2   8 100   8
```

These examples showed you several new logical operators (<, >, ==, &). See the help page `?Logic` for more details on logical operators. We will return to logical data in Section sec:workinglogic.


#### Assigning new values to subsets

All of this becomes very useful if we realize that new values can be easily assigned to subsets. This works for any of the examples above. For instance,


```r
# Where nums1 was 100, make it -100
nums1[nums1 == 100] <- -100

# Where nums2 was less than 5, make it zero
nums2[nums2 < 5] <- 0
```

\BeginKnitrBlock{rmdtry}<div class="rmdtry">Using the first set of examples in this section, practice assigning new values to subsets of vectors.</div>\EndKnitrBlock{rmdtry}

### Dataframes {#subsetdataframes}

In base R, there are two ways to take a subset of a dataframe: using the square bracket notation (`[]`) as in the above examples, or using the `filter` function from the `dplyr` package. We will learn both, as they are both useful from time to time. 

Similar to vectors, dataframes can be indexed with row and column numbers using `mydataframe[row,column]`.

Here, `row` refers to the row number (which can be a vector of any length), and `column` to the column number (which can also be a vector). You can also refer to the column by its *name* rather than its number, which can be very useful. All this will become clearer after some examples.

Let's look at a few examples using the Allometry dataset.


```r
# Load data
data(allometry)

# Extract tree diameters: take the 4th observation of the 2nd variable:
allometry[4,2]
```

```
## [1] 28.7
```

```r
# We can also index the dataframe by its variable name:
allometry[4,"diameter"]
```

```
## [1] 28.7
```

```r
# Extract the first 3 rows of 'height':
allometry[1:3, "height"]
```

```
## [1] 27.04 27.42 21.23
```

```r
# Extract the first 5 rows, of ALL variables
# Note the use of the comma followed by nothing
# This means 'every column' and is very useful!
allometry[1:5,]
```

```
##   species diameter height   leafarea branchmass
## 1    PSME    54.61  27.04 338.485622  410.24638
## 2    PSME    34.80  27.42 122.157864   83.65030
## 3    PSME    24.89  21.23   3.958274    3.51270
## 4    PSME    28.70  24.96  86.350653   73.13027
## 5    PSME    34.80  29.99  63.350906   62.39044
```

```r
# Extract the fourth column
# Here we use nothing, followed by a comma,
# to indicate 'every row'
allometry[,4]
```

```
##  [1] 338.485622 122.157864   3.958274  86.350653  63.350906  61.372765
##  [7]  32.077794 147.270523 141.787332  45.020041 145.809802 349.057010
## [13] 176.029213 319.507115 234.368784   4.851567   7.595163  11.502851
## [19]  25.380647  65.698749 160.839174  31.780702 189.733007 253.308265
## [25]  91.538428  90.453658  99.736790  34.464685  68.150309  46.326060
## [31] 160.993131   9.806496  20.743280  21.649603  66.633675  54.267994
## [37]  19.844680 131.727303  22.365837   2.636336 411.160376  15.476022
## [43]  14.493428 169.053644 139.651156 376.308256 417.209436 103.672633
## [49]  33.713580 116.154916  44.934469  18.855867 154.078625  70.602797
## [55] 169.163842   7.650902  93.072006 277.494360 131.856837 121.428976
## [61] 212.443589  82.093031   6.551044
```

```r
# Select only 'height' and 'diameter', store in new dataframe:
allomhd <- allometry[,c("height", "diameter")]
```

As we saw when working with vectors (see Section \@ref(vectorindexing)), we can use expressions to extract data. Because each column in a dataframe is a vector, we can apply the same techniques to dataframes, as in the following examples. 

We can also use one vector in a dataframe to find subsets of another. For example, what if we want to find the value of one vector, if another vector has a particular value?


```r
# Extract diameters larger than 60
allometry$diameter[allometry$diameter > 60]
```

```
## [1] 69.85 69.60 60.20 70.61 60.20 60.71 70.61 73.66 61.47
```

```r
# Extract all rows of allom where diameter is larger than 60.
# Make sure you understand the difference with the above example!
allometry[allometry$diameter > 60,]
```

```
##    species diameter height leafarea branchmass
## 12    PSME    69.85  31.35 349.0570   543.9731
## 23    PIPO    69.60  39.37 189.7330   452.4246
## 38    PIPO    60.20  31.73 131.7273   408.3383
## 41    PIPO    70.61  31.93 411.1604  1182.4222
## 44    PIPO    60.20  35.14 169.0536   658.2397
## 45    PIMO    60.71  39.84 139.6512   139.2559
## 46    PIMO    70.61  40.66 376.3083   541.3062
## 58    PIMO    73.66  44.64 277.4944   275.7165
## 60    PIMO    61.47  44.99 121.4290   199.8634
```

```r
# We can use one vector to index another. For example, find the height of the tree
# that has the largest diameter, we can do:
allometry$height[which.max(allometry$diameter)]
```

```
## [1] 44.64
```

```r
# Recalling the previous section, this is identical to:
allometry[which.max(allometry$diameter), "height"]
```

```
## [1] 44.64
```

```r
# Get 10 random observations of 'leafarea'. Here, we make a new vector 
# on the fly with sample(), which we use to index the dataframe.
allometry[sample(1:nrow(allometry),10),"leafarea"]
```

```
##  [1]  33.713580   6.551044  46.326060 122.157864 141.787332 176.029213
##  [7]  70.602797  22.365837   3.958274  66.633675
```

```r
# As we did with vectors, we can also use %in% to select a subset.
# This example selects only two species in the dataframe.
allometry[allometry$species %in% c("PIMO","PIPO"),]
```

```
##    species diameter    height   leafarea branchmass
## 23    PIPO    69.60 39.369999 189.733007  452.42455
## 24    PIPO    58.42 35.810000 253.308265  595.64015
## 25    PIPO    33.27 20.800001  91.538428  160.44416
## 26    PIPO    44.20 29.110001  90.453658  149.72883
## 27    PIPO    30.48 22.399999  99.736790   44.13532
## 28    PIPO    27.43 27.690001  34.464685   22.98360
## 29    PIPO    43.18 35.580000  68.150309  106.40410
## 30    PIPO    38.86 33.120001  46.326060   58.24071
## 31    PIPO    52.58 41.160003 160.993131  214.34109
## 32    PIPO    20.83 23.340000   9.806496    8.25614
## 33    PIPO    24.13 25.940001  20.743280   22.60111
## 34    PIPO    24.89 25.110000  21.649603   16.77015
## 35    PIPO    45.97 30.389999  66.633675   87.36908
## 36    PIPO    35.05 28.399999  54.267994   51.09006
## 37    PIPO    23.88 23.380001  19.844680   13.98343
## 38    PIPO    60.20 31.729999 131.727303  408.33826
## 39    PIPO    12.45  7.360001  22.365837   16.98648
## 40    PIPO     4.83  3.570000   2.636336    1.77810
## 41    PIPO    70.61 31.929997 411.160376 1182.42222
## 42    PIPO    11.43  6.920000  15.476022    9.25151
## 43    PIPO    11.94  5.849999  14.493428    7.55701
## 44    PIPO    60.20 35.139998 169.053644  658.23971
## 45    PIMO    60.71 39.840003 139.651156  139.25590
## 46    PIMO    70.61 40.659999 376.308256  541.30618
## 47    PIMO    57.66 38.889998 417.209436  310.56688
## 48    PIMO    43.13 36.240000 103.672633  100.40178
## 49    PIMO    18.29 23.130000  33.713580   14.48567
## 50    PIMO    43.43 37.589998 116.154916  108.44781
## 51    PIMO    18.54 21.289999  44.934469   16.54457
## 52    PIMO    12.95 13.440000  18.855867    8.71068
## 53    PIMO    37.85 36.590000 154.078625   72.02907
## 54    PIMO    26.92 29.049999  70.602797   35.87333
## 55    PIMO    38.61 35.519999 169.163842  114.06445
## 56    PIMO     6.48  5.420000   7.650902    3.50621
## 57    PIMO    31.75 34.559999  93.072006   44.72725
## 58    PIMO    73.66 44.640000 277.494360  275.71655
## 59    PIMO    28.19 22.590000 131.856837   91.76231
## 60    PIMO    61.47 44.989998 121.428976  199.86339
## 61    PIMO    51.56 40.229999 212.443589  220.55688
## 62    PIMO    18.29 12.980000  82.093031   28.04785
## 63    PIMO     8.38  4.950000   6.551044    4.36969
```

```r
# Extract tree diameters for the PIMO species, as long as diameter > 50
allometry$diameter[allometry$species == "PIMO" & allometry$diameter > 50]
```

```
## [1] 60.71 70.61 57.66 73.66 61.47 51.56
```

```r
# (not all output shown)
```

\BeginKnitrBlock{rmdtry}<div class="rmdtry">As with vectors, we can quickly assign new values to subsets of data using the `<-` operator. Try this on some of the examples above.</div>\EndKnitrBlock{rmdtry}

### A faster method

While the above method to index dataframes is very flexible and concise, sometimes it leads to code that is difficult to understand. It is also easy to make mistakes when you subset dataframes by the column or row number (imagine the situation where the dataset has changed and you redo the analysis). Consider the `filter` function as a convenient and safe alternative, from the `dplyr` package. (Note that base R provides a nearly identical function, `subset`, but `filter` is much faster). 

With `filter`, you can select rows that meet a certain criterion, and columns as well. This example uses the pupae data. The last example shows the use of `select` from `dplyr` to keep only certain columns, conveniently with the pipe operator.


```r
# Read data
data(pupae)

# For filter(), select().
library(dplyr)

# Take subset of pupae, ambient temperature treatment and CO2 is 280.
# Note: statements separated by commas are interpreted as AND by filter()
filter(pupae, 
       T_treatment == "ambient",
       CO2_treatment == 280,
       Gender == 0)
```

```
##   T_treatment CO2_treatment Gender PupalWeight Frass
## 1     ambient           280      0       0.244 1.900
## 2     ambient           280      0       0.221    NA
## 3     ambient           280      0       0.280 1.996
## 4     ambient           280      0       0.257 1.069
## 5     ambient           280      0       0.275 2.198
## 6     ambient           280      0       0.254 2.220
## 7     ambient           280      0       0.258 1.877
## 8     ambient           280      0       0.224 1.488
```

```r
# (not all output shown)

# Take subset where Frass is larger than 2.9. 
# Also, keep only variables 'PupalWeight' and 'Frass'.
filter(pupae, Frass > 2.6) %>%
  select(PupalWeight, Frass)
```

```
##   PupalWeight Frass
## 1       0.319 2.770
## 2       0.384 2.672
## 3       0.385 2.603
## 4       0.405 3.117
## 5       0.473 2.631
## 6       0.469 2.747
```


### Deleting columns

It is rarely necessary to delete columns from a dataframe, unless you want to save a copy of the dataframe to disk (see Section \@ref(exportingdata)). Instead of deleting columns, you can take a subset and make a new dataframe to continue with. Also, it should not be necessary to delete columns from the dataframe that you have accidentally created in a reproducible script: when things go wrong, simply clear the workspace and run the entire script again.

That aside, you have the following options to delete a column from a dataframe. 


```r
# A simple example dataframe
dfr <- data.frame(a=-5:0, b=10:15)

# Delete the second column (make a new dataframe 'dfr2' that does not include that column)
# This only works if you know the column index (here, 2) for sure,
# it does not work with a column name!
dfr2 <- dfr[,-2]

# Using select() from the dplyr, we can drop columns by name:
dfr2 <- select(dfr, -a)
```



## Exporting data {#exportingdata}

To write a dataframe to a comma-separated values (CSV) file, use the `write.csv` function. For example,


```r
# Some data
dfr <- data.frame(x=1:3, y=2:4)

# Write to disk (row names are generally not wanted in the CSV file).
write.csv(dfr,"somedata.csv", row.names=FALSE)
```

If you want more options, or a different delimiter (such as TAB), look at the `write.table` function. Note that if you write a dataset to a text file, and read it back in with `read.csv` (or `read.table`), the dataset will not be exactly the same. One reason is the number of digits used in the text file, or that you have converted some columns to character, numeric, or whatever. 

If you want to store a dataframe to disk, and later read it back into R exactly as it was before, it is preferred to use a **binary format**. This idea works for any R object, not just dataframes:


```r
# Much faster than writing text files, resulting files are much smaller,
# and objects are saved exactly as they were in R.
saveRDS(dfr, "somefile.rds")

# To read the object back in, do:
dfr <- readRDS("somefile.rds")
```

It is also possible to save all objects that are currently loaded in memory (everything that shows up with `ls()`) with the command `save.image`. However, we strongly urge against it, as it is easy to lose track of which objects are important, and it is too easy to make a mess of things. A fresh installation of Rstudio saves all your objects after you quit Rstudio, but as we mentioned in Section \@ref(installingr), we suggest you switch that behaviour off.



## Special data types

Now that we know how to read in dataframes, it is time we take a closer look at the types of data that can be contained in a dataframe. For the purpose of this book, a dataframe can contain six types of data. These are summarized in the table below:



```
## Warning in print.xtable(xtab, tabular.environment = "longtable"): Attempt
## to use "longtable" with floating = TRUE. Changing to FALSE.
```

% latex table generated in R 3.6.1 by xtable 1.8-4 package
% Wed Oct 30 20:47:46 2019
\begin{longtable}{rllll}
  \hline
 & Data type & Description & Example & Section \\ 
  \hline
1 & numeric & Any number, including `double` (double precision floating point) and `integer` (whole numbers). In R you very rarely have to worry about the exact nature of numeric values. & $\backslash$texttt\{c(1, 12.3491, 10/2, 10\verb|^|6, pi)\} &  \\ 
  2 & character & Strings of text & $\backslash$texttt\{c('apple', 'pear', letters[1:3])\} & $\backslash$@ref(workingtext) \\ 
  3 & factor & Categorial variable. Preferred over character when few unique levels (values) present in the data. Must use in statistical models, plotting. Internally stored as an integer corresponding to the *level* of the factor variable. & $\backslash$texttt\{factor(c('Control','Fertilized','Irrigated'))\} & $\backslash$@ref(workingfactors) \\ 
  4 & logical & Either TRUE or FALSE. Internally stored as 0 (FALSE) or 1 (TRUE). & $\backslash$texttt\{10 == 100/10\} & $\backslash$@ref(workinglogic) \\ 
  5 & Date & Special Date class. Internally stored as number of days since 1970-1-1. & $\backslash$texttt\{as.Date(Sys.time())\} & $\backslash$@ref(readingdates)  \\ 
  6 & POSIXct & Special Date-time class. Internally stored as number of seconds since 1970-1-1, and may have timezone attributes. & $\backslash$texttt\{lubridate::ymd\_hms('1883-08-26 14:00')\} & $\backslash$@ref(datetime) \\ 
   \hline
\hline
\end{longtable}


Also, R has a very useful built-in data type to represent **missing values**. This is represented by `NA` (Not Available) (see Section \@ref(workingmissing)).

We will show how to convert between data types at the end of this chapter (Section \@ref(converttype)).



### Working with factors {#workingfactors}

The *factor* data type is used to represent qualitative, categorical data. 

When reading data from file, for example with `read.csv`, R will automatically convert any variable to a factor if it is unable to convert it to a numeric variable. If a variable is actually numeric, but you want to treat it as a factor, you can use `as.factor` to convert it, as in the following example.


```r
# Make sure you have loaded the lgrdata package
data(pupae)

# This dataset contains a temperature (T_treatment) and CO2 treatment (CO2_treatment).
# Both should logically be factors, however, CO2_treatment is read as numeric:
str(pupae)
```

```
## 'data.frame':	84 obs. of  5 variables:
##  $ T_treatment  : Factor w/ 2 levels "ambient","elevated": 1 1 1 1 1 1 1 1 1 1 ...
##  $ CO2_treatment: int  280 280 280 280 280 280 280 280 280 280 ...
##  $ Gender       : int  0 1 0 0 0 1 0 1 0 1 ...
##  $ PupalWeight  : num  0.244 0.319 0.221 0.28 0.257 0.333 0.275 0.312 0.254 0.356 ...
##  $ Frass        : num  1.9 2.77 NA 2 1.07 ...
```

```r
# To convert it to a factor, we use:
pupae$CO2_treatment <- as.factor(pupae$CO2_treatment)

# Compare with the above,
str(pupae)
```

```
## 'data.frame':	84 obs. of  5 variables:
##  $ T_treatment  : Factor w/ 2 levels "ambient","elevated": 1 1 1 1 1 1 1 1 1 1 ...
##  $ CO2_treatment: Factor w/ 2 levels "280","400": 1 1 1 1 1 1 1 1 1 1 ...
##  $ Gender       : int  0 1 0 0 0 1 0 1 0 1 ...
##  $ PupalWeight  : num  0.244 0.319 0.221 0.28 0.257 0.333 0.275 0.312 0.254 0.356 ...
##  $ Frass        : num  1.9 2.77 NA 2 1.07 ...
```

In the `allometry` example dataset, the `species` variable is a good example of a factor. A factor variable has a number of 'levels', which are the text values that the variable has in the dataset. Factors can also represent treatments of an experimental study. For example,


```r
data(allometry)
levels(allometry$species)
```

```
## [1] "PIMO" "PIPO" "PSME"
```

Shows the three species in this dataset. We can also count the number of rows in the dataframe for each species, like this:


```r
table(allometry$species)
```

```
## 
## PIMO PIPO PSME 
##   19   22   22
```

Note that the three species are always shown in the order of the **levels of the factor**: when the dataframe was read, these levels were assigned based on alphabetical order. Often, this is not a very logical order, and you may want to rearrange the levels to get more meaningful results.

In our example, let's shuffle the levels around, using `factor`.


```r
allometry$species <- factor(allometry$species, levels=c("PSME","PIMO","PIPO"))
```

Now revisit the commands above, and note that the results are the same, but the order of the `levels` of
the `factor` is different. You can also reorder the levels of a factor by the values of another variable, see the example in Section \@ref(reorder).


#### Turn numeric data into factors

We can also generate new factors, and add them to the dataframe. This is a common application:
 


```r
# Add a new variable to allom:  'small' when diameter is less than 10, 'large' otherwise.
allometry$treeSizeClass <- factor(ifelse(allometry$diameter < 10, "small", "large"))

# Now, look how many trees fall in each class.
# Note that somewhat confusingly, 'large' is printed before 'small'.
# Once again, this is because the order of the factor levels is alphabetical by default.
table(allometry$treeSizeClass)
```

```
## 
## large small 
##    56     7
```

What if we want to add a new factor based on a numeric variable with more than two levels? In that case, we cannot use `ifelse`. We must find a different method. Look at this example using `cut`.


```r
# The cut function takes a numeric vectors and cuts it into a categorical variable.
# Continuing the example above, let's make 'small','medium' and 'large' tree size classes:
allometry$treeSizeClass <- cut(allometry$diameter, breaks=c(0,25,50,75), 
                           labels=c("small","medium","large"))

# And the results,
table(allometry$treeSizeClass)
```

```
## 
##  small medium  large 
##     22     24     17
```


#### Empty factor levels

It is important to understand how factors are used in R: they are not simply text variables, or 'character strings'. Each unique value of a factor variable is assigned a *level*, which is used every time you summarize your data by the factor variable.

Crucially, even when you delete data, the original factor *level* is still present. Although this behaviour might seem strange, it makes a lot of sense in many cases (zero observations for a particular factor level can be quite informative!).

Sometimes it is more convenient to drop empty factor levels with the `droplevels` function. Consider this example:


```r
# Note that 'T_treatment' (temperature treatment) is a factor with two levels,
# with 37 and 47 observations in total:
table(pupae$T_treatment)
```

```
## 
##  ambient elevated 
##       37       47
```

```r
# Suppose we decide to keep only the ambient treatment:
pupae_amb <- dplyr::filter(pupae, T_treatment == "ambient")

# Now, the level is still present, although empty:
table(pupae_amb$T_treatment)
```

```
## 
##  ambient elevated 
##       37        0
```

```r
# In this case, we don't want to keep the empty factor level.
# Use droplevels to get rid of any empty levels:
pupae_amb2 <- droplevels(pupae_amb)
```

\BeginKnitrBlock{rmdtry}<div class="rmdtry">Compare the `summary` of `pupae_amb` and `pupae_amb2`, and note the differences.</div>\EndKnitrBlock{rmdtry}


#### Changing the levels of a factor {#changelevels}

Sometimes you may want to change the levels of a factor, for example to replace abbreviations with more readable labels. To do this, we can assign new values with the `levels` function, as in the following example using the pupae data:


```r
# Change the levels of T_treatment by assigning a character vector to the levels.
levels(pupae$T_treatment) <- c("Ambient","Ambient + 3C")

# Or only change the first level, using subscripting.
levels(pupae$T_treatment)[1] <- "Control"
```


\BeginKnitrBlock{rmdtry}<div class="rmdtry">Using the method above, you can also merge levels of a factor, simply by assigning the same new level to both old levels. Try this on a dataset of your choice (for example, in the `allom` data, you can assign new species levels, 'Douglas-fir' for PSME, and 'Pine' for both PIMO and PIPO). Then check the results with `levels()`.</div>\EndKnitrBlock{rmdtry}


### Working with logical data {#workinglogic}

Some data can only take two values: true, or false. For data like these, R has the *logical* data type. 

The following examples use various logical operators, and all return TRUE or FALSE, or a vector of them. The help page `?Syntax` has a comprehensive list of operators in R (including the logical operators).




```r
# Answers to (in)equalities are always logical:
10 > 5
```

```
## [1] TRUE
```

```r
101 == 100 + 1
```

```
## [1] TRUE
```

```r
# ... or use objects for comparison:
apple <- 2
pear <- 3
apple == pear
```

```
## [1] FALSE
```

```r
# NOT equal to.
apple != pear
```

```
## [1] TRUE
```

```r
# Logical comparisons like these also work for vectors, for example:
nums <- c(10,21,5,6,0,1,12)
nums > 5
```

```
## [1]  TRUE  TRUE FALSE  TRUE FALSE FALSE  TRUE
```


The functions `which`, `any` and `all` are very useful to know when working with logical data:



```r
# Find which of the numbers are larger than 5:
which(nums > 5)
```

```
## [1] 1 2 4 7
```

```r
# Other useful functions are 'any' and 'all':
# Are any numbers larger than 25?
any(nums > 25)
```

```
## [1] FALSE
```

```r
# Are all numbers less than or equal to 10?
all(nums <= 10)
```

```
## [1] FALSE
```


You have already been using logical data when we filtered dataframes and vectors:



```r
# Use & for AND, for example to take subsets where two conditions are met:
subset(pupae, PupalWeight > 0.4 & Frass > 3)
```

```
##    T_treatment CO2_treatment Gender PupalWeight Frass
## 25     ambient           400      1       0.405 3.117
```

```r
# Use | for OR
nums[nums < 2 | nums > 20]
```

```
## [1] 21  0  1
```


Logical data are coded by integer numbers (0 = `FALSE`, 1 = `TRUE`), but normally you don't see this, since R will only *print* `TRUE` and `FALSE` 'labels'. However, once you know this, some analyses become even easier. 



```r
# How many numbers are larger than 5?
#- Short solution
sum(nums > 5)
```

```
## [1] 4
```

```r
#- Long solution
length(nums[nums > 5])
```

```
## [1] 4
```

```r
# What fraction of data is larger than some value?
mean(pupae$PupalWeight > 0.3)
```

```
## [1] 0.4880952
```



### Working with missing values {#workingmissing}

In R, missing values are represented  with `NA`, a special data type that indicates the data is simply *Not Available*.

Many functions can handle missing data, usually in different ways. For example, suppose we have the following vector:


```r
myvec1 <- c(11,13,5,6,NA,9)
```

In order to calculate the mean, we might want to either exclude the missing value (and calculate the mean of the remaining five numbers), or we might want `mean(myvec1)` to fail (produce an error). This last case is useful if we don't expect missing values, and want R to only calculate the mean when there are no `NA`'s in the dataset.

These two options are shown in this example:


```r
# Calculate mean: this fails if there are missing values
mean(myvec1)
```

```
## [1] NA
```

```r
# Calculate mean after removing the missing values
mean(myvec1, na.rm=TRUE)
```

```
## [1] 8.8
```

Many functions have an argument `na.rm`, or similar. Refer to the help page of the function to learn about the various options (if any) for dealing with missing values. For example, see the help pages `?lm` and `?sd`.

The function `is.na` returns `TRUE` when a value is missing, which can be useful to see which values are missing, or how many,



```r
# Is a value missing? (TRUE or FALSE)
is.na(myvec1)
```

```
## [1] FALSE FALSE FALSE FALSE  TRUE FALSE
```

```r
# Which of the elements of a vector is missing?
which(is.na(myvec1))
```

```
## [1] 5
```

```r
# How many values in a vector are NA?
sum(is.na(myvec1))  
```

```
## [1] 1
```

#### Making missing values

In many cases it is useful to change some bad data values to `NA`. We can use our indexing skills to do so,


```r
# Some vector that contains bad values coded as -9999
datavec <- c(2,-9999,100,3,-9999,5)

# Assign NA to the values that were -9999
datavec[datavec == -9999] <- NA 
```

In other cases, missing values arise when certain operations did not produce the desired result. Consider this example,



```r
# A character vector, some of these look like numbers:
myvec <- c("101","289","12.3","abc","99")

# Convert the vector to numeric:
as.numeric(myvec)
```

```
## Warning: NAs introduced by coercion
```

```
## [1] 101.0 289.0  12.3    NA  99.0
```

The warning message **NAs introduced by coercion** means that missing values were produced by when we tried to turn one data type (character) to another (numeric).



#### Not A Number

Another type of missing value is the result of calculations that went wrong, for example:


```r
# Attempt to take the logarithm of a negative number:
log(-1)
```

```
## Warning in log(-1): NaNs produced
```

```
## [1] NaN
```

The result is `NaN`, short for *Not A Number*. 

Dividing by zero is not usually meaningful, but R does not produce a missing value:


```r
1000/0
```

```
## [1] Inf
```

It produces 'Infinity' instead. 


#### Missing values in dataframes {#naindataframe}

When working with dataframes, you often want to remove missing values for a particular analysis. We'll use the `pupae` dataset for the following examples. Note we use the `dplyr` package for `filter`.


```r
# (re-)load data
data(pupae)

# Look at a summary to see if there are missing values:
summary(pupae)
```

```
##    T_treatment CO2_treatment       Gender        PupalWeight    
##  ambient :37   Min.   :280.0   Min.   :0.0000   Min.   :0.1720  
##  elevated:47   1st Qu.:280.0   1st Qu.:0.0000   1st Qu.:0.2562  
##                Median :400.0   Median :0.0000   Median :0.2975  
##                Mean   :344.3   Mean   :0.4487   Mean   :0.3110  
##                3rd Qu.:400.0   3rd Qu.:1.0000   3rd Qu.:0.3560  
##                Max.   :400.0   Max.   :1.0000   Max.   :0.4730  
##                                NA's   :6                        
##      Frass      
##  Min.   :0.986  
##  1st Qu.:1.515  
##  Median :1.818  
##  Mean   :1.846  
##  3rd Qu.:2.095  
##  Max.   :3.117  
##  NA's   :1
```

```r
# Notice there are 6 NA's (missing values) for Gender, and 1 for Frass.

# Option 1: take subset of data where Gender is not missing:
pupae_subs1 <- filter(pupae, !is.na(Gender))

# Option 2: take subset of data where Frass AND Gender are not missing
pupae_subs2 <- filter(pupae, 
                      !is.na(Frass),
                      !is.na(Gender))

# A more rigorous subset: remove all rows from a dataset where ANY variable 
# has a missing value:
pupae_nona <- pupae[complete.cases(pupae),]

# This was an example where indexing (using square brackets) is a bit more 
# convenient than the dplyr approach, where we have to write the following.
# A number of similar looking statements do not work!
pupae_nona <- pupae %>% filter(complete.cases(.))
```


#### Subsetting when there are missing values {#subsetmissing}

When there are missing values in a vector, and you take a subset (for example all data larger than some value), should the missing values be included or dropped? There is no one answer to this, but it is important to know that `subset` drops them, but the square bracket method (`[]`) keeps them. 

Consider this example, and especially the use of `which` to drop missing values when subsetting.


```r
# A small dataframe
dfr <- data.frame(a=1:4, b=c(4,NA,6,NA))

# subset drops all missing values
# Note: dplyr::filter also drops NA
subset(dfr, b > 4, select=b)
```

```
##   b
## 3 6
```

```r
# square bracket notation keeps them
dfr[dfr$b > 4,"b"]
```

```
## [1] NA  6 NA
```

```r
# ... but drops them when we use 'which'
dfr[which(dfr$b > 4),"b"]
```

```
## [1] 6
```



### Working with text {#workingtext}

Many datasets include variables that are text only (think of comments, species names, locations, sample codes, and so on), it is useful to learn how to modify, extract, and analyse text-based ('character') variables.

Consider the following simple examples when working with a single character string:
 

```r
# Count number of characters in a bit of text:
sentence <- "Not a very long sentence."
nchar(sentence)
```

```
## [1] 25
```

```r
# Extract the first 3 characters:
substr(sentence, 1, 3)
```

```
## [1] "Not"
```

We can also apply these functions to a vector:


```r
# Substring all elements of a vector
substr(c("good","good riddance","good on ya"),1,4)
```

```
## [1] "good" "good" "good"
```

```r
# Number of characters of all elements of a vector
nchar(c("hey","hi","how","ya","doin"))
```

```
## [1] 3 2 3 2 4
```


#### Combining text {#paste}

To glue bits of text together, use the `paste` function, like so:


```r
# Add a suffix to each text element of a vector:
txt <- c("apple","pear","banana")
paste(txt, "fruit", sep = "-")
```

```
## [1] "apple-fruit"  "pear-fruit"   "banana-fruit"
```

```r
# Glue them all together into a single string using the collapse argument
paste(txt, collapse = "-")
```

```
## [1] "apple-pear-banana"
```

```r
# Combine numbers and text:
paste("Question", 1:3)
```

```
## [1] "Question 1" "Question 2" "Question 3"
```

```r
# This can be of use to make new variables in a dataframe,
# as in this example where we combine two factors to create a new one:
pupae$T_CO2 <- with(pupae, paste(T_treatment, CO2_treatment, sep="-"))
head(pupae$T_CO2)
```

```
## [1] "ambient-280" "ambient-280" "ambient-280" "ambient-280" "ambient-280"
## [6] "ambient-280"
```


\BeginKnitrBlock{rmdtry}<div class="rmdtry">Run the final example above, and inspect the variable `T_CO2` (with `str`) that we added to the dataframe. Make it into a factor variable using `as.factor`, and inspect the variable again.</div>\EndKnitrBlock{rmdtry}


#### Text in dataframes and `grep` {#textgrep}

When you read in a dataset (with `read.csv`, `read.table` or similar), any variable that R cannot convert to numeric *is automatically converted to a factor*. This means that if a column has even just one value that is text (or some garble that does not represent a number), the column cannot be numeric. 

When you want non-numeric columns to be read in as *character* instead of *factor*, you must do:


```r
cereals <- read.csv("cereals.csv", stringsAsFactors = FALSE)
```

Here, the argument `stringsAsFactors=FALSE` avoided the automatic conversion of character variables to factors. 

While we know that factors are very useful, sometimes we want a variable to be treated like text. For example, if we plan to analyse text directly, or extract numbers or other information from bits of text. Let's look at a few examples using the `titanic` dataset.


```r
# Load dataset
data(titanic)

# Is passenger name stored as a character?
is.character(titanic$Name)
```

```
## [1] FALSE
```

```r
# Evidently not. We can convert it to character:
titanic$Name <- as.character(titanic$Name)

# ... or as part of a dplyr chain:
titanic <- titanic %>% mutate(Name = as.character(Name))
```

The following example uses `grep`, a very powerful function. This function can make use of *regular expressions*, a flexible tool for text processing.


```r
# Extract cereal names (for convenience).
data(cereals)
cerealnames <- cereals$Cereal.name

# Find the cereals that have 'Raisin' in them.
# grep() returns the index of values that contain Raisin
grep("Raisin",cerealnames)
```

```
##  [1] 23 45 46 50 52 53 59 60 61 71
```

```r
# grepl() returns TRUE or FALSE
grepl("Raisin",cerealnames)
```

```
##  [1] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
## [12] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
## [23]  TRUE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
## [34] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
## [45]  TRUE  TRUE FALSE FALSE FALSE  TRUE FALSE  TRUE  TRUE FALSE FALSE
## [56] FALSE FALSE FALSE  TRUE  TRUE  TRUE FALSE FALSE FALSE FALSE FALSE
## [67] FALSE FALSE FALSE FALSE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE
```

```r
# That result just gives you the indices of the vector that have 'Raisin' in them.
# these are the corresponding names:
cerealnames[grep("Raisin",cerealnames)]
```

```
##  [1] "Crispy_Wheat_&_Raisins"           
##  [2] "Muesli_Raisins,_Dates,_&_Almonds" 
##  [3] "Muesli_Raisins,_Peaches,_&_Pecans"
##  [4] "Nutri-Grain_Almond-Raisin"        
##  [5] "Oatmeal_Raisin_Crisp"             
##  [6] "Post_Nat._Raisin_Bran"            
##  [7] "Raisin_Bran"                      
##  [8] "Raisin_Nut_Bran"                  
##  [9] "Raisin_Squares"                   
## [10] "Total_Raisin_Bran"
```

```r
# Now find the cereals whose name starts with Raisin.
# The ^ symbol is part of a 'regular expression', it indicates 'starts with':
grep("^Raisin",cerealnames)
```

```
## [1] 59 60 61
```

```r
# Or end with 'Bran'
# The $ symbol is part of a 'regular expression', and indicates 'ends with':
grep("Bran$", cerealnames)
```

```
##  [1]  1  2  3 20 29 53 59 60 65 71
```

As mentioned, `grep` can do a *lot* of different things, so don't be alarmed if you find the help page *a bit overwhelming*. However, there are a few options worth knowing about. One very useful option is to turn off the case-sensitivity, for example:


```r
grep("bran", cerealnames, ignore.case=TRUE)
```

```
##  [1]  1  2  3  4  9 10 20 29 53 59 60 65 71
```
finds `Bran` and `bran` and `BRAN`.

Finally, using the above tools, let's add a new variable to the `cereal` dataset that is `TRUE` when the name of the cereal ends in 'Bran', otherwise it is `FALSE`. For this example, the `grepl` function is more useful (because it returns TRUE and FALSE).


```r
# grepl will return FALSE when Bran is not found, TRUE otherwise
cereals$BranOrNot <- grepl("Bran$", cerealnames)

# Quick summary:
table(cereals$BranOrNot)
```

```
## 
## FALSE  TRUE 
##    67    10
```

\BeginKnitrBlock{rmdreading}<div class="rmdreading">A clear explanation of regular expressions is on the Wikipedia page <http://en.wikipedia.org/wiki/Regular_expressions>. Also worth a look is Robin Lovelace's introduction specifically for R and RStudio at <https://www.r-bloggers.com/regular-expressions-in-r-vs-rstudio/>. </div>\EndKnitrBlock{rmdreading}


#### More control with the `stringr` package

In the previous section we used `grep` and `grepl` to find text in strings, which we can use to subset dataframes, or simply to find data. We can do a lot more with strings, but things get a bit difficult quickly when we stick to base R. Instead, the `stringr` package provides an easy-to-use interface to many common text operations. The following examples give a few very handy tips, but the package provides much more (see `library(help=stringr)`).


```r
library(stringr)

# Extract all numbers from a string
# Note that the result is a character!
str_extract("I have 5 apples", "[0-9]")
```

```
## [1] "5"
```

```r
# Extract the nth word from text.
word("Dantchoff, Mr Khristo", 2)
```

```
## [1] "Mr"
```

```r
# Replace text that matches a pattern.
# (sub or gsub from base R also work OK)
str_replace("Dantchoff, Mr Khristo", "Mr", "Mrs")
```

```
## [1] "Dantchoff, Mrs Khristo"
```


#### Combining text and R objects with `glue`

When controlling output, we often want to combine results stored in R objects, and text. As we saw in Section \@ref(paste), we can use `paste` to achieve this, but there's a better way:


```r
library(glue)

data(titanic)
glue("The titanic dataset has {nrow(titanic)} rows. ",
     "For {percmissing}% of the data, Age is missing.",
     percmissing = round(100 * mean(is.na(titanic$Age)), 1))
```

```
## The titanic dataset has 1313 rows. For 42.4% of the data, Age is missing.
```

With `glue`, we can include both R code directly within `{}` (as we did with `nrow(titanic)`), and also use temporary variables (here: `percmissing`) that are given as extra arguments. Also note that strings separated by `,` are pasted together by `glue`.








### Working with dates and times {#workingwithdates}

Admittedly, working with dates and times in R is somewhat annoying at first. The built-in help files on this subject describe all aspects of this special data type, but do not offer much for the beginning R user. This section covers basic operations that you may need when analysing and formatting datasets.

For working with dates, we use the `lubridate` package, which simplifies it tremendously. 


#### Reading dates {#readingdates}

The built-in `Date` class in R is encoded as an integer number representing the number of days since 1-1-1970 (but this actual origin does not matter for the user). Converting a character string to a date with `as.Date` is straightforward if you use the standard order of dates: `YYYY-MM-DD`. So, for example,


```r
as.Date("2008-5-22")
```

```
## [1] "2008-05-22"
```

The output here is not interesting, R simply prints the date. Because dates are represented as numbers in R, we can do basic arithmetic:



```r
# First load lubridate when working with dates or date-time combinations.
# (as.Date and difftime are part of the base package, but lubridate provides
# years(), months(), and many more functions used below).
library(lubridate)

# A date, 7 days later:
as.Date("2011-5-12") + 7
```

```
## [1] "2011-05-19"
```

```r
# Difference between dates.
as.Date("2009-7-1") - as.Date("2008-12-1")
```

```
## Time difference of 212 days
```

```r
# With difftime, you can specify the units:
difftime(as.Date("2009-7-1"), as.Date("2008-12-1"), units = "weeks")
```

```
## Time difference of 30.28571 weeks
```

```r
# To add other timespans, use functions months(), years() or weeks() to
# avoid problems with leap years
as.Date("2013-8-18") + years(10) + months(1)
```

```
## [1] "2023-09-18"
```

\BeginKnitrBlock{rmdtry}<div class="rmdtry">The previous example showed a very useful trick for adding numbers to a `Date` to get a new `Date` a few days later. Confirm for yourself that this method accounts for leap years. That is, the day before `2011-3-1` should be `2011-2-28` (2011 is not a leap year). But what about `2012-3-1`?</div>\EndKnitrBlock{rmdtry}

Often, text strings representing the date are not in the standard format. Fortunately, it is possible to convert any reasonable sequence to a `Date` object in R. All we have to do is provide a character string to `as.Date` and tell the function the order of the fields.

To convert any format to a Date, we can use the `lubridate` package, which contains the functions `ymd`, `mdy`, and all other combinations of y, m, and d. These functions are pretty smart, as can be seen in these examples:


```r
# Day / month / year
as.Date(dmy("31/12/1991"))
```

```
## [1] "1991-12-31"
```

```r
# Month - day - year (note, only two digits for the year)
as.Date(mdy("4-17-92"))
```

```
## [1] "1992-04-17"
```

```r
# Year month day
as.Date(ymd("1976-5-22"))
```

```
## [1] "1976-05-22"
```

```r
#-- Unusual formatting can be read in with the 'format' argument
# in as.Date. See ?strptime for a list of codes.
# For example, Year and day of year ("%j" stands for 'Julian date')
as.Date("2011 121", format="%Y %j")
```

```
## [1] "2011-05-01"
```

Another method to construct date objects is when you do not have a character string as in the above example, but separate numeric variables for year, month and day. In this case, use the `ISOdate` function:


```r
as.Date(ISOdate(2008,12,2))
```

```
## [1] "2008-12-02"
```

Finally, here is a simple way to find the number of days since you were born, using `today` from the lubridate package.


```r
# Today's date (and time) can be with the today() function
today()
```

```
## [1] "2019-10-30"
```

```r
# We can now simply subtract your birthday from today's date.
today() - as.Date("1976-5-22")
```

```
## Time difference of 15866 days
```


#### Example: using dates in a dataframe {#datesdataframe}

The `as.Date` function that we met in the previous section also works with vectors of dates, and the `Date` class can also be part of a dataframe. Let's take a look at the Hydro data to practice working with dates.


```r
# Load the hydro dataset
data(hydro)

# Now convert this to a Date variable. 
# If you first inspect head(hydro$Date), you will see the format is DD/MM/YYYY
hydro$Date <- as.Date(dmy(hydro$Date))
```

If any of the date conversions go wrong, the `dmy` function (or its equivalents) should print a message letting you know. You can double check if `any` of the converted dates is `NA` like this:


```r
any(is.na(hydro$Date))
```

```
## [1] FALSE
```

We now have successfully read in the date variable. The `min` and `max` functions are useful to check the range of dates in the dataset:


```r
# Minimum and maximum date (that is, oldest and most recent),
min(hydro$Date)
```

```
## [1] "2005-08-08"
```

```r
max(hydro$Date)
```

```
## [1] "2011-08-08"
```

```r
#... and length of measurement period:
max(hydro$Date) - min(hydro$Date)
```

```
## Time difference of 2191 days
```

Finally, the `Date` class is very handy when plotting. Let's make a simple graph of the Hydro dataset. The following code produces Fig. \@ref(fig:hydroplot). Note how the `X` axis is automatically formatted to display the date in a (fairly) pretty way.


```r
ggplot(hydro, aes(x = Date, y = storage)) +
  geom_line()
```

![(\#fig:hydroplot)A simple plot of the hydro data.](02-dataskills1_files/figure-latex/hydroplot-1.pdf) 


#### Date-Time combinations {#datetime}

For dates that include the time, R has a special class called `POSIXct`. The `lubridate` package makes it easy to work with this class. 

Internally, a date-time is represented as the number of seconds since the 1st of January, 1970. Time zones are also supported, but we will not use this functionality in this book (as it can be quite confusing).

From the lubridate package, we can use any combination of (y)ear,(m)onth,(d)ay, (h)our, (m)inutes, (s)econds. For example `ymd_hms` converts a character string in that order.

Let's look at some examples,


```r
# Load lubridate
library(lubridate)

# The standard format is YYYY-MM-DD HH:MM:SS
ymd_hms("2012-9-16 13:05:00")
```

```
## [1] "2012-09-16 13:05:00 UTC"
```

```r
# Read two times (note the first has no seconds, so we can use ymd_hm)
time1 <- ymd_hm("2008-5-21 9:05")
time2 <- ymd_hms("2012-9-16 13:05:00")

# Time difference:
time2 - time1
```

```
## Time difference of 1579.167 days
```

```r
# And an example with a different format, DD/M/YY H:MM
dmy_hm("23-1-89 4:30")
```

```
## [1] "1989-01-23 04:30:00 UTC"
```

```r
# To convert a date-time to a Date, you can also use the as.Date function,
# which will simply drop the time.
as.Date(time1)
```

```
## [1] "2008-05-21"
```

As with `Date` objects, we can calculate timespans using a few handy functions. 

```r
# What time is it 3 hours and 15 minutes from now?
now() + hours(3) + minutes(15)
```

```
## [1] "2019-10-31 00:02:47 CET"
```

\BeginKnitrBlock{rmdtry}<div class="rmdtry">The 2012 Sydney marathon started at 7:20AM on September 16th. The winner completed the race in 2 hours, 11 minutes and 50 seconds. What was the time when the racer crossed the finish line? Using the `weekdays` function, which day was the race held? </div>\EndKnitrBlock{rmdtry}


#### Example:  date-times in a dataframe

Now let's use a real dataset to practice the use of date-times. We also introduce the functions `month`, `yday`, `hour` and `minute` to conveniently extract components of date-time objects.

The last command produces Fig. \@ref(fig:hfemetdate).

```r
# Read the 2008 met dataset from the HFE.
data(hfemet2008)

# Convert 'DateTime' to POSIXct class.
# The order of the original data is MM/DD/YYYY HH:MM
# We also add various date and time fields to the dataframe.
hfemet <- hfemet2008 %>%
  mutate(DateTime = mdy_hm(DateTime),
         Date = as.Date(DateTime),
         DOY = yday(DateTime),
         hour = hour(DateTime),
         minute = minute(DateTime),
         month = month(DateTime))

# Make sure all datetimes were converted OK (if not, NAs would be produced)
any(is.na(hfemet$DateTime))
```

```
## [1] FALSE
```

```r
# FALSE is good here!

# Make a simple line plot for data from one month.
filter(hfemet, month ==  6) %>%
  ggplot(aes(x = DateTime, y =  Tair)) +
  geom_line()
```

![(\#fig:hfemetdate)Air temperature for June at the HFE](02-dataskills1_files/figure-latex/hfemetdate-1.pdf) 

```r
# We can also take a subset of just one day, using the Date variable we added:
hfemet_oneday <- filter(hfemet, Date == as.Date("2008-11-1"))
```


#### Sequences of dates and times {#seqdates}


It is often useful to generate sequences of dates. We can use `seq` as we do for numeric variables (as we already saw in Section \@ref(sequences)).


```r
# A series of dates, by day:
seq(from=as.Date("2011-1-1"), to=as.Date("2011-1-5"), by="day")
```

```
## [1] "2011-01-01" "2011-01-02" "2011-01-03" "2011-01-04" "2011-01-05"
```

```r
# Two-weekly dates:
seq(from=as.Date("2011-1-1"), length=4, by="2 weeks")
```

```
## [1] "2011-01-01" "2011-01-15" "2011-01-29" "2011-02-12"
```

```r
# Monthly:
seq(from=as.Date("2011-12-13"), length=4, by="months")
```

```
## [1] "2011-12-13" "2012-01-13" "2012-02-13" "2012-03-13"
```

Similarly, you can generate sequences of date-times.


```r
# Generate a sequence with 30 min timestep:
# Here, the 'by' field specifies the timestep in seconds.
fromtime <- ymd_hm("2012-6-1 14:30")
seq(from=fromtime, length=3, by=30*60)
```

```
## [1] "2012-06-01 14:30:00 UTC" "2012-06-01 15:00:00 UTC"
## [3] "2012-06-01 15:30:00 UTC"
```


### Converting between data types {#converttype}

It is often useful, or even necessary, to convert from one data type to another. For example, when you read in data with `read.csv` or `read.table`, any column that contains some non-numeric values (that is, values that cannot be converted to a number) will be converted to a factor variable. Sometimes you actually want to convert it to numeric, which will result in some missing values (`NA`) when the value could not be converted to a number.

Another common example is when one of your variables should be read in as a factor variable (for example, a column with treatment codes), but because all the values are numeric, R will simply assume it is a numeric column.

Before we learn how to convert, it is useful to make sure you know what type of data you have to begin with. To find out what type of data a particular vector is, we use `str` (this is also useful for any other object in R).


```r
# Numeric
y <- c(1,100,10)
str(y)
```

```
##  num [1:3] 1 100 10
```

```r
# This example also shows the dimension of the vector ([1:3]).

# Character
x <- "sometext"
str(x)
```

```
##  chr "sometext"
```

```r
# Factor
p <- factor(c("apple","banana"))
str(p)
```

```
##  Factor w/ 2 levels "apple","banana": 1 2
```

```r
# Logical
z <- c(TRUE,FALSE)
str(z)
```

```
##  logi [1:2] TRUE FALSE
```

```r
# Date
sometime <- as.Date("1979-9-16")
str(sometime)
```

```
##  Date[1:1], format: "1979-09-16"
```

```r
# Date-Time
library(lubridate)
onceupon <- ymd_hm("1969-8-18 09:00")
str(onceupon)
```

```
##  POSIXct[1:1], format: "1969-08-18 09:00:00"
```


To test for a particular type of data, use the `is.*something*` functions,
which give `TRUE` if the object is of that type, for example:



```r
# Test for numeric data type:
is.numeric(c(10,189))
```

```
## [1] TRUE
```

```r
# Test for character:
is.character("HIE")
```

```
## [1] TRUE
```

We can convert between types with the `as.*something*` class of functions.


```r
# First we make six example values that we will use to convert
mynum <- 1001
mychar <- c("1001","100 apples")
myfac <- factor(c("280","400","650"))
mylog <- c(TRUE,FALSE,FALSE,TRUE)
mydate <- as.Date("2015-03-18")
mydatetime <- ymd_hm("2011-8-11 16:00")

# A few examples:

# Convert to character
as.character(mynum)
```

```
## [1] "1001"
```

```r
as.character(myfac)
```

```
## [1] "280" "400" "650"
```

```r
# Convert to numeric
# Note that one missing value is created
as.numeric(mychar)
```

```
## Warning: NAs introduced by coercion
```

```
## [1] 1001   NA
```

```r
# Warning!!!
# When converting from a factor to numeric, first convert to character
# !!!
as.numeric(as.character(myfac))
```

```
## [1] 280 400 650
```

```r
# Convert to Date
as.Date(mydatetime)
```

```
## [1] "2011-08-11"
```

```r
# Convert to factor
as.factor(mylog)
```

```
## [1] TRUE  FALSE FALSE TRUE 
## Levels: FALSE TRUE
```

\BeginKnitrBlock{rmdcaution}<div class="rmdcaution">When converting a factor to numeric (i.e. when the factor labels can be converted to numeric values), first convert it to character: `x <- as.numeric(as.character(somefactor))`. The reason for this is that factor variables are internally stored as integer numbers, referring to the *levels* of the factor.</div>\EndKnitrBlock{rmdcaution}


## Exercises



### Working with a single vector - indexing

Recall Exercise \@ref(vecexerc1). Load the `rainfall` data once more.




We now practice subsetting a vector (see Section \@ref(vectorindexing)).

**1.** Take a subset of the rainfall data where rain is larger than 20.


 **2.** What is the mean rainfall for days where the rainfall was at least 4?


 **3.** Subset the vector where it is either exactly zero, or exactly 0.6. 




### Alphabet aerobics 2

The 26 letters of the Roman alphabet are conveniently accessible in R via `letters` and `LETTERS`. These are not functions, but vectors that are always loaded.


**1.** What is the 18th letter of the alphabet?



**2.** What is the last letter of the alphabet (pretend you don't know the alphabet has 26 letters)?




**3.** Use `?sample` to figure out how to sample with replacement. Generate a random subset of fifteen letters. Are any letters in the subset duplicated? *Hint:* use the `any` and `duplicated` functions. Which letters?





### Basic operations with the Cereals data

First read or otherwise understand Section \@ref(vecdataframes).

For this exercise, we will use the `cereals` dataset (from the `lgrdata` package, use `data(cereals)` to load it).

**1.** Read in the dataset, look at the first few rows with `head` and inspect the data types of the variables in the dataframe with `str`.



**2.** Add a new variable to the dataset called `totalcarb`, which is the sum of `carbo` and `sugars`. You can do this is in (at least) three ways! (One of which is `mutate` from `dplyr` - a good idea to start using this powerful function).



**3.** How many cereals in the dataframe are 'hot' cereals? Either make a subset and count the rows, or subset the `Cold.or.Hot` vector directly, and determine its `length` (do not use `table` yet, or a similar function!).
 


**4.** How many unique manufacturers are included in the dataset? *Hint:* use `length` and `unique`.



**5.** Now try the `n_distinct` function from `dplyr`.


**6.** Take a subset of the dataframe of all cereals that have less than 80 calories,
AND have more than 20 units of vitamins.



**7.** Take a subset of the dataframe containing cereals that contain at least 1 unit of sugar, and keep only the variables 'Cereal.name', 'calories' and 'vitamins'. Then inspect the first few rows of the dataframe with `head`.





**8.** For one of the above subsets, write a new CSV file to disk using `write.csv` (see Section \@ref(exportingdata)).




**9.** Rename the column 'Manufacturer' to 'Producer' (see Section  \@ref(namesdataframe)).







### A short dataset


**1.** Read the following data into R (number of cuckoos seen in a week by an avid birdwatcher). Give the resulting dataframe a reasonable name. *Hint:*To read this dataset, look at Section  \@ref(datainscript) for a possibility (there are at least two ways to read this dataset, or you can type it into Excel and save as a CSV file if you prefer).

```
Day nrbirds
sunday 3
monday 2
tuesday 5
wednesday 0
thursday 8
friday 1
saturday 2
```





**2.** Add a day number to the dataset you read in above (sunday=1, saturday=7). Recall the `seq` function (Section  \@ref(sequences)).


**3.** Delete the 'Day' variable (to only keep the `daynumber` that you added above). 




**4.** On which `daynumber` did you observe the most honeyeaters? *Hint:* use `which.max`, in combination with indexing.





**5.** Sort the dataset by number of birds seen. *Hint:* use the `order` function to find the order of the number of birds, then use this vector to index the dataframe.





### Titanic - Part 1


**1.** Read the `titanic` data from the `lgrdata` package.
 



**2.** Make two new dataframes : a subset of male survivors, and a subset of female survivors. Recall Section  \@ref(subsetdataframes).  Use `filter` from `dplyr`.



**3.** Based on the previous question, what was the name of the oldest surviving male? In what class was the youngest surviving female? *Hint:* use `which.max`, `which.min` on the subsets you just created. 

The easiest solution here is to use square bracket notation, but you can also solve this question in steps.




**4.** Take 15 random names of passengers from the Titanic, and sort them alphabetically. *Hint:* use `sort`.




### Titanic - Part 2
 
**1.** Convert the 'Name' (passenger name) variable to a 'character' variable, and store it in the dataframe. See Section  \@ref(textgrep).


**2.** How many observations of 'Age' are missing from the dataframe? See examples in Section  \@ref(workingmissing).


**3.** Make a new variable called 'Status', based on the 'Survived' variable already in the dataset. For passengers that did not survive, Status should be 'dead', for those who did, Status should be 'alive'. Make sure this new variable is a factor. See the example with the `ifelse` function in Section  \@ref(workingfactors).


**4.** Count the number of passengers in each class (1st, 2nd, 3rd). *Hint:* use `table` as shown in Section  \@ref(workingfactors).


**5.** Using `grep`, find the six passengers with the last name 'Fortune'. Make this subset into a new dataframe. Did they all survive? *Hint:* to do this, make sure you recall how to use one vector to index a dataframe (see Section  \@ref(subsetdataframes)). Also, the `all` function might be useful here (see Section  \@ref(workinglogic)).


**6.** As in *2.*, for what proportion of the passengers is the age unknown? Was this proportion higher for 3rd class than 1st and 2nd? *Hint:* First make a subset of the dataframe where age is missing (see Section \@ref(naindataframe)), and then use `table`, as well as `nrow`.





### Hydro dam

Use the `hydro` dam data (used in Section \@ref(datesdataframe)).

**1.** Start by reading in the data. Change the first variable to a `Date` 
class (see Section \@ref(readingdates)). 




**2.** Are the successive measurements in the dataset always exactly one week apart? *Hint:* use `diff`.


**3.** Assume that a dangerously low level of the dam is 235 $Gwh$. How many weeks was the dam level equal to or lower than this value?



**4.** (**Hard question**). For question *2.*, how many times did `storage` decrease below 235 (regardless of how long it remained below 235)? *Hint:* use `diff` and `subset`).





### HFE tree measurements

Use the data for an experiment where trees were irrigated and/or fertilized (the `hfeplotmeans` dataset).

**1.** Read the data, write a copy to a new object called `trees` (easier to type!) and look at various summaries of the dataset. Use `summary`, `str` and `describe` (the latter is in the `Hmisc` package).




**2.** From these summaries, find out how many missing values there are for `height` and `diameter`. Also count the number of missing values as shown in Section  \@ref(workingmissing).


**3.** Inspect the levels of the treatment (`treat`), with the `levels` function. Also count the number of levels with the `nlevels` function. Now assign new levels to the factor, replacing the abbreviations with a more informative label. Follow the example in Section  \@ref(changelevels).



**4.** Using `table`, count the number of observations by `treat`, to check if the dataset is balanced. Be aware that `table` simply counts the number of rows, regardless of missing values. Now take a subset of the dataset where `height` is not missing, and check the number of observations again.




**5.** For which dates do missing values occur in `height` in this dataset? *Hint:* use a combination of `is.na` and `unique`.






### Flux data

In this exercise, you will practice working with Dates and Date-Time combinations, with a timeseries dataset (the `fluxtower` dataset from the `lgrdata` package). 

In this dataset, a new row was produced every 30min - of various meteorological measurements above a forest in Spain. For example, `FCO2` is the flux of carbon dioxide out of forest, so that negative values indicate photosynthesis.

**1.** Read the dataframe. Rename the first column to 'DateTime' (recall Section  \@ref(namesdataframe)).



**2.** Convert DateTime to a `POSIXct` class. Beware of the formatting (recall Section  \@ref(datetime)).




**3.** Did the above action produce any missing values? Were these already missing in the original dataset?



**4.** Add a variable to the dataset called 'Quality'. This variable should be 'bad' when the variable 'ustar' is less than 0.15, and 'good' otherwise. Recall the example in Section \@ref(workingfactors).



**5.** Add a 'month' column to the dataset, as well as 'year'. 



**6.** Look at the 'Rain' column. There are some problems; re-read the data or find another way to display `NA` whenever the data have an invalid value. *Hint:* look at the argument `na.strings` in `read.table`. 

Did it rain on this forest in Spain?





<!-- ### Alphabet Aerobics 3 -->

<!-- In this exercise you will practice a bit more working with text, using the lyrics of the song 'Alphabet Aerobics' by Blackalicious. The lyrics are provided as a text file, which we can most conveniently read into a vector with `readLines`, like this, -->
<!-- \begin{verbatim} -->
<!-- lyric <- readLines("alphabet.txt") -->
<!-- \end{verbatim} -->


<!--  **1.** Read the text file into a character vector like above. Count the number of characters in each line (*Hint* : use `nchar`). -->
<!-- ```{r } -->
<!-- lyric <- readLines("alphabet.txt") -->
<!-- nchar(lyric) -->
<!-- ``` -->

<!-- **1.** Extract the first character of each line (recall examples in Section  \@ref(workingwithtext} on p.~ \@ref(workingwithtext}), and store it in a vector. Now sort this vector alphabetically and compare it to the unsorted vector. Are they the same? (*Hint* : use the `==` operator to compare the two vectors). How many are the same, that is, how many of the first letters are actually in alphabetical order already? -->
<!-- ```{r } -->
<!-- # Extract first character of each line -->
<!-- lyric1 <- substr(lyric,1,1) -->

<!-- # Sort and compare -->
<!-- sort(lyric1) == lyric1 -->

<!-- # And find if they are all the same -->
<!-- all(sort(lyric1) == lyric1) -->

<!-- # Count how many lines were already alphabetical -->
<!-- sum(sort(lyric1) == lyric1) -->
<!-- ``` -->

<!-- **1.** Find the most frequent word used in the lyrics. To do this, first paste all lyrics together into one string, then split the string into words, remove commas, then count the words. You will need to use a new function that we will see again in Section \ref{ \@ref(lapply}. *Hint* : use a combination of `paste`, `strsplit`, `gsub`, `table` and `sort`. -->

<!-- ```{r } -->
<!-- # Paste together all lyrics -->
<!-- lyr_all <- paste(lyric, collapse=" ") -->

<!-- # Remove commas -->
<!-- lyr_all <- gsub(",","",lyr_all) -->

<!-- # Split by space -->
<!-- lyr_words <- strsplit(lyr_all, " ") -->

<!-- # Sort frequencies in descending order and show only first few words -->
<!-- head(sort(table(lyr_words), decreasing=TRUE)) -->
<!-- ``` -->


### DNA Aerobics

DNA sequences can also be represented using text strings. In this exercise, you will make an artificial DNA sequence. 


**1.** Make a random DNA sequence, consisting of a 100 random selections of the letters `C`,`A`,`G`,`T`, and paste the result together into one character string (*Hint* : use `sample` as in Section  \@ref(randomnumbers) with replacement, and use `paste` as shown in Section  \@ref(workingtext). Write it in one line of R code.





















