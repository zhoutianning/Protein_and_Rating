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
   | 'n_steps'        | Number of steps in recipee                                   |
   | 'steps'          | Text for recipe steps, in order                              |
   | 'description'    | User-provided description                                    |

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

```html
<iframe src="assets/protein_hist.html" width=800 height=600 frameBorder=0></iframe>
```



