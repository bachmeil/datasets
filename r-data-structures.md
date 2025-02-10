# R Data Structures

This handout provides you with a brief overview of R's data structures. You'll need to understand the basics in order to successfully use R this semester. I've included links to further reading if you want to go deeper.

# Atomic Data Types

Programming languages have *atomic data types*. The name just indicates that these types can't be broken down further. For our purposes this semester, the atomic types are `double` (the usual types of decimal numbers, such as 3.894), `integer`, logical (`TRUE` or `FALSE`), and character (more commonly referred to as strings in other languages, and taking values like "ECON 935").

# Vectors

A vector is a collection of elements of the same atomic data type. Of course, anyone in this course will already be familiar with the concept of a vector in mathematics. Most commonly a vector is used to hold the values of one variable, such as the wage for all individuals in the sample or the inflation rate over a period of time.

R vectors have most of the properties you'd expect. We can create a vector with three elements using this notation:

```
y <- c(4.8, 3.2, 7.2)
```

You can do mathematical operations on vectors:

```
y+2.5
y-2.5
y*2.5
y/2.5
```

Vectors are not necessarily numerical. If you run this code

```
y > 5
```

you get a logical vector:

```
[1] FALSE FALSE  TRUE
```

You can also have a character vector. An example would be the names of the variables in your dataset:

```
varnames <- c("gdp", "ffr", "u", "inf", "emp")
```

