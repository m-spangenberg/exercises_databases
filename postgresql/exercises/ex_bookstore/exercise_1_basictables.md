## Exercise: The Bookstore

* Create a simple bookstore database
* Create a functional schema that deals with books and writers

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