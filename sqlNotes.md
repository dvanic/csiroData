# 1. Selecting Data

- Have you worked with spreadsheets before? Advanced filtering?

A **relational database** is a way to store and manipulate information. Databases are arranged as tables. Each table has columns (also known as fields) that describe the data, and rows (also known as records) which contain the data.


*NB: Every database manager — Oracle, IBM DB2, PostgreSQL, MySQL, Microsoft Access, and SQLite — stores data in a different way, so a database created with one cannot be used directly by another. However, every database manager can import and export data in a variety of formats, like .csv, so it is possible to move information from one to another.*

**SQL** stands for “Structured Query Language”. 

When we are using a database, we send commands (usually called queries) to a database manager: a program that manipulates the database for us. The database manager does whatever lookups and calculations the query specifies, returning the results in a tabular form that we can then use as a starting point for further queries.


### Getting Into and Out Of SQLite, Getting help
    
    $ cd /path/to/survey/data/
    $ sqlite3 survey.db
    .exit or .quit. 
For some terminals, Ctrl-D can also work. 
    
    .help.
All SQLite-specific commands are prefixed with a . to distinguish them from SQL commands. Type .tables to list the tables in the database.

    .tables
You can change some SQLite settings to make the output easier to read. First, set the output mode to display left-aligned columns. Then turn on the display of column headers.
    
    .mode column
    .header on

### Task 1 (me)
Write an SQL query that displays scientists’ names. 

    SELECT family, personal FROM Person;

- Demonstrate case insensitivity!
- Talk about convention.
- Show that if you forget the ; the command doesn’t finish

- Show that
	- columns can be interchanged in output

          SELECT personal, family FROM Person;
	
	- Or repeated
    
          SELECT id, id, id FROM Person;

	- And wildcards work
    
          SELECT * FROM Person;

***
### Task (STUDENTS)
Write a query that selects only the name column from the Site table.

    SELECT name FROM Site;

***
*** 
# 2. Sorting and Removing Duplicates

In beginning our examination of the Antarctic data, we want to know:

- what kind of quantity measurements were taken at each site;
- which scientists took measurements on the expedition;
- the sites where each scientist took measurements

To determine which measurements were taken at each site, we can examine the Survey table. Data is often redundant, so queries often return redundant information. For example, if we select the quantities that have been measured from the Survey table, we get this:

    SELECT quant FROM Survey;

    SELECT DISTINCT quant FROM Survey;

This can be used across any number of columns to find distinct pairs.

- If we want to determine which sites have which quant measurement, we can use the DISTINCT keyword on multiple columns. 
- If we select more than one column, the distinct pairs of values are returned:
    
      SELECT DISTINCT taken, quant FROM Survey;

Note that (unlike unix) sorting was NOT required!

#### Sorting output

    SELECT * FROM Person ORDER BY id;

Descending 

    SELECT * FROM person ORDER BY id DESC;

Ascending (default)

    SELECT * FROM person ORDER BY id ASC;

In order to look at which scientist measured quantities at each site, we can look again at the Survey table. We can also sort on several fields at once. For example, this query sorts results first in ascending order by taken, and then in descending order by person within each group of equal taken values:

     SELECT taken, person, quant FROM Survey ORDER BY taken ASC, person DESC;

Looking at the table, it seems like some scientists specialized in certain kinds of measurements. We can examine which scientists performed which measurements by selecting the appropriate columns and removing duplicates.

    SELECT DISTINCT quant, person FROM Survey ORDER BY quant ASC;

***
### TASK (Students)
Write a query that selects distinct dates from the Visited table.

    SELECT DISTINCT dated FROM Visited;

### Task2 (Students)
Write a query that displays the full names of the scientists in the Person table, ordered by family name.

    SELECT personal, family FROM Person ORDER BY family;


*** 
***

# 3. Filtering = super useful

Select only records from site DR-1 in Visited
```
SELECT * FROM Visited WHERE site='DR-1';
```

First, it checks at each row in the Visited table to see which ones satisfy the WHERE. It then uses the column names following the SELECT keyword to determine which columns to display.

This processing order means that we can filter records using WHERE based on values in columns that aren’t then displayed:
 
FIGURE HERE

#### Boolean operators for filtering: and

We can use many other Boolean operators to filter our data. We can ask for all information from the DR-1 site collected before 1930:

    SELECT * FROM Visited WHERE site='DR-1' AND dated<'1930-01-01';

#### DATE Types
Most database managers have a special data type for dates. In fact, many have two: one for dates, such as “May 31, 1971”, and one for durations, such as “31 days”. SQLite doesn’t: instead, it stores dates as either text (in the ISO-8601 standard format “YYYY-MM-DD HH:MM:SS.SSSS”), real numbers (Julian days, the number of days since November 24, 4714 BCE), or integers (Unix time, the number of seconds since midnight, January 1, 1970). 


#### Boolean operators for filtering: or
What measurements were taken by either Lake or Roerich?

    SELECT * FROM Survey WHERE person='lake' OR person='roe';

Alternatively, we can use IN to see if a value is in a specific set:

    SELECT * FROM Survey WHERE person IN ('lake', 'roe');

Parentheses matter!

Want to get salinity measured by lake or Roerich…

    SELECT * FROM Survey WHERE quant='sal' AND person='lake' OR person='roe';

