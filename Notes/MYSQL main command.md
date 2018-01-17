Here are the main SQL commands to know for vanilla software engineer interview. Make sure you know at least all of them if you mention that you know mySQL on your CV. 

# Mysql
## Insertion

* Create database

```
CREATE DATABASE database
```

* Use database
```
USE database
```

* Create table
```
key_1 INT NOT_NULL AUTO_INCREMENT PRIMARY KEY,
key_2 VARCHAR(50) NOT NULL,
key_3 TIMESTAMP,
key_4 ENUM('M', 'F'),
key_5 VARCHAR(130) DEFAULT "PA"
)
```

* Describe a tabe

```CREATE TABLE table(
DESCRIBE table
```
* Insert values
```
INSERT INTO table (attribute, attribute2) VALUES (a, b) (a2, b2);
```

* Show values in table

```
SELECT * FROM table
```

* simple selection  with 
```SELECT attr_1, attr_2 COUNT(*), CONCAT(attr_1, " + ", attr_2) as Name,
FROM tables
WHERE YEAR(attr_2) > 1965 OR attr_1 in ("X", "U")
GROUP BY attr_1
ORDER BY attr_1 ASC/DESC
LIMIT 5 OFFSET 1 # 1 to 6 results then
```

You can't group by, after having ordered

* Modify table
```ALTER TABLE table
ADD COLUMN col_name colunm_def
DROP COLUMN col_name ```
Alter Table is known to be very slow, as it create copying over all the data from the table and recreating the indexes
```

## Joining

* Join tables
  You can write tables join with where clause
```SELECT t.attr_1, t_2.attr_1 
FROM table as t, table_2 as t_2
WHERE ...
```

But it is usually better practise to use the SQL JOIN function. Here is a summary of all what can be done?

* INNER JOIN
  Join two tables on given attributes. 
  ```SELECT attr_1, attr_2 FROM table_1 INNER JOIN table_2 on (table_1.attr_3 = table_2.attr_3 AND/OR other attributes)```
  All results have a correspondance in both tables

You can also do multiple join like:
```SELECT attr_1, attr_2 FROM table_1 INNER JOIN table_2 on table_1.attr_3=table_2.attr_3 INNER JOIN table_3 on table_2.attr_4=table_3.attr_4```

* LEFT/RIGHT
  If you join two tables on given column, it might appear that certain value in a column don't appear on both side. If you do a LEFT JOIN, every value in the first table that doesn't have a match in the second table still appears in the results (respective to if it feets the where clause), with every column of the second table set to NULL value
  ```SELECT * FROM table_0 as t_0 LEFT JOIN table_1 as t_1 ON t_0.attr1 = t_1.attr_1```

Here ON defines the relationship between the tables, while the WHERE defines the rows that you interested in.
The reason there is the possibility to right or left join, as both can be written using the other, is because people have different culture of writing from left to right or vice versa, and so some people might prefer a certain type of join

* USING
  if the join is made using the same attribute name, you can use ```INNER JOIN table_2 USING(commun_attr_name)```

* NATURAL JOIN
  Natural join automatically do an INNER JOIN on all the attributes that have the same name for the two joined tables 

## Aggregation and other operations

* GROUP BY and GROUP_CONCAT
  Group function are necessary to get summary statistics of a dataset; COUNT, MAX, MIN, AVG, SUM, DISTINCT are all group functions
```
SELECT attr_1, GROUP_CONCAT(attr_2)
FROM table
GROUP BY attr_1;
```

* HAVING
  The HAVING clause was added to SQL because the WHERE keyword could not be used with aggregate functions.
```
SELECT * FROM table WHERE attr_1=2 GROUP BY attr_2 HAVING COUNT(attr_2)=2;
```
* UNION
  Concatenaste results. Needs to have the same number of columns. They also need to be coherent

```
SELECT a, b, null, d from table0
UNION
SELECT null, b, c, d from table1
```

When you do an UNION, duplicates get removed. UNION ALL keeps all the duplicates.

* LIKE

Pattern searching, "%" match any charaacters, and "_" match any single character.

```
SELECT * FROM table WHERE attr_1 LIKE "begin_middle%"
```
Also exists in negative `NOT LIKE`
LIKE is insensible to upper/lower case, so to make it sensible just put LIKE BINARY

* BETWEEN

```
SELECT * FROM TABLE WHERE attr BETWEEN a AND b
```
* DISTINCT 

```
SELECT DISTINCT * FROM table;
```
## Transaction

Here is the example of a transaction

```
BEGIN TRANSACTION
SELECT * FROM table FOR UPDATE; // This FOR UPDATE tell the database to lock the rows returns
UPDATE SET * WHERE ù
COMMIT;
```
## Indexes

It is possible to add new indexes, to serve certain read queries faster. For example if you often query by a certain column , it might be good to create an index for this table and this given column

You can index on multiple columns. It will sort in the order of the column given. So if you provide (attr_1, attr_2), it will sort by attr1, then attr2. Note that if you build a multi column index (a1, a2, a3), you also construct an index for (a1) or (a1, a2)

#### Create an index with the  table

```
CREATE TABLE table (
	attr_0 ...,
	attr_1 ...,
	INDEX index_name (attr_used_for_index)
)
```

#### Construct an index from an already existing database

```
ALTER TABLE table
ADD INDEX [nom_index] (colonne_index [, colonne2_index ...]);
```

## Event Scheduler

You can schedule events in *MYSQL*, for example so as to delete every rows that have a data older than the current date.

## How requests are handle by Mysql


When doing a MYsql request, it generates an array which contains the execution of the resquest. This table is temporary. To create results table that are permanent, you need to create views

## Normalization and Denormalization

**Normalization** is not for MYSQL, it is a general database concept. It is essentially a process to eliminate redundant data between different tables

**Denormalization**: It is a process to increase the speed of computing, at the expense of adding more write (removing join operation but having some data write at multiple places). It allows to reduce the number of range queries

# How to create database:
1. Create the Entity/Association diagram.
2. Entities have attribute
3. Create Associations  (binary 0.1, 0.n, 1.n) or ternary
4. Create relational model
  -> All entities that possess a cardinality *.1 with another entities via an association get the primary key from the other 
  -> All association which are *.n on both side becomes relation (a new table)
  -> Create a new relation with primary key for ternary association