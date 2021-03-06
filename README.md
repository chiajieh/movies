# movies
Modeling and Prediction for Movies
---
View project on https://chiajieh.github.io/movies/
---
Chinwe Ajieh | chinweajieh@gmail.com
---

## Setup

### Load Packages

```{r load-packages, message = FALSE}
library(ggplot2)
library(dplyr)
library(statsr)
library(GGally)
library(gridExtra)
library(boot)
library(broom)
```

### Load Data


```{r load-data}
load("movies.Rdata")
```


* * * 
## Project Sections{.tabset}

<a id="sections"></a>
This project consists of six parts, and some of these parts have sub-sections. Kindly click on the `Section Breakdown` tab for the details of each section and click the numbers to navigate between sections.

### <span>&#8203;</span>

### Section Breakdown

[1.](#1) Data.

[2.](#2) Research Question.

[3.](#3) Exploratory Data Analysis and Inference.

  -[3.1](#3a) Characteristics of Top 200 Box Office Movies.

   ---[3.1.1](#3ai) Difference in the Average IMDb Rating of Movies in and not in the Top 200 Box Office List.

   ---[3.1.2.](#3aii) Confidence Interval for the Average IMDb Rating of Movies in the Top 200 Box Office List.

   ---[3.1.3.](#3aiii) Top 200 Box Office Movies and Best Picture Oscar Nomination/Award.

   ---[3.1.4.](#3aiv) Relationship between IMDb Rating and Some Variables, Focusing on Movies in the Top 200 Box Office List.

  -[3.2.](#3b) Relationship between IMDb Rating, Critics’ Scores and Selected Variables.

[4.](#4) Modeling.

  -[4.1.](#4a) Simple Linear Regression.

  -[4.2.](#4b) Multiple Linear Regression.

   ---[4.2.1.](#4bi) Model Selection.
   
   ----[4.2.1.1](#4bi1) Model Diagnostics.
   
   ----[4.2.1.2](#4bi2) Interpreting the Regression Parameters: Slope and Intercept.
   
   ----[4.2.1.3](#4bi3) Interpreting the p-value and Strength of the Fit of the Linear Model using Adjusted R2

[5.](#5) Prediction.

[6.](#6) Conclusion





* * *
><a id="1"></a>

```{r1, echo = FALSE}
```

## Part 1: Data

The data set consists of 651 movies randomly sampled from movies produced and released before 2016, with 32 variables. Information about the dataset was obtained from [Rotten Tomatoes](https://www.rottentomatoes.com/) and [IMDb](https://www.imdb.com/).

The observations can be classified as a product of retrospective observational study, hence no random assignment. We cannot make causal conclusions from the data (we can only associate). The data were randomly sampled as stated above, hence generalizable. 

Since there is no causal relationship but only association, the data is generalizable to all movies produced and released before 2016.


* * *
><a id="2"></a>

```{r2, echo = FALSE}
```

## Part 2: Research Question
Anectodally, the goal of a movie is to entertain an audience and to generate revenue while doing so. One way to track a movie's revenue is via [Box Office Mojo](https://www.boxofficemojo.com/) called box office revenue. A movie making it to "Top 200 Box Office List" is a good way to go. Some will argue that it is a significant measure of a movie's popularity and I do agree.

As a movie producer, as much as a critics score/rating is important, the audience score/rating should be of utmost consideration if the aim is to entertain, keeping in mind revenue. So, **we want to know if critics score is a significant predictor of audience rating/score while simultaneously controlling for other variables.**

**For our project, we will develop a parsimonious model to predict the audience rating from critics score and other variables in the data.**

In answering our research question, we will look at the following:

- The characteristics of movies that made it to the Top 200 Box Office List.

- Relationship between IMDb rating, critics' score, and selected variables

Interest: A movie making it to the Top 200 Box Office List is an evidence of the audience' choice, and a good way to determine the popularity of a movie is to predict the rating of the audience itself.

* * *
><a id="3"></a>

```{r3, echo = FALSE}
```

## Part 3: Exploratory Data Analysis and Inference

For ease of analysis, first we check the summary and structure of our data, using summary (movies) and str(movies) respectively. See [the codebook](https://d3c33hcgiwev3.cloudfront.net/_73393031e98b997cf2445132f89606a1_movies_codebook.html?Expires=1556496000&Signature=Q7qeG6LVNZs7243QHxstgP4oNqiGCtTTyb3a884k8yx0R5cBANo-tMqRimbGIP5VKUu11e3iKGVgfq0IyIK0n35dc0GVb4bOXJGvyxQTj6G7Ze13ceC0rYI~3ZMCZG2iDytGK8~z0bMMQhT9J8l5bdAgaYmk3Twb4x8FhUuZdNg_&Key-Pair-Id=APKAJLTNE6QMUY6HBC5A) for more details. The result of our summary will not be included in the project.

```{r}
str(movies)
```

```{r summary, results='hide'}
summary(movies)
```

><a id="3a"></a>

```{r3a, echo = FALSE}
```

### 3.1 Characteristics of Top 200 Box Office Movies

We will examine the relationship between the audience rating/score (`imdb_rating` and `audience_score`-Rotten Tomatoes) and movies in the Top 200 Box Office List to determine the best variable for our response variable.

First, let's get the summary of our Top 200 Box Office List (yes and no).

```{r}
summary(movies$top200_box)
```

From our sampled movies, only 15 movies are listed, and 636 movies are not in the Top 200 Box Office List.

We will look at the audience rating/score of the sampled movies both in and not in the Top 200 Box Office List, using both `imdb_rating` and `audience_score` variables. 


```{r}
# `imdb_rating` of `top200_box`
ggplot(data = movies, aes(x = top200_box, y = imdb_rating)) +
  geom_boxplot()
```

The `imdb_rating` for movies in the Top 200 Box Office List is high and slightly skewed to the left while movies not in Top 200 Box Office List are strongly left-skewed with notable outliers. A typical `imdb_rating` for movies in the Top 200 Box Office List is about 7.3 and less variable than movies not in the top 200. The bottom 25% `imdb_rating` of movies in the top 200 Box Office List is below 6.9 and the least score about 5.6.

We will calculate the summary statistics of the `imdb_rating` of movies in and not in the Top 200 Box Office List to get an accurate estimate. 

```{r}
# imdb_rating of Movies in the Top 200 Box Office List
movies %>% 
  filter(top200_box == "yes") %>%
  summarise(min = min(imdb_rating), max = max(imdb_rating), median = median(imdb_rating), 
            mean = mean(imdb_rating), sd = sd(imdb_rating), IQR = IQR(imdb_rating), 
            Q1 = quantile(imdb_rating, 0.25), Q3 = quantile(imdb_rating, 0.75))

# imdb_rating of Movies not in the Top 200 Box Office List
movies %>% 
  filter(top200_box == "no") %>%
  summarise(min = min(imdb_rating), max = max(imdb_rating), median = median(imdb_rating), 
            mean = mean(imdb_rating), sd = sd(imdb_rating), IQR = IQR(imdb_rating), 
            Q1 = quantile(imdb_rating, 0.25), Q3 = quantile(imdb_rating, 0.75))

```

Movies not in the Top 200 Box Office List have an `imdb_rating` as low as 1.9 to a high 9. 

Before we delve deeper into analyzing `imdb_rating` of movies in the Top 200 Box Office List, let us visualize the `audience_score` for movies in and not in the Top 200 Box Office List. This is to guide us in our decision for the better variable to use for our response variable.

```{r}
# audience_score of top200_box
ggplot(data = movies, aes(x = top200_box, y = audience_score)) +
  geom_boxplot()
```

The `audience_score` for movies in the Top 200 Box Office List are skewed to the right with visible outliers on the left while movies not in Top 200 Box Office List are slightly left-skewed. A typical `audience_score` for movies in the Top 200 Box Office List is about 81% and less variable than movies not in the top 200. The bottom 25% `audience_score` of movies in the top 200 Box Office List is below 70% and the least score of about 34%.

We can see that these distributions are more variable compared to that of `imdb_rating.` Also, for movies listed in the Top 200 Box Office List, the distribution of `audience_score` has notable outliers while that of `imdb_rating` is slightly left-skewed. Since we are interested in learning what attributes make a movie popular, hence focusing on movies in the Top 200 Box Office List, we will take the `imdb_rating` as a better estimate for our audience rating. <a href="#sections">[Back to sections]</a>

><a id="3ai"></a>

```{r3ai, echo = FALSE}
```

#### 3.1.1 Difference in the Average `imdb_ratings` of Movies in and not in the Top 200 Box Office List

*Exercise: Is the average `imdb_rating` of movies in the Top 200 Box Office List greater than the average `imdb_rating` of movies not in the top 200 Box Office List?*

We are interested in finding out if the average `imdb_rating` of movies in the top 200 Box Office List is generally greater than movies not in the List. Our point estimate is the observed difference between the sampled `imdb_ratings` of movies in and not in the Top 200 Box Office List. 

- The conditions of independence are met - movies were randomly sampled, and both categories represent less than 10% of movies in and not in the Top 200 Box Office List. The movies are independent of each other, and both categories are too (non-paired). 

- For the sample size/skew condition - the `imdb_rating` distribution of  movies not in the Top 200 Box Office List is strongly skewed to the left, and a sample size of 636 is sufficient to model its mean. While for movies in the Top 200 Box Office List, the distribution of `imdb_rating` is slightly left-skewed and with a sample size of 15, we can relax the normality condition, as slight skew is not problematic.

Since the conditions are met, we will go ahead with the hypothesis test for inference, for comparing two independent means.

$H_0$: $mu_{rating.movies.in}$ $-$ $mu_{rating.movies.not.in}$ $=$ $0$. There is no difference between the average `imdb_ratings` of movies in and not in the Top 200 Box Office List.

$H_A$: $mu_{rating.movies.in}$ $-$ $mu_{rating.movies.not.in}$ $>$ $0$. The average `imdb_rating` of movies in the Top 200 Box Office List is greater than the average `imdb_rating` of movies not in the Top 200 Box Office List.


```{r}
inference(y = imdb_rating, x = top200_box, data = movies, type = "ht", statistic = "mean", order = "yes, no",
          method = "theoretical", alternative = "greater")
```

The p-value is less than 0.05, so we reject the null hypothesis. **The data provide convincing evidence that the average `imdb_rating` of movies in the Top 200 Box Office List is greater than the average `imdb_rating` of movies not in the Top 200 Box Office List.** <a href="#sections">Back to sections</a>

><a id="3aii"></a>

```{r3aii, echo = FALSE}
```

#### 3.1.2 Confidence Interval for the Average `imdb_rating` of Movies in the Top 200 Box Office List

Since our data provide convincing evidence that the `imdb_rating` of movies in Top 200 Box Office List is greater, we will go ahead to establish a confidence interval for the average `imdb_rating` of movies in Top 200 Box Office List. 

The independence condition is met as discussed previously. Also, we can relax the normality condition for the sample size of 15. We can thus apply the t-distribution for the confidence interval for one sample mean, using the formula below.

\[ t-confidence\ interval\ for\ the\ mean = \bar{x} \pm t_{df}^*SE
\]

```{r}
# Confidence Interval for the Average imdb_rating of Movies in the Top 200 Box Office List
n <- 15
tstar = qt(0.975, df = 14)

movies %>%
  filter(top200_box == "yes") %>%
summarise(lower = mean(imdb_rating) - tstar * (sd(imdb_rating) / sqrt(n)),
            upper = mean(imdb_rating) + tstar * (sd(imdb_rating) / sqrt(n)))

```

**We are 95% confident that the average `imdb_rating` of movies in Top 200 Box Office List is between 6.76 to 7.52.**

If we are not satisfied with the sample size condition, we can decide to use the bootstrap method to estimate the sample mean (average `imdb_rating` of sampled movies in the Top 200 Box Office List) and estimate our confidence interval for the population.


```{r}
# Bootstrapping
moviesbt <- movies %>%
  filter(top200_box == "yes") %>%
  select(imdb_rating)

moviesboot <- moviesbt$imdb_rating

set.seed(102)

bootmean <- function(original_vector, resample_vector){
  mean(original_vector[resample_vector])}

mean_imdbrating <- boot(moviesboot, bootmean, R = 10000)

tidy(mean_imdbrating)
```

The bootstrap mean of 7.14 is an accurate estimate of our sample mean.

```{r}
boot.ci(mean_imdbrating)
```

The percentile bootstrap confidence interval can be adopted as the bias of the sample mean estimate is negligible, and the distribution is slightly skewed. However, the bootstrap confidence interval corresponds with our t-confidence interval so we can maintain our previous result.

*So can we conclude that `imdb_rating` is a significant predictor of a movie being in the Top 200 Box Office List? This question requires further analysis, not within the scope of our study at the moment.* We have seen from our analysis so far that movies in the Top 200 Box Office List have high `imdb_rating` and as such popular. <a href="#sections">Back to sections</a>

><a id="3aiii"></a>

```{r3aiii, echo = FALSE}
```

#### 3.1.3 Top 200 Box Office Movies and Best Picture Oscar Nomination/Award

Some argue that Oscar' nomination of a movie is a way to promote a movie and nominees are solely based on professional's choice. We will visualize the movies in and not in the Top 200 Box Office List that were nominated for best picture Oscar award and for those that won the award; then carry out a hypothesis test to evaluate if there is a difference in the proportion. 

```{r}
Plot1 <- movies %>%
  group_by(top200_box) %>% 
  count(best_pic_win) %>% 
  mutate(proportion = n/sum(n))%>%
ggplot(aes(x = top200_box, y = proportion, fill = best_pic_win)) +
  geom_bar(stat = "identity") +
  geom_text(aes(label = paste0(round(proportion * 100, 2), "%"), vjust = -0.2)) +
  labs(title = "Best Picture Oscar Award for Top 200 
       Box Office Movies (No/Yes)", x = "Top 200 Box Office List") +
  theme(axis.title.y = element_blank() , axis.text.y = element_blank(), axis.ticks.y = element_blank())
  

Plot2 <- movies %>%
  group_by(top200_box) %>% 
  count(best_pic_nom) %>% 
  mutate(proportion = n/sum(n))%>%
ggplot(aes(x = top200_box, y = proportion, fill = best_pic_nom)) +
  geom_bar(stat = "identity") +
  geom_text(aes(label = paste0(round(proportion  * 100, 2), "%"), vjust = -0.02)) +
  labs(title = "Nominated for Best Picture Oscar Award 
for Top 200 Box Office Movies (No/Yes)", x = "Top 200 Box Office List") +
  theme(axis.title.y = element_blank(), axis.text.y = element_blank(), axis.ticks.y = element_blank())

grid.arrange(Plot1, Plot2, ncol = 2)
```

0.94% of movies not in the Top 200 Box Office List won best picture Oscar award while 6.67% of movies in the Top 200 Box Office List won best picture Oscar award.

Also, 3.14% of movies not in the Top 200 Box Office List were nominated for the best picture Oscar award while 13.33% of movies in the Top 200 Box Office List were nominated for the best picture Oscar award.

From the plot, the proportion of sampled movies in the Top 200 Box Office List nominated for best picture Oscar award, also those who won the award, is greater than those not in the Top 200 Box Office List.  So the question that comes to mind is: is there actually a significant difference in the proportions of movies in and not in Top 200 Box Office List, nominated for best picture Oscar award? This question is applicable to the proportion of movies in and not in the Top 200 Box Office List that won the best picture Oscar award.

Before we go ahead with the hypothesis test, we will calculate the counts and check if the conditions for inference for conducting a hypothesis test, to compare two proportions, are met:

```{r}
# Count of Movies in and not in the Top 200 Box Office List that were Nominated for Best Picture Oscar Award
movies %>%
  group_by(top200_box) %>% 
  count(best_pic_nom)
```

*Hypothesis Test for Movies in and not in the Top 200 Box Office List that were Nominated for Best Picture Oscar Award:*

$H_0$: $p_{movies.in.nom}$ $=$ $p_{movies.not.in.nom}$. The population proportion of movies in the Top 200 Box Office List nominated for best picture Oscar award is same as the population proportion of movies not in  the Top 200 Box Office List nominated for best picture Oscar award.

$H_A$: $p_{movies.in.nom}$ $!=$ $p_{movies.not.in.nom}$. There is a difference between the population proportions of movies in the Top 200 Box Office List nominated for the best picture Oscar award and those not in the Top 200 Box Office List nominated for best picture Oscar award.

Conditions for inference for conducting a hypothesis test, to compare two proportions:

- Independence: *within groups-met:* random sample: both populations were sampled randomly; the 10% condition is met for both populations. So, sampled movies in the Top 200 Box Office List nominated for the best picture Oscar award are independent of each other and also those not in the Top 200 Box Office List nominated for best picture Oscar award.
*between groups-met:* We do not expect sampled movies in the Top 200 Box Office List nominated for the best picture Oscar award and those not in  the Top 200 Box Office List nominated for best picture Oscar award to be dependent.

- Sample size/skew: We need the pooled proportion to check the success-failure condition.

\[Success\ Condition = n \hat{p}_{pool} >= 10
\]
\[Failure\ Condition = n(1 -  \hat{p}_{pool}) >= 10
\]

```{r}
# Pooled proportion = total successes/total n
phat_pool = (20 + 2)/(616 + 20 + 13 + 2)
phat_pool

# Movies in the Top 200 Box Office List: success
15 * phat_pool

# Movies in the Top 200 Box Office List: failure
15 * (1 - phat_pool)

# Movies not in the Top 200 Box Office List: success
636 * phat_pool

# Movies not in the Top 200 Box Office List: failure
636 * (1 - phat_pool)
```

The success condition for the movies in the Top 200 Box Office List is not met, so we conduct our inference via simulation.

```{r}
inference(y = best_pic_nom, x = top200_box, data = movies, type = "ht", statistic = "proportion", success = "yes",
          method = "simulation", alternative = "twosided", nsim = 15000)
```

At 5% significant level, the p-value is greater; hence we fail to reject the null hypothesis. **The data do not provide strong evidence that the population proportion of movies in the Top 200 Box Office List nominated for the best picture Oscar award is different from those not in the Top 200 Box Office List nominated for best picture Oscar award.**

So we will go ahead with the inference for those that won the best picture Oscar award. 

```{r}
# Count of Movies in and not in the Top 200 Box Office List that won Best Picture Oscar Award
movies %>%
  group_by(top200_box) %>% 
  count(best_pic_win)
```


*Hypothesis Test for Movies in and not in the Top 200 Box Office List that won Best Picture Oscar Award:*

$H_0$: $p_{movies.in.win}$ $=$ $p_{movies.not.in.win}$. The population proportion of movies in the Top 200 Box Office List that won best picture Oscar award is same as the population proportion of movies not in the Top 200 Box Office List that won best picture Oscar award.

$H_A$: $p_{movies.in.win}$ $!=$ $p_{movies.not.in.win}$. There is a difference between the population proportions of movies in the Top 200 Box Office List that won best picture Oscar award and those not in the Top 200 Box Office List that won best picture Oscar award.

Conditions for inference for conducting a hypothesis test, to compare two proportions:

- Independence: *within groups-met:* random sample: both populations were sampled randomly; the 10% condition is met for both populations. So, sampled movies in the Top 200 Box Office List that won best picture Oscar award are independent of each other and also those not in  the Top 200 Box Office List that won best picture Oscar award.
*between groups-met:* We do not expect sampled movies in the Top 200 Box Office List that won the best picture Oscar award and those not in the Top 200 Box Office List that won best picture Oscar award to be dependent.

- Sample size/skew: We need the pooled proportion to check the success-failure condition for a hypothesis test.

\[Success\ Condition = n \hat{p}_{pool} >= 10
\]
\[Failure\ Condition = n(1 -  \hat{p}_{pool}) >= 10
\]

```{r}
# Pooled proportion = total successes/total n
phat_pool_win = (6 + 1)/(630 + 6 + 14 + 1)
phat_pool_win

# Movies in the Top 200 Box Office List: success
15 * phat_pool_win

# Movies in the Top 200 Box Office List: failure
15 * (1 - phat_pool_win)

# Movies not in the Top 200 Box Office List: success
636 * phat_pool_win

# Movies not in the Top 200 Box Office List: failure
636 * (1 - phat_pool_win)
```

The success conditions for the movies in and not in the Top 200 Box Office List are not met, so we conduct our inference via simulation.

```{r}
inference(y = best_pic_win, x = top200_box, data = movies, type = "ht", statistic = "proportion", success = "yes",
          method = "simulation", alternative = "twosided", nsim = 15000)
```

The p-value is greater than 0.05, so we fail to reject the null hypothesis. **The data do not provide strong evidence that the population proportion of movies in the Top 200 Box Office List that won best picture Oscar award is different from those not in the Top 200 Box Office List that won best picture Oscar award.** <a href="#sections">Back to sections</a>

Oscar' best picture nomination and win seem independent of whether a movie is in the Top 200 Box Office List or not.

><a id="3aiv"></a>

```{r3aiv, echo = FALSE}
```

#### 3.1.4 Relationship between IMDb Rating and Some Variables, Focusing on Movies in the Top 200 Box Office List

We will evaluate the relationship betwen `imdb_rating`, `critics_score` and top 200 Box Office movies (in and not in the list). 
```{r}
Plot3 <- ggplot(data = movies) +
  geom_jitter(aes( x = critics_score, y = imdb_rating, color = top200_box)) + 
  ggtitle("Imdb_rating ~ Critics Score ~ Top 200 
           Box Office Movies (No/Yes)") +
  theme(legend.justification = c(1,0), legend.position = c(1,0))

Plot4 <- ggplot(data = movies, aes(x = top200_box, y = critics_score)) +
  geom_boxplot() + 
  ggtitle("Critics Score ~ Top 200 Box Office 
               Movies (No/Yes)")

grid.arrange(Plot3, Plot4, ncol = 2)

# Correlation Coefficient of the Relationship between imdb_rating and critics_score
summarise(.data = movies, cor(x = critics_score, y = imdb_rating))
```
<a id="corr"></a>
The plot shows a strong, positive linear relationship between `imdb_rating` and `critics_score` with a correlation coefficient, R of [0.76](#corrlink). In the next section, we will find out if critics score (`critics_score`) is a significant predictor of audience rating (`imdb_rating`) while simultaneously controlling for other variables.

The distribution of `critics_score` of movies in the Top 200 Box Office List is moderately skewed to the left with values between about 53% and 95% and two notable outliers of about 31% and 38%, but less variable than the `critics_score` of movies not in the Top 200 Box Office List. The `critics_score` of movies not in the Top 200 Box Office List is slightly left skewed with values as low as 1% and as high as 100%. We already know, from our previous analysis, that the `imdb_rating` of movies in the Top 200 Box Office List is greater than that of movies not in the Top 200 Box Office List (also as displayed in the Imdb_rating ~ Critics Score ~ Top 200 Movies' plot)

The summary statistics for critics score are as follows:

```{r}
# critics_score of Movies in the Top 200 Box Office List
movies %>% 
  filter(top200_box == "yes") %>%
  summarise(min = min(critics_score), max = max(critics_score), median = median(critics_score), 
            mean = mean(critics_score), sd = sd(critics_score), IQR = IQR(critics_score), 
            Q1 = quantile(critics_score, 0.25), Q3 = quantile(critics_score, 0.75))

# critics_score of Movies not in the Top 200 Box Office List
movies %>% 
  filter(top200_box == "no") %>%
  summarise(min = min(critics_score), max = max(critics_score), median = median(critics_score), 
            mean = mean(critics_score), sd = sd(critics_score), IQR = IQR(critics_score), 
            Q1 = quantile(critics_score, 0.25), Q3 = quantile(critics_score, 0.75))
```


A typical critics score of movies in the Top 200 Box Office List is 83% with variability of 18% while a typical critics score of movies not in the Top 200 Box Office List is 60.5% with a variability of 49%.

Next, we visualize the relationship between `imdb_rating`, `runtime` and `top200_box` and also that of `imdb_rating`, `genre` and `top200_box`
<a id="runtime"></a>
```{r plots5-6, fig.width = 12, fig.height= 5}
Plot5 <- ggplot(data = movies) +
  geom_jitter(aes( x = runtime, y = imdb_rating, color = top200_box)) + 
  ggtitle("IIMDb rating ~ Runtime ~ Top 200 Box Office Movies (No/Yes)")

Plot6 <- ggplot(data = movies) +
  geom_histogram(aes( x = imdb_rating, fill = genre)) +
  facet_grid(top200_box ~. ) + 
  ggtitle("IMDb rating ~ Genre ~ Top 200 Box Office Movies (No/Yes)")

grid.arrange(Plot5, Plot6, ncol = 2)

# Correlation Coefficient of the Relationship between imdb_rating and runtime
movies %>% 
  filter(!is.na(runtime)) %>%
  summarise(cor(x = runtime, y = imdb_rating))
```

The IMDb rating ~ runtime ~ top 200 Box Office movies (no/yes) plot shows a weak, positive, linear relationship between `imdb_rating` and `runtime` with a correlation coefficient, $R$ of 0.27 with about one notable leverage point that could be influential. [Back to diagnostics plot](#linearr)

The runtime for movies in the Top 200 Box Office List is between about 87.5mins to 162.5mins with an outlier of about 194 mins while the runtime of movies not in the Top 200 Box Office List seems to be concentrated around about 105mins with outliers, to the left as low as about 37mins and to the right, as high as about 265mins.

The `imdb_rating` varies within each genre of movies, and each distribution is typically left-skewed. The movies in  and not in the Top 200 Box Office List are of different genres. A great percentage of our sampled movies is drama; this could be by chance or most movies produced are predominantly drama. <a href="#sections">Back to sections</a>

><a id="3b"></a>

```{r3b, echo = FALSE}
```

### 3.2 Relationship between IMDb Rating, Critics' Scores and Selected Variables

The relationship between `imdb_rating` and `critics_score` are positive and strongly linear as discussed earlier. We will visualize their relationship with selected variables from the dataframe `movies` and with a new variable `best_dir_act_s_win`. 

*Creating the Variable - `best_dir_act_s_win`:*

We want to know if there is a difference in the `critics_score` and `imdb_rating` for movies that featured at least two of either one of the main actor, main actress or a director who won an Oscar award previously. So, we create a new variable `best_dir_act_s_win` and add to a new dataframe `movies1`, from the `movies` dataframe.

```{r}
movies1 <- movies %>%
  mutate(best_dir_act_s_win = ifelse(best_dir_win == "yes" & best_actor_win == "yes", "yes",
                              ifelse(best_dir_win == "yes" & best_actress_win == "yes", "yes", 
                              ifelse(best_actor_win == "yes" & best_actress_win == "yes", "yes", 
                              ifelse(best_dir_win == "yes" & best_actor_win == "yes" & 
                                       best_actress_win == "yes", "yes", "no")))))
```

We will go ahead to visualize the relationships.

```{r plots 7-14, fig.width= 12, fig.height= 10}
Plot7 <- ggplot(data = movies) +
  geom_jitter(aes( x = critics_score, y = imdb_rating, color = best_pic_win)) 

Plot8 <- ggplot(data = movies) +
  geom_jitter(aes( x = critics_score, y = imdb_rating, color = best_pic_nom)) +
  scale_color_manual(values = c("orange", "blue"))

Plot9 <- ggplot(data = movies) +
  geom_jitter(aes( x = critics_score, y = imdb_rating, color = best_actor_win)) +
  scale_color_manual(values = c("violet", "blue"))

Plot10 <- ggplot(data = movies) +
  geom_jitter(aes( x = critics_score, y = imdb_rating, color = best_actress_win)) +
  scale_color_manual(values = c("salmon", "purple"))

Plot11 <- ggplot(data = movies) +
  geom_jitter(aes( x = critics_score, y = imdb_rating, color = best_dir_win)) +
  scale_color_manual(values = c("gold", "darkgreen")) 

Plot12 <- ggplot(data = movies1) +
  geom_jitter(aes( x = critics_score, y = imdb_rating, color = best_dir_act_s_win)) +
  scale_color_manual(values = c("orange", "blue"))

Plot13 <- ggplot(data = movies) +
  geom_jitter(aes( x = critics_score, y = imdb_rating, color = genre))

Plot14 <- ggplot(data = movies) +
  geom_histogram(aes( x = critics_score, fill = genre))

grid.arrange(Plot7, Plot8, Plot9, Plot10, Plot11, Plot12, Plot13, Plot14, ncol = 2, 
             top = "Relationship between IMDb Rating, Critics Score,and Selected Variables")
```

The sampled movies nominated for a best picture Oscar award and those that won, typically have high `imdb_rating` and `critics_score.`

The `imdb_rating` of movies that won best picture Oscar award varies from about 7.3 to 8.5 (skewed to the right) and typically have a `critics_score` between about 81.25% to 97.5%. While for movies that did not win the best picture Oscar award has `imdb_rating` of about 1.9 to 8.5 and `crtics_score` of about 2% to 100%. Since the number of  movies that did not win a best picture Oscar award outnumbers those that won, their `imdb_rating` and `critics_score` are more variable. For movies nominated for the best picture Oscar award, there is a notable outlier in the distribution of `critics_score` (a score as low as 31% compared to the typical `critics_score` range of 75% to 97%).

There seems to be little or no difference in the `imdb_rating` and `critics_score` of movies that one of the main actors had ever won an Oscar and those that had actors that had never won an Oscar. Same can be said for movies that have a main actress that has ever won an Oscar or not. However, for movies that featured a main actress that has ever won an Oscar, the minimum `imdb_rating` is about 4.25, higher compared to those that did not, with a minimum `imdb_rating` of about 1.9. For movies directed by a director that has ever won an Oscar, the minimum `imdb_rating` is about 5.05, and there is little difference in their `critics_score` compared to those not directed by an Oscar winner.

Movies that either has one of the main actor, main actress or a director that has ever won an Oscar award have a minimum `critics_score` lower than 12.5% each. From our plot of the new variable `best_dir_act_s_win`, we can see that movies that have at least two of either one of the main actor, main actress or a director who has ever won an Oscar award have a minimum `critics_score` of about 39.5%. Also, the minimum `imdb_rating` for this category is about 5.8, highest compared to the `imdb_rating` of movies that either has one of the main actor, main actress or a director that has ever won an Oscar award.
 
*It seems movies directed by a director that has ever won an Oscar award tend to have above average audience rating, but a critic's score seems to be independent of whether one of the main actor, main actress or a director has ever won an Oscar award. However, critic's score seems to be influenced by a movie that has at least two of either one of the main actor, main actress or a director who has ever won an Oscar award. This might have been due to chance, that is the movies are extremely good movies; hence the critics' scores. We need to carry out further analysis to determine if there is an actual difference in `critics_score` for the different scenarios.*

The `imdb_rating` and `critics_score` varies within each genre of movies. <a href="#sections">Back to sections</a>


* * *
><a id="4"></a>

```{r4, echo = FALSE}
```

## Part 4: Modeling

><a id="4a"></a>

```{r4a, echo = FALSE}
```

### 4.1 Simple Linear Regression

We will develop a simple linear model, to analyze if `critics_score` is a statistically significant predictor of `imdb_rating` before we proceed with developing a parsimonious model, to predict the audience rating from `critics_score` and other variables in the data.

><a id="linear"></a>

```{r}
ggplot(data = movies, aes(x = critics_score, y = imdb_rating)) +
  geom_jitter() +         # we use geom_jitter in case of overplotting
  geom_smooth(method = "lm", se = FALSE)
```

><a id="corrlink"></a>

The relationship between `imdb_rating` and `critics_score` is positive and strongly linear as stated [earlier](#corr). 

Let us go ahead with the linear model, write out the equation and then define the slope and intercept of the relationship.

```{r}
m_imdbcritics <- lm(imdb_rating ~ critics_score, data = movies)
summary(m_imdbcritics)
```

\[\widehat{imdb\_rating} = 4.8075715 + 0.0292177\ critics\_score
\]

**For each additional percentage increase in critics score on Rotten Tomatoes, we would expect the imdb_rating of movies to increase on average by 0.0292177.**

**Movies with a `critics_score` of 0 are expected on average to have an `imdb_rating` of 4.8075715.**

We will examine if the conditions of the least squares regression are reasonable before we interpret the p-value and the strength of the fit of the linear model, R-squared $(R^2)$.

- The linearity condition is met as the relationship between `imdb_rating` and `critics_score` are [linear](#linear).

- The independence condition is met as the data were randomly sampled and the sampled movies in this distribution are less than 10% of movies produced and released before 2016. 

```{r plots 15-18, fig.width = 8}
# Nearly Normal Residuals
Plot15 <- ggplot(data = m_imdbcritics, aes(x = .resid)) +
  geom_histogram() +
  ggtitle("Histogram of Residuals")

Plot16 <- ggplot(data = m_imdbcritics, aes(sample = .resid)) +
  stat_qq()  + 
  stat_qq_line() + 
  ggtitle("Normal Probability Plot of Residuals")

# Constant Variability of Residuals
Plot17 <- ggplot(data = m_imdbcritics, aes(x = .fitted, y = .resid)) +
  geom_jitter() +
  geom_hline(yintercept = 0, linetype = "dashed") +
  labs(x = "Fitted Values", y = "Residuals") +
  ggtitle("Residuals vs. Fitted Values")

# Plotting the Absolute Values of the Residuals
Plot18 <- ggplot(data = m_imdbcritics, aes(x = .fitted, y = abs(.resid))) +
  geom_jitter() +
  ggtitle("Absolute Value of Residuals vs. Fitted Values")

grid.arrange(Plot15, Plot16, Plot17, Plot18, ncol = 2)
```

The histogram and normal probability plot of residuals show that the residuals are nearly normally distributed and centered around 0. 

The residual plot and the plot of the absolute value of the residuals against the fitted values show that the variability of residuals around the 0 line is approximately constant with a few outliers

Since the conditions of least squares regression are reasonable, at 5% significant level, `critics_score` is a statistically significant predictor of `imdb_rating`. Also, 58.46% of the variability in `imdb_rating` is explained by the model (explained by the `critics_score`).<a href="#sections">Back to sections</a>

><a id="4b"></a>
```{r4b, echo = FALSE}
```

### 4.2 Multiple Linear Regression

We know a linear relationship exists between `imdb_rating` and `critics_score`; now, we will develop a parsimonious model to predict the IMDb rating from `critics_score` and other variables in the data. 

Before we proceed with the stepwise model selection, we need to decide the variables we will include in our model.

The following variables will not be included in our model:

- `actor1` to `actor5`: As stated in the project instruction page, the information in the variables was used to determine if the movie features an actor or an actress who have ever won an Oscar award; hence they will not be included in our model.

- `title,` `studio` and `director`: Each movie has its own unique title; hence there are more than 600 titles in our `movies` data. For the `director` variable, some movies in our observation might have been directed by the same director. However there are numerous movie directors, and the best way to single them out is to use the `best_dir_win` variable (whether or not the director of the movie ever won an Oscar). Both variables are nominal/regular categorical variables with more than 600 different characters hence difficult to model.

  The `studio` variable, however an ordinal categorical variable, has 211 levels thus will be difficult to model.

- `thtr_rel_year,' `thtr_rel_day, `dvd_rel_year` and `dvd_rel_day`: We will exclude these variables but include the month the movie is released in theaters and on DVD. We believe the month of release should be of utmost consideration compared to the day and year as we have various seasons and the festive periods which could influence the audience choice.

- `imdb_num_votes`: This variable will not be included in our model as before a movie is released to the public and even immediately after its release; we would not have an idea of the intended number of votes.

- `critics_rating`: The critics rating on Rotten Tomatoes will not be included in our model, to prevent multicollinearity, as this variable is determined with the `critics_score` variable.

- `audience_score` and `audience_rating`: We can say the `audience_score` variable on Rotten Tomatoes is similar to the `imdb_rating` on IMDb as they are both numerical variables that measure an audience opinion, hence it is advisable to exclude the `audience_score` variable from our model. Also, since the `audience_rating` is determined from the `audience_score` variable, it should not be included in the model. We can examine the relationship between these variables and the `critics_score` variable to make a better decision.

```{r}
ggpairs(movies, columns = 16:18 )
```


The `critics_score` and `audience_score` variables are collinear (correlated). Also, there appears to be a constant variance in the `audience_rating` variable with respect to the `critics_score` variable. In addition to the reason mentioned earlier, to prevent model estimate complication, it is best not to include the `audience_score` and `audience_rating` variables in the model. <a href="#sections">Back to sections</a>

><a id="4bi"></a>

```{r4bi, echo = FALSE}
```

#### 4.2.1 Model Selection {.tabset .tabset-fade .tabset-pills}

For our stepwise model selection, we will use the backward elimination method using the p-value approach. We decided to use the backward elimination method as we already have a full model in mind, which an elimination method will be applied, to drop one predictor at a time to achieve a parsimonious model. We chose the p-value approach as our focus is to obtain statistically significant predictors of our response variable, `imdb_rating.`

We will proceed with our full model, drop the variable with the highest p-value, refit the model and repeat the process until we have a model that has all significant variables. We choose a significant level of 5%. *Click on each tab to navigate the steps.*

##### Full Model
```{r}
# Full Model
m_imdbrating <- lm(imdb_rating ~ critics_score + title_type + genre + runtime + mpaa_rating + thtr_rel_month +
                     dvd_rel_month + best_actor_win + best_actress_win + best_dir_win + top200_box, data = movies)

summary(m_imdbrating)
```

The variable, `best_actor_win` has the highest p-value hence we drop it and proceed with the model.


##### Step 1

```{r}
# Step 1
m_imdbrating1 <- lm(imdb_rating ~ critics_score + title_type + genre + runtime + mpaa_rating + thtr_rel_month + 
                     dvd_rel_month + best_actress_win + best_dir_win + top200_box, data = movies)

summary(m_imdbrating1)
```

The variable, `title_type: Feature Film` has the highest p-value but we cannot drop it, as one of the levels of movie types is significant. Instead, we drop the variable, `best_actress_win` and proceed.

*Note that we cannot drop the individual level of a variable, especially when one of its level is significant, we keep the whole variable in. It simply means a variable with different levels can only be dropped as a whole when all of its levels are insignificant.*

##### Step 2
```{r}
# Step 2
m_imdbrating2 <- lm(imdb_rating ~ critics_score + title_type + genre + runtime + mpaa_rating + thtr_rel_month + 
                     dvd_rel_month + best_dir_win + top200_box, data = movies)

summary(m_imdbrating2)
```

We drop the variable `top200_box` next.

##### Step 3
```{r}
# Step 3
m_imdbrating3 <- lm(imdb_rating ~ critics_score + title_type + genre + runtime + mpaa_rating + thtr_rel_month + 
                     dvd_rel_month + best_dir_win, data = movies)

summary(m_imdbrating3)
```

We drop the `best_dir_win` variable and proceed with the model.

##### Step 4
```{r}
# Step 4
m_imdbrating4 <- lm(imdb_rating ~ critics_score + title_type + genre + runtime + mpaa_rating + thtr_rel_month + 
                     dvd_rel_month, data = movies)

summary(m_imdbrating4)
```

We drop the `thtr_rel_month` variable next.

##### Step 5
```{r}
# Step 5
m_imdbrating5 <- lm(imdb_rating ~ critics_score + title_type + genre + runtime + mpaa_rating + 
                     dvd_rel_month, data = movies)

summary(m_imdbrating5)
```

Next, we drop the `mpaa_rating` variable.

##### Step 6
```{r}
# Step 6
m_imdbrating6 <- lm(imdb_rating ~ critics_score + title_type + genre + runtime + dvd_rel_month, data = movies)

summary(m_imdbrating6)
```

We drop the `title_type` variable.

##### Step 7
```{r}
# Step 7
m_imdbrating7 <- lm(imdb_rating ~ critics_score + genre + runtime + dvd_rel_month, data = movies)

summary(m_imdbrating7)
```

We drop the `dvd_rel_month` variable.

##### Final Model

<a id="modelresult"></a>

```{r}
# Final Model
mfinal_imdbrating <- lm(imdb_rating ~ critics_score + genre + runtime, data = movies)

summary(mfinal_imdbrating)
```
</div>


We can summarize our final model for predicting `imdb_rating` with the equation below:

\[\begin{align}
\widehat{imdb\_rating} = 4.095 + 0.026\ critics\_score\ - 0.167\ genre: Animation\ + 0.394\ genre: Art\ House\ \&\ International\\ -\ 0.159\ genre: Comedy\ + 0.582\ genre: Documentary\ + 0.114\ genre: Drama\ - 0.183\ genre: Horror\\ +\ 0.347\ genre: Musical\ \&\ Performing Arts\ + 0.112\ genre: Mystery\ \&\ Suspense\ + 0.001\ genre: Other\\ -\ 0.413\ genre: Science\ Fiction\ \&\ Fantasy\ + 0.008\ runtime\end{align}
\]

><a id="4bi1"></a>

```{r4bi1, echo = FALSE}
```

##### 4.2.1.1 Model Diagnostics

Before we interpret our parameter estimate, p-value and the strength of the fit of the linear model, adjusted R-squared $(R^2)$, we will examine if the conditions of the least squares regression are reasonable, using diagnostic plots.

```{r plots19-25, fig.width= 10, fig.height= 12}
# Linear Relationship between (Numerical) x and y
Plot19 <- ggplot(data = mfinal_imdbrating, aes(x = critics_score, y = .resid)) +
  geom_jitter() +
  geom_hline(yintercept = 0, linetype = "dashed") +
  ggtitle("Residuals vs. Critics Score")

Plot20 <- ggplot(data = mfinal_imdbrating, aes(x = runtime, y = .resid)) +
  geom_jitter() +
  geom_hline(yintercept = 0, linetype = "dashed") +
  ggtitle("Residuals vs. Runtime")

# Nearly Normal Residuals
Plot21 <- ggplot(data = mfinal_imdbrating, aes(x = .resid)) +
  geom_histogram() +
  ggtitle("Histogram of Residuals")

Plot22 <- ggplot(data = mfinal_imdbrating, aes(sample = .resid)) +
  stat_qq()  + 
  ggtitle("Normal Probability Plot of Residuals")

# Constant Variability of Residuals
Plot23 <- ggplot(data = mfinal_imdbrating, aes(x = .fitted, y = .resid)) +
  geom_jitter() +
  geom_hline(yintercept = 0, linetype = "dashed") +
  labs(x = "Fitted Values", y = "Residuals") +
  ggtitle("Residuals vs. Fitted Values")

# Plotting the Absolute Values of the Residuals # to also confirm constant variability
Plot24 <- ggplot(data = mfinal_imdbrating, aes(x = .fitted, y = abs(.resid))) +
  geom_jitter() +
  ggtitle("Absolute Value of Residuals vs. Fitted Values")

# Variability across Categorical Variable- "genre"
Plot25 <- ggplot(data = mfinal_imdbrating, aes(x = genre, y = .resid)) +
  geom_boxplot() +
  ggtitle("Residuals vs. Genre") +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))


Plot26 <- ggplot(data = mfinal_imdbrating, aes(x = seq_along(.resid), y = .resid)) +
  geom_jitter() +
  labs(x = "Order of Collection", y = "Residuals", title = "Residuals in their Order of Data Collection")

grid.arrange(Plot19, Plot20, Plot21, Plot22, Plot23, Plot24, Plot25, Plot26, ncol = 2)
```


<a id="linearr"></a>

- Linear Relationship between x vs. y: The plot of residuals vs. `critics_score` shows random scatter of residuals around 0; so does the plot of residuals vs. `runtime.` However, for the plot of residuals vs. `runtime,` the variability of points around the least square line is not actually constant, indicating heteroscedasticity, though a linear relationship exists but weaker compared to the plot of residuals vs. `critics_score.` ([See our earlier discussion](#runtime) on the bivariate relationship between `imdb_rating` and `runtime`). 

  From the residuals vs. genre plot, there is a difference in the variability of the residuals across the groups.

- Nearly Normal Residuals: From the normal probability plot and histogram of residuals, there is random scatter of residuals around 0. However, there are a few outliers to the left which are no cause for concern as the number of observations is large enough. The condition of the nearly normal distribution of residuals around 0 is met.

- Constant Variability of Residuals: The residuals vs. fitted values plot shows random scatter of residuals with a nearly constant width around 0. However, there are few outliers for low and high fitted values. Also, the absolute value of the residuals against their fitted values plot shows the nearly constant variance of the residuals.

- Independent residuals: The plot of residuals in their order of data collection shows no connection (no structure) between observations, hence independent. Also, the data were randomly sampled, and less than 10% of movies produced and released before 2016.

The conditions for this regression model are reasonable, so we will go ahead to interpret the parameter estimates, p-value and strength of the fit of the linear model, adjusted $R^2$. <a href="#sections">Back to sections</a>

><a id="4bi2"></a>

```{r4bi2, echo = FALSE}
```
##### 4.2.1.2 Interpreting the Regression Parameters: Slope and Intercept

- **Slope of Critics' Score:** All else held constant, for each additional percentage increase in critics score on Rotten Tomatoes, we would expect the `imdb_rating` of movies to increase on average by 0.026. The addition of other variables has reduced the slope estimate of `critics_score` by 12.17% when compared to the slope from our previous model (simple linear regression model).

- **Confidence Interval for the Slope of Critics' Score`:**

\[\begin{align} =\ point\ estimate\ \pm\ margin\ of\ error
\\ =\ b_1 \pm\ t^*_{df}SE_{b1}\end{align}\]

```{r}
SE = 0.001034
b1 = 0.025661
tstar_df = qt(0.975, df = 637)
lower = b1 - (tstar_df * SE)
upper = b1 + (tstar_df * SE)

confint <- matrix(c(lower, upper), ncol = 2, nrow = 1)
colnames(confint) <- c("Lower", "Upper")
rownames(confint) <- c("Confidence Interval")
as.table(confint)
```

  **We are 95% confident that, all else being equal, the model predicts that for each additional percentage increase in critics score on Rotten Tomatoes, we would expect the `imdb_rating` of movies to increase on average between 0.024 and 0.028.**


- **Slope of Runtime:** All else held constant, for each 1-minute increase in the runtime of movie, the model predicts the `imdb_rating` of movies to increase on average by 0.008.

- **Intercept:** Action & Adventure movies with no `critics_score` and no `runtime` are expected to have an `imdb_rating` of 4.095. *The reference level for movies' genre is Action & Adventure.*

  **The intercept is meaningless in context, serves to adjust the height of the line as no runtime simply means no movie.**

From our model, for movie genres, Science Fiction & Fantasy movie is predicted to have the lowest `imdb_rating,` and Documentary movie is expected to have the highest `imdb_rating,` all else held constant.

><a id="4bi3"></a>

```{r4bi3, echo = FALSE}
```
##### 4.2.1.3 Interpreting the p-value and Strength of the Fit of the Linear Model, using Adjusted $R^2$

To determine the significance of the whole model and one of the predictors, `critics_score,` we will interpret the p-values and make an inference.  

**Inference for the Model as a Whole**

$H_0$: $\beta_i$ $=$ $\beta_1$ $=$......$=$ $\beta_k$ $=$ $0$

$H_A$: At least one $\beta_i$ is different than $0$.

The result from our final model is as follows: $*[F-statistic: 91.34\ on\ 12\ and\ 637\ DF,\  p-value: < 2.2e-16]*$

At 5% significant level, we reject the null hypothesis; hence **the model is significant.** This simply means that at least one of the slopes is not zero.

**Hypothesis Testing for the Slope of Critics' Score**

*Question:* Is `critics_score` a significant predictor of the IMDb rating of movies, given all other variables in the model?

$H_0$: $\beta_1$ $=$ $0$, when all other variables are included in the model.

$H_A$: $\beta_1$ $\not=$ $0$, when all other variables are included in the model.

From our [model result](#modelresult), the p-value of the `critics_score` $(< 2 \times 10^{-16})$ is less than $0.05$, so we reject the null hypothesis. **`critics_score` is thus a significant predictor of the `imdb_rating` of movies, given all other variables in the model.**

**Strength of the  Fit of the Linear Model, Adjusted $R^2$**

62.55% of the variability in `imdb_rating` is explained by the model. So, 37.45% of the variability in `imdb_rating` is not explained by this model.

The $adjusted\ R^2,\ 0.6255,$ of the final model increased by 0.63% from that of the full model $(adjusted\ R^2 = 0.6216)$. 

There is a 7% increase in the adjusted $R^2$ of this model compared to the adjusted $R^2$ of the simple linear model that considered only one predictor variable (`critics_score`).




* * *
<div id="predictions_conclusion">

><a id="5"></a>

```{r5, echo = FALSE}
```
## Part 5: Prediction

We will use the created model to predict the `imdb_rating` of some new movies from 2016 (not in our sample distribution) and quantify the prediction uncertainty, at a confidence level of 95%. The `critics_score`, `genre`, `runtime` and actual `imdb_rating` of each movie were obtained from [Rotten Tomatoes](https://www.rottentomatoes.com/) and [IMDb](https://www.imdb.com/).

```{r}
Zootopia <- data.frame(critics_score = 97, genre = "Animation", runtime = 108)
predict1 <- predict(mfinal_imdbrating, Zootopia, interval = "prediction")

Batman <- data.frame(critics_score = 27, genre = "Action & Adventure", runtime = 151)
predict2 <- predict(mfinal_imdbrating, Batman, interval = "prediction")

Deadpool <- data.frame(critics_score = 84, genre = "Action & Adventure", runtime = 108)
predict3 <- predict(mfinal_imdbrating, Deadpool, interval = "prediction")

Pets <- data.frame(critics_score = 73, genre = "Animation", runtime = 87)
predict4 <- predict(mfinal_imdbrating, Pets, interval = "prediction")

Captain <- data.frame(critics_score = 82, genre = "Drama", runtime = 118)
predict5 <- predict(mfinal_imdbrating, Captain, interval = "prediction")

Zoolander2 <- data.frame(critics_score = 23, genre = "Comedy", runtime = 101)
predict6 <- predict(mfinal_imdbrating, Zoolander2, interval = "prediction")

Me_before_you <- data.frame(critics_score = 56, genre = "Drama", runtime = 106)
predict7 <- predict(mfinal_imdbrating, Me_before_you, interval = "prediction")

imdb_rating <- c(8, 8, 6.5, 6.5, 7.9, 4.7, 7.4)

predict <- data.frame ("Movie" = c("Zootopia", "Batman V Superman: Dawn of Justice", "Deadpool (2016)", 
                                 "The Secret Life of Pets (2016)", "Captain Fantastic", "Zoolander 2 (2016)", 
                                 "Me Before You"),
"Prediction" = c(sprintf("%2.1f", predict1[1]), sprintf("%2.1f", predict2[1]), 
                 sprintf("%2.1f", predict3[1]), sprintf("%2.1f", predict4[1]), 
                 sprintf("%2.1f", predict5[1]), sprintf("%2.1f", predict6[1]), 
                 sprintf("%2.1f", predict7[1])),
"Conf_Int." = c(sprintf("%2.1f-%2.1f", predict1[2], predict1[3]), 
                sprintf("%2.1f-%2.1f", predict2[2], predict2[3]), 
                sprintf("%2.1f-%2.1f", predict3[2], predict3[3]), 
                sprintf("%2.1f-%2.1f", predict4[2], predict4[3]), 
                sprintf("%2.1f-%2.1f", predict5[2], predict5[3]), 
                sprintf("%2.1f-%2.1f", predict6[2], predict6[3]), 
                sprintf("%2.1f-%2.1f", predict7[2], predict7[3])),
"Actual_rating" = imdb_rating)

predict
```

The description of each column of the table above is as follows:

- `Movie`: New movies from 2016, not in the sample distribution.

- `Prediction`: The model prediction for each movie.

- `Conf_Int.`: 95% confidence interval of the expected/predicted `imdb_rating`

- `Actual_rating`: Actual IMDb rating of each movie.

*We will interpret only the confidence interval of the Zootopia movie for the purpose of this course.*

The model predicts, with 95% confidence, that the Zootopia movie (an animated movie) with a `critics_score` of 97% is expected to have an average `imdb_rating` between 5.9 to 8.6. 

*Note that the actual `imdb_rating` of the Zootopia movie was within the predicted 95% confidence interval.* <a href="#sections">Back to sections</a>


* * *

><a id="6"></a>

```{r6, echo = FALSE}
```

## Part 6: Conclusion

From our data, movies in the Top 200 Box Office List generally have high `imdb_rating` with few having below a 6.9 but does not necessarily mean nomination of the best picture Oscar award nor a win. Also, the `critics_score` is typically high with less than 30% having below a 70.5% but with an exception as low as 31% (just like the Batman V Superman: Dawn of Justice movie with a `critics_score` of 27%). `Critics_score` seems to be independent of if an actor, actress or director has ever won an Oscar, but movies that featured at least two of either an actor, actress or director who has previously won an award appears to have slightly higher `critics_score`.

`Critics_score` is a significant predictor of `imdb_rating` from our model and 62.55% of the variability in `imdb_rating` is explained by the model. This is evident in our predictions above - the actual `imdb_rating` of all except the Batman V Superman: Dawn of Justice movies in our prediction list was within the predicted 95% confidence interval. The prediction was accurate for the movie "The Secret Life of Pets (2016)".

The model, however significant has some shortcomings as listed below:

- For the model to be feasible, we need to develop a model to predict the critics' score on Rotten Tomatoes.

- We observed some fluctuation in the variability of residuals across the `genre` groups. Also, for `runtime,` there appears to be a non-constant variability of residual points scattered around 0. This suggests that a statistically significant `runtime` variable might not be significant, though it is important to note that the p-value is actually very small $(8.65 \times10^{-8})$.

- Though there was a 0.63% increase in the adjusted $R^2$ of the model, from the full model, indicating a higher predictive power, the p-value approach of model selection relies on an arbitrary significant level hence different significant level means different model.

- We are not definite if appropriate weighting measures were applied to the data to guide against bias due to a sampling of movies that have more than one part, to ensure the independence of each sampled movie.

- Most movies are a combination of more than one genre, for example, the Batman V Superman: Dawn of Justice is both an Action & Adventure and a Science, Fiction & Fantasy movie; which was not considered in this model.

The `critics_score,` `genre` and `runtime` of movies, however statistically significant, does not explain 37.45% of the variability in the `imdb_rating` for our model. Hence it is necessary to pay attention to the overall quality of the movie and other variables like the script, picture, sound, not present in this data. 

A low `critics_score` does not necessarily equate to a low `imdb_rating` and guarantee that a movie will not be listed in top 200 Box Office Movies.

For further analysis, we recommend the following:

- Weighted or/and a robust regression to guide against bias due to heteroscedasticity.

- For model selection, apply backward elimination method using an adjusted $R^2$ approach, for more reliable predictions.


* * *


## References

For this analysis, I used contents from the following websites and materials as a guide: 

**1.**  http://www.cookbook-r.com/Graphs/Facets_(ggplot2)/

**2.**  http://rpubs.com/chiajieh

**3.**  https://rstudio-pubs-static.s3.amazonaws.com/236787_a3b63b84e9b8423abf88411ee83106f3.html 

**4.**  https://stackoverflow.com

**5.**  http://www.sthda.com 

**6.**  Cosma Shalizi. [*"Using R Markdown for Class Reports".*] (http://www.stat.cmu.edu/~cshalizi/rmarkdown/). (2016). 

**7.**  David M Diez, Christopher D Barr and Mine Cetinkaya-Rundel. [*"OpenIntro Statistics, Third Edition"*](https://www.openintro.org/). (2016). 

**8.**  Eugenia Inzaugarat. *"Linear and Bayesian Modeling in R: Predicting Movie Popularity"* on [Medium](https://towardsdatascience.com/linear-and-bayesian-modelling-in-r-predicting-movie-popularity-6c8ef0a44184). (2018).

**9.**  Jim Frost. [*"Heteroscedasticity in Regression Analysis"*](https://statisticsbyjim.com/regression/heteroscedasticity-regression/)

**10.** Karl Broman. [*"Knitr with R Markdown"*](https://kbroman.org/knitr_knutshell/pages/Rmarkdown.html)

**11.** Luke Johnston. *"Resampling Techniques In R: Bootstrapping And Permutation Testing"* on [University of Toronto Coders](https://uoftcoders.github.io/studyGroup/lessons/r/resampling/lesson/).

**12.** *"Statistics with R Specialization"*, by Duke University on Coursera - (ongoing)

**13.** Yan Holtz. [*"Pimp my RMD: a few tips for R Markdown"*](https://holtzy.github.io/Pimp-my-rmd/). (2018).

**14.** Yi Yang. [*"An Example R Markdown"*](http://www.math.mcgill.ca/yyang/regression/RMarkdown/example.html). (2017).

**15.** Yihui Xie, J. J. Allaire and Garrett Grolemund. [*"R Markdown: The Definitive Guide"*](https://bookdown.org/yihui/rmarkdown/). (2019).

</div>