You can read more on vectors [in the official documentation](https://cran.r-project.org/doc/manuals/r-release/R-intro.html#Vectors-and-assignment).

# Scalars

Note that, for whatever reason, R does not actually have a scalar data type. This probably isn't important if you haven't programmed in other languages, but if you have, it may be confusing at first.

Rather than having a scalar data type, what appears to be a scalar is actually a vector with one element. This code creates a vector with a single element:

```
x <- 3.7
```

We can see that it's a vector and has only one element:

```
is.vector(x)
length(x)
```

# Arrays

Another data structure you'll have encountered in your math classes is the n-dimensional array. For the type of work that we'll be doing in this class, we'll mainly use arrays with two dimensions, which you're used to calling a matrix. A matrix in R is an array with two subscripts.

You can create a matrix like this:

```
m <- matrix(1:10, ncol=2)
```

R has a wide variety of matrix and linear algebra operations built into the language. I'll introduce them as needed. For further reading, you can check [the official documentation](https://cran.r-project.org/doc/manuals/r-release/R-intro.html#Matrix-facilities).

# Time Series

It should not surprise you that the most important data structure in this class is the time series. There are two types of time series. The first is a single time series variable

```
y.vector <- c(7.8, 1.1, 1.9, 6.4, 2.3)
y.ts <- ts(y.vector, start=c(2024,6), frequency=12)
```

In this example, `y.vector` is a vector with five elements. It could be time series or cross-sectional data. `y.ts` is a time series. It's a *compound data structure* because it holds multiple pieces of information inside one variable. In the case of a time series, it holds the data (`y.vector`) plus the frequency (monthly) and the dates it covers (starting in June 2024). The fact that it holds the extra data is very helpful. You can take lags and differences of `y.ts` and R will properly add the appropriate time series properties to those transformations. You can add two time series together and R will properly align the dates for you.

You'll get much practice with time series throughout the semester, but here's a motivating example:

```
x1 <- ts(1:12, start=c(2023,7), frequency=12)
x2 <- ts(13:36, start=c(2023,1), frequency=12)
z <- x2 - x1
frequency(z)
length(z)
start(z)
```

`x1` runs from July 2023 to June 2024 (12 observations). `x2` runs from January 2023 to December 2024 (24 observations). `z` has 12 observations, because those are the dates that match across `x1` and `x2`. `z` starts on the correct date of July 2023, it has the correct frequency (monthly), and it has the correct value of 18. This allows you to just use time series variables for this type of operation and never have to worry about aligning data, fixing the dates, etc. I wrote my dissertation in GAUSS. It was not an enjoyable experience having to track all of this stuff manually.

The other type of time series is the matrix time series. The only difference is that the matrix time series is a compound data structure holding a matrix of data plus time series properties. Each row of a matrix time series corresponds to a particular date.

One way to create a matrix time series is to use `cbind`:

```
xx <- cbind(x1, x2)

# xx has 24 rows
# cbind does not delete any data. It sets missing values as NA.
nrow(xx)

# Type is mts
class(xx)

# Type is ts
class(x1)
```

If you don't want missing values, you can use `ts.intersect`, which only includes rows for which all variables have an observation:

```
xs <- ts.intersect(x1, x2)

# 12 rows
nrow(xs)

# Also an mts object
class(xs)

# Starts in July 2023
start(xs)

# Monthly data
frequency(xs)

# Grab observations for 2024; takes last six rows
window(xs, start=c(2024,1))
```

# Lists

One reason R is different than other languages is because it is a dialect of Lisp, which dates back to the 1950s, and is the second oldest programming language after Fortran. The name was originally spelled LISP, which stands for "list processor". As you might guess, R being a dialect of Lisp means lists are an important part of R.[^1]

A list in R is a *heterogeneous data structure*. That means you can store as many different types of information as you want in a list. An example is the output of a linear regression. Among other things, you have the coefficient vector, the residuals (a vector, but with a longer length than the coefficient vector), the data used to do the regression (a matrix), and the regression formula (which is completely different from a vector or matrix). The list can hold all of these items and more, no matter how diverse they are. You can even store function definitions inside a list.

[^1]: A bit of trivia. Herbert Simon, who won the 1978 Nobel Prize in economics, won the 1975 Turing Award (the equivalent of the Nobel Prize for computer science) for creating the linked list.

The official documentation for lists [is found here](https://cran.r-project.org/doc/manuals/r-release/R-intro.html#Lists).

# Data Frames

The most common way to store data in R is the *data frame*, which is a list holding vectors of the same length or matrices with the same number of rows. Be forewarned that data frames are confusing to many new R users, at least those that have used other languages before using R. That's because Lisp and list data structures are different from the approach taken by those other programming languages.

If you are going to work with data frames in your own research, I encourage you to read up on them. The official documentation [is here](https://cran.r-project.org/doc/manuals/r-release/R-intro.html#Data-frames).

Having said that, you probably won't want to spend much time on data frames if you're doing time series anlysis. In general, we do not use data frames with time series data. The design of data frames, as lists with vectors of the same length, makes them work well for cross-sectional data, but makes them useless for time series data. Let's take a look at what I mean.

Suppose we have these two annual time series, one starting in 1980 and one in 1981:

```
y1 <- ts(1:12, start=1980)
y2 <- ts(13:24, start=1981)
```

`y1` and `y2` have the same length, but the first element of `y1` and `y2` correspond to different dates. Yet if you create a data frame with those two time series, look what happens:

```
y.df <- data.frame(y1, y2)
```

You can create the data frame because `data.frame` converts `y1` and `y2` to vectors with length 12, which amounts to stripping off their time series properties. This doesn't mean data frames are doing things incorrectly. It just means they weren't designed to be used with time series data.

There's a simple rule when working with time series: **use ts or mts objects, and forget about data frames**.

# When You Can't Avoid Data Frames

Unfortunately, you can't always avoid data frames. This is primarily an issue when you read in data. Whether you read data from a CSV file, an SQL database, or an Excel spreadsheet, the function will usually return a data frame.

Fortunately, this is not much of a problem. You just need to read in the data and then add time series properties. Here's an example where we read in the price of gas from a CSV file. The gasoline price data was downloaded from FRED. It has a header and two columns, the date (a string) in the first column and the national average gasoline price (a number) in the second column.

```
gas.raw <- read.csv("https://raw.githubusercontent.com/bachmeil/datasets/refs/heads/main/gas.csv", header=TRUE)
class(gas.raw)
```

`gas.raw` is a data frame. As discussed above, it's useless for our purposes, because it doesn't keep track of the time series properties. Create a new ts variable with the right time series properties:

```
gas <- ts(gas.raw[,2], start=c(1991,2), frequency=12)
class(gas)
plot(gas)
```

If you want a matrix, as in a plain matrix, with no time series properties, you can always convert a data frame to a matrix easily:

```
gas.matrix <- as.matrix(gas.raw)
```

The catch is that since the first column is a string, it'll convert everything to a string. This makes sense, because in your mathematics classes, you only had elements that were numbers. All the elements of a matrix have to be the same type. What you need to do is slice off just the numerical parts of the data frame:

```
gas.matrix <- as.matrix(gas.raw[,2])
```

By taking the second column only, we get a matrix of numbers. You can then do linear algebra operations or whatever you want:

```
t(gas.matrix) %*% gas.matrix
```