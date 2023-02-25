# League of Legends Competitive Matches Exploratory Data Analysis

by Lucien Chen

## Introduction

It feels like everytime I introduce myself, the first question I receive is something along the lines of: "Wait, your name is Lucien? Like *Lucian* from *League of Legends*?" I always found this question to be strange, since the spelling and pronounciation are quite different.

Inspired by this story, and having played League and watched the competitive scene for over 10 years, I thought it would be interesting to take a dive into the stats of competitive matches, particularly those of *Lucian*.

For some context, League of Legends is an MMO, RTS game that is enjoyed by millions of players all around the world, and we are going to be exploring a dataset that contains information about competitive matches across many different regions and tiers to try to answer the question: 

__*Is Lucian a better carry than other ADCS?*__

Source: [data](https://oracleselixir.com/tools/downloads)

## Important Terminology

Throughout this notebook/website, there are going to be several abbreviations and other terms used by people in the League community. Here is a short list of terms that will be relevant to our analysis:

 - *League*: a shortened way to refer to League of Legends
 - *Champion*: a playable character in the game
 - *Lane*: the physical location of the battle field, can be Top, Jungle, Mid or Bot
 - *Role*: a predefined set of positions and objectives that certain champions are assigned to, etc. Top, Jungle, etc.
 - *AD*: short for Attack Damage, the stat that increases a champion's physical damage output
 - *ADC*: short for Attack Damage Carry, a champion that is played in the bot lane accompanied by a support 

## Cleaning the Data
Originally, the dataset had 149,232 rows and 123 columns. However, not every column is relevant since we are only investigating the relative strength of one champion. As such, I have selected certain columns that will be useful in our analysis. I also cleaned up some of the data since they weren't in the correct format, e.g Changing *datacompleteness* to be of Boolean data type when it was previously inputted as a str.

Here are the columns I kept for the analysis and what they represent:
 - **gameid**: unique identifer of each match
 - **datacompleteness**: a boolean representing whether or not the stats in the game are completely filled
 - **league**: identifier of the tier and region the game was played in, for example the LCK is the Tier 1 league for Korea
 - **side**: which half of the map did the player or team play on, Blue or Red
 - **position**: the position a champion played
 - **champion**: the champion played
 - **win**: whether or not the player or team won
 - **kills**: number of kills for each player or team
 - **deaths**: number of deaths for each player or team
 - **assists**: number of assists for each player or team
 - **damagetochampions**: how much damage each player or team dealt to enemy champions
 - **damageshare**: how much of each players damage makes up the total damage 
 - **earnedgold**: how much gold each player or team earned
 - **golddiffat10**: how much of a gold (dis)advantage a player or team has at 10 minutes, relative to their opponent
 - **xpdiffat10**: how much of an experience (dis)advantage a player or team has at 10 minutes, relative to their opponent

And here is what the first few rows of the cleaned up dataset look like:

| gameid                | datacompleteness   | league   | side   | position   | champion   | win   |   kills |   deaths |   assists |   damagetochampions |   damageshare |   earnedgold |   golddiffat10 |   xpdiffat10 |
|:----------------------|:-------------------|:---------|:-------|:-----------|:-----------|:------|--------:|---------:|----------:|--------------------:|--------------:|-------------:|---------------:|-------------:|
| ESPORTSTMNT01_2690210 | True               | LCK CL   | Blue   | top        | Renekton   | False |       2 |        3 |         2 |               15768 |     0.278784  |         7164 |             52 |          -44 |
| ESPORTSTMNT01_2690210 | True               | LCK CL   | Blue   | jng        | Xin Zhao   | False |       2 |        5 |         6 |               11765 |     0.208009  |         5368 |            485 |          432 |
| ESPORTSTMNT01_2690210 | True               | LCK CL   | Blue   | mid        | LeBlanc    | False |       2 |        2 |         3 |               14258 |     0.252086  |         5945 |            162 |           71 |
| ESPORTSTMNT01_2690210 | True               | LCK CL   | Blue   | bot        | Samira     | False |       2 |        4 |         2 |               11106 |     0.196358  |         6835 |            296 |          265 |
| ESPORTSTMNT01_2690210 | True               | LCK CL   | Blue   | sup        | Leona      | False |       1 |        5 |         6 |                3663 |     0.0647631 |         2908 |            528 |         -587 |


Now we want to see if ADCs actually fulfill their role as carries.
<iframe src="assets/position_damageshare.html" width=800 height=600 frameBorder=0></iframe>

This graph displays the percentiles of damageshares for each position. Notice that ADCs have the highest damage share at each breakpoint (ex. 25th percentile, median, max, etc.) which is a clear indicator that they are indeed the carries of their games.

We are also interested in if *Lucian* has a significant playrate. If he is picked in very few games, it may be an indication that he is only useful within certain situations which may not be representative of all games played.

<iframe src="assets/adc_pickrate.html" width=800 height=600 frameBorder=0></iframe>

It looks like *Lucian* has about a 5% pickrate. While that's not the highest pickrate compared to all the other ADCs, it is still quite a large number of games so we can proceed.

## Dealing with Missingness

We are trying to find metrics to quantify whether or not Lucian is a better ADC. We are interested in relative statistics, since something like overall damage dealt will naturally increase with longer games. It looks like there are a lot of missing values for golddiffat10 and xpdiffat10. Since either of these columns represent how well Lucian does relative his opponents we can just focus in on the missingness of just golddiffat10. Let's take a look at exactly how many missing values there are in the entire dataframe and try to figure out why that is

<iframe src="assets/missingness_tvd.html" width=900 height=600 frameBorder=0></iframe>

Here is the distribution of total variation distances using a permutation test, where we randomly shuffled the datacompleteness column. The TVDs seem to be low so let's plot our observed TVD of 1 to see how drastic the difference is.

<iframe src="assets/missingness_tvd_with_observed.html" width=900 height=600 frameBorder=0></iframe>

As we can see the likelihood of having a TVD that high is 0 and so the values in the golddiffat10 columns are likely not missing at random.

## Performing a Hypothesis Test

Let's finally answer our question:

__*Is Lucian a better carry than other ADCs?*__

We are going to measure ability to carry by revisiting winrate. If a champion wins more often than other champions, on average, there is a high likelihood that the champion is more effective in their role, i.e better at carrying.

Let's take a look at the win rates as well as how many games each champion appeared in:

| champion     |     mean |   count |
|:-------------|---------:|--------:|
| Aphelios     | 0.490725 |    4043 |
| Ashe         | 0.455446 |     303 |
| Caitlyn      | 0.527548 |     726 |
| Draven       | 0.500938 |     533 |
| Ezreal       | 0.462601 |    1738 |
| Jhin         | 0.470588 |    1156 |
| Jinx         | 0.502806 |    3920 |
|Kai'Sa       | 0.479    |    1000 |
| Kalista      | 0.524338 |    1171 |
| Kog'Maw      | 0.421965 |     173 |
| Lucian       | 0.533507 |    1149 |
| Miss Fortune | 0.519481 |     539 |
| Nilah        | 0.492754 |      69 |
| Samira       | 0.503571 |     280 |
| Senna        | 0.524331 |     822 |
| Seraphine    | 0.521341 |     328 |
| Sivir        | 0.522556 |    1064 |
| Tristana     | 0.494737 |     475 |
| Twitch       | 0.496599 |     441 |
| Varus        | 0.477431 |     576 |
| Vayne        | 0.453901 |     141 |
| Xayah        | 0.482523 |    1173 |
| Zeri         | 0.533549 |    2474 |
| Ziggs        | 0.420513 |     195 |"

<iframe src="assets/adc_wr.html" width=800 height=600 frameBorder=0></iframe>

It looks like *Lucian's* win rate is quite high. The graph shows the winrates for each ADC depending on which side their team played on. It looks like *Lucian* has average performance on red side, but has strong outperformance on blue side causing his average win rate to spike.

Let's define our hypotheses:
 - Null Hypothesis: Lucian's winrate is the same as other ADCs and his high winrate is purely due to random chance.
 - Alternative Hypothesis: Lucian's winrate is higher than other ADcs, it is not just due to random chance.

To test this out, we'll run a hypothesis test at the 95% confidence level.

We observe that Lucian has a 53.35% win rate accross 1149 games. We want to see if his win rate is statistically higher from the rest of the ADCs, so we will randomly select 1149 games and find the winrate of the sample which will be our test statistic.

<iframe src="assets/wr_hyp_test.html" width=800 height=600 frameBorder=0></iframe>

The chance that we would see a winrate that is just as great or greater than Lucian's winrate difference is 1.33% across 10,000 simulations which is rather low.

We will reject the null at the 95% confidence level since it is so rare to see a win rate as high as Lucian's which suggests that he may possibly be a better carry.

## Key Finding

The data suggests that Lucian may be a better carry, however his overall winrate is biased by his outperformance on the blue side. With this in mind, there may be extraneous factors that affect his performance relative to other ADCs.