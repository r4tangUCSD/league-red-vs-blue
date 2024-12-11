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

*teams*

'| gameid             | teamname    | side   |   result | pick1        | pick2    | pick3   | pick4   | pick5      | ban1     | ban2     | ban3         | ban4      | ban5      |   goldat10 |   killsat10 |   totalgold |   kills |   gamelength | league   | datacompleteness   | picks                                              |\n|:-------------------|:------------|:-------|---------:|:-------------|:---------|:--------|:--------|:-----------|:---------|:---------|:-------------|:----------|:----------|-----------:|------------:|------------:|--------:|-------------:|:---------|:-------------------|:---------------------------------------------------|\n| 10660-10660_game_1 | LNG Esports | Blue   |        0 | Kalista      | Senna    | Orianna | Maokai  | Aatrox     | Akali    | Nocturne | K\'Sante      | Lee Sin   | Wukong    |        nan |         nan |       49907 |       3 |         1886 | DCup     | partial            | [\'Kalista\' \'Senna\' \'Orianna\' \'Maokai\' \'Aatrox\']    |\n| 10660-10660_game_1 | Rare Atom   | Red    |        1 | Renata Glasc | Varus    | LeBlanc | Rell    | Rumble     | Poppy    | Ashe     | Neeko        | Vi        | Jarvan IV |        nan |         nan |       61737 |      16 |         1886 | DCup     | partial            | [\'Renata Glasc\' \'Varus\' \'LeBlanc\' \'Rell\' \'Rumble\'] |\n| 10660-10660_game_2 | LNG Esports | Blue   |        0 | Neeko        | Bel\'Veth | Kennen  | Senna   | Tahm Kench | Nocturne | Udyr     | Renata Glasc | Nautilus  | Lee Sin   |        nan |         nan |       49552 |       3 |         1911 | DCup     | partial            | [\'Neeko\' "Bel\'Veth" \'Kennen\' \'Senna\' \'Tahm Kench\'] |\n| 10660-10660_game_2 | Rare Atom   | Red    |        1 | Kalista      | Jax      | LeBlanc | Rell    | Jarvan IV  | Poppy    | Ashe     | Rumble       | Tristana  | Lucian    |        nan |         nan |       63623 |      17 |         1911 | DCup     | partial            | [\'Kalista\' \'Jax\' \'LeBlanc\' \'Rell\' \'Jarvan IV\']     |\n| 10660-10660_game_3 | LNG Esports | Blue   |        1 | Neeko        | Caitlyn  | Lux     | Jax     | Bel\'Veth   | Rell     | Nocturne | Tristana     | Jarvan IV | Rumble    |        nan |         nan |       51091 |      21 |         1324 | DCup     | partial            | [\'Neeko\' \'Caitlyn\' \'Lux\' \'Jax\' "Bel\'Veth"]         |'

*players*

'| gameid             | teamname    | side   | playername   |   result | champion   |   goldat10 |   killsat10 |   totalgold |   kills |   gamelength | league   | datacompleteness   |\n|:-------------------|:------------|:-------|:-------------|---------:|:-----------|-----------:|------------:|------------:|--------:|-------------:|:---------|:-------------------|\n| 10660-10660_game_1 | LNG Esports | Blue   | Zika         |        0 | Aatrox     |        nan |         nan |       11083 |       1 |         1886 | DCup     | partial            |\n| 10660-10660_game_1 | LNG Esports | Blue   | Weiwei       |        0 | Maokai     |        nan |         nan |        8636 |       0 |         1886 | DCup     | partial            |\n| 10660-10660_game_1 | LNG Esports | Blue   | Scout        |        0 | Orianna    |        nan |         nan |       10743 |       0 |         1886 | DCup     | partial            |\n| 10660-10660_game_1 | LNG Esports | Blue   | GALA         |        0 | Kalista    |        nan |         nan |       12224 |       2 |         1886 | DCup     | partial            |\n| 10660-10660_game_1 | LNG Esports | Blue   | Mark         |        0 | Senna      |        nan |         nan |        7221 |       0 |         1886 | DCup     | partial            |'

***Univariate Analysis***

I performed univariate analysis on the distributions of a few columns. For example, here's a historgram I created that shows the the distribution of total kills. This data is from the "teams" dataframe, so each value represents the cumulative number of kills all 5 players on a team got.

<iframe
  src="assets/kill_histogram.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
We see that this distribution is unimodal and looks relatively normal. It is slightly right skewed, which makes sense because if a match lasts a very long time, players will have more kills at the end.

***Bivariate Analysis***

In this scatterplot I plot each champion's pickrate with their banrate. Before, I theorized that if a champion is very powerful, it will have both a high pickrate (because teams want to uitilize this champion) and a high banrate (because teams don't want to deal with a powerful opponent). Here we see that banrate and pickrate are, indeed, positively correlated.

