# Does Complexity Cook? 
By Thalesh Gupta

A data science project for DSC 80 at UCSD exploring whether recipe complexity affects user ratings on Food.com.

---

# Introduction

Food.com is a recipe-sharing platform where users can upload recipes and receive reviews and ratings from other users. This project explores whether recipe complexity affects user satisfaction.

Specifically, I investigate the question:

**How does recipe complexity — measured by the number of preparation steps and number of ingredients — relate to the average user rating of a recipe?**

Understanding this relationship can help determine whether users prefer simpler recipes that are easier to make, or whether more complex recipes provide enough additional value to receive similar or higher ratings.

## Dataset

This project uses two datasets from Food.com:

- `RAW_recipes.csv`: Contains information about **83,782 recipes**, including preparation time, ingredients, steps, and nutritional information.
- `interactions.csv`: Contains **731,927 user interactions**, including reviews and ratings.

The two datasets were combined by matching the `id` column in `RAW_recipes.csv` with the `recipe_id` column in `interactions.csv`.

The main columns relevant to this analysis are:

| Column | Description |
| --- | --- |
| `id` | Unique identifier for each recipe |
| `recipe_id` | Connects user reviews to recipes in the recipe dataset |
| `rating` | User rating of a recipe from 1 to 5 |
| `avg_rating` | Average rating of a recipe after combining all user ratings |
| `minutes` | Preparation and cooking time required for the recipe |
| `n_steps` | Number of recipe instructions, representing preparation complexity |
| `n_ingredients` | Number of ingredients required, representing ingredient complexity |
| `nutrition` | Nutritional information about the recipe |

---

# Data Cleaning and Exploratory Data Analysis

## Data Cleaning

To prepare the dataset for analysis, I performed several cleaning steps:

1. I merged `RAW_recipes.csv` with `interactions.csv` using recipe identifiers. The `id` column from the recipe dataset corresponds to the `recipe_id` column in the interactions dataset.

2. I replaced ratings of `0` with missing values (`NaN`). Since valid Food.com ratings range from 1 to 5, a rating of 0 represents a user interaction without an actual rating.

3. Since each recipe can have multiple user reviews, I calculated the average rating for each recipe and created a new column called `avg_rating`.

4. I removed duplicate recipe rows so each recipe appears once in the cleaned dataset.

5. I converted the `submitted` column into datetime format to allow possible time-based analysis.

The first few rows of the cleaned dataset are shown below:

| name | id | minutes | submitted | n_steps | n_ingredients | rating | avg_rating |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 brownies in the world best ever | 333281 | 40 | 2008-10-27 | 10 | 9 | 4.0 | 4.0 |
| 1 in canada chocolate chip cookies | 453467 | 45 | 2011-04-11 | 12 | 11 | 5.0 | 5.0 |
| 412 broccoli casserole | 306168 | 40 | 2008-05-30 | 6 | 9 | 5.0 | 5.0 |
| millionaire pound cake | 286009 | 120 | 2008-02-12 | 7 | 7 | 5.0 | 5.0 |
| 2000 meatloaf | 475785 | 90 | 2012-03-06 | 17 | 13 | 5.0 | 5.0 |

---

## Univariate Analysis

The distribution below shows the number of preparation steps across recipes.

<iframe
  src="proj04/assets/steps_distribution.html"
  width="800"
  height="600"
  frameborder="0">
</iframe>

The distribution of `n_steps` is right-skewed, with most recipes containing between 5–15 preparation steps. Very few recipes contain more than 40 steps, suggesting that most Food.com recipes are relatively straightforward.

The distribution of average recipe ratings is also shown below.

<iframe
  src="proj04/assets/rating_distribution.html"
  width="800"
  height="600"
  frameborder="0">
</iframe>

The distribution of average ratings is heavily left-skewed, with most recipes receiving ratings between 4 and 5 stars. This suggests that Food.com ratings are generally high, possibly because users are more likely to rate recipes they enjoyed.

---

## Bivariate Analysis

The following plot compares recipe complexity (number of steps) with average user rating.

<iframe
  src="proj04/assets/steps_vs_rating.html"
  width="800"
  height="600"
  frameborder="0">
</iframe>

There does not appear to be a strong linear relationship between the number of steps and average rating. Recipes of all complexity levels receive a wide range of ratings, suggesting that additional steps alone may not strongly influence user satisfaction.

The relationship between number of ingredients and average rating is also shown below.

<iframe
  src="proj04/assets/ingredients_vs_rating.html"
  width="800"
  height="600"
  frameborder="0">
</iframe>

Similarly, there is no strong relationship between ingredient count and average rating. Recipes with both few and many ingredients can receive high ratings.

---

## Interesting Aggregates

To further examine the relationship between complexity and rating, I grouped recipes into bins based on their number of preparation steps.

| steps_bin | avg_rating |
| --- | --- |
| 1–5 | 4.637856 |
| 6–10 | 4.616250 |
| 11–15 | 4.617573 |
| 16–20 | 4.638008 |
| 20+ | 4.647305 |

Average ratings remain very similar across all step complexity groups, ranging from approximately 4.62 to 4.65 stars. This suggests that recipe complexity alone does not strongly determine user satisfaction, motivating further hypothesis testing.