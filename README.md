# Investigating the Relationship Between Recipe Rating and Calories

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

## Cleaning and EDA

<br/>

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

This scatterplot shows the average rating based on the number of calories in a recipe, and there …

We can also see that most recipes are under 1,000 calories and most have an average rating greater than 4, and analysis of the dataset shows that of the 82,944 recipes, 266 have over 1,000 calories and a rating less than 4. 

<iframe src="assets/Bivariate_plt2.html" width=800 height=600 frameBorder=0></iframe>

This scatterplot shows the number of steps based on the number of ingredients in a recipe, and there does not appear to be a pattern such that a higher number of ingredients correlates to a higher or lower number of steps. We can also see that most recipes have at most 25 ingredients and 40 steps, and analysis of the dataset shows that of the 82,944 recipes, 9 have over 25 ingredients and 40 steps.

<br/>

### Interesting Aggregates:

<br/>

This grouped table shows the average number of steps based on the number of ingredients in the recipe, and it does appear to have a correlation between those two variables. Number of ingredients are putted into 10 bins. By calculating the average steps base on each number of ingredients bins, it appears that with more ingredients, the number of the steps increases. 

<!-- <br/>

**Average Number of Steps based on the Number of Ingredients** -->

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


---
## Assessment of Missingness

---
## Hypothesis Testing

