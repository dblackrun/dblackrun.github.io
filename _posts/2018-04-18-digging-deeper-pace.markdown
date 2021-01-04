---
layout: post
title:  "Digging Deeper Into Pace in the NBA"
date:   2018-04-18T09:09:00Z
tags: basketball
---

In NBA stats pace is the number of possessions a team gets per 48 minutes. It is used as a measure of how fast a team plays. While it is generally a good measure, it is flawed in a few ways. The first reason is that it doesn't separate offensive pace and defensive pace. When pace is brought up, often people are talking about the pace of a team's offense. A team that plays fast on offense and forces their opponents to play slow on defense may not have a fast pace by the traditional definition. The second is that getting offense rebounds slows down your pace. A team that generates quick shots but gets a lot of offensive rebounds may have a slower pace number than a team that is slower to get their first shot on a possession but doesn't offensive rebound as often. Using python and the possession data I shared [here](https://dblackrun.github.io/2018/04/17/nba-possession-data.html) I'm going to calculate the average time of possession excluding second chance time on offense and defense and compare those to the pace numbers.


```python
import pandas as pd
```


```python
possessions = pd.read_csv('possession_details_00217.csv')
```

We have the start time, end time and second chance time in the table already. Let's create a new column for possession length, excluding second chance time, by subtracting the end time and second chance time from the start time. We will call this column "first_chance_time".


```python
possessions['first_chance_time'] = possessions['StartTime'] - possessions['EndTime'] - possessions['SecondChanceTime']
```