Compare!

    SELECT * FROM Survey WHERE quant='sal' AND (person='lake' OR person='roe');


### Like (partial matches)

The percent symbol acts as a wildcard, matching any characters in that place. It can be used at the beginning, middle, or end of the string:

    SELECT * FROM Visited WHERE site LIKE 'DR%';

Finally, we can use DISTINCT with WHERE to give a second level of filtering:

    SELECT DISTINCT person, quant FROM Survey WHERE person='lake' OR person='roe';

Growing queries 
-	how we’ve been working so far
-	small testable subset!!!


***
####Task (Students)
Suppose we want to select all sites that lie more than 42 degrees from the poles. Our first query is:

    SELECT * FROM Site WHERE (lat > -48) OR (lat < 48);

Correct
Because we used OR, a site on the South Pole for example will still meet the second criteria and thus be included. Instead, we want to restrict this to sites that meet both criteria:


    SELECT * FROM Site WHERE (lat > -48) AND (lat < 48);


#### Task (Students) – outliers AKA weird readings
Normalized salinity readings are supposed to be between 0.0 and 1.0. Write a query that selects all records from Survey with salinity values outside this range.

```
SELECT * FROM Survey WHERE (quant == "sal") AND (reading < 0 OR reading > 1);
SELECT * FROM Survey WHERE quant='sal' AND ((reading > 1.0) OR (reading < 0.0));
```


Which of these expressions are true?
```
1.	'a' LIKE 'a'
2.	'a' LIKE '%a'
3.	'beta' LIKE '%a'
4.	'alpha' LIKE 'a%%'
5.	'alpha' LIKE 'a%p%'
```
/all true/


***
***

# 4. Calculating New Values
Multiply column by a value
SELECT 1.05 * reading FROM Survey WHERE quant='rad';

Expressions can use any of the fields, all of usual arithmetic operators, and a variety of common functions. (Exactly which ones depends on which database manager is being used.) \

Convert temperature readings from Fahrenheit to Celsius and round to two decimal places:
SELECT taken, round(5*(reading-32)/9, 2) FROM Survey WHERE quant='temp';
Full mutate syntax
SELECT taken, round(5*(reading-32)/9, 2) as Celsius FROM Survey WHERE quant='temp';

We can also combine values from different fields, for example by using the string concatenation operator ||:

SELECT personal || ' ' || family FROM Person;

The UNION operator combines the results of two queries:
SELECT * FROM Person WHERE id='dyer' UNION SELECT * FROM Person WHERE id='roe';




### Students task
After further reading, we realize that Valentina Roerich was reporting salinity as percentages. Write a query that returns all of her salinity measurements from the Survey table with the values divided by 100.

    SELECT taken, reading / 100 FROM Survey WHERE person='roe' AND quant='sal';

### Task
Use UNION to create a consolidated list of salinity measurements in which Valentina Roerich’s, and only Valentina’s, have been corrected as described in the previous challenge. The output should be something like:
taken	reading
619	0.13
622	0.09
734	0.05
751	0.1
752	0.09
752	0.416
837	0.21
837	0.225

```
SELECT taken,reading FROM Survey WHERE person!='roe' AND quant='sal' UNION SELECT taken,reading / 100 FROM Survey WHERE person='roe' AND quant='sal' ORDER BY taken ASC;
```


### Task 3
The site identifiers in the Visited table have two parts separated by a ‘-‘:
SELECT DISTINCT site FROM Visited;
site
DR-1
DR-3
MSK-4
Some major site identifiers (i.e. the letter codes) are two letters long and some are three. The “in string” function instr(X, Y) returns the 1-based index of the first occurrence of string Y in string X, or 0 if Y does not exist in X. The substring function substr(X, I, [L]) returns the substring of X starting at index I, with an optional length L. Use these two functions to produce a list of unique major site identifiers. (For this data, the list should contain only “DR” and “MSK”).

```
SELECT DISTINCT substr(site, 1, instr(site, '-') - 1) AS MajorSite FROM Visited;
```


***
***

# 5. Missing Data

- Null aka NA in R

Dealing with null requires a few special tricks and some careful thinking.

Visited table. There are eight records, but #752 doesn’t have a date — or rather, its date is null:
SELECT * FROM Visited;

Like in R:
Null doesn’t behave like other values; before/after 1930
```
SELECT * FROM Visited WHERE dated<'1930-01-01';
SELECT * FROM Visited WHERE dated>='1930-01-01';
```

Null is missing in both!

To check whether a value is null or not, we must use a special test IS NULL:

```
SELECT * FROM Visited WHERE dated IS NULL;
```
Or not…
```
SELECT * FROM Visited WHERE dated IS NOT NULL;
```

Measurements by Lake (or maybe Lake?)
    
    SELECT * FROM Survey WHERE quant='sal' AND (person!='lake' OR person IS NULL);

UNLIKE R though!!!

In contrast to arithmetic or Boolean operators, aggregation functions that combine multiple values, such as min, max or avg, ignore null values. In the majority of cases, this is a desirable output: for example, unknown values are thus not affecting our data when we are averaging it. 

***
####TASK
Write a query that sorts the records in Visited by date, omitting entries for which the date is not known (i.e., is null).

    SELECT * FROM Visited WHERE Dated IS NOT NULL ORDER BY Dated;

