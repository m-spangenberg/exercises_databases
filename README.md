# Databases

Below are my study notes on databases. I've made these notes to further my understanding of the inner-workings of databases and database management systems.

## Quick Setup: Testing Environment

First I'm going to set up an environment where I can work with a popular RDBMS. I would like to have both command-line and GUI access, so for my purposes I have chosen PostgreSQL as the RDBMS and pgAdmin as the administration panel. If you are new to databases, get your feet wet with SQLite instead of Postgres.

### PostgreSQL in Docker

```bash
# make a working directory
mkdir postgresql
 
# make a data directory
cd postgresql
mkdir psql-data
 
# get the postgres image amd the admin panel
sudo docker pull postgres
sudo docker pull dpage/pgadmin4
 
# create a docker container for postgresql and the admin panel
sudo docker run \
    --name post-dev \
    -e 'POSTGRES_PASSWORD=YOURPASSWORD' \
    -v ${HOME}/path/to/folder/psql-data:/var/lib/postgresql/data \
    -p 5551:5551 \
    -d postgres
    
sudo docker run \
    --name post-dev-admin \ 
    -p 80:80 \
    -e 'PGADMIN_DEFAULT_EMAIL=YOUREMAILADDRESS' \
    -e 'PGADMIN_DEFAULT_PASSWORD=YOURPASSWORD' \
    -d dpage/pgadmin4
 
# you can enter the postgresql container
sudo docker exec -it post-dev bash
# and get a shell for postgres
psql -h localhost -U postgres
# See: https://www.postgresql.org/docs/current/tutorial.html 

# or connect to it with the admin panel, just grab the postgresql containers IP first.
sudo docker inspect post-dev {{ .NetworkSettings.Networks.$network.IPAddress }}
# It should be something like: 172.17.0.2
# go to 127.0.0.1:80 to connect to the admin panel and add the server
```

## Database Management Systems (DBMS)

On the surface it seems simple enough, an application queries a database which then returns some data. But between the data and the interface of the DBMS lies a stack of processors and managers that ensure queries and data are handled in specific ways.

## Structured Query Language (SQL)

To interface with the DBMS we must use SQL to issue queries. These queries, or commands, can be distinguished by the types of tasks they perform within the DBMS:

* Data Control Language (**DCL**) -> exists to assign access rights to data
* Transaction Control Language (**TCL**) -> groups  SQL commands as transactions
* Data Manipulation Language (**DML**) -> insert, manipulate, and retrieve data
* Data Definition Language (**DDL**) -> defines what is admissable data within set constraints

```bash
# fully specified
insert into <table> values (<val-list>);
# partially specified
insert into <table> (<col-list>) values (<val-list>);
```

### Data Definition Language (DDL)

With DDL we define a database schema that describes the content we can have in our database. A schema in a relational database consists as a collection of two-dimensional arrays of information labeled with columns and each column is associated with a datatype. The data that is accepted by the scheme is defined by constraints which either link single relations or multiple relations.

| id 	| name   	| surname  	| age 	| license-driving     	|
|----	|--------	|----------	|-----	|---------------------	|
| 1  	| mark   	| lasperre 	| 42  	| 1234-4321-1234-1234 	|
| 2  	| robert 	| gershwin 	| 69  	| 4321-1234-1234-1234 	|
| 3  	| adam   	| nakumi   	| 34  	| 1234-1234-1234-4321 	|

### Schema Definition in SQL

In SQL specifically, we don't talk about relations, we talk about tables. There are fine differences between relations in the Relational Data Model and tables in SQL but superficially they can be used to mean the same thing.

```sql
CREATE TABLE Students(Sid int, Sname text, Gpa real);
CREATE TABLE Enrollment(Sid int, Cid int);
CREATE TABLE Courses(Cid int, Cname text);
```

```bash
CREATE TABLE <table> (<table-def>)
# <table> is the table name
# <table-def> is comme-separated column definitions
# <col-name><col-type> is the form for definitions
```

### Integrity Constraints

Above we have created three tables. Each column has a name, a key if you will, and a datatype associated with that column. The datatype is what is called an `Integrity Constraint`, as the DBMS enforces what types of data are permissable for each column ensuring the integrity of the model.

### Primary Key Constraints

