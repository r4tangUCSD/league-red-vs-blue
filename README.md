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
| gamelength         | Duration of the game in seconds                  |
| league             | The league in which the game took place (e.g., LCS, LEC, etc.) |
| datacompleteness   | Indicates whether the data for the game is complete|

\* In professional league matches, before players start picking their champions, they get to ban champions they don't want their enemies to pick. Each player can ban one champion - that's why teams have 5 champion picks and 5 bans.

---

## Data Cleaning and Exploratory Data Analysis

***Data cleaning***

First, I begin my data cleaning process by only keeping the relevant columns listed above. Later in this project, I use the columns "goldat10" and "killsat10" to help predict the result of a match. Because of this, it doesn't make sense to keep entries that last less than 10 minutes, so I filter those rows out using the "gamelength" column. As mentioned, earlier, this dataset includes rows the correspond to players and rows that correspond to whole teams. These different kinds of entries have different types of data encoded in them, so the next step I perform is to separate my data into two dataframes: "players" and "teams."

Next, I examine each column for missingness. I find that most columns don't have any missing values, save for a few:
-the columns pick1-pick5 occasionaly have NaN values. When this happens, an entire team's picks are all gone. I dropped these columns, since there weren't many.
-similarly, the ban1-ban5 columns occasionally have NaN values, but this time it doesn't make sense to remove them because players can choose not to ban a champion in any given match.
-the goldat10 and killsat10 columns have missing values. I explore this more in depth in part 3.

Below are the heads of both the "players" dataframe and the "teams" dataframe.

***Univariate Analysis***

I performed univariate analysis on the distributions of a few columns. For example, here's a historgram I created that shows the the distribution of total kills. This data is from the "teams" dataframe, so each value represents the cumulative number of kills all 5 players on a team got.

We see that this distribution is unimodal and looks relatively normal. It is slightly right skewed, which makes sense because if a match lasts a very long time, players will have more kills at the end.

***Bivariate Analysis***

In this scatterplot I plot each champion's pickrate with their banrate. Before, I theorized that if a champion is very powerful, it will have both a high pickrate (because teams want to uitilize this champion) and a high banrate (because teams don't want to deal with a powerful opponent). Here we see that banrate and pickrate are, indeed, positively correlated.

***Interesting Aggregates***

The arena that League of Legends matches takes place in, "Summoner's Rift," is mostly symmetrical; however, there are a few key differences that could give one side a hypothetical advantage over another. My good friend Parker, a league fanatic, tells me that the Blue side, which starts in the southwest portion of the arena, has a better chance of winning compared to the Red side, which starts in the north eastern corner.

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


