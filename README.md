# Metis Project 2: Linear Regression / Tennis Analytics

#### Goal: Identify statistics that have the biggest impact in predicting a men’s singles tennis player’s career high ranking

## 1. Data Acquisition via Web Scraping (Selenium, BeautifulSoup)

### 1.1 Scrape 2019 YE Rankings and 2018 Stats Data

Website: https://www.ultimatetennisstatistics.com/season?season=2019

Using Selenium webdriver and setting a fake user agent (to prevent from getting blocked), I instructed webdriver to open up the webpage above in a new Google Chrome window.  Then I directed webdriver to click on "Rankings," then the '15' dropdown (number of results), and finally click on 'All' to show the entire list of rankings for the 2019 men's singles tennis season.  Then I used BeautifulSoup to extract the page's source code into a soup object.  Following that I used three functions that I created (that are defined in the 'scraping_cleaning_data_functions.py' file).
   1. parse_headers to get a list of the column names.
   2. parse_table_data to get a list of lists, each list containing the data of one row in the table on the website
   3. create_df_stats to take the outputs of the two previous functions to create a dataframe that looks like the table on the website.

Then I cleaned up the 'Rank' and 'Points' columns/values using two more functions that I created (which are also defined in the 'scraping_cleaning_data_functions.py' file).  Then I pickled this dataframe for easier access in the future, renamed a couple of columns, and created a new dataframe df_2019_rankings_sum with just three columns: Name, Rank_YE2019, BestRank.

After that I directed webdriver to close the Google Chrome window, and open a new window with the following url: https://www.ultimatetennisstatistics.com/season?season=2018.

This time I directed webdriver to click on "Statistics."  The first statistic that shows up is "Aces."  I then told webdriver to click on the '15' dropdown and select 'All.'  Then I used BeautifulSoup to extract the source code, applied the same three functions as above to get the data in a dataframe matching the table on the website, and pickled the dataframe.

