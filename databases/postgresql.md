# PostgreSQL

An object-relational database system

## I. Installation
#### A. Mac OSX:
 
  brew install postgresql
  
#### B. Ubuntu
  sudo apt-get update
  sudo apt-get install postgresql postgresql-contrib
 
## II. Console Commands

#### A. Connecting to PostgreSQL Server

To connect to the PostgreSQL server with as user `postgres`:

    psql -U postgres
    
By default, psql connects to a PostgreSQL server running on localhost at port 5432. To connect to a different port and/or host. Add the -p and -h tag:

    psql -U postgres -p 12345 -h 192.32.123.32
    
Once in, you may navigate via the following commands:
  
* \l - list databases
* \c - change databases
* \d - list tables
* \df - list functions
* \df - list functions with definitions
* \q - quit

## III. Database Creation

    CREATE DATABASE < database name >;
    
    # Creates database with name: test_db
    CREATE DATABASE test_db

## IV. Database Drop

    DROP DATABASE < database name >;
    
     # Drops database with name: test_db
    DROP DATABASE test_db

## V. Table Creation
   
    CREATE TABLE programs(
      programid SERIAL PRIMARY KEY,
      degree CHARACTER VARYING,
      program CHARACTER VARYING
    );
    
    CREATE TABLE students(
      studentid SERIAL PRIMARY KEY,
      student_number CHARACTER VARYING UNIQUE,
      first_name CHARACTER VARYING,
      last_name CHARACTER VARYING,
      programid INTEGER REFERENCES programs,
      insertedon TIMESTAMP WITHOUT TIME ZONE DEFAULT now()
    );

#### A. Column Data Types

* SERIAL
* CHARACTER VARYING
* CHARACTER(10)
* INTEGER
* TIMESTAMP WITHOUT TIME ZONE


#### B. Common Added Options

* PRIMARY KEY
* UNIQUE
* DEFAULT



## VI. CRUD Operations

### A. Insertion of Rows

** Template: **

  INSERT INTO table_name(column1, column2, column3...)
  VALUES(value1, value2, value3...);

** Sample: **

  INSERT INTO programs(degree, program)
  VALUES('BS', 'Computer Science');
  
  INSERT INTO programs(degree, program)
  VALUES('BS', 'Business Administration and Accountancy');

    INSERT INTO students(student_number, first_name, last_name, programid)
    VALUES('2010-00031', 'Juan', 'Cruz', 1);

    INSERT INTO students(student_number, first_name, last_name, programid)
    VALUES('2010-00032', 'Pedro', 'Santos', 2);
    
### B. Read/Lookup of Row

#### i. Get All Rows
    
    SELECT * FROM students;

#### ii. Get Rows Satisfying Certain Conditions

  # Gets row/s with studentid = 1
  
    SELECT * FROM students where studentid = 1;

  # Gets row/s where the last_name starts with 'cru' (non case sensitive)
  
    SELECT * FROM students where last_name ilike 'cru%';

  # Gets row/s where the student_number column is either 2010-0033, '2010-30011', or '2010-18415'
  
    SELECT * FROM students where student_number in ('2010-00033', '2010-30011', '2010-18415');
    
    
#### iii. Get Specific Columns from Resulting Rows

     # Selects the lastname and firstname from the students table
  
     SELECT last_name, firstname from students;

     # Selects the program column from rows of the programs table satisfying the condition and then prepending the given string
  
      SELECT 'BUSINESS PROGRAM: ' || program from programs where program ilike '%business%';


### C. Update of Row

#### i. Update all Rows

    UPDATE students SET last_name = 'Cruz';

#### ii. Update Rows Satisfying Conditions

    UPDATE students SET last_name = 'Santos' where studentid = 1;

    UPDATE programs SET degree = 'BA' where programid NOT IN (2);


### D. Deletion of Row

#### i. Delete all Rows

     DELETE FROM students

#### ii. Delete Rows Satisfying Conditions

    DELETE FROM students WHERE studentid NOT IN (1,2)

## VII. Queries

### A. Joins

#### i. Inner Join

##### Syntax:

     SELECT * FROM table_1 JOIN table_2 using (common_column_name);

##### Example:

     SELECT student_number, program FROM students JOIN programs using (programid);

#### ii. Left Join

##### Syntax:

     SELECT * FROM table_1 LEFT JOIN table_2 on table_1.column_name = table_2.column_name;

##### Example:

We insert a student row without a program
  
     INSERT INTO students(student_number, first_name, last_name)
     VALUES('2010-35007', 'Juana', 'Change');
  
Doing a left join would still return the recently inserted row but with empty Programs-related fields.
  
     SELECT * FROM students LEFT join programs on students.programid = programs.programid;
  

#### iii. Right Join

##### Syntax:

     SELECT * FROM table_1 RIGHT JOIN table_2 on table_1.column_name = table_2.column_name;

##### Example:

We insert a program row without any students attached
  
     INSERT INTO programs(degree, program)
     VALUES('BS', 'Information Technology');
  
Doing a right join would still return the recently inserted row but with empty Students-related fields.
  
     SELECT * FROM students RIGHT join programs on students.programid = programs.programid;


### B.Where

Specify conditions by which rows from the query will be filtered.

     SELECT * from students where programid IS NOT NULL;

### C. Group By

Allows use of aggregate functions with the attributes provided to the GROUP BY clause as basis for aggregations

     SELECT program, COUNT(*) FROM students
     JOIN programs USING (programid) GROUP BY program;
  
Above example counts students per program.

### D. Having

Similar to WHERE but applies the condition to the groups produced with GROUP BY.

     SELECT program, COUNT(*) FROM students
     JOIN programs USING (programid) GROUP BY program HAVING COUNT(*) > 1;

### E. Union

Joins resulting datasets from multiple queries.

     select * from students where programid in (1, 2)

     UNION

     select * from students;