####TASK
What do you expect the query:

    SELECT * FROM Visited WHERE dated IN ('1927-02-08', NULL);

to produce? What does it actually produce?

####TASK
Discussion point:

Some database designers prefer to use a sentinel value to mark missing data rather than null. For example, they will use the date “0000-00-00” to mark a missing date, or -1.0 to mark a missing salinity or radiation reading (since actual readings cannot be negative). What does this simplify? What burdens or risks does it introduce?



***
***
# 6. Aggregation

We now want to calculate ranges and averages for our data. 

```
SELECT min(dated) FROM Visited;

SELECT max(dated) FROM Visited;

SELECT avg(reading) FROM Survey WHERE quant='sal';

SELECT count(reading) FROM Survey WHERE quant='sal';

SELECT sum(reading) FROM Survey WHERE quant='sal';
```

 find the range of sensible salinity measurements:

```
SELECT min(reading), max(reading) FROM Survey WHERE quant='sal' AND reading<=1.0;
```

We can also combine aggregated results with raw results, although the output might surprise you:
```
SELECT person, count(*) FROM Survey WHERE quant='sal' AND reading<=1.0;
```

Why does Lake’s name appear rather than Roerich’s or Dyer’s? The answer is that when it has to aggregate a field, but isn’t told how to, the database manager chooses an actual value from the input set. It might use the first one processed, the last one, or something else entirely.

Another important fact is that when there are no values to aggregate — for example, where there are no rows satisfying the WHERE clause — aggregation’s result is “don’t know” rather than zero or some other arbitrary value:
SELECT person, max(reading), sum(reading) FROM Survey WHERE quant='missing';

Aggregating all records at once doesn’t always make sense. For example, suppose we suspect that there is a systematic bias in our data, and that some scientists’ radiation readings are higher than others. We know that this doesn’t work:
```
SELECT person, count(reading), round(avg(reading), 2)
FROM  Survey
WHERE quant='rad';
```
because the database manager selects a single arbitrary scientist’s name rather than aggregating separately for each scientist. Since there are only five scientists, we could write five queries of the form:

What we need to do is tell the database manager to aggregate the hours for each scientist separately using a GROUP BY clause:
```
SELECT   person, count(reading), round(avg(reading), 2)
FROM     Survey
WHERE    quant='rad'
GROUP BY person;
```
Just as we can sort by multiple criteria at once, we can also group by multiple criteria. To get the average reading by scientist and quantity measured, for example, we just add another field to the GROUP BY clause:
```
SELECT   person, quant, count(reading), round(avg(reading), 2)
FROM     Survey
GROUP BY person, quant;
Let’s go one step further and remove all the entries where we don’t know who took the measurement:
SELECT   person, quant, count(reading), round(avg(reading), 2)
FROM     Survey
WHERE    person IS NOT NULL
GROUP BY person, quant
ORDER BY person, quant;
```

### Extra - having - The HAVING keyword
we have seen the keyword WHERE, allowing to filter the results according to some criteria. SQL offers a mechanism to filter the results based on aggregate functions, through the HAVING keyword.

For example, we can request to only return infomation about values that have been collected by people who have surveyed more than 3 sites:

    SELECT person FROM Survey GROUP BY person HAVING COUNT(taken) > 3;


### Extra - Saving Queries for Future Use

Records from 1930 only
```
CREATE VIEW obs1930 AS
SELECT * FROM Visited WHERE substr(dated,1,4)  = "1930";
SELECT * FROM obs1930;
```



***
####TASK
How many temperature readings did Frank Pabodie record, and what was their average value?

    SELECT count(reading), avg(reading) FROM Survey WHERE quant='temp' AND person='pb';


####TASK
The average of a set of values is the sum of the values divided by the number of values. Does this mean that the avg function returns 2.0 or 3.0 when given the values 1.0, null, and 5.0?
3 because it’s diving by 2.
```
SELECT AVG(a) FROM (
    SELECT 1 AS a
    UNION ALL SELECT NULL
    UNION ALL SELECT 5);
```

####TASK
We want to calculate the difference between each individual radiation reading and the average of all the radiation readings. We write the query:

    SELECT reading - avg(reading) FROM Survey WHERE quant='rad';

What does this actually produce, and why?


####TASK

The function group_concat(field, separator) concatenates all the values in a field using the specified separator character (or ‘,’ if the separator isn’t specified). Use this to produce a one-line list of scientists’ names, such as:

William Dyer, Frank Pabodie, Anderson Lake, Valentina Roerich, Frank Danforth

Can you find a way to order the list by surname?

    SELECT group_concat(temp)  FROM (SELECT personal || ' ' || family  as temp FROM Person ORDER BY family);

## Extratask 
 Create a view for only the DR sites from the visited table

     SELECT  * FROM Visited WHERE substr(site, 1, instr(site, '-') - 1) == "DR";


***
***
# 7. Combining Data
Lets make lat/long/visited table…

[relationshipmap](sql-join-structure.svg)TODO


The SQL command to do this is JOIN. To see how it works, let’s start by joining the Site and Visited tables:

    SELECT * FROM Site JOIN Visited;

