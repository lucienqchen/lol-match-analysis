---
layout: default
title: "League of Legends Competitive Matches Exploratory Data Analysis"
permalink: /lol-match-analysis
---

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

<iframe src="assets/position_damageshare.html" width=800 height=600 frameBorder=0></iframe>

This graph displays the percentiles of damageshares for each position.

<iframe src="assets/adc_pickrate.html" width=800 height=600 frameBorder=0></iframe>

<iframe src="assets/adc_wr.html" width=800 height=600 frameBorder=0></iframe>

## Dealing with Missingness

It looks like there are a lot of missing values for golddiffat10 and xpdiffat10. Since either of these columns represent how well Lucian does relative his opponents we can just focus in on the missingness of just golddiffat10. Let's take a look at exactly how many missing values there are in the entire dataframe and try to figure out why that is.

<iframe src="assets/missingness_tvd.html" width=800 height=600 frameBorder=0></iframe>

<iframe src="assets/missingness_tvd_with_observed.html" width=800 height=600 frameBorder=0></iframe>

## Performing a Hypothesis Test

Let's finally answer our question:

__*Is Lucian a better carry than other ADCs?*__

We are going to measure ability to carry by revisiting winrate. If a champion wins more often than other champions, on average, there is a high likelihood that the champion is more effective in their role, i.e better at carrying.

Let's define our hypotheses:
 - Null Hypothesis: Lucian's winrate is the same as other ADCs and his high winrate is purely due to random chance.
 - Alternative Hypothesis: Lucian's winrate is higher than other ADcs, it is not just due to random chance.

To test this out, we'll run a hypothesis test at the 95% confidence level.

We can see that Lucian has a 53.35% win rate accross 1149 games. We want to see if his win rate is statistically higher from the rest of the ADCs, so we will randomly select 1149 games and find the winrate of the sample which will be our test statistic.

<iframe src="assets/wr_hyp_test.html" width=800 height=600 frameBorder=0></iframe>

The chance that we would see a winrate that is just as great or greater than Lucian's winrate difference is 1.33% across 10,000 simulations which is rather low.

We will reject the null at the 95% confidence level since it is so rare to see a win rate as high as Lucian's which suggests that he may possibly be a better carry.