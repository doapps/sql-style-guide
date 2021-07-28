
# MySql Style Guideline

This is a small section where you can see the mysql style guides which you can follow to implement your databases.



## Conventions to cover

- Database
- Tables
- Columns
- Foreign key
- Functions

  
## Database
Database names must consist of the letter a to z(letters in lower case),the numbers 0 to 9 and the underscore(_) or dash(-) symbols. In addition to this, the database 
names always need to start with a lettter.


```sql
use my_database;
```


## Tables
The table name need to be in plural form and not add a prefix like 'tbl' or any other such a descriptive prefix. Avoid as posible concatenating two tables names together to create the name of a relationship
table.

```sql
--Bad
CREATE TABLE cars_mechanics;

--Good
CREATE TABLE services;
```

## Columns
The column names always need to be in singular name and in lower case, where possible avoid simply using id ad the primary identifier for the table. In addition to this, tables must have at least one key to be complete and useful.

```sql
CREATE TABLE students{
    student_id int,
    name varchar(2555),
    last_name varchar(255)
};
```

## Foreign key
The foreign keys need to be in singular form and lower case, also must start with the prefix 'fk_'.

```sql
CREATE TABLE cars{
    car_id int,
    name varchar(255),
    FOREIGN KEY (fk_company_id) REFERENCES companies(company_id)
};
```

## Functions
The function names need to start with the prefix 'fn_', and must be in lower case.

```sql
CREATE FUNCTION fn_cal_income()
RETURNS INT 

BEGIN
  DECLARE income INT;
  SET income = 930;

  RETURN income;

END;

```


  