JOIN creates the **cross product** of two tables, i.e., it joins each record of one table with each record of the other table to give all possible combinations. Since there are three records in Site and eight in Visited, the join’s output has 24 records (3 * 8 = 24) . And since each table has three fields, the output has six fields (3 + 3 = 6).
What the join hasn’t done is figure out if the records being joined have anything to do with each other. It has no way of knowing whether they do or not until we tell it how. To do that, we add a clause specifying that we’re only interested in combinations that have the same site name, thus we need to use a filter:

     SELECT * FROM Site JOIN Visited ON Site.name=Visited.site;

Table.field to specify field names in the output of the join.


```
SELECT Site.lat, Site.long, Visited.dated
FROM   Site JOIN Visited
ON     Site.name=Visited.site;
```

If joining two tables is good, joining many tables must be better. In fact, we can join any number of tables simply by adding more JOIN clauses to our query, and more ON tests to filter out combinations of records that don’t make sense:
```
SELECT Site.lat, Site.long, Visited.dated, Survey.quant, Survey.reading
FROM   Site JOIN Visited JOIN Survey
ON     Site.name=Visited.site
AND    Visited.id=Survey.taken
AND    Visited.dated IS NOT NULL;
```

We can tell which records from Site, Visited, and Survey correspond with each other because those tables contain primary keys and foreign keys. A primary key is a value, or combination of values, that uniquely identifies each record in a table. A foreign key is a value (or combination of values) from one table that identifies a unique record in another table. 


Another way of saying this is that a foreign key is the primary key of one table that appears in some other table. In our database, Person.id is the primary key in the Person table, while Survey.person is a foreign key relating the Survey table’s entries to entries in Person.

Most database designers believe that every table should have a well-defined primary key. They also believe that this key should be separate from the data itself, so that if we ever need to change the data, we only need to make one change in one place. One easy way to do this is to create an arbitrary, unique ID for each record as we add it to the database. 

As the query below demonstrates, SQLite automatically numbers records as they’re added to tables, and we can use those record numbers in queries:

```
SELECT rowid, * FROM Person;
```

http://sql-joins.leopard.in.ua/

***
TASK
Write a query that lists all radiation readings from the DR-1 site.


```
SELECT Survey.reading 
FROM Site JOIN Visited JOIN Survey 
ON Site.name=Visited.site
AND Visited.id=Survey.taken
WHERE Site.name="DR-1" 
AND Survey.quant ="rad";
```

TASK
Write a query that lists all sites visited by people named “Frank”.
```
SELECT DISTINCT Site.name
FROM Site JOIN Visited JOIN Survey JOIN Person
ON Site.name=Visited.site
AND Visited.id=Survey.taken
AND Survey.person=Person.id
WHERE Person.personal="Frank";
```

TASK
Describe what this is doing
```
SELECT Site.name FROM Site JOIN Visited
ON Site.lat<-49.0 AND Site.name=Visited.site AND Visited.dated>='1932-01-01';
```

TASK


Write a query that shows each site with exact location (lat, long) ordered by visited date, followed by personal name and family name of the person who visited the site and the type of measurement taken and its reading. Please avoid all null values. Tip: you should get 15 records with 8 fields.
```
SELECT Site.name, Site.lat, Site.long, Person.personal, Person.family, Survey.quant, Survey.reading, Visited.dated
FROM Site JOIN Visited JOIN Survey JOIN Person
ON Site.name=Visited.site
AND Visited.id=Survey.taken
AND Survey.person=Person.id
WHERE Survey.person IS NOT NULL
AND Visited.dated IS NOT NULL
ORDER BY Visited.dated;
```

Extra Task:
Join all of the tables together to make a single table, as if you were going to write out the data to a flat spreadsheet.

```
SELECT * FROM Survey JOIN Visited JOIN Site JOIN Person ON Survey.taken = Visited.id AND Site.name = Visited.site AND Person.id = Survey.person;
```
***

## Data Hygiene

1. Every value should be atomic, i.e., not contain parts that we might want to work with separately. (First name, last name example)
2. every record should have a unique primary key
	- a serial number that has no intrinsic meaning
	- one of the values in the record (like the id field in the Person table)
 	- a combination of values: the triple (taken, person, quant) from the Survey table uniquely identifies every measurement
3. there should be no redundant information (unlike spreadsheet!)
4. the units for every value should be stored explicitly

***
### Task - Identifying Atomic Values
Which of the following are atomic values? Which are not? Why?

- New Zealand
- 87 Turing Avenue
- January 25, 1971
- the XY coordinate (0.5, 3.3)

### Identifying a Primary Key
What is the primary key in this table? I.e., what value or combination of values uniquely identifies a record?

latitude longitude	date	temperature
57.3	-22.5	2015-01-09	-14.2

    Latitude, longitude, and date are all required to uniquely identify the temperature record.

***

## Creating and Modifying Data

CREATE TABLE and DROP TABLE.

- each is a single command


      CREATE TABLE Person1(id text, personal text, family text);
      CREATE TABLE Site1(name text, lat real, long real);
      CREATE TABLE Visited1(id integer, site text, dated text);
      CREATE TABLE Survey1(taken integer, person text, quant real, reading real);

We can get rid of one of our tables using:

      DROP TABLE Survey1;

HERE BE DRAGONS


Different database systems support different data types for table columns, but most provide the following:

- INTEGER	a signed integer
- REAL	a floating point number
- TEXT	a character string
- BLOB	a “binary large object”, such as an image

