# Statistical Group Comparisons

In the following tutorial, we show how to conduct statistical group comparisons. We want to know whether two or more groups of a variable differ given the outcome. As an example, we analyze the number sold items of some retail shop before and after some treatment. We want to know whether the number of sold items differs before and after the treatment.

The number of sold items before the treatment is our Y, and the number of sold items after the treatment is our X. In total, we observe the number of sold items four months after treatment and collect the data every month. The data is taken from this website: https://www.methodenberatung.uzh.ch/de/datenanalyse_spss/unterschiede/zentral/friedman.html and can be found in this github repo as `Friedman.csv`.


Various tests exist for comparing the tendency of two or more groups. The choice of the test depends on the distribution of the outcome variable. The most common tests are:

**For mean comparisons** (outcomes are assumed to be normally distributed):
- t-test: for the comparison of two groups
    + Are the means of the two groups different?
- ANOVA: for the comparison of more than two groups
    + Are the means of g>2 groups different?

**For median comparisons** (outcomes are not assumed to be normally distributed):
- Wilcoxon rank-sum test (Wilcoxon-Mann-Whitey): for the comparison of two groups
    + Are the medians of the two groups different?
- Kruskal-Wallis test: for the comparison of more than two groups
    + Are the medians of g>2 groups different?



## Friedman Test
The Friedman test is a non-parametric statistical test developed by Milton Friedman. The test works for dependent samples, and tests whether the central tendencies of ordinal data -- that is data that can be ranked -- differ across multiple groups.

In the simplest case, the data matrix has the following structure:
- The rows represent the different **groups** (e.g., the different mother-father-child triads).
- The columns represent the different **treatments** (e.g., the different measurement points).

The different rows represent changes in a blocking factor B (*row factor*). The different columns represent changes in a factor A (*column factor*). The test computes the ranks within each block.

The gretl package we employ in the following, only allows for a single row factor B (e.g. different animals) but multiple column factors A (e.g. different treatments). Friedman's test assumes a model of the form:

x_ij = mu + alpha_i + beta_j + epsilon_ij

where:
- x_ij is the measurement value in the i-th row and j-th column,
- mu is the overall location parameter,
- beta_i is the effect of the i-th row,
- alpha_j is the effect of the j-th column, and
- epsilon_ij is the error term.

The test statistic is based on the ranks of the data within each level of B (here only a single one). It tests for a difference across levels of A (columns). Under the null hypothesis alpha_i = 0 and under the alternative alpha_i != 0 (not zero).
If the p-value is large, there is not sufficient evidence against null hypothesis that the column-sample medians are equal. If the p-value is small, there is sufficient evidence against the null hypothesis that the column-sample medians are equal, and at least one column-sample median is significantly different than the others; i.e., there is a main effect due to factor A.

Friedman's test makes the following assumptions about the data in X:

- All data come from populations having the same continuous distribution, apart from possibly different locations due to column and row effects.
- All observations are mutually independent.



### References
- https://en.wikipedia.org/wiki/Friedman_test
- https://www.mathworks.com/help/stats/friedman.html
- https://www.itl.nist.gov/div898/software/dataplot/refman1/auxillar/friedman.htm
- https://www.itl.nist.gov/div898/software/dataplot/refman1/auxillar/quade.htm
- https://www.methodenberatung.uzh.ch/de/datenanalyse_spss/unterschiede/zentral/friedman.html
- https://www.uni-koeln.de/~a0032/statistik/texte/mult-comp.pdf


### Examples of Possible Research Questions

- A new strength training program is introduced in an ice hockey team. The muscle mass of the players is measured before the training, immediately after, and three months later. How does the muscle development of the ice hockey players progress?
- In a study on aggression in daycare centers, mothers, fathers, and daycare staff are asked about the behavior of the children. Do the assessments differ?


### Prerequisites of the Friedman Test
- The dependent variable is at least ordinal-scaled.
- There are related samples (*verbundene Stichproben*), but the related "groups" of measurement values are independent of each other (e.g., the different mother-father-child triads are independent of each other).

### Let's have a look at the data
We print the first ten rows of both GDP per capita series, `gdp60` and `gdp85`, to get an overview of the data:

```Gretl
set verbose off
open Friedman.csv

# Collect series names in a list object
list L = Vortest Monat_1 Monat_2 Monat_3 Monat_4

# print the first ten rows of the list members
print L --byobs --range=1:10
```

The output shows the following:

~~~
        Vortest      Monat_1      Monat_2      Monat_3      Monat_4

 1          275          273          288          273          244
 2          292          283          284          285          329
 3          281          274          298          270          252
 4          284          275          271          272          258
 5          285          294          307          278          275
 6          283          279          301          276          279
 7          290          265          298          291          295
 8          294          277          295          290          271
 9          300          304          293          279          271
