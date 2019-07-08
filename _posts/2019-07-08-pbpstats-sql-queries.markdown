---
layout: post
title:  "SQL Queries for PBPStats Database"
date:   2019-07-08T09:09:00Z
---

I've created a [Docker image](https://github.com/dblackrun/pbpstats-docker) with a PostgreSQL database with 2018-19 NBA data I use for [pbpstats.com.](https://www.pbpstats.com/) It is queryable in a web browser using pgadmin. The instructions for getting started are in the github repo. I'm sharing some queries to show what types of things are possible. The main table is the possession_details table, which has a row with data for each possession. Here is the schema, it should mostly be self-explanatory:

# entity_data
* entityid (all player and team ids are nba.com ids. lineup ids are '-' separated player ids)
* entitytype
* name
* teamid
* seasonkey
* league
* shortname

# game_data
* gameid (nba.com game id)
* teamid
* opponentteamid
* location
* gamedate


# game_logs
* gameid
* entityid
* entitytype
* teamid
* data (json with key-value pairs for all stats)

# possession_details
* gameid
* period
* possessionnumber
* offenseteamid
* defenseteamid
* starttime (time remaining in the period, in seconds, when the possession started)
* endtime (time remaining in the period, in seconds, when the possession ended)
* previouspossessionendeventnum
* endeventnum
* starttype
* startscoredifferential (from perspective of the offensive team)
* offensiverebounds (number of offensive rebounds on the possession)
* secondchancetime
* previouspossessionendshooterplayerid (if previous possession ended with a missed fg, this will have the player id of the player who missed the shot)
* previouspossessionendreboundplayerid (if previous possession ended with a missed fg, this will have the player id of the player who rebounded the shot)
* previouspossessionendturnoverplayerid (if previous possession ended with a live ball turnover, this will have the player id of the player who turned the ball over)
* previouspossessionendstealplayerid (if previous possession ended with a live ball turnover, this will have the player id of the player who stole the ball)
* playerstats - format:
```
  {
    team id: {
      lineup id: {
        opponent lineup id: {
          player id: {
            stat key: stat value,
            ...
          }
        }
      }
    }
  }
```
* shotdata (list with data on each shot)
* events
* previouspossessionevents


# season_totals
* entityid
* entitytype
* teamid
* seasonkey
* starttype (possession start type)
* data (json with key-value pairs for all stats)
* league


# starter_state_season_totals
* entityid
* entitytype
* teamid
* seasonkey
* starterstate (starter state from the perspective of the team. ex 4v3 means team had 4 starters on and opponent had 3. starters are defined on a game-by-game basis)
* data (json with key-value pairs for all stats)
* league

Here are a few sample queries:

Get detailed shot logs:
```
SELECT gameid,
	period,
	possessionnumber,
	offenseteamid,
	defenseteamid,
	starttime,
	endtime,
	starttype,
	previouspossessionENDshooterplayerid,
	previouspossessionENDreboundplayerid,
	previouspossessionENDturnoverplayerid,
	previouspossessionENDstealplayerid,
	shot_data.*
FROM possession_details,
	jsonb_to_recordset(possession_details.shotdata) AS shot_data("X" FLOAT, "Y" FLOAT, "PlayerId" int, "ShotValue" int, "Made" BOOLEAN, "Time" FLOAT, "ScoreMargin" int, "ShotQuality" FLOAT, "Blocked" BOOLEAN, "BlockPlayerId" int, "Assisted" BOOLEAN, "AssistPlayerId" int, "Putback" BOOLEAN, "ShotType" VARCHAR, "OrebShotPlayerId" int, "OrebReboundPlayerId" int, "OrebShotType" VARCHAR, "SecondsSinceOReb" FLOAT, "LineupId" VARCHAR, "OpponentLineupId" VARCHAR)
```

To get assist combos
```
SELECT
assist_ed.name AS assist_player,
scorer_ed.name AS scorer_player,
c.*,
c.atrim + c.shortmidrange + c.longmidrange AS total2s,
c.arc3 + c.corner3 AS total3s,
c.atrim + c.shortmidrange + c.longmidrange + c.arc3 + c.corner3 AS total
FROM
(
	SELECT
	b.assistpid,
	b.scorerpid,
	b.teamid,
	COALESCE(SUM(CASE when b.shot_type = 'AtRim' THEN b.stat_value else 0 END), 0) AS atrim,
	COALESCE(SUM(CASE when b.shot_type = 'ShortMidRange' THEN b.stat_value END), 0) AS shortmidrange,
	COALESCE(SUM(CASE when b.shot_type = 'LongMidRange' THEN b.stat_value END), 0) AS longmidrange,
	COALESCE(SUM(CASE when b.shot_type = 'Arc3' THEN b.stat_value END), 0) AS arc3,
	COALESCE(SUM(CASE when b.shot_type = 'Corner3' THEN b.stat_value END), 0) AS corner3
	FROM
	(
		SELECT
		split_part(a.stat_key, ':', 1) AS assistpid,
		split_part(a.stat_key, ':', 3) AS scorerpid,
		a.teamid,
		split_part(a.stat_key, ':', 4) AS shot_type,
		a.stat_value::int
		FROM
		(
			SELECT entityid AS teamid, (jsonb_each(season_totals.data)).key AS stat_key, (jsonb_each_text(season_totals.data)).value AS stat_value
			FROM season_totals
			where entitytype = 'Team'
			AND starttype = 'All'
		) a
		where a.stat_key like '%AssistsTo%'
	) b
	group by b.assistpid, b.scorerpid, b.teamid
) c
inner join (SELECT * FROM entity_data where entitytype = 'Player') assist_ed
ON c.assistpid = assist_ed.entityid
AND c.teamid = assist_ed.teamid
inner join (SELECT * FROM entity_data where entitytype = 'Player') scorer_ed
ON c.scorerpid = scorer_ed.entityid
AND c.teamid = scorer_ed.teamid
ORDER BY total DESC
```

