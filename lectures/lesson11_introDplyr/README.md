lesson 11: Data manipulation
================

# Data analysis

`dplyr` is a package within the `tidyverse` for data manipulation:
transforming, selecting, grouping and summarizing variables. The
philosphy of `dplyr` is to offer a more intuitive syntax than base R.
Also, it makes easy to translate R/dplyr code into SQL queries, which
means we can use dplyr to query remote sources such as a MySQL database.

## The pipe operator

To use the tidyverse more efficiently, the `tidyverse` introduces an
operator that is not originally available in base R, but allows to
*chain* functions together, rather than *nesting* them. This operator
`%>%` is called the pipe. What the `%>%` does, is to pass the value that
preceeds `%>%` to the first position of the subsequent function (or
alternatively anywhere you place a `.`). For instance:

``` r
round(mean(c(1:10, NA), na.rm = T), 0)
```

Can be written as

``` r
c(1:10, NA) %>% 
  mean(na.rm = T) %>% 
  round(0)
```

    ## [1] 6

Which is equivalent to

``` r
c(1:10, NA) %>% 
  mean(x = . , na.rm = T) %>% #the dot is a pronoun for what precedes the pipe operator
  round(x = ., digits = 0)
```

## dplyr verbs

dplyr has seven fundamental verbs for data manipulation.

To manipulate rows:

  - `filter()`: select rows
  - `distinct()`: select unique set of rows
  - `arrange()`: sort rows

To manipulate columns:

  - `select()`: select/drop columns
  - `mutate()`: add columns

To group:

  - `group_by()`: group by a variable(s)
  - `count()`: sum grouping by a variable(s)
  - `summarise()`: run a function producing 1 output for each grouping
    level

To manipulate dataframes:

  - `*_join()`: merge multiple dataframe together

## Manipulating rows

### Filter

`filter()` retrieves all rows that evaluate to `TRUE`:

``` r
#to retain all Spanish males:
selfiesCasualties %>% 
  filter(country == 'Spain' & gender == 'Male')
#to retain only casualties in 2014 and 2015
selfiesCasualties %>% 
  filter(between(year, 2014, 2015))
```

1.  Using `titanicDt`, filter the count of female children who did not
    survive

2.  Using `beerDt`, filter the records from 1903 to 1997 (use the
    shortcut `dplyr::between()`)

3.  Using `untidyReview`, filter all records with missing values on
    `reviewLocation`. Hint: you need to use `is.na()` to test what
    elements are missing.

4.  Using `selfiesCasualties`, filter all records for victims who dies
    abroad (`country` different than `nationality`)

To filter positionally, use `slice()` or
`filter(between(row_number(), 1, 10))`. For instance, to filter the
first 3 rows from `titanicDt`

``` r
slice(titanicDt, 1:3) 
titanicDt %>% filter(between(row_number(), 1, 3))
```

### Distinct

`distinct` retains unique rows from the input table grouping by the
variables passed to `...`. For instance, the combination of unique
country-gender combinations:

``` r
selfiesCasualties %>% 
  distinct(country, gender)
```

By default, `distinct` retains only the columns of the grouping
variables. Use `.keep_all = FALSE` to retain all columns:

1.  Using `selfiesCasualties`, select distinct levels for `class` and
    keep all variables

2.  Using `titanicDt`, create a 2 columns dataset for the unique
    combinations of `Age` and `Sex`

### Arrange

`arrange` sorts the rows by the grouping variables you pass to the
function:

``` r
selfiesCasualties %>% 
  arrange(class)
```

1.  Using `titanicDt`, arrange by Class, Age, Sex

2.  Using `titanicDt`, arrange descending by `n` (check the examples for
    `desc()`)

## Manipulate Columns

### Select and mutate

`select` keeps only the variables you pass (or it drops them if you use
`-`). For example to select only `class` and `country`:

``` r
selfiesCasualties %>% 
  select(class, country)
```

To select multiple columns, you can use `class:age` (from variable class
to variable gender), or any from the `?select_helpers`.

1.  Using `sn_revenue`, select only variables for company names and year
    2015.

2.  Using `untidyReview`, select only the variables containing review
    ratings.

`mutate()` adds a new variable. For example, we can add the birthdate of
the victims by subtracting their age from the year of the accident:

``` r
selfiesCasualties %>% 
  mutate(birthDate = year - as.numeric(age)) %>% 
  slice(1:2)
```

1.  In `beerDt`, convert the consumption from gallons per capita to
    liters (1 gallon 3.78 liters)

2.  In `socialNetwork_users` add a percent sign to the variable percent
    (use `paste0()`)

