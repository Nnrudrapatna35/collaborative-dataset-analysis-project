Data Analysis
================
Power Ninja Data Turtles
12.03.2019

### Load data and packages

``` r
library(tidyverse)
library(infer)
library(broom)

fight_songs <- read_csv("/cloud/project/data/fight-songs.csv")
fight_songs <- fight_songs %>%
  select(-X21)
```

### Research Question 1

Our first research question is:

> "How does the tempo (`bpm`) and duration (`sec_duration`) of a college
> football team’s fight song predict the content of the song,
> specifically the number of clichés/tropes (`trope_count`)?

Before we delve deeper into our analysis, let’s first take a look at the
distributions of our two explanatory variables, `bpm` and
`sec_duration`.

Starting with `bpm`, which is a measure of a fight song’s tempo, we will
create a histogram and find the relevant summary statistics:

``` r
ggplot(fight_songs, mapping = aes(x = bpm)) +
  geom_histogram(binwidth = 10) +
  labs(title = "Tempo of College Fight Songs", x = "Beats per Minute (bpm)", y = "Number of Songs")
```

![](data-analysis_files/figure-gfm/histogram-bpm-1.png)<!-- -->

Let’s also calculate the summary statistics for this distribution.
Specifically, we will use the median as a measure of center and the
interquartile range as a measure of spread (due to the bimodal nature of
the distribution). In addition, we will find the upper and lower
quartiles (Q3 and Q1, respectively), and the maximum and minimum values:

``` r
fight_songs %>%
  summarise(min = min(bpm), Q1 = quantile(bpm, .25), median = median(bpm), Q3 = quantile(bpm, .75), max = max(bpm), IQR = IQR(bpm))
```

    ## # A tibble: 1 x 6
    ##     min    Q1 median    Q3   max   IQR
    ##   <dbl> <dbl>  <dbl> <dbl> <dbl> <dbl>
    ## 1    65    90    140   151   180    61

Based on the histogram, it is clear that the shape of the data is
clearly bimodal, with two distinct peaks occurring around 70 bpm and
around 150 bpm, with a split in the data at around 100 bpm. There are
more songs that are clustered around the higher bpm mode. The center
(median) occurs at 140 bpm, and the spread (IQR) is 61 bpm, indicating
that there is a moderate amount of variability in tempos. There are no
outliers in this distribution. Due to the bimodal result, it seems like
there is a natural grouping between “slow” songs and “fast” songs, so we
will mutate our data set to add a new variable, `tempo`, which is “slow”
if a song’s bpm is less than 100 bpm, “fast” if a song’s tempo is
greater than 100 bpm.

``` r
fight_songs <- fight_songs %>%
  mutate(tempo = case_when(
    bpm <= 100 ~ "slow",
    bpm > 100 ~ "fast"
  ))
```

Now, let’s start exploring our second explanatory variable,
`sec_duration`, which is the duration of a song in seconds. In order to
do this, we will create a histogram of the distribution and find the
relevant summary statistics.

``` r
ggplot(fight_songs, mapping = aes(x = sec_duration)) +
  geom_histogram(binwidth = 20) +
  labs(title = "Duration of College Fight Songs", x = "Duration (s)", y = "Number of Songs")
```

![](data-analysis_files/figure-gfm/histogram-summary-stats-sec_duration-1.png)<!-- -->

``` r
fight_songs %>%
  summarise(min = min(sec_duration), Q1 = quantile(sec_duration, .25), median = median(sec_duration), Q3 = quantile(sec_duration, .75), max = max(sec_duration), IQR = IQR(sec_duration))
```

    ## # A tibble: 1 x 6
    ##     min    Q1 median    Q3   max   IQR
    ##   <dbl> <dbl>  <dbl> <dbl> <dbl> <dbl>
    ## 1    27    58     67    85   172    27

