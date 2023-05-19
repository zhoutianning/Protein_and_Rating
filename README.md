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