For the next statistic, I directed webdriver to click on the 'Category' dropdown and select '1st Serve Won %.'  I repeated the steps above extracting the source code, getting the data into a dataframe, and pickling the dataframe.  Following that, I merged this dataframe with the previous dataframe (containing 'Aces."

I repeated this process for ten more statistics (see below), each time directing webdriver to click on the statistic, extracting the source code with BeautifulSoup, creating a dataframe with my functions, pickling the dataframe, and merging this latest dataframe with the dataframe containing previous statistics.  
    - 2nd Serve Won %
    - 1st Srv Return Won %
    - 2nd Srv Return Won %
    - Services Games Won %
    - Return Games Won %
    - Upsets Against %
    - Break Points Won %
    - Scrape Tie-Breaks Won %
    - Points Dominance %
    - Gms. to Matches Ov.-Perf.

Finally, I merged the dataframe with all the statistics (df_2018_stats) with the dataframe containing the next year's rankings (df_2019_rankings_sum) to create a master dataframe (df_2018_master) and pickled it.

### 1.2 Scrape 2018 YE Rankings and 2017 Stats Data

Same process as above, summarized below:
   1. Scrape 2018 rankings and summarize into df_2018_rankings_sum dataframe
   2. Scrape 2017 statistics, adding each statistic one by one to df_2017_stats dataframe
   3. Merge two dataframes above to create df_2017_master dataframe and pickle it

### 1.3 Scrape 2017 YE Rankings and 2016 Stats Data

Same process as above, summarized below:
   1. Scrape 2017 rankings and summarize into df_2017_rankings_sum dataframe
   2. Scrape 2016 statistics, adding each statistic one by one to df_2016_stats dataframe
   3. Merge two dataframes above to create df_2016_master dataframe and pickle it

### 1.4 Scrape 2016 YE Rankings and 2015 Stats Data

Same process as above, summarized below:
   1. Scrape 2016 rankings and summarize into df_2016_rankings_sum dataframe
   2. Scrape 2015 statistics, adding each statistic one by one to df_2015_stats dataframe
   3. Merge two dataframes above to create df_2015_master dataframe and pickle it

### 1.5 Combine all dataframes

Finally I renamed 'Rank_YE...." column in all the master dataframes to 'Rank_YE_NextSeason,' verified that all the column names are the same across all four dataframes, concatenated them into a single dataframe, and picked it. 


## 2. Data Cleaning and EDA / Plotting

First I cleaned the data by dropping all rows with any null values and converting all columns that would be used later for modeling to float64 data types.  I also removed an outlier (i.e. tennis player who was injured) and cleaned up the column names.  Then I created a correlation matrix and a heatmap using the Spearman method.  I chose Spearman correlation vs. Pearson correlation (the default) because the former includes non-linear relationships between variables, while the latter only includes linear relationships.

I also conducted a pairplot of all the continuous variables/features.  Based on the results, I decided to feature engineer a few new polynomial features with degree 2.  Further, I plotted histograms of all the variables (using desc_num_feature, a function I created which is defined in 'data_EDA_modeling_function.py' file) and box/whisker plots of all the columns.  Noticing that both 'aces' and 'bestrank" (target) have right skewed distributions, I created two additional features taking the logarithm of the original columns. 


## 3. Data Modeling / Linear Regression

### 3.1 Baseline Model(s)

First I fit a linear regression model using statsmodels OLS.  Then I selected the features with p-values lower than 0.15 (since a low p-value suggests there is a high likelihood that that a particular feature has a meaningful impact/correlation with the target).  I narrowed down the list further and removed a couple of features because they were directly correlated with another feature, each time keeping the feature with the lower p-value.  

Then I created a dataframe with all my selected features, and a separate dataframe with my target.  I split both into cross-validation/test based on an 80/20 split.  After that I performed cross-validation with 5 splits to test three different models:
    1. Linear regression
    2. Ridge regression (alpha = 0.01)
    3. LASSO regression (alpha = 0.01)
  
I selected r squared and RMSE as my scoring metrics for comparing the three models, calculating the mean values for both metrics across the five folds.  Based on this, the LASSO model performed the best (highest r squared and lowest RMSE) so I continued to iterate on different LASSO models.

### 3.2 Model 4 - Hyperparameter Tuning

The next step was to combine hyperparameter tuning with cross-validation to find the optimal alpha, which turned out to be approximately 0.013667.  Then I performed cross-validation again with this alpha value for only the LASSO model and scored the train and validation data.

### 3.3 Future Models

For the next model, I removed 'break_points_won_pct' because a positive coefficient does not make sense, leveraging my domain knowledge.  I performed cross-validation and scored the model again.  This model led to a slightly higher r squared and lower RMSE for validation data compared to the previous model.

Finally, I removed "aces_squared" because it had a zero coefficient in 2/5 folds in previous LASSO model.  I did cross-validation again and scored the model, which led to further small improvements in r squared and RMSE.

### 3.4 Final Model

Finally, I fit the model on the entire 80% cross-validation data and scored it on the test data.  The RMSE on test data was approximately 11.7678, or almost 22% lower than the RMSE on the first LASSO model!
 
## 4. Post-Modeling / Model Accuracy / Residuals

### 4.1 Test - create dataframe, analyze actual vs. model predictions, look at residuals
I first created a dataframe df_tennis_model_test based on x_test, y_test, and the final model's predictions (y_test_pred).  Then I created a dataframe df_tennis_players_model first by filtering to include only the indices in x_test, and then filtering columns to include only the columns/features included in the final model, plus other metadata columns (e.g. name, season).  After spot-checking a couple of rows to make sure the indices and column values matched between df_tennis_model_test and df_tennis_players_model, I cleaned up the latter to only include a few columns and merged the two dataframes.  Then I calculated residuals and plotted them. 

The next step was to take a deeper dive into the residuals.  In particular, I was curious about 'undervalued' players, i.e. players where the model predicted a higher rank (or lower number) than the players' actual career high ranking.  I found five such players in the test data.  One in particular, John Millman of Australia, caught my attention immediately, since he had upset 20-time Grand Slam winner Roger Federer (who many tennis fans consider to be the Greatest of All Time, or GOAT) at the US Open in 2018.

I also wanted to take a closer look at my model's accuracy on the test data.  Overall, just under 50% of predictions were within 5 ranking points, and just over 70% of predictions were within 10 ranking points.

### 4.2 Cross-Validation - create dataframe, analyze actual vs. model predictions
Similar to above, I created a dataframe df_cv_model based on x_crossval, y_cv, and model's predictions (i.e. y_cv_pred).  Then I created another dataframe df_cv_players, filtering to include only the indices in x_crossval, and then filtering columns to include only the columns/features included in the final model, plus other metadata columns (e.g. name, season).  After spot-checking a couple of rows to make sure the indices and column values matched between df_cv_model and df_cv_players, I cleaned up the latter to include only a few columns and merged the two dataframes.  Then I calculated residuals.

Again, I was curious about 'undervalued' players.  Among the two I found, one, Hyeon Chung of South Korea, really stood out because he had upset 12-time Grand Slam Winner Novak Djokovic at the Australian Open in 2018, which was a major, shocking upset in the tennis world at that time.

### 4.3 Interpreting coefficients and impact on target (bestrank)
First I found the standard deviations of the original four features (in my final model).  Then I calculated the coefficients in the original scales of the four features by dividing by these standard deviations, and organized the data in the dataframe model_features.  

Next I wanted to find the intercept of the equation/model with the pre-scaled coefficients and features, and the target (log_bestrank).  I used the first row of data in x_test, my final model, and linear algebra to find the intercept of approximately 7.8863.  Now I had figured out the following equation, where the inputs/features were the original pre-scaled values:
log_bestrank = 
7.886305985188102 
- 0.02258998 * first_srv_return_won_pct 
- 0.01485758 * upsets_against_pct
- 1.93878343 * games_to_matches_perf_ratio 
- 2.15198387 * points_dominance_squared 

Then I created a function final_model_pred_bestrank that would take as inputs an array of values (for the four features), coefficients (pre-scaling), and the intercept and return the predicted value for bestrank (not log_bestrank).  I used this function to compare the predicted bestrank (from this linear algebra method) to the predicted bestrank (from my model) to confirm that the function (and the equation above) are correct.

Finally, I took a sample data point (x1) and created four new sample data points (x2, x3, x4, x5), each one incrementing a specific feature/column.  I used the function above final_model_pred_bestrank to calculate the predicted best rank, and then calculated the difference between the predicted values for x2 to x5 vs. predicted value for x1 (i.e. change in ranking for incremental changes in features).

## 5. Appendix - Other Models

### Models 7 and 8 - Polynomial
I started with the features from my baseline model (degree 1 only).  Then I created Polynomial Features of degree 3 with those features, which resulted in 84 total features.  Following that, I used LASSO regularization to narrow it down to 8 features with non-zero coefficients. Then I created a dataframe with those 8 features, and another dataframe with just the target.  I performed cross-validation and scored the model on my train and validation data.  

Then I dropped a couple of features that had coefficients of zero in at least a couple of the folds above, performed cross-validation, and scored the model again.  Comparing models 6 (final model I chose) vs. model 8, although model 8 had a higher r squared score and lower RMSE for validation compared to model 6, model 8 was significantly more complex and less interpretable with its polynomial and interaction terms.  Hence ultimately I chose the simpler model 6 as my final model.


## 6. Author

Matt Wang

#### Thanks for reading!

<img src="https://storage.needpix.com/rsynced_images/tennis-3552164_1280.jpg" width="550">