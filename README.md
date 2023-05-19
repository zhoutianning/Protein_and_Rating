### Investigation on the relationship between the amount of proteins in a recipe and its average ratings

# Introduction to Dataset

Nowadays, fried chicken and hambergers are very popular among young people and we are interested in investigate **the relationship between the amount of proteins in a recipe and its average rating.**

This dataset contanins all the information about recipes and interations from users of the website. People can easily find high rating food recipes and the feedbacks to decide what to cook after work.

Our question helps to answer whether recipes with high protein tends to have high rating by most users' preferences, we can also find out if this website is suitable for vegetarians to use.

The dataset dataset contains recipes and ratings from [food.com](https://www.food.com/).
We have two datasets for this project:

1. RAW_recipes.csv (83782 rows), which contains the recipes and the following attributes.

   | Column           | Description                                                  |
   | ---------------- | ------------------------------------------------------------ |
   | 'name'           | Recipe name                                                  |
   | 'id'             | Recipe ID                                                    |
   | 'minutes'        | Minutes to prepare recipe                                    |
   | 'contributor_id' | User ID who submitted this recipe                            |
   | 'submitted'      | Date recipe was submitted                                    |
   | 'tags'           | Food.com tags for recipe                                     |
   | 'nutrition'      | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value”e |
   | 'n_steps'        | Number of steps in recipe                                    |
   | 'steps'          | Text for recipe steps, in order                              |
   | 'description'    | User-provided description                                    |
   | 'ingredients'    | The list of ingredients used                                 |
   | 'n_ingredients'  | The number of ingredients used                               |

2. RAW_interactions.csv (731927 rows), which contains reviews and ratings submitted for the recipes in RAW_recipes.csv.

   | Column      | Description         |
   | ----------- | ------------------- |
   | 'user_id'   | User ID             |
   | 'recipe_id' | Recipe ID           |
   | 'date'      | Date of interaction |
   | 'rating'    | Rating given        |
   | 'review'    | Review text         |

# Cleaning and EDA (Exploratory Data Analysis)

## The steps we took for data cleaning:

1. Left merge the recipes and interactions datasets together by recipe id. So we can have the 'rating' attribute in the dataframe, as later on we would investiage the ralationship of recipes' protein(PDV) and their average ratings.

   ```python
   # read in the raw datasets
   recipe = pd.read_csv('RAW_recipes.csv')
   ratings = pd.read_csv('RAW_interactions.csv')
   
   # Left merge the recipes and interactions datasets together.
   merged = recipe.merge(ratings, how='left', left_on='id', right_on='recipe_id')
   ```

2. In the merged dataset, we fill all ratings of 0 with np.nan. The reason is that some users just write reveiw texts without giving the rating. Our analysis would calculate the average rate of each recipe and mean is very sensitive to extreme values like 0. We replace them with null value and not take them account to the mean, so the mean rating would not be skewed low.

   ```python
   # In the merged dataset, fill all ratings of 0 with np.nan.
   merged['rating'] = merged['rating'].replace(0, np.NAN)
   ```

3. Find the average rating per recipe, as a Series. Meger this Series containing the average rating per recipe back to the recipes dataset. As there are more than one rating for each recipe and we would consider the average rating as our useful feature.

   ```python
   # Average rating per recipe, as a Series.
   average_rating_per_recipe = merged.groupby('id')['rating'].mean()
   
   # Add this Series containing the average rating per recipe back to the recipes dataset
   recipe_cleaned = recipe.copy()
   arpr_df = average_rating_per_recipe.to_frame()
   recipe_cleaned = recipe_cleaned.merge(arpr_df, how='left', left_on='id', right_index=True)
   ```

4. We only investigate the observations with less than 1440 minutes cooking time. We removed observations with minutes longer than 1440 minutes. Because these recipes are for making alcohols. However, in our study, we would like to focus on recipes of dishes that can be completed within a day.

   ```python
   recipe_cleaned = recipe_cleaned[recipe_cleaned['minutes'] <= 1440]
   ```

5. Creat a 'rate_category' column which match each rating to its corresponding category. Then we can plot boxplot of protein(PDV) for each rate category. We may find some trend or difference useful for our analysis.

   ```python
   # Define a helper function to categorize average rates into 5 different categories.
   def rate_category(rate):
       if 0 <= rate <= 1:
           return 1
       elif 1 < rate <= 2:
           return 2
       elif 2 < rate <= 3:
           return 3
       elif 3 < rate <= 4:
           return 4
       else:
           return 5
   
   # Add the 'rate_category' column to the dataframe.
   recipe_cleaned['rate_category'] = recipe_cleaned['rating'].apply(rate_category)
   ```

6. Extract the protein attribut from the nutrition column, since we mainly focus on the relationship between protein and average rating.

   ```python
   # Create a column of protein (PDV) from nutrition
   recipe_cleaned['protein'] = recipe_cleaned['nutrition'].apply(lambda x: x[1:-1]).str.split(',').apply(lambda x: x[4]).astype(float)
   ```

## Head of cleaned DataFrame: cleaned_recipe

<iframe src="assets/clean.png" width=1500 height=1100 frameBorder=0></iframe>

## Univariate Analysis

<iframe src="assets/protein_hist.html" width=500 height=400 frameBorder=0></iframe>

We plotted a histogram of the protein column to learn about the general distribution of protein PDV in the recipes. From the plot, we learned that the median of protein is 18 PDV. The 25-percentile and 75-percentile are 7 and 49 PDV, respectively. This means 50% of the recipes contain protein PDV within this range.

## **Bivariate Analysis (Protein(PDV) and Rating)**

<iframe src="assets/protein_and_rating.html" width=500 height=400 frameBorder=0></iframe>

We plotted a scatterplot of 'rating' and 'protein', the points with highest protein all have 5.0 rating and majority of the points are gathering over 500 protein and over 4.0 rating. From the scatterplot we cannot tell there is a perfect line of best fit which demonstrate theit relations, to futher investigate the relationship we need to do the systematic hypothesis testing.

## Interesting Aggregates

|               |  min |      mean |    max |
| ------------: | ---: | --------: | -----: |
| rate_category |      |           |        |
|             1 |  0.0 | 29.485569 |  459.0 |
|             2 |  0.0 | 31.687011 |  246.0 |
|             3 |  0.0 | 33.454647 |  835.0 |
|             4 |  0.0 | 35.281046 | 1361.0 |
|             5 |  0.0 | 32.724035 | 4356.0 |

Since we want to investigate the relationship between protein and rating, we generated a table by grouping recipes based on rate category and aggregate the min, mean, and max of proteins in each category. There is a trend that higher rating groups have higher mean protein PDV, which leads us to think that there is a relationship between protein and rating.

# Assessment of Missingness

We do not believe there is a column in the dataset that is NMAR(Not Missing At Random). There are two columns with missing data which are 'rating' and 'description' and majority missing data appears in 'rating'. We believe the missingness of 'rating' may correlated with other attributes such as 'minutes' (Some recipes requires too much time to cook may have less users to try and rate). We are going to do several permutation tests to test if the distribution of ('minutes', 'n_steps', 'submitted', 'n_ingredients') is the same when column 'rating' is missing and when column 'rating' is not missing.

##  **Comparing null and non-null 'rating' distributions for 'minutes'**

<iframe src="assets/minutes_dist.html" width=500 height=400 frameBorder=0></iframe>

This is the distribution of 'minutes' in two different groups: 

* value of 'rating' is missing 
* value of 'rating' is not missing

Null hypothesis: the distribution of 'minutes' is the same when 'rating' is missing and when 'rating' is not missing.

As the two distributions are **quantitative (numerical) **and look like shifted versions of the same basic shape, we use the **absolute difference in group means** as test statistic.

```python
# observed statistic
obs_stat_minutes = rc.groupby('rating_missing')['minutes'].mean().diff().abs().iloc[-1]
```

obs_stat_minutes: 13.747290902003819

```python
# permutation test
n = 1000
diffs_minutes = []
shuffled = rc.copy()
for _ in range(n):
    shuffled['minutes'] = np.random.permutation(shuffled['minutes'])
    stat = shuffled.groupby('rating_missing')['minutes'].mean().diff().abs().iloc[-1]
    diffs_minutes.append(stat)

# p value 
p_val_minutes = (np.array(diffs_minutes) >= obs_stat_minutes).mean()
```

p_val_minutes: 0.0

**As the p value is significantly small** **(less than 0.05)**, we **reject **the null hypothesis, then column 'rating' is **MAR ** (Missing At Random) dependent on column 'minutes'.

<iframe src="assets/minutes_stat_dist.html" width=500 height=400 frameBorder=0></iframe>

## **Comparing null and non-null 'rating' distributions for 'protein'**

<iframe src="assets/protein_dist.html" width=500 height=400 frameBorder=0></iframe>

This is the distribution of 'protein' in two different groups: 

* value of 'rating' is missing 
* value of 'rating' is not missing

Null hypothesis: the distribution of 'protein' is the same when 'rating' is missing and when 'rating' is not missing.

As the two distributions are **quantitative (numerical) **and look like shifted versions of the same basic shape, we use the **absolute difference in group means** as test statistic.

```python
# observed statistic
obs_stat4 = rc.groupby('rating_missing')['protein'].mean().diff().abs().iloc[-1]
```

obs_stat4: 1.1081245331370013

```python
# permutation test
n = 1000
diffs = []
shuffled = rc.copy()
for _ in range(n):
    shuffled['protein'] = np.random.permutation(shuffled['protein'])
    stat = shuffled.groupby('rating_missing')['protein'].mean().diff().abs().iloc[-1]
    diffs.append(stat)
    
# p value 
p_val4 = (np.array(diffs) >= obs_stat4).mean()
```

p_val4: 0.248

**As the p value is greater than 0.05**, we **fail to reject **the null hypothesis, then column 'rating' is **MAR ** (Missing At Random) dependent on column 'protein'.

<iframe src="assets/protein_stat_dist.html" width=500 height=400 frameBorder=0></iframe>

# Hypothesis testing

Null hypothesis: There is no relationship between the proteins(PDV) in a recipe and its average ratings.

Alternative hypothesis: There is a relationship between the proteins(PDV) in a recipe and its average ratings, specifically, the higher the protein it contains, the higher the average ratings it gets.

We chose the **signed difference between the mean ratings** in the high protein recipe sample and population as the test statistic. We chose mean because the two distributions have very similar shapes. We chose signed difference instead of absolute difference because our alternative hypothesis has a direction: high protein recipes have higher average ratings than that of all recipes. In order to show such relationship, we need test statistics greater than or equal to the observed test statistic. We use 0.05 as the significance level. 

The resulting p-value is 0.393, which is greater than 0.05, so we **fail to reject** our null hypothesis. This indicates that out high protein recipe sample has the same distribution as the population - all recipes. Therefore, it is more likely that there does not exist a relationship between the protein PDV in a recipe and its average ratings.

```python
# use the signed difference between the means in the sample and population as the test statistic
high_protein_mean_rating = high_protein_recipe['rating'].mean()
overall_mean_rating = recipe_cleaned['rating'].mean()
obs_stat = high_protein_mean_rating - overall_mean_rating
```

obs_stat: 0.003060097141928786

```python
# hypothesis test
n = 10000
diffs_in_mean_ratings = []
for _ in range(n):
    sample_index = np.random.choice(np.arange(recipe_cleaned.shape[0]), size=high_protein_recipe.shape[0])
    sample_mean_rating = recipe_cleaned.iloc[sample_index]['rating'].mean()
    diff = sample_mean_rating - overall_mean_rating
    diffs_in_mean_ratings.append(diff)

# p-value
p_val = (np.array(diffs_in_mean_ratings) >= obs_stat).mean()
```

p_val: 0.4051

<iframe src="assets/hyp_stat_dist.html" width=500 height=400 frameBorder=0></iframe>