The table has data on all possessions, including end of period possessions where a team has no real chance to score with a last second heave. As I noted [here](https://dblackrun.github.io/2018/04/17/nba-possession-data.html), I count possessions by including the "OffPoss" or "DefPoss" key in the "PlayerStats" column. Those end of period possessions where a team has no real chance to score won't have the "OffPoss" key, so if we keep only the rows with the "OffPoss" key we can remove those possessions. Note that when we read the csv the type on the "PlayerStats" column is a string.  If you want to sum up stats in this column you will need to convert it to a dictionary. For our purposes though, we can just check if the string contains the "OffPoss" string.


```python
counted_possessions = possessions[possessions.PlayerStats.str.contains('OffPoss')]
```

Now we can use the groupby function to calculate the mean of the "first_chance_time" column for each team on offense.


```python
first_chance_time_by_team = counted_possessions[['OffenseTeamId', 'first_chance_time']].groupby('OffenseTeamId').mean()
```

Let's take a look at the results.


```python
first_chance_time_by_team
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>first_chance_time</th>
    </tr>
    <tr>
      <th>OffenseTeamId</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1610612737</th>
      <td>14.302262</td>
    </tr>
    <tr>
      <th>1610612738</th>
      <td>14.614438</td>
    </tr>
    <tr>
      <th>1610612739</th>
      <td>14.161539</td>
    </tr>
    <tr>
      <th>1610612740</th>
      <td>13.371514</td>
    </tr>
    <tr>
      <th>1610612741</th>
      <td>13.755922</td>
    </tr>
    <tr>
      <th>1610612742</th>
      <td>15.117969</td>
    </tr>
    <tr>
      <th>1610612743</th>
      <td>14.296213</td>
    </tr>
    <tr>
      <th>1610612744</th>
      <td>13.349889</td>
    </tr>
    <tr>
      <th>1610612745</th>
      <td>14.018915</td>
    </tr>
    <tr>
      <th>1610612746</th>
      <td>13.876505</td>
    </tr>
    <tr>
      <th>1610612747</th>
      <td>13.315304</td>
    </tr>
    <tr>
      <th>1610612748</th>
      <td>14.780122</td>
    </tr>
    <tr>
      <th>1610612749</th>
      <td>13.994308</td>
    </tr>
    <tr>
      <th>1610612750</th>
      <td>14.925630</td>
    </tr>
    <tr>
      <th>1610612751</th>
      <td>14.375985</td>
    </tr>
    <tr>
      <th>1610612752</th>
      <td>14.571052</td>
    </tr>
    <tr>
      <th>1610612753</th>
      <td>14.222292</td>
    </tr>
    <tr>
      <th>1610612754</th>
      <td>14.358370</td>
    </tr>
    <tr>
      <th>1610612755</th>
      <td>13.513031</td>
    </tr>
    <tr>
      <th>1610612756</th>
      <td>13.847236</td>
    </tr>
    <tr>
      <th>1610612757</th>
      <td>14.813750</td>
    </tr>
    <tr>
      <th>1610612758</th>
      <td>14.945469</td>
    </tr>
    <tr>
      <th>1610612759</th>
      <td>14.961810</td>
    </tr>
    <tr>
      <th>1610612760</th>
      <td>13.656384</td>
    </tr>
    <tr>
      <th>1610612761</th>
      <td>14.115663</td>
    </tr>
    <tr>
      <th>1610612762</th>
      <td>14.796022</td>
    </tr>
    <tr>
      <th>1610612763</th>
      <td>14.996125</td>
    </tr>
    <tr>
      <th>1610612764</th>
      <td>14.424539</td>
    </tr>
    <tr>
      <th>1610612765</th>
      <td>14.491313</td>
    </tr>
    <tr>
      <th>1610612766</th>
      <td>14.263511</td>
    </tr>
  </tbody>
</table>
</div>



This gives us the average time for each team but with just the team id, it's not very useful. I have a map of team id to team abbreviation that we can use to merge with the above table.


```python
team_id_abbreviation_map = {
    1610612737: 'ATL',
    1610612738: 'BOS',
    1610612739: 'CLE',
    1610612740: 'NOP',
    1610612741: 'CHI',
    1610612742: 'DAL',
    1610612743: 'DEN',
    1610612744: 'GSW',
    1610612745: 'HOU',
    1610612746: 'LAC',
    1610612747: 'LAL',
    1610612748: 'MIA',
    1610612749: 'MIL',
    1610612750: 'MIN',
    1610612751: 'BKN',
    1610612752: 'NYK',
    1610612753: 'ORL',
    1610612754: 'IND',
    1610612755: 'PHI',
    1610612756: 'PHX',
    1610612757: 'POR',
    1610612758: 'SAC',
    1610612759: 'SAS',
    1610612760: 'OKC',
    1610612761: 'TOR',
    1610612762: 'UTA',
    1610612763: 'MEM',
    1610612764: 'WAS',
    1610612765: 'DET',
    1610612766: 'CHA',
}

teams = pd.DataFrame.from_dict(team_id_abbreviation_map, orient='index')
teams.columns = ['Team']

first_chance_time_by_team = first_chance_time_by_team.merge(teams, left_index=True, right_index=True)
```

Now we can sort the results.


```python
first_chance_time_by_team.sort_values(by=['first_chance_time'])
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>first_chance_time</th>
      <th>Team</th>
    </tr>
    <tr>
      <th>OffenseTeamId</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1610612747</th>
      <td>13.315304</td>
      <td>LAL</td>
    </tr>
    <tr>
      <th>1610612744</th>
      <td>13.349889</td>
      <td>GSW</td>
    </tr>
    <tr>
      <th>1610612740</th>
      <td>13.371514</td>
      <td>NOP</td>
    </tr>
    <tr>
      <th>1610612755</th>
      <td>13.513031</td>
      <td>PHI</td>
    </tr>
    <tr>
      <th>1610612760</th>
      <td>13.656384</td>
      <td>OKC</td>
    </tr>
    <tr>
      <th>1610612741</th>
      <td>13.755922</td>
      <td>CHI</td>
    </tr>
    <tr>
      <th>1610612756</th>
      <td>13.847236</td>
      <td>PHX</td>
    </tr>
    <tr>
      <th>1610612746</th>
      <td>13.876505</td>
      <td>LAC</td>
    </tr>
    <tr>
      <th>1610612749</th>
      <td>13.994308</td>
      <td>MIL</td>
    </tr>
    <tr>
      <th>1610612745</th>
      <td>14.018915</td>
      <td>HOU</td>
    </tr>
    <tr>
      <th>1610612761</th>
      <td>14.115663</td>
      <td>TOR</td>
    </tr>
    <tr>
      <th>1610612739</th>
      <td>14.161539</td>
      <td>CLE</td>
    </tr>
    <tr>
      <th>1610612753</th>
      <td>14.222292</td>
      <td>ORL</td>
    </tr>
    <tr>
      <th>1610612766</th>
      <td>14.263511</td>
      <td>CHA</td>
    </tr>
    <tr>
      <th>1610612743</th>
      <td>14.296213</td>
      <td>DEN</td>
    </tr>
    <tr>
      <th>1610612737</th>
      <td>14.302262</td>
      <td>ATL</td>
    </tr>
    <tr>
      <th>1610612754</th>
      <td>14.358370</td>
      <td>IND</td>
    </tr>
    <tr>
      <th>1610612751</th>
      <td>14.375985</td>
      <td>BKN</td>
    </tr>
    <tr>
      <th>1610612764</th>
      <td>14.424539</td>
      <td>WAS</td>
    </tr>
    <tr>
      <th>1610612765</th>
      <td>14.491313</td>
      <td>DET</td>
    </tr>
    <tr>
      <th>1610612752</th>
      <td>14.571052</td>
      <td>NYK</td>
    </tr>
    <tr>
      <th>1610612738</th>
      <td>14.614438</td>
      <td>BOS</td>
    </tr>
    <tr>
      <th>1610612748</th>
      <td>14.780122</td>
      <td>MIA</td>
    </tr>
    <tr>
      <th>1610612762</th>
      <td>14.796022</td>
      <td>UTA</td>
    </tr>
    <tr>
      <th>1610612757</th>
      <td>14.813750</td>
      <td>POR</td>
    </tr>
    <tr>
      <th>1610612750</th>
      <td>14.925630</td>
      <td>MIN</td>
    </tr>
    <tr>
      <th>1610612758</th>
      <td>14.945469</td>
      <td>SAC</td>
    </tr>
    <tr>
      <th>1610612759</th>
      <td>14.961810</td>
      <td>SAS</td>
    </tr>
    <tr>
      <th>1610612763</th>
      <td>14.996125</td>
      <td>MEM</td>
    </tr>
    <tr>
      <th>1610612742</th>
      <td>15.117969</td>
      <td>DAL</td>
    </tr>
  </tbody>
</table>
</div>



So the Lakers, Warriors and Pelicans are the fastest teams, and the Spurs, Grizzlies and Mavericks are the slowest. We can perform the same exercise for the team on defense.


```python
opponent_first_chance_time_by_team = counted_possessions[['DefenseTeamId', 'first_chance_time']].groupby('DefenseTeamId').mean()
opponent_first_chance_time_by_team = opponent_first_chance_time_by_team.merge(teams, left_index=True, right_index=True)
opponent_first_chance_time_by_team.sort_values(by=['first_chance_time'])
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>first_chance_time</th>
      <th>Team</th>
    </tr>
    <tr>
      <th>DefenseTeamId</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1610612751</th>
      <td>13.731740</td>
      <td>BKN</td>
    </tr>
    <tr>
      <th>1610612756</th>
      <td>13.873823</td>
      <td>PHX</td>
    </tr>
    <tr>
      <th>1610612737</th>
      <td>13.972635</td>
      <td>ATL</td>
    </tr>
    <tr>
      <th>1610612757</th>
      <td>13.982557</td>
      <td>POR</td>
    </tr>
    <tr>
      <th>1610612750</th>
      <td>13.989717</td>
      <td>MIN</td>
    </tr>
    <tr>
      <th>1610612742</th>
      <td>14.030748</td>
      <td>DAL</td>
    </tr>
    <tr>
      <th>1610612752</th>
      <td>14.048389</td>
      <td>NYK</td>
    </tr>
    <tr>
      <th>1610612753</th>
      <td>14.060700</td>
      <td>ORL</td>
    </tr>
    <tr>
      <th>1610612766</th>
      <td>14.145543</td>
      <td>CHA</td>
    </tr>
    <tr>
      <th>1610612759</th>
      <td>14.177091</td>
      <td>SAS</td>
    </tr>
    <tr>
      <th>1610612746</th>
      <td>14.195889</td>
      <td>LAC</td>
    </tr>
    <tr>
      <th>1610612739</th>
      <td>14.203967</td>
      <td>CLE</td>
    </tr>
    <tr>
      <th>1610612763</th>
      <td>14.213172</td>
      <td>MEM</td>
    </tr>
    <tr>
      <th>1610612755</th>
      <td>14.223131</td>
      <td>PHI</td>
    </tr>
    <tr>
      <th>1610612764</th>
      <td>14.224541</td>
      <td>WAS</td>
    </tr>
    <tr>
      <th>1610612748</th>
      <td>14.233789</td>
      <td>MIA</td>
    </tr>
    <tr>
      <th>1610612762</th>
      <td>14.250223</td>
      <td>UTA</td>
    </tr>
    <tr>
      <th>1610612740</th>
      <td>14.269125</td>
      <td>NOP</td>
    </tr>
    <tr>
      <th>1610612758</th>
      <td>14.306128</td>
      <td>SAC</td>
    </tr>
    <tr>
      <th>1610612743</th>
      <td>14.309011</td>
      <td>DEN</td>
    </tr>
    <tr>
      <th>1610612738</th>
      <td>14.327065</td>
      <td>BOS</td>
    </tr>
    <tr>
      <th>1610612747</th>
      <td>14.346368</td>
      <td>LAL</td>
    </tr>
    <tr>
      <th>1610612765</th>
      <td>14.382196</td>
      <td>DET</td>
    </tr>
    <tr>
      <th>1610612761</th>
      <td>14.404660</td>
      <td>TOR</td>
    </tr>
    <tr>
      <th>1610612754</th>
      <td>14.464885</td>
      <td>IND</td>
    </tr>
    <tr>
      <th>1610612745</th>
      <td>14.528096</td>
      <td>HOU</td>
    </tr>
    <tr>
      <th>1610612744</th>
      <td>14.592067</td>
      <td>GSW</td>
    </tr>
    <tr>
      <th>1610612741</th>
      <td>14.603764</td>
      <td>CHI</td>
    </tr>
    <tr>
      <th>1610612749</th>
      <td>14.932726</td>
      <td>MIL</td>
    </tr>
    <tr>
      <th>1610612760</th>
      <td>14.991538</td>
      <td>OKC</td>
    </tr>
  </tbody>
</table>
</div>



OKC really illustrates the issues when you just look at pace. If you go by [pace](https://www.basketball-reference.com/leagues/NBA_2018.html#misc_stats::12), they are a mid-pack team that plays at a slightly slower than average pace. You may wonder why a team with a one man fast break like Russell Westbrook isn't playing at a faster pace. We can dig deeper and see why that may not actually be the case. When you split up offense and defense and exclude second chance time, they average 13.7 seconds per possession on offense, 5th fastest in the league, while their opponents average 15 seconds per possession, the slowest in the league. So they play fast on offense and force opponents to play slow on defense. They also lead the league in offensive rebounding rate, so all those second chances will slow down their pace numbers. This leads to them having a below average number for pace despite playing pretty fast on offense.

Pace is fine as a general measure, but when used to describe how fast a team plays on offense it can lead to some incorrect conclusions. Digging deeper by splitting up offense and defense can give us a better idea of the speed at which a team plays.
