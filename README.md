# Databases

Below are my study notes on databases. I've made these notes to further my understanding of the inner-workings of databases and database management systems.

## Introduction To Database Systems

### Database Management Systems (DBMS)

On the surface it seems simple enough, an application queries a database which then returns some data. But between the data and the interface of the DBMS lies a stack of processors and managers that ensure queries and data are handled in specific ways.

### Structured Query Language (SQL)

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

### Introduction To SQL

#### Data Manipulation Language (DML)

Commands which are responsible for allowing CRUD operations to take place in the database.

* Insert
* Delete
* Update
* Analyze

#### Inserting Data

We can insert `fully specified` or `partially specified` data into rows. Fully specified means we specify the value for each column in the table, hence we fully specify what we want to be inserted. Partially specified, means we only target a specific column or subset of columns, and the DBMS fill the remaining columns automatically with NULL unless there is an integrity constraint which does not allow for NULL.

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
### Simple SQL Data Analysis

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

#### Simplification by Alias

To save ourselves some time while writing queries, it is possible to alias table names in the following manner.

```bash
SELECT b.booktitle
FROM books b
JOIN authors a ON (b.bookid = a.bookid)
JOIN writers w ON (a.writerid = w.writerid)
WHERE (w.first_name) = 'Marcus';
```

#### Predicate Statements

A predicate is not a conditional. A conditional statement allows for branching logic whereas a predicate is a question that results in either boolean true or boolean false. SQL support many types of predicates that return a boolean value when they resolve.

* Inequality (`>`, `>=`): `books.bookrating > 80`
* Not Equal (`<>`): `books.booktitle <> 'Life of Pi'`
* Exists in list (`IN`): `booktitle IN ('list_item', 'list_item_2')`
* Regular Expression: (`LIKE`): `WHERE booktitle LIKE 'L__%'`

#### Composite Predicates

* Logical conjunction: `AND`
* Logical disjunction: `OR`
* Negation: `NOT`

#### Select Clauses

* Wildcard selection of multiple columns: `*`
* Select all columns from a table: `<table>.*`
* Expressions in the select clause: `SELECT 3 * (<col1> + <col2>)`
* Alias assignment for output columns: `SELECT bookname as booktitle`

#### Join Syntax

* Standard join: `<table1> JOIN <table2> ON (<table1>.<col> = <table2>.<col>)`
* Abreviated join: `<table1> JOIN <table2> USING (<col>)`
* Natural join: automatically join columns with the same name: `<table1> NATURAL JOIN <table2>`
* Terse form of a Standard join: `FROM <table1>, <table2> WHERE <join-condition>`

#### Distinct

* Eliminate duplicates using `SELECT DISTINCT`

#### Aggregation

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

#### Group Predicates

* `WHERE` applies to a single row
* `HAVING` specifies grouped data

#### Ordering Syntax

* We can use `ORDER BY` to arrange an `<order-item-list>`
* The syntax is: `<order-item>:<col> <direction>`
* The direction can be `ASC` ascending or `DESC` desending.
* Is applied __after__ grouping when using group-by queries.

### More Advanced SQL Features

#### Unknown Values

In SQL, an unknown value is referred to as `NULL`, **NULL is not ZERO**, it is the _absence of a value_, or a value which holds no type. Because SQL uses `Ternary` (three-valued) logic, the outcome of a logical predicate can be either true, false or unknown (NULL). We check the outcome of an expression with `= True`, `= False`, and `IS NULL`.

Ternary Logic Examples with NULL.

* `SELECT 3 = NULL` will result in `NULL`, not `False`
* `SELECT NULL = NULL` will result in `NULL`, not `True`
* `SELECT NULL IS NULL` will result in `True`
* `SELECT NULL IS NOT NULL` will result in `False`
* `SELECT TRUE OR NULL` will result in `True`
* `SELECT TRUE AND NULL` will result in `NULL`

#### Joins with Unknowns

A standard join keeps only `matching` row pairs. It eliminates rows without matching rows in other tables. Sometimes we want to keep rows regardless, to achieve this we can perform an `OUTER JOIN`, this type of join will fill missing field data with `NULL` values in order to maintain partial rows.

