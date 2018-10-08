Tidy data and joins
================
Sukey
October 4, 2018

-   [Data Reshaping Prompts ( and relationship to aggregation )](#data-reshaping-prompts-and-relationship-to-aggregation)
    -   [Activity\#1 cheatsheet](#activity1-cheatsheet)
        -   [Introduction](#introduction)
        -   [Load data](#load-data)
        -   [gather](#gather)
        -   [spread](#spread)
        -   [unite](#unite)
        -   [separate](#separate)
        -   [Simple completion of missing values](#simple-completion-of-missing-values)
-   [Join Prompts (join, merge, look up)](#join-prompts-join-merge-look-up)
    -   [Activity \#1 join join join](#activity-1-join-join-join)
        -   [left\_join](#left_join)
        -   [right\_join](#right_join)
        -   [inner\_join](#inner_join)
        -   [full\_join](#full_join)
        -   [semi\_join](#semi_join)
        -   [anti\_join](#anti_join)

Data Reshaping Prompts ( and relationship to aggregation )
==========================================================

**Problem: You have data in one “shape” but you wish it were in another. Usually this is because the alternative shape is superior for presenting a table, making a figure, or doing aggregation and statistical analysis.**

**Solution: Reshape your data. For simple reshaping, gather( ) and spread( ) from tidyr will suffice. Do the thing that is possible / easier now that your data has a new shape.**

Activity\#1 cheatsheet
----------------------

### Introduction

Tidyr mainly provides a function similar to the pivot table in Excel:

-   The gather and spread functions convert data between long and wide formats
-   The separate and union methods provide the function of data group splitting and merging, which is applied to the conversion of nominal data.

***R defines neat data as: the data for each variable is stored in its own column, and the data for each observation is stored in its own row. Neat data is the basis for data reprocessing.***

The tidyr package mainly involves:

-   Simple completion of missing values
-   Long table converts into wide table and wide table converts into long table
    -   gather- converts wide data to longer format. It is analogous to the melt function from reshape2
    -   spread - converts long data to wider format. It is analogous to the cast function from reshape2
    -   ***The opposite of gather( ) is spread( ), the former stacking different columns, the latter separating the same column***
-   Column splitting and column merging
    -   Separate - splits a column into multiple columns by separator
    -   Unite-concatenates multiple columns as specified and is a column

***Tidyr package: ( gather ( wide data to long data ), spread ( long data to wide data ), unit ( multiple columns into one column ), separate ( separate one column into multiple columns )***

### Load data

``` r
## Use the  GSSvocab dataset in the datasets package to demonstrate
library( tidyr )
```

    ## Warning: package 'tidyr' was built under R version 3.5.1

``` r
library( dplyr )
```

    ## Warning: package 'dplyr' was built under R version 3.5.1

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library( gridExtra )
```

    ## Warning: package 'gridExtra' was built under R version 3.5.1

    ## 
    ## Attaching package: 'gridExtra'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     combine

``` r
library(carData)


myt <- ttheme_default( 
         # Use hjust and x to center the text
         # Alternate the row fill colours
                 core = list( fg_params=list( hjust = 0.5 , x = 0.5 ),   # 1: right 0:left 0.5:center
                             bg_params=list( fill=c( "green", "pink" ) ) ),

         # Change column header to white text and red background
                 colhead = list( fg_params=list( col="white" ),
                                bg_params=list( fill="red" ) ) )

GSSvocab_distinct <-  GSSvocab %>% distinct()

grid.arrange( top = " First 10 rows of   GSSvocab_distinct",
             tableGrob( head(    GSSvocab_distinct,10  ),  ## look the first 10 rows
             theme=myt,
             rows=NULL
              ) )
```

![](Tidy_data_and_joins_files/figure-markdown_github/chunk%201%20original%20data-1.png)

![](https://github.com/STAT545-UBC-students/hw04-Sukeysun/blob/master/pictures/chunk%201%20original%20data-1.png)

### gather

Use the `gather( )` function to implement a wide table-to-long table with the following syntax:

**gather( data, key, value, ..., na.rm = FALSE, convert = FALSE ):**

-   data: the data that needs to be converted
-   key: assign all the columns in the original data frame to a new variable key
-   value: assigns all values in the original data frame to a new variable value
-   ...: can specify which columns are grouped together in the same column
-   na.rm: whether to delete missing values

``` r
## In addition to the gender column, the remaining columns are aggregated into two columns, named attribute and value.

 GSSvocabNew <-   GSSvocab_distinct %>% 
  gather( attribute, value, -gender )
```

    ## Warning: attributes are not identical across measure variables;
    ## they will be dropped

``` r
grid.arrange( top = " head and tail of  GSSvocabNew ( gather all columns except gender ) ",
             tableGrob( head(   GSSvocabNew,10  ),
             theme=myt,
             rows=NULL ),
             tableGrob( tail(   GSSvocabNew,10  ),
             theme=myt,
             rows=NULL ),
             nrow = 1 )   ## set these two tables side by side
```

![](Tidy_data_and_joins_files/figure-markdown_github/chunk%202%20gather1-1.png)

![](https://github.com/STAT545-UBC-students/hw04-Sukeysun/blob/master/pictures/chunk%202%20gather1-1.png)

The tables above give has the head and tail of our new data frame--GSSvocabNew. We can see that all colnames except gender in GSSvocab\_distinct are gathered in "attributte" and the value assigned to them are gathered in "value".

**A good point about tidyr is that we can gather only a number of columns while the other columns remain the same**

``` r
## If we want gather all the columns between ageGroup and age while keeping the other columns unchanged

 GSSvocabNew1 <-   GSSvocab_distinct %>% 
  gather( attribute, value, ageGroup:age )
```

    ## Warning: attributes are not identical across measure variables;
    ## they will be dropped

``` r
grid.arrange( top = " head and tail of  GSSvocabNew1 ( gather columns between ageGroup and age )",
             tableGrob( head(   GSSvocabNew1,10 ),
             theme=myt,
             rows=NULL ),
             tableGrob( tail(   GSSvocabNew1,10  ),
             theme=myt,
             rows=NULL ),
             nrow = 1 )
```

![](Tidy_data_and_joins_files/figure-markdown_github/chunk%203%20gather2-1.png)

![](https://github.com/STAT545-UBC-students/hw04-Sukeysun/blob/master/pictures/chunk%203%20gather2-1.png)

``` r
## if we dont want to gather adjacent column
 GSSvocabNew2 <-   GSSvocab_distinct %>% 
  gather(  attribute, value,
          `year`,`nativeBorn`,
         -educ )
```

    ## Warning: attributes are not identical across measure variables;
    ## they will be dropped

``` r
grid.arrange( top = " head and tail of  GSSvocabNew2 ( gather year and nativeBorn )",
             tableGrob( head(   GSSvocabNew2,5  ),
             theme=myt,
             rows=NULL ),
             tableGrob( tail(   GSSvocabNew2,5  ),
             theme=myt,
             rows=NULL ),
             nrow = 2 )
```

![](Tidy_data_and_joins_files/figure-markdown_github/chunk%204%20gather3-1.png)

![](https://github.com/STAT545-UBC-students/hw04-Sukeysun/blob/master/pictures/chunk%204%20gather3-1.png)

### spread

Sometimes, in order to meet the requirements of modeling or drawing, it is often necessary to convert a long table into a wide table or a wide table into a long table. How to implement the conversion of these two data table types. Use the spread( ) function to implement a long table widening table with the following syntax:

**spread( data, key, value, fill = NA, convert = FALSE, drop = TRUE ):**

-   data: is a long table that needs to be converted
-   key: a variable that needs to expand the value of the variable into a field
-   value: the value to be scattered
-   fill: For missing values, assign the value of fill to the missing value after transformation

``` r
## if we want to conver a long table to a wide one
 GSSvocabSpread <-  GSSvocabNew %>% 
## to avoid duplicate identifiers
  group_by( attribute ) %>% 
  mutate(ind = row_number()) %>% 
  spread(  attribute, value  )

GSSvocabSpread$ind <- NULL  ## remove ind

grid.arrange( top = " comparison of  GSSvocab_distinct and  GSSvocabSpread",
             tableGrob( head(   GSSvocab_distinct,5  ),
             theme=myt,
             rows=NULL ),
      
             tableGrob( head(   GSSvocabSpread,5  ),
             theme=myt,
             rows=NULL ),
             nrow = 2
             )
```

![](Tidy_data_and_joins_files/figure-markdown_github/chunk%205%20spread-1.png)

![](https://github.com/STAT545-UBC-students/hw04-Sukeysun/blob/master/pictures/chunk%205%20spread-1.png)

When I simply used spread(attribure, value), I got Error: Duplicate identifiers for rows. Then I found method [here](https://stackoverflow.com/questions/39053451/using-spread-with-duplicate-identifiers-for-rows). **However, I dont know why some values have been converted into character**

``` r
## check the difference between  GSSvocab_add and  GSSvocabSpread
## convert character to numeric in order to be compareable to GSSvocab_distinct
GSSvocabSpreadnew <- transform( GSSvocabSpread, 
                                vocab = as.numeric( vocab ), 
                                age = as.numeric( age ),
                                educ = as.numeric( educ ) )

setdiff(  GSSvocab_distinct, GSSvocabSpreadnew )
```

    ## Warning: Column `ageGroup` joining character vector and factor, coercing
    ## into character vector

    ## Warning: Column `educGroup` joining character vector and factor, coercing
    ## into character vector

    ## Warning: Column `nativeBorn` joining character vector and factor, coercing
    ## into character vector

    ## Warning: Column `year` joining character vector and factor, coercing into
    ## character vector

    ## [1] gender     age        ageGroup   educ       educGroup  nativeBorn
    ## [7] vocab      year      
    ## <0 rows> (or 0-length row.names)

It returns 0 rows, that is to say the values in GSSvocab\_add and GSSvocabSpread are matching. And the only difference between the two table above is only the sequence of column.

### unite

The effect of unite( ) is to combine multiple columns into one column.The syntax is as follows:

**unite( data, col, ..., sep = "-", remove = TRUE )**

-   data: is the data frame
-   col: the new column name to be combined
-   ...: specify which columns need to be combined
-   sep: the connector between the combined columns, the default is underlined
-   remove: whether to delete the combined column

``` r
## create a new dataframe
set.seed( 1 )
date <- as.Date( '2018-10-04' ) + 0:14
hour <- sample( 1:24, 15 )   ## random select 15 numbers between 1 and 24
min <- sample( 1:60, 15 )
second <- sample( 1:60, 15 )
event <- sample( letters, 15 ) ## random select 15 letters
data <- tibble( date, hour, min, second, event ) 

grid.arrange( top = "create a new data",
             tableGrob( head(  data,10  ),
             theme=myt,
             rows=NULL ) )
```

![](Tidy_data_and_joins_files/figure-markdown_github/chunk%207%20create%20a%20new%20data-1.png)

![](https://github.com/STAT545-UBC-students/hw04-Sukeysun/blob/master/pictures/chunk%207%20create%20a%20new%20data-1.png)

``` r
# Combine date, hour, min and second columns into new column datetime
dataNew <- data %>%
  unite( datehour, date, hour, sep = '  ' ) %>%  ## be careful: there are two spaces
  unite( datetime, datehour, min, second, sep = ':' )

grid.arrange( top = "unite data",
             tableGrob( head(  dataNew, 10  ),
             theme=myt,
             rows=NULL ) )
```

![](Tidy_data_and_joins_files/figure-markdown_github/chunk%208%20unite-1.png)

![](https://github.com/STAT545-UBC-students/hw04-Sukeysun/blob/master/pictures/chunk%208%20unite-1.png)

### separate

separate( ) function splits a column into multiple columns, which can be used for splitting log data or datetime data. The syntax is as follows:

**separate( data, col, into, sep = "\[^\[:alnum:\]\]+", remove = TRUE, convert = FALSE, extra = "warn", fill = "warn", ... ):**

-   data: is the data frame
-   col: the column that needs to be split
-   into: newly created column name, which is a string vector
-   sep: separator of the split column
-   remove: whether to delete the split column

``` r
# we can use the separate function to restore data to the time it was created.
# First, divide the datetime into a date column and a time column. 
# Then, divide the time column into hours, min, and second columns.

data1 <- dataNew %>%
  separate( datetime, 
           c( 'date', 'time' ), 
           sep = '  ' ) %>%  ## two sapces 
  separate( time, 
           c( 'hour', 'min', 'second' ), 
           sep = ':' )

grid.arrange( top = "separate data",
             tableGrob( head(  data1, 10  ),
             theme=myt,
             rows=NULL ) )
```

![](Tidy_data_and_joins_files/figure-markdown_github/chunk%209%20separate-1.png)

![](https://github.com/STAT545-UBC-students/hw04-Sukeysun/blob/master/pictures/chunk%209%20separate-1.png)

### Simple completion of missing values

Create an example of a data frame with missing values

``` r
## create data with missing value

x <- c( 1,2,7,8,NA,10,22,NA,15 )
y <-c( 'a',NA,'b',NA,'b','a','a','b','a' )
df <- tibble( x = x, y = y )

grid.arrange( top = " data with missing value",
             tableGrob( df,
             theme=myt,
             rows=NULL ) )
```

![](Tidy_data_and_joins_files/figure-markdown_github/chunk%2010%20data%20with%20missing%20value-1.png)

![](https://github.com/STAT545-UBC-students/hw04-Sukeysun/blob/master/pictures/chunk%2010%20data%20with%20missing%20value-1.png)

calculate the median and mean

``` r
( x_mean <- mean( df$x, na.rm = TRUE ) )
```

    ## [1] 9.285714

``` r
( x_median <- median( df$x, na.rm = TRUE ) )
```

    ## [1] 8

``` r
## which.max will find the location of value appears most often
( y_mode <- df$y[which.max (table( df$y ) ) ] ) 
```

    ## [1] "a"

Replace the missing values of x and y in data frame df

``` r
## replace missing value of x with x_mean and missing value of y with y_mode

df2 <- replace_na( data = df, replace = list( x = x_mean, y = y_mode ) ) 

grid.arrange( top = " data with replaced value",
             tableGrob( df2,
             theme=myt,
             rows=NULL ) )
```

![](Tidy_data_and_joins_files/figure-markdown_github/chunk%2011%20replace-1.png)

![](https://github.com/STAT545-UBC-students/hw04-Sukeysun/blob/master/pictures/chunk%2011%20replace-1.png)

The general steps for dealing with missing values:

1.  Identify missing values;
2.  Detecting the cause of the missing data;
3.  Delete observations with missing values or replace missing values with reasonable values.

The first step to identify missing values is very simple. we can use the is.na(), is.nan(), and is.infinite() functions to identify if there is a missing data set.
The second step is to understand the cause of the missing according to the actual scenario service, such as sensitive data causing users to fill in or network, machine faults leading to data faults, etc.;
The third step is to deal with the important steps of the missing, which can generally be handled by reasoning, row deletion and multiple interpolation.

***find missing value***

As mentioned above, we can use is.na(), is.nan(), and is.infinite() to identify if there is a missing value in the dataset, but this method returns whether all the vectors in the vector or data frame are missing values. Obviously, if the amount of data is very large, the result returned by this method is not easy to accept. In that situation, we can use the md.pattern() function in the mice package to discover patterns of missing values in the dataset. However, this method can only identify NA and NaN in R as missing values, but can not treat -Inf and Inf as missing values. In that case, we can replace these values with NA.

``` r
## create data without missing value
set.seed( 20 )
x1 <- runif( n = 10, min = 1, max = 1000 )
x2 <- 100*rnorm( n = 10 ) + 10
x3 <- rt( n = 10, df = 3 )
x4 <- rf( n = 10, df1 = 2, df2 = 3)
y <- 2*x1 - 0.3*x2 + 0.6*x3 - 1.2*x4 + rnorm( 10 )
nonemiss.df <- data.frame(y = y, x1 = x1, x2 = x2, x3 = x3, x4 = x4)


set.seed( 20 )
miss.df <- data.frame( y = y, x1 = x1, x2 = x2, x3 = x3, x4 = x4 )
miss.df[ sample( 1:nrow(miss.df ), 2 ),1 ] <- NA    ## random select two rows in x1 to be NA
miss.df[ sample( 1:nrow(miss.df ), 3 ),2 ] <- NA
miss.df[ sample( 1:nrow(miss.df ), 1 ),3 ] <- NA

grid.arrange( top = "data without missing value and data with missing value  ",
             tableGrob( nonemiss.df,
             theme=myt,
             rows=NULL ),
             tableGrob( miss.df,
             theme=myt,
             rows=NULL ) )
```

![](Tidy_data_and_joins_files/figure-markdown_github/chunk%2012%20create%20data%20without%20missing%20value%20and%20data%20with%20missing%20value-1.png)

![](https://github.com/STAT545-UBC-students/hw04-Sukeysun/blob/master/pictures/chunk%2012%20create%20data%20without%20missing%20value%20and%20data%20with%20missing%20value-1.png)

***use md.pattern( )***

``` r
library( mice )
```

    ## Warning: package 'mice' was built under R version 3.5.1

    ## Loading required package: lattice

    ## 
    ## Attaching package: 'mice'

    ## The following object is masked from 'package:tidyr':
    ## 
    ##     complete

    ## The following objects are masked from 'package:base':
    ## 
    ##     cbind, rbind

``` r
md.pattern( miss.df )
```

![](Tidy_data_and_joins_files/figure-markdown_github/chunk%2013%20visualize%20missing%20value-1.png)

    ##   x3 x4 x2 y x1  
    ## 4  1  1  1 1  1 0
    ## 3  1  1  1 1  0 1
    ## 2  1  1  1 0  1 1
    ## 1  1  1  0 1  1 1
    ##    0  0  1 2  3 6

![](https://github.com/STAT545-UBC-students/hw04-Sukeysun/blob/master/pictures/chunk%2013%20visualize%20missing%20value-1.png)

6 is the total number of missing values. x3 and x4 have no missing value while x2 has one missing values, y has two missing values and x1 has 3 missing values

***Missing data processing method*** **Reasoning**

The method fills in or restores missing values based on mathematical or logical relationships between variables, such as inferring possible values of missing values based on relationships between variables; inferring missing genders by name or inferring user likelihood based on purchased product characteristics The age group to which it belongs.

**Row deletion**

Any row with one or more missing values in the dataset will be deleted. Generally, the missing data is completely randomly generated, and the missing amount is only a small part of the dataset. we can consider using this method to process the missing values.

**Multiple interpolation**

This method is a method of processing missing values based on repeated simulations. It will generate a complete set of data from a data set containing missing values, all of which are replaced by the Monte Carlo method. There are many alternative methods, such as Bayesian linear regression, self-service linear regression, Logist regression and linear discriminant analysis.

``` r
## drop rows with NA values

delete.df <- miss.df %>% 
  drop_na()

grid.arrange( top = "process missing data--delete rows ",
             tableGrob( delete.df,
             theme=myt,
             rows=NULL ) )
```

![](Tidy_data_and_joins_files/figure-markdown_github/chunk%2014%20process%20missing%20data--delete%20rows-1.png)

![](https://github.com/STAT545-UBC-students/hw04-Sukeysun/blob/master/pictures/chunk%2014%20process%20missing%20data--delete%20rows-1.png)

Join Prompts (join, merge, look up)
===================================

**Problem: You have two data sources and you need info from both in one new data object.**
**Solution: Perform a join, which borrows terminology from the database world, specifically SQL.**

Activity \#1 join join join
---------------------------

In this part, I use country\_codes data set and gapminder data set. country\_codes is a dataframe of gapminder country names and ISO 3166-1 country codes

### left\_join

``` r
library( gapminder )
```

    ## Warning: package 'gapminder' was built under R version 3.5.1

``` r
left_cg <- country_codes %>% 
  left_join( gapminder, by = "country" )
```

    ## Warning: Column `country` joining character vector and factor, coercing
    ## into character vector

``` r
grid.arrange( top = "left join of country_codes and gapminder ",
             tableGrob( head( left_cg, 10 ),
             theme=myt,
             rows=NULL ) )
```

![](Tidy_data_and_joins_files/figure-markdown_github/chunk%2015%20left%20join%20of%20country_codes%20and%20gapminder-1.png)

![](https://github.com/STAT545-UBC-students/hw04-Sukeysun/blob/master/pictures/chunk%2015%20left%20join%20of%20country_codes%20and%20gapminder-1.png)

Now, we have a new data frame and the it has all columns appear in country\_codes or in gapminder. However, the row number of this new dataframe is 1749, which seems to have no relationship between the row number of gapminder (1704) and that of country\_codes which is 187. According to R doucment:
left\_join(x,y):
return all rows from x, and all columns from x and y. Rows in x with no match in y will have NA values in the new columns. If there are multiple matches between x and y, all combinations of the matches are returned.
let's check if there is NA values in the new columns

``` r
## find the subset of left_cg with country name not in gapminder
diff_cg <- subset(left_cg, !(country %in% gapminder$country))

grid.arrange( top = "check if there is NA values in the new columns ",
             tableGrob( head( diff_cg, 10 ),
             theme=myt,
             rows=NULL ) )
```

![](Tidy_data_and_joins_files/figure-markdown_github/chunk%2016%20check%20if%20there%20is%20NA%20values%20in%20the%20new%20columns-1.png)

![](https://github.com/STAT545-UBC-students/hw04-Sukeysun/blob/master/pictures/chunk%2016%20check%20if%20there%20is%20NA%20values%20in%20the%20new%20columns-1.png)

The table above is as we expected, it has NA values.

### right\_join

``` r
right_cg <- country_codes %>% 
  right_join( gapminder , by = "country")
```

    ## Warning: Column `country` joining character vector and factor, coercing
    ## into character vector

``` r
grid.arrange( top = "right join of gapminder and country_codes ",
             tableGrob( head( right_cg, 10 ),
             theme=myt,
             rows=NULL ) )
```

![](Tidy_data_and_joins_files/figure-markdown_github/chunk%2017%20right%20join%20of%20gapminder%20and%20country_codes-1.png)

![](https://github.com/STAT545-UBC-students/hw04-Sukeysun/blob/master/pictures/chunk%2017%20right%20join%20of%20gapminder%20and%20country_codes-1.png)

As shown in R document: right\_join(x,y) returns all rows from y, and all columns from x and y. Rows in y with no match in x will have NA values in the new columns. If there are multiple matches between x and y, all combinations of the matches are returned.

``` r
## find all unique country names in country_codes and gapminder
code_country <- unique( country_codes$country ) 
gapminder_country <- unique( gapminder$country )

## check if gapminder_country is a subset of code_country
all( gapminder_country %in% code_country) ## returns true
```

    ## [1] TRUE

``` r
## check which rows have NA values
which(is.na(right_cg)) # return nothing
```

    ## integer(0)

Since the all(gapminder\_country %in% code\_country) returns TRUE, we can make sure that the country name of gapminder is a subset of the country name in country\_codes. That is to say, all country names in gapminder have matching values in that of country\_codes. So there suppose to be no NA value in right\_cg, which can be proved by which(is.na(right\_cg)).

### inner\_join

``` r
inner_cg <- inner_join( country_codes, gapminder, by = "country")
```

    ## Warning: Column `country` joining character vector and factor, coercing
    ## into character vector

``` r
grid.arrange( top = "inner join of country_codes and gapminder ",
             tableGrob( head( inner_cg, 10 ),
             theme=myt,
             rows=NULL ) )
```

![](Tidy_data_and_joins_files/figure-markdown_github/chunk%2019%20inner_join%20of%20country_codes%20and%20gapminder-1.png)

![](https://github.com/STAT545-UBC-students/hw04-Sukeysun/blob/master/pictures/chunk%2019%20inner_join%20of%20country_codes%20and%20gapminder-1.png)

let's check if we lose some countries:

``` r
inner_country <- unique( inner_cg$country ) 

## check the difference between code_country and inner_country
setdiff( code_country, inner_country )
```

    ##  [1] "Armenia"               "Aruba"                
    ##  [3] "Azerbaijan"            "Bahamas"              
    ##  [5] "Barbados"              "Belarus"              
    ##  [7] "Belize"                "Bhutan"               
    ##  [9] "Brunei"                "Cape Verde"           
    ## [11] "Cyprus"                "Estonia"              
    ## [13] "Fiji"                  "French Guiana"        
    ## [15] "French Polynesia"      "Georgia"              
    ## [17] "Grenada"               "Guadeloupe"           
    ## [19] "Guyana"                "Kazakhstan"           
    ## [21] "Latvia"                "Lithuania"            
    ## [23] "Luxembourg"            "Macao, China"         
    ## [25] "Maldives"              "Malta"                
    ## [27] "Martinique"            "Micronesia, Fed. Sts."
    ## [29] "Moldova"               "Netherlands Antilles" 
    ## [31] "New Caledonia"         "Papua New Guinea"     
    ## [33] "Qatar"                 "Russia"               
    ## [35] "Samoa"                 "Solomon Islands"      
    ## [37] "Suriname"              "Tajikistan"           
    ## [39] "Timor-Leste"           "Tonga"                
    ## [41] "Turkmenistan"          "Ukraine"              
    ## [43] "United Arab Emirates"  "Uzbekistan"           
    ## [45] "Vanuatu"

We lose some rows from country\_codes, although these countries appear in country\_codes, they are not in gapminder. inner\_join( ) only returns infromation of countries which appear in country\_codes and gapminder

### full\_join

``` r
full_cg <- full_join( gapminder, country_codes)
```

    ## Joining, by = "country"

    ## Warning: Column `country` joining factor and character vector, coercing
    ## into character vector

``` r
grid.arrange( top = "full join of country_codes and gapminder ",
             tableGrob( head( full_cg, 10 ),
             theme=myt,
             rows=NULL ) )
```

![](Tidy_data_and_joins_files/figure-markdown_github/chunk%2021%20full%20join%20of%20gapminder%20and%20country_codes-1.png)

![](https://github.com/STAT545-UBC-students/hw04-Sukeysun/blob/master/pictures/chunk%2021%20full%20join%20of%20gapminder%20and%20country_codes-1.png)

full\_join(x,y) returns all rows and all columns from both x and y. Where there are not matching values, returns NA for the one missing. In the case where the country name of gapminder is a subset of the country name in country\_codes, the full\_cg should be same as left\_cg:

``` r
setdiff( full_cg, left_cg )
```

    ## # A tibble: 0 x 8
    ## # ... with 8 variables: country <chr>, iso_alpha <chr>, iso_num <int>,
    ## #   continent <fct>, year <int>, lifeExp <dbl>, pop <int>, gdpPercap <dbl>

As we ecpected, it returns 0 rows, which means full\_cg is same as left\_cg.

### semi\_join

``` r
sj_cg <- semi_join( country_codes, gapminder, by = "country")
```

    ## Warning: Column `country` joining character vector and factor, coercing
    ## into character vector

``` r
sj_gc <- semi_join( gapminder, country_codes, by = "country")
```

    ## Warning: Column `country` joining factor and character vector, coercing
    ## into character vector

``` r
grid.arrange( top = "semi_join of country_codes and gapminder ",
             tableGrob( head( sj_cg, 5 ),
             theme=myt,
             rows=NULL ) ,
             tableGrob( head( sj_gc, 5 ),
             theme=myt,
             rows=NULL )
             )
```

![](Tidy_data_and_joins_files/figure-markdown_github/chunk%2022%20semi_join%20of%20country_codes%20and%20gapminder-1.png)

![](https://github.com/STAT545-UBC-students/hw04-Sukeysun/blob/master/pictures/chunk%2022%20semi_join%20of%20country_codes%20and%20gapminder-1.png)

We can notice from these two table aboves, semi\_join(x,y) returns all rows from x which have matching values in y. sj\_cg is a dataframe that has matching values in column country of gapminder and sj\_gc is a dataframe which has all rows that have matching values in column country of country\_codes.

### anti\_join

``` r
anti_cg <- anti_join( country_codes, gapminder, by = "country" )
```

    ## Warning: Column `country` joining character vector and factor, coercing
    ## into character vector

``` r
grid.arrange( top = "anti join of country_codes and gapminder ",
             tableGrob( head( anti_cg, 10 ),
             theme=myt,
             rows=NULL ) )
```

![](Tidy_data_and_joins_files/figure-markdown_github/chunk%2023%20anti_join%20of%20country_codes%20and%20gapminder-1.png)

![](https://github.com/STAT545-UBC-students/hw04-Sukeysun/blob/master/pictures/chunk%2023%20anti_join%20of%20country_codes%20and%20gapminder-1.png)

anti\_join(x,y) returns all rows from x where there are not matching values in y, keeping just columns from x.
