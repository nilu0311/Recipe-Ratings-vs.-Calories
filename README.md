# Investigating the Relationship Between Recipe Rating and Calories
Cassidy Lu and Harshita Saha

<br/>

## Introduction


The advent of the internet has made it possible for instant access to a virtually unlimited amount of information within seconds. This has provided users with an opportunity to expand their knowledge in any domain, including those of food and recipes. As users search for recipes to try, and hope to find ones that they like, websites that publish recipes continue to grow and seek to develop recipes that users will rate highly. 

In this project, we are investigating a dataset of recipes and ratings from `food.com`. The question we are trying to answer in this project is: **Is there a relationship between a recipe being rated 5 stars on average, and the calories of that recipe?** Specifically, we want to check if the difference in calories of recipes that were rated 5 stars on average, and those that were not, can be attributed to random chance.

The answer to this question can provide recipe publishers with an additional parameter to consider when developing recipes, and can also be used to further hypothesize whether such a relationship, if it exists, is limited to correlation or if it extends to causation, incentivizing websites to recommend certain recipes more often, based on their calories. 

The raw datasets for this project are a subset of the data scraped for this https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf paper, and solely contain recipes and reviews from `food.com` beginning 2008. The two downloaded datasets, `raw_recipes` and `raw_interactions`, contain 83,782 and 73,1927 rows respectively. 

The columns from each dataset that are relevant to our analyses are as below:


`raw_recipes`:
- `id`: Unique id of the recipe.
- `minutes`: Estimated time to prepare the recipe.
- `nutrition`: Nutritional information of the recipe, the first number in it is the calories of the recipe.
- `n_steps`: Number of steps in the recipe.
- `n_ingredients`: Number of ingredients in the recipe.
- `description`: String description of the recipe.

`raw_interactions`
- `user_id`: Unique id of the user who left the rating / comment on the recipe.
- `recipe_id`: Unique id of the recipe.
- `rating`: The rating the user left on the recipe (0 if they did not leave one).


After downloading these datasets, we wanted to extract the average rating for each unique recipe, In order to do this, we merged the `raw_recipe` and `raw_interactions` datasets together on recipe id, merging left on the recipe ids in `raw_recipe`, since not all recipes may have had any user interactions, and we want a DataFrame containing relevant information from `raw_recipe` with one row per recipe. We then filled all `0` ratings with `np.nan` since from analysis of `food.com`, it appeared that users cannot actually leave ratings of 0, but can leave a review without any ratings, meaning that rating values were 0 when no ratings were left. 

Using this merged dataset, we calculated the average ratings of each recipe, and then added these values as the 'avg_rating' column, to each recipe they corresponded to, by merging on the `raw_recipes` dataset. It is this dataset, `raw_merged`, that we will clean and then use for our analysis for the rest of the report. 


---
<br/>

## Cleaning and EDA

### Data Cleaning:

<br/>

We first converted column data types from the original DataFrame as we saw fit:
- Converted `id` to a string, which was originally stored as an int.
- Converted 'nutrition', originally stored as a string representation of a list, to a list.
- We similarly converted types of other columns that were not used in our analysis.

We then replaced any invalid values in `raw_merged` with `np.nan`, which included:
- Negative values of `minutes`, `n_steps`, and `n_ingredients`.
- Empty `tags`, `nutrition`, `steps`, and `ingredients` lists.
- Strings of `description` that did not contain any alphanumeric characters, which we interpreted as being an invalid description. 

We extracted the first value from the `nutrition` list, which represents the calories of each recipe, and added a new numerical column `calories` to the DataFrame. We removed rows where the calories were above the 99th percentile, since plotting this column revealed extreme outliers that appeared unreasonable, and we also made sure to check that all values in this column were non-negative. We then added a boolean column, `is_5`, set to True if the `avg_rating` value of that recipe was 5, and False otherwise. Finally, we dropped columns irrelevant to our analysis.

The DataFrame resulting from this cleaning, which we will refer to as `clean_merged`, contains 82,944 rows, and will be used for the rest of our analysis.

