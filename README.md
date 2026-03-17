# What Makes a 5-Star Recipe?

**By:** Mahir Oza, Xiaolong Yu, Donald Yu, Ali Karim

---

## Introduction

This project focuses on a comprehensive dataset of recipes and ratings derived from Food.com, which was originally scraped by researchers for a study on recommender systems. We picked this dataset because food is definitely a common shared interest among all our group members, and we're genuinely curious and excited to explore the data and discover meaningful insights.

The data consists of two primary CSV files: `RAW_recipes.csv`, which details recipe attributes such as cooking time, nutritional information, and preparation steps, and `RAW_interactions.csv`, which contains user-submitted reviews and ratings. For the purpose of this analysis, we are utilizing a subset of the original data that includes only recipes and reviews posted since 2008.

Our central question is: **What types of recipes tend to have higher average ratings, and can we predict a recipe's rating based on its characteristics?** Understanding these trends is relevant not only for home cooks looking to optimize their meal planning but also for recipe platforms looking to improve their recommendation systems.

The dataset contains **83,782 recipes** with the following relevant columns:

| Column | Description |
|---|---|
| `name` | Recipe name |
| `minutes` | Cooking time in minutes |
| `n_steps` | Number of preparation steps |
| `n_ingredients` | Number of ingredients |
| `calories` | Total calories per serving |
| `protein` | Protein content (PDV) |
| `sugar` | Sugar content (PDV) |
| `total_fat` | Total fat content (PDV) |
| `average_rating` | Mean of all user ratings (1.0–5.0) |

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

We performed several cleaning steps to prepare the data for analysis:

1. **Merged the two datasets** using a left join on recipe ID, so each recipe is matched with its user ratings.
2. **Replaced ratings of 0 with NaN** because a rating of 0 does not exist on Food.com's 1–5 scale, these represent missing ratings, not actual zero-star reviews.
3. **Computed average ratings** by grouping all ratings per recipe and taking the mean, then merging this back into the recipes dataframe.
4. **Parsed the nutrition column** from a string representation of a list into seven individual numeric columns: calories, total_fat, sugar, sodium, protein, saturated_fat, and carbohydrates.
5. **Cleaned cooking time** by replacing 0 minute values with NaN (a recipe can't take 0 minutes), removing recipes over 1440 minutes (24 hours) as likely data entry errors, and capping extreme outliers at the 99th percentile.
6. **Parsed list columns** (tags, ingredients, steps) from strings into actual Python lists, and created count columns (num_tags, num_ingredients, num_steps).
7. **Extracted date features** (submitted_year, submitted_month) from the submission date.

Here is the head of our cleaned DataFrame:

| name | minutes | n_steps | n_ingredients | calories | protein | sugar | total_fat | average_rating |
|---|---|---|---|---|---|---|---|---|
| 1 brownies in the world best ever | 40.0 | 10 | 9 | 138.4 | 3.0 | 50.0 | 10.0 | 4.0 |
| 1 in canada chocolate chip cookies | 45.0 | 12 | 11 | 595.1 | 13.0 | 211.0 | 46.0 | 5.0 |
| 412 broccoli casserole | 40.0 | 6 | 9 | 194.8 | 22.0 | 6.0 | 20.0 | 5.0 |
| millionaire pound cake | 120.0 | 7 | 7 | 878.3 | 20.0 | 326.0 | 63.0 | 5.0 |
| 2000 meatloaf | 90.0 | 17 | 13 | 267.0 | 29.0 | 12.0 | 30.0 | 5.0 |

### Univariate Analysis

<iframe
  src="assets/rating-distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution of average recipe ratings is heavily left-skewed, with the vast majority of recipes rated between 4 and 5 stars. This suggests that users tend to only rate recipes they enjoyed, creating a strong selection bias in the data.

### Bivariate Analysis

<iframe
  src="assets/ingredients-vs-rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This box plot shows the relationship between the number of ingredients and average rating. The median rating stays near 5.0 across all ingredient counts, but simpler recipes (fewer ingredients) tend to have more variability, with more low-rating outliers. Recipes with 20+ ingredients have tighter, more consistently high ratings.

### Interesting Aggregates

We grouped recipes by calorie level (Low, Medium, High) and step complexity (Few, Medium, Many) to examine how these factors interact:

| calorie_level | Few Steps | Medium Steps | Many Steps |
|---|---|---|---|
| Low | 4.66 | 4.62 | 4.63 |
| Medium | 4.63 | 4.61 | 4.63 |
| High | 4.60 | 4.62 | 4.64 |

The table shows that average ratings are remarkably consistent across groups, with very little variation between different calorie levels and step complexities.

---

## Assessment of Missingness

### NMAR Analysis

We believe the `average_rating` column is likely **NMAR** (Not Missing At Random). A recipe has a missing average rating when no users have rated it, and people tend to rate recipes they have actually made. Morecomplex recipes may receive fewer ratings simply because fewer people attempt them, so whether a recipe gets rated depends on how popular or approachable it is, which is not captured by other columns in the dataset. If we had data on recipe page views or the number of times a recipe was saved, this could help explain the missingness and potentially make it MAR.

### Missingness Dependency

We analyzed whether the missingness of `average_rating` depends on other columns using permutation tests with 10,000 shuffles at α = 0.05.

**Depends on `n_steps` (p = 0.0000):** The missingness of average_rating does depend on the number of steps. Recipes with missing ratings tend to have more steps on average (observed difference of 1.4930 steps). This makes sense because more complex recipes with many steps may attract fewer users to try and rate them.

<iframe
  src="assets/steps-missingness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Does not depend on `sodium` (p = 0.8762):** The missingness of average_rating does not significantly depend on sodium content. This suggests that a recipe's sodium level has no meaningful relationship with whether it gets rated or not.

<iframe
  src="assets/sodium-missingness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

---

## Hypothesis Testing

**Null Hypothesis:** The average rating of recipes with many ingredients (above the median) is the same as the average rating of recipes with few ingredients (at or below the median). Any observed difference is due to random chance.

**Alternative Hypothesis:** The average rating of recipes with many ingredients is different from the average rating of recipes with few ingredients.

**Test Statistic:** Absolute difference in group means.

**Significance Level:** α = 0.05

We used a permutation test with 10,000 shuffles because our rating data is heavily skewed and does not follow a normal distribution, making a t-test inappropriate.

**Result:** We obtained a p-value of **0.3731**, which exceeds our significance level of 0.05. Therefore, we **fail to reject the null hypothesis**, there is not enough evidence to conclude that the number of ingredients has a significant effect on average recipe ratings.

---

## Framing a Prediction Problem

We are predicting the **average rating** of a recipe. This is a **regression** problem because average rating is a continuous variable ranging from 1.0 to 5.0.

We chose `average_rating` as our response variable because understanding what recipe characteristics lead to higher ratings is valuable both for recipe platforms looking to improve recommendations and for home cooks trying to create crowd-pleasing recipes.

**Evaluation Metric:** We are using **RMSE** (Root Mean Squared Error) because it is interpretable in rating points and penalizes large prediction errors more heavily than MAE, which is important when predicting ratings on a 1 to 5 scale.

**Time of Prediction:** At the time a recipe is first posted, we know its attributes (cooking time, steps, ingredients, nutritional info, tags) but have no user feedback yet. Therefore, we only use features available at the time of submission, including: minutes, n_steps, n_ingredients, calories, protein, sugar, total_fat, and other recipe characteristics, and exclude any user interaction data like ratings or reviews.

---

## Baseline Model

Our baseline model is a **Linear Regression** using seven quantitative features: `calories`, `n_ingredients`, `n_steps`, `minutes`, `protein`, `sugar`, and `total_fat`. All features are numerical, so no ordinal or nominal encodings were needed. We scaled all features using `StandardScaler` and implemented the model in a single sklearn `Pipeline`.

**Performance:**
- RMSE: **0.6444**
- R^2: **0.0015**

We do not consider this a good model. The R^2 of 0.0015 means the model explains less than 1% of the variance in ratings. This is largely because the rating distribution is so heavily skewed toward 5 stars that the model essentially learns to predict around 4.7 for every recipe. Linear Regression assumes a linear relationship between features and ratings, which is too simplistic to capture the complex patterns in recipe data. The purpose of this baseline is to establish a benchmark that our final model can improve upon.

---

## Final Model

We tested three different approaches to improve on the baseline, exploring both numeric only models and text based models.

### Model Comparison

| Model | RMSE | R^2 | Key Approach |
|---|---|---|---|
| Baseline (Linear Regression) | 0.6444 | 0.0015 | 7 numeric features |
| Random Forest | 0.6434 | 0.0048 | Numeric + engineered binary features |
| HistGradientBoosting | 0.6346 | 0.0039 | Numeric + ratio/log features |
| **TF-IDF + Ridge** | **0.6285** | **0.0230** | **Text (20K words) + numeric** |

We selected **TF-IDF + Ridge Regression** as our final model because it achieved the best performance on both RMSE and R^2. We combined all available text features, including: recipe names, descriptions, tags, and ingredient lists, into a single text field per recipe, then converted them into 20000 numerical features using TF-IDF vectorization. These text features were combined with the same 7 scaled numeric features from the baseline and trained with Ridge Regression, using GridSearchCV with 5-fold cross-validation to tune the regularization parameter.

**Performance:**
- RMSE: **0.6285**
- R^2: **0.0230**

The key insight is that text features capture meaningful patterns about what makes users rate recipes highly that numerical features alone cannot. The R^2 increased from 0.0015 to 0.0230, meaning the final model explains about 15 times more variation in ratings than the baseline. However, performance remains limited overall because recipe ratings are heavily skewed toward 4 and 5 stars, leaving relatively little variation for any model to learn from.

<iframe
  src="assets/predicted-vs-actual.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The scatter plot compares what our model predicted versus the actual ratings for each recipe in the test set. The red dashed line represents perfect predictions. Most dots cluster in the top-right corner around 4 to 5 stars, reflecting the heavy skew in our dataset. The model tends to predict ratings around 4.5 to 4.8 for almost every recipe, even for recipes that actually received low ratings. This means the model struggles to distinguish between poorly rated and highly rated recipes, largely because there are so few low-rated recipes for it to learn from. Despite this limitation, TF-IDF + Ridge still outperforms all other models we tested.

---

## Fairness Analysis

To evaluate fairness, we examined whether our final model (TF-IDF + Ridge) performed differently for recipes with **many ingredients** versus recipes with **few ingredients**. We chose this grouping because the number of ingredients is a natural measure of recipe complexity and was central to our earlier hypothesis testing.

**Null Hypothesis:** Our model is fair. Its RMSE for recipes with few ingredients and recipes with many ingredients are roughly the same, and any differences are due to random chance.

**Alternative Hypothesis:** Our model is unfair. Its RMSE for recipes with few ingredients is higher than its RMSE for recipes with many ingredients.

**Test Statistic:** Difference in RMSE (few ingredients - many ingredients)

**Significance Level:** α = 0.05

Recipes were split based on the **median number of ingredients** in the test set, and RMSE was computed separately for each group. We then ran a permutation test with 10,000 shuffles to determine if the observed difference was statistically significant.

**Performance by group:**
- RMSE (Few Ingredients): **0.6345**
- RMSE (Many Ingredients): **0.6205**

<iframe
  src="assets/fairness-rmse.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The RMSE for recipes with few ingredients (0.6345) and many ingredients (0.6205) are very close, with a difference of only 0.0140. Our permutation test produced a p-value of **0.1914**, which exceeds our significance level of 0.05. Therefore, we **fail to reject the null hypothesis**, there is not enough evidence to conclude that the model performs unfairly across recipe complexity levels. The model behaves similarly for both simpler and more complex recipes.
