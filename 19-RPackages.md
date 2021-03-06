# Mastering package dependencies {#masteringpackages}





Besides having to write dependable code yourself, we also have to worry about dependencies of R packages, that is, code written by other R users.


\BeginKnitrBlock{rmdreading}<div class="rmdreading">
First scan through the section on R Packages (\@ref(packages)) to make sure you grasp all the fundamentals of R packages.
</div>\EndKnitrBlock{rmdreading}

When developing more robust project workflows, we have to worry about several issues to do with the **reproducibility** of our code. 

- What package version did we use? 
- Does our code still work for newer package versions?
- Do loaded package have conflicts?
- Can we bundle the R packages along with our code?



## What version?

When an external package has changed on CRAN since you last used it, the *version* will have changed. You can inspect the version of a package with,


```r
packageVersion("dplyr")
```

```
## [1] '1.0.5'
```

In a script, you can make sure a package is loaded with *at least* some version:


```r
packageVersion("dplyr") > "0.8.1"
```

```
## [1] TRUE
```

Which you can use like,


```r
if(packageVersion("dplyr") < "0.8.1"){
  stop("First update the dplyr package!")
}
```



## All packages: loaded or installed

To list all loaded packages, do:


```r
pck <- .packages()
pck
```

```
##  [1] "knitr"     "magrittr"  "tibble"    "stats"     "graphics"  "grDevices"
##  [7] "utils"     "datasets"  "methods"   "base"
```

And to list all *installed* packages,


```r
.packages(TRUE)
```



## Avoiding conflicts {#avoidconflicts}

To make sure you use a function from the intended package, you can use the `::` operator. 


```r
out <- dplyr::select(mtcars, cyl, disp)
```

In this example, we avoid a common problem with `select` and other functions in packages with very general names.

When writing packages and other production-level code, the use of `::` is strongly recommended. In practice in writing dependable scripts, I recommend using `::` where practical, and less for functions that have obviously unique names in the context.

Finally, one way to avoid conflicts is to use as few packages as possible, as described in a later section.

\BeginKnitrBlock{rmdreading}<div class="rmdreading">When loading the `tidyverse` package, a collection of R packages by the same group of developers, we are presented with a list of conflicts with other installed packages. Try it out.</div>\EndKnitrBlock{rmdreading}



## The search path and conflicts {#searchpath}

After you load a package with `library()`, it is added to the *search path*. This is an order of environments where R will look for the value of some object - often an R function.

Use `search` to inspect the current search path:


```r
search()
```

```
##  [1] ".GlobalEnv"        "package:knitr"     "package:magrittr" 
##  [4] "package:tibble"    "package:stats"     "package:graphics" 
##  [7] "package:grDevices" "package:utils"     "package:datasets" 
## [10] "package:methods"   "Autoloads"         "package:base"
```

When you type a function in R, like `summary()`, where will R find the definition of that function? In the search path above, it **starts at the first entry**, the global environment (containing the objects you see in the Environment pane in Rstudio). The search continues until a function is found with the name `summary`, and then R stops searching.

If you have **conflicts**, where the name of a function is defined in more than one package, the order in which you loaded the packages will affect from which package the function will actually be used. However, do not use this fact as a way to manage package dependencies, instead read the next section carefully.

To find the actual **location of the package files** on your system, use the following command (this might be helpful to archive all actual package files used in a project).


```r
searchpaths()
```

```
##  [1] ".GlobalEnv"                                  
##  [2] "C:/RLIBRARY/knitr"                           
##  [3] "C:/RLIBRARY/magrittr"                        
##  [4] "C:/RLIBRARY/tibble"                          
##  [5] "C:/Program Files/R/R-4.0.5/library/stats"    
##  [6] "C:/Program Files/R/R-4.0.5/library/graphics" 
##  [7] "C:/Program Files/R/R-4.0.5/library/grDevices"
##  [8] "C:/Program Files/R/R-4.0.5/library/utils"    
##  [9] "C:/Program Files/R/R-4.0.5/library/datasets" 
## [10] "C:/Program Files/R/R-4.0.5/library/methods"  
## [11] "Autoloads"                                   
## [12] "C:/PROGRA~1/R/R-40~1.5/library/base"
```




## Bundling package dependencies with `renv`


If you want to be able to run your project a long time after you created it, one of the packages may have changed - causing an error or unexpected result. Although this is comparatively rare, it is still a possibility you want to avoid.

If we transport the script to another location, how can we be sure that the code will be working? Sometimes, newer versions of packages break old code, and in some cases packages are no longer hosted on CRAN after some time (though this is rare).

Another problem arises because all your R projects normally share the same library of R packages on your machine. That means that if you install a newer version in one project, it might cause code to stop working *in another project*. 

The new `renv` package by Rstudio is at the time of writing the best solution to creating a project that is truly portable, since we can transport the packages alongside with the code.

\BeginKnitrBlock{rmdtry}<div class="rmdtry">Please read the documentation and installation instructions here: https://rstudio.github.io/renv/.</div>\EndKnitrBlock{rmdtry}

In practice, we basically just run this command once in a new (or existing) project:


```r
renv::init()
```

\BeginKnitrBlock{rmdcaution}<div class="rmdcaution">The above command can take >10min to execute.</div>\EndKnitrBlock{rmdcaution}

All package dependencies are downloaded and kept in a special `renv` subfolder (the original source code for all the packages).

After initializing `renv` in a directory, you can keep using `install.packages` and `library` as usual, the packages are now installed in your local `renv` folder, and in a global **cache** which `renv` manages for you.

If you are using, say package version 1.2, and version 1.3 is installed on your machine from a different project, the `renv` project will still use 1.2, unless we update this package locally ourselves (simply with `install.packages`).



## Minimizing package dependencies

Although a solution like `renv` can help us to make scripts reproducable, in practice this rule of thumb is a valuable lesson:

**Use as few external packages in your code as possible**

Complicated projects tend to break easily, and often external R packages are to blame. 

- Don't use packages when you only use one simple function from it. In that case, copy the function to your own code base, and `source` it in a script.

- Try to use packages that are still in active development, have a significant user- (or fan-) base, such as the `tidyverse` packages, and are popular.

These warnings should not shy you away from exploring the great universe of >15000 R packages that are now publicly available!









