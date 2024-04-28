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
**Source: https://www.methodenberatung.uzh.ch/de/datenanalyse_spss/unterschiede/zentral/friedman.html**

The Friedman Test for dependent samples tests whether the central tendencies of several dependent samples differ. The Friedman Test is used when the assumptions for an analysis of variance are not met.

The term "dependent samples" is used when a measurement value in one sample and a certain measurement value in another sample influence each other. This is the case in three situations:

1. **Repeated Measurements**: The measurement values come from the same person, for example, when measuring before a treatment, during a treatment, and after a treatment, or when different treatments are applied to the same person and are to be compared.
2. **Natural Pairs**: The measurement values come from different people who are somehow related (e.g., wife – husband – couple therapist, psychologist – patient, lawyer – client, or twins).
3. **Matching**: The measurement values come from different people who have been assigned to each other, for example, based on a comparable value on a third variable (which is not the focus of the investigation).


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


### Conduct the test Friedman Test in Gretl



