---
layout: default
title: " A Data-Driven Look at Recipe Ratings"
author: "Daniel Leifer and Liam Herron"
---

# Introduction

Like most people who love to cook, we spend a lot of time looking for great recipes and are curious about if we can quantify what makes them great. This project uses a dataset of recipes and user interactions from [Food.com](https://www.food.com) to investigate how different recipe attributes correlate with user satisfaction, as measured by average ratings.

**Central Question:**  
*How can we predict a recipe’s rating based solely on the information available about the recipe itself?*

This question matters because understanding which factors influence user ratings can guide home cooks, professional chefs, recipe creators, and platforms like Food.com in developing better, more appealing recipes. For example, if we find that recipes with more detailed instructions or fewer steps tend to be rated higher, this insight could help recipe authors improve clarity and simplicity in their creations. 

**Data Overview:**  
We used two datasets from Food.com in our analysis:
1. **Recipes Dataset:** Contains details of **83,782** recipes.
2. **Interactions (Ratings) Dataset:** Contains details of **731927** user-submitted ratings and reviews of these recipes.

**Relevant Columns & Their Descriptions:**

From the Recipes Dataset:
- **`name`**: The recipe’s title.
- **`id`**: A unique id for each recipe.
- **`minutes`**: The preparation time to make a recipe.
- **`n_steps`**: The number of steps in the recipe instructions.
- **`ingredients`** and **`n_ingredients`**: The full ingredient list and the count of the number of ingredients in the list.
- **`nutrition`**: A list with nutritional data (calories, fat, sugar, sodium, protein, etc.).
- **`description`**: A user-provided description of the recipe.
- **`tags`**: Categorical indicators (e.g., cuisine type, dietary restrictions) that may influence ratings.

From the Interactions (Ratings) Dataset:
- **`rating`**: The key target variable we aim to predict. This is a average scored by the user in their review. We agregate this by recipe in our analysis.
- **`recipe_id`**: For linking user feedback to recipes.
- **`review`**: Optional user text feedback that can reflect sentiment, though we focus primarily on numerical ratings.

In summary, our analysis revolves around how features like preparation time, number of steps, ingredients, and nutritional information correlate with recipe ratings. By understanding these relationships, we can better predict which recipes will be highly regarded and more likely to please future cooks and diners.


# Data Cleaning and Exploratory Data Analysis

Before diving into analysis, we carefully cleaned and prepared the data to ensure that our findings and predictions would be both reliable and meaningful.

**Data Cleaning Steps:**
1. **Merging Datasets:**  
   We began with two datasets: one containing recipes and their attributes, and another containing user interactions (ratings and reviews). We performed a left merge using the recipe ID to combine attributes (like number of steps, ingredients, and nutrition) with user feedback.

2. **Handling Missing and Invalid Ratings:**  
   We filled all possible missing ratings with NaN. Most notably we replaced all ratings of `0` with `NaN`. A zero rating doesn’t represent a user’s negative opinion; it often indicates no rating was provided at all. We later used step-mean imputation to fill these (more on this later on).

3. **Computing Average Ratings per Recipe:**  
   After merging, we calculated the average rating for each recipe and added it to the recipes dataset. This gives us a single, clean metric (`rating`) to work with for our modeling tasks.

4. **Imputing Missing Values:**  
   To handle missing values in columns like `rating` and possibly others, we used a random-imputation approach, drawing new values from observed distributions. This technique preserves the statistical characteristics of the data, ensuring that our analysis isn’t biased by arbitrary substitutions or by dropping rows entirely.

5. **Extracting and Cleaning Nutrition Information:**  
   We split the `nutrition` column (which contained multiple nutritional metrics) into separate columns (e.g., `calories`, `sugar_pdv`, `sodium_pdv`) for easier analysis. This step allowed us to treat each nutritional feature independently, providing more granular insights.

## Head of the Cleaned DataFrame

Below is a preview of the first few rows of our cleaned and merged DataFrame:

<div style="overflow-x: auto;">
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>ID</th>
      <th>Minutes</th>
      <th>Contributor ID</th>
      <th>Submitted</th>
      <th>Tags</th>
      <th>Nutrition</th>
      <th>n_steps</th>
      <th>Steps</th>
      <th>Description</th>
      <th>Protein PDV</th>
      <th>Saturated Fat PDV</th>
      <th>Carbohydrates PDV</th>
      <th>Description Length</th>
      <th>Tags Count</th>
      <th>Bin</th>
      <th>Rounded Rating</th>
      <th>Time Bin</th>
      <th>Sodium Amount</th>
      <th>Rating Bin</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Impossible Macaroni and Cheese Pie</td>
      <td>275022</td>
      <td>50</td>
      <td>531768</td>
      <td>2008-01-01</td>
      <td>['60-minutes-or-less', 'time-to-make', 'course']</td>
      <td>[386.1, 34.0, 7.0, 24.0, 41.0, 62.0, 8.0]</td>
      <td>11</td>
      <td>['heat oven to 400 degrees fahrenheit', 'grease pan']</td>
      <td>One of my mom's favorite Bisquick recipes...</td>
      <td>41.0</td>
      <td>62.0</td>
      <td>8.0</td>
      <td>11.0</td>
      <td>328</td>
      <td>[10, 15)</td>
      <td>3.0</td>
      <td>31-60</td>
      <td>24.0</td>
      <td>2-3</td>
    </tr>
    <tr>
      <td>Impossible Rhubarb Pie</td>
      <td>275024</td>
      <td>55</td>
      <td>531768</td>
      <td>2008-01-01</td>
      <td>['60-minutes-or-less', 'time-to-make', 'course']</td>
      <td>[377.1, 18.0, 208.0, 13.0, 13.0, 30.0, 20.0]</td>
      <td>6</td>
      <td>['heat oven to 375 degrees', 'grease pan']</td>
      <td>A childhood favorite of mine. My mom loved...</td>
      <td>13.0</td>
      <td>30.0</td>
      <td>20.0</td>
      <td>20.0</td>
      <td>190</td>
      <td>[5, 10)</td>
      <td>3.0</td>
      <td>31-60</td>
      <td>13.0</td>
      <td>2-3</td>
    </tr>
    <tr>
      <td>Impossible Seafood Pie</td>
      <td>275026</td>
      <td>45</td>
      <td>531768</td>
      <td>2008-01-01</td>
      <td>['60-minutes-or-less', 'time-to-make', 'course']</td>
      <td>[326.6, 30.0, 12.0, 27.0, 37.0, 51.0, 5.0]</td>
      <td>7</td>
      <td>['preheat oven to 400f', 'lightly grease large pan']</td>
      <td>This is an oldie but a goodie. Mom's stand...</td>
      <td>37.0</td>
      <td>51.0</td>
      <td>5.0</td>
      <td>25.0</td>
      <td>203</td>
      <td>[5, 10)</td>
      <td>3.0</td>
      <td>31-60</td>
      <td>27.0</td>
      <td>2-3</td>
    </tr>
    <tr>
      <td>Paula Deen's Caramel Apple Cheesecake</td>
      <td>275030</td>
      <td>45</td>
      <td>666723</td>
      <td>2008-01-01</td>
      <td>['60-minutes-or-less', 'time-to-make', 'course']</td>
      <td>[577.7, 53.0, 149.0, 19.0, 14.0, 67.0, 21.0]</td>
      <td>11</td>
      <td>['preheat oven to 350f', 'reserve 3 / 4 cup apple mixture']</td>
      <td>Thank you Paula Deen! Hubby just happened...</td>
      <td>14.0</td>
      <td>67.0</td>
      <td>21.0</td>
      <td>45.0</td>
      <td>167</td>
      <td>[10, 15)</td>
      <td>5.0</td>
      <td>31-60</td>
      <td>19.0</td>
      <td>4-5</td>
    </tr>
    <tr>
      <td>Midori Poached Pears</td>
      <td>275032</td>
      <td>25</td>
      <td>307114</td>
      <td>2008-01-01</td>
      <td>['lactose', '30-minutes-or-less', 'time-to-make']</td>
      <td>[386.9, 0.0, 347.0, 0.0, 1.0, 0.0, 33.0]</td>
      <td>8</td>
      <td>['bring midori , sugar , spices , rinds and water to boil']</td>
      <td>The green colour looks fabulous and the t...</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>33.0</td>
      <td>25.0</td>
      <td>159</td>
      <td>[5, 10)</td>
      <td>5.0</td>
      <td>16-30</td>
      <td>0.0</td>
      <td>4-5</td>
    </tr>
  </tbody>
</table>
</div>




## Univariate Analysis

To understand our data, we first look at the distribution of key variables individually.

### Distribution of Average Recipe Ratings
<iframe
  src="assets/ratings-distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*Explanation:* This histogram shows that the vast majority of recipes have high average ratings, with a notable clustering around whole numbers like 4 and 5. This suggests that users generally tend to rate recipes positively. Understanding this skew is essential since a strong positive bias might make it harder to differentiate truly exceptional recipes from merely good ones.

### Distribution of Number of Ingredients
<iframe
  src="assets/ingredients-distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*Explanation:* Most recipes contain around 5 to 12 ingredients, with a peak near 8. This indicates that recipes are often somewhat complex but not over the top. Knowing this helps guide our interpretation of complexity-related features later on.

### Distribution of Number of Steps
<iframe
  src="assets/steps-distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*Explanation:* We also found that most recipes have between 4 and 12 steps. Like with ingredients, this indicates that recipes are often somewhat complex but not over the top.

---

## Bivariate Analysis

To investigate relationships between variables, we compare them directly.

### Relationship Between Number of Steps and Ratings
<iframe
  src="assets/rating-vs-steps-bins.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*Explanation:* This grouped histogram displays ratings across different `n_steps` bins. The distributions are similar, indicating that simply knowing the number of steps is not a strong predictor of rating on its own. In other words, complexity alone doesn’t guarantee a better or worse rating.

### Relationship Between Number of Ingredients and Ratings
<iframe
  src="assets/ingredients-vs-ratings.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*Explanation:* This set of box-plots displays ratings across different `n_ingredients`. The distributions are similar, indicating that simply knowing the number of ingredients is not a strong predictor of rating on its own. In other words, complexity alone doesn’t guarantee a better or worse rating. Maybe the combination of number of steps and ingredients will reveal something later...

---

## Grouped Table and Aggregates

We also created a grouped table to see how average nutritional values vary by rating ranges:

Average Sodium PDV, Sugar PDV, and Calories by Rating Bins

| rating_bins | sodium_pdv | sugar_pdv | calories   |
|-------------|------------|-----------|------------|
| 1-2         | 32.465152  | 77.036364 | 467.941818 |
| 2-3         | 37.374908  | 81.851064 | 437.671460 |
| 3-4         | 28.573349  | 63.574061 | 430.385835 |
| 4-5         | 28.638137  | 69.117757 | 429.133339 |


*Significance:*  
This table provides insight into whether healthier recipes (e.g., lower sodium, fewer calories) correlate with higher ratings. Not suprisingly, because it is a diverse set of recipes, recipes rated between 3 and 5 show some variation in sugar and sodium. Howerver overall recipes with less sugar, sodium, and calories may tend to score better.

---

## Imputation of Missing Values

We employed a step-mean imputation technique for missing values (such as rating when originally 0). This approach involves imputing missing values by calculating the mean of nearby values within defined bins, maintaining the overall distribution's statistical properties. By using this method, we ensure the imputed values align closely with the original data patterns, avoiding bias toward extreme values. This technique preserves the integrity of our dataset, allowing for more accurate downstream analyses and model predictions. You can see that the data distribution looks very similar before and after the impuation. Here is the distribution before and after imputation:

Before:

<iframe
  src="assets/before-ratings-distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

After: 

<iframe
  src="assets/ratings-distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>



In summary, the data cleaning and EDA steps laid a solid foundation for our predictive modeling. By carefully handling missing ratings, merging datasets, and exploring both univariate and bivariate relationships, we are now prepared to frame a prediction problem and move toward building predictive models in the next steps.


# Framing a Prediction Problem

**Type of Problem:**  
Our goal is to predict a numerical value: the average recipe rating. Since the rating is a continuous variable, this is a **regression** problem rather than a classification one.

**Response Variable:**  
The response variable we aim to predict is the **average rating** of each recipe. We chose this because the rating provides a direct measure of how well-received a recipe is by users. By predicting the rating, we can gain insights into which recipe attributes contribute most to its success.

**Evaluation Metric:**  
We will use **Mean Squared Error (MSE)** as our primary evaluation metric. MSE is well-suited for regression tasks because it directly measures the average squared difference between predicted and actual ratings.  It also penalizes large errors more heavily, encouraging the model to be accurate even on recipes that would otherwise be outliers.  
While we also considered other metrics metrics like MAE or R², we decided on MSE for this particular data set to really try and understand the driving factors behind all of the recipe ratings.

**A note on the features we chose:**  
We were careful to only use features available at the “time of prediction.” In other words, we relied solely on attributes of the recipe itself (e.g., ingredients, number of steps, preparation time, nutritional content, tags) that are known as soon as the recipe is created and published and we chose not include user ratings or reviews from the future, since our goal is to predict how the recipe would be rated before any user feedback is received. This ensures our model’s predictions are realistic and reflect the information a recipe creator or platform would have when they post it.


# Baseline Model

For our baseline model, we chose a straightforward **Linear Regression** approach. We included just two features derived directly from the recipes dataset to predict the average rating:

1. **`n_steps`** (Quantitative): The number of steps in the recipe instructions. This is a numeric value that can theoretically range from very few to many. That being said we know most of the data falls between 5 and 12 steps.

2. **`n_ingredients`** (Quantitative): The count of ingredients used in the recipe. Another strictly numeric value, where a higher count typically implies more complexity to a recipe. We know from earlier analysis that there is a similar distribution to that of the number of steps.

**Data Types of the Features:**

**Quantitative Features:**  
  We did not encode either of these because they are numeric.
  - `n_steps`  
  - `n_ingredients`

**Ordinal Features:**  
None of the initial baseline features were inherently ordinal. 

**Nominal Features:**  
None were used in the baseline model. We didn’t include categorical variables (like tags or submission date categories) at this stage, so we didn't use one-hot encoding or similar transformations.

---

**Model Performance:**  
We evaluated the baseline model’s predictions against the observed ratings using the Mean Squared Error (MSE). The MSE for our baseline model was approximately **0.316**.

In context, considering that ratings typically range from 1 to 5, this MSE of 0.316 means our predictions are, on average, off by about `sqrt(0.316) ≈ 0.56` points. Basically, if the true rating is a 4.0, the model would often predict somewhere between 3.4 to 4.6.

**Is the Model “Good”?**  
While this baseline performance is not terrible, it’s definently not highly accurate, particularly given how many ratings are clustered between 4 and 5. There’s likely room for improvement by incorporating more features and transformations. Given that we’ve only used two very simple features, with limited correleation to rating, it’s expected that this initial model doesn’t capture the complexity of what makes a recipe highly rated.

To summarize, our baseline model provides a starting point. We hope that adding more features—such as nutritional information, complexity of instructions, and more will help improve our predictive accuracy.


# Final Model

**Features Added and Their Rationale:**

In our baseline model, we relied primarily on the number of steps and the number of ingredients. While these features capture some sense of a recipe’s complexity, the complexity of recipe is not the only indicator of how well recieved it will be. We believed that other factors could also help us better understand what drives user ratings. We added the following features:

1. **Nutritional Information (e.g., `calories`, `sodium_pdv`, `sugar_pdv`)**:  
   There is two side here. Users care about health when eating, but also extra sugar, salt, and calories can make a recipe taste better. For example, a dish that is lower in sodium may be more appealing to health-conscious consumers, that being said the average person might like recipes with lots of salt more.

2. **Description Length (`descriptionLen`)**:  
   The length and detail of the recipe description might affect the users expectations and understanding of the food they are making. A more longer and more detailed description might be linked to higher ratings. There also may be a factor at play where the people who spend more time describing their recipes put more time into the recipes themselves.

3. **Time-Based Groupings (`date_group`)**:  
   Over time, user tastes and Food.com’s standards may shift. Grouping recipes by submission date captures trends related to time. The food most popular a while ago, is similar to the food most popular today, but certainly not the same. Users may also have given higher reviews on average at different points in time.  By acknowledging time related patterns, we align our model with the reality that the data was generated across different time periods,  potentially shaping ratings.

4. **Non-Linear Transformations (Polynomial Features of `n_steps`)**:  
   The relationship between the complexity of a recipe (as measured by the number of steps) and rating is unlikely to be strictly linear. For example very simple recipes and very elaborate ones may be targetted at different audiences. Introducing polynomial features allows the model to capture more nuanced, relationships that we might otherwise miss. 

We added features and transformations that we thought might meaningfully influence a recipe rating. We didn't know before wether they would improve accuracy or not. Our goal was to think about and base our model off real humans recipe considerations and not just to optimize metrics blindly.

---

**Modeling Algorithm and Hyperparameter Selection:**

For our final model, we used a **Ridge Regression** model. Ridge is a form of linear model that includes a regularization penalty on large coefficients, helping prevent overfitting and improving the model’s ability to generalize to new data.

- **Hyperparameters Tuned:**  
  We focused on the `alpha` parameter for Ridge Regression, which controls the strength of regularization. Larger `alpha` values place more emphasis on shrinking coefficients toward zero, potentially smoothing out complex patterns but reducing variance.

- **Hyperparameter Selection Method:**  
  We used **GridSearchCV** to try a range of `alpha` values. GridSearchCV performed cross-validation for each candidate `alpha`, helping us identify what would minimize the mean squared error.

- **Best Hyperparameters and Model Architecture:**  
  The best-performing `alpha` was selected from a range of powers of two. Including regularization improved the model’s stability and performance.

To conclude our analysis, we trained our model on all the data once it was properly tuned to maximize its performance on new data in the future.

---

**Performance Improvements Over the Baseline:**

Our baseline model (using just `n_steps` and `n_ingredients`) had an MSE of roughly 0.316. After adding meaningful features and fine-tuning hyperparameters with Ridge Regression, we achieved a small improved MSE,  of 0.314 on the test set. This is a small change, but there are still a few improvements:

1. **Stability and Robustness:**  
   Our final model incorporated features that represent user behaviors, nutritional preferences, and temporal factors, making the model’s reasoning more realistic and based in many factors and less dependent on the two previous features along.

2. **Added Regularization:**  
   By adding regularization we showed that the model’s performance is not just a training set fluke. We now have more confidence that the model is not overfitted and can handle future recipes well.

3. **Learning Experience:**  
   By identifying that adding these features and transformations alone, only produce only small improvements we learned that rating predictions is complex and challenging. In the futre this serves as a springboard for ideas to better refine our model and hone in on what may drive the data.



In summary, while our final model’s performance improvement may be modest in terms of MSE, the approach is now more attuned to the real-world processes behind recipe creation and user feedback. Even though there is plenty of room for potential improvement, the final model, has more of the right ingredients to predict what makes a recipe truly shine.