|     id | description                                                                                                                                                                                                                                                                                                                                                                       |   minutes |   n_steps |   n_ingredients |   calories |   avg_rating | is_5   |
|-------:|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------:|----------:|----------------:|-----------:|-------------:|:-------|
| 333281 | these are the most; chocolatey, moist, rich, dense, fudgy, delicious brownies that you'll ever make.....sereiously! there's no doubt that these will be your fav brownies ever for you can add things to them or make them plain.....either way they're pure heaven!                                                                                                              |        40 |        10 |               9 |      138.4 |            4 | False  |
| 453467 | this is the recipe that we use at my school cafeteria for chocolate chip cookies. they must be the best chocolate chip cookies i have ever had! if you don't have margarine or don't like it, then just use butter (softened) instead.                                                                                                                                            |        45 |        12 |              11 |      595.1 |            5 | True   |
| 306168 | since there are already 411 recipes for broccoli casserole posted to "zaar" ,i decided to call this one  #412 broccoli casserole.i don't think there are any like this one in the database. i based this one on the famous "green bean casserole" from campbell's soup. but i think mine is better since i don't like cream of mushroom soup.submitted to "zaar" on may 28th,2008 |        40 |         6 |               9 |      194.8 |            5 | True   |
| 286009 | why a millionaire pound cake?  because it's super rich!  this scrumptious cake is the pride of an elderly belle from jackson, mississippi.  the recipe comes from "the glory of southern cooking" by james villas.                                                                                                                                                                |       120 |         7 |               7 |      878.3 |            5 | True   |
| 475785 | ready, set, cook! special edition contest entry: a mediterranean flavor inspired meatloaf dish. featuring: simply potatoes - shredded hash browns, egg, bacon, spinach, red bell pepper, and goat cheese.                                                                                                                                                                         |        90 |        17 |              13 |      267   |            5 | True   |


<br/>

### Univariate Analysis:

<br/>

For the univariate analysis of the `clean_merged` DataFrame, we first extracted the `avg_rating`, `calories`, and `n_steps` columns, and generated plots as below:

<iframe src="assets/Univariate_plt1.html" width=800 height=600 frameBorder=0></iframe>


This histogram shows the distribution of the `avg_rating` column in our cleaned dataset, and we can see that slightly over 75% of  recipes have average ratings that fall in the bin [4.5, 5.5), meaning that most recipes had a rating from 4.5 to 5. On analysis of the dataset, of the 82,944 recipes, 60,340 were rated at least 4.5, and 47,276 were rated 5, on average.

<iframe src="assets/Univariate_plt2.html" width=800 height=600 frameBorder=0></iframe>

This histogram shows the distribution of the `calories` column in our cleaned dataset, and we can see that most recipes are under 1,100 calories, with approximately 64% having calories in the range [100, 500). On analysis of the dataset, of the 82,944 recipes, 19,290 had at least 500 calories, and 3,041 had at least 1,100 calories.

<br/>

### Bivariate Analysis:

<br/>

For the bivariate analysis of the `clean_merged` DataFrame, we first extracted the `avg_rating`, `calories`, `n_steps`, and `n_ingredients` columns, and generated plots as below:

<iframe src="assets/Bivariate_plt1.html" width=800 height=600 frameBorder=0></iframe>

This scatterplot shows the average rating based on the number of calories in a recipe, and there does not appear to be a pattern such that a higher number of calories correlates to a higher or lower average rating. 

We can also see that most recipes are under 1,000 calories and most have an average rating greater than 4, and analysis of the dataset shows that of the 82,944 recipes, 266 have over 1,000 calories and a rating less than 4.  

<iframe src="assets/Bivariate_plt2.html" width=800 height=600 frameBorder=0></iframe>

This scatterplot shows the number of steps based on the number of ingredients in a recipe, and there does not appear to be a pattern such that a higher number of ingredients correlates to a higher or lower number of steps, but we see that recipes with a number of ingredients in the range [10, 20] are more likely to have over 60 steps. 

We can also see that most recipes have at most 25 ingredients and 40 steps, and analysis of the dataset shows that of the 82,944 recipes, 9 have over 25 ingredients and 40 steps. 


<br/>

### Interesting Aggregates:

<br/>


| n_ingredients range   |   n_steps |
| (0.968, 4.2]          |   6.03895 |
| (4.2, 7.4]            |   7.83826 |
| (7.4, 10.6]           |   9.93157 |
| (10.6, 13.8]          |  11.8703  |
| (13.8, 17.0]          |  14.3151  |
| (17.0, 20.2]          |  17.225   |
| (20.2, 23.4]          |  19.1429  |
| (23.4, 26.6]          |  20.6111  |
| (26.6, 29.8]          |  23.9792  |
| (29.8, 33.0]          |  22.913   |



This grouped table shows the average number of steps based on the range that the number of ingredients of a recipe falls in, and there appears to be a correlation such that recipes with a number of ingredients that fall in a higher bin generally have a greater number of steps on average. We can see that that is not always consistent, as in bin (29.8, 33.0], and that the average number of steps of recipes with a number of ingredients in the range (26.6, 29.8] is the highest, at almost 24 steps. 
 

