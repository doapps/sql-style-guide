# SQL Style Guideline

## Use lowercase SQL

It's just as readable as uppercase SQL and you won't have to constantly be holding down a shift key.

```sql
-- Good
select * from users

-- Bad
SELECT * FROM users

-- Bad
Select * From users
```

## Single line vs multiple line queries

The only time you should place all of your SQL on a single line is when you're selecting one thing and there's no additional complexity in the query:

```sql
-- Good
select * from users

-- Good
select id from users

-- Good
select count(*) from users
```

Once you start adding more columns or more complexity, the query becomes easier to read if it's spread out on multiple lines:

```sql
-- Good
select
    id,
    email,
    created_at
from users

-- Good
select *
from users
where email = 'example@domain.com'

-- Good
select
    user_id,
    count(*) as total_charges
from charges
group by user_id

-- Bad
select id, email, created_at
from users

-- Bad
select id,
    email
from users
```

## Left align SQL keywords

Some IDEs have the ability to automatically format SQL so that the spaces after the SQL keywords are vertically aligned. This is cumbersome to do by hand (and in my opinion harder to read anyway) so I recommend just left aligning all of the keywords:

```sql
-- Good
select 
    id,
    email
from users
where email like '%@gmail.com'

-- Bad
select id, email
  from users
 where email like '%@gmail.com'
```

## Use single quotes

Some SQL dialects like BigQuery support using double quotes, but for most dialects double quotes will wind up referring to column names. For that reason, single quotes are preferable:

```sql
-- Good
select *
from users
where email = 'example@domain.com'

-- Bad
select *
from users
where email = "example@domain.com"
```

### Use `!=` over `<>`

Simply because `!=` reads like "not equal" which is closer to how we'd say it out loud.

```sql
-- Good
select count(*) as paying_users_count
from users
where plan_name != 'free'
```

## Commas should be at the the end of lines

```sql
-- Good
select
    id,
    email
from users

-- Bad
select
    id
    , email
from users
```

## Indenting where conditions

When there's only one where condition, leave it on the same line as `where`:

```sql
-- Good
select email
from users
where id = 1234

-- Bad
select email
from users
where
    id = 1234
```

When there are multiple, indent each one one level deeper than the `where`. Put logical operators at the end of the previous condition:

```sql
-- Good
select
    id,
    email
from users
where 
    created_at >= '2019-03-01' and 
    vertical = 'work'
    
-- Bad
select
    id,
    email
from users
where created_at >= '2019-03-01'
    and vertical = 'work'
```

## Avoid spaces inside of parenthesis

```sql
-- Good
select *
from users
where id in (1, 2)

-- Bad
select *
from users
where id in ( 1, 2 )
```

## Break long lists of `in` values into multiple indented lines

```sql
-- Good
select *
from users
where email in (
    'user-1@example.com',
    'user-2@example.com',
    'user-3@example.com',
    'user-4@example.com'
)
```

## Table names should be a plural snake case of the noun

```sql
-- Good
select * from users
select * from visit_logs

-- Bad
select * from user
select * from visitLog
```

## Column names should be snake_case

```sql
-- Good
select
    id,
    email,
    timestamp_trunc(created_at, month) as signup_month
from users

-- Bad
select
    id,
    email,
    timestamp_trunc(created_at, month) as SignupMonth
from users
```

## Column name conventions

* Boolean fields should be prefixed with `is_`, `has_`, or `does_`. For example, `is_customer`, `has_unsubscribed`, etc.
* Date-only fields should be suffixed with `_date`. For example, `report_date`.
* Date+time fields should be suffixed with `_at`. For example, `created_at`, `posted_at`, etc.

## Column order conventions

Put the primary key first, followed by foreign keys, then by all other columns. If the table has any system columns (`created_at`, `updated_at`, `is_deleted`, etc.), put those last.

```sql
-- Good
select
    id,
    name,
    created_at
from users

-- Bad
select
    created_at,
    name,
    id,
from users
```

## Include `inner` for inner joins

Better to be explicit so that the join type is crystal clear:

```sql
-- Good
select
    users.email,
    sum(charges.amount) as total_revenue
from users
inner join charges on users.id = charges.user_id

-- Bad
select
    users.email,
    sum(charges.amount) as total_revenue
from users
join charges on users.id = charges.user_id
```

## For join conditions, put the table that was referenced first immediately after the `on`

By doing it this way it makes it easier to determine if your join is going to cause the results to fan out:

```sql
-- Good
select
    ...
from users
left join charges on users.id = charges.user_id
-- primary_key = foreign_key --> one-to-many --> fanout
  
select
    ...
from charges
left join users on charges.user_id = users.id
-- foreign_key = primary_key --> many-to-one --> no fanout

-- Bad
select
    ...
from users
left join charges on charges.user_id = users.id
```

## Single join conditions should be on the same line as the join

```sql
-- Good
select
    users.email,
    sum(charges.amount) as total_revenue
from users
inner join charges on users.id = charges.user_id
group by email

-- Bad
select
    users.email,
    sum(charges.amount) as total_revenue
from users
inner join charges
on users.id = charges.user_id
group by email
```

When you have mutliple join conditions, place each one on their own indented line:

```sql
-- Good
select
    users.email,
    sum(charges.amount) as total_revenue
from users
inner join charges on 
    users.id = charges.user_id and
    refunded = false
group by email
```

## Avoid aliasing table names most of the time

It can be tempting to abbreviate table names like `users` to `u` and `charges` to `c`, but it winds up making the SQL less readable:

```sql
-- Good
select
    users.email,
    sum(charges.amount) as total_revenue
from users
inner join charges on users.id = charges.user_id

-- Bad
select
    u.email,
    sum(c.amount) as total_revenue
from users u
inner join charges c on u.id = c.user_id
```

Most of the time you'll want to type out the full table name.

There are two exceptions:

If you you need to join to a table more than once in the same query and need to distinguish each version of it, aliases are necessary.

Also, if you're working with long or ambiguous table names, it can be useful to alias them (but still use meaningful names):

```sql
-- Good: Meaningful table aliases
select
  companies.com_name,
  beacons.created_at
from stg_mysql_helpscout__helpscout_companies companies
inner join stg_mysql_helpscout__helpscout_beacons_v2 beacons on companies.com_id = beacons.com_id

-- OK: No table aliases
select
  stg_mysql_helpscout__helpscout_companies.com_name,
  stg_mysql_helpscout__helpscout_beacons_v2.created_at
from stg_mysql_helpscout__helpscout_companies
inner join stg_mysql_helpscout__helpscout_beacons_v2 on stg_mysql_helpscout__helpscout_companies.com_id = stg_mysql_helpscout__helpscout_beacons_v2.com_id

-- Bad: Unclear table aliases
select
  c.com_name,
  b.created_at
from stg_mysql_helpscout__helpscout_companies c
inner join stg_mysql_helpscout__helpscout_beacons_v2 b on c.com_id = b.com_id
```

## Include the table when there is a join, but omit it otherwise

When there are no join involved, there's no ambiguity around which table the columns came from so you can leave the table name out:

```sql
-- Good
select
    id,
    name
from companies

-- Bad
select
    companies.id,
    companies.name
from companies
```

But when there are joins involved, it's better to be explicit so it's clear where the columns originated:

```sql
-- Good
select
    users.email,
    sum(charges.amount) as total_revenue
from users
inner join charges on users.id = charges.user_id

-- Bad
select
    email,
    sum(amount) as total_revenue
from users
inner join charges on users.id = charges.user_id

```

## Always rename aggregates and function-wrapped arguments

```sql
-- Good
select count(*) as total_users
from users

-- Bad
select count(*)
from users

-- Good
select timestamp_millis(property_beacon_interest) as expressed_interest_at
from hubspot.contact
where property_beacon_interest is not null

-- Bad
select timestamp_millis(property_beacon_interest)
from hubspot.contact
where property_beacon_interest is not null
```

## Be explicit in boolean conditions

```sql
-- Good
select * from customers where is_cancelled = true
select * from customers where is_cancelled = false

-- Bad
select * from customers where is_cancelled
select * from customers where not is_cancelled
```

