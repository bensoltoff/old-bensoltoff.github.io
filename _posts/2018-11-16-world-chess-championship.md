---
layout: post
title: "Winning the World Chess Championship"
description: "Expanding on the Riddler's Problem"
output: html_document
date: 2018-11-16 12:00:00 -0600
category: r
tags: [r]
comments: true
---



[This week's Riddler Express from FiveThirtyEight](https://fivethirtyeight.com/features/the-riddler-just-had-to-go-and-reinvent-beer-pong/):

> The World Chess Championship is underway. It is a 12-game match between the world’s top two grandmasters. Many chess fans feel that 12 games is far too short for a biennial world championship match, allowing too much variance.
    Say one of the players is better than his opponent to the degree that he wins 20 percent of all games, loses 15 percent of games and that 65 percent of games are drawn. Wins at this match are worth 1 point, draws a half-point for each player, and losses 0 points. In a 12-game match, the first player to 6.5 points wins.
    What are the chances the better player wins a 12-game match? How many games would a match have to be in order to give the better player a 75 chance of winning the match outright? A 90 percent chance? A 99 percent chance?
    
I'm currently teaching a [math/stats course](https://css18.github.io/) and we've recently covered a ton of different probability distributions. This problem can be defined by the [**multinomial distribution**](https://en.wikipedia.org/wiki/Multinomial_distribution), which is a generalizable form of the binomial distribution. In the original setup of the problem, $n=12$, $k=3$, and probabilities $p_0 = 0.2, p_1 = 0.15, p_2 = 0.6$ for the better player winning, losing, and drawing respectively.

Based on the example [here](https://rpubs.com/JanpuHou/296336), I wrote a generalizable function to estimate the probability of win, lose, and draw for all possible outcomes for any number of $n$ matches and probabilities $p$, and applied it to matches with length between 1 and 300 using the probabilities identified in the problem.


{% highlight r %}
library(tidyverse)
library(magrittr)

theme_set(theme_minimal(base_size = 18))
{% endhighlight %}


{% highlight r %}
chess_outcomes <- function(n_matches, prob){
  # define all possibilities
  X <- expand.grid(n = 0:n_matches,
                   k = 0:3) %>%
    as.matrix %>%
    t
  X <- X[, colSums(X) <= n_matches]
  X <- rbind(X, n_matches:n_matches - colSums(X))
  dimnames(X) <- list(c("win", "lose", "draw"), NULL)
  
  # calculate probabilities of each outcome
  X_prob <- array_branch(X, margin = 2) %>%
    map_dbl(dmultinom, prob = prob)
  
  # combine together
  outcomes <- X %>%
    t %>%
    as_tibble %>%
    mutate(points = win * 1 + lose * 0 + draw * .5,
           prob = X_prob)
  
  return(outcomes)
}
{% endhighlight %}


{% highlight r %}
# iterate over a varying number of matches
varying_matches <- data_frame(n_matches = 1:300) %>%
  mutate(outcomes = map(n_matches, chess_outcomes, prob = c(0.2, 0.15, 0.65)),
         prob_win = map2_dbl(outcomes, n_matches, ~ mean(.x$points > (.y / 2))),
         prob_draw = map2_dbl(outcomes, n_matches, ~ mean(.x$points == (.y / 2))),
         prob_loss = map2_dbl(outcomes, n_matches, ~ mean(.x$points < (.y / 2))))
{% endhighlight %}


{% highlight r %}
ggplot(varying_matches, aes(n_matches, prob_win)) +
  geom_line() +
  scale_y_continuous(labels = scales::percent) +
  labs(title = "Probability of winning the world chess championship",
       x = "Maximum number of matches",
       y = "Probability of victory")
{% endhighlight %}

![center](/figs/2018-11-16-world-chess-championship/plot-results-1.png)

{% highlight r %}
varying_matches %>%
  gather(outcome, prob, starts_with("prob")) %>%
  mutate(outcome = str_remove(outcome, "prob_"),
         outcome = str_to_title(outcome)) %>%
  ggplot(aes(n_matches, prob, color = outcome)) +
  geom_line() +
  scale_y_continuous(labels = scales::percent) +
  scale_color_brewer(type = "qual") +
  labs(title = "Probability of winning the world chess championship",
       x = "Maximum number of matches",
       y = "Probability",
       color = NULL) +
  theme(legend.position = "bottom")
{% endhighlight %}

![center](/figs/2018-11-16-world-chess-championship/plot-results-2.png)

For a standard 12 game match, the probability the better player wins is 0.7826. To achieve a 75% or better probability of success, the match length should be 11. For a 90% chance, it should be a best of 26 games match. For 99%, make that best of 251.