Most databases also support Booleans and date/time values; SQLite uses the integers 0 and 1.


When we create a table, we can specify several kinds of constraints on its columns. For example, a better definition for the Survey table would be:

```
	CREATE TABLE Survey1(
    taken   integer not null, -- where reading taken
    person  text,             -- may not know who took it
    quant   real not null,    -- the quantity measured
    reading real not null,    -- the actual reading
    primary key(taken, quant),
    foreign key(taken) references Visited(id),
    foreign key(person) references Person(id)
);
```


Once tables have been created, we can add, change, and remove records using our other set of commands, INSERT, UPDATE, and DELETE.


The simplest form of INSERT statement lists values in order:

```
INSERT INTO Site1 VALUES('DR-1', -49.85, -128.57);
INSERT INTO Site1 VALUES('DR-3', -47.15, -126.72);
INSERT INTO Site1 VALUES('MSK-4', -48.87, -123.40);
```

We can also insert values into one table directly from another:

```
CREATE TABLE JustLatLong(lat text, long text);
INSERT INTO JustLatLong SELECT lat, long FROM Site;
```

Modifying existing records is done using the UPDATE statement. To do this we tell the database which table we want to update, what we want to change the values to for any or all of the fields, and under what conditions we should update the values.

If we made a mistake when entering the lat and long values of the last INSERT statement above:

    UPDATE Site1 SET lat=-47.87, long=-122.40 WHERE name='MSK-4';

!!! Be careful to not forget the where clause or the update statement will modify all of the records in the database.



Deleting records can be a bit trickier, because we have to ensure that the database remains internally consistent. If all we care about is a single table, we can use the DELETE command with a WHERE clause that matches the records we want to discard. For example, once we realize that Frank Danforth didn’t take any measurements, we can remove him from the Person table like this:

    DELETE FROM Person WHERE id = 'danforth';


But what if we removed Anderson Lake instead? Our Survey table would still contain seven records of measurements he’d taken, but that’s never supposed to happen: Survey.person is a foreign key into the Person table, and all our queries assume there will be a row in the latter matching every value in the former.

This problem is called referential integrity: we need to ensure that all references between tables can always be resolved correctly. One way to do this is to delete all the records that use 'lake' as a foreign key before deleting the record that uses it as a primary key. If our database manager supports it, we can automate this using cascading delete. However, this technique is outside the scope of this chapter.


TODO cascade delete here https://www.techonthenet.com/sqlite/foreign_keys/foreign_delete.php


#### Hybrid Storage Models
- actual data as files
- filenames into database

*** 
#### TASK - Replacing NULL
Write an SQL statement to replace all uses of null in Survey.person with the string 'unknown'.

    UPDATE Survey SET person="unknown" WHERE person IS NULL;


#### Generating Insert Statements
One of our colleagues has sent us a CSV file containing temperature readings by Robert Olmstead, which is formatted like this:

Taken,Temp
619,-21.5
622,-15.5

Write a small Python program that reads this file in and prints out the SQL INSERT statements needed to add these records to the survey database. Note: you will need to add an entry for Olmstead to the Person table. If you are testing your program repeatedly, you may want to investigate SQL’s INSERT or REPLACE command.


#### Backing up with SQL

SQLite has several administrative commands that aren’t part of the SQL standard. One of them is .dump, which prints the SQL commands needed to re-create the database. Another is .read, which reads a file created by .dump and restores the database. A colleague of yours thinks that storing dump files (which are text) in version control is a good way to track and manage changes to the database. What are the pros and cons of this approach? (Hint: records aren’t stored in any particular order.)


Ans
Advantages
A version control system will be able to show differences between versions of the dump file; something it can’t do for binary files like databases
A VCS only saves changes between versions, rather than a complete copy of each version (save disk space)
The version control log will explain the reason for the changes in each version of the database
Disadvantages
Artificial differences between commits because records don’t have a fixed order

***

## Python interface
```
import sqlite3

connection = sqlite3.connect("survey.db")
cursor = connection.cursor()
cursor.execute("SELECT Site.lat, Site.long FROM Site;")
results = cursor.fetchall()
for r in results:
    print(r)
cursor.close()
connection.close()
```

The program starts by importing the sqlite3 library.

Line 2 establishes a connection to the database. Since we’re using SQLite, all we need to specify is the name of the database file. Other systems may require us to provide a username and password as well. Line 3 then uses this connection to create a cursor. Just like the cursor in an editor, its role is to keep track of where we are in the database.

On line 4, we use that cursor to ask the database to execute a query for us. The query is written in SQL, and passed to cursor.execute as a string. It’s our job to make sure that SQL is properly formatted; if it isn’t, or if something goes wrong when it is being executed, the database will report an error.

The database returns the results of the query to us in response to the cursor.fetchall call on line 5. This result is a list with one entry for each record in the result set; if we loop over that list (line 6) and print those list entries (line 7), we can see that each one is a tuple with one element for each field we asked for.

Finally, lines 8 and 9 close our cursor and our connection, since the database can only keep a limited number of these open at one time. Since establishing a connection takes time, though, we shouldn’t open a connection, do one operation, then close the connection, only to reopen it a few microseconds later to do another operation. Instead, it’s normal to create one connection that stays open for the lifetime of the program.