<iframe
  src="assets/banrate_pickrate_scatter.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

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

---

## Hypothesis Testing

Null Hypothesis: On average, the average number of blue side kills is equal to the average number of red side kills

Alternate Hypothesis: On average, the average number of blue side kills is greater than the average number of red side kills

Test Statistic: Difference of means: mean number of blue side kills - mean number of red side kills.

I perform a permutation test because I want to explore whether the two sides come from the same distribution.

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

-"*side*:" A nominal variable that is either red or blue. As explored earlier, the blue side is more likely to win, so I include it as a feature that could help predict a team's victory. Since this a nominal variable, i use OneHotEncoder().

-"*goldat10*:" A quantative variable. A team in League of Legends needs money to buy items and upgrade their champion's stats. If a team has a lot of gold early on in a game, it follows that they'll probably have more resources to help them win.

-"*killsat10*:" A quantative variable. In League of Legends, kills temporarily take enemy champions out of the game and also awards a player bonus gold. Like "goldat10", this variable probably also is correlated with 'result.'

Baseline Model Accuracy: 0.6337142857142857, which is better than blindly guessing but not good enough to make meaningful predictions.

---

## Final Model

My final model employed a RandomForestClassifier() instead of LogisticRegression(). In addition to the features in the bassline model, all of which I kept without modifying, my final model had two more features:

-"*Average champ winrate and average champ pickrate*:" Two quantitative variables. I knew from the beginning I wanted to include the champion picks into the prediction because they were a key element of the game that can heavily influence whether a team wins or not. Picking strong champions and creating synergistic team compositions should set a team up for success. Even though in theory, this should justify using champion picks as a feature, in practice, they weren't as effective as I'd hoped. I greatly struggled in encoding champion pick data efficiently and getting meaningful results.

In the end, I don't think that any of the numorous encoding methods I tried (such has OneHotEncoding() every champion - which led to 160+ columns) had a super big effect on accuracy of model. My final model first extracts the average winrate and pickrate of each champion using data from the "players" database. It takes in the 5 champions that a team picked and maps each champion to it's corresponding winrate and pickrate. Then, it averages out both values, resulting in two numbers that represent how often (on average) these champions are picked and played. I implemented this with a custom function and a FunctionTransformer().

-"*Team winrate*:" Another quantitative variable similar to the one above. I used data from the "teams" dataframe to create a list of winrates for each team that competed in 2024. Then, I mapped the "teamname" column to its corresponding winrate using a custom function and a FunctionTransformer(). I thought this feature would help predict the outcome of a match because if a team historically performs better than other teams, they're more likely to win any given match.

To find the best hyperparameters for the random forest, I used a GridSearchCV with 5-fold cross validation.


The hyperparameters that worked the best were:

    'model__max_depth': 10

    'model__min_samples_leaf': 15

    'model__min_samples_split': 20

    'model__n_estimators': 150


In the end, my model only made marginal improvements:

Best Cross-Validation Score: 0.6720714285714287

Final accuracy: 0.678

The fact that this model didn't improve that much could mean a few things. First, it might mean that I haven't yet found a way to effectively encode the champion pick data as a feature. Perhaphs there is more pre-game and early-game match data, like "first_blood" (the team that gets the first kill), that could aid in predicting the result. It could also mean that, at the highest levels of professional League of Legends, that players are skilled enough to often turn the tide of a losing game. Perhaphs professional esports players have the agency and game knowledge to make effective comebacks and counter strong champions, which would make our features less impactful in predicting the result of a match.

---

## Fairness Analysis

League of Legends has 5 "tier one" regional professional leagues: 

| Abbreviation       | League                                     |
|--------------------|--------------------------------------------------|
| LCK            | League of Legends Champions Korea	             |
| LPL           | League of Legends Pro League           |
| LEC               | League of Legends EMEA Championship         |
| LTA         | League of Legends Championship of The Americas  |
| LCP             | Outcome of the game (win or loss)                |

I'm interested in whether or not my model performs better for tier one matches compared to all other leagues.

Null Hypothesis: Our model is fair. Its precision for tier one matches and other matches are roughly the same, and any differences are due to random chance.

Alternative Hypothesis: Our model is unfair. Its precision for tier one matches is higher than its precision for other matches.

Test statistic: other_precision - t1_precision. I guessed tier one teams would have better predictions because maybe the highest level of play had the most consistent factors. Contrary to my belief, the other leagues had a slightly higher precision.

Tier-One Precision: 0.7150127226463104

Other Leagues Precision: 0.7303229414231791

Observed Precision Difference: 0.015310218776868667

I perform a permutation test because I want to see if the model predicts as though these two groups come from the same distribution.

P-value: 0.106

We fail to reject the null hypothesis.