* `<table1> LEFT OUTER JOIN <table2> ON ...` - Keeps rows in left table
* `<table1> RIGHT OUTER JOIN <table2> ON ...` - Keeps rows in right table
* `<table1> FULL OUTER JOIN <table2> ON ...` - Keeps rows in both tables

example: We want to see the number of enrollments per student
```sql
SELECT studentname, COUNT(courseid)
FROM studenttable LEFT OUTER JOIN enrollmenttable
ON (studenttable.studentid = enrollmenttable.studentid)
```

In the above example, if a student does not have matching enrollment information the join operation will assing null values in the students courseid fields, and then those null values will not be counted when performing the query.

#### Set Operations

A powerful feature of SQL is the ability to combine queries, in effect creating composite queries. The easiest way to do this is to use **Set Operations** on compatible queries.

* `<query-1> UNION <query-2>` - This **eliminates** duplicates
* `<query-1> UNION ALL <query-2>` - This **preserves** duplicates
* `<query-1> INTERSECT <query-2>` - This **preserves** similarities between query results
* `<query-1> EXCEPT <query-2>` - This **preserves** differences between query results

All of the above **Set Operations** must be `union-compatible`, in other words they must have the same structure (number of columns, column names, etc.)

#### Query Nesting

With nesting we can use queries as part of other queries, for example we can say `FROM <query>` replacing the table, and as part of a condition in a `WHERE` clause, `WHERE <query>`. The smaller queries inside the larger query are called `sub-queries`. Formally, in th context of this relationshop, a query is referred to as a containing query and a sub-query is referred to as a contained query. Sub-queries that can run independently, in other words that are not dependant on their containing query to produce a result (self-contained), are called Uncorrelated sub-queries. Correlated sub-queries can reference their containing queries.

**Uncorrelated examples**: Inner queries can run independently from their containing queries.

```sql
SELECT subquery.studentname FROM(SELECT studentname FROM studentstable) AS subquery;
SELECT studentname FROM studentstable WHERE gpa >= (SELECT MAX(gpa) FROM studentstable);
```

**Correlated example**: Inner queries are dependant on their containing queries.

```sql
SELECT s1.studentname FROM studentstable s1 WHERE s1.gpa >= ALL(SELECT s2.gpa FROM studentstable s2 WHERE s1.studentname = s2.studentname)
```

**Multiple nested queries example**: Queries nested in queries nested in queries.

```sql
SELECT course.coursename FROM coursestable course WHERE NOT EXISTS (SELECT * FROM studentstable student WHERE NOT EXISTS( SELECT * FROM enrollmenttable enrolled WHERE enrolled.courseid = course.courseid AND enrolled.studentid = student.studentid))
```

#### Sub-Queries in Conditionals

It is possible to use sub-queries in conditional statements that will resolve to a boolean value. For instance, we can check to see if a sub-query returns an empty result by issuing: `EXISTS(<sub-query>)`, which will return `TRUE` if not empty. We can also check for a specific value in a sub-query: `<value> IN (<sub-query>)`, which will resolve to `TRUE` if the value matches. It is also possible to check if a condition holds true for `ALL` rows: `>= ALL(<sub-query>)` or only some. `ANY`: `>= ANY(<sub-query>)`

Here we check for students where gpa is greater or equal to the gpa of all students, this will result in the query returning the maximum gpa. You can conceptualize this nested query as being two nested for-loops, although this is not what the Database System is actually doing to the data.

```sql
SELECT studentname FROM studentstable WHERE gpa >= (SELECT gpa FROM studentstable);
```

## Data Storage

### Data Storage Hardware

The **Memory Hierarchy** in Computer Science, refers to the relation between **access speed** and **storage capacity**. Usually, the higher the speed, the lower the capacity, and so this creates a triangle with very fast memory inside the CPU called **registers** at the top of the pyramid and slower magnetic **tape backups** at the bottom. It is also important to remember that storage is either `volatile` or `non-volatile`, meaning data will either disappear once power is cycled, or persist in a stable way between states. Data must move through this hierarchy up towards the processor, so the processor can access the stored information.

