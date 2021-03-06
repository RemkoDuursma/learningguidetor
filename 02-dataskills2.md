# Data skills - Part 2 {#dataskills2}


 

## Summarizing dataframes

There are a few useful functions to print general summaries of a dataframe, to see which variables are included, what types of data they contain, and so on. We already looked at `summary` and `str` in Section \@ref(dataframes).

Two more very useful functions are from the `Hmisc` package. The first, `describe`, is much like `summary`, but offers slightly more sophisticated statistics. The second, `contents`, is similar to `str`, but does a very nice job of summarizing the `factor` variables in your dataframe, prints the number of missing variables, the number of rows, and so on. 


 

```r
# Load data
data(pupae)

# Make sure CO2_treatment is a factor (it will be read as a number)
pupae$CO2_treatment <- as.factor(pupae$CO2_treatment)

# Show contents:
library(Hmisc)  
contents(pupae)
```

```
## 
## Data frame:pupae	84 observations and 5 variables    Maximum # NAs:6
## 
## 
##               Levels Storage NAs
## T_treatment        2 integer   0
## CO2_treatment      2 integer   0
## Gender               integer   6
## PupalWeight           double   0
## Frass                 double   1
## 
## +-------------+----------------+
## |Variable     |Levels          |
## +-------------+----------------+
## |T_treatment  |ambient,elevated|
## +-------------+----------------+
## |CO2_treatment|280,400         |
## +-------------+----------------+
```



Here, `storage` refers to the internal storage type of the variable: note that the factor variables are stored as 'integer', and other numbers as 'double' (this refers to the precision of the number).


### Making summary tables {#tapplyaggregate}

#### Summarizing vectors with `tapply` {#tapply}

If we have the following dataset called `plantdat`,

<img src="screenshots/exampledata.png" width="33%" />

and execute the command


```r
with(plantdat, tapply(Plantbiomass, Treatment, mean))
```

we get the result

<img src="screenshots/tapplyresult.png" width="33%" />

Note that the result is a `vector` (elements of a vector can have names, like columns of a dataframe).


If we have the following dataset called `plantdat2`,

<img src="screenshots/exampledatalarger.png" width="33%" />

and execute the command


```r
with(plantdat2, tapply(Plantbiomass, list(Species, Treatment), mean))
```

we get the result

<img src="screenshots/tapplyresultlarger.png" width="33%" />

Note that the result here is a `matrix`, where `A` and `B`, the species codes, are the rownames of this matrix.

Often, you want to summarize a variable by the levels of another variable. For example, in the `rain` data, the `Rain` variable gives daily values, but we might want to calculate annual sums,


```r
# Read data
data(rain)

# Annual rain totals.
with(rain, tapply(Rain, Year, FUN=sum))
```

```
##   1996   1997   1998   1999   2000   2001   2002   2003   2004   2005 
##  717.2  640.4  905.4 1021.3  693.5  791.5  645.9  691.8  709.5  678.2
```

The `tapply` function applies a function (`sum`) to a vector (`Rain`), that is split into chunks depending on another variable (`Year`). 

We can also use the `tapply` function on more than one variable at a time. Consider these examples on the `pupae` data. 


```r
# Average pupal weight by CO2 and T treatment:
with(pupae, tapply(PupalWeight, list(CO2_treatment, T_treatment), FUN=mean))
```

```
##       ambient elevated
## 280 0.2900000  0.30492
## 400 0.3419565  0.29900
```

```r
# Further split the averages, by gender of the pupae.
with(pupae, tapply(PupalWeight, list(CO2_treatment, T_treatment, Gender), FUN=mean))
```

```
## , , 0
## 
##      ambient  elevated
## 280 0.251625 0.2700000
## 400 0.304000 0.2687143
## 
## , , 1
## 
##       ambient  elevated
## 280 0.3406000 0.3386364
## 400 0.3568333 0.3692857
```

As the examples show, the `tapply` function produces summary tables by one or more factors. The resulting object is either a vector (when using one factor), or a matrix (as in the examples using the pupae data).

The limitations of `tapply` are that you can only summarize one variable at a time, and that the result is not a dataframe.

The main advantage of `tapply` is that we can use it as input to `barplot`, as the following example demonstrates (Fig. fig:pupgroupedbar})


```r
# Pupal weight by CO2 and Gender. Result is a matrix.
pupm <- with(pupae, tapply(PupalWeight, list(CO2_treatment,Gender), 
                           mean, na.rm=TRUE))

# When barplot is provided a matrix, it makes a grouped barplot.
# We specify xlim to make some room for the legend.
barplot(pupm, beside=TRUE, legend.text=TRUE, xlim=c(0,8),
        xlab="Gender", ylab="Pupal weight")
```

