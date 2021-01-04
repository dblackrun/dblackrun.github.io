---
layout: post
title:  "NBA Possession Data"
date:   2018-04-17T09:09:00Z
tags: basketball
---

I have decided to share my possession details dataset for the 2017-18 regular season. This data is the starting point for all stats on [PBPStats.com](https://www.pbpstats.com/). I have parsed out possession details from play by play data to create detailed stats for each possession. I spent dozens of hours putting this together and I think there is a lot that can be done with this data. By making it public I'm hoping people will do some stuff with it that I would have never thought of doing.

* [Download Link](https://s3.amazonaws.com/pbpstats/db_dumps/possession_details_00217.csv.zip)

Here is the schema:

* GameId - stats.nba.com game id
* Period
* PossessionNumber - starting at 1 for each period
* OffenseTeamId - stats.nba.com team id
* DefenseTeamId - stats.nba.com team id
* StartTime - seconds remaining in period at start of possession
* EndTime - seconds remaining in period at end of possession
* StartType - how the previous possession ended, ex. OffAtRimMiss
* StartScoreDifferential - at the start of the possession, from perspective of the team on offense
* OffensiveRebounds - number of offensive rebounds on the possession
* SecondChanceTime - seconds from when offensive rebound was secured until end of possession
* PreviousPossessionEndShooterPlayerId - stats.nba.com player id, exists if previous possession ends on a missed shot
* PreviousPossessionEndReboundPlayerId - stats.nba.com player id, exists if previous possession ends on a missed shot
* PreviousPossessionEndTurnoverPlayerId - stats.nba.com player id, exists if previous possession ends on a steal
* PreviousPossessionEndStealPlayerId - stats.nba.com player id, exists if previous possession ends on a steal
* PlayerStats - JSON, format:
```
  {
    "TeamId": {
      "LineupId": {
        "OpponentLineupId": {
          "PlayerId": {
            "StatKey": StatValue,
            ...
          }
        }
      }
    }
  }
```
Lineup ids are "-" separated player ids. Team rebounds and team turnovers are counted with the player id "0". Each StatKey should be pretty self-explanatory.

A few other notes:

* I use the OffPoss/DefPoss stat keys to count possessions.
* When there is a substitution in the middle of a possession, only the players that finish the possession get credited with the possession count.
* Any possession that starts in the final 2 seconds of a period that doesn't have points scored appears as a row in the table, but won't have a OffPoss/DefPoss stat key for any of the players. This way end of quarter heaves won't count towards possession counts.
* Play by play data is imperfect and human error can create some issues trying to parse out possessions. I have done the best I can to sort some of these issues out, but I'm sure there are still issues I haven't discovered that may be causing some things in this dataset to be off. If you notice anything that seems off let me know.