Based on the histogram, we can see that the distribution of fight song
durations is roughly symmetric (slightly skewed to the right) and
unimodal, with a peak at around 70. There are some outliers on the
higher end of the spectrum (`sec_duration` \> 125.5), indicating that
these songs are significantly longer than the others. The center of the
distribution occurs at around 67 seconds, and the IQR of the
distribution is 27 seconds, which is relatively narrow, indicating that
fight songs do not have dramatically different lengths. Like we did with
`bpm`, let’s add a new variable, `length`, which is “short” if a song is
less than or equal to the median of 67 seconds, “long” if a song is
greater than the median of 67 seconds.

``` r
fight_songs <- fight_songs %>%
  mutate(length = case_when(
    sec_duration <= 67 ~ "short",
    sec_duration > 67 ~ "long"
  ))
```

Now that we have an understanding of our two explanatory variables, we
want to add a new variable, `classify`, which combines `bpm` and
`sec_duration` by labeling each song with one of four classifications:
“slow and short”, “slow and long”, “fast and short”, and “fast and
long”.

``` r
fight_songs <- fight_songs %>%
  mutate(classify = case_when(
    tempo == "slow" & length == "short" ~ "slow and short",
    tempo == "slow" & length == "long" ~ "slow and long",
    tempo == "fast" & length == "short" ~ "fast and short",
    tempo == "fast" & length == "long" ~ "fast and long",
  ))

fight_songs %>%
  count(classify)
```

    ## # A tibble: 4 x 2
    ##   classify           n
    ##   <chr>          <int>
    ## 1 fast and long     23
    ## 2 fast and short    25
    ## 3 slow and long      9
    ## 4 slow and short     8

Using these classifications, let’s visualize the groupings and fill in
each observation using the number of tropes associated with that
observtion.

``` r
ggplot(fight_songs, mapping = aes(x = sec_duration, y = bpm)) +
  geom_jitter(aes(color = trope_count), ) + 
  geom_hline(yintercept = 100, linetype = "dashed", color = "orange") +
  geom_vline(xintercept = 67, linetype = "dashed", color = "orange") +
  theme_minimal() +
  labs(title = "Classification of College Fight Songs", 
       subtitle = "by tempo and duration",
       x = "Duration (sec)",
       y = "Tempo (bpm)",
       color = "Number of Tropes") +
  scale_color_gradientn(colours = rainbow(5))
```

![](data-analysis_files/figure-gfm/visualize-classify-1.png)<!-- -->

The upper left hand region of the graph represents songs that are “fast
and short”. The lower left hand region of the graph represents songs
that are “slow and short.” The upper right hand region of the graph
represents songs that are “fast and long”, and the lower right hand
region of the graph represents songs that are “shlow and long.”

Finally, let’s get an understanding of our response variable,
`trope_count`, which is a measure of the number of clichés/tropes in a
given fight song. We will also use a histogram and summary statistics
for this univariate analysis.

``` r
ggplot(fight_songs, mapping = aes(x = trope_count)) +
  geom_histogram(binwidth = 1) +
  labs(title = "Number of Clichés in College Fight Songs", x = "Number of Clichés", y = "Number of Songs")
```

![](data-analysis_files/figure-gfm/histogram-summary-stats-trope_count-1.png)<!-- -->

``` r
fight_songs %>%
  summarise(min = min(trope_count), Q1 = quantile(trope_count, .25), median = median(trope_count), Q3 = quantile(trope_count, .75), max = max(trope_count), IQR = IQR(trope_count), sd = sd(trope_count))
```

    ## # A tibble: 1 x 7
    ##     min    Q1 median    Q3   max   IQR    sd
    ##   <dbl> <dbl>  <dbl> <dbl> <dbl> <dbl> <dbl>
    ## 1     0     3      4     5     8     2  1.67

Based on the histogram and summary statistics, we can see that the
distribution for number of tropes is unimodal with a peak at around 4
and skewed slightly to the left. There is 1 outlier at the maximum value
of our distribution (8 tropes). The center of the distribution is at
around 4 tropes, and the IQR of 2 indicates that there is not a large
amount of variability in the number of tropes for college fight songs.