Get Utah's points given up and number of possessions after a Ricky Rubio live ball turnover:
```
SELECT sum(COALESCE((a.stats->>'OffPoss')::FLOAT, 0)/5) AS off_poss, sum(COALESCE((a.stats->>'PlusMinus')::FLOAT, 0)/5) AS points -- sum gets total, but each player on the floor is credited with these stats, so we need to divide by 5 to get team total
FROM
(
	SELECT (jsonb_each(playerstats)).key AS team_id,
		(jsonb_each((jsonb_each(playerstats)).value)).key AS lineup_id,
		(jsonb_each((jsonb_each((jsonb_each(playerstats)).value)).value)).key AS opponent_lineup_id,
		(jsonb_each((jsonb_each((jsonb_each((jsonb_each(playerstats)).value)).value)).value)).key AS player_id,
		(jsonb_each((jsonb_each((jsonb_each((jsonb_each(playerstats)).value)).value)).value)).value AS stats
	FROM possession_details
	WHERE starttype = 'OffLiveBallTurnover'
	AND defenseteamid = 1610612762 -- Utah's team id: https://stats.nba.com/team/1610612762/
	AND previouspossessionendturnoverplayerid = 201937 -- Ricky Rubio's player id: https://stats.nba.com/player/201937/
	AND gameid like '002%' -- regular season only, regular season game ids start 002, playoffs start 004
) a
WHERE a.team_id != '1610612762' -- exclude Utah's stats so plus minus doesn't sum to 0
```

Get who Myles Turner blocked, in descending order of how many of their shots he blocked:
```
SELECT entity_data.name, count(*) as shots_blocked
FROM possession_details,
	jsonb_to_recordset(possession_details.shotdata) AS shot_data("X" FLOAT, "Y" FLOAT, "PlayerId" INT, "ShotValue" INT, "Made" BOOLEAN, "Time" FLOAT, "ScoreMargin" INT, "ShotQuality" FLOAT, "Blocked" BOOLEAN, "BlockPlayerId" INT, "Assisted" BOOLEAN, "AssistPlayerId" INT, "Putback" BOOLEAN, "ShotType" VARCHAR, "OrebShotPlayerId" INT, "OrebReboundPlayerId" INT, "OrebShotType" VARCHAR, "SecondsSinceOReb" FLOAT, "LineupId" VARCHAR, "OpponentLineupId" VARCHAR)
INNER JOIN entity_data
ON shot_data."PlayerId"::text = entity_data.entityid
WHERE shot_data."BlockPlayerId" = 1626167
GROUP BY entity_data.name
ORDER BY shots_blocked DESC
```

Get all shots attempted by Pascal Siakam within 4 seconds of a Raptors defensive rebound:
```
SELECT gameid,
	period,
	possessionnumber,
	offenseteamid,
	defenseteamid,
	starttime,
	endtime,
	starttype,
	previouspossessionENDshooterplayerid,
	previouspossessionENDreboundplayerid,
	previouspossessionENDturnoverplayerid,
	previouspossessionENDstealplayerid,
	shot_data.*
FROM possession_details,
	jsonb_to_recordset(possession_details.shotdata) AS shot_data("X" FLOAT, "Y" FLOAT, "PlayerId" INT, "ShotValue" INT, "Made" BOOLEAN, "Time" FLOAT, "ScoreMargin" INT, "ShotQuality" FLOAT, "Blocked" BOOLEAN, "BlockPlayerId" INT, "Assisted" BOOLEAN, "AssistPlayerId" INT, "Putback" BOOLEAN, "ShotType" VARCHAR, "OrebShotPlayerId" INT, "OrebReboundPlayerId" INT, "OrebShotType" VARCHAR, "SecondsSinceOReb" FLOAT, "LineupId" VARCHAR, "OpponentLineupId" VARCHAR)
WHERE shot_data."PlayerId" = 1627783
AND starttime - shot_data."Time" <= 4
AND previouspossessionendreboundplayerid != 0
```

Get player minutes played against all 5 opponent starters, in descending order:
```
SELECT starter_state_season_totals.entityid, entity_data.name, SUM(COALESCE((data->>'SecondsPlayedOff')::FLOAT, 0)) / 60 + SUM(COALESCE((data->>'SecondsPlayedDef')::FLOAT, 0)) / 60 AS minutes
FROM starter_state_season_totals
INNER JOIN entity_data
ON starter_state_season_totals.entityid = entity_data.entityid
AND starter_state_season_totals.teamid = entity_data.teamid -- need to join on team to avoid duplicate rows for players who changed teams during the season - they have multiple rows in entity_data table
WHERE starter_state_season_totals.entitytype = 'Player'
AND starterstate LIKE '%v5'
GROUP BY starter_state_season_totals.entityid, entity_data.name
ORDER BY minutes DESC
```

If you use this and find any errors let me know. I have manually fixed some issues with the play-by-play, but I would guess there may be some issues I haven't discovered yet.
