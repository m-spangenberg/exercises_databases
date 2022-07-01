## Excercise: Music Collection

* Download a public dataset of your choice
* Load it into the database
* Try to find interesting relations within the data

Manually import a csv file to a table
data for this exercise comes from: https://corgis-edu.github.io/corgis/csv/music/

Create and access new database
```bash
createdb musiccollection -h localhost -U postgres
psql musiccollection -h localhost -U postgres
```

create master table for csv ingest
```sql
create table music (
    artist_familiarity DECIMAL,
    artist_hotttnesss DECIMAL,
    artist_id VARCHAR(256),
    artist_latitude DECIMAL,
    artist_location DECIMAL,
    artist_longitude DECIMAL,
    artist_name VARCHAR(256),
    artist_similar DECIMAL,
    artist_terms VARCHAR(256),
    artist_terms_freq DECIMAL,
    release_id INTEGER,
    release_name INTEGER,
    song_artist_mbtags DECIMAL,
    song_artist_mbtags_count DECIMAL,
    song_bars_confidence DECIMAL,
    song_bars_start DECIMAL,
    song_beats_confidence DECIMAL,
    song_beats_start DECIMAL,
    song_duration DECIMAL,
    song_end_of_fade_in DECIMAL,
    song_hotttnesss DECIMAL,
    song_id VARCHAR(256),
    song_key DECIMAL,
    song_key_confidence DECIMAL,
    song_loudness DECIMAL,
    song_mode INTEGER,
    song_mode_confidence DECIMAL,
    song_start_of_fade_out DECIMAL,
    song_tatums_confidence DECIMAL,
    song_tatums_start DECIMAL,
    song_tempo DECIMAL,
    song_time_signature DECIMAL,
    song_time_signature_confidence DECIMAL,
    song_title INTEGER,
    song_year INTEGER
);
```

Copy CSV file into docker container
```bash
sudo docker cp music.csv post-dev:/var/lib/postgresql/data/music.csv
```

Import csv into music table
```sql
copy music from '/var/lib/postgresql/data/music.csv' delimiter ',' csv header;
```

Select artists based on a genre: I choose country music
```sql
select artist_name from music where artist_terms like '%country%';
```
terminal output:
```bash
                   artist_name                   
-------------------------------------------------
 Blue Rodeo
 Jimmy Wakely
 Billie Jo Spears
 Glen Campbell
...
```

Display the loudest songs in the database in descending order from each artist based on the loudness of the song and a partial genre the metal genre, each artist must be distinct. It's worth noting loudness here is represented in Decibels so negative numbers are louder.

```sql
select distinct artist_name, song_loudness from music where artist_terms like '%metal%' AND song_loudness < '-10.00' order by song_loudness asc limit 4;
```
terminal output:
```bash
 artist_name | song_loudness 
-------------+---------------
 Hinge       |       -32.987
 Don Davis   |        -27.24
 Tony Martin |       -26.928
 Rainbow     |        -23.44
(4 rows)
```

Select artists based on partial name matching inside known genre
```sql
select artist_name from music where artist_terms = 'gangster rap' AND artist_name like '%Notorious%';
```

terminal output:
```bash
     artist_name      
----------------------
 The Notorious B.I.G.
(1 row)
```