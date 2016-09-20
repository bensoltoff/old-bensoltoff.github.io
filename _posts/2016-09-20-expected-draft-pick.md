---
layout: post
title: "A Modified Draft Pick Selection Order"
description: "Expanding on the Riddler's Problem"
output: html_document
date: 2016-09-20 12:00:00 -0600
category: r
tags: [r]
comments: true
---



In preparation for teaching a [new computing course for the social sciences](https://uc-cfss.github.io), I've been practicing building interactive websites using [Shiny](http://shiny.rstudio.com/) for R. The [latest Riddler puzzle from FiveThirtyEight](http://fivethirtyeight.com/features/how-high-can-count-von-count-count/) was an especially interesting challenge, combining aspects of computational simulation and Shiny programing:

> You are one of 30 team owners in a professional sports league. In the past, your league set the order for its annual draft using the teams’ records from the previous season — the team with the worst record got the first draft pick, the team with the second-worst record got the next pick, and so on. However, due to concerns about teams intentionally losing games to improve their picks, the league adopts a modified system. This year, each team tosses a coin. All the teams that call their coin toss correctly go into Group A, and the teams that lost the toss go into Group B. All the Group A teams pick before all the Group B teams; within each group, picks are ordered in the traditional way, from worst record to best. If your team would have picked 10th in the old system, what is your expected draft position under the new system?

> Extra credit: Suppose each team is randomly assigned to one of T groups where all the teams in Group 1 pick, then all the teams in Group 2, and so on. (The coin-flipping scenario above is the case where T = 2.) What is the expected draft position of the team with the Nth-best record?

One could go the analytical route to solve this

<blockquote class="twitter-tweet" data-conversation="none" data-lang="en"><p lang="en" dir="ltr">I think I worked out the analytic solution... <a href="https://t.co/HNus4TIZEJ">pic.twitter.com/HNus4TIZEJ</a></p>&mdash; Russell Maier (@MaierRussell) <a href="https://twitter.com/MaierRussell/status/778056486593454080">September 20, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

But I wanted to take a computational, brute-force approach. This type of problem is ripe for Markov chain Monte Carlo (MCMC) methods, which I've use before in [Riddler solutions](http://www.bensoltoff.com/r/can-you-win-this-hot-new-game-show/).

The main task is to write a function that calculates the new draft position for a team given their current draft pick and potential assignment into one of $K$ groups. The function I wrote is:


{% highlight r %}
library(tidyverse)
{% endhighlight %}



{% highlight text %}
## Loading tidyverse: ggplot2
## Loading tidyverse: tibble
## Loading tidyverse: tidyr
## Loading tidyverse: readr
## Loading tidyverse: purrr
## Loading tidyverse: dplyr
{% endhighlight %}



{% highlight text %}
## Conflicts with tidy packages ----------------------------------------------
{% endhighlight %}



{% highlight text %}
## filter(): dplyr, stats
## lag():    dplyr, stats
{% endhighlight %}



{% highlight r %}
draft_pick_sim <- function(n_teams = 30, n_groups = 2, n_sims = 100){
  old <- 1:n_teams

  sims <- replicate(n_sims, sample(1:n_groups, n_teams, replace = T)) %>%
    tbl_df %>%
    bind_cols(data_frame(old)) %>%
    gather(sim, outcome, -old) %>%
    group_by(sim) %>%
    arrange(sim, outcome, old) %>%
    mutate(new = row_number())
  
  return(sims)
}
{% endhighlight %}

For each simulation, I randomly sample each team into one of `n_groups`, then calculate draft order from worst-to-first within each group and then between groups. From this I can then calculate the expected draft position for each team given their original draft order.

So given the original problem setup, the expected draft positions for each team given random assignment into one of two groups is:


{% highlight r %}
draft_pick_sim(n_sims = 10000) %>%
  group_by(old) %>%
  summarize(mean = mean(new)) %>%
  knitr::kable(caption = "Expected Draft Position (based on 10,000 simulations)",
               col.names = c("Original Draft Position",
                             "Expected Draft Position"))
{% endhighlight %}



| Original Draft Position| Expected Draft Position|
|-----------------------:|-----------------------:|
|                       1|                    8.32|
|                       2|                    8.76|
|                       3|                    9.08|
|                       4|                    9.73|
|                       5|                   10.32|
|                       6|                   10.79|
|                       7|                   11.23|
|                       8|                   11.87|
|                       9|                   12.28|
|                      10|                   12.67|
|                      11|                   13.20|
|                      12|                   13.61|
|                      13|                   14.36|
|                      14|                   14.58|
|                      15|                   15.29|
|                      16|                   15.79|
|                      17|                   16.06|
|                      18|                   16.82|
|                      19|                   17.31|
|                      20|                   17.57|
|                      21|                   18.39|
|                      22|                   18.87|
|                      23|                   19.29|
|                      24|                   19.74|
|                      25|                   20.38|
|                      26|                   20.68|
|                      27|                   21.29|
|                      28|                   21.72|
|                      29|                   22.21|
|                      30|                   22.78|

The team originally with the 10th draft can expect to have the *13th pick* under this new approach.

What turned into the more complicated part was turning this function into a working Shiny app. [I encourage you to try it out](https://bensoltoff.shinyapps.io/draft_pick/), as it generalizes the problem by providing expected draft picks given $N$ teams and $K$ groups.


