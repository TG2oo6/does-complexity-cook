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

[INSERT YOUR CLEANED HEAD MARKDOWN TABLE HERE]


---

## Univariate Analysis

The distribution below shows the number of preparation steps across recipes.

<iframe
  src="assets/steps_distribution.html"
  width="800"
  height="600"
  frameborder="0">
</iframe>

The distribution of `n_steps` is right-skewed, with most recipes containing between 5–15 preparation steps. Very few recipes contain more than 40 steps, suggesting that most Food.com recipes are relatively straightforward.


---

## Bivariate Analysis

The following plot compares recipe complexity (number of steps) with average user rating.

<iframe
  src="assets/steps_vs_rating.html"
  width="800"
  height="600"
  frameborder="0">
</iframe>

There does not appear to be a strong linear relationship between the number of steps and average rating. Recipes of all complexity levels receive a wide range of ratings, suggesting that additional steps alone may not strongly influence user satisfaction.


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