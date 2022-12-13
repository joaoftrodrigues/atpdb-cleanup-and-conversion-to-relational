# ATP database - Cleanup and conversion to relational

Parting from a non relational database containing information from tennis players, origined from atp website (https://www.atptour.com/en), it's intended to clean it through mongoDB.

Further, after cleaning process, the database structure will be adapted to a relational database (SQL). 


# Data

- _atpplayers.json_
- _countries.csv_
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