```
import sqlite3

def get_name(database_file, person_id):
    query = "SELECT personal || ' ' || family FROM Person WHERE id='" + person_id + "';"

    connection = sqlite3.connect(database_file)
    cursor = connection.cursor()
    cursor.execute(query)
    results = cursor.fetchall()
    cursor.close()
    connection.close()

    return results[0][0]

print("Full name for dyer:", get_name('survey.db', 'dyer'))
```

Discuss if input is

    dyer'; DROP TABLE Survey; SELECT '



Solution

Use a prepared statement:


```
import sqlite3

def get_name(database_file, person_id):
    query = "SELECT personal || ' ' || family FROM Person WHERE id=?;"

    connection = sqlite3.connect(database_file)
    cursor = connection.cursor()
    cursor.execute(query, [person_id])
    results = cursor.fetchall()
    cursor.close()
    connection.close()

    return results[0][0]

print("Full name for dyer:", get_name('survey.db', 'dyer'))
```


The key changes are in the query string and the execute call. Instead of formatting the query ourselves, we put question marks in the query template where we want to insert values. When we call execute, we provide a list that contains as many values as there are question marks in the query. The library matches values to question marks in order, and translates any special characters in the values into their escaped equivalents so that they are safe to use.




We can also use sqlite3’s cursor to make changes to our database, such as inserting a new name. For instance, we can define a new function called add_name like so:



```
import sqlite3

def add_name(database_file, new_person):
    query = "INSERT INTO Person VALUES (?, ?, ?);"

    connection = sqlite3.connect(database_file)
    cursor = connection.cursor()
    cursor.execute(query, list(new_person))
    cursor.close()
    connection.close()


def get_name(database_file, person_id):
    query = "SELECT personal || ' ' || family FROM Person WHERE id=?;"

    connection = sqlite3.connect(database_file)
    cursor = connection.cursor()
    cursor.execute(query, [person_id])
    results = cursor.fetchall()
    cursor.close()
    connection.close()

    return results[0][0]

# Insert a new name
add_name('survey.db', ('barrett', 'Mary', 'Barrett'))
# Check it exists
print("Full name for barrett:", get_name('survey.db', 'barrett'))
```

Note that in versions of sqlite3 >= 2.5, the get_name function described above will fail with an IndexError: list index out of range, even though we added Mary’s entry into the table using add_name. This is because we must perform a connection.commit() before closing the connection, in order to save our changes to the database.

```
import sqlite3

def add_name(database_file, new_person):
    query = "INSERT INTO Person VALUES (?, ?, ?);"

    connection = sqlite3.connect(database_file)
    cursor = connection.cursor()
    cursor.execute(query, list(new_person))
    cursor.close()
    connection.commit()
    connection.close()


def get_name(database_file, person_id):
    query = "SELECT personal || ' ' || family FROM Person WHERE id=?;"

    connection = sqlite3.connect(database_file)
    cursor = connection.cursor()
    cursor.execute(query, [person_id])
    results = cursor.fetchall()
    cursor.close()
    connection.close()

    return results[0][0]

# Insert a new name
add_name('survey.db', ('barrett', 'Mary', 'Barrett'))
# Check it exists
print("Full name for barrett:", get_name('survey.db', 'barrett'))
```

***
TASK - Filling a Table vs. Printing Values
Write a Python program that creates a new database in a file called original.db containing a single table called Pressure, with a single field called reading, and inserts 100,000 random numbers between 10.0 and 25.0. How long does it take this program to run? How long does it take to run a program that simply writes those random numbers to a file?

```
import sqlite3
# import random number generator
from numpy.random import uniform

random_numbers = uniform(low=10.0, high=25.0, size=100000)

connection = sqlite3.connect("original.db")
cursor = connection.cursor()
cursor.execute("CREATE TABLE Pressure (reading float not null)")
query = "INSERT INTO Pressure values (?);"

for number in random_numbers:
    cursor.execute(query, [number])

cursor.close()
# save changes to file for next exercise
connection.commit()
connection.close()
```

Same thing, write to text
```
from numpy.random import uniform

random_numbers = uniform(low=10.0, high=25.0, size=100000)
with open('random_numbers.txt', 'w') as outfile:
    for number in random_numbers:
        # need to add linebreak \n
        outfile.write("{}\n".format(number))

```        

TASK - Write a Python program that creates a new database called backup.db with the same structure as original.db and copies all the values greater than 20.0 from original.db to backup.db. Which is faster: filtering values in the query, or reading everything into memory and filtering in Python?


The first example reads all the data into memory and filters the numbers using the if statement in Python.

```python
import sqlite3

connection_original = sqlite3.connect("original.db")
cursor_original = connection_original.cursor()
cursor_original.execute("SELECT * FROM Pressure;")
results = cursor_original.fetchall()
cursor_original.close()
connection_original.close()

connection_backup = sqlite3.connect("backup.db")
cursor_backup = connection_backup.cursor()
cursor_backup.execute("CREATE TABLE Pressure (reading float not null)")
query = "INSERT INTO Pressure values (?);"

for entry in results:
    # number is saved in first column of the table
    if entry[0] > 20.0:
        cursor_backup.execute(query, entry)

cursor_backup.close()
connection_backup.commit()
connection_backup.close()
```
In contrast the following example uses the conditional SELECT statement to filter the numbers in SQL. The only lines that changed are in line 5, where the values are fetched from original.db and the for loop starting in line 15 used to insert the numbers into backup.db. Note how this version does not require the use of Python’s if statement.

