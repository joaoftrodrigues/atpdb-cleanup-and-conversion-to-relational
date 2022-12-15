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

## Queries
**1 - começei, nao correu tudo no meu** 



`SELECT tournamentlocation.Country as "Country", count(player.PlayerID) as "Nr. of Players", count(tournament.TournamentName) as "Nr of Tournaments", count(gameround.RoundName) as "Nr. of Rounds" 
FROM player, gameround, tournament, tournamentlocation
where player.ID=gameround.ID and tournament.ID=tournamentlocation.ID
GROUP BY tournamentlocation.Country
LIMIT 10`






**2 - resultados semelhantes ao mestrado , verificar na nossa bd**

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

**3 - resultados iguaisisnhos mestrado**

`CREATE view total_wl_ AS
SELECT player.Name,COUNT(*) as total 
from player, gameround, tournament
where player.ID=gameround.ID and gameround.ID=tournament.ID and (gameround.Result="W" or gameround.Result="L") 
and tournament.TournamentName IN("Australian Open","Roland Garros","Wimbledon", "US Open")
group by player.Name

SELECT player.Name as "Player Name", count(*) / total_wl_.total as "Winning games(%)" 
FROM player, tournament, gameround, handskill, total_wl_ 
where total_wl_.Name = player.Name and player.ID = tournament.ID and tournament.ID=gameround.ID 
and Player.ID=handskill.ID and handskill.PrimaryHand="Left-handed" and gameround.Result="W" 
and tournament.TournamentName IN("Australian Open","Roland Garros","Wimbledon", "US Open") 
GROUP BY player.Name 
ORDER BY 2 DESC 
LIMIT 10`

**4 - resultados iguaisinhos mestrado**

`SELECT player.Name,COUNT(gameround.Result) from player,gameround,tournament where player.ID=gameround.ID and gameround.ID=tournament.ID and gameround.Result="W" and tournament.Ground="Hard" GROUP BY player.Name ORDER BY 2 DESC limit 5`