10          284          297          352          292          284
~~~

In total, we have 10 rows (also called *groups*) and 5 columns (called *treatments*).

The following commands prepare the data for plotting the changes in the number of sold items over time:

```Gretl
matrix mdata = {L}'  # store as matrix and transpose the matrix

gnuplot --matrix=mdata --time-series --with-lines --output=display \
  {set grid; set title "No. of sold items for each of 10 shops";\
   set ylabel "no. of sold items"; set xlabel "Treatment periods";}
```

<img src="https://github.com/atecon/gretl_tutorials/blob/main/statistical_group_comparison/figures/items_sold_per_shop.png" width="40%" />

The plot shows the number of sold items for each of the ten shops over the four treatment periods. The number of sold items varies between the shops and over the treatment periods. The Friedman Test is used to test whether the number of sold items differs over the treatment periods.

This following factorized boxplot shows the distribution of the number of sold items for each shop over the treatment periods. The figure can be created by means of the following commands:

```Gretl
matrix mdata = {L}  # collect columns of the list in a matrix

boxplot --matrix=mdata --output=display \
  {set grid; set title "Distribution of items sold per treatment";}
```

<img src="https://github.com/atecon/gretl_tutorials/blob/main/statistical_group_comparison/figures/boxplot_items_sold_per_treatment.png" width="40%" />

It looks like the tendency varies over the treatment periods. The Friedman Test is used to test whether the central tendencies of the dependent samples differ.


### Conduct Friedman Test in Gretl

For conducting the test in Gretl, we use the user-contributed package named `Friedman`. The package only allows for a single row factor B (e.g. different animals) but multiple column factors A (e.g. different treatments).

First, we need to install the package from the Gretl package server. The following command installs the package:

```Gretl
pkg install Friedman
```
This needs only be done once. Next, we load the package into memory:

```Gretl
include Friedman.gfn
```

You can read the help file of the package by running the following command:

```Gretl
help Friedman
```

Execute the Friedman Test by running the following command:

```Gretl
matrix results = Friedman(L)
print results
```

This command passes a list of series to the function `Friedman()`. The function returns a matrix with the test statistics and p-values.

The output of the test is:

~~~
Friedman test results with 10 blocks and 5 'treatments':
  Chi2(4) stat 13.2589, pvalue 0.0100777

results (4 x 1)

      13.259
    0.010078
      1.0000
      0.0000
~~~

We see that the p-value is 0.01, which is less than 0.05. Therefore, we reject the null hypothesis that the central tendencies of the dependent samples are constant over the treatment periods. The central tendencies of the dependent samples differ significantly, and hence, the treatment had some effect.

The Friedman test, however, cannot answer the question of which treatment period differs from the others. For this, we need to conduct a post-hoc test. Such a test is not implemented in Gretl so far.

## Conduct Quade Test in Gretl
The Quade test was proposed by Dana Quade in 1979 which can be used for interval scale data. It is a non-parametric test that is used to compare the central tendencies of more than two groups. The test is an extension of the Wilcoxon signed rank test and is equivalent to it when the treatments (columns) are two.

The Friedman package in Gretl also provides the Quade test. Here is how to conduct the test:

```Gretl
list L = Vortest Monat_1 Monat_2 Monat_3 Monat_4
matrix result_quade = Quade(L)
print result_quade
```

which returns the following output:

~~~
Quade test results with 10 blocks and 5 'treatments':
  F(4, 36), stat 2.35941, pvalue 0.0717163

result_quade (2 x 1)

      2.3594
    0.071716
~~~

The null hypothesis of the Quade test is that the central tendencies of the dependent samples are constant over the treatment periods. The p-value is 0.07, which is greater than 0.05. Therefore, we do not reject the null hypothesis that the central tendencies of the dependent samples are constant over the treatment periods. The Quade test does not provide evidence that the treatment had an effect.

### Note

The Quade test is based on the following assumptions:

- The rows are mutually independent. That means that the results within one block (row) do not affect the results within other blocks.
- The data can be meaningfully ranked.
- The data have at least interval scale so that the sample range may be determined within each block.

The Quade test is similar to the Friedman test. A few distinctions:
- For k = 2 columns, the Friedman test is equivalent to a sign test while the Quade test is equivalent to a signed rank test.
- According to Conover, the Quade test is typically more powerful for k < 5 while the Friedman test tends to become more powerful for k ≥ 5.
- The Friedman test only requires ordinal scale data (i.e., the data can be ranked) while the Quade test requires at least interval scale data (the range within a block can be computed).