A primary key constraint refers to a single table in the schema which contains a column of keys of which no two rows are allowed to have the same value. The primary key is in effect the ID for that given row in relation to the rest of the schema.

It is possible to alter the existing tables we created by issuing the following command:

```bash
ALTER TABLE <table> ADD Primary Key (<key-cols>);
# <table> is the table name
# <key-cols> is the list of column names 
```

```sql
ALTER TABLE Students ADD PRIMARY KEY(Sid);
ALTER TABLE Enrollment ADD PRIMARY KEY(Sid, Cid);
ALTER TABLE Courses ADD PRIMARY KEY(Cid);
```

Every table can have only one `Primary Key`, but you can have multiple columns in your Primary Key, which we call a `Composite Primary Key`. This Composite Primary Key exists in what is commonly called a Link, Join, or `Junction Table`. This Junction Table contains only `Foreign Keys` in order to facilitate a many-to-many relationship.

In the case of our example above, the Enrollment table has a Student ID and a Course ID, and neither are unique on their own, but combined they represent a unique combination that becomes our Composite Primary Key. So the Student ID in the Students table links to the Enrollment table which then is paired with a Course ID which links the Enrollment table to the Courses table.

| Students Table       	| Enrollment Table 	| Courses Table |
|--------------------	|----------------	|--------------	|
| [Sid , Sname, Gpa] 	| [Sid, Cid]     	| [Cid, Cname] 	|
| Pkey: Sid          	| Pkey: Sid, Cid 	| Pkey: Cid    	|

### Foreign Key Constraint

A `Foreign Key` links two tables by identifying a group of columns in one table and mapping those columns to the primary key of another table, those Foreign Key column values must appear as Primary Keys in the related table and is how we map each row from the first table to the second table. 

```bash
ALTER TABLE <table-1> ADD Foreign Key (<fkey-cols>)
REFERENCES <table-2> (<pkey-cols>);
# <table-1> is the table that contains foreign key columns <fkey-cols>
# <table-2> is the table that contains primary key columns <pkey-cols>
```

Here we create a link from the Student ID in the Enrollment table to the Student ID in the Students table and the Course ID in the Enrollment table to the Course ID in the Courses table. In both instances we tell the current table we are referencing a Primary Key from another table. In the land of tables, if tables were countries, this `Primary Key` originates from a foreign table, so it is a `Foreign Key`.

```sql
ALTER TABLE Enrollment ADD FOREIGN KEY(Sid) REFERENCES Students(Sid);
ALTER TABLE Enrollment ADD FOREIGN KEY(Cid) REFERENCES Courses(Cid);
```

## Excercise: The Bookstore

Let's create a database for a bookstore called Pageturners. The owner of the bookstore is used to keeping track of the books in his store with a spreadsheet. It's our job to migrate his spreadsheet to a database with an appropriate schema. Their requirements are as follows:

* they have a collection of `books`, each with a `title` and a `unique id`
* and a list of `writers`, each writer has a `unique name`
* naturally `writers` are `authors` of various `books`

Fortunately for us, Pageturners has sent us their spreadsheet to get started.

| Id 	| Title                          	| Author          	                 |
|------	|--------------------------------	|--------------------------------    |
| 1    	| Life Of Pi                     	| Yann Martel     	                 |
| 2    	| Whitethorn                     	| Bryce Courtenay 	                 |
| 3    	| Meditations: A New Translation 	| Marcus Aurelius 	                 |
| 4    	| The 48 Laws of Power           	| Robert Greene   	                 |
| 5    	| The Art Of War                 	| Sun Tzu         	                 |
| 6    	| Database Guru                 	| Jimmy Droptables, Jane Droptables  |
| ...  	| ...                             	| ...            	                 |

The schema for our `PageturnersDB` will look as follows: A Books table containing a book id, which will act as the table's Primary Key, and the title of the book. Writers table, which has the writer's name and a unique id for each writer, which will act as this tables Primary Key. And finally, a Authors table, which will contain two foreign keys, one being the book id from the Books table, and the writer id from the Writers table. With this schema, we are able to maintain a many-to-many relationship, meaning books can have multiple writers and writers can write multiple books. 

**Remember: Foreign Keys do not need to be unique. Only Primary Keys must be unique.**