3.  In `untidyReview`, turn `reviewStayYear` into a integer

### Grouping

`group_by` adds grouping levels to a tibble. For instance, you can
combine it with `summarise` to count the number of female and male
victims by country:

``` r
selfiesCasualties %>% 
  group_by(country, gender) %>% 
  summarise(tot = n())
```

Note that every time you use `summarise()`, your tibble drops one
grouping level. For instance in the latter example, data are still
grouped by country, and any further dplyr verb would apply groupwise
unless we `ungroup()` first.

1.  Using `titanicDt`, calculate the average casualties by class (note
    that the dataset has both survivors/non-survivors counts)

<!-- end list -->

``` r
titanicDt %>%
 filter(Survived == "No") %>%
 group_by(Class) %>%
 summarise(tot = mean(n))

titanicDt %>%
 group_by(Class, Survived = "No") %>%
 dplyr::summarise(average = base::mean(n))
```

2.  Using `titanicDt`, calculate the average casualties by gender (note
    that the dataset has both survivors/non-survivors counts)

3.  Using `beerDt`, calculate the average consumption before and after
    your birthyear (hint: you may create a new variable)

4.  Using `titanicDt`, calulcate the count of casualties by nationality,
    then arrange descending.

`n()` is the count operator in dplyr (it works only within a
`summarise()`, `mutate()` or `filter()`), and counts the rows by group
level without taking any argument. Because counting observations is a
very common task, dplyr offers the wrapper `count`, which is equivalent
to `group_by %>% summarise( n() )` except that it `ungroup()` after.

``` r
selfiesCasualties %>% 
group_by(country, gender) %>% summarise(count = n())
selfiesCasualties %>% 
  count(country, gender)
```

Exercise: Use `selfiesCasualties` to calculate the percent of casualties
by class. One way, is to combine `groub_by()`, `sum()` and `n()`. Make
sure you understand the difference between `sum()` and `n()` and how
they behave when grouping.

## Manipulating dataframes

Functions from the `*_join` family serve to merge datasets. To join
different datasets, you need at least one variable to serve as a key for
joining. For instance, consider the case where you have two separates
dataset about the count of twitter and FB users during the same years
(Source: eMarketer):

``` r
twitter_users
fbMobile_users
```

You want to create a single dataset containing the records for both. You
can use `full_join` to bind together facebook and twitter users by year:

``` r
twitter_users %>% 
  full_join(fbMobile_users, by = c('Year' = 'Year'), suffix  = c('twitter', 'fb'))

twitter_users %>% 
  inner_join(fbMobile_users, by = 'Year', suffix  = c('twitter', 'fb'))
```

If you do not pass a character vector of variables to join, `*_join`
uses all the variables with common names across the two tables.

In the previous example `full_join`, `left_join`, or `right_join` give
the same result. However, each of them merge using different ruels. To
show how the differe, we add a row to each dataset:

``` r
fbMobile_users <- fbMobile_users %>% 
  add_row(Year = 2014, millions = 122) %>% arrange(Year)

twitter_users <- twitter_users %>% 
  add_row(Year = 2021, millions = 74.6) %>% arrange(Year)
```

By default, `full_join` creates an `NA` for the missing observation in
`fbMobile_users` for 2021:

``` r
fbMobile_users %>% 
  full_join(twitter_users, by = 'Year', suffix = c('fb', 'twitter'))
```

To keep only the years for those which appear in both datasets:

``` r
fbMobile_users %>% 
  inner_join(twitter_users, by = 'Year', suffix  = c('fb', 'twitter'))
```

To summarise:

``` r
#use ?left_join() ... to view the documentation
left_join(twitter_users, fbMobile_users, by = 'Year') #retain all records from A, add those from B that match ()
right_join(twitter_users, fbMobile_users, by = 'Year') #retain all records from B, add those from A that match
anti_join(twitter_users, fbMobile_users, by = 'Year') #retain all records from A with not matches in B (keep just columns from A)
semi_join(twitter_users, fbMobile_users, by = 'Year') #retain all records from A with matches in B (keep just columns from A)
```

# Exercise

``` r
#we need 2 datasets from ncyflights13 package:
if (!require(nycflights13)) install.packages('nycflights13')
```

1.  Use the appropriate join to add the column with full carrier names
    to `flights` using `airlines` :

2.  Count the number of flights for each carrier using the full carrier
    name. Then plot a bar chart of the counts.

3.  Calculate the average air\_time for the flights by each carrier.
    Plot the result in a barchart.

4.  What is the busiest route (i.e., origin-destination pair)?

5.  What’s the busiest day for each of the departing airports?
