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

| name | minutes | n_steps | n_ingredients | avg_rating |
| --- | --- | --- | --- | --- |
| 1 brownies in the world best ever | 40 | 10 | 9 | 4.0 |
| 1 in canada chocolate chip cookies | 45 | 12 | 11 | 5.0 |
| 412 broccoli casserole | 40 | 6 | 9 | 5.0 |
| millionaire pound cake | 120 | 7 | 7 | 5.0 |
| 2000 meatloaf | 90 | 17 | 13 | 5.0 |

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

The observed difference in mean preparation time between recipes with missing and non-missing reviews was **246.0984 minutes**. After 1000 permutations, the resulting p-value was **0.0130**.

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

The observed difference in mean rating between recipes with missing and non-missing reviews was **0.1243**. After 1000 permutations, the resulting p-value was **0.6510**.

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

The resulting p-value was **0.5090**.

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

---

# Baseline Model

For my baseline model, I trained a multiple linear regression model to predict the average rating (`avg_rating`) of a recipe.

The model uses two features:

- `n_steps` (**quantitative**): the number of preparation steps in the recipe
- `n_ingredients` (**quantitative**): the number of ingredients used in the recipe

The model contains 2 quantitative features, 0 ordinal features, and 0 nominal features. Since there are no categorical features in this baseline model, no one-hot encoding or ordinal encoding was necessary.

I used an `sklearn` Pipeline containing:

1. `StandardScaler` to standardize the numerical features
2. `LinearRegression` to fit the prediction model

The dataset was split into training and testing sets, and model performance was evaluated using Root Mean Squared Error (RMSE). RMSE was chosen because it measures the average prediction error in the same units as the response variable (stars), while also penalizing larger prediction mistakes more heavily.

The baseline model achieved:

| Dataset | RMSE |
| --- | --- |
| Training Set | 0.6419 |
| Test Set | 0.6360 |

The training and testing RMSE values are very similar, suggesting that the model is not overfitting and generalizes similarly to unseen data.

However, a test RMSE of approximately 0.64 on a 1–5 star rating scale indicates that the model has limited predictive ability. This suggests that recipe complexity features such as `n_steps` and `n_ingredients` alone do not explain much variation in user ratings.

I do not consider this baseline model to be a strong model because it only uses two simple recipe complexity features. In the final model, I will include additional recipe information and perform more feature engineering to improve prediction performance.

---

# Final Model

For my final model, I used a `RandomForestRegressor` to predict `avg_rating`. I chose this model because random forests can capture nonlinear relationships between recipe characteristics and average rating, which a linear regression model may miss.

## Feature Engineering

Compared to the baseline model, I added two new features:

| Feature | Type | Description |
| --- | --- | --- |
| `calories` | Quantitative | Extracted from the `nutrition` column. Recipes with different calorie levels may represent different types of dishes, which could be related to user ratings. |
| `log_minutes` | Quantitative | A log-transformed version of `minutes`. Cooking time is heavily right-skewed, so this transformation reduces the impact of extremely long cooking times. |

I also kept the baseline features `n_steps` and `n_ingredients`. Since `n_steps` was right-skewed, I applied a `QuantileTransformer` to make its distribution more uniform before fitting the model.

## Modeling Algorithm and Hyperparameter Tuning

The final model used a pipeline with:

1. A `ColumnTransformer` for preprocessing
2. A `RandomForestRegressor` for prediction

The final model was evaluated using the same training and testing split as the baseline model, allowing the RMSE values to be directly compared.

I used `GridSearchCV` with 3-fold cross-validation to search over the following hyperparameters:

| Hyperparameter | Values Tested | Reason |
| --- | --- | --- |
| `n_estimators` | 50, 100 | Controls the number of trees in the random forest |
| `max_depth` | 5, 10, None | Controls the depth of each tree and helps manage overfitting |

The best hyperparameters were:

- `max_depth = 5`
- `n_estimators = 100`

## Final Model Performance

The final model achieved:

| Model | Train RMSE | Test RMSE |
| --- | --- | --- |
| Baseline Model | 0.6419 | 0.6360 |
| Final Model | 0.6392 | 0.6353 |

The final model reduced the test RMSE by about **0.0007** compared to the baseline model.

This improvement is very small, suggesting that recipe characteristics such as `n_steps`, `n_ingredients`, `calories`, and `log_minutes` are weak predictors of average recipe rating. This is consistent with the earlier exploratory analysis and hypothesis test, where recipe complexity did not show a strong relationship with average rating.

The final model still appears to generalize reasonably well because the training and testing RMSE values are close. However, the overall predictive performance remains limited, suggesting that average recipe ratings may depend on factors not captured by these recipe-level features.

---

# Fairness Analysis

For the fairness analysis, I evaluated whether the final model performs differently for simple recipes compared to complex recipes.

- **Group X:** Simple recipes (`n_steps <= 10`)
- **Group Y:** Complex recipes (`n_steps > 10`)

Since this is a regression problem, I used **RMSE** as the evaluation metric.

## Hypotheses

**Null Hypothesis:**  
The model performs similarly for simple and complex recipes. The RMSE difference between groups is due to random chance.

**Alternative Hypothesis:**  
The model performs differently for simple and complex recipes. There is a significant difference between the RMSE for simple recipes and complex recipes.

## Test Statistic

I used the difference in RMSE as the test statistic:

**Complex Recipe RMSE − Simple Recipe RMSE**

The observed results were:

| Group | RMSE |
| --- | --- |
| Simple Recipes | 0.6234 |
| Complex Recipes | 0.6543 |

The observed difference was **0.0309**.

<iframe
  src="proj04/assets/fairness_test.html"
  width="800"
  height="600"
  frameborder="0">
</iframe>

After 1000 permutations, the resulting p-value was **0.0660**.

Since the p-value is greater than the significance level of 0.05, I fail to reject the null hypothesis. Although the model has slightly higher prediction error for complex recipes than simple recipes, this difference is not statistically significant.

Therefore, I do not find evidence that the model is unfair with respect to recipe complexity. The model appears to have similar predictive performance across simple and complex recipes, though this conclusion is limited to the groups, features, dataset, and fairness definition used in this analysis.