| Books Table  	      | Authors Table  	         | Writers Table 	                |
|--------------	      |----------------	         |---------------	                |
| [bookid, booktitle] | [bookid, writers]        | [writerid, first_name, last_name]|
| Pkey: bookid    	  | Fkey: bookid, writerid 	 | Pkey: writerid     	            |

As I am using Docker, these are my steps for my first exercise.

```bash
# Create the database
createdb pageturnersdb -h localhost -U postgres

# if you made a mistake you can drop (delete) the db with
dropdb pageturnersdb -h localhost -U postgres

# Enter the database
psql pageturnersdb -U postgres

# create the books table with a bid as the primary key
# the id integrity constraint is integer
# and the title is a string no longer than 240 characters.
create table books(bookid bigserial primary key, booktitle varchar(240));

# create the writers table with wid as the primary key
# and wname as a string no longer than 240 characters.
create table writers(writerid bigserial primary key, first_name varchar(240), last_name varchar(240));

# finally we create join or junction table that references the other two tables
create table authors (bookid int references books(bookid), writerid int references writers(writerid));
```

### Data Manipulation Language (DML)

Commands which are responsible for allowing CRUD operations to take place in the database.

* Insert
* Delete
* Update
* Analyze

#### Inserting Data

We can insert `fully specified` or `partially specified` data into rows. Fully specified means we specify the value for each column in the table, hence we fully specify what we want to be inserted. Partially specified, means we only target a specific column or subset of columns, and the DBMS fill the remaining columns automatically with NULL unless there is an integrity constraint which does not allow for NULL.

**Remember: It is possible ot load data into tables via files**

```bash
# finally we populate the tables with data
insert into books (booktitle) VALUES ('Life of Pi');
insert into books (booktitle) VALUES ('Whitethorn');
insert into books (booktitle) VALUES ('Meditations: A New Translation');
insert into books (booktitle) VALUES ('The 48 Laws of Power');
insert into books (booktitle) VALUES ('The Art Of War');
insert into books (booktitle) VALUES ('Database Guru');

# performing `select * from books;` will show you the inserted fields
 bookid |           booktitle            
--------+--------------------------------
      1 | Life of Pi
      2 | Whitethorn
      3 | Meditations: A New Translation
      4 | The 48 Laws of Power
      5 | The Art Of War
      6 | Database Guru
(6 rows)

# now let's insert some writers into the writers table
insert into writers (first_name, last_name) VALUES ('Yann', 'Martel');
insert into writers (first_name, last_name) VALUES ('Bryce', 'Courtenay');
insert into writers (first_name, last_name) VALUES ('Marcus', 'Aurelius');
insert into writers (first_name, last_name) VALUES ('Robert', 'Greene');
insert into writers (first_name, last_name) VALUES ('Sun', 'Tzu');

# `select * from writers;` should show the following:
 writerid | first_name | last_name 
----------+------------+-----------
        1 | Bryce      | Courtenay
        2 | Marcus     | Aurelius
        3 | Robert     | Greene
        4 | Sun        | Tzu
        5 | Yann       | Martel
(5 rows)

# we forgot to add two writers
insert into writers (first_name, last_name) VALUES ('Jimmy', 'Droptables');
insert into writers (first_name, last_name) VALUES ('Jane', 'Droptables');

# now let's connect books to writers
insert into authors (bookid, writerid) VALUES (2, 1);
insert into authors (bookid, writerid) VALUES (3, 2);
insert into authors (bookid, writerid) VALUES (4, 3);
insert into authors (bookid, writerid) VALUES (5, 4);
insert into authors (bookid, writerid) VALUES (6, 6);
insert into authors (bookid, writerid) VALUES (6, 7);
insert into authors (bookid, writerid) VALUES (1, 5);

# finally, if we try `select * from authors;`:
 bookid | writerid 
--------+----------
      2 |        1
      3 |        2
      4 |        3
      5 |        4
      6 |        6
      6 |        7
      1 |        5
(7 rows)
```
#### Deleting Data

Deleting data from tables can be achieved with `DELETE FROM <table> WHERE <condition>`. For instance, Pageturners has been told Jimmy Droptables wasn't an author on Database Guru.