<div class="figure">
<img src="02-dataskills2_files/figure-html/pupgroupedbar-1.svg" alt="A grouped barplot of average pupal weight by CO2 and Gender for the pupae dataset. This is easily achieved via the use of tapply." width="672" />
<p class="caption">(\#fig:pupgroupedbar)A grouped barplot of average pupal weight by CO2 and Gender for the pupae dataset. This is easily achieved via the use of tapply.</p>
</div>
  

#### Quick summary tables with `summaryBy` {#summaryby}
  

If we have the following dataset called `plantdat`,

<img src="screenshots/exampledata.png" width="33%" />

and execute the command


```r
library(doBy)
summaryBy(Plantbiomass ~ treatment, FUN=mean, data=plantdat)
```
  
we get the result

<img src="screenshots/summarybyresult.png" width="33%" />

Note that the result here is a `dataframe`.


If we have the following dataset called `plantdat2`,

<img src="screenshots/exampledatalarger.png" width="33%" />

and execute the command


```r
summaryBy(Plantbiomass ~ Species + Treatment, FUN=mean, data=dfr)
```

we get the result

<img src="screenshots/summarybyresultlarger.png" width="33%" />

Note that the result here is a `dataframe`.

In practice, it is often useful to make summary tables of multiple variables at once, and to end up with a dataframe. In this book we first use `summaryBy`, from the `doBy` package, to achieve this.

With `summaryBy`, we can generate multiple summaries (mean, standard deviation, etc.) on more than one variable in a dataframe at once. We can use a convenient formula interface for this. It is of the form,


```r
summaryBy(Yvar1 + Yvar2 ~ Groupvar1 + Groupvar2, FUN=c(mean,sd), data=mydata)
```

where we summarize the (numeric) variables `Yvar1` and `Yvar2` by all combinations of the (factor) variables `Groupvar1` and `Groupvar2`.




```r
# Load the doBy package
library(doBy)

# read pupae data if you have not already
data(pupae)

# Get mean and standard deviation of Frass by CO2 and T treatments
summaryBy(Frass ~ CO2_treatment + T_treatment,
          data=pupae, FUN=c(mean,sd))
```

```
##   CO2_treatment T_treatment Frass.mean  Frass.sd
## 1           280     ambient         NA        NA
## 2           280    elevated   1.479520 0.2387150
## 3           400     ambient   2.121783 0.4145402
## 4           400    elevated   1.912045 0.3597471
```

```r
# Note that there is a missing value. We can specify na.rm=TRUE,
# which will be passed to both mean() and sd(). It works because those
# functions recognize that argument (i.e. na.rm is NOT an argument of 
# summaryBy itself!)
summaryBy(Frass ~ CO2_treatment + T_treatment,
          data=pupae, FUN=c(mean,sd), na.rm=TRUE)
```

```
##   CO2_treatment T_treatment Frass.mean  Frass.sd
## 1           280     ambient   1.953923 0.4015635
## 2           280    elevated   1.479520 0.2387150
## 3           400     ambient   2.121783 0.4145402
## 4           400    elevated   1.912045 0.3597471
```

```r
# However, if we use a function that does not recognize it, we first have to
# exclude all missing values before making a summary table, like this:
pupae_nona <- pupae[complete.cases(pupae),]

# Get mean and standard deviation for
# the pupae data (Pupal weight and Frass), by CO2 and T treatment.
# Note that length() does not recognize na.rm (see ?length), which is
# why we have excluded any NA from pupae first.
summaryBy(PupalWeight+Frass ~ CO2_treatment + T_treatment,
          data=pupae_nona,
          FUN=c(mean,sd,length))
```

```
##   CO2_treatment T_treatment PupalWeight.mean Frass.mean PupalWeight.sd
## 1           280     ambient        0.2912500   1.957333     0.04895847
## 2           280    elevated        0.3014583   1.473167     0.05921000
## 3           400     ambient        0.3357000   2.103250     0.05886479
## 4           400    elevated        0.3022381   1.931000     0.06602189
##    Frass.sd PupalWeight.length Frass.length
## 1 0.4192227                 12           12
## 2 0.2416805                 24           24
## 3 0.4186310                 20           20
## 4 0.3571969                 21           21
```

You can also use any function that returns a vector of results. In the following example we calculate the 5% and 95% quantiles of all numeric variables in the allometry dataset. To do this, use `.` for the left-hand side of the formula.


```r
# . ~ species means 'all numeric variables by species'.
# Extra arguments to the function used (in this case quantile) can be set here as well,
# they will be passed to that function (see ?quantile).
data(allometry)
summaryBy(. ~ species, data=allometry, FUN=quantile, probs=c(0.05, 0.95))
```

```
##   species diameter.5% diameter.95% height.5% height.95% leafarea.5%
## 1    PIMO      8.1900      70.9150  5.373000     44.675    7.540916
## 2    PIPO     11.4555      69.1300  5.903499     39.192   10.040843
## 3    PSME      6.1635      56.5385  5.276000     32.602    4.988747
##   leafarea.95% branchmass.5% branchmass.95%
## 1     380.3984      4.283342       333.6408
## 2     250.1295      7.591966       655.1097
## 3     337.5367      3.515638       403.7902
```


#### Summarizing dataframes with `dplyr` {#dplyr}

We started with the `summaryBy` package, since it is so easy to use. A more modern and popular approach is to use `dplyr` for all your dataframe summarizing needs. The main advantage is that `dplyr` is very, very fast. For datasets with > hundreds of thousands of rows, you will notice an incredible speed increase. For millions of rows, you really have to use `dplyr` (or `data.table`, but we don't cover that package in this book).

Making summary tables as we did in the above examples requires two steps:

1. Group your dataframe by one or more factor variables
2. Apply summarizing functions to each of the groups

As is the norm, we use the pipe operator (`%>%`) to keep the steps apart. Here is a simple example to get started.


```r
library(dplyr)

group_by(pupae, CO2_treatment, T_treatment) %>%
  summarize(Frass_mean = mean(Frass, na.rm=TRUE),
            Frass_sd = sd(Frass, na.rm=TRUE))
```

```
## `summarise()` has grouped output by 'CO2_treatment'. You can override using the `.groups` argument.
```

```
## # A tibble: 4 x 4
## # Groups:   CO2_treatment [2]
##   CO2_treatment T_treatment Frass_mean Frass_sd
##           <int> <fct>            <dbl>    <dbl>
## 1           280 ambient           1.95    0.402
## 2           280 elevated          1.48    0.239
## 3           400 ambient           2.12    0.415
## 4           400 elevated          1.91    0.360
```


I used `as.data.frame` at the end, so we arrive at an actual dataframe, not a tibble (only for the reason so you can compare the output to the previous examples). The `dplyr` package (and others) always produce tibbles, which are really just dataframes but with some adjusted printing methods.

In `summarize` you can specify each of the new variables that should be produced, in this case giving mean and standard deviation of Frass. If we want to apply a number of functions over many variables, we can use `summarize_at`, like so:


```r
group_by(pupae, CO2_treatment, T_treatment) %>%
  summarize_at(.vars = c("Frass", "PupalWeight"),
                .funs = c("mean", "sd"))
```

```
## # A tibble: 4 x 6
## # Groups:   CO2_treatment [2]
##   CO2_treatment T_treatment Frass_mean PupalWeight_mean Frass_sd PupalWeight_sd
##           <int> <fct>            <dbl>            <dbl>    <dbl>          <dbl>
## 1           280 ambient          NA               0.29    NA             0.0512
## 2           280 elevated          1.48            0.305    0.239         0.0605
## 3           400 ambient           2.12            0.342    0.415         0.0658
## 4           400 elevated          1.91            0.299    0.360         0.0662
```

The result is identical to our last example with `summaryBy`.

Let's look at a more advanced example using weather data collected at the Hawkesbury Forest Experiment in 2008. The data given are in half-hourly time steps. It is a reasonable request to provide data as daily averages (for temperature) and daily sums (for precipitation). 

The following code produces a daily weather dataset, and Fig. \@ref(fig:hfemetaggregate).


```r
# Read data, convert DateTime field to a proper Datetime class using
# lubridate's mdy_hm function, and add a Date column with as.Date.
# Instead of loading packages, we use :: in the example below to make sure
# the functions are used from the right package.
data(hfemet2008)

hfemet_agg <- hfemet2008 %>%
  mutate(DateTime = lubridate::mdy_hm(DateTime),
         Date = as.Date(DateTime)) %>%
  group_by(Date) %>%
  summarize(Rain = sum(Rain),
            Tair = mean(Tair))

# A simple plot of daily rainfall.
library(ggplot2)
ggplot(hfemet_agg, aes(x = Date, y = Rain)) + 
  geom_bar(stat="identity")
```

<div class="figure">
<img src="02-dataskills2_files/figure-html/hfemetaggregate-1.svg" alt="Daily rainfall at the HFE in 2008" width="672" />
<p class="caption">(\#fig:hfemetaggregate)Daily rainfall at the HFE in 2008</p>
</div>


#### Using `padr` to aggregate timeseries data

In the previous example we saw a quick and concise methods to aggregate and filter a dataframe that includes a single date-time column (here roughly referred to as timeseries data). In the example, we calculated daily totals and averages, but what if we want to aggregate by other timespans, like two-week intervals, 6 months, or 15 minutes? The `padr` package is very convenient for this sort of mutation.

The following example uses `padr` in combination with several functions from `dplyr` to make a table of total rainfall in 4 hour increments for one day, using the `hfemet2008` dataset.


```r
library(dplyr)
library(padr)
library(lubridate)

# From the lgrdata package
data(hfemet2008)

mutate(hfemet2008, DateTime = mdy_hm(DateTime)) %>% # convert to proper DateTime
  filter(as.Date(DateTime) == "2008-6-3") %>%       # select a single day
  thicken("4 hours", round="down") %>%              # add datetime in 4hour steps
  group_by(DateTime_4_hour) %>%                     # set grouping variable
  summarize(Rain = sum(Rain)) %>%                   # sum over the grouping variable
  select(DateTime = DateTime_4_hour, Rain)          # show two variables
```

```
## # A tibble: 6 x 2
##   DateTime             Rain
##   <dttm>              <dbl>
## 1 2008-06-03 00:00:00   2.4
## 2 2008-06-03 04:00:00   1.8
## 3 2008-06-03 08:00:00  11.8
## 4 2008-06-03 12:00:00   5.6
## 5 2008-06-03 16:00:00   6.6
## 6 2008-06-03 20:00:00  26.4
```


### Tables of counts {#xtabs}

It is often useful to count the number of observations by one or more multiple factors. One option is to use `tapply` or `summaryBy` in combination with the `length` function. A much better alternative is to use the `xtabs` and `ftable` functions, in addition to the simple use of `table`. Alternatively, `dplyr` provides a `count` function. We will look at both options.

Consider these examples using the Titanic data.


```r
# Read titanic data
data(titanic)

# Count observations by passenger class
table(titanic$PClass)
```

```
## 
## 1st 2nd 3rd 
## 322 280 711
```

```r
# With more grouping variables, it is more convenient to use xtabs.
# Count observations by combinations of passenger class, sex, and whether they survived:
xtabs( ~ PClass + Sex + Survived, data=titanic)
```

```
## , , Survived = 0
## 
##       Sex
## PClass female male
##    1st      9  120
##    2nd     13  148
##    3rd    132  441
## 
## , , Survived = 1
## 
##       Sex
## PClass female male
##    1st    134   59
##    2nd     94   25
##    3rd     80   58
```

```r
# The previous output is hard to read, consider using ftable on the result:
ftable(xtabs( ~ PClass + Sex + Survived, data=titanic))
```

```
##               Survived   0   1
## PClass Sex                    
## 1st    female            9 134
##        male            120  59
## 2nd    female           13  94
##        male            148  25
## 3rd    female          132  80
##        male            441  58
```

```r
# Using dplyr, the result is a dataframe (actually, a tibble)
library(dplyr)
titanic %>% count(PClass, Sex, Survived)
```

```
##    PClass    Sex Survived   n
## 1     1st female        0   9
## 2     1st female        1 134
## 3     1st   male        0 120
## 4     1st   male        1  59
## 5     2nd female        0  13
## 6     2nd female        1  94
## 7     2nd   male        0 148
## 8     2nd   male        1  25
## 9     3rd female        0 132
## 10    3rd female        1  80
## 11    3rd   male        0 441
## 12    3rd   male        1  58
```

### Adding summary variables to dataframes {#summaryvars}

We saw how `tapply` can make simple tables of averages (or totals, or other functions) of some variable by the levels of one or more factor variables. The result of `tapply` is typically a vector with a length equal to the number of levels of the factor you summarized by (see examples in Section \@ref(tapply)). 

Consider the `allometry` dataset, which includes tree height for three species. Suppose you want to add a new variable 'MaxHeight', that is the maximum tree height observed per species. We can use `ave` to achieve this:


```r
# Read data
allom <- allometry %>%
  mutate(MaxHeight = ave(height, species, FUN=max))

# Look at first few rows (or just type allom to see whole dataset)
head(allom)
```

```
##   species diameter height   leafarea branchmass MaxHeight
## 1    PSME    54.61  27.04 338.485622  410.24638      33.3
## 2    PSME    34.80  27.42 122.157864   83.65030      33.3
## 3    PSME    24.89  21.23   3.958274    3.51270      33.3
## 4    PSME    28.70  24.96  86.350653   73.13027      33.3
## 5    PSME    34.80  29.99  63.350906   62.39044      33.3
## 6    PSME    37.85  28.07  61.372765   53.86594      33.3
```

Note that you can use any function in place of `max`, as long as that function can take a vector as an argument, and returns a single number. 

\BeginKnitrBlock{rmdtry}<div class="rmdtry">If you want results similar to `ave`, you can use `summaryBy` with the argument `full.dimension=TRUE`. Try `summaryBy` on the `pupae` dataset with that argument set, and compare the result to `full.dimension=FALSE`, which is the default. </div>\EndKnitrBlock{rmdtry}


### Reordering factor levels based on a summary variable {#reorder}

It is often useful to tabulate your data in a meaningful order. We saw that, when using `summaryBy`, `tapply` or similar functions, that the results are always in the order of your factor levels. Recall that the default order is alphabetical. This is rarely what you want.

You can reorder the factor levels by some summary variable. For example,


```r
# Reorder factor levels for 'Manufacturer' in the cereal data 
# by the mean amount of sodium.

# Read data, show default (alphabetical) levels:
data(cereals)
levels(cereals$Manufacturer)
```

```
## [1] "A" "G" "K" "N" "P" "Q" "R"
```

```r
# Now reorder:
cereals <- mutate(cereals, 
                 Manufacturer = reorder(Manufacturer, sodium, 
                                        median, na.rm=TRUE))

# And inspect the new levels
levels(cereals$Manufacturer)
```

```
## [1] "A" "N" "Q" "P" "K" "G" "R"
```

```r
# Tables are now printed in order:
with(cereals, tapply(sodium, Manufacturer, median))
```

```
##     A     N     Q     P     K     G     R 
##   0.0   7.5  75.0 160.0 170.0 200.0 200.0
```

This trick comes in handy when making barplots; it is customary to plot them in ascending order if there is no specific order to the factor levels, as in this example.

The following code produces Fig. \@ref(fig:coweetabar).



```r
# Here we read the data, add a reordered factor variable,
# and continue with making the summary table used in the plot.
data(coweeta)

coweeta_table <- coweeta %>%
  mutate(species = reorder(species, height, mean, na.rm=TRUE)) %>%
  group_by(species) %>%
  dplyr::summarize(height_mean = mean(height, na.rm=TRUE),
            height_sd = sd(height, na.rm=TRUE))

ggplot(coweeta_table, aes(x = species, y = height_mean)) +
  geom_bar(stat="identity", fill="red") +
  geom_errorbar(aes(ymin = height_mean - height_sd,
                    ymax = height_mean + height_sd), width=0.2) +
  labs(y = "Height (m)", x = "Species")
```

<div class="figure">
<img src="02-dataskills2_files/figure-html/coweetabar-1.svg" alt="An ordered barplot for the coweeta tree data (error bars are 1 SD)." width="672" />
<p class="caption">(\#fig:coweetabar)An ordered barplot for the coweeta tree data (error bars are 1 SD).</p>
</div>

The above example uses the more modern approach with `ggplot2`  and `dplyr`, but we can get practically the same result with `doBy` and `gplots` - below is the code for comparison (output not shown).


```r
library(doBy)
coweeta$species <- with(coweeta, reorder(species, height, mean, na.rm=TRUE))
coweeta_agg <- summaryBy(height ~ species, data=coweeta, FUN=c(mean,sd))

# For barplot2, which adds options for error bars
library(gplots)

# This par setting makes the x-axis labels vertical, so they don't overlap.
par(las=2)
with(coweeta_agg, barplot2(height.mean, names.arg=species,
                           space=0.3, col="red",plot.grid=TRUE,
                           ylab="Height (m)",
                           plot.ci=TRUE,
                           ci.l=height.mean - height.sd,
                           ci.u=height.mean + height.sd))
```



\BeginKnitrBlock{rmdtry}<div class="rmdtry">The above example orders the factor levels by increasing median sodium levels. 
Try reversing the factor levels, using the following code after `reorder`.
`coweeta$species <- factor(coweeta$species, levels=rev(levels(coweeta$species)))`
Here we used `rev` to reverse the levels.</div>\EndKnitrBlock{rmdtry}


## Combining dataframes


### Merging dataframes {#merge}

If we have the following dataset called `plantdat`,

<img src="screenshots/exampledata.png" width="33%" />

and we have another dataset, that includes the same `PlantID` variable (but is not necessarily ordered, nor does it have to include values for every plant):
  
<img src="screenshots/leafnitrogendata.png" width="33%" />

and execute the command


```r
merge(plantdat, leafnitrogendata, by="PlantID")
```

we get the result

<img src="screenshots/mergeresult.png" width="33%" />

Note the missing value (`NA`) for the plant for which no leaf nitrogen data was available.

In many problems, you do not have a single dataset that contains all the measurements you are interested in -- unlike most of the example datasets in this tutorial. Suppose you have two datasets that you would like to combine, or `merge`. This is straightforward in R, but there are some pitfalls.

Let's start with a common situation when you need to combine two datasets that have a different number of rows. 


```r
# Two dataframes
data1 <- data.frame(unit=c("x","x","x","y","z","z"),Time=c(1,2,3,1,1,2))
data2 <- data.frame(unit=c("y","z","x"), height=c(3.4,5.6,1.2))

# Look at the dataframes
data1
```

```
##   unit Time
## 1    x    1
## 2    x    2
## 3    x    3
## 4    y    1
## 5    z    1
## 6    z    2
```

```r
data2
```

```
##   unit height
## 1    y    3.4
## 2    z    5.6
## 3    x    1.2
```

```r
# Merge dataframes:
combdata <- merge(data1, data2, by="unit")

# Combined data
combdata
```

```
##   unit Time height
## 1    x    1    1.2
## 2    x    2    1.2
## 3    x    3    1.2
## 4    y    1    3.4
## 5    z    1    5.6
## 6    z    2    5.6
```

Sometimes, the variable you are merging with has a different name in either dataframe. In that case, you can either rename the variable before merging, or use the following option:


```r
merge(data1, data2, by.x="unit", by.y="item")
```

Where `data1` has a variable called 'unit', and `data2` has a variable called 'item'.

Other times you need to merge two dataframes with multiple key variables. Consider this example, where two dataframes have measurements on the same units at some of the the same times, but on different variables:


```r
# Two dataframes
data1 <- data.frame(unit=c("x","x","x","y","y","y","z","z","z"),
Time=c(1,2,3,1,2,3,1,2,3),
Weight=c(3.1,5.2,6.9,2.2,5.1,7.5,3.5,6.1,8.0))
data2 <- data.frame(unit=c("x","x","y","y","z","z"),
Time=c(1,2,2,3,1,3),
Height=c(12.1,24.4,18.0,30.8,10.4,32.9))

# Look at the dataframes
data1
```

```
##   unit Time Weight
## 1    x    1    3.1
## 2    x    2    5.2
## 3    x    3    6.9
## 4    y    1    2.2
## 5    y    2    5.1
## 6    y    3    7.5
## 7    z    1    3.5
## 8    z    2    6.1
## 9    z    3    8.0
```

```r
data2
```

```
##   unit Time Height
## 1    x    1   12.1
## 2    x    2   24.4
## 3    y    2   18.0
## 4    y    3   30.8
## 5    z    1   10.4
## 6    z    3   32.9
```

```r
# Merge dataframes:
combdata <- merge(data1, data2, by=c("unit","Time"))

# By default, only those times appear in the dataset that have measurements
# for both Weight (data1) and Height (data2)
combdata
```

```
##   unit Time Weight Height
## 1    x    1    3.1   12.1
## 2    x    2    5.2   24.4
## 3    y    2    5.1   18.0
## 4    y    3    7.5   30.8
## 5    z    1    3.5   10.4
## 6    z    3    8.0   32.9
```

```r
# To include all data, use this command. This produces missing values for some times:
merge(data1, data2, by=c("unit","Time"), all=TRUE)
```

```
##   unit Time Weight Height
## 1    x    1    3.1   12.1
## 2    x    2    5.2   24.4
## 3    x    3    6.9     NA
## 4    y    1    2.2     NA
## 5    y    2    5.1   18.0
## 6    y    3    7.5   30.8
## 7    z    1    3.5   10.4
## 8    z    2    6.1     NA
## 9    z    3    8.0   32.9
```

```r
# Compare this result with 'combdata' above!
```


### Using join from `dplyr` {#dplyrjoin}

We showed how to use the `merge` function above, which is provided by base R. For larger datasets, it is advisable to use the `join*` functions from `dplyr`.

Instead of specifying which rows to keep with arguments `all.x`, `all`, etc., `dplyr` provides several functions that should make some intuitive sense. The table below compares `merge` and `join*`.

+------------------------------------+-------------------------------+
| `merge()`                          | `dplyr::join*`                |
+====================================+===============================+
| `merge(dat1, dat2, all = FALSE)`   | `inner_join(dat1, dat2)`      |
+------------------------------------+-------------------------------+
| `merge(dat1, dat2, all.x = TRUE)`  | `left_join(dat1, dat2)`       |
+------------------------------------+-------------------------------+
| `merge(dat1, dat2, all.y = TRUE)`  | `right_join(dat1, dat2)`      |
+------------------------------------+-------------------------------+
| `merge(dat1, dat2, all = TRUE)`    | `full_join(dat1, dat2)`       |
+------------------------------------+-------------------------------+

One other function is provided that has no simple equivalent in base R, `anti_join`, which can be used to find all observations that have *no match* between the two datasets. This can be handy for error-checking.

Consider the cereal dataset, which gives measurements of all sorts of contents of cereals. Suppose the measurements for 'protein', 'vitamins' and 'sugars' were all produced by different laboratories, and each lab sent you a separate dataset. To make things worse, some measurements for sugars and vitamins are missing, because samples were lost in those labs. 


```r
# Read the three datasets given to you from the three different labs:
data(cereal1)
data(cereal2)
data(cereal3)

# As always, look at the first few rows of each dataset.
head(cereal1, 3)
```

```
##      Cereal.name protein
## 1 Frosted_Flakes       1
## 2     Product_19       3
## 3  Count_Chocula       1
```

```r
head(cereal2, 3)
```

```
##     cerealbrand vitamins
## 1    Product_19      100
## 2 Count_Chocula       25
## 3    Wheat_Chex       25
```

```r
head(cereal3, 3)
```

```
##             cerealname sugars
## 1       Frosted_Flakes     11
## 2           Product_19      3
## 3 Mueslix_Crispy_Blend     13
```

```r
# The name of the variable that ties the datasets together, 
# the 'cereal name' differs between the datasets, as do the number of rows.
# We can use merge() three times, but the data are easiest to merge with dplyr:
library(dplyr)

cereal_combined <- full_join(cereal1, cereal2, by=c("Cereal.name" = "cerealbrand")) %>%
                   full_join(cereal3, by=c("Cereal.name" = "cerealname"))

cereal_combined
```

```
##                  Cereal.name protein vitamins sugars
## 1             Frosted_Flakes       1       NA     11
## 2                 Product_19       3      100      3
## 3              Count_Chocula       1       25     NA
## 4                 Wheat_Chex       3       25     NA
## 5                 Honey-comb       1       25     NA
## 6  Shredded_Wheat_spoon_size       3        0     NA
## 7       Mueslix_Crispy_Blend       3       25     13
## 8          Grape_Nuts_Flakes       3       25      5
## 9    Strawberry_Fruit_Wheats       2       NA      5
## 10                  Cheerios       6       25      1
```

```r
# Note that missing values (NA) have been inserted where some data were not available.
```


### Row-binding dataframes {#rbind}

If we have the following dataset called `plantdat`,

<img src="screenshots/rbindinput.png" width="33%" />

and we have another dataset (`plantdatmore`), *with exactly the same columns* (including the names and order of the columns),


<img src="screenshots/moredataforrbind.png" width="33%" />

and execute the command


```r
rbind(plantdat, plantdatmore)
```

we get the result

<img src="screenshots/rbindresult.png" width="33%" />


Using `merge`, we were able to glue dataframes together side-by-side based on one or more 'index' variables. Sometimes you have multiple datasets that can be glued together top-to-bottom, for example when you have multiple very similar dataframes. We can use the `rbind` function, like so:


```r
# Some fake data
mydata1 <- data.frame(var1=1:3, var2=5:7) 
mydata2 <- data.frame(var1=4:6, var2=8:10) 

# The dataframes have the same column names, in the same order:
mydata1
```

```
##   var1 var2
## 1    1    5
## 2    2    6
## 3    3    7
```

```r
mydata2
```

```
##   var1 var2
## 1    4    8
## 2    5    9
## 3    6   10
```

```r
# So we can use rbind to row-bind them together:
rbind(mydata1, mydata2)
```

```
##   var1 var2
## 1    1    5
## 2    2    6
## 3    3    7
## 4    4    8
## 5    5    9
## 6    6   10
```

Let's look at the above `rbind` example again but with a modification where some observations are duplicated between dataframes. This might happen, for example, when working with files containing time-series data and where there is some overlap between the two datasets. The `union` function from the `dplyr` package only returns unique observations:
  


```r
# Some fake data
mydata1 <- data.frame(var1=1:3, var2=5:7) 
mydata2 <- data.frame(var1=2:4, var2=6:8) 

# The dataframes have the same column names, in the same order:
mydata1
```

```
##   var1 var2
## 1    1    5
## 2    2    6
## 3    3    7
```

```r
mydata2
```

```
##   var1 var2
## 1    2    6
## 2    3    7
## 3    4    8
```

```r
# 'rbind' leads to duplicate observations, 'union' removes these:
dplyr::union(mydata1, mydata2)
```

```
##   var1 var2
## 1    1    5
## 2    2    6
## 3    3    7
## 4    4    8
```

```r
rbind(mydata1, mydata2)
```

```
##   var1 var2
## 1    1    5
## 2    2    6
## 3    3    7
## 4    2    6
## 5    3    7
## 6    4    8
```

  
Sometimes, you want to `rbind` dataframes together but the column names do not exactly match. One option is to first process the dataframes so that they do match (using subscripting). Or, just use the `bind_rows` function from `dplyr`. Look at this example where we have two dataframes that have only one column in common, but we want to keep all the columns (and fill with `NA` where necessary),


```r
# Some fake data
mydata1 <- data.frame(index=c("A","B","C"), var1=5:7) 
mydata2 <- data.frame(var1=8:10, species=c("one","two","three")) 

# smartbind the dataframes together
dplyr::bind_rows(mydata1, mydata2)
```

```
##   index var1 species
## 1     A    5    <NA>
## 2     B    6    <NA>
## 3     C    7    <NA>
## 4  <NA>    8     one
## 5  <NA>    9     two
## 6  <NA>   10   three
```

*Note:* an equivalent function to bind dataframes side-by-side is `cbind`, which can be used instead of `merge` when no index variables are present. However, in this book, the use of `cbind` is discouraged for dataframes as it can lead to problems that are difficult to fix, and in all practical applications a merge is preferable.


## Reshaping data {#reshaping}

### From wide to long

In the majority of analyses in R, we like to have our data in 'long' format (nowadays sometimes called the 'tidy' format), where the data for some kind of measurement are in one column, and we have one or more factor variables distinguishing groups like individual, date, treatment, and so on. It is however common encounter data in 'wide format', which can be converted to long format by reshaping appropriately.

The first example uses the dutch election data.


```r
data(dutchelection)
head(dutchelection,3)
```

```
##         Date  VVD PvdA  PVV CDA   SP D66  GL  CU SGP PvdD FiftyPlus
## 1 2012-03-22 22.1 16.8 13.9 9.4 16.8 7.7 4.5 3.3 1.5  2.4       1.1
## 2 2012-04-05 23.6 17.1 13.3 8.8 16.3 8.7 4.1 3.2 1.4  2.0       0.8
## 3 2012-04-19 24.0 17.3 12.0 8.2 17.0 8.8 3.5 3.3 1.6  3.1       0.8
```

The data include percentage votes for 11 Dutch political parties in a 2012 election, and somewhat intuitively the parties have been ordered as columns (so that each row adds up to ca. 100%, ignoring some tiny parties). For purposes of analysis, we would like to reshape this dataset to long format, so that we have just columns 'Date', 'Party', and 'Poll'. Right now 'Party' is in the column names, and 'Poll' represents the polling numbers; the actual data in the cells.

As is often the case, there are many ways to solve this. For most basic reshaping tasks we recommend the `tidyr` package, however some other methods may be needed for more complex tasks, as we find in further examples below. 


```r
library(tidyr)
# The new dataframe will have a column 'Party' with current columns,
# *except* Date (hence the -Date),
# and values will be stored in 'Poll'
elect_long <- gather(dutchelection, Party, Poll, -Date)
head(elect_long,3)
```

```
##         Date Party Poll
## 1 2012-03-22   VVD 22.1
## 2 2012-04-05   VVD 23.6
## 3 2012-04-19   VVD 24.0
```


```r
# Identical results can be had with melt() from reshape2,
# a function which otherwise includes more options.
# See `?melt.data.frame` for more options (not `?melt`, 
# which is the generic function).
library(reshape2)
elect_long <- melt(dutchelection, variable.name="Party", value.name="Poll", id.vars="Date")
```

The advantage of `melt` is that we can include more than one ID variables, like in the following example. This example also shows the common need for some text processing after reshaping.

In this simple dataset, length of feet and hands was measured on two persons on three consecutive days. We want to reshape it to end up with columns 'Day', 'Person', 'Length' and 'BodyPart' (feet or hands). 


```r
dat <- data.frame(Day=rep(1:3, each=2), Person=rep(letters[1:2], 3), 
                  Length.feet = rep(c(2,3), 3),
                  Length.hands=rep(c(3,4),3))
dat
```

```
##   Day Person Length.feet Length.hands
## 1   1      a           2            3
## 2   1      b           3            4
## 3   2      a           2            3
## 4   2      b           3            4
## 5   3      a           2            3
## 6   3      b           3            4
```

```r
# Now reshape to long format, using melt.data.frame
dat_long <- melt(dat, id.vars=c("Day","Person"), value.name="Length", 
	variable.name="BodyPart") %>%
  mutate(BodyPart = gsub("Length.", "", BodyPart))

dat_long
```

```
##    Day Person BodyPart Length
## 1    1      a     feet      2
## 2    1      b     feet      3
## 3    2      a     feet      2
## 4    2      b     feet      3
## 5    3      a     feet      2
## 6    3      b     feet      3
## 7    1      a    hands      3
## 8    1      b    hands      4
## 9    2      a    hands      3
## 10   2      b    hands      4
## 11   3      a    hands      3
## 12   3      b    hands      4
```



### From long to wide

Occasionally we like to use wide format, where groups of data are placed next to each other instead of on top of each other. For the first example consider this simple dataset with a set of questions asked to two persons,


```r
survey <- read.table(text="user	question	answer
a	question_1	hi
a	question_2	hey
a	question_3	oh
a	question_4	no
a	question_5	yes
a	question_6	obv
b	question_1	cool
b	question_2	good
b	question_3	yes
b	question_4	sweet
b	question_5	wow
b	question_6	no", header=TRUE)

survey
```

```
##    user   question answer
## 1     a question_1     hi
## 2     a question_2    hey
## 3     a question_3     oh
## 4     a question_4     no
## 5     a question_5    yes
## 6     a question_6    obv
## 7     b question_1   cool
## 8     b question_2   good
## 9     b question_3    yes
## 10    b question_4  sweet
## 11    b question_5    wow
## 12    b question_6     no
```

In this case we actually want to have all data for each 'user' in a single row, thus creating columns 'question_1', 'question_2' and so on. The first solution uses `spread` from `tidyr`.


```r
library(tidyr)
spread(survey, question, answer)
```

```
##   user question_1 question_2 question_3 question_4 question_5 question_6
## 1    a         hi        hey         oh         no        yes        obv
## 2    b       cool       good        yes      sweet        wow         no
```

The second solution uses `dcast` from `reshape2`. I show this solution because unlike `spread`, `dcast` can be used for more complicated reshaping problems.


```r
library(reshape2)
dcast(survey, user ~ question, value.var="answer")
```

```
##   user question_1 question_2 question_3 question_4 question_5 question_6
## 1    a         hi        hey         oh         no        yes        obv
## 2    b       cool       good        yes      sweet        wow         no
```

Here, the formula indicates 'user goes in rows, question in columns', and `value.var` is the name of the variable used to populate the cells.

For the second, more complex, example we use a small version of the `eucface_gasexchange` dataset, which includes measurements of photosynthesis (`Photo`) on two experimental plots (`Plot`) on three Dates (`Date`), under two experimental treatments (`treatment`). Clearly we have multiple levels of nesting of our data, and each of these nesting levels is represented by a different variable. 


```r
gas <- read.table(text="Date	Plot	treatment	Photo
A	1	Amb	14.4
A	1	Ele	16.3
A	2	Amb	17.8
A	2	Ele	13.3
B	1	Amb	16.4
B	1	Ele	19
B	2	Amb	15
B	2	Ele	10.8
C	1	Amb	17.5
C	1	Ele	19.8
C	2	Amb	13.5
C	2	Ele	12.1
", header=TRUE)
```

Now we have two variables that describe which measurements 'belong together' (experimental plot, and treatment). We can no longer use `spread` (which uses just one of those variables), but `dcast` does work nicely with multiple groupings:


```r
dcast(gas, Date + Plot ~ treatment, value.var="Photo")
```

```
##   Date Plot  Amb  Ele
## 1    A    1 14.4 16.3
## 2    A    2 17.8 13.3
## 3    B    1 16.4 19.0
## 4    B    2 15.0 10.8
## 5    C    1 17.5 19.8
## 6    C    2 13.5 12.1
```

We have another possibility here, to instead place the values for the individual plots and treatments next to each other. Note we now move `Plot` to the right-hand side of the formula in `dcast`.


```r
dcast(gas, Date ~ Plot + treatment, value.var="Photo")
```

```
##   Date 1_Amb 1_Ele 2_Amb 2_Ele
## 1    A  14.4  16.3  17.8  13.3
## 2    B  16.4  19.0  15.0  10.8
## 3    C  17.5  19.8  13.5  12.1
```



## More complex `dplyr` examples

In this chapter, we have learned many new tools to work with data - filtering, summarizing, reshaping, and so on. One advantage of the `dplyr` package over functions in base R is that the various operations can be efficiently combined with the `%>%` operator, combining all your data converting and shaping steps into one. 

This will become clear with some examples. 


### Tree growth data: filter and multiple groupings

The first example uses the `hfeifbytree` data from the `lgrdata` package. This dataset contains measurements of height and stem diameter on nearly 1200 Eucalyptus trees in Sydney, repeated 17 times. The trees grow in plots subjected to different treatments (control, fertilization, irrigation, or both).

On some of the dates, all trees were measured, while on other dates a small subsample was taken (ca. 10% of all trees). We want to make a plot of average tree height by treatment over time, but use only the dates were all trees were measured. The following code makes Fig. \@ref(fig:hfeifmeanplot).


```r
data(hfeifbytree)

library(dplyr)
library(ggplot2)   
library(ggthemes)

hfeif_meanh <- 
  mutate(hfeifbytree, Date = as.Date(Date)) %>%  # Convert to proper Date
  group_by(Date) %>%              # Set up the grouping variable
  filter(n() > 500) %>%           # Keep groups with more than 500 observations
  group_by(Date, treat) %>%       # New grouping; to get average by Date and treatment
  summarize(height = mean(height, na.rm=TRUE)) %>% # Average height, discard NA.
  rename(treatment = treat)       # rename a variable
```

```
## `summarise()` has grouped output by 'Date'. You can override using the `.groups` argument.
```

```r
# It is possible to make the plot inside the data pipeline, but
# we recommend separating them!
ggplot(hfeif_meanh, aes(x = Date, y = height, col = treatment)) +
  geom_point(size = 2) + 
  geom_line() +
  scale_colour_tableau() +
  ylim(c(0,20))
```

<div class="figure">
<img src="02-dataskills2_files/figure-html/hfeifmeanplot-1.svg" alt="Average tree height by treatment over time, for the hfeifbytree data" width="672" />
<p class="caption">(\#fig:hfeifmeanplot)Average tree height by treatment over time, for the hfeifbytree data</p>
</div>


### Crude oil production: find top exporters

In the second example we will use the `oil` data, a dataset with annual crude oil production for the top 8 oil-producing countries since 1971. We want to find the top three countries for the period 1980-1985, sort them, and replace the country abbreviations with country names.


```r
data(oil)

# First we make a dataframe with the abbreviations.
# Very handy here is tribble from tibble; making it easier to write
# small dataframes directly into your script.
library(tibble)
abb_key <- tribble(~country, ~country_full,
                      "MEX",    "Mexico",
                      "USA",    "USA",
                      "CHN",    "China",
                      "IRN",    "Iran",
                      "SAU",    "Saudi-Arabia",
                      "IRQ",    "Iraq",
                      "KWT",    "Kuwait",
                      "VEN",    "Venezuela")

# To avoid a warning, we first convert country to character
# (not strictly necessary)
oil_top <-
  mutate(oil, 
         country = as.character(country),
         production_M = production / 1000) %>%  # Convert to million tonnes.
  full_join(abb_key, by="country") %>%          # Add full country name
  filter(year %in% 1980:1985) %>%               # Subset for years between 1980-1985
  group_by(country_full) %>%
  summarize(production_M = mean(production_M)) %>%
  arrange(desc(production_M)) %>%               # Sort by production; in descending order
  as.data.frame %>%                             # To be consistent, output a dataframe
  head(., 3)                                    # Only show the top 32

# Finally make a table that looks nice in an rmarkdown document
library(pander)
oil_top %>% pander(., 
                   caption = "Top three oil producing countries, 1980-1985.",
                   col.names = c("Country", "Annual production (MTOE)"))
```


-----------------------------------------
   Country      Annual production (MTOE) 
-------------- --------------------------
     USA                 443.5           

 Saudi-Arabia            335.1           

    Mexico               138.8           
-----------------------------------------

Table: Top three oil producing countries, 1980-1985.


\BeginKnitrBlock{rmdtry}<div class="rmdtry">Run the example above, and modify the code so that we find the *maximum* oil production for each country, and the table should show the 2 countries with the lowest maximum.</div>\EndKnitrBlock{rmdtry}



### Weight loss data: make an irregular timeseries regular {#weightlossdplyr}

In this example we will use the `weightloss` data, again from the `lgrdata` package. A person recorded his weight on irregular intervals (1-3 days), to check if his diet was having the desired effect. Because the dataset is irregular, it is not easy to calculate daily weight loss for the entire dataset. We can use a combination of `dplyr`, `zoo` and `padr` to clean up this dataset.

The `pad` function is quite powerful: it takes a dataframe, figures the time-indexing variable (in our case, Date), and stretches the dataset to include all Dates along the dataset, filling NA where no data were available.

The `zoo` package contains many useful functions for timeseries data. Here we use `na.approx` to linearly interpolate missing values. We also make a plot of the weight loss data, clarifying which data were interpolated. The following code make Fig. \@ref(fig:weightlossinterp).


```r
library(lubridate)
library(dplyr)
library(zoo)
library(padr)

data(weightloss)

weightloss2 <- mutate(weightloss, 
                      Date = dmy(Date),                # proper Date class
                      Weight = Weight * 0.4536) %>%    # convert to kg
               pad(interval ="day") %>%                # timeseries is now regular
               mutate(measured = !is.na(Weight),  # FALSE when the value was interpolated
                      Weight = na.approx(Weight))      # linear interpolation (zoo)
  
ggplot(weightloss2, aes(x = Date, y = Weight)) +
  geom_point(colour = ifelse(weightloss2$measured, "darkgrey", "red2"))
```

<div class="figure">
<img src="02-dataskills2_files/figure-html/weightlossinterp-1.svg" alt="The weightloss data, with linearly interpolated missing values (red)." width="672" />
<p class="caption">(\#fig:weightlossinterp)The weightloss data, with linearly interpolated missing values (red).</p>
</div>


## Exercises





### Summarizing the cereal data

1. Read the cereal data, and produce quick summaries using `str`, `summary`,  `contents` and `describe` (recall that the last two are in the `Hmisc` package). Interpret the results.





2. Find the average sodium, fiber and carbohydrate contents by `Manufacturer`. Use either `summaryBy` or `dplyr`.



3. Add a new variable 'SodiumClass', which is 'high' when sodium > 150 and 'low' otherwise. Make sure the new variable is a factor. Look at the examples in Section  \@ref(workingfactors) to recall how to do this. Now, find the average, minimum and maximum sugar content for 'low' and 'high' sodium. *Hint:* make sure to use `na.rm=TRUE`, because the dataset contains missing values.



4. Find the maximum sugar content by Manufacturer and sodiumClass, using `tapply`. Inspect the result and notice there are missing values. Try to use `na.rm=TRUE` as an additional argument to `tapply`, only to find out that the values are still missing. Finally, use `xtabs` (see Section  \@ref(xtabs), p.  \@ref(xtabs)) to count the number of observations by the two factors to find out if we have missing values in the `tapply` result.



5. Repeat the previous question with `summaryBy` or `dplyr`. Compare the results.


```
## `summarise()` has grouped output by 'sodiumClass'. You can override using the `.groups` argument.
```

6. Count the number of observations by Manufacturer and whether the cereal is 'hot' or 'cold', using `xtabs` (see Section  \@ref(xtabs)).




}


### Words and the weather

1. Using the 'Age and memory' dataset (`memory` from the `lgrdata` package), find the mean and maximum number of words recalled by 'Older' and 'Younger' age classes.



2. The `hfemet2008` dataset contains meteorological measurements at a station near Sydney, Australia. Find the mean air temperature by month. To do this, first add the month variable as shown in Section  \@ref(datetime).




### Merging data


1. Load the `pupae` dataset. The data contain measurements of larva ('pupae') weight and 'frass' (excrement) production while allowed to feed on leaves, grown under different concentrations of carbon dioxide (CO2). Also read this short dataset, which gives a label 'roomnumber' for each CO$_2$ treatment.

\begin{verbatim}
|CO2_treatment |Roomnumber  |
|-------------:|-----------:|
|280           |1           |
|400           |2           |
\end{verbatim}

To read this dataset, consider the `data.frame` function described in Section  \@ref(vecstodfr).

1. Merge the short dataset onto the pupae data. Check the result.





### Merging multiple datasets

Read Section  \@ref(dplyrjoin), and learn how to merge more than two datasets together.

First, run the following code to construct three dataframes that we will attempt to merge together.

```
dataset1 <- data.frame(unit=letters[1:9], treatment=rep(LETTERS[1:3],each=3),
                       Damage=runif(9,50,100))
unitweight <- data.frame(unit=letters[c(1,2,4,6,8,9)], Weight = rnorm(6,100,0.3))
treatlocation <- data.frame(treatment=LETTERS[1:3], Glasshouse=c("G1","G2","G3"))
```




1. Merge the three datasets together, to end up with one dataframe that has the columns 'unit', 'treatment', 'Glasshouse', 'Damage' and 'Weight'. Some units do not have measurements of `Weight`. Merge the datasets in two ways to either include or exclude the units without `Weight` measurements. To do this, either use `merge` or `dplyr::join` twice - you cannot merge more than two datasets in one step.



}


### Ordered boxplot

1. First read the `cereals` data and learn how to make a boxplot, using R base graphics:



Notice the *ordering* of the boxes from left to right, and compare it to `levels` of the factor variable `Manufacturer`.


2. Now, redraw the plot with Manufacturer in order of increasing mean sodium content (use `reorder`, see Section  \@ref(reorder)).



3. Inspect the help page (`?boxplot`), and change the boxplots so that the width varies with the number of observations per manufacturer (*Hint:* find the `varwidth` argument).




<!-- ### Variances in the I x F {#ifvariance} -->

<!-- Here, we use the tree inventory data from the irrigation by fertilization (I x F) experiment in the Hawkesbury Forest Experiment (HFE) (see Section  \@ref(ifdata}, p.  \@ref(ifdata}). -->

<!-- } -->
<!-- 1. Use only data from 2012 for this exercise. You can use the file \file{HFEIFplotmeans2012.csv} if you want to skip this step. -->
<!-- ```{r } -->
<!-- # Read and prepare data. To use year(), the lubridate package must be loaded. -->
<!-- hfeif <- read.csv('HFEIFplotmeans.csv', stringsAsFactors=FALSE) -->
<!-- hfeif$Date <- as.Date(mdy(hfeif$Date)) -->
<!-- hfeif$Year <- year(hfeif$Date) -->
<!-- hfeif2012 <- subset(hfeif, Year == 2012) -->
<!-- ``` -->

<!-- \item{#ite:variances} \intermed There are four treatments in the dataframe. Calculate the variance of diameter for each of the treatments (this should give four values). These are the *within-treatment* variances. Also calculate the variance of tree diameter across all plots (this is one number). This is the *plot-to-plot variance*. -->

<!-- ```{r } -->
<!-- # Variances across plots within treatments  -->
<!-- withinvar <- with(hfeif2012, tapply(diameter,treat,FUN=var)) -->

<!-- # Variance of diameter across all plots -->
<!-- plotvar <- var(hfeif2012$diameter) -->
<!-- ``` -->


<!-- 1. In \ref{ite:variances}, also calculate the mean within-treatment variance. Compare the value to the plot-to-plot variance. What can you tentatively conclude about the treatment effect? -->
<!-- ```{r } -->
<!-- # Average within-treatment variance -->
<!-- mean(withinvar) -->

<!-- # Variance across all plots -->
<!-- plotvar -->

<!-- # Variance within treatments seems much smaller than -->
<!-- # the variance across all plots. -->
<!-- # This indicates that there is likely a treatment effect. -->
<!-- ``` -->


<!-- } -->






