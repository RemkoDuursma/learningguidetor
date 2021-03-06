# Basic statistics and linear modelling {#linmodel}


## Introduction

This book is not an *Introduction to statistics*. There are many books focused on *statistics*, with or without example R code. The focus throughout this book is on the R code itself, as we try to present clear and short solutions to common analysis tasks. In the following, we assume you have a basic understanding of linear regression, Student's $t$-tests, ANOVA, and confidence intervals for the mean.

Linear modelling is a general technique that includes linear regression, but also basic means testing, comparing samples, and more complex combinations of class and numeric predictors. 

In linear modelling we are interested in understanding variation in a single *numeric response variable* (or sometimes binary). We can include all sorts of *predictors* ('explanatory variables', 'independent variables'), depending on what we chose we end up with various techniques:

- factor variable(s), test difference in mean in our predictor across the factor ("Analysis of variance", "t-test")
- a single numeric predictor (simple linear regression)
- multiple numeric predictors, and interactions (multiple linear regression)
- mixture of factors and numeric predictors (in the simplest case, analysis of covariance - ANCOVA)

There are many more complicated possibilities, but the above is what this chapter is all about.


**Packages used in this chapter**

The examples will generally let you know which packages are used, but for your convenience, here is a complete list of the packages used. 

For plotting, we often use `ggplot2`, `ggthemes`, `scales` and `ggpubr` (these are frequently omitted from the examples):

Otherwise:

- `lgrdata` (for the example datasets)
- `pastecs` (for descriptive statistics)
- `dplyr` (for various data skills and the pipe operator `%>%`)
- `pander` (to format R output for use in rmarkdown documents)
- `broom` (for tidy printing of fitted models)
- `multcomp` (for multiple comparisons)
- `visreg` (for visualizing regression models)
- `car` (for various functions used in regression)
- `emmeans` (for computing marginal effects from regression models)






## Probability distributions {#distributions}

You will have encountered a number of probability distributions before. For example, the *Binomial* distribution is a model for the distribution of the number of *successes* in a sequence of independent trials, for example, the number of heads in a coin tossing experiment. Another commonly used discrete distribution is the *Poisson*, which is a useful model for many kinds of count data. Of course, the most important distribution of all is the *Normal* or Gaussian distribution.

R provides sets of functions to find densities, cumulative probabilities, quantiles, and to draw random numbers from many important distributions. The names of the functions all consist of a one letter prefix that specifies the type of function and a stem which specifies the distribution. Look at the examples in the table below.


\begin{tabular}{l|l}
\hline
Prefix & Meaning\\
\hline
`d` & density\\
\hline
`p` & cumulative probability\\
\hline
`q` & quantile\\
\hline
`r` & simulate\\
\hline
\end{tabular}


\begin{tabular}{l|l}
\hline
Suffix & Meaning\\
\hline
`binom` & Binomial\\
\hline
`pois` & Poisson\\
\hline
`norm` & Normal\\
\hline
`t` & Student's t\\
\hline
`chisq` & Chi-squared\\
\hline
`f` & F\\
\hline
\end{tabular}

Using the prefix and the suffix, we can construct each desired function. For example, 


```r
# Calculate the probability of 3 heads out of 10 tosses of a fair coin.
# This is a (d)ensity of a (binom)ial distribution.
dbinom(3, 10, 0.5)
```

```
## [1] 0.1171875
```

```r
# Calculate the probability that a normal random variable (with 
# mean of 3 and standard deviation of 2) is less than or equal to 4.
# This is a cumulative (p)robability of a (norm)al variable.
pnorm(4, 3, 2)
```

```
## [1] 0.6914625
```

```r
# Find the t-value that corresponds to a 2.5% right-hand tail probability
# with 5 degrees of freedom.
# This is a (q)uantile of a (t)distribution.
qt(0.975, 5)
```

```
## [1] 2.570582
```

```r
# Simulate 5 Poisson random variables with a mean of 3. 
# This is a set of (r)andom numbers from a (pois)son distribution.
rpois(5, 3)
```

```
## [1] 1 3 3 3 5
```

See the help page `?Distributions` for more details.

To make a quick plot of a distribution, we can use the density function in combination with `curve`. The following code makes Fig. \@ref(fig:distplot).


```r
# A standard normal distribution
curve(dnorm(x, sd=1, mean=0), from=-3, to=3,
      ylab="Density", col="blue")

# Add a t-distribution with 3 degrees of freedom.
curve(dt(x, df=3), from =-3, to=3, add=TRUE, col="red")

# Add a legend (with a few options, see ?legend)
legend("topleft", c("Standard normal","t-distribution, df=3"), lty=1, col=c("blue","red"),
       bty='n', cex=0.8)
```