```bash
# We can delete him from the join table called authors.
delete from authors where writerid = 6;
delete from writers where writerid = 6;
```

#### Updating Data

Updating data, much like delete, has a simple syntax: `UPDATE <table> SET <col> = <val> WHERE <condition>`. Pageturners has been told Jane Droptables is actually Jane Hadersville!

```bash
# Let's update her name before someone notices.
update writers set last_name = 'Hadersville' WHERE writerid = 7;
```
#### Analyze Data

An SQL query is a new relationship that has to be generated by the DBMS. We describe our desired output as a query and the system has to figure out how to display that output. Most SQL queries consist of three parts. `SELECT` which describes column relation, `FROM` which describes source relations, and `WHERE` which defines conditions which need to be satisfied before we can receive a result. The majority of syntax needed for a SQL query can be reduced to the following: `select <col> from <table-1> join <table-2> on <join-condition> where <where-condition>`

```bash
# Let's see if we can perform a query on our pageturnersdb to return all work by Marcus Aurelius
SELECT books.booktitle
FROM books
JOIN authors ON (books.bookid = authors.bookid)
JOIN writers ON (authors.writerid = writers.writerid)
WHERE writers.writerid = 2;
# You'll notice because the data is related, it's possible to retrieve 
# the queries intended answer in more than one way.
SELECT books.booktitle
FROM books
JOIN authors ON (books.bookid = authors.bookid)
JOIN writers ON (authors.writerid = writers.writerid)
WHERE (writers.first_name) = 'Marcus';
```

##### Simplification by Alias

To save ourselves some time while writing queries, it is possible to alias table names in the following manner.

```bash
SELECT b.booktitle
FROM books b
JOIN authors a ON (b.bookid = a.bookid)
JOIN writers w ON (a.writerid = w.writerid)
WHERE (w.first_name) = 'Marcus';
```

##### Predicate Statements

A predicate is not a conditional. A conditional statement allows for branching logic whereas a predicate is a question that results in either boolean true or boolean false. SQL support many types of predicates that return a boolean value when they resolve.

* Inequality (`>`, `>=`): `books.bookrating > 80`
* Not Equal (`<>`): `books.booktitle <> 'Life of Pi'`
* Exists in list (`IN`): `booktitle IN ('list_item', 'list_item_2')`
* Regular Expression: (`LIKE`): `WHERE booktitle LIKE 'L__%'`

##### Composite Predicates

* Logical conjunction: `AND`
* Logical disjunction: `OR`
* Negation: `NOT`

##### Select Clauses

* Wildcard selection of multiple columns: `*`
* Select all columns from a table: `<table>.*`
* Expressions in the select clause: `SELECT 3 * (<col1> + <col2>)`
* Alias assignment for output columns: `SELECT bookname as booktitle`

##### Join Syntax

* Standard join: `<table1> JOIN <table2> ON (<table1>.<col> = <table2>.<col>)`
* Abreviated join: `<table1> JOIN <table2> USING (<col>)`
* Natural join: automatically join columns with the same name: `<table1> NATURAL JOIN <table2>`
* Terse form of a Standard join: `FROM <table1>, <table2> WHERE <join-condition>`

##### Distinct

* Eliminate duplicates using `SELECT DISTINCT`

##### Aggregation

* Numerical expressions: `SUM, AVG, MIN, MAX`
* Count rows in result: `COUNT(*)` 
* Count result row in column: `COUNT(<col>)`
* Count distinct values in column: `COUNT(DISTINCT <col>)`
* Aggregating multiple columns: `GROUP BY <col-list>`

example: let's count the the remaining rows with `COUNT(*)`
```sql
SELECT COUNT(*)
FROM books
    JOIN Authors ON (books.bookid = authors.bookid)
    JOIN Writers ON (authors.writerid = writers.writerid)
WHERE books.booktitle = 'Life of Pi';
```

##### Group Predicates

* `WHERE` applies to a single row
* `HAVING` specifies grouped data

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

**Remember: Kaggle.com is an excellent resource for datasets.**

## Excercise: Video Games

https://youtu.be/4cWkVbC2bNE?t=6946


## NO-SQL
## NEW-SQL
## Graph Data
## Spacial Data
## Data Streams