Now, let’s return to our research question by seeing whether the amount
of clichés varies based on a song’s classification. First, we will
create violin plots for the distribution of the number of tropes for
each classification.

``` r
ggplot(fight_songs, mapping = aes(x = classify, y = trope_count)) +
  geom_violin(draw_quantiles = c(.25, .5, .75)) +
  geom_jitter() +
  labs(title = "Number of Clichés", subtitle = "by Song Classification", x = "Song Classification", y = "Number of Clichés")
```

![](data-analysis_files/figure-gfm/boxplots-summary-statsnumber_tropes-1.png)<!-- -->

``` r
fight_songs %>%
  group_by(classify) %>%
  summarize(median = median(trope_count), IQR = IQR(trope_count), sd = sd(trope_count))
```

    ## # A tibble: 4 x 4
    ##   classify       median   IQR    sd
    ##   <chr>           <dbl> <dbl> <dbl>
    ## 1 fast and long       4   2.5  1.73
    ## 2 fast and short      4   1    1.56
    ## 3 slow and long       5   2    2.28
    ## 4 slow and short      4   2    1.13

Not surprisingly, all of the distributions, except for “slow and long”,
are all centered at 4 tropes, which is thie median number of tropes for
all songs in the data set. However, there is one classification, “slow
and long,” which has a median of 5. We would like to check whether this
median is statistically significant or not. To do this, we will conduct
a hypothesis test for the median number of tropes for songs that are
considered “slow and long”. Our null hypothesis is that the true median
number of tropes for “slow and long” songs is 4, H0: median(“slow and
long”) = 4. Our alternative hypothesis is that the true median number of
tropes is different than 4, Ha: median(“slow and long”) ≠ 4. We will use
an alpha level of 0.05. Let’s create and visualize the null distribution
and calculate the respective p-value.

``` r
slow_long <- fight_songs %>%
  filter(classify == "slow and long")

set.seed(11101962)
null_slow_long <- slow_long %>%
  specify(response = trope_count) %>%
  hypothesize(null = "point", med = 4) %>%
  generate(reps = 1000, type = "bootstrap") %>%
  calculate(stat = "median")

get_p_value(null_slow_long, obs_stat = 5, direction = "both")
```

    ## # A tibble: 1 x 1
    ##   p_value
    ##     <dbl>
    ## 1   0.038

``` r
visualise(null_slow_long) + 
  labs(title = "Null Distribution for Median Number of Tropes",
       subtitle = "for songs that are slow and long",
       x = "Sample Median Number of Tropes",
       y = "Count") +
  shade_p_value(5, "both")
```

![](data-analysis_files/figure-gfm/hypothesis-test-slow-and-long-1.png)<!-- -->

Based on our p-value of 0.038, which is less than alpha = 0.05, we
reject the null hypothesis. There is convincing evidence that the median
number of tropes for songs that are slow and long is different than the
population average of 4. However, our sample size is very small (9
observations), so we must be wary of our results.

We can also see from the visualization “Number of Clichés” that the
spread for “slow and short” songs is a lot smaller compared to all other
song classifications. We will test whether the standard deviation for
“slow and short” songs is significantly different from the population
standard deviation of 1.674182. The null hypothesis is that the true
standard deviation for the number of tropes for “slow and short” songs
is 1.674182, H0: sigma = 1.674182. The alternative hypothesis is tht the
true standard deviation for the number of tropes for “slow and short”
songs is less than 1.674182, Ha: sigma \< 1.674182. We will use an alpha
level of 0.05.

``` r
slow_short <- fight_songs %>%
  filter(classify == "slow and short")

set.seed(11101962)
null_slow_short <- slow_short %>%
  specify(response = trope_count) %>%
  hypothesize(null = "point", sigma = 1.674182) %>%
  generate(reps = 1000, type = "bootstrap") %>%
  calculate(stat = "sd")

get_p_value(null_slow_short, obs_stat = 1.125992, direction = "left")
```

    ## # A tibble: 1 x 1
    ##   p_value
    ##     <dbl>
    ## 1   0.685