<br/>


| avg_rating range   |   calories |
| (0.996, 1.4]       |    381.308 |
| (1.4, 1.8]         |    391.865 |
| (1.8, 2.2]         |    399.412 |
| (2.2, 2.6]         |    326.286 |
| (2.6, 3.0]         |    389.212 |
| (3.0, 3.4]         |    381.695 |
| (3.4, 3.8]         |    397.611 |
| (3.8, 4.2]         |    392.149 |
| (4.2, 4.6]         |    383.457 |
| (4.6, 5.0]         |    383.374 |



<br/>


This grouped table shows the average number of calories based on the range that the average rating of a recipe falls in, and there appears to be no correlation that would allow us to say that recipes with an average rating that fall in a higher bin have a greater or lesser number of calories on average. We can also see that recipes with an average rating in the bin (1.8, 2.2] have the maximum of around 399 calories on average, while bin (2.2, 2.6] has the minimum of around 326 calories on average.

| n_steps_range   |   (0.968, 4.2] |   (4.2, 7.4] |   (7.4, 10.6] |   (10.6, 13.8] |   (13.8, 17.0] |   (17.0, 20.2] |   (20.2, 23.4] |   (23.4, 26.6] |   (26.6, 29.8] |   (29.8, 33.0] |
| (0.901, 10.9]   |        276.354 |      320.298 |       361.951 |        392.138 |        430.997 |        452.631 |        537.127 |        452.805 |        865.217 |        361.317 |
| (10.9, 20.8]    |        344.189 |      382.13  |       412.293 |        451.501 |        496.685 |        570.917 |        579.733 |        700.842 |        664.119 |        820.633 |
| (20.8, 30.7]    |        522.252 |      427.802 |       450.002 |        498.475 |        586.654 |        663.496 |        665.892 |        666.707 |        750.157 |        750.17  |
| (30.7, 40.6]    |        538.807 |      440.715 |       406.838 |        540.47  |        603.2   |        695.371 |        662.76  |        909.662 |        752.371 |       1210.9   |
| (40.6, 50.5]    |        337.364 |      540.222 |       555.521 |        517.41  |        539.306 |        672.257 |        929.242 |        790.26  |        502.4   |        531.2   |
| (50.5, 60.4]    |        111.667 |      332.933 |       402.6   |        702.033 |        662.973 |        693.825 |        600.15  |        nan     |        815     |        nan     |
| (60.4, 70.3]    |        431.8   |      793.1   |       141.8   |        314.975 |        515.45  |       1092.67  |        nan     |        nan     |        nan     |        594.3   |
| (70.3, 80.2]    |        nan     |      336.9   |       nan     |        nan     |        497.875 |        689.1   |       1527.3   |        414     |        491.7   |        nan     |
| (80.2, 90.1]    |        970.3   |      143     |       221.25  |        nan     |        282.3   |        786     |        nan     |       2309.5   |        nan     |        nan     |
| (90.1, 100.0]   |        nan     |      nan     |       nan     |        nan     |        nan     |       1289.2   |        nan     |        nan     |        nan     |        nan     |

This pivot table uses the combination of ranges that the number of ingredients and the number of steps of a recipe falls in, and shows the average calories of all recipes that fall in that intersection. For recipes in a specific bin of the number of steps, the average number of calories does not appear to show a consistent pattern as the number of ingredients increases, and similarly there does not appear to be a pattern for average calories of recipes in a specific bin of the number of ingredients as the number of steps increases. 

---

<br/>

## Assessment of Missingness

<br/>

### NMAR Analysis

<br/>

We believe that the `description` column is NMAR, because the column contains `np.nan` when the `description` was originally `np.nan` or invalid, which we interpreted as being a `description` that did not contain any alphanumeric values in the value string. The missing `description` values depend on the missing value itself, because a `description` is missing when the individual who created the recipe either did not enter a `description` at all, or they entered an invalid (entirely non-alphanumeric) `description`. Additionally we identified this column as being NMAR because the missing `description` did not depend on any other column in our cleaned DataFrame. 

<br/>

### Missingness Dependency

<br/>

In this section we will focus on analyzing the missingness in the `avg_rating` column. Specifically we want to determine whether or not the data in this column is MAR or MCAR. To approach this question, we will look at the distribution of values in other columns when values in   `avg_rating` are missing and are not missing.If these distributions look the same regardless of the missingness of the `avg_rating`, we conclude that missingness in the `avg_rating` column does not depend on that column, and that the missingness in the `avg_rating` column depends on that column otherwise. If we find a single column that the missingness of `avg_rating` depends on, we conclude that `avg_rating` is MAR, and not MCAR.