![(\#fig:distplot)Two univariate distributions plotted with curve()](05-linearregression_files/figure-latex/distplot-1.pdf) 

\BeginKnitrBlock{rmdtry}<div class="rmdtry">Make a histogram of a sample of random numbers from a distribution of your choice. use `hist` and one of the functions starting with `r` from above.</div>\EndKnitrBlock{rmdtry}



## Descriptive Statistics {#descstat}

Descriptive statistics summarise some of the properties of a given data set. Generally, we are interested in measures of location (central tendency, such as mean and median) and scale (variance or standard deviation). Other descriptions can include the sample size, the range, and so on. We already encountered a number of functions that can be used to summarize a vector. 

Let's look at some examples for the Pupae dataset.
  

```r
# Read data
data(pupae)

# Extract the weights (for convenience)
weight <- pupae$PupalWeight

# Find the number of observations
length(weight)
```

```
## [1] 84
```

```r
# Find the average (mean) weight
mean(weight)
```

```
## [1] 0.3110238
```

```r
# Find the Variance
var(weight)
```

```
## [1] 0.004113951
```

Note that R will compute the sample variance (not the population variance). The standard deviation can be calculated as the square root of the variance, or use the `sd` function directly.


```r
# Standard Deviation
sqrt(var(weight))
```

```
## [1] 0.06414009
```

```r
# Standard Deviation
sd(weight)
```

```
## [1] 0.06414009
```

Robust measures of the location and scale are the median and inter-quartile range; R has functions for these.


```r
# median and inter-quartile range
median(weight)
```

```
## [1] 0.2975
```

```r
IQR(weight)
```

```
## [1] 0.09975
```

The median is the 50th percentile or the second quartile. The `quantile` function can compute quartiles as well as arbitrary percentiles/quantiles.


```r
# Default: computes quartiles.
quantile(weight)
```

```
##      0%     25%     50%     75%    100% 
## 0.17200 0.25625 0.29750 0.35600 0.47300
```

```r
# Or set any quantiles
quantile(weight, probs=seq(0,1,0.1))
```

```
##     0%    10%    20%    30%    40%    50%    60%    70%    80%    90% 
## 0.1720 0.2398 0.2490 0.2674 0.2892 0.2975 0.3230 0.3493 0.3710 0.3910 
##   100% 
## 0.4730
```

**Missing Values**: All of the above functions will return `NA` if the data contains *any* missing values. However, they also provide an option to remove missing values (`NA`s) before their computations (see also Section \@ref(workingmissing)).


```r
mean(pupae$Frass)
```

```
## [1] NA
```

```r
mean(pupae$Frass, na.rm=TRUE)
```

```
## [1] 1.846446
```

The `summary` function provides a lot of the above information in a single command:

```r
summary(weight)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##  0.1720  0.2562  0.2975  0.3110  0.3560  0.4730
```

The `moments` package provides 'higher moments' if required, for example, 
the `skewness` and `kurtosis`.

```r
# load the moments package
library(moments)
skewness(weight)
```

```
## [1] 0.3851656
```

```r
kurtosis(weight)
```

```
## [1] 2.579144
```

The `pastecs` package includes a useful function that calculates many descriptive statistics for numeric vectors, including the standard error for the mean (for which R has no built-in function).


```r
library(pastecs)
```

```
## 
## Attaching package: 'pastecs'
```

```
## The following object is masked from 'package:magrittr':
## 
##     extract
```

```
## The following objects are masked from 'package:dplyr':
## 
##     first, last
```

```r
# see ?stat.desc for description of the abbreviations
stat.desc(weight)
```

```
##      nbr.val     nbr.null       nbr.na          min          max 
## 84.000000000  0.000000000  0.000000000  0.172000000  0.473000000 
##        range          sum       median         mean      SE.mean 
##  0.301000000 26.126000000  0.297500000  0.311023810  0.006998258 
## CI.mean.0.95          var      std.dev     coef.var 
##  0.013919253  0.004113951  0.064140091  0.206222446
```

```r
# conveniently, the output is a character vector which we can index by name,
# for example extracting the standard error for the mean
stat.desc(weight)["SE.mean"]
```

```
##     SE.mean 
## 0.006998258
```


Sometimes you may wish to calculate descriptive statistics for subgroups in the data. We will come back to this extensively in Section \@ref(tapplyaggregate) and later sections.




## Testing differences between groups



### Testing a single sample {#singlesample}

In some applications we simply want to know whether the mean of our data (the sample) is equal to some hypothesized value, or whether it is clearly higher or lower. The simplest approach is to compute a 95% *confidence interval for the mean*, and then check whether the hypothesized value falls inside this interval (in which case you cannot conclude the value is different).

One way to get confidence intervals in R is to use the quantile functions for the relevant distribution. A $100(1-\alpha)$\% confidence interval for the mean on normal population is given by,

\[\bar{x} \pm t_{\alpha/2, n-1} \frac{s}{\sqrt{n}}\]

where $\bar{x}$ is the sample mean, $s$ the sample standard deviation and $n$ is the sample size. $t_{\alpha/2, n-1}$ is the $\alpha/2$ tail point of a $t$-distribution on $n-1$ degrees of freedom. That is, if $T$ has a $t$-distribution on $n-1$ degrees of freedom.

\[P(T \leq t_{\alpha/2, n-1}) = 1-\alpha/2 \]

The R code for this confidence interval can be written as,


```r
# Sample data - the pupae data.
data(pupae)
weight <- pupae$PupalWeight

# 95% confidence interval - set to 0.1 for a 90% interval
alpha <- 0.05 
xbar <- mean(weight)
s <- sd(weight)
n <- length(weight)
half.width <- qt(1-alpha/2, n-1)*s/sqrt(n)

# Confidence Interval 
c(xbar - half.width, xbar + half.width)
```

```
## [1] 0.2971046 0.3249431
```

Here, we assumed a normal distribution for the population. You may have been taught that if `n` is *large*, say n>30, then you can use a normal approximation. That is, replace `qt(1-alpha/2, n-1)` with `qnorm(1-alpha/2)`, but there is no need, R can use the t-distribution for any *n* (and the results will be the same, as the t-distribution converges to a normal distribution when the df is large).

\BeginKnitrBlock{rmdtry}<div class="rmdtry">Confirm that the $t$-distribution converges to a normal distribution when *n* is large (using `qt` and `qnorm`).</div>\EndKnitrBlock{rmdtry}

### Hypothesis testing {#inference}

There may be a reason to ask whether a dataset is consistent with a certain mean. For example, are the pupae weights consistent with a population mean of 0.29? For normal populations, we can use Student's $t$-test, available in R as the `t.test` function. In all following results, we use the `pander` function (from the `pander`) package to make the output short and readable (and in markdown format). For best results, use it in an rmarkdown document (though it works fine otherwise as well). We assume you have loaded these two packages:


```r
library(pander)
library(dplyr)
```

Let's test the null hypothesis that the population mean is 0.29:
  

```r
t.test(weight, mu=0.29) %>% pander
```


------------------------------------------------------------------------
 Test statistic   df     P value     Alternative hypothesis   mean of x 
---------------- ---- ------------- ------------------------ -----------
     3.004        83   0.00352 * *         two.sided            0.311   
------------------------------------------------------------------------

Table: One Sample t-test: `weight`
  
Note that we get the t-statistic, degrees of freedom (n-1) and a P value for the test, with the specified alternative hypothesis (not equal, i.e. two-sided). In addition, `t.test` gives us the estimated mean of the sample.

We can use `t.test` to do one-sided tests,


```r
t.test(weight, mu=0.29, alternative="greater") %>% pander
```


------------------------------------------------------------------------
 Test statistic   df     P value     Alternative hypothesis   mean of x 
---------------- ---- ------------- ------------------------ -----------
     3.004        83   0.00176 * *          greater             0.311   
------------------------------------------------------------------------

Table: One Sample t-test: `weight`

The `t.test` is appropriate for data that is approximately normally distributed. You can check this using a histogram or a QQ-plot. If the data is not very close to a normal distribution then the `t.test` is often still appropriate, as long as the sample is large.

If the data is not normal and the sample size is small, there are a couple of alternatives: transform the data (often a log transform is enough) or use a *nonparametric* test, in this case the Wilcoxon signed rank test. We can use the `wilcox.test` function for the latter, its interface is similar to `t.test` and it tests the hypothesis that the data is symmetric about the hypothesized population mean. For example,


```r
wilcox.test(weight, mu=0.29) %>% pander
```


--------------------------------------------------------
 Test statistic     P value      Alternative hypothesis 
---------------- -------------- ------------------------
      2316        0.009279 * *         two.sided        
--------------------------------------------------------

Table: Wilcoxon signed rank test with continuity correction: `weight`
  
#### Test for proportions

Sometimes you want to test whether observed proportions are consistent with a hypothesized population proportion. For example, consider a coin tossing experiment where you want to test the hypothesis that you have a fair coin (one with an equal probability of landing heads or tails). In your experiment, you get  60 heads out of 100 coin tosses. Do you have a fair coin? We can use the `prop.test` function:
  

```r
# 60 'successes' out of a 100 trials, the hypothesized probability is 0.5.
prop.test(x=60, n=100, p=0.5) %>% pander
```


--------------------------------------------------------------
 Test statistic   df   P value   Alternative hypothesis    p  
---------------- ---- --------- ------------------------ -----
      3.61        1    0.05743         two.sided          0.6 
--------------------------------------------------------------

Table: 1-sample proportions test with continuity correction: `60 out of 100, null probability 0.5`

Likewise, we can perform one-sided tests with the argument ` alternative="greater"` (not shown).


### Inference for two populations

Commonly, we wish to compare two (or more) populations. For example, the `pupae` dataset has pupal weights for female and male pupae. We may wish to compare the weights of males (`gender=0`) and females (`gender=1`). 

To compare the pupal weights of males and females, we use the *formula* interface for `t.test`. The formula interface is important because we will use it in many other functions, like linear regression and linear modelling. 


```r
# (output not shown)
# We assume equal variance between the groups, giving slightly more power,
# but see section 'unequal variances' further below.
t.test(PupalWeight ~ Gender, data=pupae, var.equal=TRUE)
```
  

#### Paired data

The `t.test` can also be used when the data are paired, for example, measurements taken before and after some treatment on the same subjects. For this example, we will use the pulse data - data on pulse rates of individuals before and after exercise. We will test the simple idea that pulse rate is different after exercise. To do this, we first extract only those subjects that exercised (`Ran=1`),


```r
# (output not shown)
pulse <- read.table("ms212.txt", header=TRUE)
pulse.before <- with(pulse, Pulse1[Ran==1])
pulse.after <- with(pulse, Pulse2[Ran==1])
t.test(pulse.after, pulse.before, paired=TRUE)
```
  
#### Unequal variances

The default for the two-sample `t.test` is actually to *not* assume equal variances. The theory for this kind of test is quite complex, and the resulting $t$-test is now only approximate, with an adjustment called the 'Satterthwaite' or 'Welch' approximation made to the degrees of freedom.


```r
t.test(PupalWeight ~ Gender,  data=pupae) %>% pander
```


-------------------------------------------------------------------
 Test statistic    df         P value       Alternative hypothesis 
---------------- ------- ----------------- ------------------------
     -7.413       74.63   1.587e-10 * * *         two.sided        
-------------------------------------------------------------------

Table: Welch Two Sample t-test: `PupalWeight` by `Gender` (continued below)

 
-----------------------------------
 mean in group 0   mean in group 1 
----------------- -----------------
     0.2725            0.3513      
-----------------------------------
  
Since this modified t-test makes fewer assumptions, you could ask why we ever use the equal variances form. If the assumption is reasonable, then this (equal variances) form will have more power, i.e. will reject the null hypothesis more often when it is actually false.


### Testing many groups at once (ANOVA)

One-way ANOVA (ANalysis Of VAriance) can be used to compare means across two or more populations. We will not go into the theory here, but the foundation of ANOVA is comparing the variation *between* the group means to the variation *within* the groups (using an F-statistic). 

We can use either the `aov` function or `lm` to perform ANOVAs. We will focus exclusively on the latter as it can be generalized more easily to other models. The use of `aov` is only appropriate when you have a balanced design (i.e., the same sample sizes in each of your groups).

To use `lm` for an ANOVA, we need a dataframe containing a (continuous) response variable and a factor variable that defines the groups. For example, in the Coweeta dataset, the species  variable is a factor that defines groups by species.  We can compute (for example) the mean height by species. Let's look at an example using the Coweeta data, but with only four species to simplify the output.



```r
library(dplyr)

# Take a subset and drop empty levels with droplevels.
data(coweeta)
cowsub <- filter(coweeta, species %in% c("bele","cofl","oxar","quru")) %>%
  droplevels

# # Quick summary table (uses dplyr)
group_by(cowsub, species) %>%
  summarize(height = mean(height))
```

```
## # A tibble: 4 x 2
##   species height
##   <fct>    <dbl>
## 1 bele     21.9 
## 2 cofl      6.80
## 3 oxar     16.5 
## 4 quru     21.1
```

We might want to ask, does the mean height vary by species? Before you do any test for significance, a graphical summary of the data is always useful. For this type of data, box plots are preferred since they visualize not just the means but also the spread of the data (Fig. \@ref(fig:allombox)).


```r
boxplot(height~species, data=cowsub)
```

![(\#fig:allombox)Simple box plot for the Coweeta data.](05-linearregression_files/figure-latex/allombox-1.pdf) 

It seems like some of the species differences are quite large. We can fit a one-way ANOVA with `lm`, like so:


```r
fit1 <- lm(height ~ species, data=cowsub)
```

The fitted coefficients can be seen in `summary(fit1)`, or more concisely with `coef(fit1)`, but we prefer the use of the `broom` package, especially the functions `tidy` and `glance` :


```r
library(broom)
tidy(fit1)
```

```
## # A tibble: 4 x 5
##   term        estimate std.error statistic  p.value
##   <chr>          <dbl>     <dbl>     <dbl>    <dbl>
## 1 (Intercept)   21.9        1.56    14.0   7.02e-14
## 2 speciescofl  -15.1        2.93    -5.15  2.04e- 5
## 3 speciesoxar   -5.36       2.35    -2.28  3.04e- 2
## 4 speciesquru   -0.761      2.27    -0.335 7.40e- 1
```

Here, `tidy` gives the table of coefficients. Notice the four estimated `Coefficients`, these represent the so-called *contrasts*. In this case, `Intercept` represents the mean of the *first* species, `bele`. The next three coefficients are the differences between each species and the first (e.g., species `cofl` has a mean that is -15.07 lower than `bele`). Also shown are the *t*-statistic (and p-value) for each coefficient, for a test where the value is compared to zero. Not surprisingly the `Intercept` (i.e., the mean for the first species) is significantly different from zero (as indicated by the very small p-value). Two of the next three coefficients are also significantly different from zero.

We can get more details of the fit using `glance`.


```r
glance(fit1)
```

```
## # A tibble: 1 x 11
##   r.squared adj.r.squared sigma statistic p.value    df logLik   AIC   BIC
##       <dbl>         <dbl> <dbl>     <dbl>   <dbl> <int>  <dbl> <dbl> <dbl>
## 1     0.533         0.481  4.95      10.3 1.11e-4     4  -91.4  193.  200.
## # ... with 2 more variables: deviance <dbl>, df.residual <int>
```

Here we see various useful statistics including the R squared (goodness of fit), and the p-value of the overall model. This p-value  tells us whether the whole model is significant. In this case, it is comparing a model with four coefficients (one for each species) to a model that just has the same mean for all groups. In this case, the model is highly significant -- i.e. there is evidence of different means for each group. In other words, a model where the mean varies between the four species performs much better than a model with a single grand mean.

### Multiple comparisons {#multcomp}

The ANOVA, as used in the previous section, gives us a single p-value for the overall 'species effect'. The summary statement further shows whether individual species are different from the first level in the model, which is not always useful. If we want to know whether the four species were all different from each other, we can use a multiple comparison test.

\BeginKnitrBlock{rmdcaution}<div class="rmdcaution">Multiple comparisons on linear models where you have more than one factor variable are tricky, and probably best left alone. Read the help page ?`glht` for more information.</div>\EndKnitrBlock{rmdcaution}

We will use the `glht` function from the `multcomp` package (as a side note, base R includes the `TukeyHSD` function, but that does not work with `lm`, only with `aov`).


```r
# First fit the linear model again (a one-way ANOVA, 
# because species is a factor)
lmSpec <- lm(height ~ species, data=cowsub)

# Load package
library(multcomp)

# Make a 'general linear hypothesis' object, Tukey style.
# (Note many other options in ?glht)
tukey_Spec <- glht(lmSpec, linfct=mcp(species="Tukey"))

# Print a summary. This shows p-values for the null hypotheses
# that species A is no different from species B, and so on.
summary(tukey_Spec)
```

```
## 
## 	 Simultaneous Tests for General Linear Hypotheses
## 
## Multiple Comparisons of Means: Tukey Contrasts
## 
## 
## Fit: lm(formula = height ~ species, data = cowsub)
## 
## Linear Hypotheses:
##                  Estimate Std. Error t value Pr(>|t|)    
## cofl - bele == 0 -15.0695     2.9268  -5.149   <0.001 ***
## oxar - bele == 0  -5.3620     2.3467  -2.285   0.1247    
## quru - bele == 0  -0.7614     2.2731  -0.335   0.9866    
## oxar - cofl == 0   9.7075     3.0295   3.204   0.0169 *  
## quru - cofl == 0  14.3081     2.9729   4.813   <0.001 ***
## quru - oxar == 0   4.6006     2.4039   1.914   0.2434    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## (Adjusted p values reported -- single-step method)
```

```r
# Some of the species are different from each other, but not all.
```

\BeginKnitrBlock{rmdtry}<div class="rmdtry">In the above summary of the multiple comparison (`summary(tukey_Spec)`), the p-values are adjusted for multiple comparisons with the so-called 'single-step method'. To use a different method for the correction (there are many), try the following example: `summary(tukey_Spec, test=adjusted("Shaffer"))`
Also look at the other options in the help pages for `?adjusted` and `?p.adjust`.</div>\EndKnitrBlock{rmdtry}

We can also produce a quick plot of the multiple comparison, which shows the pair-wise differences between the species with confidence intervals.

This code produces Fig. \@ref(fig:tukeyplot).


```r
# A plot of a fitted 'glht' object (multiple comparison)
plot(tukey_Spec)
```

![(\#fig:tukeyplot)A standard plot of a multiple comparison.](05-linearregression_files/figure-latex/tukeyplot-1.pdf) 



### Comparing many groups by two predictors (two-way ANOVA) {#twoway}

Sometimes there are two (or more) *treatment* factors. The 'age and memory' dataset (see `?memory`)  includes the number of words remembered from a list for two age groups and five memory techniques. 

This dataset is balanced, as shown below. in a table of counts for each of the combinations. First we fit a linear model of the *main effects*.

 

```r
data(memory)

# To make the later results easier to interpret, reorder the Process
# factor by the average number of words remembered.
memory <- mutate(memory, 
                 Process = reorder(Process, Words, mean))

# Count nr of observations
xtabs( ~ Age + Process, data=memory)
```

```
##          Process
## Age       Counting Rhyming Adjective Imagery Intentional
##   Older         10      10        10      10          10
##   Younger       10      10        10      10          10
```

```r
# Fit linear model
fit2 <- lm(Words ~ Age + Process, data=memory)

# Full output (not shown)
# summary(fit2)

# Instead, show just coefficient summary table in nice format
broom::tidy(fit2)
```

```
## # A tibble: 6 x 5
##   term               estimate std.error statistic  p.value
##   <chr>                 <dbl>     <dbl>     <dbl>    <dbl>
## 1 (Intercept)            5.20     0.763     6.81  8.99e-10
## 2 AgeYounger             3.10     0.623     4.97  2.94e- 6
## 3 ProcessRhyming         0.5      0.985     0.507 6.13e- 1
## 4 ProcessAdjective       6.15     0.985     6.24  1.24e- 8
## 5 ProcessImagery         8.75     0.985     8.88  4.41e-14
## 6 ProcessIntentional     8.90     0.985     9.03  2.10e-14
```

The `summary` of the fitted model displays the individual t-statistics for each estimated coefficient. As with the one-way ANOVA, the significance tests for each coefficient are performed relative to the base level (by default, the first level of the factor). In this case, for the `Age` factor, the `Older` is the first level, and for the `Process` factor, `Adjective` is the first level. Thus all other coefficients are tested relative to the "Older/Adjective" group. The $F$-statistic at the end is for the overall model, it tests whether the model is significantly better than a model that includes only a mean count.

If we want to see whether `Age` and/or `Process` have an effect, we need F-statistics for these terms. Throughout this book, to compute p-values for terms in linear models, we use the `Anova` function from the `car` package.


```r
# Perform an ANOVA on a fitted model, giving F-statistics
library(car)
Anova(fit2)
```

```
## Anova Table (Type II tests)
## 
## Response: Words
##            Sum Sq Df F value    Pr(>F)    
## Age        240.25  1  24.746 2.943e-06 ***
## Process   1514.94  4  39.011 < 2.2e-16 ***
## Residuals  912.60 94                      
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

In this form, the $F$-statistic is formed by comparing models that do not include the term, but include all others. For example, `Age` is tested by comparing the full model against a model that includes all other terms (in this case, just `Process`).


### Interactions

An important question when we have more than one factor in an experiment is whether there are any interactions. For example, do `Process` effects differ for the two `Age` groups, or are they simply additive? We can add interactions to a model by modifying the formula. An interaction is indicated using a "`:`". We can also include all *main effects and interactions* using the  `*` operator.


```r
# Two equivalent ways of specifying a linear model that includes all main effects
# and interactions:
fit3 <- lm(Words ~ Age + Process + Age:Process, data=memory)

# Is the same as:
fit3.2 <- lm(Words ~ Age * Process, data=memory)
Anova(fit3.2)
```

```
## Anova Table (Type II tests)
## 
## Response: Words
##              Sum Sq Df F value    Pr(>F)    
## Age          240.25  1 29.9356 3.981e-07 ***
## Process     1514.94  4 47.1911 < 2.2e-16 ***
## Age:Process  190.30  4  5.9279 0.0002793 ***
## Residuals    722.30 90                      
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

The `Anova` table shows that the interaction is significant. When an interaction is significant, this tells you nothing about the direction or magnitude of the interaction term. You can inspect the estimated coefficients in the `summary` output, but we recommend to first visualize the interaction with simple plots, as the coefficients can be easily misinterpreted. One way to visualize the interaction is to use the `interaction.plot` function, as in the following example. 

This code produces Fig. \@ref(fig:interactionplot). If there were no interaction between the two factor variables, you would expect to see a series of parallel lines (because the effects of `Process` and `Age` would simply be additive).


```r
# Plot the number of words rememberd by Age and Process
# This standard plot can be customized in various ways, see ?interaction.plot
with(memory, interaction.plot(Age, Process, Words))
```

![(\#fig:interactionplot)An interaction plot for the memory data, indicating a strong interaction (because the lines are not parallel).](05-linearregression_files/figure-latex/interactionplot-1.pdf) 

\BeginKnitrBlock{rmdtry}<div class="rmdtry">When there is a signficant interaction term, we might want to know under what levels of one factor is the effect of another factor signficant. This can be done easily using functions in the `emmeans` package. Load the `emmeans` package, and run the code `fit3.emm <- emmeans(fit3, ~ Age | Process)`, followed by `pairs(fit3.emm)`. You can now inspect in great detail differences between levels of your predictors.</div>\EndKnitrBlock{rmdtry}


### Comparing models

In the above example, we fitted two models for the Memory dataset: one without, and one with the interaction between `Process` and `Age`. We assessed the significance of the interaction by inspecting the p-value for the `Age:Process` term in the `Anova` statement. Another possibility is to perform a likelihood ratio test on two 'nested' models, the model that includes the term, and a model that excludes it. We can perform a likelihood ratio test with the `anova` function, not to be confused with `Anova` from the `car` package!\footnote{You may have seen the use of `anova` on a single model, which also gives an ANOVA table but is confusing because a sequential (Type-I) test is performed, which we strongly advise against as it is never the most intuitive test of effects.}


```r
# We can perform an anova on two models to compare the fit.
# Note that one model must be a subset of the other model. 
# In this case, the second model has the same predictors as the first plus
# the interaction, and the likelihood ratio test thus tests the interaction.
anova(fit2,fit3)
```

```
## Analysis of Variance Table
## 
## Model 1: Words ~ Age + Process
## Model 2: Words ~ Age + Process + Age:Process
##   Res.Df   RSS Df Sum of Sq      F    Pr(>F)    
## 1     94 912.6                                  
## 2     90 722.3  4     190.3 5.9279 0.0002793 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

A second common way to compare different models is by the AIC (*Akaike's Information Criterion*), which is calculated based on the likelihood (a sort of goodness of fit), and penalized for the number of parameters (i.e. number of predictors in the model). The model with the lowest AIC is the preferred model. Calculating the `AIC` is simple,


```r
AIC(fit2, fit3)
```

```
##      df      AIC
## fit2  7 518.9005
## fit3 11 503.5147
```

We once again conclude that the interaction improved the model fit substantially.

### Diagnostics

The standard ANOVA assumes normality of the residuals, and we should always check this assumption with some diagnostic plots (Fig. \@ref(fig:anovadiag1)). Although R has built-in diagnostic plots, we prefer the use of `qqPlot` and `residualPlot`, both from the `car` package.


```r
par(mfrow=c(1,2))

library(car)

# Residuals vs. fitted
residualPlot(fit3)

# QQ-plot of the residuals
qqPlot(fit3)
```

```
## [1] 26 86
```

![(\#fig:anovadiag1)Simple diagnostic plots for the Memory ANOVA.](05-linearregression_files/figure-latex/anovadiag1-1.pdf) 

The QQ-plot shows some slight non-normality in the upper right. The non-normality probably stems from the fact that the `Words` variable is a 'count' variable.

\BeginKnitrBlock{rmdtry}<div class="rmdtry">Check whether a log-transformation of `Words` makes the residuals closer to normally distributed.</div>\EndKnitrBlock{rmdtry}






<!-- ### Testing or predicting? -->


<!-- *Inference* is answering questions about population parameters based on a sample. The mean of a random sample from a population is an estimate of the population mean. Since it is a single number it is called a point estimate. It is often desirable to estimate a range within which the population parameter lies with high probability. This is called a confidence interval. -->

<!-- In other applications we are not so much interested in testing for difference, but instead want to develop a model that we can use for *prediction* of new values. -->

<!-- ... -->

<!-- **Testing: p-values and confidence intervals** -->

<!-- **Predicting: cross-validation** -->

<!-- The real quality of a predictive model is assessed by a completely independent test dataset. We can use the model to predict new outcomes, given the test data, and compare the predictions to the true values. In practice, however, a truly independent test dataset is rarely if ever available. Instead, we can use a trick to pretend we have lots of 'independent' test datasets: split the original data into a 'training' set (the data used to fit the model), and a 'test' set (the data used to test the model). -->

<!-- Rather than treat this topic exhaustively, we will show one very popular method to assess predictive power of models: k-fold cross validation.  -->

<!-- ... -->

<!-- (separate chapter with p-values, confidence intervals? Or mixed in here? -->
<!-- cross validation also applies to non-linear models, classification) -->


<!-- Sometimes models are better for prediction of new cases, at the expense of a slightly worse fit for the data used to fit them. Ridge regression is an example (section on shrinkage?). -->

<!-- ```{r eval=FALSE} -->
<!-- # shrink package, works on lm coefficients -->
<!-- # https://www.rdocumentation.org/packages/shrink/versions/1.2.1/topics/shrink -->
<!-- # MASS::lm.ridge -->
<!-- # ridge with glmnet -->
<!-- # https://drsimonj.svbtle.com/ridge-regression-with-glmnet -->

<!-- ``` -->






## Linear regression



### Icecream sales: a motivating example {#icecream}

It is well known that ice cream sales are higher during warm weather. As an ice cream salesman, you hardly have to apply regression analysis to know this - it is just obvious. Suppose that one year, you sell icecream at Oosterpark in Amsterdam. Sales are pretty good, the summer is nice and warm. The next year you try your luck at the Dappermarkt, a busy market in the east of Amsterdam. You tally your sales every week, and finally make a plot comparing average weekly sales:



```r
data(icecream)

library(ggplot2)
library(ggthemes)

ggplot(icecream, aes(x = location, y = sales, fill = location)) + 
  scale_fill_manual(values=c("white","grey")) + 
  geom_boxplot() +
  theme(aspect.ratio = 2, legend.position="none") +
  lims(y=c(0,1200))
```

![(\#fig:unnamed-chunk-38)Average weekly ice cream sales at two locations, over two years](05-linearregression_files/figure-latex/unnamed-chunk-38-1.pdf) 

Based on this simple boxplot, you would either conclude that there is no difference in sales, or that perhaps sales were slightly better in Oosterpark. We can also perform a *t*-test for two samples, which tests the hypothesis that the two means are not different (in other words, location has no effect on sales).


```r
t.test(sales ~ location, data=icecream)
```

```
## 
## 	Welch Two Sample t-test
## 
## data:  sales by location
## t = -1.3227, df = 36.442, p-value = 0.1942
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  -183.16655   38.51966
## sample estimates:
## mean in group Dappermarkt  mean in group Oosterpark 
##                  660.4856                  732.8090
```

\BeginKnitrBlock{rmdnote}<div class="rmdnote">The example above - as all other examples in this chapter - shows the raw output produced by R. To produce attractive output for use in an rmarkdown document, use the `pander` package: `pander::pander(t.test(sales ~ location, data=icecream))`. The `pander` function also works for many other R objects.</div>\EndKnitrBlock{rmdnote}


From this test we conclude no difference: the P value is very large (meaning the observed sample difference is not unusual, and could have easily arisen by chance).

Of course, there is something rather fishy about this experiment. We are testing the difference between two locations - but both locations were 'tested' in different years, when other conditions may have been different. In other words, we have not performed a proper experiment at all - changing just one variable at a time - location is *confounded* by the year. 

We do not need to get into detail here to design a proper scientific ice cream sales experiment, instead we are going to make the point that we can account for some *confounding variables* via regression analysis. How about temperature? As an astute ice cream salesman, you downloaded weekly average air temperature for Amsterdam, and added these to the data. Now let's make a plot using *air temperature as a covariate*.



```r
# Assuming you have loaded ggplot2 and ggthemes
ggplot(icecream, aes(x = temperature, y = sales, fill = location)) +
  geom_point(shape = 21, size=3) +
  scale_fill_manual(values = c("white","dimgrey")) +
  lims(y=c(0,1200), x=c(0,35))
```

![(\#fig:unnamed-chunk-41)Ice cream sales by location, and varying with temperature.](05-linearregression_files/figure-latex/unnamed-chunk-41-1.pdf) 

In this figure we arrive at a completely different conclusion - but only because we have taken into account the covariate weekly temperature. Now, *at a given temperature*, ice cream sales were higher at the new location - Dappermarkt. This is an example of how a confounding variable can mask (or even reverse) effects of some other variable. It seems obvious from the figure that temperature was overall lower when you sold ice cream at the Dappermarkt, explaining why average weekly sales were in the end no different.

\BeginKnitrBlock{rmdtry}<div class="rmdtry">Use a *t*-test to test that temperature was lower at the Dappermarkt location.</div>\EndKnitrBlock{rmdtry}

Using regression analysis, we can test the assertion that ice cream sales are higher at Dappermarkt, *at a given temperature*. But before we do that, we first introduce the tools to test difference between means (as we just did with a simple two-sample t-test), and introduce basic linear regression. We return to the icecream problem in Section \@ref(icecreamtest).



### Simple and multiple linear regression

To fit linear models of varying complexity, we can use the `lm` function. The simplest model is a straight-line relationship between an `x` and a `y` variable. In this situation, the assumption is that the `y`-variable (the response) is a linear function of the `x`-variable (the independent variable), plus some random noise or measurement error. For the simplest case, both `x` and `y` are assumed to be continuous variables. 

We use this method to study the relationship between two continuous variables: a *response* ('y variable') and a *predictor* (or independent variable) ('x variable'). We can use multiple regression to study the relationship between one response and more than one predictor. 

We are going to use the Cereals data to inspect the "health rating" (`rating`), and two predictors: fibre content (`fiber`) and sugar content (`sugar`). To start with, let's make two scatter plots, side by side.



```r
# Read the data, if you haven't already
data(cereals)

library(scales)
library(gridExtra)

# Two simple scatter plots on a log-log scale
g1 <- ggplot(cereals, aes(y = rating, x = fiber)) +
  geom_point(pch=15) + lims(y=c(0,100)) +
  geom_smooth(method='lm')

g2 <- ggplot(cereals, aes(y = rating, x = sugars)) +
  geom_point(pch=19, col="dimgrey") + lims(y=c(0,100)) +
  geom_smooth(method='lm')

grid.arrange(g1, g2, ncol = 2)
```

![(\#fig:allomdiamheight)Leaf area as a function of height and diameter (note the log-log scale).](05-linearregression_files/figure-latex/allomdiamheight-1.pdf) 

Here we make use of `ggplot2`'s built-in regression lines, which are easily added to the plot. The default behaviour is to add a 95% confidence band as well. In this case we immediately see that `fiber` is a worse predictor than `sugars` for the health rating, given the wider confidence interval for the location of the mean response (i.e. the regression line). 

We are going to fit a model that looks like, using statistical notation,
\begin{equation}
y = \alpha+\beta_1 x_1 +\beta_2 x_2 +\varepsilon
(\#eq:multiplelin)
\end{equation}

where $\alpha$ is the intercept, $x_1$ and $x_2$ are the two predictors (fiber and sugars), and the *two* slopes are $\beta_1$ and $\beta_2$. We are particularly interested in testing for significant effect of both predictors, which is akin to saying that we are testing the values of the two slopes against zero. 


```r
# Fit a multiple regression without interactions, and inspect the summary
fit4 <- lm(rating ~ sugars + fiber, data=cereals)

# A detailed summary (not shown for brevity)
# summary(fit4)

# A clean summary, especially for in an rmarkdown file
fit4 %>% pander
```


---------------------------------------------------------------
     &nbsp;        Estimate   Std. Error   t value   Pr(>|t|)  
----------------- ---------- ------------ --------- -----------
 **(Intercept)**    52.17       1.556       33.54    4.266e-46 

   **sugars**       -2.244      0.1632     -13.75    5.975e-22 

    **fiber**       2.867       0.2979      9.623    1.276e-14 
---------------------------------------------------------------

Table: Fitting linear model: rating ~ sugars + fiber

As you can see, we again use the `pander` package to give super clean results, which can be used in an `rmarkdown` document. We see here that overall we have an R^2^ of 81% (not bad), and that both predictors contribute significantly (the P values are very low for both).

Next, we perform some diagnostic plots (shown in Fig. \@ref(fig:cerealdiag)). The standard errors (and confidence intervals, and hypothesis tests) will be unreliable if the assumptions of normality (right panel) or linearity (left panel) are seriously violated. If you are only interested in the mean response against your predictors, but not so much their uncertainties, you can usually ignore these plots.


```r
# Basic diagnostic plots.
par(mfrow=c(1,2))
car::residualPlot(fit4)
car::qqPlot(fit4)
```

![(\#fig:cerealdiag)Diagnostic plots for linear regression of the Cereals data.](05-linearregression_files/figure-latex/cerealdiag-1.pdf) 

```
## [1] 27 31
```

Now that we have a fitted model, what do its predictions actually look like? What relationship have we learned between health rating, sugars, and fiber content? The `visreg` package provides a very concise method to visualize (many kinds of) fitted regression models.

In this case, the command `visreg::visreg(fit4)` would show two plots very similar to the ones above - which does not really add much information. What if we can show the effects of both predictors, in just one plot? The following code produces Fig. \@ref(fig:visregcereal).


```r
library(visreg)

visreg(fit4, "sugars", by = "fiber", overlay = TRUE, 
       gg = TRUE) + 
  lims(y = c(0,100))
```

![(\#fig:visregcereal)Visualizing linear regression of the Cereals data with visreg.](05-linearregression_files/figure-latex/visregcereal-1.pdf) 

Here we clearly, and in one panel, visualize the negative effect of adding more sugar in a cereal, and the positive effect of adding more fiber. The `visreg` function plots the *predictions* from our model at an arbitrary selection of predictor values (here, 0, 1.5 and 5). 

\BeginKnitrBlock{rmdtry}<div class="rmdtry">In the call to `visreg`, use the argument `breaks=c(1,5)` to set the predictor values to those of your choice.</div>\EndKnitrBlock{rmdtry}

We can also make 3D plots, though these are not always useful, and I personally prefer a 'pseudo-3D' plot as in the example above to visualize the contribution of two variables.

Of course, we are not restricted to just two predictors, and we can include  interactions as well. In the previous model (`fit4`) the effects of `fiber` and `sugars` were *additive*, in other words the effect of fiber was independent of the amount of sugars. We can test this assumption by adding the interaction to the model. 


```r
# A multiple regression that includes all main effects as wel as interactions
fit5 <- lm(rating ~ sugars + fiber + fiber:sugars, data=cereals)
summary(fit5) %>% pander::pander(.)
```


----------------------------------------------------------------
      &nbsp;        Estimate   Std. Error   t value   Pr(>|t|)  
------------------ ---------- ------------ --------- -----------
 **(Intercept)**     51.38       1.695       30.31    9.996e-43 

    **sugars**       -2.097      0.2056      -10.2    1.277e-15 

    **fiber**        3.226       0.428       7.537    1.12e-10  

 **sugars:fiber**   -0.07363    0.06316     -1.166     0.2475   
----------------------------------------------------------------


--------------------------------------------------------------
 Observations   Residual Std. Error   $R^2$    Adjusted $R^2$ 
-------------- --------------------- -------- ----------------
      76               6.112          0.8198       0.8123     
--------------------------------------------------------------

Table: Fitting linear model: rating ~ sugars + fiber + fiber:sugars

(*Note*: the formula can also be written as `fiber*sugars`)

In this case, the interaction is not significant: note the large P value for the *sugars:fiber* interaction. We can also inspect the `AIC`, and notice that the total model AIC has not decreased, which also indicates the model has not improved by adding an interaction term.


```r
AIC(fit4, fit5)
```

```
##      df      AIC
## fit4  4 496.1572
## fit5  5 496.7359
```


## Linear models with factors and continuous variables {#lmfaccont}

So far we have looked at ANOVA, including two-way ANOVA, where a response variable is modelled as dependent on two treatments or factors, and *regression* including multiple linear regression where a response variable is modelled as dependent on two continuous variables. These are just special cases of the *linear model*, and we can extend these simple models by including a mix of factors and continuous predictors. The situation where we have one continuous variable and one factor variable was classically` known as ANCOVA (analysis of covariance), but using the `lm` function we can specify any model with a combination of predictors.

In the following example, we inspect the height of trees against the stem diameter (`DBH`) for the Coweeta example dataset. When plotting all data, we are led to believe that the overall response of height to DBH is highly non-linear (left panel). However, when we fit a simple linear model for each species separately (right panel), the relationship appears to consist of a combination of linear responses.



```r
data(coweeta)

g1 <- ggplot(coweeta, aes(x = DBH, y = height)) +
  geom_point() + geom_smooth(method="loess", se=FALSE) +
  scale_colour_tableau() + lims(x=c(0,60), y=c(0,30))

g2 <- ggplot(coweeta, aes(x = DBH, y=height, col=species)) +
  geom_point() + geom_smooth(method="lm", se=FALSE) +
  scale_colour_tableau() + lims(x=c(0,60), y=c(0,30))

grid.arrange(g1, g2, ncol = 2, widths=c(0.4, 0.6))
```

![](05-linearregression_files/figure-latex/coweeta_twoplots-1.pdf)<!-- --> 


To fit a model that allows a different response for each species, we use the notation from before:


```r
fit6 <- lm(height ~ species * DBH, data=coweeta)
```



### Visualizing fitted regression models {#predictedeffects}

The coefficients associated with a factor predictor in a linear model are given as contrasts (i.e. differences between factor levels). 

While this is useful for comparisons of treatments, it is often more instructive to visualize the predictions at various combinations of factor levels. 

A number of options exist to extract and visualize fitted regression models - to make sense of differences between groups, the effects of covariates, and the impact of interactions. We prefer the `visreg` package, which can be used to make attractive plots of the predictions of a linear model.

The following example makes Fig. \@ref(fig:visreg1).


```r
library(visreg)

# Load data (lgrdata package needed)
data(memory)

# To make the later results easier to interpret, reorder the Process
# factor by the average number of words rememberer (we did this earlier in 
# the chapter already for this dataset, it is repeated here).
memory <- mutate(memory, 
                 Process = reorder(Process, Words, mean))

# Two linear models: one without, and one with an interaction
fit7 <- lm(Words ~ Age + Process, data=memory)
fit8 <- lm(Words ~ Age*Process, data=memory)

# Here we specify which variable should be on the X axis (Process),
# and which variable should be added with different colours (Age).
visreg(fit7, "Process", by="Age", overlay=TRUE)
visreg(fit8, "Process", by="Age", overlay=TRUE)
```

![(\#fig:visreg1)Visualization of two fitted linear model with the visreg package. The model on the right includes an interaction between Process and Age, the model on the left does not.](05-linearregression_files/figure-latex/visreg1-1.pdf) 

You can test for yourself that the interaction is a highly significant predictor in the model.


### Group differences in regression models: back to ice cream sales {#icecreamtest}

At the beginning of this chapter (Section \@ref(icecream)), we looked at an example where including a covariate drastically changed our conclusions about an effect. Simply testing the difference between two locations where we sold ice creams showed no effect, but including temperature at a covariate, it appeared that *at a given temperature*, there was a difference between the locations. 

Here, we continue this example as it shows that we need to closely inspect a fitted model before making any conclusions - and that estimated coefficients of a regression model can lead to wrong conclusions, if not carefully examined.

To continue, first read in the icecream data as shown in Section \@ref(icecream).
We now fit a linear model with one numeric, and one factor variable. We immediately include all effects, including the interaction.


```r
lm_ice1 <- lm(sales ~ location * temperature, data=icecream)
```

You can inspect all details in the summary of this model yourself (`summary(lm_ice1)`) - here we just look at the estimated coefficients from the model. Recall that in a linear model, the intercept is assigned to the first level of the factor variable in the model, in this case `levels(icecream$location)` shows that this is 'Dappermarkt'. 


```r
pander(lm_ice1)
```


----------------------------------------------------------------------------------
               &nbsp;                 Estimate   Std. Error   t value   Pr(>|t|)  
------------------------------------ ---------- ------------ --------- -----------
          **(Intercept)**              78.41       27.72       2.829     0.00759  

       **locationOosterpark**          99.64       42.53       2.343     0.02477  

          **temperature**              37.48       1.704        22      1.837e-22 

 **locationOosterpark:temperature**    -11.19      2.255      -4.963    1.683e-05 
----------------------------------------------------------------------------------

Table: Fitting linear model: sales ~ location * temperature

Four coefficients are shown, these are:

- *Intercept* : ice cream sales for the first level of 'Location' (Dappermarkt), when temperature equals zero (i.e. the intersection of the regression line with the y-axis when x = 0).
-  *locationOosterpark* : The difference in intercept for Oosterpark location, compared to the first level (Dappermarkt). A positive value here seems to indicate higher sales at the Oosterpark location (when temperature is zero - read further below!).
- *temperature* : The slope of sales per unit temperature (i.e. for every degree increase in temperature, sales go up this much), for Dappermarkt.
- *locationOosterpark:temperature* : the difference in *slope* for the Oosterpark, compared to Dappermarkt. This shows the slope is lower for Oosterpark. 

Based on the above, it seems difficult to draw conclusions about the difference between locations. This is because the intercept is not at all informative: it quantifies ice cream sales when temperature equals zero, but we do not even try to sell ice cream when it is cold outside. To visualize what is going on, we modify the standard `visreg` plot to draw the regression lines all the way to zero. We also show how to adjust the appearance of the plot (see `?plot.visreg` for details).


```r
visreg(lm_ice1, "temperature", by="location", 
       overlay = TRUE, xlim=c(0,40),
       line.par=list(col=c("dimgrey","grey")),
       fill.par=list(col=c("#BEBEBE80","#BEBEBE80")),
       points.par=list(col=c("dimgrey","grey"), cex=1.2))
```

![](05-linearregression_files/figure-latex/unnamed-chunk-49-1.pdf)<!-- --> 

We now see that the regression lines 'cross over' around a temperature of 10, giving a higher intercept for Oosterpark.

A meaningful test in situations like this is to test for group differences *at a given value of our covariate(s)*. A number of approaches exist in various add-on packages, but we prefer to first show the basic approach using `predict` - a function that has methods for practically all models we use. It also offers great flexibility, and we don't have to wonder what exactly we are testing.

Somewhat arbitrarily, we want to test for location differences at a temperature of 25 degrees. To do this, I will construct confidence intervals for both locations, as follows. In the call to `predict`, it is important to list values for all variables in the model. This makes this approach less practical for models with many more regressors (see below for another approach).


```r
# Predict from model, at temperature =25.
pred_ice <- predict(lm_ice1, 
                    newdata = data.frame(temperature = 25, 
                                         location = levels(icecream$location)),
                    se = TRUE)
```

If you inspect this object, you will find that sales at Dappermarkt are 1015.4 with a standard error of 18.1, and sales at Oosterpark are 835.2 (SE 10.1). The standard errors may be used to construct approximate 95% confidence intervals (by using twice the standard error).

A more convenient approach is to use the `emmeans` package to extract *marginal effects* of our fitted model. The default approach in this package is to test for group differences at the average level of other predictors. This approach is also called 'least-square means'. However, we can adjust what value we want to use.



```r
library(emmeans)
emmeans(lm_ice1, specs = "location", at = list(temperature = 25))
```

```
## NOTE: Results may be misleading due to involvement in interactions
```

```
##  location    emmean   SE df lower.CL upper.CL
##  Dappermarkt   1015 18.1 36      979     1052
##  Oosterpark     835 10.1 36      815      856
## 
## Confidence level used: 0.95
```

You can ignore the warning in this case, since we have used both the predictors in our model - there are no other 'hidden' predictors that may complicate matters. As you can see for yourself,  the `emmeans` package gives the same results as the basic `predict` approach, but it is certainly much more practical.


### Logarithmic transformation {#logtransform}

For data that are very heteroscedasctic, that is, have quickly increasing variance with the value of the predictor, it is usually necessary to use a logarithm transformation of the data. In extreme cases, not applying the log-transform leads to very poor model fits, with very large influence of a few points, and possibly a fit that does not describe the data well at all.

See Fig. \@ref(fig:cowtransform) for an example where a log-transform is used.


```r
data(coweeta)
library(scales)

g1 <- ggplot(coweeta, aes(x = biomass, y = folmass)) + 
  geom_point() +
  labs(x = "Mass of tree (kg)", y = "Mass of leaves (kg)")
    
g2 <- g1 + 
  scale_x_log10() + 
  scale_y_log10() +
  annotation_logticks() +
  expand_limits(x=c(1,10^4), y=100)

grid.arrange(g1, g2, ncol = 2)
```

![(\#fig:cowtransform)An example of a dataset that screams for a logarithmic transformation. Left panel: untransformed data, showing very non-constant variance, indicating larger variation in the response variable for larger individuals. Right panel: both variables have been log-transformed, giving a nice linear relationship with constant variance.](05-linearregression_files/figure-latex/cowtransform-1.pdf) 


\BeginKnitrBlock{rmdtry}<div class="rmdtry">Redo the above plot, using `origin` (i.e. American, Japanese or European) to color the symbols. This is another good example that you should always consider covariates in a linear model.</div>\EndKnitrBlock{rmdtry}

The following code fits two linear models to the data above, one to the raw, untransformed data, and one after log10-transforming both the predictor and the response variable. In previous examples we used `residualPlot` from the `car` package to quickly make plots of residuals versus fitted values, here we implement a simple version using `ggplot2` (and note that we jump ahead and define our own function, a topic we return to in Chapter \@ref(programming)).


```r
lm1 <- lm(folmass ~ biomass, data = coweeta)
lm2 <- lm(log10(folmass) ~ log10(biomass), data = coweeta)

residPlot <- function(model){
  ggplot(model, aes(.fitted, .resid)) + 
    geom_hline(yintercept = 0) + 
    geom_point() +
    stat_smooth(method="loess", span=0.8)
}

grid.arrange(residPlot(lm1), residPlot(lm2), ncol = 2)
```

![(\#fig:cowdiagnostics)Residuals versus fitted for the Coweeta biomass regression model, either without (left panel) or with (right panel) log-transformation of x and y variables.](05-linearregression_files/figure-latex/cowdiagnostics-1.pdf) 

As Fig. \@ref(fig:cowdiagnostics) shows, the log-transformation results in much better model diagnostics. We can expect that the model will give an overall better fit to the data, because when we did not transform, the location of the regression line will be overly influenced by values far away from the line. After transformation, this effect will be much reduced, and the model will be more robust. 

Second, the standard errors of the coefficients in the untransformed fit are very much biased, leading to confidence intervals that are much too narrow. In this case, a log-transformation could be used to remedy this problem, but in other cases we want to avoid transformations, or they are simply not adequate. In that case, we might use the bootstrap. 

This well known and often used transformation does have some consequences for interpreting the linear model fit that are often misunderstood. The key aspect to understand is that the model is no longer *additive* as the usual linear model (where effects of predicts are simply added), but rather *multiplicative* (effects of various predictors are *multiplied*). This has consequences for understanding interactions in the model as well.




### Adding quadratic and polynomial terms {#quadlm}

So far we have seen models with just linear terms, but it is straightforward and often necessary to add quadratic ($x^2$) or higher-order terms (e.g. $x^3$) when the response variable is far from linear in the predictors. You can add any transformation of the predictors in an `lm` model by nesting the transformation inside the `I()` function, like so:


```r
data(allometry)
lmq1 <- lm(height ~ diameter + I(diameter^2), data=allometry)
```

This model fits both the linear (`diameter`) and squared terms (`I(diameter\^2)`) of the predictor, as well as the usual intercept term. If you want to fit all polynomial terms up to some order, you can use the `poly` function like so,


```r
lmq1 <- lm(height ~ poly(diameter, 2), data=allometry)
```

This model specification is exactly equivalent to the above, and is more convenient when you have multiple quadratic / polynomial terms and interactions with factor variables. 

The following example quickly tests whether a quadratic term improves the model fit of `height` vs. `diameter` for the species `PIPO` in the allometry dataset. 


```r
data(allometry)
pipo <- subset(allometry, species == "PIPO")

# Fit model with the quadratic term:
lmq2 <- lm(height ~ diameter + I(diameter^2), data=pipo)

# The very small p-value for the quadratic terms shows that the 
# relationship is clearly not linear, but better described with a 
# quadratic model.
Anova(lmq2)
```

```
## Anova Table (Type II tests)
## 
## Response: height
##               Sum Sq Df F value    Pr(>F)    
## diameter      821.25  1  53.838 5.905e-07 ***
## I(diameter^2) 365.78  1  23.980 0.0001001 ***
## Residuals     289.83 19                      
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

When fitting a quadratic model, it is again very useful to use `visreg` to inspect the model that is estimated, since it becomes even more difficult to make sense of the various linear, quadratic, and intercept terms, especially when interactions with factor variables are added. Consider this example for the allometry dataset, which makes Fig. \@ref(fig:allomquad).


```r
# Fit a linear model with linear, quadratic terms, and all species interactions.
allomquadfit <- lm(height ~ poly(diameter, 2)*species, data=allometry)

# Inspect summary(allomquadfit) to confirm you cannot possibly make sense of this
# many coefficients.

# But plotting the predictions is much easier to understand:
visreg(allomquadfit, "diameter", by="species", overlay=TRUE)
```

![(\#fig:allomquad)Height modelled as a quadratic function of diameter, by species.](05-linearregression_files/figure-latex/allomquad-1.pdf) 



### Which predictors are more important? {#importance}

Frequently we are interested in knowing which of the predictor variables explain more variation in the response variable, in other words are more strongly correlated with the response.

One often-seen approach is to compute all correlations between the response and each of the predictors, and conclude that those with higher correlation coefficients will be most important. The flaw in this approach is that predictor variables are very often correlated with each other, making the approach potentially misleading. Cross-correlations between variables can easily hide important predictor variables. Nonetheless it can be a useful start, as we do here with the `automobiles` data. The following code makes Fig. \@ref(fig:mobilecorplot).


```r
data(automobiles)

# A modern implementation of a correlation plot in the corrplot package.
library(corrplot)
auto_cor <- cor(automobiles[,4:9], use = "complete.obs")
corrplot(auto_cor, method = "ellipse")
```

![(\#fig:mobilecorplot)Correlation plot for the automobiles data.](05-linearregression_files/figure-latex/mobilecorplot-1.pdf) 

As we can see in Fig. \@ref(fig:mobilecorplot), the response variable (fuel_efficiency) seems well correlated with four of the response variables, but weaker with `acceleration`.

Sometimes we see the use of the absolute value of the t-statistic to rank variables by 'importance' in the regression model. Within a single model, the t-statistic will be exactly related to the p-value (low p-values mean large t-statistic). Both t-statistics and p-values are very poor measures of importance. A large t-statistic may also arise when the sample size is large, the measurements are accurate, or there is low variability overall. 

Instead what we are often interested in is how much variation is explained by each variable in the model. This calculation is complicated by the fact that predictor variables tend to be correlated with each other, so you cannot simply add up the R^2^ due to each predictor separately.

The `relaimpo` package includes a number of importance measures - the recommended version can be interpreted as the % variation explained by each predictor variable:



```r
# A model with five predictors
fit_fuel1 <- lm(fuel_efficiency ~ cylinders + engine_volume +
                                  horsepower + weight + acceleration,
                data = automobiles)

# For importance measures
library(relaimpo)

# recommended method lmg in relaimpo
imp_fitfuel <- calc.relimp(fit_fuel1, type="lmg")

# The resulting object is a bit complicated.
# If you print the object (simply type imp_fitfuel), lots of information is given.
# Instead we just need this:
sort(imp_fitfuel$lmg, TRUE)
```

```
##        weight    horsepower engine_volume     cylinders  acceleration 
##    0.21667697    0.19895781    0.18619931    0.17158534    0.04737539
```

The values mean that `weight` explains 21% of the variation, but acceleration only 4%. Note that the sum of these values adds up (nearly precisely) to the R^2^ of the model (look at `summary(fit_fuel1)`).


\BeginKnitrBlock{rmdtry}<div class="rmdtry">A more common approach is to test how mucb the R^2^ decreases if a predictor is dropped from the model. When important variables are removed, the model fits a lot worse overall. This can be achieved via:
  
`drop1(fit_fuel1)`

Where RSS is the 'residual sum of squares' after dropping the term; higher means the variable is 'more important'.</div>\EndKnitrBlock{rmdtry}

## Exercises




### Probabilities

For this exercise, refer to the tables and examples in Section \@ref(distributions).


1.  For a normal random variable $X$ with mean 5.0, and standard deviation 2.0, find the probability that $X$ is less than 3.0.



2.  Find the probability that $X$ is *greater than* 4.5.



3.  Find the value $K$ so that $P(X > K) = 0.05$.




4.  When tossing a fair coin 10 times, find the probability of seeing no heads (*Hint:* this is a binomial distribution).



5.  Find the probability of seeing exactly 5 heads.



6.  Find the probability of seeing more than 7 heads.






### Univariate distributions


1.  Simulate a sample of 100 random data points from a normal distribution with mean 100 and standard deviation 5, and store the result in a vector called `x`.


2.  Plot a histogram and a boxplot of the vector you just created (see Section\@ref(hist)). If you haven't make a boxplot, simply use the `boxplot` function on your vector!



3.  Calculate the sample mean, standard deviation, median and interquartile range.


4.  Using the data above, test the hypothesis that the mean equals 100 (using `t.test`). In science, it is customary (though debatable) to call an effect `significant` if the p-value is smaller than 0.05. Note that here we test whether the true mean is different from 100 - in this case we *know* the true mean since the data were sampled from a normal distribution with mean 100. **Bonus question**: how often do we find a p-value smaller than 0.05 in this example, do you think? (that is, resampling the data from step 1., and then testing).



5.  Test the hypothesis that mean equals 90. 


6.  Repeat the above two tests using a Wilcoxon signed rank test. Compare the p-values with those from the $t$-tests you just did.





### More $t$-tests

For this question, use the `pupae` data.


1.  Use the `t.test` function to compare `PupalWeight` by `T_treatment`.


2.  Repeat above using a Wilcoxon rank sum test. 

```
## Warning in wilcox.test.default(x = c(0.244, 0.319, 0.221, 0.28, 0.257,
## 0.333, : cannot compute exact p-value with ties
```

3.  Run the following code to generate some data:

```
base <- rnorm(20, 20, 5)
x <- base + rnorm(20,0,0.5)
y <- base + rnorm(20,1,0.5)
```




4.  Using a two-sample t-test compare the means of `x` and `y`, assume that the variance is equal for the two samples.


5.  Repeat the above using a paired $t$-test. How has the $p$-value changed? 


6.  Which test is most appropriate?





### Simple linear regression

Continue with the `pupae` data. Perform a simple linear regression of `Frass` on `PupalWeight`. Produce and inspect the following:

1.  Summary of the model.


2.  Diagnostic plots.


3.  All of the above for a subset of the data, where `Gender` is 0, and `CO2\_treatment` is 400.




### Quantile Quest

You have already used quantile-quantile (QQ) plots many times, but in this exercise you will get to the bottom of the idea of comparing quantiles of distributions.

As in the previous exercises, we will use the `pupae` data.


1.  From the pupae data, extract `PupalWeight` and store it as a vector called 'pupweight'. Make a histogram of this vector (\@ref(hist)), noticing that the distribution seems perhaps quite like the normal distribution.



2.  When we say 'quite like the normal distribution', we mean that the overall shape seems similar. Now simulate a histogram like the one above, using `rnorm` with the mean and standard deviation of the pupal weights (i.e. `pupweight`), and the same sample size as well. Plot it repeatedly to get an idea of whether the simulated histogram looks similar often enough.



3.  Of course a visual comparison like that is not good enough, but it is a useful place to start. We can also compare the quantiles as follows. If we calculate the 25% and 75% quantiles of `pupweight`, we are looking for the values below which 25% or 75% of all observations occur. Clearly if two distributions have the same *shape*, their quantiles should be roughly similar. Calculate the 25, 50 and 75% quantiles for `pupweight`, and also calculate them for the normal distribution using `qnorm`. Are they similar?




4.  Now repeat the above exercise, but calculate many quantiles (e.g. from 2.5% to 97.5% with steps of 2.5% or whatever you choose) for both the measured data, and the standard normal distribution. Compare the two with a simple scatter plot, and add a 1:1 line. If you are able to do this, you just made your own QQ-plot (and if not, I suggest you inspect the solutions to this Exercise). *Hint:* use `seq` to make the vector of quantiles, and use it both in `quantile` and `qnorm`. Save the results of both those as vectors, and plot. As a comparison, use `qqPlot(pupweight, distribution="norm")` (`car` package), make sure to plot the normal quantiles on the X-axis.





### One-way ANOVA


1.  For the `titanic` data, use a one-way ANOVA to compare the average passenger age by passenger class. (Note: by default, `lm` will delete all observations where `Age` is missing.)



2.  For the Age and Memory data (Section\@ref(agemem}, p. sec:agemem}), make a subset of the `Older` subjects, and conduct a one-way ANOVA to compare words remembered by memory technique.




### Two-way ANOVA


1.  Using the pupae dataset, fit a two-way ANOVA to compare `PupalWeight` to `Gender` and `CO2_treatment`. Which main effects are significant? After reading in the pupae data, make sure to convert `Gender` and `CO2_treatment` to a factor first (see Section\@ref(workingfactors)).



2.  Is there an interaction between `Gender` and `CO2_treatment`?



3.  Repeat the above using `T_treatment` instead of `CO2_treatment`.







### Regression: the `pulse` dataset {#multregexerc}

The `pulse` data contains measurements of heart rate ("pulse") for individuals before and after running (and some control individuals) - and the individuals' height, weight and age (and other interesting variables). The `Pulse1` variable is the resting heart rate before any treatment, and `Pulse2` the heart rate after the treatment (half the subjects engaged in running). 

1.   Read the data and fit a multiple linear regression of `Pulse1` against `Age`, `Weight` and `Height` (add the variables to the model in that order). Are any terms significant at the 5\% level? What is the R^2^?




2.  Now also include the factor `Exercise` in the regression model (an indicator of whether the subject exercises frequently or not). You will need to first convert `Exercise` to a factor as it is stored numerically in the data. Does adding `Exercise` improve the model?




3.  Using the same  data, fit a model of `Pulse2` as a function of `Pulse1` and `Ran` as main effects only (Note: convert `Ran` to a factor first). Use the `visreg` package to understand the fit.





4.  Now add the interaction between `Pulse1` and `Ran`. Is it significant? Also look at the effects plot, how is it different from the model without an interaction?





5. Read Section \@ref(importance) on variable importance. Add a new variable to the `pulse` dataset, as the difference between `Pulse2` and `Pulse1` (the change in heartrate after exercise). Now fit a linear regression model with all of these terms: "Height", "Weight", "Age", "Gender", "Smokes", "Alcohol","Exercise","Ran". As expected `Ran` wil be most important, but how do all the other variables rank in importance?








### Logistic regression


1.  Using the Pulse data once more, build a model to see if `Pulse2` can predict whether people were in the `Ran` group. Make sure that `Ran` is coded as a factor.



1.  The `visreg` package is very helpful in visualizing the fitted model. For the logistic regression you just fit , run the following code and make sure you understand the output. (This code assumes you called the object `fit6`, if not change `fit6` to the name you used.)
\begin{verbatim}
library(visreg)
visreg(fit6, scale="response")
\end{verbatim}





<!-- ### Generalized linear model (GLM) -->


<!-- \item First run the following code to generate some data, -->
<!-- \begin{verbatim} -->
<!-- len <- 21 -->
<!-- x <- seq(0,1,length=len) -->
<!-- y <- rpois(len,exp(x-0.5)) -->
<!-- \end{verbatim} -->
<!-- ```{r echo=FALSE} -->
<!-- len <- 21 -->
<!-- x <- seq(0,1,length=len) -->
<!-- y <- rpois(len,exp(x-0.5)) -->
<!-- ``` -->

<!-- 1.  Fit a Poisson GLM to model `y` on `x`. Is `x` significant in the model? -->
<!-- ```{r } -->
<!-- fit7 <- glm(y ~ x, family=poisson) -->
<!-- summary(fit7) -->
<!-- ``` -->

<!-- 1.  Repeat above with a larger sample size (e.g., `len <- 101`). Compare the results. -->
<!-- ```{r } -->
<!-- len <-101 -->
<!-- x <- seq(0, 1, length=len) -->
<!-- y <- rpois(len, exp(x-0.5)) -->

<!-- fit <-glm(y ~ x, family=poisson) -->
<!-- summary(fit) -->
<!-- ``` -->

<!-- 1.  The `memory` data were analysed assuming a normal error distribution in Section\@ref(twoway} and using a Poisson error distribution in Section\@ref(glm}, and each approach resulted in a different outcome for the significance of the interaction term. The participants in the study were asked to remember up to 27 words, not an unlimited number, and some of the participants were remembering close to this upper limit. Therefore, it may make sense to think of the response as consisting of a number of 'successes' and a number of 'failures', as we do in a logistic regression. Use `glm` to model the response using a binomial error distribution. Refer to Section\@ref(tabuldataglm} (p. sec:tabuldataglm}) for a similar example. -->

<!-- ```{r } -->
<!-- # read in data -->
<!-- memory <- read.delim('eysenck.txt') -->

<!-- # Fit glm model, with response represented by two columns:  -->
<!-- # number of words remembered and not remembered, or -->
<!-- # 'successes' and 'failures'. -->
<!-- m1 <- glm(cbind(Words, 27-Words) ~ Age * Process, data=memory, family=binomial) -->

<!-- # estimate significance of the main effects -->
<!-- Anova(m1) -->
<!-- ``` -->









