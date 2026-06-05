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

---

# Assessment of Missingness

## NMAR Analysis

I believe the missingness of the `review` column could be NMAR (Not Missing At Random). Users who choose not to leave a written review may differ from users who write reviews in ways that are not directly captured by the dataset. For example, users may only leave a rating without a review because they are less motivated to explain their opinion or have lower engagement with the recipe.

To determine whether the missingness is truly NMAR, additional information such as user engagement, time spent viewing the recipe page, or whether the user usually writes detailed reviews would be needed.

---

## Missingness Dependency

I analyzed the missingness of the `review` column by performing permutation tests to determine whether the missingness depends on other columns in the dataset.

### Dependency on Preparation Time

First, I tested whether the missingness of `review` depends on `minutes`, the preparation time of a recipe.

**Null Hypothesis:** The distribution of `minutes` is the same for recipes with missing and non-missing reviews.

**Alternative Hypothesis:** The distribution of `minutes` differs between recipes with missing and non-missing reviews.

The observed difference in mean preparation time between recipes with missing and non-missing reviews was **246.0984 minutes**. After 1000 permutations, the resulting p-value was **0.0350**.

<iframe
  src="proj04/assets/missingness_minutes_test.html"
  width="800"
  height="600"
  frameborder="0">
</iframe>

Since the p-value is below the significance level of 0.05, I reject the null hypothesis. This suggests that the missingness of `review` depends on `minutes`, meaning recipe preparation time may be related to whether users choose to leave written feedback.

---

### Dependency on Rating

I also tested whether the missingness of `review` depends on the recipe's `rating`.

**Null Hypothesis:** The distribution of ratings is the same for recipes with missing and non-missing reviews.

**Alternative Hypothesis:** The distribution of ratings differs between recipes with missing and non-missing reviews.

The observed difference in mean rating between recipes with missing and non-missing reviews was **0.1243**. After 1000 permutations, the resulting p-value was **0.7590**.

Since the p-value is much greater than 0.05, I fail to reject the null hypothesis. This suggests that the missingness of `review` does not depend on the user's rating.

---

# Hypothesis Testing

To further investigate whether recipe complexity is associated with user ratings, I performed a permutation test comparing simple and complex recipes.

Recipes were separated into two groups:

- **Simple recipes:** recipes with 10 or fewer preparation steps
- **Complex recipes:** recipes with more than 10 preparation steps

## Hypotheses

**Null Hypothesis:**  
The distribution of average ratings for simple recipes and complex recipes is the same. Any observed difference in mean ratings is due to random chance.

**Alternative Hypothesis:**  
The distribution of average ratings for simple recipes and complex recipes is different.

## Test Statistic

I used the difference in group means as my test statistic:

**Mean rating of complex recipes − mean rating of simple recipes**

This statistic was chosen because the question focuses on whether recipe complexity is associated with higher or lower user ratings, and the average rating provides a direct measure of user satisfaction.

The significance level used was **0.05**.

The observed results were:

- Simple recipes mean rating: **4.6242**
- Complex recipes mean rating: **4.6273**
- Observed difference: **0.0031**

A permutation test with 1000 simulations was performed.

<iframe
  src="proj04/assets/hypothesis_test.html"
  width="800"
  height="600"
  frameborder="0">
</iframe>

The resulting p-value was **0.5070**.

Since the p-value is greater than the significance level of 0.05, I fail to reject the null hypothesis.

This suggests that there is not sufficient evidence to conclude that simple and complex recipes have different average ratings. Recipe complexity, measured by number of preparation steps, does not appear to have a strong association with user ratings in this dataset.

---

# Framing a Prediction Problem

The prediction problem I chose is:

**Can we predict the average user rating of a recipe using information about the recipe when it is first posted?**

This is a **regression problem** because the response variable, `avg_rating`, is a continuous numerical value ranging from 1 to 5.

## Response Variable

The response variable is `avg_rating`, which represents the average user rating received by each recipe.

I chose this variable because average rating serves as a measure of user satisfaction, which directly relates to my overall goal of understanding how recipe characteristics influence how users perceive recipes.

## Features Available at Time of Prediction

At the time of prediction, the model would only have access to information available when a recipe is first posted, before users rate or review it.

Examples of usable features include:

- `n_steps`: number of preparation steps
- `n_ingredients`: number of ingredients required
- `minutes`: preparation and cooking time
- extracted nutritional information

I will not use features such as individual user ratings or written reviews because these are generated after users interact with the recipe. Including these features would introduce data leakage because they would not be available when predicting the rating of a new recipe.

## Evaluation Metric

I will evaluate my model using **Root Mean Squared Error (RMSE)**.

RMSE measures the average prediction error in rating points, making it easy to interpret because it uses the same units as the response variable.

I chose RMSE over other metrics such as Mean Absolute Error (MAE) because RMSE penalizes larger prediction errors more heavily. In this context, predicting a recipe's rating very incorrectly is more problematic than making several small prediction errors.