In addition, the choice of statistic to check the missingness of `avg_rating` by another column depends on a few key pieces of information. Because the values we will be using in this step are all numerical / quantitative, we can either use the difference in group means of the Kolmogorov-Smirnov statistic. If the distributions of a column by missingness of `avg_rating` look like shifted versions of the same shape, we use the difference in group means, and if the distributions look like different shapes but are centered at the same location we use the  Kolmogorov-Smirnov statistic.

<br/>

> #### **Missingness of Average Rating by Calories:**

First we will look at the missingness of `avg_rating` by the `calories` column. The absolute difference in the means and medians of the `calories` column by missingness of `avg_rating` are 45.77 and 29.50 respectively.

In the overlaid density plots of the distributions below, we can see that they have the same basic shape.


<iframe src="assets/cal_density.html" width=800 height=600 frameBorder=0></iframe>

In the overlaid histograms of the distributions below, we can see that the centers are shifted.

<iframe src="assets/cal_bar.html" width=800 height=600 frameBorder=0></iframe>


Because the distributions of the values in the `calories` column by missingness of 'avg_rating' appear to be shifted versions of the same basic shape, we use the absolute difference in group means as our test statistic. 

The result of conducting a permutation test using absolute difference in means as the test statistic yielded a p-value of 0.00. With 0.05 as our significance level, this p-value signifies that there is likely a relationship between the missingness of `avg_rating`  and the values in the  `calories` column, and therefore we can conclude that 'avg_rating' is MAR, and not MCAR.

<br/>

> #### **Missingness of Average Rating by Number of Ingredients**
 
Now we will look at the missingness of `avg_rating` by the `n_ingredients` column. The absolute difference in the means and medians of the 'n_ingredients' column by missingness of `avg_rating` are 0.26 and 0.0 respectively.

In the overlaid density plots of the distributions below, we can see that they do not have the same basic shape.

<iframe src="assets/ing_bar.html" width=800 height=600 frameBorder=0></iframe>

In the overlaid histograms of the distributions below, we can see that the centers are not shifted.

<iframe src="assets/ing_density.html" width=800 height=600 frameBorder=0></iframe>

Because the distributions of the values in the `n_ingredients` column by missingness of `avg_rating` appear to have different shapes, but are similarly centered, we use the  Kolmogorov-Smirnov statistic as our test statistic. 

The result of conducting a permutation test using the Kolmogorov-Smirnov statistic as the test statistic yielded a p-value of 0.10. With 0.05 as our significance level, this p-value signifies that there is likely not a relationship between the missingness of `avg_rating` and the values in the  `n_ingredients` column.

---

<br/>

## Hypothesis Testing: 

In this section we will investigate if there is a relationship between a recipe being rated 5 stars on average, and the calories of that recipe. 
The plot below shows the distribution of the `calories` column based on whether or not a recipe was rated 5 stars on average. We can see that the distributions have the same basic shape and appear to be centered similarly.

<iframe src="assets/calorie_ht.html" width=800 height=600 frameBorder=0></iframe>


In order to answer our question, we decided to check if our observed difference in mean calories of recipes that were rated 5 stars on average, and those that were not, which came out to be around -0.0097, can be attributed to random chance. 
Toward this end, we set up a hypothesis test as below:

**Null Hypothesis**: In the population, calories of 5 star rating recipes and non-5 star rating recipes have the same distribution, and the observed differences in our samples are due to random chance.

**Alternative Hypothesis**: In the population, 5 star rating recipes have more calories than non-5 star rating recipes, on average. The observed difference in our samples cannot be explained by random chance alone.

Since our null hypothesis states that an average rating of 5 stars has no relationship to calories, we will conduct a permutation test to check if calories for each category, recipes with and without a rating of 5 stars on average, come from different distributions.

Upon conducting a permutation test, using the difference in group means for calories of recipes that were rated 5 on average and those that were not as our test statistic, we calculate a p-value of 0.503. With a significance level of 0.05, we fail to reject our null hypothesis. Therefore, we fail to reject that the calories of 5 star average rating recipes and non-5 star average rating recipes have the same distribution.


The empirical distribution of the test statistics has been visualized below, as well as the observed statistic.  

<iframe src="assets/last.html" width=800 height=600 frameBorder=0></iframe>

