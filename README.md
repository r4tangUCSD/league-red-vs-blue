# Predicting victories in League of Legends
by Ryan Tang (r4tang@ucsd.edu)

---

## Introduction

League of Legends is a multiplayer online battle arena (MOBA) that is widely considered one of the greatest games of all time. In a match of league of legends, two teams of five players battle to fight past their opponents' defenses and destroy the enemy "Nexus." To do this, each player collects gold, buys items, and fights both monsters and other players. Each player controls a unique "champion," each with their own unique skill and abilities. As of December 2024, there are 169 champions released. 

League of Legends is also the world's biggest esport. International and regional leagues and tournaments are held every year; these events sell out stadiums and attract hundreds of millions of livestream viewers. In this project, I'll be analyzing match data from many different leagues through 2024. This data was provided by Oracle's Elixir (https://oracleselixir.com/tools/downloads). With this data, I aim to predict whether a given team will win or lose a match using only pre-game and early-game statistics. Through this process, I explore which factors most greatly affect the outcome of a match.

The raw dataset has 116124 entries and 161 columns, corresponding to 9667 total matches. Each match has 12 entries corresponding to it - 10 for each player in the match, and 2 for each team's aggregate statistics. Some of these columns (such as "champion") are only relevant for entries about individual players. Others, like "pick(1-5)," are only relevant for whole teams.

| Column Name        | Description                                      |
|--------------------|--------------------------------------------------|
| gameid             | Unique identifier for each game                  |
| teamname           | The name of the team playing the game            |
| side               | Side of the team in the game (blue or red)       |
| playername         | Username of the player                |
| result             | Outcome of the game (win or loss)                |
| champion           | Champion picked by a player                 |
| pick1              | First champion pick by the team                  |
| pick2              | Second champion pick by the team                 |
| pick3              | Third champion pick by the team                  |
| pick4              | Fourth champion pick by the team                 |
| pick5              | Fifth champion pick by the team                  |
| ban1*              | First champion ban by the team                   |
| ban2               | Second champion ban by the team                  |
| ban3               | Third champion ban by the team                   |
| ban4               | Fourth champion ban by the team                  |
| ban5               | Fifth champion ban by the team                   |
| goldat10           | Total gold earned by at 10 minutes      |
| killsat10          | Number of kills by at 10 minutes        |
| totalgold          | Total gold earned at the end of the game |
| kills              | Total number of kills at the end of the game    |
| deaths             | Total number of deaths at the end of the game   |
| gamelength         | Duration of the game in seconds                  |
| league             | The league in which the game took place (e.g., LCS, LEC, etc.) |
| datacompleteness   | Indicates whether the data for the game is complete|

\* In professional league matches, before players start picking their champions, they get to ban champions they don't want their enemies to pick. Each player can ban one champion - that's why teams have 5 champion picks and 5 bans.

---

## Data Cleaning and Exploratory Data Analysis

***Data cleaning***

Describe, in detail, the data cleaning steps you took and how they affected your analyses. The steps should be explained in reference to the data generating process. Show the head of your cleaned DataFrame (see Part 2: Report for instructions).

***Univariate Analysis***

Embed at least one plotly plot you created in your notebook that displays the distribution of a single column (see Part 2: Report for instructions). Include a 1-2 sentence explanation about your plot, making sure to describe and interpret any trends present. (Your notebook will likely have more visualizations than your website, and that’s fine. Feel free to embed more than one univariate visualization in your website if you’d like, but make sure that each embedded plot is accompanied by a description.)

***Bivariate Analysis***

Embed at least one plotly plot that displays the relationship between two columns. Include a 1-2 sentence explanation about your plot, making sure to describe and interpret any trends present. (Your notebook will likely have more visualizations than your website, and that’s fine. Feel free to embed more than one bivariate visualization in your website if you’d like, but make sure that each embedded plot is accompanied by a description.)

***Interesting Aggregates***

Embed at least one grouped table or pivot table in your website and explain its significance.

---

## Assessment of Missingness

***NMAR Analysis***

State whether you believe there is a column in your dataset that is NMAR. Explain your reasoning and any additional data you might want to obtain that could explain the missingness (thereby making it MAR). Make sure to explicitly use the term “NMAR.”

***Missingness Dependency***

Present and interpret the results of your missingness permutation tests with respect to your data and question. Embed a plotly plot related to your missingness exploration; ideas include:
• The distribution of column 
Y
 when column 
X
 is missing and the distribution of column 
Y
 when column 
X
 is not missing, as was done in Lecture 8.
• The empirical distribution of the test statistic used in one of your permutation tests, along with the observed statistic.

Here's what a Markdown table looks like. Note that the code for this table was generated _automatically_ from a DataFrame, using

```py
print(counts[['Quarter', 'Count']].head().to_markdown(index=False))
```

| Quarter     |   Count |
|:------------|--------:|
| Fall 2020   |       3 |
| Winter 2021 |       2 |
| Spring 2021 |       6 |
| Summer 2021 |       4 |
| Fall 2021   |      55 |

---

## Hypothesis Testing

Clearly state your null and alternative hypotheses, your choice of test statistic and significance level, the resulting 
p
-value, and your conclusion. Justify why these choices are good choices for answering the question you are trying to answer.

Optional: Embed a visualization related to your hypothesis test in your website.

Tip: When making writing your conclusions to the statistical tests in this project, never use language that implies an absolute conclusion; since we are performing statistical tests and not randomized controlled trials, we cannot prove that either hypothesis is 100% true or false.

---

## Framing a Prediction Problem

Clearly state your prediction problem and type (classification or regression). If you are building a classifier, make sure to state whether you are performing binary classification or multiclass classification. Report the response variable (i.e. the variable you are predicting) and why you chose it, the metric you are using to evaluate your model and why you chose it over other suitable metrics (e.g. accuracy vs. F1-score).

Note: Make sure to justify what information you would know at the “time of prediction” and to only train your model using those features. For instance, if we wanted to predict your final exam grade, we couldn’t use your Final Project grade, because the project is only due after the final exam! Feel free to ask questions if you’re not sure.

---

## Baseline Model

Describe your model and state the features in your model, including how many are quantitative, ordinal, and nominal, and how you performed any necessary encodings. Report the performance of your model and whether or not you believe your current model is “good” and why.

Tip: Make sure to hit all of the points above: many projects in the past have lost points for not doing so.

---

## Final Model

State the features you added and why they are good for the data and prediction task. Note that you can’t simply state “these features improved my accuracy”, since you’d need to choose these features and fit a model before noticing that – instead, talk about why you believe these features improved your model’s performance from the perspective of the data generating process.

Describe the modeling algorithm you chose, the hyperparameters that ended up performing the best, and the method you used to select hyperparameters and your overall model. Describe how your Final Model’s performance is an improvement over your Baseline Model’s performance.

Optional: Include a visualization that describes your model’s performance, e.g. a confusion matrix, if applicable.

---

## Fairness Analysis

Clearly state your choice of Group X and Group Y, your evaluation metric, your null and alternative hypotheses, your choice of test statistic and significance level, the resulting 
p
-value, and your conclusion.

Optional: Embed a visualization related to your permutation test in your website.


