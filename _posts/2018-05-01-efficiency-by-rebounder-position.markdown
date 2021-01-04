---
layout: post
title:  "NBA Efficiency by Rebounder Position"
date:   2018-05-01T09:09:00Z
tags: basketball
---

I recently shared my [possession data](https://dblackrun.github.io/2018/04/17/nba-possession-data.html) and wrote a brief [post](https://dblackrun.github.io/2018/04/18/digging-deeper-pace.html) on how to use it to dig deeper into pace. Today I'm going to do some exploratory analysis on that data to breakdown the efficiency on possessions following a missed field goal based on the position of the rebounder. Intuition would suggest that when a primary ball handler gets a defensive rebound it should create more fast break opportunities and lead to a more efficient offense. Let's see if the data backs that up.


```python
import pandas as pd
import json
```


```python
possessions = pd.read_csv('possession_details_00217.csv')
```

Possessions off a live ball rebound will have a player id in the 'PreviousPossessionEndReboundPlayerId' column. I'm also going to exclude possessions following missed free throws since possessions after missed free throws are less free flowing than after missed field goals.


```python
off_fg_rebound = possessions[(possessions['StartType'] != 'OffFTMiss') | (possessions['PreviousPossessionEndReboundPlayerId'] != 0)]
off_fg_rebound.is_copy = False
```

The 'PlayerStats' column contains all stats in a nested dictionary. Here is a helper function that can be used to sum up a list of keys.


```python
def sum_player_stats_keys_for_possession(row, keys):
    """
    keys - list if keys to be summed up
    """
    team_id = str(row['OffenseTeamId'])
    player_stats_dict = json.loads(row['PlayerStats'])

    value = 0

    for lineup_id in player_stats_dict[team_id].keys():
        for opponent_lineup_id in player_stats_dict[team_id][lineup_id].keys():
            for player_id in player_stats_dict[team_id][lineup_id][opponent_lineup_id].keys():
                player_stats = player_stats_dict[team_id][lineup_id][opponent_lineup_id][player_id]
                for key in keys:
                    value += player_stats.get(key, 0)
    return value
```

Here are all the possible stat keys to sum up to get total points.


```python
points_keys = ['FtsMade', 'AssistedAtRim', 'AssistedAtRim', 'AssistedShortMidRange', 'AssistedShortMidRange', 'AssistedLongMidRange', 'AssistedLongMidRange', 'AssistedCorner3', 'AssistedCorner3', 'AssistedCorner3', 'AssistedArc3', 'AssistedArc3', 'AssistedArc3','UnassistedAtRim', 'UnassistedAtRim', 'UnassistedShortMidRange', 'UnassistedShortMidRange', 'UnassistedLongMidRange', 'UnassistedLongMidRange', 'UnassistedCorner3', 'UnassistedCorner3', 'UnassistedCorner3', 'UnassistedArc3', 'UnassistedArc3', 'UnassistedArc3']
```

We can then use the apply function to add a new column to the table with the sum of the keys. Note for the possession counts we need to divide the total by 5 since there is a value for each player on the floor for each possession.


```python
off_fg_rebound['points'] = off_fg_rebound.apply(sum_player_stats_keys_for_possession, axis=1, args=(points_keys,))
off_fg_rebound['possessions'] = off_fg_rebound.apply(sum_player_stats_keys_for_possession, axis=1, args=(['OffPoss'],)) / 5
```

I'm going to use the positions on [basketball-reference](https://www.basketball-reference.com/). These aren't perfect but are fine for some exploratory analysis.


```python
positions = pd.read_csv('positions.csv')
```

We need to merge the two tables by the player id.


```python
off_fg_rebound = off_fg_rebound.merge(positions, left_on='PreviousPossessionEndReboundPlayerId', right_on='player_id')
```

We can now group by position and sum up the total points and possessions for each position


```python
by_position = off_fg_rebound[['position', 'points', 'possessions']].groupby('position').sum()
by_position['efficiency'] = by_position['points'] / by_position['possessions']
by_position
```



<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>points</th>
      <th>possessions</th>
      <th>efficiency</th>
    </tr>
    <tr>
      <th>position</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>C</th>
      <td>26224</td>
      <td>23888.0</td>
      <td>1.097790</td>
    </tr>
    <tr>
      <th>PF</th>
      <td>20642</td>
      <td>18707.0</td>
      <td>1.103437</td>
    </tr>
    <tr>
      <th>SF</th>
      <td>12694</td>
      <td>11616.0</td>
      <td>1.092803</td>
    </tr>
    <tr>
      <th>SG</th>
      <td>15917</td>
      <td>14306.0</td>
      <td>1.112610</td>
    </tr>
    <tr>
      <th>PG</th>
      <td>13970</td>
      <td>12355.0</td>
      <td>1.130716</td>
    </tr>
  </tbody>
</table>
</div>



So there is a small boost to efficiency following defensive rebounds by point guards. What about fast break opportunities? Let's look at how often teams score within 5 seconds of securing a rebound.


```python
score_within_5_seconds = off_fg_rebound[(off_fg_rebound['points'] > 0) & (off_fg_rebound['StartTime'] - off_fg_rebound['EndTime'] < 5)]
```


```python
score_within_5_seconds[['position', 'possessions']].groupby('position').sum()
```


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>possessions</th>
    </tr>
    <tr>
      <th>position</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>C</th>
      <td>1744.0</td>
    </tr>
    <tr>
      <th>PF</th>
      <td>1575.0</td>
    </tr>
    <tr>
      <th>SF</th>
      <td>1075.0</td>
    </tr>
    <tr>
      <th>SG</th>
      <td>1451.0</td>
    </tr>
    <tr>
      <th>PG</th>
      <td>1628.0</td>
    </tr>
  </tbody>
</table>
</div>


Teams score within 5 seconds 7.3% (1744/23888) of the time after a center gets a defensive rebound and 13.2% (1628/12355) of the time after a point guard gets a defensive rebound. So when point guards get defensive rebounds there are more transition baskets scored.

This is nothing ground breaking. It doesn't mean teams should go out of their way to have their point guard gather all defensive rebounds, it could just be that point guards get longer rebounds which lead to more fast breaks. Deeper analysis should be done before we draw a real conclusion, but it's nice to see the data match the intuition on the surface.
