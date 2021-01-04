---
layout: post
title:  "Using Tracking Shot Filter Data To Measure Shot Difficulty and Shot Making"
date:   2020-08-31T09:09:00Z
tags: basketball
---

The tracking shot data on [NBA Stats](https://stats.nba.com/) allows us to get shooting data based on closest defender distance, shot clock range, touch time and number of dribbles. Using this data we can get a rough measure of shot making by comparing how they shoot compared to the league average for each tracking shot filter bucket. An example of a filter bucket would be: closest defender 4-6 feet, shot clock 15-7, touch time < 2 seconds, 0 dribbles, 3pt FGA. We can calculate what a player or team's expected eFG% would be if they shot the league average percentage for each filter bucket and maintained their distribution of shots across each filter bucket. It's not perfect because there can still be variations in shot difficultly within a filter bucket, but it's the best we can do with public data at this time.

If we compare look at the difference between actual eFG% and expected eFG% we can see who the best and worst shot makers are. Among high usage players in the playoffs so far Jamal Murray and Donovan Mitchell have been by far the best shot makers, with Pascal Siakam and Eric Gordon the worst.

![shot_making]({{ "/assets/shot_making.png" | absolute_url }})

If we look at just the expected eFG% we can see who takes the toughest and easiet shots. Khris Middleton and Kawhi Leonard have taken the toughest shots in the playoffs, with LeBron and Giannis taking the easiest shots.

![shot_difficulty]({{ "/assets/shot_difficulty.png" | absolute_url }})