```python
import sqlite3

connection_original = sqlite3.connect("original.db")
cursor_original = connection_original.cursor()
cursor_original.execute("SELECT * FROM Pressure WHERE reading > 20.0;")
results = cursor_original.fetchall()
cursor_original.close()
connection_original.close()

connection_backup = sqlite3.connect("backup.db")
cursor_backup = connection_backup.cursor()
cursor_backup.execute("CREATE TABLE Pressure (reading float not null)")
query = "INSERT INTO Pressure values (?);"

for entry in results:
    cursor_backup.execute(query, entry)

cursor_backup.close()
connection_backup.commit()
connection_backup.close()
```

***
***
# R and SQL
Here’s a short R program that selects latitudes and longitudes from an SQLite database stored in a file called survey.db:


```{r}
library(RSQLite)
connection <- dbConnect(SQLite(), "../survey.db")
results <- dbGetQuery(connection, "SELECT Site.lat, Site.long FROM Site;")
print(results)
dbDisconnect(connection)

```

For example, this function takes a user’s ID as a parameter and returns their name:
Wrong WAY:
```{r}
connection <- dbConnect(SQLite(), "../survey.db")

getName <- function(personID) {
  query <- paste0("SELECT personal || ' ' || family FROM Person WHERE id =='",
                  personID, "';")
  return(dbGetQuery(connection, query))
}

print(paste("full name for dyer:", getName('dyer')))

dbDisconnect(connection)
```

But if we use 

    dyer'; DROP TABLE Survey; SELECT '



Correct way (just show this...)

```{r}
library(RSQLite)
connection <- dbConnect(SQLite(), "survey.db")

getName <- function(personID) {
  query <- "SELECT personal || ' ' || family FROM Person WHERE id == ?"
  return(dbGetPreparedQuery(connection, query, data.frame(personID)))
}

print(paste("full name for dyer:", getName('dyer')))

dbDisconnect(connection)
```



### Database helper functions in R
R’s database interface packages (like RSQLite) all share a common set of helper functions useful for exploring databases and reading/writing entire tables at once.

To view all tables in a database, we can use dbListTables():

```
connection <- dbConnect(SQLite(), "survey.db")
dbListTables(connection)
```


To view all column names of a table, use dbListFields():

    dbListFields(connection, "Survey")

To read an entire table as a dataframe, use dbReadTable():

    dbReadTable(connection, "Person")


To write an entire table to a database, you can use dbWriteTable(). Note that we will always want to use the row.names = FALSE argument or R will write the row names as a separate column. In this example we will write R’s built-in iris dataset as a table in survey.db.

```
dbWriteTable(connection, "iris", iris, row.names = FALSE)
head(dbReadTable(connection, "iris"))
```

REMEMBER TO CLOSE DATABASE
```
dbDisconnect(connection)
```

### Working with dbplyr

```
install.packages("dbplyr")
download.file(url = "https://ndownloader.figshare.com/files/2292171", destfile = "portal_mammals.sqlite", mode = "wb")
library(dplyr)
library(dbplyr)
mammals <- DBI::dbConnect(RSQLite::SQLite(), "data/portal_mammals.sqlite")
src_dbi(mammals)
```
DBI is not something that you’ll use directly as a user. It allows R to send commands to databases irrespective of the database management system used. The RSQLite package allows R to interface with SQLite databases.

This command does not load the data into the R session (as the read_csv() function did). Instead, it merely instructs R to connect to the SQLite database contained in the portal_mammals.sqlite file.

We can query the table using dbplyr syntax:
```
tbl(mammals, sql("SELECT year, species_id, plot_id FROM surveys"))

```
Querying the database with the dplyr syntax

```
surveys <- tbl(mammals, "surveys")
surveys %>% select(year, species_id, plot_id)
head(surveys, n = 10)
```

But then some stuff doesn't work ... why?

```{r}
nrow(tbl)
```

To lift the curtain, we can use dplyr’s show_query() function to show which SQL commands are actually sent to the database:

```{r}
show_query(head(surveys, n = 10))
```

R never gets to see the full surveys table - and that’s why it could not tell us how many rows it contains. On the bright side, this allows us to work with large datasets - even too large to fit into our computer’s memory.

First, let’s only request rows of the surveys table in which weight is less than 5 and keep only the species_id, sex, and weight columns.

```
surveys %>%
  filter(weight < 5) %>%
  select(species_id, sex, weight)
```

### Laziness
Hadley Wickham explains:When working with databases, dplyr tries to be as lazy as possible:

- It never pulls data into R unless you explicitly ask for it.
- It delays doing any work until the last possible moment - it collects together everything you want to do and then sends it to the database in one step.

If we wanted to, we could add on even more steps, e.g. remove the sex column in an additional select call:

```
data_subset <- surveys %>%
  filter(weight < 5) %>%
  select(species_id, sex, weight)%>%
  select(-sex)
```


To instruct R to stop being lazy, e.g. to retrieve all of the query results from the database, we add the  collect() command to our pipe. It indicates that our database query is finished: time to get the final results and load them into the R session.

```
data_subset <- surveys %>%
  filter(weight < 5) %>%
  select(species_id, sex, weight) %>%
  collect()
```