``` r
visualise(null_slow_short) + 
  labs(title = "Null Distribution for Standard Deviation of Tropes",
       subtitle = "for songs that are slow and short",
       x = "Sample Standard Deviation of Tropes",
       y = "Count") +
  shade_p_value(1.125992, "left")
```

![](data-analysis_files/figure-gfm/hypothesis-test-slow-and-short-1.png)<!-- -->

Based on the p-value of 0.685, which is greater than our alpha level of
0.05, we fail to reject the null hypothesis. There is insufficient
evidence that the true standard deviation for slow and short songs is
less than the population standard deviation of 1.674182.

Now, let’s find the full linear model that predicts number of tropes
(`trope_count`) from tempo (`bpm`) and duration (`sec_duration`). We can
also check whether there the coefficients for `bpm` and `sec_duration`
are statistically significant by finding their respective p-values and
using an alpha level of 0.05. Let our null hypotheses be that the slopes
associated with both variables are 0, H0: beta(`bpm`) = 0 and
beta(`sec_duration`) = 0. Let our alternative hypotheses be that the
slopes associated with both variables are significantly different than
0, Ha: beta(`bpm`) ≠ 0 and beta(\`sec\_duration) ≠ 0.

``` r
(m_full <- lm(trope_count ~ bpm + sec_duration, fight_songs)) %>%
  tidy()
```

    ## # A tibble: 3 x 5
    ##   term          estimate std.error statistic  p.value
    ##   <chr>            <dbl>     <dbl>     <dbl>    <dbl>
    ## 1 (Intercept)   4.58       1.11       4.13   0.000109
    ## 2 bpm          -0.00757    0.00640   -1.18   0.241   
    ## 3 sec_duration  0.000208   0.00846    0.0246 0.980

``` r
glance(m_full)$AIC
```

    ## [1] 256.96

``` r
glance(m_full)$r.squared
```

    ## [1] 0.02260804

Based on the output, the full linear model is `trope_count`-hat = 4.58 -
0.00757 \* `bpm` + 0.000208 \* `sec_duration`. The R-squared value is
0.022608, which means that approximately 2.2608044% of the variability
in trope counts can be explained by the linear model that predicts trope
count from a song’s tempo and duration. Given this R-squared model, our
model is very weak.

The intercept tells us that for a song with 0 bpm and that is 0 seconds
long, the expected number of tropes is 4.58 (this is nonsensical, as
there is no such thing as a song that is 0 bpm or 0 minutes). The
intercept of -0.00757 for `bpm` tells us that for an increase in 1 bpm,
the number of tropes is expected to decrease by 0.00757. The intercept
of 0.000208 for `sec_duration` tells us that for an increase in a song’s
duration by 1 second, the number of tropes is expected to increase by
0.000208. However, the p-values for the coefficients of `bpm` and
`sec_duration`, 0.241 and 0.980. respectively, are both greater than our
alpha level of 0.05. Therefore, we fail to reject the null hypothesis.
There is insufficient evidence that the coefficients for `bpm` and
`sec_duration` are different than 0.

Circling back to our research question, we now have evidence that our
two explanatory variables, `bpm` and `sec_duration`, do not seem to have
any correlation to the trope count for a given fight song.

In order to make sure that there is no better model, we will use the
`step()` function and use backwards selection with AIC as the selection
criterion.

``` r
(m_trope_count <- step(m_full, direction = "backward")) %>%
  tidy() %>%
  select(term, estimate)
```

    ## Start:  AIC=70.5
    ## trope_count ~ bpm + sec_duration
    ## 
    ##                Df Sum of Sq    RSS    AIC
    ## - sec_duration  1    0.0017 175.33 68.499
    ## - bpm           1    3.9608 179.29 69.950
    ## <none>                      175.33 70.498
    ## 
    ## Step:  AIC=68.5
    ## trope_count ~ bpm
    ## 
    ##        Df Sum of Sq    RSS    AIC
    ## - bpm   1    4.0538 179.38 67.984
    ## <none>              175.33 68.499
    ## 
    ## Step:  AIC=67.98
    ## trope_count ~ 1

    ## # A tibble: 1 x 2
    ##   term        estimate
    ##   <chr>          <dbl>
    ## 1 (Intercept)     3.62

``` r
glance(m_trope_count)$AIC
```

    ## [1] 254.4464

``` r
glance(m_trope_count)$r.squared
```

    ## [1] 0

Since backwards selection removed both `bpm` and `sec_duration` from our
model, we can conclude that these two variables are not valid predictors
for the number of tropes in a college fight song.

### Research Question 2

Our second research question is:

> "How do the characteristics of fight songs of college football teams
> correspond to their respective historical levels of success?

Before we delve deeper into our analysis, let’s first take a look at the
distributions of our four explanatory variables, `victory_win_won`,
`opponents`, `nonsense`, and `rah`, and our response variable, `rank`.

Starting with `victory_win_won`, which is whether the song says
“victory,” “win,” or “won”, we will create a bar graph.

``` r
ggplot(fight_songs, mapping = aes(x = victory_win_won)) +
  geom_bar() +
  labs(title = "Distribution of whether college fight songs include 'victory', 'win', or 'won'", x = "Whether Fight Song Includes 'Victory', 'Win', or 'Won'", y = "Number of Colleges")
```

![](data-analysis_files/figure-gfm/visualize_victory_win_won-1.png)<!-- -->

Next, we will look at `opponents`, which is whether the song says
mentions an opponent. We will create a bar graph.

``` r
ggplot(fight_songs, mapping = aes(x = opponents)) +
  geom_bar() +
  labs(title = "Distribution of whether college fight songs mention opponents", x = "Whether Fight Song Mentions Opponents", y = "Number of Colleges")
```

![](data-analysis_files/figure-gfm/visualize_opponents-1.png)<!-- -->

Next, we will explore `nonsense`, which is whether the song includes any
nonsense words. We will create a bar graph.

``` r
ggplot(fight_songs, mapping = aes(x = nonsense)) +
  geom_bar() +
  labs(title = "Distribution of whether college fight songs include nonsense words", x = "Fight Song Includes Nonsense Words", y = "Number of Colleges")
```

![](data-analysis_files/figure-gfm/visualize_nonsense-1.png)<!-- -->

Next, we will look at `rah`, which is whether the song says the word
“rah”. We will create a bar graph.

``` r
ggplot(fight_songs, mapping = aes(x = rah)) +
  geom_bar() +
  labs(title = "Distribution of whether college fight songs include 'rah'", x = "Fight Song Includes 'Rah'", y = "Number of Colleges")
```

![](data-analysis_files/figure-gfm/visualize_rah-1.png)<!-- -->

Finally, we will look at `rank`, which is the team’s AP college football
ranking. We will create a histogram and calculate the appropriate
summary statistics.

``` r
ggplot(fight_songs, mapping = aes(x = rank)) +
  geom_histogram(bins = 10) +
  labs(title = "Distribution of college football rankings", x = "Rank", y = "Number of Colleges")
```

![](data-analysis_files/figure-gfm/visualize_rank-1.png)<!-- -->

In calculating the summary statistics for `rank`, we will use the median
as a measure of center and the interquartile range as a measure of
spread (due to the skewed nature of the distribution). In addition, we
will find the upper and lower quartiles (Q3 and Q1, respectively), and
the maximum and minimum values:

``` r
fight_songs %>%
  summarise(min = min(rank), Q1 = quantile(rank, .25), median = median(rank), Q3 = quantile(rank, .75), max = max(rank), IQR = IQR(rank))
```

    ## # A tibble: 1 x 6
    ##     min    Q1 median    Q3   max   IQR
    ##   <dbl> <dbl>  <dbl> <dbl> <dbl> <dbl>
    ## 1     1    17     34    55   120    38

Based on the histogram, it is clear that the shape of the data is
unimodal. The distribution is skewed to the right, with more colleges
having higher ranked teams than lower ranked teams. The center (median)
occurs at 34, and the spread (IQR) is 38, indicating that there is a
moderate amount of variability in rankings. There is one outlier (a
college with a rank higher than 112) with a rank of 120.

We hypothesize that a statistically significant relationship exists
between `victory_win_won` and `rank` because it is reasonable to asssume
historically successful college football teams would incorporate
symbolic elements of their dominance into their iconic fight songs. In
this case, we believe the words “victory”, “win”, and “won” are symbolic
elements of dominance, giving us reason to believe higher-ranked teams
are more likely to have their fights songs include these words.

First, I will find the median rank based on `victory_win_won` (yes or
no).

``` r
fight_songs %>%
  group_by(victory_win_won) %>%
  summarise(med_rank = median(rank))
```

    ## # A tibble: 2 x 2
    ##   victory_win_won med_rank
    ##   <chr>              <dbl>
    ## 1 No                  37.5
    ## 2 Yes                 32

I observe a difference of 5.5 (37.5 - 32) in rank between colleges with
and without “victory”, “win”, or “won” in their fight songs.

The null hypothesis is that there is no difference in the median rank of
college football teams between those with and without “victory”, “win”,
or “won” in their fight songs: H0 = mu(victory\_win\_wonNO) -
mu(victory\_win\_wonYES) = 0.

The alternative hypothesis is that there is a difference in the median
rank of college football teams between those with and without “victory”,
“win”, or “won” in their fight songs: H0 = mu(victory\_win\_wonNO) -
mu(victory\_win\_wonYES) =/= 0.

Now, I will run a hypothesis test, calculate the p-value, and interpret
the results in order to determine whether there is a statistically
significant difference in median rank between colleges with/without
“victory”, “win”, or “won” in their fight songs.

``` r
set.seed(08312000)
null_dist <- fight_songs %>%
  specify(response = rank, explanatory = victory_win_won) %>%
  hypothesize(null = "independence") %>% 
  generate(reps = 1000, type = "permute") %>%
  calculate(stat = "diff in medians", 
            order = c("No", "Yes"))

get_p_value(null_dist, obs_stat = 5.5, direction = "both")
```

    ## # A tibble: 1 x 1
    ##   p_value
    ##     <dbl>
    ## 1    0.66

Based on the above output, since the p value, 0.66, is greater than our
alpha level of 0.05, we fail to reject the null hypothesis. In other
words, there is no convincing evidence of a difference in median ranks
of college football teams based on whether their fight songs include the
words “victory”, “win”, or “won”. Our original hypothesis was incorrect.

We also hypothesize that a statistically significant relationship exists
between `opponents` and `rank` since higher-ranked college football
teams are likely to have developed more emotionally-charged rivalries
with their highly successful peers. With such long-standing, emotional
rivalries, it is reasonable to believe fight songs associated with these
highly-ranked college football programs allude to their rivals by name.

First, I will find the median rank based on `opponents` (yes or no).

``` r
fight_songs %>%
  group_by(opponents) %>%
  summarise(med_rank = median(rank))
```

    ## # A tibble: 2 x 2
    ##   opponents med_rank
    ##   <chr>        <dbl>
    ## 1 No            34  
    ## 2 Yes           34.5

I observe a difference of -0.5 (34 - 34.5) in rank between colleges that
do and do not mention opponents in their fight songs.

The null hypothesis is that there is no difference in the median rank of
college football teams between those who do and do not mention their
opponents in their fight songs: H0 = mu(opponentsNO) - mu(opponentsYES)
= 0.

The alternative hypothesis is that there is a difference in the median
rank of college football teams between those who do and do not mention
their opponents in their fight songss: H0 = mu(opponentsNO) -
mu(opponentsYES) =/= 0.

Now, I will run a hypothesis test, calculate the p-value, and interpret
the results in order to determine whether there is a statistically
significant difference in median rank between colleges with/without
mentioning their opponents in their fight songs.

``` r
set.seed(08312000)
null_dist <- fight_songs %>%
  specify(response = rank, explanatory = opponents) %>%
  hypothesize(null = "independence") %>% 
  generate(reps = 1000, type = "permute") %>%
  calculate(stat = "diff in medians", 
            order = c("No", "Yes"))

get_p_value(null_dist, obs_stat = -0.5, direction = "both")
```

    ## # A tibble: 1 x 1
    ##   p_value
    ##     <dbl>
    ## 1   0.962

Based on the above output, since the p value, 0.962, is greater than our
alpha level of 0.05, we fail to reject the null hypothesis. In other
words, there is no convincing evidence of a difference in median ranks
of college football teams based on whether their fight songs mention
their opponents. Our original hypothesis was incorrect.

Moreover, we hypothesize that a statistically significant relationship
exists between `nonsense` and `rank` because there have been a plethora
of articles written about how, in the past, highly successful teams
included nonsensical phrases in their fight songs to distract the
players on the opposing college football teams. Since the fight songs
included in our dataset were written decades ago (as evidenced by the
`year` variable), it is reasonable to assume the songs associated with
the historically-best college teams will have a higher likelihood of
including nonsense (e.g “Hooperay”).

First, I will find the median rank based on `nonsense` (yes or no).

``` r
fight_songs %>%
  group_by(nonsense) %>%
  summarise(med_rank = median(rank))
```

    ## # A tibble: 2 x 2
    ##   nonsense med_rank
    ##   <chr>       <dbl>
    ## 1 No           31  
    ## 2 Yes          42.5

I observe a difference of -11.5 (31 - 42.5) in rank between colleges
with and without nonsense words in their fight songs.

The null hypothesis is that there is no difference in the median rank of
college football teams between those with and without nonsense words in
their fight songs: H0 = mu(nonsenseNO) - mu(nonsenseYES) = 0.

The alternative hypothesis is that there is a difference in the median
rank of college football teams between those with and without nonsense
words in their fight songs: H0 = mu(nonsenseNO) - mu(nonsenseYES) =/= 0.

Now, I will run a hypothesis test, calculate the p-value, and interpret
the results in order to determine whether there is a statistically
significant difference in median rank between colleges with/without
nonsense words in their fight songs.

``` r
set.seed(08312000)
null_dist <- fight_songs %>%
  specify(response = rank, explanatory = nonsense) %>%
  hypothesize(null = "independence") %>% 
  generate(reps = 1000, type = "permute") %>%
  calculate(stat = "diff in medians", 
            order = c("No", "Yes"))

get_p_value(null_dist, obs_stat = -11.5, direction = "both")
```

    ## # A tibble: 1 x 1
    ##   p_value
    ##     <dbl>
    ## 1   0.364

Based on the above output, since the p value, 0.364, is greater than our
alpha level of 0.05, we fail to reject the null hypothesis. In other
words, there is no convincing evidence of a difference in median ranks
of college football teams based on whether their fight songs include
nonsense words. Our original hypothesis was incorrect.

Finally, we hypothesize there is no statistically significant
relationship between `rah` and `rank` since “rah” seems like it would be
a common word in a fight song, irrespective of the quality (how
successful) of a college football team.

First, I will find the median rank based on `rah` (yes or no).

``` r
fight_songs %>%
  group_by(rah) %>%
  summarise(med_rank = median(rank))
```

    ## # A tibble: 2 x 2
    ##   rah   med_rank
    ##   <chr>    <dbl>
    ## 1 No        32  
    ## 2 Yes       37.5

I observe a difference of -5.5 (32 - 37.5) in rank between colleges with
and without “rah” in their fight songs.

The null hypothesis is that there is no difference in the median rank of
college football teams between those with and without “rah” in their
fight songs: H0 = mu(rahNO) - mu(rahYES) = 0.

The alternative hypothesis is that there is a difference in the median
rank of college football teams between those with and without “rah” in
their fight songs: H0 = mu(rahNO) - mu(rahYES) =/= 0.

Now, I will run a hypothesis test, calculate the p-value, and interpret
the results in order to determine whether there is a statistically
significant difference in median rank between colleges with/without
“rah” in their fight songs.

``` r
set.seed(08312000)
null_dist <- fight_songs %>%
  specify(response = rank, explanatory = rah) %>%
  hypothesize(null = "independence") %>% 
  generate(reps = 1000, type = "permute") %>%
  calculate(stat = "diff in medians", 
            order = c("No", "Yes"))

get_p_value(null_dist, obs_stat = -5.5, direction = "both")
```

    ## # A tibble: 1 x 1
    ##   p_value
    ##     <dbl>
    ## 1   0.628

Based on the above output, since the p value, 0.628, is greater than our
alpha level of 0.05, we fail to reject the null hypothesis. In other
words, there is no convincing evidence of a difference in median ranks
of college football teams based on whether their fight songs include
“rah”. Our original hypothesis was correct.

Now, let’s find the full linear model that predicts the rank of a
college football team (`rank`) based on various characteristics of the
college’s fight song: `victory_win_won`, `opponents`, `nonsense`, and
`rah`.

``` r
(rank_full_model <- lm(rank ~ victory_win_won + opponents + nonsense + rah, data = fight_songs)) %>%
  tidy() %>%
  select(term, estimate)
```

    ## # A tibble: 5 x 2
    ##   term               estimate
    ##   <chr>                 <dbl>
    ## 1 (Intercept)           39.0 
    ## 2 victory_win_wonYes    -6.25
    ## 3 opponentsYes          -5.56
    ## 4 nonsenseYes           10.6 
    ## 5 rahYes                 6.80

Based on the output, the full linear model is `rank-hat` = 39.017795 -
6.253008 \* `victory_win_wonYES` - 5.559138 \* `opponentsYes` +
10.557542 \* `nonsenseYes` + 6.797271 \* `rahYes`. The R-squared value
is 0.0622875, which means that there is approximately 6.2287474% of the
variation in rank can be accounted for by the model. Given this
R-squared value, the model is very weak.

The intercept tells us that for a college with “No” responses for
victory\_win\_won, opponents, nonsense, and rah, the expected rank of
the school’s football team is 39.017795. The intercept of -6.253008 for
victory\_win\_wonYES tells us that if a fight song includes the words
“victory”, “win”, or “won”, the rank of the football team is expected
to decrease by 6.253008 The intercept of -5.559138 for opponentsYES
tells us that if a fight song mentions the school’s opponent, the rank
of the football team is expected to decrease by 5.559138. The intercept
of 10.557542 for nonsenseYES tells us that if a fight song includes any
nonsense words, the rank of the football team is expected to increase by
10.557542. The intercept of 6.797271 for rahYes tells us that if a fight
song includes the word “rah”, the rank of the football team is expected
to increase by 6.797271.

In order to make sure that there is no better model, we will use the
step() function and use backwards selection with AIC as the selection
criterion.

``` r
best_model <- step(rank_full_model, direction = "backward") %>%
  tidy() %>%
  select(term, estimate)
```

    ## # A tibble: 1 x 2
    ##   term        estimate
    ##   <chr>          <dbl>
    ## 1 (Intercept)     37.6

Since backwards selection removed `victory_win_won`, `opponents`,
`nonsense`, and `rah` from our model, we can conclude that these four
variables are not valid predictors for the rank of a college football
team.