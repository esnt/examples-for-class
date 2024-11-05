---
layout: default
---

# Example 2
*Note: All identifying links have been removed for anonymity*

---

## The Question
The ultimate goal of this project is to predict the outcome of a college football game while it is still in progress.  More details can be found on my previous blog post, but in summary the data I am using is a combination of data from <a href="https://www.cbssports.com/" target="_blank">CBS Sport's Site</a> and methods from <a href="http://www.glicko.net/research/nfl-chapter.pdf" target="_blank">this paper</a> by Mark Glickman and Hal Stern.

## EDA
Before doing any sort of statistical modeling, it is always important to do some exploratory data analysis (EDA).  EDA helps a statistician to understand the data so they can work with it correctly.  The code for all the plots I will show can be found in my github repo, and all the data can be found here.

First, I wanted to understand the distribution of the response variable, the final differential of the games.  As a reminder, differentials are calculated as the home team's score minus the away team's score, so if it is positive then the home team is winning and vice versa.

<img src="{{site.url}}/{{site.baseurl}}/assets/images/s2/final_diff_distribution.jpeg" alt="" style="width:800px;"/>

As you can see, there are definite spikes in the differentials, the largest of which is at 7, which also happens to be both the mean and median value of the data.  This could indicate that there is a home team advantage in college football, since more of the observations show the home team winning than the away team.


This next plot shows the relationship between final differential and the main predictor variable, halftime differential.

<img src="{{site.url}}/{{site.baseurl}}/assets/images/s2/final_by_half.jpeg" alt="" style="width:800px;"/>

There definitely seems to be a strong linear relationship between these two variables, which bodes well for future modeling.


Next, I wanted to check how accurate the methods I used from Glickman and Stern's paper are.  Since the whole point of using their work was to estimate team strength before the game, I looked at how it ranks teams based on all the data from 2017 through 2022:

<img src="{{site.url}}/{{site.baseurl}}/assets/images/s2/top_25_all.jpeg" alt="" style="width:800px;"/>

I also graphed out how it ranked teams based on only the data from the regular 2022 season, which turned out surprisingly close to <a href="https://www.ncaa.com/rankings/football/fbs/associated-press" target="_blank">AP's top 25 rankings</a> for the year.

<img src="{{site.url}}/{{site.baseurl}}/assets/images/s2/top_25_2022.jpeg" alt="" style="width:800px;"/>


Now that I was confident in my main variables, I used a couple of methods to look at the correlation between all my variables.  This is important before doing any sort of modeling in order to avoid problems with multicollinearity.

<img src="{{site.url}}/{{site.baseurl}}/assets/images/s2/heatmap.jpeg" alt="" style="width:800px;"/>

<img src="{{site.url}}/{{site.baseurl}}/assets/images/s2/pairplot.jpeg" alt="" style="width:800px;"/>

There is a lot of information here and several things I could talk about, but I will focus on the most important things I noticed.  
- One very interesting thing is that the predicted differentials based on all data and only recent data both have the same correlation with final differential, but are not perfectly correlated with each other.  I am curious what this effect will have when I begin to do modeling with the data.
- homewins-awaywins is not extremely correlated with my other variables estimating team strength.  This gives me even more confidence that the methods I use to estimate team strength isn't simply based off of the number of wins a team has.
- As one would expect, the correlation between the final differential and the differential at the first quarter, halftime, and third quarter increases.  This probably means that when I do my modeling, I will be much less certain about my predictions after the first quarter than at halftime, and the same with halftime and third quarter.
- Several of the variables look like they are linearly associated with each other, which could lead to some multicollinearity problems in the future.
- Most of my variables look like they are clearly linearly associated with the final differential.  This will be very useful information to help me choose what sort of model to use.

## Conclusion
In summary, I gained a lot of useful information from this EDA process.  I learned more about my variables, gained more trust in them, and have some ideas of how best to model them.  That said, this blog post has only scratched the surface of the EDA I have done for this project.  I have just chosen the most relevant and, in my opinion, interesting plots.  I am very excited to do more work with this project and begin doing some modeling.