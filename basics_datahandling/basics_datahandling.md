# Gretl Crash Course - Basics on how to handle datasets

In this tutorial you'll learn how to:

1. Create and open datasets from a range of sources
2. Update and preprocess data in the dataset
3. Output your preprocessed data to csv

This tutorial replicates Nicholas Renotte's video tutoral using Python's Pandas library:

https://youtu.be/tRKeLrwfUgU?feature=shared

with the associated [github repository](https://github.com/nicknochnack/Pandasin20Minutes).

We use the same dataset in the tutorial as he does. The [csv-file can be found here](https://github.com/atecon/gretl_tutorials/blob/main/basics_datahandling/telco_churn.csv).

# Install and load a package
We will install a package named ["PandasPort"](https://github.com/atecon/PandasPort) which simplifies certain operations. Per default, this package ist not installed on your machine, so we neeed to download it from the Gretl package server by the following command (only once needed!):

```
pkg install PandasPort
```
Now, we can load the package into memory to make use of its features:
```
include PandasPort.gfn

# help PandasPort  # Read the help optionally
```


# Create/ load a dataset

## Create from a matrix

While a csv may comprise a column with string-valued series, a matrix can only have numerical values. Here is how to create a dataset with three rows and 2 variables through a matrix. Note that you need to create an empty dataset with the right number of rows using the [`nulldata`](https://gretl.sourceforge.net/gretl-help/cmdref.html#nulldata) command first:

~~~
nulldata 3
matrix m = {1, 44; 2, 55; 3, 66}
print m

list L = mat2list(m)  # transform a matrix to a list of data series
head(L)
~~~

which returns:

~~~
m (3 x 2)

   1   44
   2   55
   3   66

     L.column1    L.column2

1            1           44
2            2           55
3            3           66
~~~


## Load from a csv
Usually the data is stored as an Excel or CSV file, instead.

Just execute the [`open`](https://gretl.sourceforge.net/gretl-help/cmdref.html#open) command and pass the path + filename (various data formats are supported):

```
open "telco_churn.csv"
```

This command loads the data as a Gretl dataset and you get the following output:

~~~
variable 1 (State): non-numeric values = 3333 (100.00 percent)
variable 4 (Internationalplan): non-numeric values = 3333 (100.00 percent)
variable 5 (Voicemailplan): non-numeric values = 3333 (100.00 percent)
variable 20 (Churn): non-numeric values = 3325 (100.00 percent)
allocating string table
Listing 21 variables:
  0) const                   1) State                   2) Accountlength
  3) Areacode                4) Internationalplan       5) Voicemailplan
  6) Numbervmailmessages     7) Totaldayminutes         8) Totaldaycalls
  9) Totaldaycharge         10) Totaleveminutes        11) Totalevecalls
 12) Totalevecharge         13) Totalnightminutes      14) Totalnightcalls
 15) Totalnightcharge       16) Totalintlminutes       17) Totalintlcalls
 18) Totalintlcharge        19) Customerservicecalls   20) Churn
~~~


# Read

## Number of rows and columns

The number of rows of the currently active dataset can be easily retrieved through the `shape()` function from the PandasPort package:

~~~
print shape(dataset)
~~~

which returns 3333 and 20 since the dataset has 3333 rows and 20 columns.


## Show top 5 and bottom 5 rows

Printing the the initial rows of the whole dataset can be done by means of the `head()` function from PandasPort again.

~~~
head(dataset)
~~~

which returns

~~~
         State Accountlength     Areacode

1           KS           128          415
2           OH           107          415
3           NJ           137          415
4           OH            84          408
5           OK            75          415

  Internationalplan Voicemailplan Numbervmailmessages

1                No           Yes                  25
2                No           Yes                  26
3                No            No                   0
4               Yes            No                   0
5               Yes            No                   0
~~~

For printing the last 7 observations, use the `tail()` function:

```
tail(dataset, 7)
```


## Sort the dataset
By means of the [`dataset`](https://gretl.sourceforge.net/gretl-help/cmdref.html#dataset) command jointly with the `sortby` option, one can easily sort the dataset. If a list is given, the sort proceeds hierarchically: if the observations are tied in sort order with respect to the first key variable then the second key is used to break the tie, and so on until the tie is broken or the keys are exhausted.

Here is an example for sorting the dataset by `State` and `Areacode`:

~~~
list SortBy = State Areacode
dataset sortby SortBy

head(dataset)
~~~

which yields:

~~~
         State Accountlength     Areacode Internationalplan Voicemailplan

1           AK            36          408                No           Yes
2           AK           104          408                No            No
3           AK            78          408                No            No
4           AK           110          408                No            No
5           AK           127          408                No            No
~~~


## Set the number columns to print

Via the [`set datacols`](https://gretl.sourceforge.net/gretl-help/cmdref.html#set) command, the user can specify how many columns to print per data block:

~~~
set datacols 3
head(dataset)
~~~

Try it out.


## Show column names
To print the variables names, execute:

~~~
varlist
~~~

which lists the names of the existing series:

```
Listing 21 variables:
  0) const                   1) State                   2) Accountlength
  3) Areacode                4) Internationalplan       5) Voicemailplan
  6) Numbervmailmessages     7) Totaldayminutes         8) Totaldaycalls
  9) Totaldaycharge         10) Totaleveminutes        11) Totalevecalls
 12) Totalevecharge         13) Totalnightminutes      14) Totalnightcalls
 15) Totalnightcharge       16) Totalintlminutes       17) Totalintlcalls
 18) Totalintlcharge        19) Customerservicecalls   20) Churn
 ```

## Rename column names
To rename a series' name, use the [`rename`]((https://gretl.sourceforge.net/gretl-help/cmdref.html#rename)) command

Here we rename series `Totaldaycalls` to `NumberOfCalls`:

~~~
rename Totaldaycalls NumberOfCalls
~~~


## Summary statistics

The [`summary`](https://gretl.sourceforge.net/gretl-help/cmdref.html#summary) command offers a way for computing summary statistics. It supports various options. Here is a simple example:

~~~
summary --simple
~~~

which computes basic statistics for all series:

~~~
                         Mean     Median       S.D.        Min        Max
Accountlength           101.1      101.0      39.82      1.000      243.0
Areacode                437.2      415.0      42.37      408.0      510.0
Numbervmailmessa~       8.099      0.000      13.69      0.000      51.00
Totaldayminutes         179.8      179.4      54.42      0.000      350.8
Totaldaycalls           100.5      101.0      20.06      0.000      165.0
Totaldaycharge          30.56      30.50      9.256      0.000      59.64
Totaleveminutes         201.0      201.4      50.68      0.000      363.7
Totalevecalls           100.1      100.0      19.93      0.000      170.0
Totalevecharge          17.08      17.12      4.311      0.000      30.91
Totalnightminutes       200.9      201.2      50.57      23.20      395.0
Totalnightcalls         100.1      100.0      19.57      33.00      175.0
Totalnightcharge        9.039      9.050      2.276      1.040      17.77
Totalintlminutes        10.24      10.30      2.792      0.000      20.00
Totalintlcalls          4.479      4.000      2.461      0.000      20.00
Totalintlcharge         2.765      2.780     0.7541      0.000      5.400
Customerservicec~       1.563      1.000      1.316      0.000      9.000
~~~

Get rid of the `--simple` option, and see what you get.

## Filtering columns

Printing the values of a single series (column), just put the name into the `head()` function:

~~~
head(State)
~~~

returning

~~~
         State

1           KS
2           OH
3           NJ
4           OH
5           OK
~~~

For printing *k* columns, you can either pass their names sperated by space, or more elgantly, define a list (an object that can hold multiple series of a dataset) first:

~~~
list L = State Churn
head(L)
~~~

and you get

~~~
       L.State      L.Churn

1           AK        FALSE
2           AK        FALSE
3           AK        FALSE
4           AK        FALSE
5           AK        FALSE
~~~

## Get unique values of a series

In case your series is string-valued, as series `State` in our current dataset, you first need to retrieve the distinct string values by means of the [`strvals()`](https://gretl.sourceforge.net/gretl-help/funcref.html#strvals) function and store the result in an array of strings. This can then be printed:

~~~
strings StateNames = strvals(State)
print StateNames --range=1:
~~~

which returns (I show only the initial entries):

~~~
Array of strings, length 51
[1] "KS"
[2] "OH"
[3] "NJ"
[4] "OK"
[5] "AL"
[6] "MA"
~~~

For numeric data, one can either use the [`uniq()`](https://gretl.sourceforge.net/gretl-help/funcref.html#uniq) (values not sorted) or [`values()`](https://gretl.sourceforge.net/gretl-help/funcref.html#values) (values sorted) functions. Both ignore eventual missing values, though:

~~~
eval uniq(Churn)
~~~

returns

~~~
  1
  2
~~~

PandasPort's function `nuniq()`  reveals the number of distinct values. Here is an example for series `State` which returns 51:

~~~
eval nuniq(State)
~~~

## Count the number of observations for each distinct value
The function `value_counts()` from the PandasPort package, counts the occurrence of each unique value in a series or matrix (column vector). This corresponds to the frequency of each distinct observation.

Let's execute the following to print the frequencies for the initial 5 distinct values of series `State`:
~~~
eval head(value_counts(State))
~~~

which returns:

~~~
       State        count 
           1           70 
           2           78 
           3           68 
           4           60 
           5           80 
~~~


## Filtering rows

For sample restriction, you use the [`smpl`](https://gretl.sourceforge.net/gretl-help/cmdref.html#smpl) command in Gretl.

Let's say we what to filter the data based on a condition, that can be done easily by jointly with the `--restrict` option which restricts the whole dataset to meet the condition:

~~~
smpl Internationalplan == "No" --restrict
head(dataset)
~~~

returns:

~~~
         State Accountlength     Areacode Internationalplan Voicemailplan Numbervmailmessages

1           KS           128          415                No           Yes                  25
2           OH           107          415                No           Yes                  26
3           NJ           137          415                No            No                   0
4           MA           121          510                No           Yes                  24
5           LA           117          408                No            No                   0
~~~

Print the number of observations (rows) of the active dataset:

~~~
printf "No. of observations (rows) restricted data = %d\n", $nobs
~~~

which returns 3010 observations.

To reset the imposed restriction, simply run:

~~~
smpl full
~~~

Print again the number of observations (rows) of the now active dataset:

~~~
printf "No. of observations (rows) full dataset = %d\n", $nobs
~~~

which returns 3333.

One may also want to combine multiple conditions. Here we retrict the data to customers who were not on the international pland and did not churn:

~~~
smpl Internationalplan == "No" && Churn == "FALSE" --restrict
head(dataset)
~~~

and you get

~~~
         State Accountlength     Areacode Internationalplan Voicemailplan Numbervmailmessages

1           KS           128          415                No           Yes                  25
2           OH           107          415                No           Yes                  26
3           NJ           137          415                No            No                   0
4           LA           117          408                No            No                   0
5           RI            74          415                No            No                   0
~~~


## Indexing

This is mainly of interest for matrices, actually. To a certain degree, you can also do indexing for series.

The following command will show the 14-th entry of the series `State`:

~~~
print State[14]
~~~

which is

~~~
MT
~~~


## Get missing values per series
In order to print the number of missing values per series, we employ PandasPort's function `nmissing()`. In the example, we define a list of series for which we want that statistics before calling the function:

~~~
list L = Areacode Accountlength
eval nmissing(L)
~~~

The output is:

~~~
                 n missing 
     Areacode            0 
Accountlength            0
~~~


## Drop rows

Again we can make use of the [`smpl`](https://gretl.sourceforge.net/gretl-help/cmdref.html#smpl) command to remove missing values. Jointly with the `--no-missing` option, all rows are excluded for which at least one variable has a missing value.

~~~
eval shape(dataset)
smpl dataset --no-missing
eval shape(dataset)
~~~

and you get

~~~
  3307     20 
~~~
after having removed some rows.  In total, 26 rows have been removed due to missing values.

Adding the `--permanent` option to the `smpl` command, would actually delete those 26 rows permanently from your dataset in memory.


## Dropping columns

Deleting columns (series) can be done via the [`delete`](https://gretl.sourceforge.net/gretl-help/cmdref.html#delete) command:

~~~
delete Areacode
~~~

One can also pass a list of series to be dropped to the command.


## Creating calculated columns

Let's create new series named `new` which is the sum of two other series and print the result:

~~~
series new = Totalnightminutes + Totalintlminutes
list L = new Totalnightminutes Totalintlminutes
head(L)
~~~

This yields:

~~~
         L.new L.Totalnightminutes L.Totalintlminutes

1        254.7               244.7               10.0
2        268.1               254.4               13.7
3        174.8               162.6               12.2
4        203.5               196.9                6.6
5        197.0               186.9               10.1
~~~

## Updating an entire column

Suppose, we want to overwrite *all* values of series `new` now by 100. Simply run:

~~~
series new = 100
head(new)
~~~

and you get

~~~
           new

1          100
2          100
3          100
4          100
5          100
~~~

## Updating a single value

Suppose, you want to update only a single observation (say the 5th) of series `new` by 200:

~~~
new[5] = 200
head(new)
~~~

yields

~~~
             L

1          100
2          100
3          100
4          100
5          200
~~~

## Replace values
Suppose, we want to replace all values series `new` with 100 by 0 and all 200 by 1. We can apply the [`replace()`](https://gretl.sourceforge.net/gretl-help/funcref.html#remove) to do so:

~~~
matrix find = {100, 200}
matrix subst = {0, 1}
series new = replace(new, find, subst)
head(new)
~~~

which yields:

~~~
             L

1            0
2            0
3            0
4            0
5            1
~~~

## Condition based updating

The string valued series `Churn` consists of two distinct values: "FALSE" and "TRUE":

~~~
eval strvals(Churn)
~~~
returns
~~~
Array of strings, length 2
[1] "FALSE"
[2] "TRUE"
~~~

We want to create a new binary series which takes the value of zero `Churn == "FALSE"` and unit otherwise. For this, we apply the so called ternary operator:

~~~
series Churn_binary = Churn == "FALSE" ? 0 : 1
list L = Churn Churn_binary
head(L, 10)
~~~

which yields

~~~
        L.Churn L.Churn_binary

 1        FALSE              0
 2        FALSE              0
 3        FALSE              0
 4        FALSE              0
 5        FALSE              0
 6        FALSE              0
 7        FALSE              0
 8        FALSE              0
 9        FALSE              0
10         TRUE              1
~~~


# Delete/ Output

## Output to CSV

We storing a dataset, the [`store`](https://gretl.sourceforge.net/gretl-help/cmdref.html#store) command is used.

Among other formats, we can save the dataset as a csv file simply by:

~~~
store "output.csv"
~~~

The csv gets stored on your local machine in your working directory.

## Output to gdt
The gretl data format has the suffix `*.gdt`. It's actually an xml file comprising various metadata which csv does not ship. Simply execute:

~~~
store "output.gdt"
~~~

The gdt file gets stored on your local machine in your working directory.

## Clear the dataset

To clear the dataset from the memory, simply execute:

~~~
clear --dataset
~~~


**That's it for the moment!**
