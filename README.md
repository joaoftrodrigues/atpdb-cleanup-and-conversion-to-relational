# ATP database - Cleanup and conversion to relational

Parting from a non relational database containing information from tennis players, origined from atp website (https://www.atptour.com/en), it's intended to clean it through mongoDB.

Further, after cleaning process, the database structure will be adapted to a relational database (SQL). 


# Data

- _atpplayers.json_
- _countries_code.csv_
- _wikipedia-iso-country-codes.csv_


_attplayers_ contains the data where the problem parts, holding information about the players and the matches on tournaments they played.

_countries_ was orginally provided for the work, having the countries' names plus its 2 letters code. This was found of lacking countries and was adapted to the project's needs.

_wikipedia-iso-country-codes_ is used to match 3 letters codes, to change the countries' names.


## Imports

To run the queries, it's needed to import all the 3 files, which can be done with follow commands:

**ATP Players**
`mongoimport --db atp --collection atpplayers --drop --file <FILEPATH>`

**Original countries with 2 letters code**
`mongoimport --db atp --collection countries --drop --file <FILEPATH> --type=csv --headerline`

**Countries with 3 letters code**
`mongoimport --db atp --collection countryCodes3L --drop --file <FILEPATH> --type=csv --headerline`


## Exports

`mongoexport --db atp --collection atpplayers --type=csv --fields _ID,PlayerID,PlayerName,Height,LinkPlayer --out c:\table_Player.csv`

`mongoexport --db atp --collection atpplayers --type=csv --fields _ID,Primary_Hand,Backhand --out c:\table_HandSkill.csv`

`mongoexport --db atp --collection atpplayers --type=csv --fields _ID,Tournament,Date_Start,Date_End,Ground --out c:\table_Tournament.csv`

`mongoexport --db atp --collection atpplayers --type=csv --fields _ID,Tournament,GameRound,GameRank,Oponent,WL --out c:\table_GameRound.csv`

`mongoexport --db atp --collection atpplayers --type=csv --fields _ID,GameRound,Score1,Score2,Score3,Score4,Score5 --out c:\table_Score.csv`

`mongoexport --db atp --collection atpplayers --type=csv --fields _ID,BornCity,BornState,BornCountry --out c:\table_Born.csv`

`mongoexport --db atp --collection atpplayers --type=csv --fields _ID,Tournament,City,State,Country --out c:\table_Location.csv`



## Queries
**1 - Number of players, tournaments and rounds per Country** 



`SELECT tournamentlocation.Country as "Country", count(player.PlayerID) as "Nr. of Players", count(tournament.TournamentName) as "Nr of Tournaments", count(gameround.RoundName) as "Nr. of Rounds" 
FROM player, gameround, tournament, tournamentlocation
where player.ID=gameround.ID and tournament.ID=tournamentlocation.ID
GROUP BY tournamentlocation.Country
LIMIT 10`






**2 - List of top 10 players on winnin rate**

`CREATE view totalwl AS
SELECT player.Name,COUNT(*) as total 
from player, gameround, tournament
where player.ID=gameround.ID and gameround.ID=tournament.ID and (gameround.Result="W" or gameround.Result="L") 
group by player.Name`


`SELECT player.Name as "Player Name", count(*) / totalwl.total  as "Winning games(%)"
FROM player, gameround, totalwl
where totalwl.Name = player.Name and player.ID=gameround.ID 
and gameround.Result="W" 
GROUP BY player.Name 
ORDER BY 2 DESC 
LIMIT 10`

![image](https://user-images.githubusercontent.com/119869654/207866098-8420ffb7-e3cc-485c-8db2-172b8f260cf6.png)

**3 - Top 10 of left-handed players in Grand Slam games, showing win rate in that tournament**

`CREATE view total_wl_ AS
SELECT player.Name,COUNT(*) as total 
from player, gameround, tournament
where player.ID=gameround.ID and gameround.ID=tournament.ID and (gameround.Result="W" or gameround.Result="L") 
and tournament.TournamentName IN("Australian Open","Roland Garros","Wimbledon", "US Open")
group by player.Name`

`SELECT player.Name as "Player Name", count(*) / total_wl_.total as "Winning games(%)" 
FROM player, tournament, gameround, handskill, total_wl_ 
where total_wl_.Name = player.Name and player.ID = tournament.ID and tournament.ID=gameround.ID 
and Player.ID=handskill.ID and handskill.PrimaryHand="Left-handed" and gameround.Result="W" 
and tournament.TournamentName IN("Australian Open","Roland Garros","Wimbledon", "US Open") 
GROUP BY player.Name 
ORDER BY 2 DESC 
LIMIT 10`

![image](https://user-images.githubusercontent.com/119869654/207865376-ca6706bb-ba47-4a09-80b0-08aed10643cb.png)


**4 - Top 5 players in hard ground, showing number of wins**

`SELECT player.Name,COUNT(gameround.Result) from player,gameround,tournament where player.ID=gameround.ID and gameround.ID=tournament.ID and gameround.Result="W" and tournament.Ground="Hard" GROUP BY player.Name ORDER BY 2 DESC limit 5`

![image](https://user-images.githubusercontent.com/119869654/207866412-3e7ea870-7050-4674-afb4-68a18923dddc.png)



