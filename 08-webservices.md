# R on the web {#webservices}


## Introduction

This short chapter will show how to interact with, and deploy web services from R. An API ('Application Programming Interface') is a way to access data or services on a server, for example to download or upload data. The most common architecture for APIs is known as RESTful Web Services (or 'REST API' for short). We will use the `httr` package to download data from REST APIs.

We start with downloading data from, and querying remotely stored databases, and include two working examples: a database (updated every 15min) in a MongoDB database, and a single table stored in a PostgreSQL database.


**Packages used in this chapter**

- By now standard packages, `lgrdata`, `dplyr`, `lubridate`
- `plumber` for launching web services.
- `DBI` to interface with databases.
- `dbplyr` to make this interface even easier.
- `purrr` (for `flatten_dfr`, though the package offers much more).




## Reading data from remote databases


### Accessing databases with the DBI package {#dbi}

Here, we briefly show how to interact with SQL databases. There are many R packages available, partly because there are many implementations of SQL databases (including postgres, which we show here). A unified interface is provided by the `DBI` package ([link to documentation](https://db.rstudio.com/dbi/)), which allows us to use nearly the same code for very different databases.

Here, we only show how to read rectangular data with the `DBI` and `dbplyr` packages.

With `dbConnect`, we setup a connection to the remotely available database (which does not download data yet). Using the connection we can list the tables available, download a table, and many other things.


```r
# General database interface
library(DBI)

# Drivers for postgres.
library(RPostgreSQL)

# For generating SQL queries.
library(dplyr)
library(dbplyr)

# Set up connection.
# !! This is a working connection (6/5/2019)
con <- dbConnect(drv = PostgreSQL(),
                 user = "mjvgsdzg",
                 password = "RsIxTmV9a95Gbb6Vo7IgG9wZNiv4Hq5L",
                 host = "balarama.db.elephantsql.com",
                 port = 5432,
                 dbname = "mjvgsdzg")

# List available tables (only one in this case)
dbListTables(con)

# Makes a direct connection to the table, but does
# not download anything yet:
cars <- tbl(con, "automobiles")

# We can download the entire table with collect,
# but note that in practice we rarely do this as databases may be extremely large.
cars_data <- collect(cars)
  
# We can now make queries using dplyr syntax, to collect only subsets of data.
# for example, find all cars with weight over 2000kg.
heavycars_query <- cars %>%
  filter(weight > 2000)

# What does the query look like in SQL?
show_query(heavycars_query)

# Now send the query to the database, collect the data.
heavycars <- collect(heavycars_query)


# Many other functions from dplyr or other can be used on databases,
# but not everything! Some trial and error may be necessary.
# Find all cars that have 'buick' in the name, using grepl:
buick_query <- filter(cars,
                grepl("buick", car_name)
               )
```



\BeginKnitrBlock{rmdtry}<div class="rmdtry">Run both examples above - getting data from a (continuously updated_ MongoDB database, and a simple static PostreSQL database. Try applying some `dplyr` filters and summaries on the data.</div>\EndKnitrBlock{rmdtry}




### NoSQL

In this example we show how to read data from MongoDB - a popular NoSQL database. With the `mongolite` package it is very straightforward to read data that can be transformed to a dataframe. Here, every item in the database is a piece of JSON, but of equal length and with the same names, so that each item can become a row in a dataframe.

For a remotely stored MongoDB database, you first have to construct the URL, which includes a username and password. The URL looks like,

```
"mongodb://username:password@hostingsite.com:portnr/databasename"
```

The next example is a working example of a database stored on mlab.com, and grabs data using the `mongolite` package.

The database contains data on number of cars parked on 13 car parks in Almere, the Netherlands. The data are updated every 15 minutes.


```r
# A working URL (5/6/2019)
parkingdata_url <- "mongodb://guest:123password123@ds229186.mlab.com:29186/almereparking"

#
library(mongolite)

# Make a database connection. Here you don't download data yet, just
# set up a connection.
db <- mongo(collection = "almereparkingjson",
            url = parkingdata_url)

# We can now download the last 1000 rows of the data.
parkingdata <- db$find(limit = 1000, sort = '{"updated": -1}')
```

If you simply want to download all data, just do `db$find()`, it takes less than a minute.

\BeginKnitrBlock{rmdreading}<div class="rmdreading">
The above example shows how to make a query with the `mongolite` package, you can read [more options in the documentation](https://jeroen.github.io/mongolite/query-data.html).
</div>\EndKnitrBlock{rmdreading}




## Accessing an API

A REST API is simply presented as a URL, to which a specific service can be accessed, and even parameters passed (when necessary). A URL may simply look like:

```
https://someserver.com/getdata
```

or may be more complicated, with queries, authentication, and so on. Let's look a simple example, using the `httr` package:


```r
library(httr)

# The first step is to reach out to the server, and get a response.
# Here, g is a complicated object that we do not inspect directly.
g <- GET("https://opendata.cbs.nl/ODataApi/OData/37789ksz/TypedDataSet")

# Important: if the server could be reached, you have permission, 
# and the service is available, you should see 200 as the response.
# Other HTML codes include 404 (service not found).
status_code(g)
```

```
## [1] 200
```

```r
# We can read the actual data with the `content` function:
dat <- content(g)

# In this case, dat has two components - the interesting one is 'value'
uitkering_data_list <- dat$value
```

Unfortunately the result is a nested list, with NULL elements in some. To make this into a dataframe, we want to flatten each list, each of these will become a row in our dataframe, and bind all the rows together. This is such a common problem that I will show a robust solution:


```r
library(dplyr) # bind_rows
library(purrr) # flatten_dfr

uitkering_data <- lapply(uitkering_data_list, flatten_dfr) %>%
  bind_rows
```

We now have data from the Dutch government on social benefits, a dataset that gets updated frequently. The use of the API directly rather than downloading the data manually is clearly a much better option.

As we saw in this example, the challenge is often not so much getting *something* from the web service, but often we have substantial work to shape it into a useful object in R (usually: a dataframe).


\BeginKnitrBlock{rmdtry}<div class="rmdtry">Try downloading data from this URL (containing Dutch demography data since 1898).
  
https://opendata.cbs.nl/ODataApi/odata/37556/TypedDataSet</div>\EndKnitrBlock{rmdtry}




## Deploying an API with `plumber`

With `plumber` we can quickly set up an API, that we can use from within any other programming language, web site, or program. Normally you would deploy the API on a server running R, so that the R service can be used from anywhere. Here, to keep things simple, we deploy the API on your own machine ('locally'), but it will work similarly to when it runs on a remote server.

The first step is to create a standalone script (with extension .R) that defines as many *functions* as you like. Each function can take arguments. Take the following example, paste it in a script, call it `plumber.R`.


```r
# plumber.R

#* Echo back the input, in uppercase
#* @param msg The message to echo
#* @get /echo
function(msg=""){
  list(msg = paste0("The message is: '", toupper(msg), "'"))
}

#* Plot a histogram
#* @png
#* @get /plot
function(){
  rand <- rnorm(100)
  hist(rand)
}

#* Return the sum of two numbers
#* @param a The first number to add
#* @param b The second number to add
#* @get /sum
function(a, b){
  as.numeric(a) + as.numeric(b)
}
```

The first function sends back an uppercase version of a message, the second function plots a histogram, and the third function adds two numbers. Obviously, you can write much, much more complicated R functions that can be subsequently called using the API. These functions are chosen because they return different things: text, a plot, and a number. Let's see how they work in practice.

With our API 'endpoints' (the three functions) defined, we can start the API via:


```r
library(plumber)

# There are no other arguments besides a directory to find the file.
p <- plumb("plumber.R")

# Deploy the API.
p$run(port=8001)
```



Your R process will now be busy (until you close it), with a message like:

```
Starting server to listen on port 8001
```

This will also open up a `swagger` window, which is a simple interface to our own Web service. Here you can 'Try out' the various end points.

\BeginKnitrBlock{rmdtry}<div class="rmdtry">Try out the web service you just created as follows: open up a browser, and try each of the three endpoints:

`127.0.0.1:8001/echo?msg="make me uppercase!"`
`127.0.0.1:8001/plot"`
`127.0.0.1:8001/sum?a=2&b=3"`

Now go back to the script above, and notice how each endpoint is defined (`echo`, `plot`, and `sum`)</div>\EndKnitrBlock{rmdtry}


\BeginKnitrBlock{rmdreading}<div class="rmdreading">
The above example comes straight from the `plumber` documentation, which you can read here: https://www.rplumber.io/

More complicated examples are available as well.</div>\EndKnitrBlock{rmdreading}


<!-- #### Dynamic routes -->

<!-- I am not going to repeat any of the introductory stuff of [the excellent plumber documentation](), just highlight some key points. We first make a file with the API definitions, using some special formatting. Suppose the file 'item_predictor_api_definition.R' contains the following: -->


<!-- ```{r} -->
<!-- #* Predict item lifetime -->
<!-- #* @param id -->
<!-- #* @get /prediction -->
<!-- function(item){ -->

<!--   my_prediction_function(item) -->

<!-- } -->
<!-- ``` -->

<!-- Here we define a single API endpoint - but not quite in the format that we wish to have, because the request to this endpoint will have to look like this: -->

<!-- ``` -->
<!-- <server ip address>:8001/prediction?item="nr123" -->
<!-- ``` -->

<!-- I suppose this is OK too, but a common recommendation is to allow a URL like `/api/item/nr123/prediction`. The key `nr123` (referring to some item ID) is therefore the dynamic bit of the URL. We can easily tell `plumber` what to do: -->

<!-- ```{r} -->
<!-- #* Predict item lifetime -->
<!-- #* @param id -->
<!-- #* @get /api/item/<item>/prediction -->
<!-- function(item){ -->

<!--   my_prediction_function(item) -->

<!-- } -->

<!-- ``` -->

<!-- Note how the Roxygen field calls the parameter `id`, not `item` as you would expect. I found it necessary in plumber version 0.4.5 to **not** use the same name as the function argument there - otherwise the service crashes with a weird message ([see this issue I openened](https://github.com/trestletech/plumber/issues/267)). -->



