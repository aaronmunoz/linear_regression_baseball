The notebook used to generate most of the information for this writeup can be found [here](https://github.com/aaronmunoz/linear_regression_baseball/blob/master/Starting_Offense_Salaries/Batter%20Regression%20Analysis.ipynb)

This is a bit of an older project. If I were to tackle this again, I'd use an Autoregressive model.
____
# Using Linear Regression to Predict Salaries

The goal of this project was to predict salaries for starting batting players in Major League Baseball. In particular, I was curious to see how linear regression algorithms react to the rapid growth of salaries in baseball.

### Data ###
For this project I scraped yearly batting stats from Baseball Reference for every player with a
listed salary between 1985 and 2017. Both traditional counting stats such as such as Hits(**H**), Walks(**W**), and Home Runs(**HR**) was collected, as well as aggregation stats such as Batting Average(**BA**) and Slugging Percentage(**SLG**).
There were two conditions for removing outliers:
    1. Players with less than 400 at bats in a season were considered part time and dropped.
    2. The 1994 baseball season was cut short by a player strike. As a results, the statistics collected in that year are for less games than a typical season, so they were dropped.
    
The target value for this model is log(salary). MLB salaries have a very heavy positive skew, so for the purpose of this project we want to predict the general magnitude of salaries.

### Modeling ###
Correlation tables and pair plots were generated to inspect the relationship of my feature statistics with my target statistic. None of the Pearson correlation coefficient values computed were above 0.3. Most pair plots resembled normal distributions tiled to the right.

![](https://github.com/aaronmunoz/linear_regression_baseball/blob/master/random_br_files/correlations.png?raw=true)

I split my data 80-20 into validation and testing sets, and always used 5 fold cross validation when validating. I ran a simple linear model using On-base Plus Slugging(**OPS**) as the feature since it’s typically seen as the stat that tells the most about a player at a high level. The results weren’t very promising (**R2**=**0.06**), so I added age(**AGE**) of the player and the mean player salary(**LG_SALARY_MEAN**) for that year to the model, as well as polynomial features.

The result was much more promising (**R2**=**0.61**), so I decided to move forward with combinations of additional features in a LARS Lasso path to view feature importance. Surprisingly I found that Total Bases(**TB**) was a more important feature than **OPS**, so I swapped **TB** into the model for **OPS** (**TB** and **OPS** have a collinear relationship).

No additional features seemed to make a difference, so I did a final validation test of my model on statsmodels.OLS to view the p-values of my coefficients. High p-values for **TB, TB * AGE,** and **TB * LG_SALARY_MEAN** indicated that they should be dropped from my model.

Other statistics such as stolen base attempts and physical attributes such as weight were looked at, but didn’t add much value to the model. Regularization was tested with the remaining features, but didn’t seem to improve predictive ability.

The resulting list of features were **AGE, LG_SALARY_MEAN, TB2, AGE2, AGE * LG_SALARY_MEAN, LG_SALARY_MEAN2**. When fit to my 80 percent validation set, and tested against my holdout set, my model produced an **R2** score of **0.6386**. Not bad, however the resulting model coefficients don’t provide much meaningful information:

Stat | Regression Coefficient 
--- | --- 
**AGE** | 0.69181
**LG_SALARY_MEAN** | -2.78609
**TB2** | 0.00001
**AGE2** | -0.01557
**AGE \* LG_SALARY_MEAN** | 0.03016
**LG_SALARY_MEAN2** | 0.09426

The only baseball stat included in the model is much weaker than the rest of the features.

### Conclusion ###
Ultimately, this model isn’t very useful. I think the main issue is that a linear model isn’t the best suited tool for handling issue rapid salary growth throughout the last 30 years in baseball. Another issue is that baseball players are also paid based off of defense, and that information was not included in this model. If I were to do this project again, I’d spend more time finding a relationship better suited for a linear regression model than baseball salary.