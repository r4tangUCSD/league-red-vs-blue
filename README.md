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

Lastly, I create a column "picks" in the teams dataframe which contains the same information as columns "pick(1-5)." I do this soley for my own convenience - in certain situations it is easy for me to perform transformations a column of lists rather than 5 separate columns.

Below are the heads of both the "players" dataframe and the "teams" dataframe.

***Univariate Analysis***

I performed univariate analysis on the distributions of a few columns. For example, here's a historgram I created that shows the the distribution of total kills. This data is from the "teams" dataframe, so each value represents the cumulative number of kills all 5 players on a team got.

We see that this distribution is unimodal and looks relatively normal. It is slightly right skewed, which makes sense because if a match lasts a very long time, players will have more kills at the end.

***Bivariate Analysis***

In this scatterplot I plot each champion's pickrate with their banrate. Before, I theorized that if a champion is very powerful, it will have both a high pickrate (because teams want to uitilize this champion) and a high banrate (because teams don't want to deal with a powerful opponent). Here we see that banrate and pickrate are, indeed, positively correlated.

***Interesting Aggregates***

The arena that League of Legends matches takes place in, "Summoner's Rift," is mostly symmetrical; however, there are a few key differences that could give one side a hypothetical advantage over another. My good friend Parker, an avid League of Legends fan, tells me that the Blue side, which starts in the southwest portion of the arena, has a better chance of winning compared to the Red side, which starts in the northeastern corner. 

To explore this, I group the data by the "side" column and see if there's truth to my friend's rumor:

It turns out he's right: the Blue side gets (on average), more kills, more gold, and has an overall higher winrate. I explore this further in Part 4.


---

## Assessment of Missingness

***NMAR Analysis***

I believe that the columns ban1 - ban5 are all Not Missing At Random (NMAR) because a player isn't required to ban a champion. Though the vast majority of players do ban a champion, for whatever reason some players may choose not to select a champion to ban. The reason why a player would do this isn't clear, because there isn't really any situation where a player would be better off not banning a champion rather than banning one. 

***Missingness Dependency***

For missingness dependency, I test the "goldat10" column. This column is always NaN when "killsat10" is also NaN, so I'm testing both columns at once. I test this column against "gamelength", "league", "deaths", and "result."

*League*

Null Hypothesis: The distribution of the "league" column is the same regardless of whether goldat10 is missing or not.

Alternate Hypothesis: The distribution of "league" column is different when goldat10 is missing compared to when it's not missing.

Observed TVD: 0.98498801564274
P-value: 0.0

We reject the null hypothesis, since the p-value < 0.05. It is likely unlikely that every league has the same amount of missing values in the goldat10 column.

*Result*

Null Hypothesis: The "result" values (win/loss) are the same regardless of whether "goldat10" is missing or not.

Alternative Hypothesis: The "result" values (win/loss) differ depending on whether "goldat10" is missing or not.

Observed TVD: 0.0
P-value: 0.529

We fail to reject the null hypothesis, since the p-value > 0.05.

*Imputing missing values:* 
To account for the missing goldat10 and killsat10 columns, I impute them using the related "totalgold" and "kills" columns. I find the average ratio between goldat10 and totalgold, and use it to estimate missing goldat10 values with their totalgold values. I use the same method to impute "killsat10"

```py
print(counts[['Quarter', 'Count']].head().to_markdown(index=False))
```

---

## Hypothesis Testing

Null Hypothesis: On average, the average number of blue side kills is equal to the average number of red side kills

Alternate Hypothesis: On average, the average number of blue side kills is greater than the average number of red side kills

Test Statistic: Difference of means: mean number of blue side kills - mean number of red side kills.

I perform a permutation tests because we want to explore whether the two sides come from the same distribution.

Observed Difference in Means: 0.6323428571428575

P-value: 0.0

We reject the null hypothesis, since the p-value < 0.05. We conclude that the average number of blue side kills is higher than the average number of red side kills.


---

## Framing a Prediction Problem

My goal is predict the result of a match given only pre-game and early-game data about a team. This is a binary classification problem since there are only two outcomes: win and lose. It follows that the response variable I am trying to predict is the "result" column, where wins are represented with 1 and losses are represented with 0.

Since I'm trying to predict the outcome of a match before it ends, I give myself only information I would have access to 10 minutes into a match. This means I can use columns like "goldat10" and "killsat10", but not match statistics from the end of a match, like "totalgold" and "kills." I also use information from before the match starts, such as "teamname" and the champions picks ("pick1" - "pick5").

To evaluate my model, I chose accuracy as a suitable metric. My dataset has equal numbers of positive and negative examples: for every team that wins, another team has lost. Since classes aren't imbalanced at all, accuracy won't be as misleading as it might be in other cases. Accuracy is also simple to interpret.


---

## Baseline Model

My baseline model uses Logistic Regression. These are the features in the baseline model:

-"side:" A nominal variable that is either red or blue. As explored earlier, the blue side is more likely to win, so I include it as a feature that could help predict a team's victory. Since this a nominal variable, i use OneHotEncoder().

-"goldat10:" A quantative variable. A team in League of Legends needs money to buy items and upgrade their champion's stats. If a team has a lot of gold early on in a game, it follows that they'll probably have more resources to help them win.

-"killsat10:" A quantative variable. In League of Legends, kills temporarily take enemy champions out of the game and also awards a player bonus gold. Like "goldat10", this variable probably also is correlated with 'result.'

 Report the performance of your model and whether or not you believe your current model is “good” and why.

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