## Use `as` to alias column names

```sql
-- Good
select
    id,
    email,
    timestamp_trunc(created_at, month) as signup_month
from users

-- Bad
select
    id,
    email,
    timestamp_trunc(created_at, month) signup_month
from users
```

## Group using column names or numbers, but not both

I prefer grouping by name, but grouping by numbers is [also fine](https://blog.getdbt.com/write-better-sql-a-defense-of-group-by-1/).

```sql
-- Good
select user_id, count(*) as total_charges
from charges
group by user_id

-- Good
select user_id, count(*) as total_charges
from charges
group by 1

-- Bad
select
    timestamp_trunc(created_at, month) as signup_month,
    vertical,
    count(*) as users_count
from users
group by 1, vertical
```

## Take advantage of lateral column aliasing when grouping by name

```sql
-- Good
select
  timestamp_trunc(com_created_at, year) as signup_year,
  count(*) as total_companies
from companies
group by signup_year

-- Bad
select
  timestamp_trunc(com_created_at, year) as signup_year,
  count(*) as total_companies
from companies
group by timestamp_trunc(com_created_at, year)
```

## Grouping columns should go first

```sql
-- Good
select
  timestamp_trunc(com_created_at, year) as signup_year,
  count(*) as total_companies
from companies
group by signup_year

-- Bad
select
  count(*) as total_companies,
  timestamp_trunc(com_created_at, year) as signup_year
from mysql_helpscout.helpscout_companies
group by signup_year
```

## Aligning case/when statements

Each `when` should be on its own line (nothing on the `case` line) and should be indented one level deeper than the `case` line. The `then` can be on the same line or on its own line below it, just aim to be consistent.

```sql
-- Good
select
    case
        when event_name = 'viewed_homepage' then 'Homepage'
        when event_name = 'viewed_editor' then 'Editor'
        else 'Other'
    end as page_name
from events

-- Good too
select
    case
        when event_name = 'viewed_homepage'
            then 'Homepage'
        when event_name = 'viewed_editor'
            then 'Editor'
        else 'Other'            
    end as page_name
from events

-- Bad 
select
    case when event_name = 'viewed_homepage' then 'Homepage'
        when event_name = 'viewed_editor' then 'Editor'
        else 'Other'        
    end as page_name
from events
```



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


## Stored procedures
The stored procedure names need to start with the prefix 'sp_', must be in lower case and contain a verb.

```sql
CREATE PROCEDURE sp_get_movies()
BEGIN
    select title,description,release_year,rating from movies;
END
```

## Triggers
The trigger name need to start with the prefix 'tr_', must be in lower case and contain the table name with the action name.

```sql
CREATE TRIGGER tr_payrolls_after_insert AFTER INSERT ON payrolls
       FOR EACH ROW SET @sum = @sum + NEW.amount;
```

# MongoDB Style Guideline

This is a small section where you can see the mongodb style guides which you can follow to implement your databases.



## Conventions to cover

- Database
- Tables
- Columns
- Functions

  
## Database
Database names must consist of the letter a to z(letters in lower case),the numbers 0 to 9 and the underscore(_) or dash(-) symbols. In addition to this, the database 
names always need to start with a lettter.


```sql
use my_database
```


## Tables
The table name need to be in plural form and not add a prefix like 'tbl' or any other such a descriptive prefix. Avoid as posible concatenating two tables names together to create the name of a relationship
table.

```mongodb
db.createCollection( services,
   {
     ....
   }
)
```

## Columns
The column names always need to be in singular name and in lower case, where possible avoid simply using id ad the primary identifier for the table. In addition to this, tables must have at least one key to be complete and useful.

```mongodb
db.createCollection( services,
   {
     name: "maintenance"
   }
)
```


## Functions

```mongodb
{
  $function: {
    body: function(name, scores) {
            let total = Array.sum(scores);
            return `Hello ${name}.  Your total score is ${total}.`
    },
    args: [ "$name", "$scores"],
    lang: "js"
  }
}

```

  