### Lets do some joins
```
plots <- tbl(mammals, "plots")
```

For example, to extract all surveys for the first plot, which has plot_id 1, we can do:

```
plots %>%
  filter(plot_id == 1) %>%
  inner_join(surveys) %>%
  collect()
```

#### TASK
Write a query that returns the number of rodents observed in each plot in each year.

Hint: Connect to the species table and write a query that joins the species and survey tables together to exclude all non-rodents. The query should return counts of rodents by year.

Optional: Write a query in SQL that will produce the same result. You can join multiple tables together using the following syntax where foreign key refers to your unique id (e.g., species_id):

```
## with dplyr syntax
species <- tbl(mammals, "species")

left_join(surveys, species) %>%
  filter(taxa == "Rodent") %>%
  group_by(taxa, year) %>%
  tally %>%
  collect()

## with SQL syntax
query <- paste("
SELECT a.year, b.taxa,count(*) as count
FROM surveys a
JOIN species b
ON a.species_id = b.species_id
AND b.taxa = 'Rodent'
GROUP BY a.year, b.taxa",
sep = "" )

tbl(mammals, sql(query))

```


#### Write a query that returns the total number of rodents in each genus caught in the different plot types.

Hint: Write a query that joins the species, plot, and survey tables together. The query should return counts of genus by plot type.

```
species <- tbl("mammals", species)
genus_counts <- left_join(surveys, plots) %>%
  left_join(species) %>%
  group_by(plot_type, genus) %>%
  tally %>%
  collect()

```
TODO write this in SQL



if we were interested in the number of genera found in each plot type? Using  tally() gives the number of individuals, instead we need to use n_distinct() to count the number of unique values found in a column.


```
xspecies <- tbl(mammals, "species")
unique_genera <- left_join(surveys, plots) %>%
    left_join(species) %>%
    group_by(plot_type) %>%
    summarize(
        n_genera = n_distinct(genus)
    ) %>%
    collect()
```

***
### Creating a new SQLite database (using R)

```
species2 <- read_csv("data/species.csv")
surveys2 <- read_csv("data/surveys.csv")
plots2 <- read_csv("data/plots.csv")
my_db <- src_sqlite("portal-database2.sqlite", create = TRUE)
my_db
copy_to(my_db, surveys)
copy_to(my_db, plots)
my_db
```

PROBLEM IF DATASET IS HUGE (WE just piped it through, i.e. loaded into memory)


### Creating a new database (from within SQL)
```
sqlite3 newdb.db
.mode csv
.tables
.import species.csv
.import species.csv Species
.tables
.schema
.import plots.csv Plots
.import surveys.csv Surveys
```

```
sqlite3 newdb2.db
.mode csv
CREATE TABLE surveys (
	record_id BIGINT,
	month BIGINT,
	day BIGINT,
	year BIGINT,
	plot_id BIGINT,
	species_id TEXT,
	sex TEXT,
	hindfoot_length FLOAT,
	weight FLOAT
);
.import surveys.csv Surveys
```


More resources here: http://db.rstudio.com/



***

# Day 2 warmup - from DC

SQL queries help us ask specific questions which we want to answer about our data. The real skill with SQL is to know how to translate our scientific questions into a sensible SQL query (and subsequently visualize and interpret our results).

The data we will be using is a time-series for a small mammal community in southern Arizona. This is part of a project studying the effects of rodents and ants on the plant community that has been running for almost 40 years. The rodents are sampled on a series of 24 plots, with different experimental manipulations controlling which rodents are allowed to access which plots.


Have a look at the following questions; these questions are written in plain English. Can you translate them to SQL queries and give a suitable answer?

1. How many plots from each type are there?
2. How many specimens are of each sex are there for each year?
3. How many specimens of each species were captured in each type of plot?
4. What is the average weight of each taxa?
5. What are the minimum, maximum and average weight for each species of Rodent?
6. What is the average hindfoot length for male and female rodent of each species? Is there a Male / Female difference?
7. What is the average weight of each rodent species over the course of the years? Is there any noticeable trend for any of the species?


```
SELECT plot_type, COUNT(*) AS num_plots FROM plots GROUP BY plot_type;
SELECT year, sex, COUNT(*) AS num_animal FROM surveys WHERE sex IS NOT NULL GROUP BY sex, year;
SELECT species_id, plot_type, COUNT(*) FROM surveys JOIN plots USING(plot_id) WHERE species_id IS NOT NULL  GROUP BY species_id, plot_type;
SELECT taxa, AVG(weight)  FROM surveys  JOIN species ON species.species_id = surveys.species_id GROUP BY taxa;  
SELECT surveys.species_id, MIN(weight), MAX(weight), AVG(weight) FROM surveys  JOIN species ON surveys.species_id = species.species_id  WHERE taxa = 'Rodent'  GROUP BY surveys.species_id;  
SELECT surveys.species_id, sex, AVG(hindfoot_length) FROM surveys JOIN species ON surveys.species_id = species.species_id  WHERE (taxa = 'Rodent') AND (sex IS NOT NULL)  GROUP BY surveys.species_id, sex;  
SELECT surveys.species_id, year, AVG(weight) as mean_weight FROM surveys  JOIN species ON surveys.species_id = species.species_id  WHERE taxa = 'Rodent' GROUP BY surveys.species_id, year;
```
