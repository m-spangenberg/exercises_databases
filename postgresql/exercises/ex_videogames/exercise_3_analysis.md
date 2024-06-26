## Excercise: Video Games

* Download a dataset
* Ingest the dataset
* Analyze the dataset

**Remember: Kaggle.com is an excellent resource for datasets.**

I'll start by setting up the PSQL database in my docker container.

```bash
# start docker container
sudo docker start post-dev

# get shell inside container
sudo docker exec -it post-dev bash

# create database inside container
createdb videogames -U postgres

# get postgres shell
psql videogames -U postgres

# create table
create table games(
    name text,
    platform text, 
    year int, 
    genre text, 
    publisher text, 
    NAsales numeric, 
    EUsales numeric, 
    JPsales numeric, 
    othersales numeric, 
    globalsales numeric,
    criticscore numeric,
    criticcount numeric,
    userscore numeric,
    usercount numeric,
    developer text,
    rating text
);

# Copy data to docker container from host
sudo docker cp videogames.csv dev-postgres:/var/lib/postgresql/data/videogames.csv

# Do some housekeeping on the data in-situ by removing any repetitions of 'N/A' from the file as we already treat empty delimiters as null.
cp /var/lib/postgresql/data/videogames.csv /var/lib/postgresql/data/videogames2.csv
sed -i -e 's/N\/A//g' /var/lib/postgresql/data/videogames.csv

# I'll then import the CSV with the COPY command.
COPY games FROM '/var/lib/postgresql/data/videogames.csv' DELIMITER ',' NULL AS '' CSV HEADER;
```

Now we can attempt some analysis of the dataset with some of our newfound query language skills.

```bash
# By issuing `select * from games limit 4;`, we should get the following output:

       name        | platform | year |  genre   | publisher | nasales | eusales | jpsales | othersales | globalsales | criticscore | criticcount | userscore | usercount | developer | rating 
-------------------+----------+------+----------+-----------+---------+---------+---------+------------+-------------+-------------+-------------+-----------+-----------+-----------+--------
 Wii Sports        | Wii      | 2006 | Sports   | Nintendo  |   41.36 |   28.96 |    3.77 |       8.45 |       82.53 |          76 |          51 |         8 |       322 | Nintendo  | E
 Super Mario Bros. | NES      | 1985 | Platform | Nintendo  |   29.08 |    3.58 |    6.81 |       0.77 |       40.24 |             |             |           |           |           | 
 Mario Kart Wii    | Wii      | 2008 | Racing   | Nintendo  |   15.68 |   12.76 |    3.79 |       3.29 |       35.52 |          82 |          73 |       8.3 |       709 | Nintendo  | E
 Wii Sports Resort | Wii      | 2009 | Sports   | Nintendo  |   15.61 |   10.93 |    3.28 |       2.95 |       32.77 |          80 |          73 |         8 |       192 | Nintendo  | E
(4 rows)

# Let's see what the 3 most popular games in the world are of all time as measured by their global sales figures in millions.
select name, globalsales from games where games.globalsales > 20 order by games.globalsales desc limit 3;

       name        | globalsales 
-------------------+-------------
 Wii Sports        |       82.53
 Super Mario Bros. |       40.24
 Mario Kart Wii    |       35.52
(3 rows)

# Let's try to group total sales for all video games by their genre.
select sum(globalsales), genre from games group by genre;

   sum   |    genre     
---------+--------------
  237.69 | Adventure
  243.02 | Puzzle
  174.50 | Strategy
  934.40 | Role-Playing
  390.42 | Simulation
  803.18 | Misc
    2.42 | 
  447.48 | Fighting
  728.90 | Racing
 1332.00 | Sports
 1745.27 | Action
 1052.94 | Shooter
  828.08 | Platform
(13 rows)

# Let's try to see what the top 5 most popular video games platforms of all time are by number of games sold.
select sum(globalsales) as totalsales, platform from games group by platform order by totalsales desc limit 5;

# Note: I've aliased sum(globalsales) to totalsales

 totalsales | platform 
------------+----------
    1255.64 | PS2
     971.63 | X360
     939.43 | PS3
     908.13 | Wii
     807.10 | DS
(5 rows)
```