* Registers -- Volatile
  * Fastest Access
  * Smallest Capacity
  * Highest Cost
  * Limited Function
* CPU Cache -- Volatile
  * L1 (fastest), L2 (faster), L3 (fast) Caches
  * Near Instantaneous Access (~1 Nanosecond)
  * Very High Bandwidth (Hundreds to Thousands of GB/s)
  * Very Low Capacity (Tens of Kilobytes)
  * Very Expensive (Hundreds of $/GB)
* Main Memory -- Volatile
  * Very Fast Access (~Tens of Nanoseconds)
  * Low Capacity (Tens to Hundreds of Gigabytes)
  * High Bandwidth (~GB/s)
  * Expensive (Many $/GB)
* Flash Memory -- Non-Volatile
  * Fast Access (~1 Millisecond)
  * High Capacity (Tens to Hundreds of Terabytes)
  * Elevated Read Speed (500MB/s)
  * Less Cheap ($0.25/GB)
* Hard Disks -- Non-Volatile
  * Slow Access (10s of Milliseconds)
  * Moderate Capacity (Tens of Terabytes)
  * Moderate Read Speed (~200MB/s)
  * Cheap ($0.035/GB)
* Tape Backups -- Non-Volatile
  * Slow Access (10s of Seconds)
  * Highest Capacity (Hundreds of Terabytes)
  * Moderate Read Speed (~300MB/s)
  * Very Cheap ($0.02/GB)

### Relevance for DBMS

Databases management system design is influenced by the memory hierarchy. Capacity limits force data down the hierarchy and access speed, which becomes a bottleneck the lower we go down the hierarchy, is influenced by how the databases algorithms were created. Random access to data is 'expensive', in Computer Science terms this costliness is described with mathematical notation referred to as [big-O notation](https://en.wikipedia.org/wiki/Big_O_notation), which describes "limiting behavior of a function when the argument tends towards a particular value".

When designing databases, we ideally want to optimize our system to minimize data movement, this can be achieved by reading data in larger chunks (pages) and keep related data closer together to make the access more efficient. Another consideration is volatility and the ability to recover from failure in the memory hierarchy.

### Data Storage Formats

#### Tables as Files

We can store the information or meta-data for the **Table Schema** in a **database catalogue**, the content of **tables** are then stored as a **collection of pages (files)**, with each page storing a few KB of data. They can store multiple rows but not entire tables. The size of these pages are chosen to maximize retrieval efficiency in relation to the storage medium.

|       |[metadata]|[metadata]|[metadata]|[metadata]| db catalogue |
|-------|----------|----------|----------|----------|--------------|
| page  | [chunks] | [chunks] | [chunks] | [chunks] |              |
| page  | [chunks] | [chunks] | [chunks] | [chunks] |              |

#### From Files to Pages

https://youtu.be/4cWkVbC2bNE?t=12580

## Tree Indexes

## Hash Indexes

## Query Processing Overview

## Operation Implementations

## Hash Join, Sort-Merge Join

## More Operators and Query Plans

## Query Optimization

## Transactions

## Isolation vis Concurrency Control

## Two-Phase Locking

## More on Locking

## Concurrency Control Without Locking

## Recovery After System Crashes 1

## Recovery After System Crashes 2

## NO-SQL

## NEW-SQL

## Graph Data

## Spacial Data

## Data Streams

## Errata

### Popular Databases

A non-exhaustive list of popular databases being used in professional environments as of 2022.

* Redis - an in-memory data structure store, used as a distributed, in-memory key–value database.
* Postgres - an open-source relational database management system emphasizing extensibility and SQL compliance
* MongoDB - a 'source-available' cross-platform document-oriented database program
* SQLite - a small, fast, SQL database engine.
* MS-SQL - a relational database management system developed by Microsoft
* MySQL - an open-source relational database management system
* Apache Cassandra - an open-source, distributed, wide-column store, NoSQL database management system
* Elasticsearch - a distributed, multitenant-capable full-text search engine
* Neo4j - ACID-compliant transactional database with native graph storage and processing