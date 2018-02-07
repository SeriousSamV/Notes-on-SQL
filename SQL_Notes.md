# Notes on SQL - MS SQL Server

> by _**Samuel0Paul**_ - [paulsamuelvishesh@live.com](mailto:paulsamuelvishesh@live.com)

<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a><br />This work by <a xmlns:cc="http://creativecommons.org/ns#" href="mailto:paulsamuelvishesh@live.com" property="cc:attributionName" rel="cc:attributionURL">Samuel0Paul</a> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.

---

## Into

Categories:

* DML (Data Manupulation Language)
  * `SELECT`: Retrieves data from one or more tables
  * `INSERT`: Adds one or more new rows to a table
  * `UPDATE`: Changes one or more exisiting rows in a table
  * `DELETE`: Deletes one or more exisiting rows in a table
* DDL (Data Definintion Language): used by DBAs to manage exisiting databases
  * `CREATE DATABASE`: Creates a Database
  * `CREATE TABLE`: Creates a new Table in a Database
  * `CREATE INDEX`: Creates a new index for a Table
  * `ALTER TABLE`: Change the structure of an exisiting table
  * `ALTER INDEX`: Change the structure of an exisiting index
  * `DROP DATABASE`: Deletes an exisiting Database
  * `DROP TABLE`: Deletes an exisiting Table
  * `DROP INDEX`: Deletes an exisiing Index

### Typical Statements for working with Database Objects

E.g.

```SQL
CREATE TABLE Invoices
(
  InvoiceID       INT           NOT NULL IDENTITY PRIMARY KEY,
  VendorID        INT           NOT NULL REFERENCES Vendors(VendorID),
  InvoiceNumber   VARCHAR(50)   NOT NULL,
  InvoiceDate     SMALLDATETIME NOT NULL,
  InvoiceTotal    MONEY         NOT NULL,
  PaymentTotal    MONEY         NOT NULL DEFAULT 0,
  CreditTotal     MONEY         NOT NULL DEFAULT 0,
  TermsID         INT           NOT NULL REFERENCES Terms(TermsID),
  InvoiceDueDate  SMALLDATETIME NOT NULL,
  PaymentDate     SMALLDATETIME NULL
);
```

#### Description

* The `REFERNECES` Clause for a colunm indicates that a column contains a _FK_ (Forien Key), and it names the table and column that contains the _PK_ (Primary Key).
* If _default_ values are specified, _values_ for those fields are not needed when a new row is _inserted_.

### Querying a Single Table

* `SELECT`
  * used to query a single Table in a Database
  * SELECT statement is followed by the _the names of the Columns_ to be retrivied
* `FROM`
  * FROM clause names the Table that contains the cloumns, called the _base table_
* `WHERE`
  * WHERE clause gives the criteria for the rows to be selected (to be filtered out)
  * _conditional expressions_ follow the WHERE clause

```SQL
SELECT InvoiceNumber, InvoiceDate, InvoiceTotal,
  PaymentTotal, CreditTotal,
  InvoiceTotal - PaymentTotal - CreditTotal AS BalanceDue
FROM Invoices
WHERE InvoiceTotal - PaymentTotal - CreditTotal > 0
ORDER BY InvoiceDate;
```

### `Join`ing Data from two or more Tables

#### `INNER JOIN`

* When we use the _inner join_, _rows_ from the _two tables_ in the Join are inscluded in the result table **only if** thier _related columns_ match.
* These '_matching columns_' are specified in the `FROM` clause of the `SELECT` statement.
* `OUTER JOIN` returns rows from one table in the join even if the other table doesn't contain a matching row.

```SQL
-- selects all rows from
--    `Vendors` and
--    `Invoices` (where Vendors.VendorID = Invoices.VendorID) and
--    `InvoiceLineItems` (where Invoices.InvoiceID = InvoiceLineItems.InvoiceID)

SELECT * FROM Vendors
  INNER JOIN Invoices
    ON Vendors.VendorID = Invoices.VendorID
  INNER JOIN InvoiceLineItems
    ON Invoices.InvoiceID = InvoiceLineItems.InvoiceID;
```

```SQL
-- selects data from Vendor and Invoices table
-- i.e. VendorName from `Vendor` and
--      InvoiceNumber, InvoiceDate, InvoiceTotal from `Invoices`
SELECT VendorName, InvoiceNumber, InvoiceDate, InvoiceTotal
FROM Vendors
  INNER JOIN Invoices
    ON Vendors.VendorID = Invoices.VendorID
WHERE InvoiceTotal >= 500
ORDER BY VendorName, InvoiceTotal DESC;
```

### Add, Update and Delete data in a Table

* use `INSERT` statement to _add_ rows to a table
* use `UPDATE` statement to _change_ the values of one or more rows of the table based on a condition you specify
* use `DELETE` statement to delete one or more rows from a table based on condition
* These statements are refered to as _action queries_

```SQL
-- adds a row to the `Invoices` table
INSERT INTO Invoices (VendorID, InvoiceNumber,
  InvoiceDate, InvoiceTotal, TermsID, InvoiceDueDate)
VALUES (12, '464262344', '4/24/2012', 156, 3, '6/19/2017');

-- change value of CreditTotal col for a selected row
UPDATE Invoices
SET CreditTotal = 25.55
WHERE InvoiceNumber = '464262344';

-- change values in the InvoiceDueDate col for all invoices with the specified TermsID
UPDATE Invoices
SET InvoiceDueDate = InvoiceDueDate + 30
WHERE TermsID = 4;

-- delete a selected record
DELETE FROM Invoices
WHERE InvoiceNumber = '464262344';

-- delete all paid invoices from the table
DELETE FROM Invoices
WHERE InvoiceTotal - PaymentTotal - CreditTotal = 0;
```

### Working with _Views_

* A _View_ is a predefined query that is stored in the database
* Once created, a View can be referred to, instead of a table in most SQL statements
* View behaves like a virtual table, a `viewed table`

```SQL
-- create a View
CREATE VIEW VendorsMin AS
  SELECT VendorName, VendorState, VendorPhone
  FROM Vendors;

-- AFTER VIEW CREATING, it can be _used_ in a SQL statement
SELECT * FROM VendorsMin
WHERE VendorState = 'CA'
ORDER BY VendorName;
```

### Working with _Stored Procedures_, _Triggers_, and _User-Defined Functions_

* _Stored Procedures_ are a set of one or more SQL statements that are stored in the Datebase
* _Triggers_ are _Stored Procedures_ which gets executed automatically when some action like _INSERT_, _UPDATE_, OR _DELETE_ operation (any DLL statement) is executed on the table

```SQL
-- create a Stored Procedure
CREATE PROCEDURE spVendorsByState @StateVar char(2) AS
  SELECT VendorName, VendorState, VendorPhone
  FROM Vendors
  WHERE VendorState = @StateVar
  ORDER BY VendorName;

-- execute the created statement
EXEC spVendorsByState 'CA';
```

---

## A litle bit about _SQL Server 2016_

* you've to attach your database file to the database engine in order to start working on your database
* you can take a _Backup_ of your working Database and _Restore_ it later
* you can use the query designer if you're incompetent in writing SQL queries or if you're just a lazy bum

---

## The Ways of the Jedi in retrieving data from a Single Table

### Basic Syntax of the `SELECT` Statement

```SQL
SELECT select_list
[FROM table_src]
[WHERE search_condition]
[ORDER BY orderby_list]
```

### A more detailed look at the `SELECT` Clause

```SQL
SELECT [ALL|DISTINCT] [TOP n [PERCENT] [WITH TIES]]
  column_specification [[AS] result_col]
  [, column_specification [[AS] result_col]] ...
```

#### Column specification can by

* All columns: `SELECT *`
* a Column Name: `SELECT VendorName, VnedorCity`
* Result of a Calculation (Arithmetic expr): `SELECT InvoiceTotal - PaymentTotal`
* Result of a Concatenation (String expr): `SELEC FName + ' ' + LName AS [Full Name]`
* Result of a Function: `SELECT GETDATE() AS [Current Date]`

### String Expressions

```SQL
-- concatenate string data
SELECT VendorName,
  VendorCity + ', ' + VendorState + ' ' + VendorZipCode AS [Address]
FROM Vendors;

-- using CONCAT function to achieve the same
SELECT VendorName,
  CONCAT(VendorCity, ', ', VendorState, ' ', VendorZipCode) AS [Address]
FROM Vendors;
```

### Arithmetic Expressions

| Syntax | Arithmetic Operation |
| ------ | -------------------- |
| `*`    | Multiplication       |
| `/`    | Division             |
| `%`    | Modulo, Reminder op  |
| `+`    | Addition             |
| `-`    | Subtraction          |

```SQL
SELECT InvoiceID,
  (InvoiceID + 7) * 3 AS [Some Bullshit] -- Arithmetic Expression
FROM Invoices
ORDER BY InvoiceID;
```

### Using `DISTINCT` to eliminate duplicate rows

* used to eliminate duplicate rows (applies to all columns being fetched)
* automatically _Sorts_ the result set by it's first Column as a consequence

```SQL
SELECT DISTINCT VendorCity, VendorState
FROM Vendors;
```

### Using `TOP` clause

* use `TOP` clause within `SELECT` caluse to limit the number of rows included in the result set
* if `PERCENT` included, the first _n_ percent of the selected rows are included in the result set
* if `WITH TIES` is included, additional rows will be included if their values match or _tie_, the values of the last row

```SQL
SELECT TOP 5 VendorID, InvoiceTotal
FROM Invoices
ORDER BY InvoiceTotal DESC;

SELECT TOP 5 PERCENT VendorID, InvoiceTotal
FROM Invoices
ORDER BY InvoiceTotal DESC;

-- `WITH TIES` indicates that, if the last row (fifth) is equal with the row next to it (sixth)
--    the extra row that matches (is in 'tie' with the last one) will also be included
--  That's why running this query results in 6 rows rather than 5,
--    cuz the last two rows are at a tie
SELECT TOP 5 WITH TIES VendorID, InvoiceTotal
FROM Invoices
ORDER BY InvoiceDate ASC;
```

### Using the `WHERE` Clause

#### Syntax

```SQL
WHERE expr_1 operator expr_2 [AND|OR] ...
```

* We can use a comparison operator to compare any two expressions that result in the **same data types** (unlike data types should be converted to data types matches in order to compare)
* if _comparision_ is `true`, then the row being tested is included in the result set, if `false` then it is not
* out SQL Server is **not case-sensitive**, beware

#### Comparision Operators

| Syntax | What it means lol        | Usage                                         |
| ------ | ------------------------ | --------------------------------------------- |
| `=`    | equal to                 | `WHERE VendorState = 'IA'`                    |
| `>`    | Greater than             | `WHERE InvoiceTotal - PaymentTotal > 0`       |
| `<`    | Less than                | `WHERE VendorName < 'M'`                      |
| `<=`   | Less than or Equal to    | `WHERE InvoiceDate <= '2014-06-30'`           |
| `>=`   | Greater than or Equal to | `WHERE InvoiceDate >= '2014-06-30'`           |
| `<>`   | Not Equal                | `WHERE CreditTotal <> 0 -- not equal to zero` |

### Using `AND`, `OR`, and `NOT`: Logical Operators

#### Syntax

```SQL
WHERE [NOT] serach_condition_1 {AND|OR} [NOT] search_condition_2 ...

-- example
SELECT * FROM Invoices
WHERE InvoiceTotal > 500
  OR InvoiceDate > '05/06/2015'
  AND NOT InvoiceTotal - PaymentTotal - CreditTotal > 0;
```

### Using the `IN` Operator

#### Syntax

```SQL
WHERE test_expr [NOT] IN ({subquery|expression_1 [, expression_2]...})

-- examples

-- IN phrase with a list of numeric values
SELECT * FROM Invoices
  WHERE TermsID IN (1, 2, 5);

-- IN phrase with a subquery
SELECT * FROM Vendors
WHERE VendorID IN
  (SELECT VendorID
   FROM Invoices
   WHERE InvoiceDate = '2015-12-24'); -- YYYY-mm-dd
```

* We can use IN phrase to test whether an expr is equal to a value in a list of expr_s
* The list of expressions can be coded in any order and it doesn't affect the order of the result set
* Can also compare the test\*expr to the items in a list returned by a **\*subquery**\_ as illustrated in the examples above

### Using the `BETWEEN` operator

#### Syntax

```SQL
WHERE test_expr [NOT] BETWEEN begin_expr AND end_expr

-- examples
SELECT * FROM Invoices
  WHERE InvoiceDate BETWEEN '2016-04-01' AND '2016-05-31';
```

* to test whether an expression falls withing a range of values. **(BETWEEN LOW AND HIGH)**
* the result set will include values that are equal to the upper and lower limit **It's inclusive**

### Using the 'LIKE' Operator

#### Syntax

```SQL
WHERE match_expr [NOT] LIKE pattern

-- examples
SELECT * FROM Vendors
  WHERE VendorCity LIKE 'SAN%';

SELECT * FROM Vendors
  WHERE VendorCity LIKE 'DAMI[EO]N'; -- matches 'Damien' and 'Damion'

SELECT * FROM Vendors
  -- NC, NJ but not NV or NY, i.e. N[everything but not any K-Y]
  WHERE VendorState LIKE 'N[^K-Y]';
```

#### Wildcard Symbols _used in pattern_

| Symbol  | Description                                           | Usage                                           |
| ------- | ----------------------------------------------------- | ----------------------------------------------- |
| `%`     | Matches any string of zero or more characters         | `S%` matches 'Sam' as well as 'Samuel'          |
| `_`     | (Underscore) Matches any single character             | `S_M` matches 'Sam', 'Sim', 'Som',..            |
| `[]`    | Matches a single character listed within the brackets | `S[AI]M` matches 'Sam' and 'Sim'                |
| `[ - ]` | Matches a single character withing the given rangle   | `S[A-D]M` matches 'Sam', 'Sbm', 'Scm' and 'SDM' |
| `[ ^ ]` | Matches a single character not listed after the caret | `S[^b-y]M` matches 'Sam' and 'Szm' only         |

### Using the `IS NULL` Clause

#### Syntax

```SQL
WHERE expression IS [NOT] NULL

-- example
SELECT * FROM Invoices
  WHERE InvoiceTotal IS NULL;
```

### Using the `ORDER BY` Clause

#### Syntax

```SQL
ORDER BY epression [ASC|DESC] [, expression [ASC|DESC]] ...

--example
-- First, the results are orders ASC wrt VendorState
-- Then the sorted result if futher sorted ASC wrt VendorCity
-- finally result is sorted DESC wrt VendorName
SELECT * FROM Vendors
  ORDER BY VendorState, VendorCity, VendorName DESC;
```

#### Using _Alias_, _Expression_ or a _Column number_ to Sort the result set

```SQL
-- ORDER BY Clause using an alias
-- ------------------------------
SELECT VendorName,
  VendorCity + ', ' + 'VendorState' + ' ' + VendorZipCode AS Address
FROM Vendors
ORDER BY Address, VendorName; -- Address is the Alias used in the SELECT clause

-- ORDER BY Clause using an expression
-- -----------------------------------
SELECT * FROM Vendors
  ORDER BY VendorsContactLName + VendorContactFName;

-- ORDER BY Clause using column position number
-- --------------------------------------------
SELECT VendorName,
  VendorCity + ', ' + 'VendorState' + ' ' + VendorZipCode AS Address
FROM Vendors
ORDER BY 2, 1; -- 2: Address and 1: VendorName
```

#### Retrieving a Range of Selected Rows

```SQL
ORDER BY order_by_list
  OFFSET offset_row_count {ROW|ROWS}
  [FETCH {FIRST|NEXT} fetch_row_count {ROW|ROWS} ONLY]

-- examples

-- retriving the first five rows
SELECT * FROM Invoices
ORDER BY InvoiceTotal DESC
  OFFSET 0 ROWS
  FETCH FIRST 5 ROWS ONLY; -- can use `FIRST` and `NEXT` interchangably

-- retrieving rows 11 to 20
SELECT * FROM Vendors
WHERE VendorState = 'CA'
ORDER BY VendorCity
  OFFSET 10 ROWS
  FETCH NEXT 10 ROWS ONLY;

-- skip first 10 rows and fetch the rest
SELECT * FROM Vendors
WHERE VendorState = 'CA'
ORDER BY VendorCity
  OFFSET 10 ROWS;
```

* Useful in _Pagination_ Scenarios in the Client-Side
* `OFFSET` Clause specifies the number of rows that _should be skipped_ before rows are returned from the result set
* `FETCH` Clause specifies the number of rows that should be _retrieved_ after skipping the specifies number of rows. _If you omit the FETCH Clause, all of the rows to the end of the result set are retrieved_
* The number of rows to be skipped and retrieved can be specified as an interger or an expression that results in one that is greater than or equal to zero

> #### NOTE
>
> ##### Pagination for client Application
>
> Because a new result set is created each time a query is executed, the client application must make sure that the result set doesn't change between queries. E.g. if after getting first 20 rows, the fith row is deleted, gettting the next 20 rows will skip one row (row #21 intially is now row #20 and thus skipped).
> To avoid this kind of Scenario, the Client application can execute all of the queries within a transaction whose _isolation level_ is set to either `SNAPSHOT` or `SERIALIZABLE`.

---

## The Dark Arts of Retrieving Data from multiple Tables

### Working with `INNER JOINS`

* A _Join_ lets you combine columns from two or more tables into a single result set
* To Join data from two tables, we should code the names of the two tables in the `FROM` Clause along with the `JOIN` keyword and an `ON` Phrase that specifies the _Join Condition_
* The _Join Condition_ indicates how the two tables should be compared

#### Syntax

```SQL
SELECT select_list
FROM table_1
  [INNER] JOIN table_2
    ON join_condition_1
  [[INNER] JOIN table_3
    ON join_condition_2] ...

-- example

-- InvoiceNumber is from table Invoices and VendorName is from table Vendors
SELECT InvoiceNumber, VendorName
FROM Vendors
  JOIN Invoices
    ON Vendors.VendorID = Invoices.VendorID;
```

#### Using a correlation name (_table alias_)

```SQL
SELECT select_list
FROM table_1 [AS] n1
  [INNER] JOIN table_2 [AS] n2
    ON n1.columnName operator n2.columnName
  [[INNER] JOIN table_3 [AS] n3
    ON n2.columnName operator n3.columnName] ...

-- example
SELECT * FROM Vendors AS v JOIN Invoices AS i
  ON v.VendorID = i.VendorID
  -- 'table alias' is optional if there is no columnName collision
WHERE InvoiceTotal - PaymentTotal - CreditTotal > 0
ORDER BY InvoiceDueDate DESC;
```

### Working with Tables from different Databases

* Just use a _Fully-Qualified Object Name_ when working with multiple Databases and Tables

> A _Fully-Qualified Object Name_ consists of four parts:
>
> 1. A Server Name (can omit as long as you query from only the server which you currently use)
> 1. A Database Name (can omit as long as you have already run the `USE DATABASE;` statement)
> 1. A Schema Name (can omit as long as we work with tables in the user's _default_ schema)
> 1. The Name of the Object itself

```SQL
GO -- to add linked server "DBServer"
USE master;
EXEC sp_addlinkedserver
  @server = 'DBServer',
  @srvproduct = '',
  @provider = 'SQLNCLI',
  @datasrc = 'DESKTOP-V1L8LOR'; -- use your own (e.g. 'localhost/instance-name')
GO -- end

SELECT *
FROM DBServer.AP.dbo.Vendors AS Vendors
  JOIN DBServer.ProductOrders.dbo.Customers AS Customers
    ON Vendors.VendorZipCode = Customers.CustZip
ORDER BY VendorState, VendorCity;

GO -- to remove linked server "DBServer"
EXEC sp_dropserver
  USE master;
  @server = 'DBServer';
GO -- end
```

#### `INNER JOIN`s that Join more that two tables

Think of _multi-table Joins_ as a _series of two-table Joins_ proceeding from _left to right_

i.e.

> `FROM TABLE_1 JOIN A ON {} JOIN B ON {} JOIN C ON {} JOIN D ON {}` <br/>
> Should be treated as <br/>
 > `((((TABLE_1 + A) + B) + C) + D)`

```SQL
-- getting data from 4 tables: Vendors, Invoices, InvoiceLineItems and GLAccounts

SELECT VendorName, InvoiceNumber, InvoiceDate, AccountDescription
FROM Vendors
  JOIN Invoice ON Vendors.VendorID = Invoice.VendorID
  JOIN InvoiceLineItems
    ON Invoices.InvoiceID = InvoiceLineItems.InvoiceID
  JOIN GLAccounts ON InvoiceLineItems.AccountNo = GLAccounts.AccountNo
WHERE InvoiceTotal - PaymentTotal - CreditTotal > 0
ORDER BY VendorName, LineItemAmount DESC;
```

#### We can also use `INNER JOIN` Implicitly without using it

##### Syntax

```SQL
SELECT select_list
FROM table_1, table_2 [, table_3]...

-- example

SELECT InvoiceNumber, Vendorname
FROM Vendors, Invoices
WHERE Vendors.VendorID = Invoices.VendorID;

-- the above is same as

SELECT InvoiceNumber, Vendorname
FROM Vendors
  INNER JOIN Invoices ON Vendors.VendorID = Invoices.VendorID;
```

### Working with `OUTER JOIN`s

* Unlike _inner_ Joins, _Outer_ Join returns all of the rows from one or both of the tables involved in the Join, regardless of whether the _Join Condition_ is true

| Syntax             | What it does                                                        |
| ------------------ | ------------------------------------------------------------------- |
| `LEFT OUTER JOIN`  | Includes all the rows from the First/Left table in the result set   |
| `RIGHT OUTER JOIN` | Includes all the rows from the Second/Right table in the result set |
| `FULL OUTER JOIN`  | Includes all the rows from both tables                              |

> Note: In outer Join, all the unmatching rows' columns will have `NULL` as it's value in the result set.

```SQL
-- Syntax
SELECT select_list
FROM table_1
  {LEFT|RIGHT|FULL} [OUTER] JOIN table_2
    ON join_condition_1
  [{LEFT|RIGHT|FULL} [OUTER] JOIN table_3
    ON join_condition_2]...

-- examples

-- left outer join
SELECT * FROM Vendors
  LEFT JOIN Invoices
    ON Vendors.VendorID = Invoices.VendorID
ORDER BY VendorName;

-- right outer join
SELECT * FROM Vendors
  RIGHT JOIN Invoices
    ON Vendors.VendorID = Invoices.VendorID
ORDER BY VendorName;

-- full outer join
SELECT * FROM Vendors
  FULL JOIN Invoices
    ON Vendors.VendorID = Invoices.VendorID
ORDER BY VendorName;
```

> Note: Outer Joins can also be used for multiple tables consecutively (works just like _multi-table inner joins_)

### Using `CROSS JOINS`

* A Cross Join produces a result set that includes each row from the first table joined with each row from the second table
* It's also called the _Cartesian Product_ Table

> NOTE: You _**can't use a Join Condition**_ in CROSS JOIN, because it's a _Cartesian Product_ table.

```SQL
-- Syntax
SELECT select_list
FROM table_1 CROSS JOIN table_2

-- example

USE Examples;
-- Departments table has 5 rows
-- Employees table has 9 rows
-- Our query returns 45 rows,
--    matching every row on the first table to all the rows in the next table, Recursively
--
SELECT Departments.DeptNo, DeptName, EmployeeID, LastName
FROM Departments CROSS JOIN Employees
ORDER BY Departments.DeptNo;
```

#### Using Cross Join _Implicitly_

```SQL
SELECT Departments.DeptNo, DeptName, EmployeeID, LastName
FROM Departments, Employees
ORDER BY Departments.DeptNo;
```

### Working with `UNION`s (Set Operator)

* Combines Rows from two or more Result Sets (on the other hand, `JOIN` combines columns from different tables for the result set)
* By Default, Union operation removes _Duplicate_ rows from the final result set (Use `ALL` to do the avoid omitting duplicates)

> NOTE: For a Union to work, both the result sets being merged should have the same number of columns and the data types of the corresponding columns also should match.

```SQL
-- Syntax
  SELECT_statement_1
UNION [ALL]
  SELECT_statement_2
[UNION [ALL]
  SELECT_statement_3]...
[ORDER BY ...]

-- Example

USE Examples;

  SELECT 'Active' [Source], InvoiceNumber, InvoiceDate, InvoiceTotal
  FROM ActiveInvoices
  WHERE InvoiceDate >= '2/1/2016'
UNION
  SELECT 'Paid' AS [Source], InvoiceNumber, InvoiceDate, InvoiceTotal
  FROM PaidInvoices
  WHERE InvoiceDate >= '2/1/2016'
ORDER BY InvoiceTotal DESC;
```

### Using the `EXCEPT` and `INTERSECT` Operators (Set Operator)

* Just like Union, `EXCEPT` and `INTERSECT` Operators works with two or more Result sets
* `EXCEPT` excludes rows from the first query of they also occur in the second query
* `INTERSECT` includes ONLY the rows that occus in both the queries

> NOTE: The number of columns and thier respective data types must be the same for both the result sets being operated on.

```SQL
-- Syntax
  SELECT_statement_1
{EXCEPT|INTERSECT}
  SELECT_statement_2
[ORDER BY order_by_list]

-- example

-- A Query that EXCLUDES rows from the first query if they also occur in the second query
  SELECT CustomerFirst, CustomerLast
  FROM Customers
EXCEPT
  SELECT FirstName, LastName
  FROM Employees
ORDER BY CustomerLast;

-- A Query that only includes rows that occur in both queries
  SELECT CustomerFirst, CustomerLast
  FROM Customer
INTERSECT
  SELECT FirstName, LastName
  FROM Employees;
```

## Working with Summary Queries (to summarize data)

### _Aggregate Functions_

| Function Syntax              | Result                                               |
| ---------------------------- | ---------------------------------------------------- |
| `AVG([ALL\|DISTINCT] expr)`   | The Average of the non-null values in the expression |
| `SUM([ALL\|DISTINCT] expr)`   | The Total of the non-null values in the expression   |
| `MIN([ALL\|DISTINCT] expr)`   | The Lowest non-null value in the expression          |
| `MAX([ALL\|DISTINCT] expr)`   | The Highest of the non-null values in the expression |
| `COUNT([ALL\|DISTINCT] expr)` | The number of non-null values in the expression      |
| `COUNT(*)`                   | The number of the rows selected by the query         |

> NOTE:
> * The Expression we specify for the `AVG` and `SUM` functions must result in a numeric value
> * The Expression for the `MIN`, `MAX`, and `COUNT` functions can result in a _numeric_, _date_, or a _string_ value

```SQL
-- examples

-- a summary query that counts unpaid invoices and calculates the total due
SELECT
  COUNT(*) [Number Of Invoices],
  SUM(InvoiceTotal - PaymentTotal - CreditTotal) [Total Due]
FROM Invoices
WHERE (InvoiceTotal - PaymentTotal - CreditTotal) > 0;

-- In a _Summary Query_, we can not include non-aggregate columns from the base table
SELECT
  COUNT(*) [Number Of Invoices],
  InvoiceID -- ERROR, can't include non-aggregate column in a Summary Query
FROM Invoices;
```

* A SELECT Statement that includes an Aggregate Function can be called as a _**Summary Query**_
* `COUNT` function doesn't ignore `NULL` values
* In a _Summary Query_, we can not include non-aggregate columns from the base table unless the column is specified in a `GROUP BY` clause or the `OVER` clause is included for each aggregate function

### Grouping and Summarizing Data using `GROUP BY` and `HAVING` Clauses

* `GROUP BY` Clause groups the rows of a result set based on one or more columns or expressions
* The Aggregate is calculated for each set of values that result from the columns named in the `GROUP BY` Clause (If we use Aggregate functions in the SELECT clause)
* If two or more columns are included in the GROUP BY Clasue, they are grouped hierarchically according to the order they've been included
* The `HAVING` Clause specifies a search condition for a group or an aggregate

```SQL
-- syntax
SELCT select_list
FROM table_src
[WHERE search_condition]
[GROUP BY group_by_list]    -- after the WHERE clause
[HAVING search_condition]   -- but before the ORDER BY clause
[ORDER BY order_by_list]

-- examples

SELECT VendorID, AVG(InvoiceTotal) [Average Inoice Amount]
FROM Invoices
GROUP BY VendorID
HAVING AVG(InvoiceTotal) > 2000
ORDER BY [Average Inoice Amount]; -- returns 8 rows

-- Groups by vendorID and counts the record in each group
-- i.e. counts the number of invoices by each particular vendor in the Invoices table
SELECT VendorID, COUNT(*) [Invoice Quantity]
FROM Invoices
GROUP BY VendorID;

-- Summary Query that calculates the no. of Invoies and Avg. Invoice Amt.
--  for Vendors in each State and City
-- i.e.
-- Joins tables Invoies and Vendors by matching VendorID
--    and then Groups the result set by VendorState and VendorCity
--    and then counts the records in it and
--      calculates the Avg of InvoiceTotal in each grouped set
SELECT VendorState, VendorCity,
  COUNT(*) [Invoice Quantity],
  AVG(InvoiceTotal) [Invoice Average]
FROM Invoices
  JOIN Vendors ON Invoices.VendorID = Vendors.VendorID
GROUP BY VendorState, VendorCity
ORDER BY VendorState, VendorCity; -- 20 rows

-- same as the last example, but only shows those groups who have at least 2 Vendors
-- i.e.
--  {SAME AS LAST QUERY} BUT
--    for each group,
--      if the no. of rows in the group is less than 2,
--        that group is not included in the result set
SELECT VendorState, VendorCity,
  COUNT(*) [Invoice Quantity],
  AVG(InvoiceTotal) [Invoice Average]
FROM Invoices
  JOIN Vendors ON Invoices.VendorID = Vendors.VendorID
GROUP BY VendorState, VendorCity
HAVING COUNT(*) >= 2
ORDER BY VendorState, VendorCity; -- 12 rows
```

#### Differences between having _search conditions_ in the `HAVING` Clause and the `WHERE` Clause

* Search condition in the `WHERE` Clause get applied _before_ the rows are grouped and aggregates are calculated
* Search condition in the `HAVING` Clause is applied _after_ the rows are grouped and aggregates are calculated

> NOTE: `WHERE` Clause can not contain Aggregate functions in it's condition expression.

```SQL
SELECT VendorName,
  COUNT(*) [Invoice Quantity],
  AVG(InvoiceTotal) [Invoice Average]
FROM Invoices
  JOIN Vendors ON Invoices.VendorID = Vendors.VendorID
GROUP BY VendorName
HAVING AVG(InvoiceTotal) > 500
ORDER BY [Invoice Quantity] DESC; -- 19 rows

SELECT VendorName,
  COUNT(*) [Invoice Quantity],
  AVG(InvoiceTotal) [Invoice Average]
FROM Invoices
  JOIN Vendors ON Invoices.VendorID = Vendors.VendorID
WHERE InvoiceTotal > 500
GROUP BY VendorName
ORDER BY [Invoice Quantity] DESC; -- 20 rows
```

## Appendix

### Index of all Keywords in SQL

* [x] `SELECT`
  * [x] `ALL` - (default) inlcudes all the results in the result set regarless of duplicates
  * [x] `DISTINCT` - Removes duplicates from the result set (compares ALL columns and Sorts the result set based on first column)
  * [x] `TOP`
    * [x] `WITH TIES`
* [x] `UNION`
* [x] `FROM`
* [x] `WHERE`
  * [x] `BETWEEN`
  * [x] `LIKE`
  * [x] `IS NULL`
* [X] `GROUP BY`
* [X] `HAVING`
* [x] `ORDER BY`
  * [x] `ASC` - (default)
  * [x] `DESC`
  * [ ] `OFFSET`
    * [ ] `FETCH`
      * [ ] `FIRST`
      * [ ] `NEXT`
      * [ ] `ROW`
      * [ ] `ROWS`
      * [ ] `ONLY`
* [x] `AS`
* [x] `DISTICT`
* [x] `CREATE`
  * [ ] `REFERENCE`
* [x] `DELETE`
* [x] `UPDATE`
* [x] `INSERT INTO`
* [x] `INNER JOIN`
  * [x] `ON`
* [x] `OUTER JOIN`
  * [x] `LEFT`
  * [x] `RIGHT`
  * [x] `FULL`
* [x] `CROSS JOIN`
* [x] `EXCEPT`
* [x] `INTERSECT`
* [x] `DROP`
  * [x] `DROP TABLE`
  * [x] `DROP COLUMN`
  * [x] `DROP INDEX`
* [x] `AND`
* [x] `OR`
* [x] `NOT`
* [x] `IN`

### Creating Example Databases to run the examples included here

#### For the `AP` Database

```SQL
USE master
GO

/****** Object:  Database AP     ******/
IF DB_ID('AP') IS NOT NULL
	DROP DATABASE AP
GO

CREATE DATABASE AP
GO

USE AP
GO

/****** Object:  Table ContactUpdates  ******/
CREATE TABLE ContactUpdates(
	VendorID int IDENTITY(1,1) NOT NULL,
	LastName varchar(50) NULL,
	FirstName varchar(50) NULL
)
GO

/****** Object:  Table GLAccounts     ******/
CREATE TABLE GLAccounts(
	AccountNo int NOT NULL,
	AccountDescription varchar(50) NOT NULL,
 CONSTRAINT PK_GLAccounts PRIMARY KEY CLUSTERED (
	AccountNo ASC
 )
)
GO

/****** Object:  Table InvoiceArchive     ******/
CREATE TABLE InvoiceArchive(
	InvoiceID int NOT NULL,
	VendorID int NOT NULL,
	InvoiceNumber varchar(50) NOT NULL,
	InvoiceDate smalldatetime NOT NULL,
	InvoiceTotal money NOT NULL,
	PaymentTotal money NOT NULL,
	CreditTotal money NOT NULL,
	TermsID int NOT NULL,
	InvoiceDueDate smalldatetime NOT NULL,
	PaymentDate smalldatetime NULL
)
GO

/****** Object:  Table InvoiceLineItems     ******/
CREATE TABLE InvoiceLineItems(
	InvoiceID int NOT NULL,
	InvoiceSequence smallint NOT NULL,
	AccountNo int NOT NULL,
	InvoiceLineItemAmount money NOT NULL,
	InvoiceLineItemDescription varchar(100) NOT NULL,
CONSTRAINT PK_InvoiceLineItems PRIMARY KEY CLUSTERED (
	InvoiceID ASC,
	InvoiceSequence ASC
 )
)
GO

/****** Object:  Table Invoices     ******/
CREATE TABLE Invoices(
	InvoiceID int IDENTITY(1,1) NOT NULL,
	VendorID int NOT NULL,
	InvoiceNumber varchar(50) NOT NULL,
	InvoiceDate smalldatetime NOT NULL,
	InvoiceTotal money NOT NULL,
	PaymentTotal money NOT NULL,
	CreditTotal money NOT NULL,
	TermsID int NOT NULL,
	InvoiceDueDate smalldatetime NOT NULL,
	PaymentDate smalldatetime NULL,
 CONSTRAINT PK_Invoices PRIMARY KEY CLUSTERED (
	InvoiceID ASC
 )
)
GO

/****** Object:  Table Terms    ******/
CREATE TABLE Terms(
	TermsID int IDENTITY(1,1) NOT NULL,
	TermsDescription varchar(50) NOT NULL,
	TermsDueDays smallint NOT NULL,
 CONSTRAINT PK_Terms PRIMARY KEY CLUSTERED (
	TermsID ASC
 )
)
GO

/****** Object:  Table Vendors    ******/
CREATE TABLE Vendors(
	VendorID int IDENTITY(1,1) NOT NULL,
	VendorName varchar(50) NOT NULL,
	VendorAddress1 varchar(50) NULL,
	VendorAddress2 varchar(50) NULL,
	VendorCity varchar(50) NOT NULL,
	VendorState char(2) NOT NULL,
	VendorZipCode varchar(20) NOT NULL,
	VendorPhone varchar(50) NULL,
	VendorContactLName varchar(50) NULL,
	VendorContactFName varchar(50) NULL,
	DefaultTermsID int NOT NULL,
	DefaultAccountNo int NOT NULL,
 CONSTRAINT PK_Vendors PRIMARY KEY CLUSTERED(
	VendorID ASC
 )
)
GO

SET IDENTITY_INSERT ContactUpdates ON
INSERT ContactUpdates (VendorID, LastName, FirstName) VALUES
(5, 'Davison', 'Michelle'),
(12, 'Mayteh', 'Kendall'),
(17, 'Onandonga', 'Bruce'),
(44, 'Antavius', 'Anthony'),
(76, 'Bradlee', 'Danny'),
(94, 'Suscipe', 'Reynaldo'),
(101, 'O''Sullivan', 'Geraldine'),
(123, 'Bucket', 'Charles')
SET IDENTITY_INSERT ContactUpdates OFF
GO

INSERT GLAccounts (AccountNo, AccountDescription) VALUES
(100, 'Cash'),
(110, 'Accounts Receivable'),
(120, 'Book Inventory'),
(150, 'Furniture'),
(160, 'Computer Equipment'),
(162, 'Capitalized Lease'),
(167, 'Software'),
(170, 'Other Equipment'),
(181, 'Book Development'),
(200, 'Accounts Payable'),
(205, 'Royalties Payable'),
(221, '401K Employee Contributions'),
(230, 'Sales Taxes Payable'),
(234, 'Medicare Taxes Payable'),
(235, 'Income Taxes Payable'),
(237, 'State Payroll Taxes Payable'),
(238, 'Employee FICA Taxes Payable'),
(239, 'Employer FICA Taxes Payable'),
(241, 'Employer FUTA Taxes Payable'),
(242, 'Employee SDI Taxes Payable'),
(243, 'Employer UCI Taxes Payable'),
(251, 'IBM Credit Corporation Payable'),
(280, 'Capital Stock'),
(290, 'Retained Earnings'),
(300, 'Retail Sales'),
(301, 'College Sales'),
(302, 'Trade Sales'),
(306, 'Consignment Sales'),
(310, 'Compositing Revenue'),
(394, 'Book Club Royalties'),
(400, 'Book Printing Costs'),
(403, 'Book Production Costs'),
(500, 'Salaries and Wages'),
(505, 'FICA'),
(506, 'FUTA'),
(507, 'UCI'),
(508, 'Medicare'),
(510, 'Group Insurance'),
(520, 'Building Lease'),
(521, 'Utilities'),
(522, 'Telephone'),
(523, 'Building Maintenance'),
(527, 'Computer Equipment Maintenance'),
(528, 'IBM Lease'),
(532, 'Equipment Rental'),
(536, 'Card Deck Advertising'),
(540, 'Direct Mail Advertising'),
(541, 'Space Advertising'),
(546, 'Exhibits and Shows'),
(548, 'Web Site Production and Fees'),
(550, 'Packaging Materials'),
(551, 'Business Forms'),
(552, 'Postage'),
(553, 'Freight'),
(555, 'Collection Agency Fees'),
(556, 'Credit Card Handling'),
(565, 'Bank Fees'),
(568, 'Auto License Fee'),
(569, 'Auto Expense'),
(570, 'Office Supplies'),
(572, 'Books, Dues, and Subscriptions'),
(574, 'Business Licenses and Taxes'),
(576, 'PC Software'),
(580, 'Meals'),
(582, 'Travel and Accomodations'),
(589, 'Outside Services'),
(590, 'Business Insurance'),
(591, 'Accounting'),
(610, 'Charitable Contributions'),
(611, 'Profit Sharing Contributions'),
(620, 'Interest Paid to Banks'),
(621, 'Other Interest'),
(630, 'Federal Corporation Income Taxes'),
(631, 'State Corporation Income Taxes'),
(632, 'Sales Tax')
GO

INSERT InvoiceLineItems (InvoiceID, InvoiceSequence, AccountNo, InvoiceLineItemAmount, InvoiceLineItemDescription) VALUES
(1, 1, 553, 3813.3300, 'Freight'),
(2, 1, 553, 40.2000, 'Freight'),
(3, 1, 553, 138.7500, 'Freight'),
(4, 1, 553, 144.7000, 'Int''l shipment'),
(5, 1, 553, 15.5000, 'Freight'),
(6, 1, 553, 42.7500, 'Freight'),
(7, 1, 553, 172.5000, 'Freight'),
(8, 1, 522, 95.0000, 'Telephone service'),
(9, 1, 403, 601.9500, 'Cover design'),
(10, 1, 553, 42.6700, 'Freight'),
(11, 1, 553, 42.5000, 'Freight'),
(12, 1, 580, 50.0000, 'DiCicco''s'),
(12, 2, 570, 75.6000, 'Kinko''s'),
(12, 3, 570, 58.4000, 'Office Max'),
(12, 4, 540, 478.0000, 'Publishers Marketing'),
(13, 1, 522, 16.3300, 'Telephone (line 5)'),
(14, 1, 553, 6.0000, 'Freight out'),
(15, 1, 574, 856.9200, 'Property Taxes'),
(16, 1, 572, 9.9500, 'Monthly access fee'),
(17, 1, 553, 10.0000, 'Address correction'),
(18, 1, 553, 104.0000, 'Freight'),
(19, 1, 160, 116.5400, 'MVS Online Library'),
(20, 1, 553, 6.0000, 'Freight out'),
(21, 1, 589, 4901.2600, 'Office lease'),
(22, 1, 553, 108.2500, 'Freight'),
(23, 1, 572, 9.9500, 'Monthly access fee'),
(24, 1, 520, 1750.0000, 'Warehouse lease'),
(25, 1, 553, 129.0000, 'Freight'),
(26, 1, 553, 10.0000, 'Freight'),
(27, 1, 540, 207.7800, 'Prospect list'),
(28, 1, 553, 109.5000, 'Freight'),
(29, 1, 523, 450.0000, 'Back office additions'),
(30, 1, 553, 63.4000, 'Freight'),
(31, 1, 589, 7125.3400, 'Web site design'),
(32, 1, 403, 953.1000, 'Crash Course revision'),
(33, 1, 591, 220.0000, 'Form 571-L'),
(34, 1, 553, 127.7500, 'Freight'),
(35, 1, 507, 1600.0000, 'Income Tax'),
(36, 1, 403, 565.1500, 'Crash Course Ad'),
(37, 1, 553, 36.0000, 'Freight'),
(38, 1, 553, 61.5000, 'Freight'),
(39, 1, 400, 37966.1900, 'CICS Desk Reference'),
(40, 1, 403, 639.7700, 'Card deck'),
(41, 1, 553, 53.7500, 'Freight'),
(42, 1, 553, 400.0000, 'Freight'),
(43, 1, 400, 21842.0000, 'Book repro'),
(44, 1, 522, 19.6700, 'Telephone (Line 3)'),
(45, 1, 553, 2765.3600, 'Freight'),
(46, 1, 510, 224.0000, 'Health Insurance'),
(47, 1, 572, 1575.0000, 'Catalog ad'),
(48, 1, 553, 33.0000, 'Freight'),
(49, 1, 522, 16.3300, 'Telephone (line 6)'),
(50, 1, 510, 116.0000, 'Health Insurance'),
(51, 1, 553, 2184.1100, 'Freight'),
(52, 1, 160, 1083.5800, 'MSDN'),
(53, 1, 522, 46.2100, 'Telephone (Line 1)'),
(54, 1, 403, 313.5500, 'Card revision'),
(55, 1, 553, 40.7500, 'Freight'),
(56, 1, 572, 2433.0000, 'Card deck'),
(57, 1, 589, 1367.5000, '401K Contributions'),
(58, 1, 553, 53.2500, 'Freight'),
(59, 1, 553, 13.7500, 'Freight'),
(60, 1, 553, 2312.2000, 'Freight'),
(61, 1, 553, 25.6700, 'Freight out'),
(62, 1, 553, 26.7500, 'Freight'),
(63, 1, 553, 2115.8100, 'Freight'),
(64, 1, 553, 23.5000, 'Freight'),
(65, 1, 400, 6940.2500, 'OS Utilities'),
(66, 1, 553, 31.9500, 'Freight'),
(67, 1, 553, 1927.5400, 'Freight'),
(68, 1, 160, 936.9300, 'Quarterly Maintenance'),
(69, 1, 540, 175.0000, 'Card deck advertising'),
(70, 1, 553, 6.0000, 'Freight'),
(71, 1, 553, 108.5000, 'Freight'),
(72, 1, 553, 10.0000, 'Address correction'),
(73, 1, 552, 290.0000, 'International pkg.'),
(74, 1, 570, 41.8000, 'Coffee'),
(75, 1, 553, 27.0000, 'Freight'),
(76, 1, 553, 241.0000, 'Int''l shipment'),
(77, 1, 403, 904.1400, 'Cover design'),
(78, 1, 403, 1197.0000, 'Cover design'),
(78, 2, 540, 765.1300, 'Catalog design'),
(79, 1, 540, 2184.5000, 'PC card deck'),
(80, 1, 553, 2318.0300, 'Freight'),
(81, 1, 553, 26.2500, 'Freight'),
(82, 1, 150, 17.5000, 'Supplies'),
(83, 1, 522, 39.7700, 'Telephone (Line 2)'),
(84, 1, 553, 111.0000, 'Freight'),
(85, 1, 553, 158.0000, 'Int''l shipment'),
(86, 1, 553, 739.2000, 'Freight'),
(87, 1, 553, 60.0000, 'Freight'),
(88, 1, 553, 147.2500, 'Freight'),
(89, 1, 400, 85.3100, 'Book copy'),
(90, 1, 553, 38.7500, 'Freight'),
(91, 1, 522, 32.7000, 'Telephone (line 4)'),
(92, 1, 521, 16.6200, 'Propane-forklift'),
(93, 1, 553, 162.7500, 'International shipment'),
(94, 1, 553, 52.2500, 'Freight'),
(95, 1, 572, 600.0000, 'Books for research'),
(96, 1, 400, 26881.4000, 'MVS JCL'),
(97, 1, 170, 356.4800, 'Network wiring'),
(98, 1, 572, 579.4200, 'Catalog ad'),
(99, 1, 553, 59.9700, 'Freight'),
(100, 1, 553, 67.9200, 'Freight'),
(101, 1, 553, 30.7500, 'Freight'),
(102, 1, 400, 20551.1800, 'CICS book printing'),
(103, 1, 553, 2051.5900, 'Freight'),
(104, 1, 553, 44.4400, 'Freight'),
(105, 1, 582, 503.2000, 'Bronco lease'),
(106, 1, 400, 23517.5800, 'DB2 book printing'),
(107, 1, 553, 3689.9900, 'Freight'),
(108, 1, 553, 67.0000, 'Freight'),
(109, 1, 403, 1000.4600, 'Crash Course covers'),
(110, 1, 540, 90.3600, 'Card deck advertising'),
(111, 1, 553, 22.5700, 'Freight'),
(112, 1, 400, 10976.0600, 'VSAM book printing'),
(113, 1, 510, 224.0000, 'Health Insurance'),
(114, 1, 553, 127.7500, 'Freight')
GO

SET IDENTITY_INSERT Invoices ON
INSERT Invoices (InvoiceID, VendorID, InvoiceNumber, InvoiceDate, InvoiceTotal, PaymentTotal, CreditTotal, TermsID, InvoiceDueDate, PaymentDate) VALUES
(1, 122, '989319-457', CAST('2015-12-08 00:00:00' AS SmallDateTime), 3813.3300, 3813.3300, 0.0000, 3, CAST('2016-01-08 00:00:00' AS SmallDateTime), CAST('2016-01-07 00:00:00' AS SmallDateTime)),
(2, 123, '263253241', CAST('2015-12-10 00:00:00' AS SmallDateTime), 40.2000, 40.2000, 0.0000, 3, CAST('2016-01-10 00:00:00' AS SmallDateTime), CAST('2016-01-14 00:00:00' AS SmallDateTime)),
(3, 123, '963253234', CAST('2015-12-13 00:00:00' AS SmallDateTime), 138.7500, 138.7500, 0.0000, 3, CAST('2016-01-13 00:00:00' AS SmallDateTime), CAST('2016-01-09 00:00:00' AS SmallDateTime)),
(4, 123, '2-000-2993', CAST('2015-12-16 00:00:00' AS SmallDateTime), 144.7000, 144.7000, 0.0000, 3, CAST('2016-01-16 00:00:00' AS SmallDateTime), CAST('2016-01-12 00:00:00' AS SmallDateTime)),
(5, 123, '963253251', CAST('2015-12-16 00:00:00' AS SmallDateTime), 15.5000, 15.5000, 0.0000, 3, CAST('2016-01-16 00:00:00' AS SmallDateTime), CAST('2016-01-11 00:00:00' AS SmallDateTime)),
(6, 123, '963253261', CAST('2015-12-16 00:00:00' AS SmallDateTime), 42.7500, 42.7500, 0.0000, 3, CAST('2016-01-16 00:00:00' AS SmallDateTime), CAST('2016-01-21 00:00:00' AS SmallDateTime)),
(7, 123, '963253237', CAST('2015-12-21 00:00:00' AS SmallDateTime), 172.5000, 172.5000, 0.0000, 3, CAST('2016-01-21 00:00:00' AS SmallDateTime), CAST('2016-01-22 00:00:00' AS SmallDateTime)),
(8, 89, '125520-1', CAST('2015-12-24 00:00:00' AS SmallDateTime), 95.0000, 95.0000, 0.0000, 1, CAST('2016-01-04 00:00:00' AS SmallDateTime), CAST('2016-01-01 00:00:00' AS SmallDateTime)),
(9, 121, '97/488', CAST('2015-12-24 00:00:00' AS SmallDateTime), 601.9500, 601.9500, 0.0000, 3, CAST('2016-01-24 00:00:00' AS SmallDateTime), CAST('2016-01-21 00:00:00' AS SmallDateTime)),
(10, 123, '263253250', CAST('2015-12-24 00:00:00' AS SmallDateTime), 42.6700, 42.6700, 0.0000, 3, CAST('2016-01-24 00:00:00' AS SmallDateTime), CAST('2016-01-22 00:00:00' AS SmallDateTime)),
(11, 123, '963253262', CAST('2015-12-25 00:00:00' AS SmallDateTime), 42.5000, 42.5000, 0.0000, 3, CAST('2016-01-25 00:00:00' AS SmallDateTime), CAST('2016-01-20 00:00:00' AS SmallDateTime)),
(12, 96, 'I77271-O01', CAST('2015-12-26 00:00:00' AS SmallDateTime), 662.0000, 662.0000, 0.0000, 2, CAST('2016-01-16 00:00:00' AS SmallDateTime), CAST('2016-01-13 00:00:00' AS SmallDateTime)),
(13, 95, '111-92R-10096', CAST('2015-12-30 00:00:00' AS SmallDateTime), 16.3300, 16.3300, 0.0000, 2, CAST('2016-01-20 00:00:00' AS SmallDateTime), CAST('2016-01-23 00:00:00' AS SmallDateTime)),
(14, 115, '25022117', CAST('2016-01-01 00:00:00' AS SmallDateTime), 6.0000, 6.0000, 0.0000, 4, CAST('2016-02-10 00:00:00' AS SmallDateTime), CAST('2016-02-10 00:00:00' AS SmallDateTime)),
(15, 48, 'P02-88D77S7', CAST('2016-01-03 00:00:00' AS SmallDateTime), 856.9200, 856.9200, 0.0000, 3, CAST('2016-02-02 00:00:00' AS SmallDateTime), CAST('2016-01-30 00:00:00' AS SmallDateTime)),
(16, 97, '21-4748363', CAST('2016-01-03 00:00:00' AS SmallDateTime), 9.9500, 9.9500, 0.0000, 2, CAST('2016-01-23 00:00:00' AS SmallDateTime), CAST('2016-01-22 00:00:00' AS SmallDateTime)),
(17, 123, '4-321-2596', CAST('2016-01-05 00:00:00' AS SmallDateTime), 10.0000, 10.0000, 0.0000, 3, CAST('2016-02-04 00:00:00' AS SmallDateTime), CAST('2016-02-05 00:00:00' AS SmallDateTime)),
(18, 123, '963253242', CAST('2016-01-06 00:00:00' AS SmallDateTime), 104.0000, 104.0000, 0.0000, 3, CAST('2016-02-05 00:00:00' AS SmallDateTime), CAST('2016-02-05 00:00:00' AS SmallDateTime)),
(19, 34, 'QP58872', CAST('2016-01-07 00:00:00' AS SmallDateTime), 116.5400, 116.5400, 0.0000, 1, CAST('2016-01-17 00:00:00' AS SmallDateTime), CAST('2016-01-19 00:00:00' AS SmallDateTime)),
(20, 115, '24863706', CAST('2016-01-10 00:00:00' AS SmallDateTime), 6.0000, 6.0000, 0.0000, 4, CAST('2016-02-19 00:00:00' AS SmallDateTime), CAST('2016-02-15 00:00:00' AS SmallDateTime)),
(21, 119, '10843', CAST('2016-01-11 00:00:00' AS SmallDateTime), 4901.2600, 4901.2600, 0.0000, 2, CAST('2016-01-31 00:00:00' AS SmallDateTime), CAST('2016-01-29 00:00:00' AS SmallDateTime)),
(22, 123, '963253235', CAST('2016-01-11 00:00:00' AS SmallDateTime), 108.2500, 108.2500, 0.0000, 3, CAST('2016-02-10 00:00:00' AS SmallDateTime), CAST('2016-02-09 00:00:00' AS SmallDateTime)),
(23, 97, '21-4923721', CAST('2016-01-13 00:00:00' AS SmallDateTime), 9.9500, 9.9500, 0.0000, 2, CAST('2016-02-02 00:00:00' AS SmallDateTime), CAST('2016-01-28 00:00:00' AS SmallDateTime)),
(24, 113, '77290', CAST('2016-01-13 00:00:00' AS SmallDateTime), 1750.0000, 1750.0000, 0.0000, 5, CAST('2016-03-02 00:00:00' AS SmallDateTime), CAST('2016-03-05 00:00:00' AS SmallDateTime)),
(25, 123, '963253246', CAST('2016-01-13 00:00:00' AS SmallDateTime), 129.0000, 129.0000, 0.0000, 3, CAST('2016-02-12 00:00:00' AS SmallDateTime), CAST('2016-02-09 00:00:00' AS SmallDateTime)),
(26, 123, '4-342-8069', CAST('2016-01-14 00:00:00' AS SmallDateTime), 10.0000, 10.0000, 0.0000, 3, CAST('2016-02-13 00:00:00' AS SmallDateTime), CAST('2016-02-13 00:00:00' AS SmallDateTime)),
(27, 88, '972110', CAST('2016-01-15 00:00:00' AS SmallDateTime), 207.7800, 207.7800, 0.0000, 1, CAST('2016-01-25 00:00:00' AS SmallDateTime), CAST('2016-01-27 00:00:00' AS SmallDateTime)),
(28, 123, '963253263', CAST('2016-01-16 00:00:00' AS SmallDateTime), 109.5000, 109.5000, 0.0000, 3, CAST('2016-02-15 00:00:00' AS SmallDateTime), CAST('2016-02-10 00:00:00' AS SmallDateTime)),
(29, 108, '121897', CAST('2016-01-19 00:00:00' AS SmallDateTime), 450.0000, 450.0000, 0.0000, 4, CAST('2016-02-28 00:00:00' AS SmallDateTime), CAST('2016-03-03 00:00:00' AS SmallDateTime)),
(30, 123, '1-200-5164', CAST('2016-01-20 00:00:00' AS SmallDateTime), 63.4000, 63.4000, 0.0000, 3, CAST('2016-02-19 00:00:00' AS SmallDateTime), CAST('2016-02-24 00:00:00' AS SmallDateTime)),
(31, 104, 'P02-3772', CAST('2016-01-21 00:00:00' AS SmallDateTime), 7125.3400, 7125.3400, 0.0000, 3, CAST('2016-02-20 00:00:00' AS SmallDateTime), CAST('2016-02-24 00:00:00' AS SmallDateTime)),
(32, 121, '97/486', CAST('2016-01-21 00:00:00' AS SmallDateTime), 953.1000, 953.1000, 0.0000, 3, CAST('2016-02-20 00:00:00' AS SmallDateTime), CAST('2016-02-22 00:00:00' AS SmallDateTime)),
(33, 105, '94007005', CAST('2016-01-23 00:00:00' AS SmallDateTime), 220.0000, 220.0000, 0.0000, 3, CAST('2016-02-22 00:00:00' AS SmallDateTime), CAST('2016-02-26 00:00:00' AS SmallDateTime)),
(34, 123, '963253232', CAST('2016-01-23 00:00:00' AS SmallDateTime), 127.7500, 127.7500, 0.0000, 3, CAST('2016-02-22 00:00:00' AS SmallDateTime), CAST('2016-02-18 00:00:00' AS SmallDateTime)),
(35, 107, 'RTR-72-3662-X', CAST('2016-01-25 00:00:00' AS SmallDateTime), 1600.0000, 1600.0000, 0.0000, 4, CAST('2016-03-04 00:00:00' AS SmallDateTime), CAST('2016-03-09 00:00:00' AS SmallDateTime)),
(36, 121, '97/465', CAST('2016-01-25 00:00:00' AS SmallDateTime), 565.1500, 565.1500, 0.0000, 3, CAST('2016-02-24 00:00:00' AS SmallDateTime), CAST('2016-02-24 00:00:00' AS SmallDateTime)),
(37, 123, '963253260', CAST('2016-01-25 00:00:00' AS SmallDateTime), 36.0000, 36.0000, 0.0000, 3, CAST('2016-02-24 00:00:00' AS SmallDateTime), CAST('2016-02-26 00:00:00' AS SmallDateTime)),
(38, 123, '963253272', CAST('2016-01-26 00:00:00' AS SmallDateTime), 61.5000, 61.5000, 0.0000, 3, CAST('2016-02-25 00:00:00' AS SmallDateTime), CAST('2016-02-28 00:00:00' AS SmallDateTime)),
(39, 110, '0-2058', CAST('2016-01-28 00:00:00' AS SmallDateTime), 37966.1900, 37966.1900, 0.0000, 3, CAST('2016-02-27 00:00:00' AS SmallDateTime), CAST('2016-02-28 00:00:00' AS SmallDateTime)),
(40, 121, '97/503', CAST('2016-01-30 00:00:00' AS SmallDateTime), 639.7700, 639.7700, 0.0000, 3, CAST('2016-02-28 00:00:00' AS SmallDateTime), CAST('2016-02-25 00:00:00' AS SmallDateTime)),
(41, 123, '963253255', CAST('2016-01-31 00:00:00' AS SmallDateTime), 53.7500, 53.7500, 0.0000, 3, CAST('2016-02-28 00:00:00' AS SmallDateTime), CAST('2016-02-27 00:00:00' AS SmallDateTime)),
(42, 123, '94007069', CAST('2016-01-31 00:00:00' AS SmallDateTime), 400.0000, 400.0000, 0.0000, 3, CAST('2016-02-28 00:00:00' AS SmallDateTime), CAST('2016-03-01 00:00:00' AS SmallDateTime)),
(43, 72, '40318', CAST('2016-02-01 00:00:00' AS SmallDateTime), 21842.0000, 21842.0000, 0.0000, 3, CAST('2016-03-01 00:00:00' AS SmallDateTime), CAST('2016-02-28 00:00:00' AS SmallDateTime)),
(44, 95, '111-92R-10094', CAST('2016-02-01 00:00:00' AS SmallDateTime), 19.6700, 19.6700, 0.0000, 2, CAST('2016-02-21 00:00:00' AS SmallDateTime), CAST('2016-02-24 00:00:00' AS SmallDateTime)),
(45, 122, '989319-437', CAST('2016-02-01 00:00:00' AS SmallDateTime), 2765.3600, 2765.3600, 0.0000, 3, CAST('2016-03-01 00:00:00' AS SmallDateTime), CAST('2016-02-28 00:00:00' AS SmallDateTime)),
(46, 37, '547481328', CAST('2016-02-03 00:00:00' AS SmallDateTime), 224.0000, 224.0000, 0.0000, 3, CAST('2016-03-03 00:00:00' AS SmallDateTime), CAST('2016-03-04 00:00:00' AS SmallDateTime)),
(47, 83, '31359783', CAST('2016-02-03 00:00:00' AS SmallDateTime), 1575.0000, 1575.0000, 0.0000, 2, CAST('2016-02-23 00:00:00' AS SmallDateTime), CAST('2016-02-21 00:00:00' AS SmallDateTime)),
(48, 123, '1-202-2978', CAST('2016-02-03 00:00:00' AS SmallDateTime), 33.0000, 33.0000, 0.0000, 3, CAST('2016-03-03 00:00:00' AS SmallDateTime), CAST('2016-03-05 00:00:00' AS SmallDateTime)),
(49, 95, '111-92R-10097', CAST('2016-02-04 00:00:00' AS SmallDateTime), 16.3300, 16.3300, 0.0000, 2, CAST('2016-02-24 00:00:00' AS SmallDateTime), CAST('2016-02-26 00:00:00' AS SmallDateTime)),
(50, 37, '547479217', CAST('2016-02-07 00:00:00' AS SmallDateTime), 116.0000, 116.0000, 0.0000, 3, CAST('2016-03-07 00:00:00' AS SmallDateTime), CAST('2016-03-07 00:00:00' AS SmallDateTime)),
(51, 122, '989319-477', CAST('2016-02-08 00:00:00' AS SmallDateTime), 2184.1100, 2184.1100, 0.0000, 3, CAST('2016-03-08 00:00:00' AS SmallDateTime), CAST('2016-03-08 00:00:00' AS SmallDateTime)),
(52, 34, 'Q545443', CAST('2016-02-09 00:00:00' AS SmallDateTime), 1083.5800, 1083.5800, 0.0000, 1, CAST('2016-02-19 00:00:00' AS SmallDateTime), CAST('2016-02-23 00:00:00' AS SmallDateTime)),
(53, 95, '111-92R-10092', CAST('2016-02-09 00:00:00' AS SmallDateTime), 46.2100, 46.2100, 0.0000, 2, CAST('2016-02-28 00:00:00' AS SmallDateTime), CAST('2016-03-02 00:00:00' AS SmallDateTime)),
(54, 121, '97/553B', CAST('2016-02-10 00:00:00' AS SmallDateTime), 313.5500, 313.5500, 0.0000, 3, CAST('2016-03-10 00:00:00' AS SmallDateTime), CAST('2016-03-09 00:00:00' AS SmallDateTime)),
(55, 123, '963253245', CAST('2016-02-10 00:00:00' AS SmallDateTime), 40.7500, 40.7500, 0.0000, 3, CAST('2016-03-10 00:00:00' AS SmallDateTime), CAST('2016-03-12 00:00:00' AS SmallDateTime)),
(56, 86, '367447', CAST('2016-02-11 00:00:00' AS SmallDateTime), 2433.0000, 2433.0000, 0.0000, 1, CAST('2016-02-21 00:00:00' AS SmallDateTime), CAST('2016-02-17 00:00:00' AS SmallDateTime)),
(57, 103, '75C-90227', CAST('2016-02-11 00:00:00' AS SmallDateTime), 1367.5000, 1367.5000, 0.0000, 5, CAST('2016-03-31 00:00:00' AS SmallDateTime), CAST('2016-03-31 00:00:00' AS SmallDateTime)),
(58, 123, '963253256', CAST('2016-02-11 00:00:00' AS SmallDateTime), 53.2500, 53.2500, 0.0000, 3, CAST('2016-03-11 00:00:00' AS SmallDateTime), CAST('2016-03-07 00:00:00' AS SmallDateTime)),
(59, 123, '4-314-3057', CAST('2016-02-11 00:00:00' AS SmallDateTime), 13.7500, 13.7500, 0.0000, 3, CAST('2016-03-11 00:00:00' AS SmallDateTime), CAST('2016-03-15 00:00:00' AS SmallDateTime)),
(60, 122, '989319-497', CAST('2016-02-12 00:00:00' AS SmallDateTime), 2312.2000, 2312.2000, 0.0000, 3, CAST('2016-03-12 00:00:00' AS SmallDateTime), CAST('2016-03-09 00:00:00' AS SmallDateTime)),
(61, 115, '24946731', CAST('2016-02-15 00:00:00' AS SmallDateTime), 25.6700, 25.6700, 0.0000, 4, CAST('2016-03-25 00:00:00' AS SmallDateTime), CAST('2016-03-26 00:00:00' AS SmallDateTime)),
(62, 123, '963253269', CAST('2016-02-15 00:00:00' AS SmallDateTime), 26.7500, 26.7500, 0.0000, 3, CAST('2016-03-15 00:00:00' AS SmallDateTime), CAST('2016-03-11 00:00:00' AS SmallDateTime)),
(63, 122, '989319-427', CAST('2016-02-16 00:00:00' AS SmallDateTime), 2115.8100, 2115.8100, 0.0000, 3, CAST('2016-03-16 00:00:00' AS SmallDateTime), CAST('2016-03-19 00:00:00' AS SmallDateTime)),
(64, 123, '963253267', CAST('2016-02-17 00:00:00' AS SmallDateTime), 23.5000, 23.5000, 0.0000, 3, CAST('2016-03-17 00:00:00' AS SmallDateTime), CAST('2016-03-19 00:00:00' AS SmallDateTime)),
(65, 99, '509786', CAST('2016-02-18 00:00:00' AS SmallDateTime), 6940.2500, 6940.2500, 0.0000, 3, CAST('2016-03-18 00:00:00' AS SmallDateTime), CAST('2016-03-15 00:00:00' AS SmallDateTime)),
(66, 123, '263253253', CAST('2016-02-18 00:00:00' AS SmallDateTime), 31.9500, 31.9500, 0.0000, 3, CAST('2016-03-18 00:00:00' AS SmallDateTime), CAST('2016-03-21 00:00:00' AS SmallDateTime)),
(67, 122, '989319-487', CAST('2016-02-20 00:00:00' AS SmallDateTime), 1927.5400, 1927.5400, 0.0000, 3, CAST('2016-03-20 00:00:00' AS SmallDateTime), CAST('2016-03-18 00:00:00' AS SmallDateTime)),
(68, 81, 'MABO1489', CAST('2016-02-21 00:00:00' AS SmallDateTime), 936.9300, 936.9300, 0.0000, 2, CAST('2016-03-11 00:00:00' AS SmallDateTime), CAST('2016-03-10 00:00:00' AS SmallDateTime)),
(69, 80, '133560', CAST('2016-02-22 00:00:00' AS SmallDateTime), 175.0000, 175.0000, 0.0000, 2, CAST('2016-03-12 00:00:00' AS SmallDateTime), CAST('2016-03-16 00:00:00' AS SmallDateTime)),
(70, 115, '24780512', CAST('2016-02-22 00:00:00' AS SmallDateTime), 6.0000, 6.0000, 0.0000, 4, CAST('2016-04-01 00:00:00' AS SmallDateTime), CAST('2016-03-29 00:00:00' AS SmallDateTime)),
(71, 123, '963253254', CAST('2016-02-22 00:00:00' AS SmallDateTime), 108.5000, 108.5000, 0.0000, 3, CAST('2016-03-22 00:00:00' AS SmallDateTime), CAST('2016-03-20 00:00:00' AS SmallDateTime)),
(72, 123, '43966316', CAST('2016-02-22 00:00:00' AS SmallDateTime), 10.0000, 10.0000, 0.0000, 3, CAST('2016-03-22 00:00:00' AS SmallDateTime), CAST('2016-03-17 00:00:00' AS SmallDateTime)),
(73, 114, 'CBM9920-M-T77109', CAST('2016-02-23 00:00:00' AS SmallDateTime), 290.0000, 290.0000, 0.0000, 1, CAST('2016-03-03 00:00:00' AS SmallDateTime), CAST('2016-02-28 00:00:00' AS SmallDateTime)),
(74, 102, '109596', CAST('2016-02-24 00:00:00' AS SmallDateTime), 41.8000, 41.8000, 0.0000, 4, CAST('2016-04-03 00:00:00' AS SmallDateTime), CAST('2016-04-04 00:00:00' AS SmallDateTime)),
(75, 123, '7548906-20', CAST('2016-02-24 00:00:00' AS SmallDateTime), 27.0000, 27.0000, 0.0000, 3, CAST('2016-03-24 00:00:00' AS SmallDateTime), CAST('2016-03-24 00:00:00' AS SmallDateTime)),
(76, 123, '963253248', CAST('2016-02-24 00:00:00' AS SmallDateTime), 241.0000, 241.0000, 0.0000, 3, CAST('2016-03-24 00:00:00' AS SmallDateTime), CAST('2016-03-25 00:00:00' AS SmallDateTime)),
(77, 121, '97/553', CAST('2016-02-25 00:00:00' AS SmallDateTime), 904.1400, 904.1400, 0.0000, 3, CAST('2016-03-25 00:00:00' AS SmallDateTime), CAST('2016-03-25 00:00:00' AS SmallDateTime)),
(78, 121, '97/522', CAST('2016-02-28 00:00:00' AS SmallDateTime), 1962.1300, 1762.1300, 200.0000, 3, CAST('2016-03-28 00:00:00' AS SmallDateTime), CAST('2016-03-30 00:00:00' AS SmallDateTime)),
(79, 100, '587056', CAST('2016-02-28 00:00:00' AS SmallDateTime), 2184.5000, 2184.5000, 0.0000, 4, CAST('2016-04-09 00:00:00' AS SmallDateTime), CAST('2016-04-07 00:00:00' AS SmallDateTime)),
(80, 122, '989319-467', CAST('2016-03-01 00:00:00' AS SmallDateTime), 2318.0300, 2318.0300, 0.0000, 3, CAST('2016-03-31 00:00:00' AS SmallDateTime), CAST('2016-03-29 00:00:00' AS SmallDateTime)),
(81, 123, '263253265', CAST('2016-03-02 00:00:00' AS SmallDateTime), 26.2500, 26.2500, 0.0000, 3, CAST('2016-04-01 00:00:00' AS SmallDateTime), CAST('2016-03-28 00:00:00' AS SmallDateTime)),
(82, 94, '203339-13', CAST('2016-03-05 00:00:00' AS SmallDateTime), 17.5000, 17.5000, 0.0000, 2, CAST('2016-03-25 00:00:00' AS SmallDateTime), CAST('2016-03-27 00:00:00' AS SmallDateTime)),
(83, 95, '111-92R-10093', CAST('2016-03-06 00:00:00' AS SmallDateTime), 39.7700, 39.7700, 0.0000, 2, CAST('2016-03-26 00:00:00' AS SmallDateTime), CAST('2016-03-22 00:00:00' AS SmallDateTime)),
(84, 123, '963253258', CAST('2016-03-06 00:00:00' AS SmallDateTime), 111.0000, 111.0000, 0.0000, 3, CAST('2016-04-05 00:00:00' AS SmallDateTime), CAST('2016-04-05 00:00:00' AS SmallDateTime)),
(85, 123, '963253271', CAST('2016-03-07 00:00:00' AS SmallDateTime), 158.0000, 158.0000, 0.0000, 3, CAST('2016-04-06 00:00:00' AS SmallDateTime), CAST('2016-04-11 00:00:00' AS SmallDateTime)),
(86, 123, '963253230', CAST('2016-03-07 00:00:00' AS SmallDateTime), 739.2000, 739.2000, 0.0000, 3, CAST('2016-04-06 00:00:00' AS SmallDateTime), CAST('2016-04-06 00:00:00' AS SmallDateTime)),
(87, 123, '963253244', CAST('2016-03-08 00:00:00' AS SmallDateTime), 60.0000, 60.0000, 0.0000, 3, CAST('2016-04-07 00:00:00' AS SmallDateTime), CAST('2016-04-09 00:00:00' AS SmallDateTime)),
(88, 123, '963253239', CAST('2016-03-08 00:00:00' AS SmallDateTime), 147.2500, 147.2500, 0.0000, 3, CAST('2016-04-07 00:00:00' AS SmallDateTime), CAST('2016-04-11 00:00:00' AS SmallDateTime)),
(89, 72, '39104', CAST('2016-03-10 00:00:00' AS SmallDateTime), 85.3100, 0.0000, 0.0000, 3, CAST('2016-04-09 00:00:00' AS SmallDateTime), NULL),
(90, 123, '963253252', CAST('2016-03-12 00:00:00' AS SmallDateTime), 38.7500, 38.7500, 0.0000, 3, CAST('2016-04-11 00:00:00' AS SmallDateTime), CAST('2016-04-11 00:00:00' AS SmallDateTime)),
(91, 95, '111-92R-10095', CAST('2016-03-15 00:00:00' AS SmallDateTime), 32.7000, 32.7000, 0.0000, 2, CAST('2016-04-04 00:00:00' AS SmallDateTime), CAST('2016-04-06 00:00:00' AS SmallDateTime)),
(92, 117, '111897', CAST('2016-03-15 00:00:00' AS SmallDateTime), 16.6200, 16.6200, 0.0000, 3, CAST('2016-04-14 00:00:00' AS SmallDateTime), CAST('2016-04-14 00:00:00' AS SmallDateTime)),
(93, 123, '4-327-7357', CAST('2016-03-16 00:00:00' AS SmallDateTime), 162.7500, 162.7500, 0.0000, 3, CAST('2016-04-15 00:00:00' AS SmallDateTime), CAST('2016-04-11 00:00:00' AS SmallDateTime)),
(94, 123, '963253264', CAST('2016-03-18 00:00:00' AS SmallDateTime), 52.2500, 0.0000, 0.0000, 3, CAST('2016-04-17 00:00:00' AS SmallDateTime), NULL),
(95, 82, 'C73-24', CAST('2016-03-19 00:00:00' AS SmallDateTime), 600.0000, 600.0000, 0.0000, 2, CAST('2016-04-08 00:00:00' AS SmallDateTime), CAST('2016-04-13 00:00:00' AS SmallDateTime)),
(96, 110, 'P-0259', CAST('2016-03-19 00:00:00' AS SmallDateTime), 26881.4000, 26881.4000, 0.0000, 3, CAST('2016-04-18 00:00:00' AS SmallDateTime), CAST('2016-04-20 00:00:00' AS SmallDateTime)),
(97, 90, '97-1024A', CAST('2016-03-20 00:00:00' AS SmallDateTime), 356.4800, 356.4800, 0.0000, 2, CAST('2016-04-09 00:00:00' AS SmallDateTime), CAST('2016-04-07 00:00:00' AS SmallDateTime)),
(98, 83, '31361833', CAST('2016-03-21 00:00:00' AS SmallDateTime), 579.4200, 0.0000, 0.0000, 2, CAST('2016-04-10 00:00:00' AS SmallDateTime), NULL),
(99, 123, '263253268', CAST('2016-03-21 00:00:00' AS SmallDateTime), 59.9700, 0.0000, 0.0000, 3, CAST('2016-04-20 00:00:00' AS SmallDateTime), NULL),
(100, 123, '263253270', CAST('2016-03-22 00:00:00' AS SmallDateTime), 67.9200, 0.0000, 0.0000, 3, CAST('2016-04-21 00:00:00' AS SmallDateTime), NULL),
(101, 123, '263253273', CAST('2016-03-22 00:00:00' AS SmallDateTime), 30.7500, 0.0000, 0.0000, 3, CAST('2016-04-21 00:00:00' AS SmallDateTime), NULL),
(102, 110, 'P-0608', CAST('2016-03-23 00:00:00' AS SmallDateTime), 20551.1800, 0.0000, 1200.0000, 3, CAST('2016-04-22 00:00:00' AS SmallDateTime), NULL),
(103, 122, '989319-417', CAST('2016-03-23 00:00:00' AS SmallDateTime), 2051.5900, 2051.5900, 0.0000, 3, CAST('2016-04-22 00:00:00' AS SmallDateTime), CAST('2016-04-24 00:00:00' AS SmallDateTime)),
(104, 123, '263253243', CAST('2016-03-23 00:00:00' AS SmallDateTime), 44.4400, 44.4400, 0.0000, 3, CAST('2016-04-22 00:00:00' AS SmallDateTime), CAST('2016-04-24 00:00:00' AS SmallDateTime)),
(105, 106, '9982771', CAST('2016-03-24 00:00:00' AS SmallDateTime), 503.2000, 0.0000, 0.0000, 3, CAST('2016-04-23 00:00:00' AS SmallDateTime), NULL),
(106, 110, '0-2060', CAST('2016-03-24 00:00:00' AS SmallDateTime), 23517.5800, 21221.6300, 2295.9500, 3, CAST('2016-04-23 00:00:00' AS SmallDateTime), CAST('2016-04-27 00:00:00' AS SmallDateTime)),
(107, 122, '989319-447', CAST('2016-03-24 00:00:00' AS SmallDateTime), 3689.9900, 3689.9900, 0.0000, 3, CAST('2016-04-23 00:00:00' AS SmallDateTime), CAST('2016-04-19 00:00:00' AS SmallDateTime)),
(108, 123, '963253240', CAST('2016-03-24 00:00:00' AS SmallDateTime), 67.0000, 67.0000, 0.0000, 3, CAST('2016-04-23 00:00:00' AS SmallDateTime), CAST('2016-04-23 00:00:00' AS SmallDateTime)),
(109, 121, '97/222', CAST('2016-03-25 00:00:00' AS SmallDateTime), 1000.4600, 1000.4600, 0.0000, 3, CAST('2016-04-24 00:00:00' AS SmallDateTime), CAST('2016-04-22 00:00:00' AS SmallDateTime)),
(110, 80, '134116', CAST('2016-03-28 00:00:00' AS SmallDateTime), 90.3600, 0.0000, 0.0000, 2, CAST('2016-04-17 00:00:00' AS SmallDateTime), NULL),
(111, 123, '263253257', CAST('2016-03-30 00:00:00' AS SmallDateTime), 22.5700, 22.5700, 0.0000, 3, CAST('2016-04-29 00:00:00' AS SmallDateTime), CAST('2016-05-03 00:00:00' AS SmallDateTime)),
(112, 110, '0-2436', CAST('2016-03-31 00:00:00' AS SmallDateTime), 10976.0600, 0.0000, 0.0000, 3, CAST('2016-04-30 00:00:00' AS SmallDateTime), NULL),
(113, 37, '547480102', CAST('2016-04-01 00:00:00' AS SmallDateTime), 224.0000, 0.0000, 0.0000, 3, CAST('2016-04-30 00:00:00' AS SmallDateTime), NULL),
(114, 123, '963253249', CAST('2016-04-02 00:00:00' AS SmallDateTime), 127.7500, 127.7500, 0.0000, 3, CAST('2016-05-01 00:00:00' AS SmallDateTime), CAST('2016-05-04 00:00:00' AS SmallDateTime))
SET IDENTITY_INSERT Invoices OFF
GO

SET IDENTITY_INSERT Terms ON
INSERT Terms (TermsID, TermsDescription, TermsDueDays) VALUES
(1, 'Net due 10 days', 10),
(2, 'Net due 20 days', 20),
(3, 'Net due 30 days', 30),
(4, 'Net due 60 days', 60),
(5, 'Net due 90 days', 90)
SET IDENTITY_INSERT Terms OFF
GO

SET IDENTITY_INSERT Vendors ON
INSERT Vendors (VendorID, VendorName, VendorAddress1, VendorAddress2, VendorCity, VendorState, VendorZipCode, VendorPhone, VendorContactLName, VendorContactFName, DefaultTermsID, DefaultAccountNo) VALUES
(1, 'US Postal Service', 'Attn:  Supt. Window Services', 'PO Box 7005', 'Madison', 'WI', '53707', '(800) 555-1205', 'Alberto', 'Francesco', 1, 552),
(2, 'National Information Data Ctr', 'PO Box 96621', NULL, 'Washington', 'DC', '20090', '(301) 555-8950', 'Irvin', 'Ania', 3, 540),
(3, 'Register of Copyrights', 'Library Of Congress', NULL, 'Washington', 'DC', '20559', NULL, 'Liana', 'Lukas', 3, 403),
(4, 'Jobtrak', '1990 Westwood Blvd Ste 260', NULL, 'Los Angeles', 'CA', '90025', '(800) 555-8725', 'Quinn', 'Kenzie', 3, 572),
(5, 'Newbrige Book Clubs', '3000 Cindel Drive', NULL, 'Washington', 'NJ', '07882', '(800) 555-9980', 'Marks', 'Michelle', 4, 394),
(6, 'California Chamber Of Commerce', '3255 Ramos Cir', NULL, 'Sacramento', 'CA', '95827', '(916) 555-6670', 'Mauro', 'Anton', 3, 572),
(7, 'Towne Advertiser''s Mailing Svcs', 'Kevin Minder', '3441 W Macarthur Blvd', 'Santa Ana', 'CA', '92704', NULL, 'Maegen', 'Ted', 3, 540),
(8, 'BFI Industries', 'PO Box 9369', NULL, 'Fresno', 'CA', '93792', '(559) 555-1551', 'Kaleigh', 'Erick', 3, 521),
(9, 'Pacific Gas & Electric', 'Box 52001', NULL, 'San Francisco', 'CA', '94152', '(800) 555-6081', 'Anthoni', 'Kaitlyn', 3, 521),
(10, 'Robbins Mobile Lock And Key', '4669 N Fresno', NULL, 'Fresno', 'CA', '93726', '(559) 555-9375', 'Leigh', 'Bill', 2, 523),
(11, 'Bill Marvin Electric Inc', '4583 E Home', NULL, 'Fresno', 'CA', '93703', '(559) 555-5106', 'Hostlery', 'Kaitlin', 2, 523),
(12, 'City Of Fresno', 'PO Box 2069', NULL, 'Fresno', 'CA', '93718', '(559) 555-9999', 'Mayte', 'Kendall', 3, 574),
(13, 'Golden Eagle Insurance Co', 'PO Box 85826', NULL, 'San Diego', 'CA', '92186', NULL, 'Blanca', 'Korah', 3, 590),
(14, 'Expedata Inc', '4420 N. First Street, Suite 108', NULL, 'Fresno', 'CA', '93726', '(559) 555-9586', 'Quintin', 'Marvin', 3, 589),
(15, 'ASC Signs', '1528 N Sierra Vista', NULL, 'Fresno', 'CA', '93703', NULL, 'Darien', 'Elisabeth', 1, 546),
(16, 'Internal Revenue Service', NULL, NULL, 'Fresno', 'CA', '93888', NULL, 'Aileen', 'Joan', 1, 235),
(17, 'Blanchard & Johnson Associates', '27371 Valderas', NULL, 'Mission Viejo', 'CA', '92691', '(214) 555-3647', 'Keeton', 'Gonzalo', 3, 540),
(18, 'Fresno Photoengraving Company', '1952 "H" Street', 'P.O. Box 1952', 'Fresno', 'CA', '93718', '(559) 555-3005', 'Chaddick', 'Derek', 3, 403),
(19, 'Crown Printing', '1730 "H" St', NULL, 'Fresno', 'CA', '93721', '(559) 555-7473', 'Randrup', 'Leann', 2, 400),
(20, 'Diversified Printing & Pub', '2632 Saturn St', NULL, 'Brea', 'CA', '92621', '(714) 555-4541', 'Lane', 'Vanesa', 3, 400),
(21, 'The Library Ltd', '7700 Forsyth', NULL, 'St Louis', 'MO', '63105', '(314) 555-8834', 'Marques', 'Malia', 3, 540),
(22, 'Micro Center', '1555 W Lane Ave', NULL, 'Columbus', 'OH', '43221', '(614) 555-4435', 'Evan', 'Emily', 2, 160),
(23, 'Yale Industrial Trucks-Fresno', '3711 W Franklin', NULL, 'Fresno', 'CA', '93706', '(559) 555-2993', 'Alexis', 'Alexandro', 3, 532),
(24, 'Zee Medical Service Co', '4221 W Sierra Madre #104', NULL, 'Washington', 'IA', '52353', NULL, 'Hallie', 'Juliana', 3, 570),
(25, 'California Data Marketing', '2818 E Hamilton', NULL, 'Fresno', 'CA', '93721', '(559) 555-3801', 'Jonessen', 'Moises', 4, 540),
(26, 'Small Press', '121 E Front St - 4th Floor', NULL, 'Traverse City', 'MI', '49684', NULL, 'Colette', 'Dusty', 3, 540),
(27, 'Rich Advertising', '12 Daniel Road', NULL, 'Fairfield', 'NJ', '07004', '(201) 555-9742', 'Neil', 'Ingrid', 3, 540),
(29, 'Vision Envelope & Printing', 'PO Box 3100', NULL, 'Gardena', 'CA', '90247', '(310) 555-7062', 'Raven', 'Jamari', 3, 551),
(30, 'Costco', 'Fresno Warehouse', '4500 W Shaw', 'Fresno', 'CA', '93711', NULL, 'Jaquan', 'Aaron', 3, 570),
(31, 'Enterprise Communications Inc', '1483 Chain Bridge Rd, Ste 202', NULL, 'Mclean', 'VA', '22101', '(770) 555-9558', 'Lawrence', 'Eileen', 2, 536),
(32, 'RR Bowker', 'PO Box 31', NULL, 'East Brunswick', 'NJ', '08810', '(800) 555-8110', 'Essence', 'Marjorie', 3, 532),
(33, 'Nielson', 'Ohio Valley Litho Division', 'Location #0470', 'Cincinnati', 'OH', '45264', NULL, 'Brooklynn', 'Keely', 2, 541),
(34, 'IBM', 'PO Box 61000', NULL, 'San Francisco', 'CA', '94161', '(800) 555-4426', 'Camron', 'Trentin', 1, 160),
(35, 'Cal State Termite', 'PO Box 956', NULL, 'Selma', 'CA', '93662', '(559) 555-1534', 'Hunter', 'Demetrius', 2, 523),
(36, 'Graylift', 'PO Box 2808', NULL, 'Fresno', 'CA', '93745', '(559) 555-6621', 'Sydney', 'Deangelo', 3, 532),
(37, 'Blue Cross', 'PO Box 9061', NULL, 'Oxnard', 'CA', '93031', '(800) 555-0912', 'Eliana', 'Nikolas', 3, 510),
(38, 'Venture Communications Int''l', '60 Madison Ave', NULL, 'New York', 'NY', '10010', '(212) 555-4800', 'Neftaly', 'Thalia', 3, 540),
(39, 'Custom Printing Company', 'PO Box 7028', NULL, 'St Louis', 'MO', '63177', '(301) 555-1494', 'Myles', 'Harley', 3, 540),
(40, 'Nat Assoc of College Stores', '500 East Lorain Street', NULL, 'Oberlin', 'OH', '44074', NULL, 'Bernard', 'Lucy', 3, 572),
(41, 'Shields Design', '415 E Olive Ave', NULL, 'Fresno', 'CA', '93728', '(559) 555-8060', 'Kerry', 'Rowan', 2, 403),
(42, 'Opamp Technical Books', '1033 N Sycamore Ave.', NULL, 'Los Angeles', 'CA', '90038', '(213) 555-4322', 'Paris', 'Gideon', 3, 572),
(43, 'Capital Resource Credit', 'PO Box 39046', NULL, 'Minneapolis', 'MN', '55439', '(612) 555-0057', 'Maxwell', 'Jayda', 3, 589),
(44, 'Courier Companies, Inc', 'PO Box 5317', NULL, 'Boston', 'MA', '02206', '(508) 555-6351', 'Antavius', 'Troy', 4, 400),
(45, 'Naylor Publications Inc', 'PO Box 40513', NULL, 'Jacksonville', 'FL', '32231', '(800) 555-6041', 'Gerald', 'Kristofer', 3, 572),
(46, 'Open Horizons Publishing', 'Book Marketing Update', 'PO Box 205', 'Fairfield', 'IA', '52556', '(515) 555-6130', 'Damien', 'Deborah', 2, 540),
(47, 'Baker & Taylor Books', 'Five Lakepointe Plaza, Ste 500', '2709 Water Ridge Parkway', 'Charlotte', 'NC', '28217', '(704) 555-3500', 'Bernardo', 'Brittnee', 3, 572),
(48, 'Fresno County Tax Collector', 'PO Box 1192', NULL, 'Fresno', 'CA', '93715', '(559) 555-3482', 'Brenton', 'Kila', 3, 574),
(49, 'Mcgraw Hill Companies', 'PO Box 87373', NULL, 'Chicago', 'IL', '60680', '(614) 555-3663', 'Holbrooke', 'Rashad', 3, 572),
(50, 'Publishers Weekly', 'Box 1979', NULL, 'Marion', 'OH', '43305', '(800) 555-1669', 'Carrollton', 'Priscilla', 3, 572),
(51, 'Blue Shield of California', 'PO Box 7021', NULL, 'Anaheim', 'CA', '92850', '(415) 555-5103', 'Smith', 'Kylie', 3, 510),
(52, 'Aztek Label', 'Accounts Payable', '1150 N Tustin Ave', 'Anaheim', 'CA', '92807', '(714) 555-9000', 'Griffin', 'Brian', 3, 551),
(53, 'Gary McKeighan Insurance', '3649 W Beechwood Ave #101', NULL, 'Fresno', 'CA', '93711', '(559) 555-2420', 'Jair', 'Caitlin', 3, 590),
(54, 'Ph Photographic Services', '2384 E Gettysburg', NULL, 'Fresno', 'CA', '93726', '(559) 555-0765', 'Cheyenne', 'Kaylea', 3, 540),
(55, 'Quality Education Data', 'PO Box 95857', NULL, 'Chicago', 'IL', '60694', '(800) 555-5811', 'Misael', 'Kayle', 2, 540),
(56, 'Springhouse Corp', 'PO Box 7247-7051', NULL, 'Philadelphia', 'PA', '19170', '(215) 555-8700', 'Maeve', 'Clarence', 3, 523),
(57, 'The Windows Deck', '117 W Micheltorena Top Floor', NULL, 'Santa Barbara', 'CA', '93101', '(800) 555-3353', 'Wood', 'Liam', 3, 536),
(58, 'Fresno Rack & Shelving Inc', '4718 N Bendel Ave', NULL, 'Fresno', 'CA', '93722', NULL, 'Baylee', 'Dakota', 2, 523),
(59, 'Publishers Marketing Assoc', '627 Aviation Way', NULL, 'Manhatttan Beach', 'CA', '90266', '(310) 555-2732', 'Walker', 'Jovon', 3, 572),
(60, 'The Mailers Guide Co', 'PO Box 1550', NULL, 'New Rochelle', 'NY', '10802', NULL, 'Lacy', 'Karina', 3, 540),
(61, 'American Booksellers Assoc', '828 S Broadway', NULL, 'Tarrytown', 'NY', '10591', '(800) 555-0037', 'Angelica', 'Nashalie', 3, 574),
(62, 'Cmg Information Services', 'PO Box 2283', NULL, 'Boston', 'MA', '02107', '(508) 555-7000', 'Randall', 'Yash', 3, 540),
(63, 'Lou Gentile''s Flower Basket', '722 E Olive Ave', NULL, 'Fresno', 'CA', '93728', '(559) 555-6643', 'Anum', 'Trisha', 1, 570),
(64, 'Texaco', 'PO Box 6070', NULL, 'Inglewood', 'CA', '90312', NULL, 'Oren', 'Grace', 3, 582),
(65, 'The Drawing Board', 'PO Box 4758', NULL, 'Carol Stream', 'IL', '60197', NULL, 'Mckayla', 'Jeffery', 2, 551),
(66, 'Ascom Hasler Mailing Systems', 'PO Box 895', NULL, 'Shelton', 'CT', '06484', NULL, 'Lewis', 'Darnell', 3, 532),
(67, 'Bill Jones', 'Secretary Of State', 'PO Box 944230', 'Sacramento', 'CA', '94244', NULL, 'Deasia', 'Tristin', 3, 589),
(68, 'Computer Library', '3502 W Greenway #7', NULL, 'Phoenix', 'AZ', '85023', '(602) 547-0331', 'Aryn', 'Leroy', 3, 540),
(69, 'Frank E Wilber Co', '2437 N Sunnyside', NULL, 'Fresno', 'CA', '93727', '(559) 555-1881', 'Millerton', 'Johnathon', 3, 532),
(70, 'Fresno Credit Bureau', 'PO Box 942', NULL, 'Fresno', 'CA', '93714', '(559) 555-7900', 'Braydon', 'Anne', 2, 555),
(71, 'The Fresno Bee', '1626 E Street', NULL, 'Fresno', 'CA', '93786', '(559) 555-4442', 'Colton', 'Leah', 2, 572),
(72, 'Data Reproductions Corp', '4545 Glenmeade Lane', NULL, 'Auburn Hills', 'MI', '48326', '(810) 555-3700', 'Arodondo', 'Cesar', 3, 400),
(73, 'Executive Office Products', '353 E Shaw Ave', NULL, 'Fresno', 'CA', '93710', '(559) 555-1704', 'Danielson', 'Rachael', 2, 570),
(74, 'Leslie Company', 'PO Box 610', NULL, 'Olathe', 'KS', '66061', '(800) 255-6210', 'Alondra', 'Zev', 3, 570),
(75, 'Retirement Plan Consultants', '6435 North Palm Ave, Ste 101', NULL, 'Fresno', 'CA', '93704', '(559) 555-7070', 'Edgardo', 'Salina', 3, 589),
(76, 'Simon Direct Inc', '4 Cornwall Dr Ste 102', NULL, 'East Brunswick', 'NJ', '08816', '(908) 555-7222', 'Bradlee', 'Daniel', 2, 540),
(77, 'State Board Of Equalization', 'PO Box 942808', NULL, 'Sacramento', 'CA', '94208', '(916) 555-4911', 'Dean', 'Julissa', 1, 631),
(78, 'The Presort Center', '1627 "E" Street', NULL, 'Fresno', 'CA', '93706', '(559) 555-6151', 'Marissa', 'Kyle', 3, 540),
(79, 'Valprint', 'PO Box 12332', NULL, 'Fresno', 'CA', '93777', '(559) 555-3112', 'Warren', 'Quentin', 3, 551),
(80, 'Cardinal Business Media, Inc.', 'P O Box 7247-7844', NULL, 'Philadelphia', 'PA', '19170', '(215) 555-1500', 'Eulalia', 'Kelsey', 2, 540),
(81, 'Wang Laboratories, Inc.', 'P.O. Box 21209', NULL, 'Pasadena', 'CA', '91185', '(800) 555-0344', 'Kapil', 'Robert', 2, 160),
(82, 'Reiter''s Scientific & Pro Books', '2021 K Street Nw', NULL, 'Washington', 'DC', '20006', '(202) 555-5561', 'Rodolfo', 'Carlee', 2, 572),
(83, 'Ingram', 'PO Box 845361', NULL, 'Dallas', 'TX', '75284', NULL, 'Yobani', 'Trey', 2, 541),
(84, 'Boucher Communications Inc', '1300 Virginia Dr. Ste 400', NULL, 'Fort Washington', 'PA', '19034', '(215) 555-8000', 'Carson', 'Julian', 3, 540),
(85, 'Champion Printing Company', '3250 Spring Grove Ave', NULL, 'Cincinnati', 'OH', '45225', '(800) 555-1957', 'Clifford', 'Jillian', 3, 540),
(86, 'Computerworld', 'Department #1872', 'PO Box 61000', 'San Francisco', 'CA', '94161', '(617) 555-0700', 'Lloyd', 'Angel', 1, 572),
(87, 'DMV Renewal', 'PO Box 942894', NULL, 'Sacramento', 'CA', '94294', NULL, 'Josey', 'Lorena', 4, 568),
(88, 'Edward Data Services', '4775 E Miami River Rd', NULL, 'Cleves', 'OH', '45002', '(513) 555-3043', 'Helena', 'Jeanette', 1, 540),
(89, 'Evans Executone Inc', '4918 Taylor Ct', NULL, 'Turlock', 'CA', '95380', NULL, 'Royce', 'Hannah', 1, 522),
(90, 'Wakefield Co', '295 W Cromwell Ave Ste 106', NULL, 'Fresno', 'CA', '93711', '(559) 555-4744', 'Rothman', 'Nathanael', 2, 170),
(91, 'McKesson Water Products', 'P O Box 7126', NULL, 'Pasadena', 'CA', '91109', '(800) 555-7009', 'Destin', 'Luciano', 2, 570),
(92, 'Zip Print & Copy Center', 'PO Box 12332', NULL, 'Fresno', 'CA', '93777', '(233) 555-6400', 'Javen', 'Justin', 2, 540),
(93, 'AT&T', 'PO Box 78225', NULL, 'Phoenix', 'AZ', '85062', NULL, 'Wesley', 'Alisha', 3, 522),
(94, 'Abbey Office Furnishings', '4150 W Shaw Ave', NULL, 'Fresno', 'CA', '93722', '(559) 555-8300', 'Francis', 'Kyra', 2, 150),
(95, 'Pacific Bell', NULL, NULL, 'Sacramento', 'CA', '95887', '(209) 555-7500', 'Nickalus', 'Kurt', 2, 522),
(96, 'Wells Fargo Bank', 'Business Mastercard', 'P.O. Box 29479', 'Phoenix', 'AZ', '85038', '(947) 555-3900', 'Damion', 'Mikayla', 2, 160),
(97, 'Compuserve', 'Dept L-742', NULL, 'Columbus', 'OH', '43260', '(614) 555-8600', 'Armando', 'Jan', 2, 572),
(98, 'American Express', 'Box 0001', NULL, 'Los Angeles', 'CA', '90096', '(800) 555-3344', 'Story', 'Kirsten', 2, 160),
(99, 'Bertelsmann Industry Svcs. Inc', '28210 N Avenue Stanford', NULL, 'Valencia', 'CA', '91355', '(805) 555-0584', 'Potter', 'Lance', 3, 400),
(100, 'Cahners Publishing Company', 'Citibank Lock Box 4026', '8725 W Sahara Zone 1127', 'The Lake', 'NV', '89163', '(301) 555-2162', 'Jacobsen', 'Samuel', 4, 540),
(101, 'California Business Machines', 'Gallery Plz', '5091 N Fresno', 'Fresno', 'CA', '93710', '(559) 555-5570', 'Rohansen', 'Anders', 2, 170),
(102, 'Coffee Break Service', 'PO Box 1091', NULL, 'Fresno', 'CA', '93714', '(559) 555-8700', 'Smitzen', 'Jeffrey', 4, 570),
(103, 'Dean Witter Reynolds', '9 River Pk Pl E 400', NULL, 'Boston', 'MA', '02134', '(508) 555-8737', 'Johnson', 'Vance', 5, 589),
(104, 'Digital Dreamworks', '5070 N Sixth Ste. 71', NULL, 'Fresno', 'CA', '93711', NULL, 'Elmert', 'Ron', 3, 589),
(105, 'Dristas Groom & McCormick', '7112 N Fresno St Ste 200', NULL, 'Fresno', 'CA', '93720', '(559) 555-8484', 'Aaronsen', 'Thom', 3, 591),
(106, 'Ford Motor Credit Company', 'Dept 0419', NULL, 'Los Angeles', 'CA', '90084', '(800) 555-7000', 'Snyder', 'Karen', 3, 582),
(107, 'Franchise Tax Board', 'PO Box 942857', NULL, 'Sacramento', 'CA', '94257', NULL, 'Prado', 'Anita', 4, 507),
(108, 'Gostanian General Building', '427 W Bedford #102', NULL, 'Fresno', 'CA', '93711', '(559) 555-5100', 'Bragg', 'Walter', 4, 523),
(109, 'Kent H Landsberg Co', 'File No 72686', 'PO Box 61000', 'San Francisco', 'CA', '94160', '(916) 555-8100', 'Stevens', 'Wendy', 3, 540),
(110, 'Malloy Lithographing Inc', '5411 Jackson Road', 'PO Box 1124', 'Ann Arbor', 'MI', '48106', '(313) 555-6113', 'Regging', 'Abe', 3, 400),
(111, 'Net Asset, Llc', '1315 Van Ness Ave Ste. 103', NULL, 'Fresno', 'CA', '93721', NULL, 'Kraggin', 'Laura', 1, 572),
(112, 'Office Depot', 'File No 81901', NULL, 'Los Angeles', 'CA', '90074', '(800) 555-1711', 'Pinsippi', 'Val', 3, 570),
(113, 'Pollstar', '4697 W Jacquelyn Ave', NULL, 'Fresno', 'CA', '93722', '(559) 555-2631', 'Aranovitch', 'Robert', 5, 520),
(114, 'Postmaster', 'Postage Due Technician', '1900 E Street', 'Fresno', 'CA', '93706', '(559) 555-7785', 'Finklestein', 'Fyodor', 1, 552),
(115, 'Roadway Package System, Inc', 'Dept La 21095', NULL, 'Pasadena', 'CA', '91185', NULL, 'Smith', 'Sam', 4, 553),
(116, 'State of California', 'Employment Development Dept', 'PO Box 826276', 'Sacramento', 'CA', '94230', '(209) 555-5132', 'Articunia', 'Mercedez', 1, 631),
(117, 'Suburban Propane', '2874 S Cherry Ave', NULL, 'Fresno', 'CA', '93706', '(559) 555-2770', 'Spivak', 'Harold', 3, 521),
(118, 'Unocal', 'P.O. Box 860070', NULL, 'Pasadena', 'CA', '91186', '(415) 555-7600', 'Bluzinski', 'Rachael', 3, 582),
(119, 'Yesmed, Inc', 'PO Box 2061', NULL, 'Fresno', 'CA', '93718', '(559) 555-0600', 'Hernandez', 'Reba', 2, 589),
(120, 'Dataforms/West', '1617 W. Shaw Avenue', 'Suite F', 'Fresno', 'CA', '93711', NULL, 'Church', 'Charlie', 3, 551),
(121, 'Zylka Design', '3467 W Shaw Ave #103', NULL, 'Fresno', 'CA', '93711', '(559) 555-8625', 'Ronaldsen', 'Jaime', 3, 403),
(122, 'United Parcel Service', 'P.O. Box 505820', NULL, 'Reno', 'NV', '88905', '(800) 555-0855', 'Beauregard', 'Violet', 3, 553),
(123, 'Federal Express Corporation', 'P.O. Box 1140', 'Dept A', 'Memphis', 'TN', '38101', '(800) 555-4091', 'Bucket', 'Charlie', 3, 553)
SET IDENTITY_INSERT Vendors OFF
GO

/****** Object:  Index IX_InvoiceLineItems_AccountNo  ******/
CREATE NONCLUSTERED INDEX IX_InvoiceLineItems_AccountNo ON InvoiceLineItems(
	AccountNo ASC
)
GO

/****** Object:  Index IX_InvoiceDate ******/
CREATE NONCLUSTERED INDEX IX_InvoiceDate ON Invoices(
	InvoiceDate DESC
)
GO

/****** Object:  Index IX_Invoices_TermsID ******/
CREATE NONCLUSTERED INDEX IX_Invoices_TermsID ON Invoices(
	TermsID ASC
)
GO

/****** Object:  Index IX_Invoices_VendorID ******/
CREATE NONCLUSTERED INDEX IX_Invoices_VendorID ON Invoices(
	VendorID ASC
)
GO

/****** Object:  Index IX_VendorName ******/
CREATE NONCLUSTERED INDEX IX_VendorName ON Vendors(
	VendorName ASC
)
GO

/****** Object:  Index IX_Vendors_AccountNo ******/
CREATE NONCLUSTERED INDEX IX_Vendors_AccountNo ON Vendors(
	DefaultAccountNo ASC
)
GO

/****** Object:  Index IX_Vendors_TermsID ******/
CREATE NONCLUSTERED INDEX IX_Vendors_TermsID ON Vendors(
	DefaultTermsID ASC
)
GO

ALTER TABLE Invoices
ADD  CONSTRAINT DF_Invoices_PaymentTotal  DEFAULT (0) FOR PaymentTotal
GO

ALTER TABLE Invoices
ADD  CONSTRAINT DF_Invoices_CreditTotal  DEFAULT (0) FOR CreditTotal
GO

ALTER TABLE Vendors
ADD  CONSTRAINT DF_Vendors_TermsID  DEFAULT (3) FOR DefaultTermsID
GO

ALTER TABLE Vendors
ADD  CONSTRAINT DF_Vendors_AccountNo  DEFAULT (570) FOR DefaultAccountNo
GO

ALTER TABLE InvoiceLineItems  
WITH NOCHECK ADD  CONSTRAINT FK_InvoiceLineItems_GLAccounts
FOREIGN KEY(AccountNo)
REFERENCES GLAccounts (AccountNo)
ON UPDATE CASCADE
GO

ALTER TABLE InvoiceLineItems
CHECK CONSTRAINT FK_InvoiceLineItems_GLAccounts
GO

ALTER TABLE InvoiceLineItems  
WITH NOCHECK
ADD CONSTRAINT FK_InvoiceLineItems_Invoices
FOREIGN KEY(InvoiceID)
REFERENCES Invoices (InvoiceID)
ON UPDATE CASCADE
ON DELETE CASCADE
GO

ALTER TABLE InvoiceLineItems
CHECK CONSTRAINT FK_InvoiceLineItems_Invoices
GO

ALTER TABLE Invoices  
WITH NOCHECK
ADD CONSTRAINT FK_Invoices_Terms
FOREIGN KEY(TermsID)
REFERENCES Terms (TermsID)
GO

ALTER TABLE Invoices CHECK CONSTRAINT FK_Invoices_Terms
GO

ALTER TABLE Invoices
WITH NOCHECK
ADD CONSTRAINT FK_Invoices_Vendors
FOREIGN KEY(VendorID)
REFERENCES Vendors (VendorID)
GO

ALTER TABLE Invoices CHECK CONSTRAINT FK_Invoices_Vendors
GO

ALTER TABLE Vendors
WITH NOCHECK
ADD CONSTRAINT FK_Vendors_GLAccounts
FOREIGN KEY(DefaultAccountNo)
REFERENCES GLAccounts (AccountNo)
GO

ALTER TABLE Vendors
CHECK CONSTRAINT FK_Vendors_GLAccounts
GO

ALTER TABLE Vendors
WITH NOCHECK
ADD CONSTRAINT FK_Vendors_Terms FOREIGN KEY(DefaultTermsID)
REFERENCES Terms (TermsID)
GO

ALTER TABLE Vendors CHECK CONSTRAINT FK_Vendors_Terms
GO

USE master
GO

ALTER DATABASE AP SET  READ_WRITE
GO
```

#### for `examples` Database

```SQL
USE master
GO

IF DB_ID('Examples') IS NOT NULL
    DROP DATABASE Examples
GO

/****** Object:  Database Examples    ******/
CREATE DATABASE Examples
GO

USE Examples
GO

/****** Object:  Table ActiveInvoices     ******/
CREATE TABLE ActiveInvoices(
	InvoiceID int IDENTITY(1,1) NOT NULL,
	VendorID int NOT NULL,
	InvoiceNumber varchar(50) NOT NULL,
	InvoiceDate smalldatetime NOT NULL,
	InvoiceTotal money NOT NULL,
	PaymentTotal money NOT NULL,
	CreditTotal money NOT NULL,
	TermsID int NOT NULL,
	InvoiceDueDate smalldatetime NOT NULL,
	PaymentDate smalldatetime NULL
)
GO

/****** Object:  Table APFlat     ******/
CREATE TABLE APFlat(
	VendorName varchar(50) NOT NULL,
	InvoiceNumber varchar(50) NOT NULL,
	ItemDescription1 varchar(100) NULL,
	ItemDescription2 varchar(100) NULL,
	ItemDescription3 varchar(100) NULL,
	ItemDescription4 varchar(100) NULL
)
GO

/****** Object:  Table Customers     ******/
CREATE TABLE Customers(
	CustID int IDENTITY(1,1) NOT NULL,
	CustomerLast nvarchar(30) NULL,
	CustomerFirst nvarchar(30) NULL,
	CustAddr nvarchar(60) NULL,
	CustCity nvarchar(15) NULL,
	CustState nvarchar(15) NULL,
	CustZip nvarchar(10) NULL,
	CustPhone nvarchar(24) NULL,
 CONSTRAINT PK_Customers PRIMARY KEY CLUSTERED(
	CustID ASC
 )
)
GO

/****** Object:  Table DateSample     ******/
CREATE TABLE DateSample(
	ID int IDENTITY(1,1) NOT NULL,
	StartDate datetime NULL
)
GO

/****** Object:  Table Departments     ******/
CREATE TABLE Departments(
	DeptNo int IDENTITY(1,1) NOT NULL,
	DeptName varchar(50) NOT NULL,
 CONSTRAINT PK_Departments PRIMARY KEY CLUSTERED(
	DeptNo ASC
 )
)
GO

/****** Object:  Table Employees     ******/
CREATE TABLE Employees(
	EmployeeID int IDENTITY(1,1) NOT NULL,
	LastName varchar(35) NOT NULL,
	FirstName varchar(35) NOT NULL,
	DeptNo int NOT NULL,
	ManagerID int NULL
)
GO

/****** Object:  Table EmployeesOld     ******/
CREATE TABLE EmployeesOld(
	EmployeeID int IDENTITY(1,1) NOT NULL,
	LastName varchar(35) NOT NULL,
	FirstName varchar(35) NOT NULL,
	DeptNo int NOT NULL,
 CONSTRAINT PK_Employees PRIMARY KEY CLUSTERED(
	EmployeeID ASC
 )
)
GO

/****** Object:  Table EmployeesTest    ******/
CREATE TABLE EmployeesTest(
	EmployeeID int IDENTITY(1,1) NOT NULL,
	LastName varchar(35) NOT NULL,
	FirstName varchar(35) NOT NULL,
	DeptNo int NOT NULL,
	ManagerID int NULL
)
GO

/****** Object:  Table Investors     ******/
CREATE TABLE Investors(
	InvestorID int IDENTITY(1,1) NOT NULL,
	LastName varchar(50) NULL,
	FirstName varchar(50) NULL,
	Address varchar(50) NULL,
	City varchar(50) NULL,
	State char(2) NULL,
	ZipCode char(10) NULL,
	Phone char(20) NULL,
	Investments money NULL,
	NetGain money NULL,
 CONSTRAINT PK_Investors PRIMARY KEY CLUSTERED(
	InvestorID ASC
 )
)
GO

/****** Object:  Table Invoices     ******/
CREATE TABLE Invoices(
	InvoiceID int NOT NULL,
	InvoiceNumber int NULL,
	InvoiceTotal smallmoney NULL
)
GO

/****** Object:  Table NullSample     ******/
CREATE TABLE NullSample(
	InvoiceID int IDENTITY(1,1) NOT NULL,
	InvoiceTotal money NULL
)
GO

/****** Object:  Table PaidInvoices    ******/
CREATE TABLE PaidInvoices(
	InvoiceID int IDENTITY(1,1) NOT NULL,
	VendorID int NOT NULL,
	InvoiceNumber varchar(50) NOT NULL,
	InvoiceDate smalldatetime NOT NULL,
	InvoiceTotal money NOT NULL,
	PaymentTotal money NOT NULL,
	CreditTotal money NOT NULL,
	TermsID int NOT NULL,
	InvoiceDueDate smalldatetime NOT NULL,
	PaymentDate smalldatetime NULL
)
GO

/****** Object:  Table Projects     ******/
CREATE TABLE Projects(
	ProjectNo char(5) NOT NULL,
	EmployeeID int NOT NULL
)
GO

/****** Object:  Table RealSample     ******/
CREATE TABLE RealSample(
	ID int IDENTITY(1,1) NOT NULL,
	R float NULL
)
GO

/****** Object:  Table SalesReps     ******/
CREATE TABLE SalesReps(
	RepID int IDENTITY(1,1) NOT NULL,
	RepFirstName varchar(50) NOT NULL,
	RepLastName varchar(50) NOT NULL,
 CONSTRAINT PK_SalesReps PRIMARY KEY CLUSTERED(
	RepID ASC
 )
)
GO

/****** Object:  Table SalesTotals    ******/
CREATE TABLE SalesTotals(
	RepID int NOT NULL,
	SalesYear char(4) NOT NULL,
	SalesTotal money NOT NULL,
 CONSTRAINT PK_SalesTotals PRIMARY KEY CLUSTERED(
	RepID ASC,
	SalesYear ASC
 )
)
GO

/****** Object:  Table Sample     ******/
CREATE TABLE Sample(
	ID int IDENTITY(1,1) NOT NULL,
	Number int NULL,
	Color char(10) NOT NULL
)
GO

/****** Object:  Table StringSample     ******/
CREATE TABLE StringSample(
	ID char(3) NULL,
	Name varchar(25) NULL,
	AltID char(3) NULL
)
GO

/****** Object:  Table Vendors     ******/
CREATE TABLE Vendors(
	VendorID int IDENTITY(1,1) NOT NULL,
	VendorName nvarchar(50) NOT NULL,
	VendorAddress1 nvarchar(50) NULL,
	VendorAddress2 nvarchar(50) NULL,
	VendorCity nvarchar(50) NOT NULL,
	VendorState nchar(2) NOT NULL,
	VendorZipCode nvarchar(10) NOT NULL,
	VendorContactLName nvarchar(35) NULL,
	VendorContactFName nvarchar(35) NULL,
	VendorPhone nvarchar(50) NULL,
	TermsID int NOT NULL,
	AccountNo int NOT NULL,
	LastTranDate smalldatetime NULL,
	YTDPurchases money NULL,
	YTDReturns money NULL,
	LastYTDPurchases money NULL,
	LastYTDReturns money NULL,
 CONSTRAINT PK_Vendors PRIMARY KEY CLUSTERED(
	VendorID ASC
 )
)
GO

SET IDENTITY_INSERT ActiveInvoices ON
INSERT ActiveInvoices (InvoiceID, VendorID, InvoiceNumber, InvoiceDate, InvoiceTotal, PaymentTotal, CreditTotal, TermsID, InvoiceDueDate, PaymentDate) VALUES
(89, 72, '39104', CAST('2016-03-10 00:00:00' AS SmallDateTime), 85.3100, 0.0000, 0.0000, 3, CAST('2016-04-09 00:00:00' AS SmallDateTime), NULL),
(94, 123, '963253264', CAST('2016-03-18 00:00:00' AS SmallDateTime), 52.2500, 0.0000, 0.0000, 3, CAST('2016-04-17 00:00:00' AS SmallDateTime), NULL),
(98, 83, '31361833', CAST('2016-03-21 00:00:00' AS SmallDateTime), 579.4200, 0.0000, 0.0000, 2, CAST('2016-04-10 00:00:00' AS SmallDateTime), NULL),
(99, 123, '263253268', CAST('2016-03-21 00:00:00' AS SmallDateTime), 59.9700, 0.0000, 0.0000, 3, CAST('2016-04-20 00:00:00' AS SmallDateTime), NULL),
(100, 123, '263253270', CAST('2016-03-22 00:00:00' AS SmallDateTime), 67.9200, 0.0000, 0.0000, 3, CAST('2016-04-21 00:00:00' AS SmallDateTime), NULL),
(101, 123, '263253273', CAST('2016-03-22 00:00:00' AS SmallDateTime), 30.7500, 0.0000, 0.0000, 3, CAST('2016-04-21 00:00:00' AS SmallDateTime), NULL),
(102, 110, 'P-0608', CAST('2016-03-23 00:00:00' AS SmallDateTime), 20551.1800, 0.0000, 1200.0000, 3, CAST('2016-04-22 00:00:00' AS SmallDateTime), NULL),
(105, 106, '9982771', CAST('2016-03-24 00:00:00' AS SmallDateTime), 503.2000, 0.0000, 0.0000, 3, CAST('2016-04-23 00:00:00' AS SmallDateTime), NULL),
(110, 80, '134116', CAST('2016-03-28 00:00:00' AS SmallDateTime), 90.3600, 0.0000, 0.0000, 2, CAST('2016-04-17 00:00:00' AS SmallDateTime), NULL),
(112, 110, '0-2436', CAST('2016-03-31 00:00:00' AS SmallDateTime), 10976.0600, 0.0000, 0.0000, 3, CAST('2016-04-30 00:00:00' AS SmallDateTime), NULL),
(113, 37, '547480102', CAST('2016-04-01 00:00:00' AS SmallDateTime), 224.0000, 0.0000, 0.0000, 3, CAST('2016-04-30 00:00:00' AS SmallDateTime), NULL)
SET IDENTITY_INSERT ActiveInvoices OFF
GO

INSERT APFlat (VendorName, InvoiceNumber, ItemDescription1, ItemDescription2, ItemDescription3, ItemDescription4) VALUES
('Wells Fargo Bank', '112897', 'DiCicco''s', 'Kinko''s', 'Office Max', 'Publishers Marketing'),
('Zylka Design', '97/522', 'MC Bouncebacks', 'SCMD Flyer', NULL, NULL),
('Zylka Design', '97/553B', 'Card revision', NULL, NULL, NULL)
GO

SET IDENTITY_INSERT Customers ON
INSERT Customers (CustID, CustomerLast, CustomerFirst, CustAddr, CustCity, CustState, CustZip, CustPhone) VALUES
(1, 'Anders', 'Maria', '345 Winchell Pl', 'Anderson', 'IN', '46014', '(765) 555-7878'),
(2, 'Trujillo', 'Ana', '1298 E Smathers St', 'Benton', 'AR', '72018', '(501) 555-7733'),
(3, 'Moreno', 'Antonio', '6925 N Parkland Ave', 'Puyallup', 'WA', '98373', '(253) 555-8332'),
(4, 'Hardy', 'Thomas', '83 d''Urberville Ln', 'Casterbridge', 'GA', '31209', '(478) 555-1139'),
(5, 'Berglund', 'Christina', '22717 E 73rd Ave', 'Dubuque', 'IA', '52004', '(319) 555-1139'),
(6, 'Moos', 'Hanna', '1778 N Bovine Ave', 'Peoria', 'IL', '61638', '(309) 555-8755'),
(7, 'Citeaux', 'Fred', '1234 Main St', 'Normal', 'IL', '61761', '(309) 555-1914'),
(8, 'Summer', 'Martin', '1877 Ete Ct', 'Frogtown', 'LA', '70563', '(337) 555-9441'),
(9, 'Lebihan', 'Laurence', '717 E Michigan Ave', 'Chicago', 'IL', '60611', '(312) 555-9441'),
(10, 'Lincoln', 'Elizabeth', '4562 Rt 78 E', 'Vancouver', 'WA', '98684', '(360) 555-2680'),
(32, 'Snyder', 'Howard', '2732 Baker Blvd.', 'Eugene', 'OR', '97403', '(503) 555-7555'),
(36, 'Latimer', 'Yoshi', 'City Center Plaza 516 Main St.', 'Elgin', 'OR', '97827', '(503) 555-6874'),
(43, 'Steel', 'John', '12 Orchestra Terrace', 'Walla Walla', 'WA', '99362', '(509) 555-7969'),
(45, 'Yorres', 'Jaime', '87 Polk St. Suite 5', 'San Francisco', 'CA', '94117', '(415) 555-5938'),
(48, 'Wilson', 'Fran', '89 Chiaroscuro Rd.', 'Portland', 'OR', '97219', '(503) 555-9573'),
(55, 'Phillips', 'Rene', '2743 Bering St.', 'Anchorage', 'AK', '99508', '(907) 555-7584'),
(65, 'Wilson', 'Paula', '2817 Milton Dr.', 'Albuquerque', 'NM', '87110', '(505) 555-5939'),
(71, 'Pavarotti', 'Jose', '187 Suffolk Ln.', 'Boise', 'ID', '83720', '(208) 555-8097'),
(75, 'Braunschweiger', 'Art', 'P.O. Box 555', 'Lander', 'WY', '82520', '(307) 555-4680'),
(77, 'Nixon', 'Liz', '89 Jefferson Way Suite 2', 'Providence', 'RI', '02909', '(401) 555-3612'),
(78, 'Wong', 'Liu', '55 Grizzly Peak Rd.', 'Butte', 'MT', '59801', '(406) 555-5834'),
(82, 'Nagy', 'Helvetius', '722 DaVinci Blvd.', 'Concord', 'MA', '01742', '(351) 555-1219'),
(89, 'Jablonski', 'Karl', '305 - 14th Ave. S. Suite 3B', 'Seattle', 'WA', '98128', '(206) 555-4112'),
(92, 'Chelan', 'Donna', '2299 E Baylor Dr', 'Dallas', 'TX', '75224', '(469) 555-8828')
SET IDENTITY_INSERT Customers OFF
GO

SET IDENTITY_INSERT DateSample ON
INSERT DateSample (ID, StartDate) VALUES
(1, CAST(0x0000762E00000000 AS DateTime)),
(2, CAST(0x000092B300000000 AS DateTime)),
(3, CAST(0x0000995D00000000 AS DateTime)),
(4, CAST(0x00009B4300A4CB80 AS DateTime)),
(5, CAST(0x00009F8A00E65057 AS DateTime)),
(6, CAST(0x00009F8E0094FAAC AS DateTime))
SET IDENTITY_INSERT DateSample OFF
GO

SET IDENTITY_INSERT Departments ON
INSERT Departments (DeptNo, DeptName) VALUES
(1, 'Accounting'),
(2, 'Payroll'),
(3, 'Operations'),
(4, 'Personnel'),
(5, 'Maintenance')
SET IDENTITY_INSERT Departments OFF
GO

SET IDENTITY_INSERT Employees ON
INSERT Employees (EmployeeID, LastName, FirstName, DeptNo, ManagerID) VALUES
(1, 'Smith', 'Cindy', 2, NULL),
(2, 'Jones', 'Elmer', 4, 1),
(3, 'Simonian', 'Ralph', 2, 2),
(4, 'Hernandez', 'Olivia', 1, 2),
(5, 'Aaronsen', 'Robert', 2, 3),
(6, 'Watson', 'Denise', 6, 3),
(7, 'Hardy', 'Thomas', 5, 2),
(8, 'O''Leary', 'Rhea', 4, 2),
(9, 'Locario', 'Paulo', 6, 1)
SET IDENTITY_INSERT Employees OFF
GO

SET IDENTITY_INSERT EmployeesOld ON
INSERT EmployeesOld (EmployeeID, LastName, FirstName, DeptNo) VALUES
(1, 'Smith', 'Cindy', 2),
(2, 'Jones', 'Elmer', 4),
(3, 'Simonian', 'Ralph', 2),
(4, 'Hernandez', 'Olivia', 1),
(5, 'Aaronsen', 'Robert', 2),
(6, 'Watson', 'Denise', 6),
(7, 'Hardy', 'Thomas', 5),
(8, 'O''Leary', 'Rhea', 4),
(9, 'Locario', 'Paulo', 6)
SET IDENTITY_INSERT EmployeesOld OFF
GO

SET IDENTITY_INSERT EmployeesTest ON
INSERT EmployeesTest (EmployeeID, LastName, FirstName, DeptNo, ManagerID) VALUES
(1, 'Smith', 'Cindy', 2, NULL),
(2, 'Jones', 'Elmer', 4, 1),
(3, 'Simonian', 'Ralph', 2, 2),
(4, 'Hernandez', 'Olivia', 1, 9),
(5, 'Aaronsen', 'Robert', 2, 4),
(6, 'Watson', 'Denise', 6, 8),
(7, 'Hardy', 'Thomas', 5, 2),
(8, 'O''Leary', 'Rhea', 4, 9),
(9, 'Locario', 'Paulo', 6, 1)
SET IDENTITY_INSERT EmployeesTest OFF
GO

SET IDENTITY_INSERT Investors ON
INSERT Investors (InvestorID, LastName, FirstName, Address, City, State, ZipCode, Phone, Investments, NetGain) VALUES
(1, 'Anders', 'Maria', '345 Winchell Pl', 'Anderson', 'IN', '46014     ', '(765) 555-7878      ', 15000.0000, 1242.5700),
(2, 'Trujilo', 'Ana', '1298 E Smathers St.', 'Benton', 'AR', '72018     ', '(510) 555-7733      ', 43500.0000, 8497.4400),
(3, 'Moreno', 'Antonio', '6925 N Parkland Ave.', 'Puyallup', 'WA', '98373     ', '(253) 555-8332      ', 22900.0000, 2338.8700),
(4, 'Hardy', 'Thomas', '83 d''Urberville Ln.', 'Casterbridge', 'GA', '31209     ', '(478) 555-1139      ', 5000.0000, -245.6900),
(5, 'Berglund', 'Christina', '22717 E 73rd Ave.', 'Dubuque', 'IA', '52004     ', '(319) 555-1139      ', 11750.0000, 865.7700)
SET IDENTITY_INSERT Investors OFF
GO

INSERT Invoices (InvoiceID, InvoiceNumber, InvoiceTotal) VALUES
(1, 8937, 50.0000),
(2, 3662, 100.0000),
(3, NULL, NULL),
(4, 4553, 250.0000),
(5, 8937, 0.0000)
GO

SET IDENTITY_INSERT NullSample ON
INSERT NullSample (InvoiceID, InvoiceTotal) VALUES
(1, 125.0000),
(2, 0.0000),
(3, NULL),
(4, 2199.9900),
(5, 0.0000)
SET IDENTITY_INSERT NullSample OFF
GO

SET IDENTITY_INSERT PaidInvoices ON
INSERT PaidInvoices (InvoiceID, VendorID, InvoiceNumber, InvoiceDate, InvoiceTotal, PaymentTotal, CreditTotal, TermsID, InvoiceDueDate, PaymentDate) VALUES
(1, 122, '989319-457', CAST('2015-12-08 00:00:00' AS SmallDateTime), 3813.3300, 3813.3300, 0.0000, 3, CAST('2016-01-08 00:00:00' AS SmallDateTime), CAST('2016-01-07 00:00:00' AS SmallDateTime)),
(2, 123, '263253241', CAST('2015-12-10 00:00:00' AS SmallDateTime), 40.2000, 40.2000, 0.0000, 3, CAST('2016-01-10 00:00:00' AS SmallDateTime), CAST('2016-01-14 00:00:00' AS SmallDateTime)),
(3, 123, '963253234', CAST('2015-12-13 00:00:00' AS SmallDateTime), 138.7500, 138.7500, 0.0000, 3, CAST('2016-01-13 00:00:00' AS SmallDateTime), CAST('2016-01-09 00:00:00' AS SmallDateTime)),
(4, 123, '2-000-2993', CAST('2015-12-16 00:00:00' AS SmallDateTime), 144.7000, 144.7000, 0.0000, 3, CAST('2016-01-16 00:00:00' AS SmallDateTime), CAST('2016-01-12 00:00:00' AS SmallDateTime)),
(5, 123, '963253251', CAST('2015-12-16 00:00:00' AS SmallDateTime), 15.5000, 15.5000, 0.0000, 3, CAST('2016-01-16 00:00:00' AS SmallDateTime), CAST('2016-01-11 00:00:00' AS SmallDateTime)),
(6, 123, '963253261', CAST('2015-12-16 00:00:00' AS SmallDateTime), 42.7500, 42.7500, 0.0000, 3, CAST('2016-01-16 00:00:00' AS SmallDateTime), CAST('2016-01-21 00:00:00' AS SmallDateTime)),
(7, 123, '963253237', CAST('2015-12-21 00:00:00' AS SmallDateTime), 172.5000, 172.5000, 0.0000, 3, CAST('2016-01-21 00:00:00' AS SmallDateTime), CAST('2016-01-22 00:00:00' AS SmallDateTime)),
(8, 89, '125520-1', CAST('2015-12-24 00:00:00' AS SmallDateTime), 95.0000, 95.0000, 0.0000, 1, CAST('2016-01-04 00:00:00' AS SmallDateTime), CAST('2016-01-01 00:00:00' AS SmallDateTime)),
(9, 121, '97/488', CAST('2015-12-24 00:00:00' AS SmallDateTime), 601.9500, 601.9500, 0.0000, 3, CAST('2016-01-24 00:00:00' AS SmallDateTime), CAST('2016-01-21 00:00:00' AS SmallDateTime)),
(10, 123, '263253250', CAST('2015-12-24 00:00:00' AS SmallDateTime), 42.6700, 42.6700, 0.0000, 3, CAST('2016-01-24 00:00:00' AS SmallDateTime), CAST('2016-01-22 00:00:00' AS SmallDateTime)),
(11, 123, '963253262', CAST('2015-12-25 00:00:00' AS SmallDateTime), 42.5000, 42.5000, 0.0000, 3, CAST('2016-01-25 00:00:00' AS SmallDateTime), CAST('2016-01-20 00:00:00' AS SmallDateTime)),
(12, 96, 'I77271-O01', CAST('2015-12-26 00:00:00' AS SmallDateTime), 662.0000, 662.0000, 0.0000, 2, CAST('2016-01-16 00:00:00' AS SmallDateTime), CAST('2016-01-13 00:00:00' AS SmallDateTime)),
(13, 95, '111-92R-10096', CAST('2015-12-30 00:00:00' AS SmallDateTime), 16.3300, 16.3300, 0.0000, 2, CAST('2016-01-20 00:00:00' AS SmallDateTime), CAST('2016-01-23 00:00:00' AS SmallDateTime)),
(14, 115, '25022117', CAST('2016-01-01 00:00:00' AS SmallDateTime), 6.0000, 6.0000, 0.0000, 4, CAST('2016-02-10 00:00:00' AS SmallDateTime), CAST('2016-02-10 00:00:00' AS SmallDateTime)),
(15, 48, 'P02-88D77S7', CAST('2016-01-03 00:00:00' AS SmallDateTime), 856.9200, 856.9200, 0.0000, 3, CAST('2016-02-02 00:00:00' AS SmallDateTime), CAST('2016-01-30 00:00:00' AS SmallDateTime)),
(16, 97, '21-4748363', CAST('2016-01-03 00:00:00' AS SmallDateTime), 9.9500, 9.9500, 0.0000, 2, CAST('2016-01-23 00:00:00' AS SmallDateTime), CAST('2016-01-22 00:00:00' AS SmallDateTime)),
(17, 123, '4-321-2596', CAST('2016-01-05 00:00:00' AS SmallDateTime), 10.0000, 10.0000, 0.0000, 3, CAST('2016-02-04 00:00:00' AS SmallDateTime), CAST('2016-02-05 00:00:00' AS SmallDateTime)),
(18, 123, '963253242', CAST('2016-01-06 00:00:00' AS SmallDateTime), 104.0000, 104.0000, 0.0000, 3, CAST('2016-02-05 00:00:00' AS SmallDateTime), CAST('2016-02-05 00:00:00' AS SmallDateTime)),
(19, 34, 'QP58872', CAST('2016-01-07 00:00:00' AS SmallDateTime), 116.5400, 116.5400, 0.0000, 1, CAST('2016-01-17 00:00:00' AS SmallDateTime), CAST('2016-01-19 00:00:00' AS SmallDateTime)),
(20, 115, '24863706', CAST('2016-01-10 00:00:00' AS SmallDateTime), 6.0000, 6.0000, 0.0000, 4, CAST('2016-02-19 00:00:00' AS SmallDateTime), CAST('2016-02-15 00:00:00' AS SmallDateTime)),
(21, 119, '10843', CAST('2016-01-11 00:00:00' AS SmallDateTime), 4901.2600, 4901.2600, 0.0000, 2, CAST('2016-01-31 00:00:00' AS SmallDateTime), CAST('2016-01-29 00:00:00' AS SmallDateTime)),
(22, 123, '963253235', CAST('2016-01-11 00:00:00' AS SmallDateTime), 108.2500, 108.2500, 0.0000, 3, CAST('2016-02-10 00:00:00' AS SmallDateTime), CAST('2016-02-09 00:00:00' AS SmallDateTime)),
(23, 97, '21-4923721', CAST('2016-01-13 00:00:00' AS SmallDateTime), 9.9500, 9.9500, 0.0000, 2, CAST('2016-02-02 00:00:00' AS SmallDateTime), CAST('2016-01-28 00:00:00' AS SmallDateTime)),
(24, 113, '77290', CAST('2016-01-13 00:00:00' AS SmallDateTime), 1750.0000, 1750.0000, 0.0000, 5, CAST('2016-03-02 00:00:00' AS SmallDateTime), CAST('2016-03-05 00:00:00' AS SmallDateTime)),
(25, 123, '963253246', CAST('2016-01-13 00:00:00' AS SmallDateTime), 129.0000, 129.0000, 0.0000, 3, CAST('2016-02-12 00:00:00' AS SmallDateTime), CAST('2016-02-09 00:00:00' AS SmallDateTime)),
(26, 123, '4-342-8069', CAST('2016-01-14 00:00:00' AS SmallDateTime), 10.0000, 10.0000, 0.0000, 3, CAST('2016-02-13 00:00:00' AS SmallDateTime), CAST('2016-02-13 00:00:00' AS SmallDateTime)),
(27, 88, '972110', CAST('2016-01-15 00:00:00' AS SmallDateTime), 207.7800, 207.7800, 0.0000, 1, CAST('2016-01-25 00:00:00' AS SmallDateTime), CAST('2016-01-27 00:00:00' AS SmallDateTime)),
(28, 123, '963253263', CAST('2016-01-16 00:00:00' AS SmallDateTime), 109.5000, 109.5000, 0.0000, 3, CAST('2016-02-15 00:00:00' AS SmallDateTime), CAST('2016-02-10 00:00:00' AS SmallDateTime)),
(29, 108, '121897', CAST('2016-01-19 00:00:00' AS SmallDateTime), 450.0000, 450.0000, 0.0000, 4, CAST('2016-02-28 00:00:00' AS SmallDateTime), CAST('2016-03-03 00:00:00' AS SmallDateTime)),
(30, 123, '1-200-5164', CAST('2016-01-20 00:00:00' AS SmallDateTime), 63.4000, 63.4000, 0.0000, 3, CAST('2016-02-19 00:00:00' AS SmallDateTime), CAST('2016-02-24 00:00:00' AS SmallDateTime)),
(31, 104, 'P02-3772', CAST('2016-01-21 00:00:00' AS SmallDateTime), 7125.3400, 7125.3400, 0.0000, 3, CAST('2016-02-20 00:00:00' AS SmallDateTime), CAST('2016-02-24 00:00:00' AS SmallDateTime)),
(32, 121, '97/486', CAST('2016-01-21 00:00:00' AS SmallDateTime), 953.1000, 953.1000, 0.0000, 3, CAST('2016-02-20 00:00:00' AS SmallDateTime), CAST('2016-02-22 00:00:00' AS SmallDateTime)),
(33, 105, '94007005', CAST('2016-01-23 00:00:00' AS SmallDateTime), 220.0000, 220.0000, 0.0000, 3, CAST('2016-02-22 00:00:00' AS SmallDateTime), CAST('2016-02-26 00:00:00' AS SmallDateTime)),
(34, 123, '963253232', CAST('2016-01-23 00:00:00' AS SmallDateTime), 127.7500, 127.7500, 0.0000, 3, CAST('2016-02-22 00:00:00' AS SmallDateTime), CAST('2016-02-18 00:00:00' AS SmallDateTime)),
(35, 107, 'RTR-72-3662-X', CAST('2016-01-25 00:00:00' AS SmallDateTime), 1600.0000, 1600.0000, 0.0000, 4, CAST('2016-03-04 00:00:00' AS SmallDateTime), CAST('2016-03-09 00:00:00' AS SmallDateTime)),
(36, 121, '97/465', CAST('2016-01-25 00:00:00' AS SmallDateTime), 565.1500, 565.1500, 0.0000, 3, CAST('2016-02-24 00:00:00' AS SmallDateTime), CAST('2016-02-24 00:00:00' AS SmallDateTime)),
(37, 123, '963253260', CAST('2016-01-25 00:00:00' AS SmallDateTime), 36.0000, 36.0000, 0.0000, 3, CAST('2016-02-24 00:00:00' AS SmallDateTime), CAST('2016-02-26 00:00:00' AS SmallDateTime)),
(38, 123, '963253272', CAST('2016-01-26 00:00:00' AS SmallDateTime), 61.5000, 61.5000, 0.0000, 3, CAST('2016-02-25 00:00:00' AS SmallDateTime), CAST('2016-02-28 00:00:00' AS SmallDateTime)),
(39, 110, '0-2058', CAST('2016-01-28 00:00:00' AS SmallDateTime), 37966.1900, 37966.1900, 0.0000, 3, CAST('2016-02-27 00:00:00' AS SmallDateTime), CAST('2016-02-28 00:00:00' AS SmallDateTime)),
(40, 121, '97/503', CAST('2016-01-30 00:00:00' AS SmallDateTime), 639.7700, 639.7700, 0.0000, 3, CAST('2016-02-28 00:00:00' AS SmallDateTime), CAST('2016-02-25 00:00:00' AS SmallDateTime)),
(41, 123, '963253255', CAST('2016-01-31 00:00:00' AS SmallDateTime), 53.7500, 53.7500, 0.0000, 3, CAST('2016-02-28 00:00:00' AS SmallDateTime), CAST('2016-02-27 00:00:00' AS SmallDateTime)),
(42, 123, '94007069', CAST('2016-01-31 00:00:00' AS SmallDateTime), 400.0000, 400.0000, 0.0000, 3, CAST('2016-02-28 00:00:00' AS SmallDateTime), CAST('2016-03-01 00:00:00' AS SmallDateTime)),
(43, 72, '40318', CAST('2016-02-01 00:00:00' AS SmallDateTime), 21842.0000, 21842.0000, 0.0000, 3, CAST('2016-03-01 00:00:00' AS SmallDateTime), CAST('2016-02-28 00:00:00' AS SmallDateTime)),
(44, 95, '111-92R-10094', CAST('2016-02-01 00:00:00' AS SmallDateTime), 19.6700, 19.6700, 0.0000, 2, CAST('2016-02-21 00:00:00' AS SmallDateTime), CAST('2016-02-24 00:00:00' AS SmallDateTime)),
(45, 122, '989319-437', CAST('2016-02-01 00:00:00' AS SmallDateTime), 2765.3600, 2765.3600, 0.0000, 3, CAST('2016-03-01 00:00:00' AS SmallDateTime), CAST('2016-02-28 00:00:00' AS SmallDateTime)),
(46, 37, '547481328', CAST('2016-02-03 00:00:00' AS SmallDateTime), 224.0000, 224.0000, 0.0000, 3, CAST('2016-03-03 00:00:00' AS SmallDateTime), CAST('2016-03-04 00:00:00' AS SmallDateTime)),
(47, 83, '31359783', CAST('2016-02-03 00:00:00' AS SmallDateTime), 1575.0000, 1575.0000, 0.0000, 2, CAST('2016-02-23 00:00:00' AS SmallDateTime), CAST('2016-02-21 00:00:00' AS SmallDateTime)),
(48, 123, '1-202-2978', CAST('2016-02-03 00:00:00' AS SmallDateTime), 33.0000, 33.0000, 0.0000, 3, CAST('2016-03-03 00:00:00' AS SmallDateTime), CAST('2016-03-05 00:00:00' AS SmallDateTime)),
(49, 95, '111-92R-10097', CAST('2016-02-04 00:00:00' AS SmallDateTime), 16.3300, 16.3300, 0.0000, 2, CAST('2016-02-24 00:00:00' AS SmallDateTime), CAST('2016-02-26 00:00:00' AS SmallDateTime)),
(50, 37, '547479217', CAST('2016-02-07 00:00:00' AS SmallDateTime), 116.0000, 116.0000, 0.0000, 3, CAST('2016-03-07 00:00:00' AS SmallDateTime), CAST('2016-03-07 00:00:00' AS SmallDateTime)),
(51, 122, '989319-477', CAST('2016-02-08 00:00:00' AS SmallDateTime), 2184.1100, 2184.1100, 0.0000, 3, CAST('2016-03-08 00:00:00' AS SmallDateTime), CAST('2016-03-08 00:00:00' AS SmallDateTime)),
(52, 34, 'Q545443', CAST('2016-02-09 00:00:00' AS SmallDateTime), 1083.5800, 1083.5800, 0.0000, 1, CAST('2016-02-19 00:00:00' AS SmallDateTime), CAST('2016-02-23 00:00:00' AS SmallDateTime)),
(53, 95, '111-92R-10092', CAST('2016-02-09 00:00:00' AS SmallDateTime), 46.2100, 46.2100, 0.0000, 2, CAST('2016-02-28 00:00:00' AS SmallDateTime), CAST('2016-03-02 00:00:00' AS SmallDateTime)),
(54, 121, '97/553B', CAST('2016-02-10 00:00:00' AS SmallDateTime), 313.5500, 313.5500, 0.0000, 3, CAST('2016-03-10 00:00:00' AS SmallDateTime), CAST('2016-03-09 00:00:00' AS SmallDateTime)),
(55, 123, '963253245', CAST('2016-02-10 00:00:00' AS SmallDateTime), 40.7500, 40.7500, 0.0000, 3, CAST('2016-03-10 00:00:00' AS SmallDateTime), CAST('2016-03-12 00:00:00' AS SmallDateTime)),
(56, 86, '367447', CAST('2016-02-11 00:00:00' AS SmallDateTime), 2433.0000, 2433.0000, 0.0000, 1, CAST('2016-02-21 00:00:00' AS SmallDateTime), CAST('2016-02-17 00:00:00' AS SmallDateTime)),
(57, 103, '75C-90227', CAST('2016-02-11 00:00:00' AS SmallDateTime), 1367.5000, 1367.5000, 0.0000, 5, CAST('2016-03-31 00:00:00' AS SmallDateTime), CAST('2016-03-31 00:00:00' AS SmallDateTime)),
(58, 123, '963253256', CAST('2016-02-11 00:00:00' AS SmallDateTime), 53.2500, 53.2500, 0.0000, 3, CAST('2016-03-11 00:00:00' AS SmallDateTime), CAST('2016-03-07 00:00:00' AS SmallDateTime)),
(59, 123, '4-314-3057', CAST('2016-02-11 00:00:00' AS SmallDateTime), 13.7500, 13.7500, 0.0000, 3, CAST('2016-03-11 00:00:00' AS SmallDateTime), CAST('2016-03-15 00:00:00' AS SmallDateTime)),
(60, 122, '989319-497', CAST('2016-02-12 00:00:00' AS SmallDateTime), 2312.2000, 2312.2000, 0.0000, 3, CAST('2016-03-12 00:00:00' AS SmallDateTime), CAST('2016-03-09 00:00:00' AS SmallDateTime)),
(61, 115, '24946731', CAST('2016-02-15 00:00:00' AS SmallDateTime), 25.6700, 25.6700, 0.0000, 4, CAST('2016-03-25 00:00:00' AS SmallDateTime), CAST('2016-03-26 00:00:00' AS SmallDateTime)),
(62, 123, '963253269', CAST('2016-02-15 00:00:00' AS SmallDateTime), 26.7500, 26.7500, 0.0000, 3, CAST('2016-03-15 00:00:00' AS SmallDateTime), CAST('2016-03-11 00:00:00' AS SmallDateTime)),
(63, 122, '989319-427', CAST('2016-02-16 00:00:00' AS SmallDateTime), 2115.8100, 2115.8100, 0.0000, 3, CAST('2016-03-16 00:00:00' AS SmallDateTime), CAST('2016-03-19 00:00:00' AS SmallDateTime)),
(64, 123, '963253267', CAST('2016-02-17 00:00:00' AS SmallDateTime), 23.5000, 23.5000, 0.0000, 3, CAST('2016-03-17 00:00:00' AS SmallDateTime), CAST('2016-03-19 00:00:00' AS SmallDateTime)),
(65, 99, '509786', CAST('2016-02-18 00:00:00' AS SmallDateTime), 6940.2500, 6940.2500, 0.0000, 3, CAST('2016-03-18 00:00:00' AS SmallDateTime), CAST('2016-03-15 00:00:00' AS SmallDateTime)),
(66, 123, '263253253', CAST('2016-02-18 00:00:00' AS SmallDateTime), 31.9500, 31.9500, 0.0000, 3, CAST('2016-03-18 00:00:00' AS SmallDateTime), CAST('2016-03-21 00:00:00' AS SmallDateTime)),
(67, 122, '989319-487', CAST('2016-02-20 00:00:00' AS SmallDateTime), 1927.5400, 1927.5400, 0.0000, 3, CAST('2016-03-20 00:00:00' AS SmallDateTime), CAST('2016-03-18 00:00:00' AS SmallDateTime)),
(68, 81, 'MABO1489', CAST('2016-02-21 00:00:00' AS SmallDateTime), 936.9300, 936.9300, 0.0000, 2, CAST('2016-03-11 00:00:00' AS SmallDateTime), CAST('2016-03-10 00:00:00' AS SmallDateTime)),
(69, 80, '133560', CAST('2016-02-22 00:00:00' AS SmallDateTime), 175.0000, 175.0000, 0.0000, 2, CAST('2016-03-12 00:00:00' AS SmallDateTime), CAST('2016-03-16 00:00:00' AS SmallDateTime)),
(70, 115, '24780512', CAST('2016-02-22 00:00:00' AS SmallDateTime), 6.0000, 6.0000, 0.0000, 4, CAST('2016-04-01 00:00:00' AS SmallDateTime), CAST('2016-03-29 00:00:00' AS SmallDateTime)),
(71, 123, '963253254', CAST('2016-02-22 00:00:00' AS SmallDateTime), 108.5000, 108.5000, 0.0000, 3, CAST('2016-03-22 00:00:00' AS SmallDateTime), CAST('2016-03-20 00:00:00' AS SmallDateTime)),
(72, 123, '43966316', CAST('2016-02-22 00:00:00' AS SmallDateTime), 10.0000, 10.0000, 0.0000, 3, CAST('2016-03-22 00:00:00' AS SmallDateTime), CAST('2016-03-17 00:00:00' AS SmallDateTime)),
(73, 114, 'CBM9920-M-T77109', CAST('2016-02-23 00:00:00' AS SmallDateTime), 290.0000, 290.0000, 0.0000, 1, CAST('2016-03-03 00:00:00' AS SmallDateTime), CAST('2016-02-28 00:00:00' AS SmallDateTime)),
(74, 102, '109596', CAST('2016-02-24 00:00:00' AS SmallDateTime), 41.8000, 41.8000, 0.0000, 4, CAST('2016-04-03 00:00:00' AS SmallDateTime), CAST('2016-04-04 00:00:00' AS SmallDateTime)),
(75, 123, '7548906-20', CAST('2016-02-24 00:00:00' AS SmallDateTime), 27.0000, 27.0000, 0.0000, 3, CAST('2016-03-24 00:00:00' AS SmallDateTime), CAST('2016-03-24 00:00:00' AS SmallDateTime)),
(76, 123, '963253248', CAST('2016-02-24 00:00:00' AS SmallDateTime), 241.0000, 241.0000, 0.0000, 3, CAST('2016-03-24 00:00:00' AS SmallDateTime), CAST('2016-03-25 00:00:00' AS SmallDateTime)),
(77, 121, '97/553', CAST('2016-02-25 00:00:00' AS SmallDateTime), 904.1400, 904.1400, 0.0000, 3, CAST('2016-03-25 00:00:00' AS SmallDateTime), CAST('2016-03-25 00:00:00' AS SmallDateTime)),
(78, 121, '97/522', CAST('2016-02-28 00:00:00' AS SmallDateTime), 1962.1300, 1762.1300, 200.0000, 3, CAST('2016-03-28 00:00:00' AS SmallDateTime), CAST('2016-03-30 00:00:00' AS SmallDateTime)),
(79, 100, '587056', CAST('2016-02-28 00:00:00' AS SmallDateTime), 2184.5000, 2184.5000, 0.0000, 4, CAST('2016-04-09 00:00:00' AS SmallDateTime), CAST('2016-04-07 00:00:00' AS SmallDateTime)),
(80, 122, '989319-467', CAST('2016-03-01 00:00:00' AS SmallDateTime), 2318.0300, 2318.0300, 0.0000, 3, CAST('2016-03-31 00:00:00' AS SmallDateTime), CAST('2016-03-29 00:00:00' AS SmallDateTime)),
(81, 123, '263253265', CAST('2016-03-02 00:00:00' AS SmallDateTime), 26.2500, 26.2500, 0.0000, 3, CAST('2016-04-01 00:00:00' AS SmallDateTime), CAST('2016-03-28 00:00:00' AS SmallDateTime)),
(82, 94, '203339-13', CAST('2016-03-05 00:00:00' AS SmallDateTime), 17.5000, 17.5000, 0.0000, 2, CAST('2016-03-25 00:00:00' AS SmallDateTime), CAST('2016-03-27 00:00:00' AS SmallDateTime)),
(83, 95, '111-92R-10093', CAST('2016-03-06 00:00:00' AS SmallDateTime), 39.7700, 39.7700, 0.0000, 2, CAST('2016-03-26 00:00:00' AS SmallDateTime), CAST('2016-03-22 00:00:00' AS SmallDateTime)),
(84, 123, '963253258', CAST('2016-03-06 00:00:00' AS SmallDateTime), 111.0000, 111.0000, 0.0000, 3, CAST('2016-04-05 00:00:00' AS SmallDateTime), CAST('2016-04-05 00:00:00' AS SmallDateTime)),
(85, 123, '963253271', CAST('2016-03-07 00:00:00' AS SmallDateTime), 158.0000, 158.0000, 0.0000, 3, CAST('2016-04-06 00:00:00' AS SmallDateTime), CAST('2016-04-11 00:00:00' AS SmallDateTime)),
(86, 123, '963253230', CAST('2016-03-07 00:00:00' AS SmallDateTime), 739.2000, 739.2000, 0.0000, 3, CAST('2016-04-06 00:00:00' AS SmallDateTime), CAST('2016-04-06 00:00:00' AS SmallDateTime)),
(87, 123, '963253244', CAST('2016-03-08 00:00:00' AS SmallDateTime), 60.0000, 60.0000, 0.0000, 3, CAST('2016-04-07 00:00:00' AS SmallDateTime), CAST('2016-04-09 00:00:00' AS SmallDateTime)),
(88, 123, '963253239', CAST('2016-03-08 00:00:00' AS SmallDateTime), 147.2500, 147.2500, 0.0000, 3, CAST('2016-04-07 00:00:00' AS SmallDateTime), CAST('2016-04-11 00:00:00' AS SmallDateTime)),
(90, 123, '963253252', CAST('2016-03-12 00:00:00' AS SmallDateTime), 38.7500, 38.7500, 0.0000, 3, CAST('2016-04-11 00:00:00' AS SmallDateTime), CAST('2016-04-11 00:00:00' AS SmallDateTime)),
(91, 95, '111-92R-10095', CAST('2016-03-15 00:00:00' AS SmallDateTime), 32.7000, 32.7000, 0.0000, 2, CAST('2016-04-04 00:00:00' AS SmallDateTime), CAST('2016-04-06 00:00:00' AS SmallDateTime)),
(92, 117, '111897', CAST('2016-03-15 00:00:00' AS SmallDateTime), 16.6200, 16.6200, 0.0000, 3, CAST('2016-04-14 00:00:00' AS SmallDateTime), CAST('2016-04-14 00:00:00' AS SmallDateTime)),
(93, 123, '4-327-7357', CAST('2016-03-16 00:00:00' AS SmallDateTime), 162.7500, 162.7500, 0.0000, 3, CAST('2016-04-15 00:00:00' AS SmallDateTime), CAST('2016-04-11 00:00:00' AS SmallDateTime)),
(95, 82, 'C73-24', CAST('2016-03-19 00:00:00' AS SmallDateTime), 600.0000, 600.0000, 0.0000, 2, CAST('2016-04-08 00:00:00' AS SmallDateTime), CAST('2016-04-13 00:00:00' AS SmallDateTime)),
(96, 110, 'P-0259', CAST('2016-03-19 00:00:00' AS SmallDateTime), 26881.4000, 26881.4000, 0.0000, 3, CAST('2016-04-18 00:00:00' AS SmallDateTime), CAST('2016-04-20 00:00:00' AS SmallDateTime)),
(97, 90, '97-1024A', CAST('2016-03-20 00:00:00' AS SmallDateTime), 356.4800, 356.4800, 0.0000, 2, CAST('2016-04-09 00:00:00' AS SmallDateTime), CAST('2016-04-07 00:00:00' AS SmallDateTime)),
(103, 122, '989319-417', CAST('2016-03-23 00:00:00' AS SmallDateTime), 2051.5900, 2051.5900, 0.0000, 3, CAST('2016-04-22 00:00:00' AS SmallDateTime), CAST('2016-04-24 00:00:00' AS SmallDateTime)),
(104, 123, '263253243', CAST('2016-03-23 00:00:00' AS SmallDateTime), 44.4400, 44.4400, 0.0000, 3, CAST('2016-04-22 00:00:00' AS SmallDateTime), CAST('2016-04-24 00:00:00' AS SmallDateTime)),
(106, 110, '0-2060', CAST('2016-03-24 00:00:00' AS SmallDateTime), 23517.5800, 21221.6300, 2295.9500, 3, CAST('2016-04-23 00:00:00' AS SmallDateTime), CAST('2016-04-27 00:00:00' AS SmallDateTime)),
(107, 122, '989319-447', CAST('2016-03-24 00:00:00' AS SmallDateTime), 3689.9900, 3689.9900, 0.0000, 3, CAST('2016-04-23 00:00:00' AS SmallDateTime), CAST('2016-04-19 00:00:00' AS SmallDateTime)),
(108, 123, '963253240', CAST('2016-03-24 00:00:00' AS SmallDateTime), 67.0000, 67.0000, 0.0000, 3, CAST('2016-04-23 00:00:00' AS SmallDateTime), CAST('2016-04-23 00:00:00' AS SmallDateTime)),
(109, 121, '97/222', CAST('2016-03-25 00:00:00' AS SmallDateTime), 1000.4600, 1000.4600, 0.0000, 3, CAST('2016-04-24 00:00:00' AS SmallDateTime), CAST('2016-04-22 00:00:00' AS SmallDateTime)),
(111, 123, '263253257', CAST('2016-03-30 00:00:00' AS SmallDateTime), 22.5700, 22.5700, 0.0000, 3, CAST('2016-04-29 00:00:00' AS SmallDateTime), CAST('2016-05-03 00:00:00' AS SmallDateTime)),
(114, 123, '963253249', CAST('2016-04-02 00:00:00' AS SmallDateTime), 127.7500, 127.7500, 0.0000, 3, CAST('2016-05-01 00:00:00' AS SmallDateTime), CAST('2016-05-04 00:00:00' AS SmallDateTime))
SET IDENTITY_INSERT PaidInvoices OFF
GO

INSERT Projects (ProjectNo, EmployeeID) VALUES
('P1011', 8),
('P1011', 4),
('P1012', 3),
('P1012', 1),
('P1012', 5),
('P1013', 6),
('P1013', 9),
('P1014', 10)
GO

SET IDENTITY_INSERT RealSample ON
INSERT RealSample (ID, R) VALUES
(1, 1.0000000000000011),
(2, 1),
(3, 0.999999999999999),
(4, 1234.56789012345),
(5, 999.04440209348),
(6, 24.04849)
SET IDENTITY_INSERT RealSample OFF
GO

SET IDENTITY_INSERT SalesReps ON
INSERT SalesReps (RepID, RepFirstName, RepLastName) VALUES
(1, 'Jonathon', 'Thomas'),
(2, 'Sonja', 'Martinez'),
(3, 'Andrew', 'Markasian'),
(4, 'Phillip', 'Winters'),
(5, 'Lydia', 'Kramer')
SET IDENTITY_INSERT SalesReps OFF
GO

INSERT SalesTotals (RepID, SalesYear, SalesTotal) VALUES
(1, '2014', 1274856.3800),
(1, '2015', 923746.8500),
(1, '2016', 998337.4600),
(2, '2014', 978465.9900),
(2, '2015', 974853.8100),
(2, '2016', 887695.7500),
(3, '2014', 1032875.4800),
(3, '2015', 1132744.5600),
(4, '2015', 655786.9200),
(4, '2016', 72443.3700),
(5, '2015', 422847.8600),
(5, '2016', 45182.4400)
GO

SET IDENTITY_INSERT Sample ON
INSERT Sample (ID, Number, Color) VALUES
(1, 8937, 'Brown     '),
(2, 3662, 'Blue      '),
(3, NULL, 'Red       '),
(4, 4553, 'Green     '),
(5, 8937, 'Yellow    '),
(6, 606, 'Purple    '),
(7, NULL, 'Orange    '),
(8, 808, 'None      '),
(9, NULL, 'None      '),
(10, NULL, 'None      '),
(11, NULL, 'None      ')
SET IDENTITY_INSERT Sample OFF
GO

INSERT StringSample (ID, Name, AltID) VALUES
('1  ', 'Lizbeth Darien', '01 '),
('2  ', 'Darnell O''Sullivan', '02 '),
('17 ', 'Lance Pinos-Potter', '17 '),
('20 ', 'Jean Paul Renard', '20 '),
('3  ', 'Alisha von Strump', '03 ')
GO

SET IDENTITY_INSERT Vendors ON
INSERT Vendors (VendorID, VendorName, VendorAddress1, VendorAddress2, VendorCity, VendorState, VendorZipCode, VendorContactLName, VendorContactFName, VendorPhone, TermsID, AccountNo, LastTranDate, YTDPurchases, YTDReturns, LastYTDPurchases, LastYTDReturns) VALUES
(1, 'US Postal Service', 'Attn:  Supt. Window Services', 'PO Box 7005', 'Madison', 'WI', '53707', 'Alberto', 'Francesco', '(209) 555-1205', 1, 552, CAST('2009-03-23 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 0.0000, 0.0000),
(2, 'National Information Data Ctr', 'PO Box 96621', '', 'Washington', 'DC', '20110', 'Irvin', 'Ania', '(301) 555-8950', 3, 540, CAST('2012-11-11 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 0.0000, 0.0000),
(3, 'Register of Copyrights', 'Library Of Congress', '', 'Washington', 'DC', '20559', 'Liana', 'Lukas', '', 3, 403, CAST('2012-11-28 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 0.0000, 0.0000),
(4, 'Jobtrak', '1990 Westwood Blvd Ste 260', '', 'Los Angeles', 'CA', '90025', 'Quinn', 'Kenzie', '(800) 555-8725', 3, 572, CAST('2012-12-10 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 0.0000, 0.0000),
(5, 'Newbrige Book Clubs', '3000 Cindel Drive', '', 'Delran', 'NJ', '08370', 'Marks', 'Michelle', '(800) 555-9980', 4, 394, CAST('2013-02-25 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 0.0000, 0.0000),
(6, 'California Chamber Of Commerce', '3255 Ramos Cir', '', 'Sacramento', 'CA', '95827', 'Mauro', 'Anton', '(916) 555-6670', 3, 572, CAST('2013-03-29 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 0.0000, 0.0000),
(7, 'Towne Advertiser''s Mailing Svcs', 'Kevin Minder', '3441 W Macarthur Blvd', 'Santa Ana', 'CA', '92704', 'Maegen', 'Ted', '', 3, 540, CAST('2013-08-24 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 0.0000, 0.0000),
(8, 'BFI Industries', 'PO Box 9369', '', 'Fresno', 'CA', '93792', 'Kaleigh', 'Erick', '(209) 555-1551', 3, 521, CAST('2014-01-08 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 0.0000, 0.0000),
(9, 'Pacific Gas & Electric', 'Box 52001', '', 'San Francisco', 'CA', '94152', 'Anthoni', 'Kaitlyn', '(209) 555-6081', 3, 521, CAST('2014-01-08 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 0.0000, 0.0000),
(10, 'Robbins Mobile Lock And Key', '4669 N Fresno', '', 'Fresno', 'CA', '93726', 'Leigh', 'Bill', '(209) 555-9375', 2, 523, CAST('2014-01-08 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 0.0000, 0.0000),
(11, 'Bill Marvin Electric Inc', '4583 E Home', '', 'Fresno', 'CA', '93703', 'Hostlery', 'Kaitlin', '(209) 555-5106', 2, 523, CAST('2014-01-15 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 0.0000, 0.0000),
(12, 'City Of Fresno', 'PO Box 2069', '', 'Fresno', 'CA', '93718', 'Mayte', 'Kendall', '(209) 555-9999', 3, 574, CAST('2014-01-15 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 0.0000, 0.0000),
(13, 'Golden Eagle Insurance Co', 'PO Box 85826', '', 'San Diego', 'CA', '92186', 'Blanca', 'Korah', '', 3, 590, CAST('2014-01-21 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 0.0000, 0.0000),
(14, 'Expedata Inc', '4420 N. First Street, Suite 108', '', 'Fresno', 'CA', '93726', 'Quintin', 'Marvin', '(209) 555-9586', 3, 589, CAST('2014-02-11 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 6.9500, 0.0000),
(15, 'ASC Signs', '1528 N Sierra Vista', '', 'Fresno', 'CA', '93703', 'Darien', 'Elisabeth', '', 1, 546, CAST('2014-02-25 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 108.0000, 0.0000),
(16, 'Internal Revenue Service', '', '', 'Fresno', 'CA', '93888', 'Aileen', 'Joan', '', 1, 235, CAST('2014-03-09 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 86.0700, 0.0000),
(17, 'Blanchard & Johnson Associates', '27371 Valderas', '', 'Mission Viejo', 'CA', '92691', 'Keeton', 'Gonzalo', '(214) 555-3647', 3, 540, CAST('2014-03-23 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 354.2700, 0.0000),
(18, 'Fresno Photoengraving Company', '1952 "H" Street', 'P.O. Box 1952', 'Fresno', 'CA', '93718', 'Chaddick', 'Derek', '(209) 555-3005', 3, 403, CAST('2014-03-23 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 50.3700, 0.0000),
(19, 'Crown Printing', '1730 "H" St', '', 'Fresno', 'CA', '93721', 'Randrup', 'Leann', '(209) 555-7473', 2, 400, CAST('2014-04-14 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 3016.1200, 0.0000),
(20, 'Diversified Printing & Pub', '2632 Saturn St', '', 'Brea', 'CA', '92621', 'Lane', 'Vanesa', '(714) 555-4541', 3, 400, CAST('2014-04-14 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 8805.7300, 0.0000),
(21, 'The Library Ltd', '7700 Forsyth', '', 'St Louis', 'MO', '63105', 'Marques', 'Malia', '(314) 555-8834', 3, 540, CAST('2014-07-26 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 100.0000, 0.0000),
(22, 'Micro Center', '1555 W Lane Ave', '', 'Columbus', 'OH', '43221', 'Evan', 'Emily', '(614) 555-4435', 2, 160, CAST('2014-08-10 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 1200.0000, 0.0000),
(23, 'Yale Industrial Trucks-Fresno', '3711 W Franklin', '', 'Fresno', 'CA', '93706', 'Alexis', 'Alexandro', '(209) 555-2993', 3, 532, CAST('2014-08-29 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 479.6300, 0.0000),
(24, 'Zee Medical Service Co', '4221 W Sierra Madre #104', '', 'Fresno', 'CA', '93722', 'Hallie', 'Juliana', '', 3, 570, CAST('2014-08-29 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 84.7800, 0.0000),
(25, 'California Data Marketing', '2818 E Hamilton', '', 'Fresno', 'CA', '93721', 'Jonessen', 'Moises', '(209) 555-3801', 4, 540, CAST('2014-09-27 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 2577.8600, 0.0000),
(26, 'Small Press', '121 E Front St - 4th Floor', '', 'Traverse City', 'MI', '49684', 'Colette', 'Dusty', '', 3, 540, CAST('2014-11-01 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 34.0000, 0.0000),
(27, 'Rich Advertising', '12 Daniel Road', '', 'Fairfield', 'NJ', '07004', 'Neil', 'Ingrid', '(201) 555-9742', 3, 540, CAST('2014-11-29 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 3000.0000, 0.0000),
(29, 'Vision Envelope & Printing', 'PO Box 3100', '', 'Gardena', 'CA', '90247', 'Raven', 'Jamari', '(310) 555-7062', 3, 551, CAST('2014-12-12 00:00:00' AS SmallDateTime), 0.0000, 0.0000, 1248.8300, 0.0000),
(30, 'Costco', 'Fresno Warehouse', '4500 W Shaw', 'Fresno', 'CA', '93711', 'Jaquan', 'Aaron', '', 3, 570, CAST('2015-02-02 00:00:00' AS SmallDateTime), 140.0000, 0.0000, 0.0000, 0.0000),
(31, 'Enterprise Communications Inc', '1483 Chain Bridge Rd, Ste 202', '', 'Mclean', 'VA', '22101', 'Lawrence', 'Eileen', '(770) 555-9558', 2, 536, CAST('2015-02-21 00:00:00' AS SmallDateTime), 500.0000, 0.0000, 0.0000, 0.0000),
(32, 'RR Bowker', 'PO Box 31', '', 'New Providence', 'NJ', '07974', 'Essence', 'Marjorie', '(800) 555-8110', 3, 532, CAST('2015-02-21 00:00:00' AS SmallDateTime), 272.4800, 0.0000, 0.0000, 0.0000),
(33, 'Nielson', 'Ohio Valley Litho Division', 'Location #0470', 'Cincinnati', 'OH', '45264', 'Brooklynn', 'Keely', '', 2, 541, CAST('2015-02-28 00:00:00' AS SmallDateTime), 5372.0000, 0.0000, 4972.8400, 0.0000),
(34, 'IBM', 'PO Box 61000', '', 'San Francisco', 'CA', '94161', 'Camron', 'Trentin', '(800) 555-4426', 1, 160, CAST('2015-03-13 00:00:00' AS SmallDateTime), 123.0000, 0.0000, 123.0000, 0.0000),
(35, 'Cal State Termite', 'PO Box 956', '', 'Selma', 'CA', '93662', 'Hunter', 'Demetrius', '(209) 555-1534', 2, 523, CAST('2015-03-27 00:00:00' AS SmallDateTime), 65.0000, 0.0000, 0.0000, 0.0000),
(36, 'Graylift', 'PO Box 2808', '', 'Fresno', 'CA', '93745', 'Sydney', 'Deangelo', '(209) 555-6621', 3, 532, CAST('2015-03-27 00:00:00' AS SmallDateTime), 62.1400, 0.0000, 630.7400, 0.0000),
(37, 'Blue Cross', 'PO Box 9061', '', 'Oxnard', 'CA', '93031', 'Eliana', 'Nikolas', '(800) 555-0912', 3, 510, CAST('2015-04-04 00:00:00' AS SmallDateTime), 372.0000, 0.0000, 0.0000, 0.0000),
(38, 'Venture Communications Int''l', '60 Madison Ave', '', 'New York', 'NY', '10010', 'Neftaly', 'Thalia', '(212) 555-4800', 3, 540, CAST('2015-04-04 00:00:00' AS SmallDateTime), 6000.0000, 0.0000, 0.0000, 0.0000),
(39, 'Custom Printing Company', 'PO Box 7028', '', 'St Louis', 'MO', '63177', 'Myles', 'Harley', '(301) 555-1494', 3, 540, CAST('2015-04-17 00:00:00' AS SmallDateTime), 12102.0100, 0.0000, 0.0000, 0.0000),
(40, 'Nat Assoc of College Stores', '500 East Lorain Street', '', 'Oberlin', 'OH', '44074', 'Bernard', 'Lucy', '', 3, 572, CAST('2015-04-17 00:00:00' AS SmallDateTime), 390.0000, 0.0000, 390.0000, 0.0000),
(41, 'Shields Design', '415 E Olive Ave', '', 'Fresno', 'CA', '93728', 'Kerry', 'Rowan', '(209) 555-8060', 2, 403, CAST('2015-05-14 00:00:00' AS SmallDateTime), 5588.9900, 0.0000, 22363.2000, 0.0000),
(42, 'Opamp Technical Books', '1033 N Sycamore Ave.', '', 'Los Angeles', 'CA', '90038', 'Paris', 'Gideon', '(213) 555-4322', 3, 572, CAST('2015-05-28 00:00:00' AS SmallDateTime), 700.0000, 0.0000, 1136.1700, 0.0000),
(43, 'Capital Resource Credit', 'PO Box 39046', '', 'Minneapolis', 'MN', '55439', 'Maxwell', 'Jayda', '(612) 555-0057', 3, 589, CAST('2015-06-11 00:00:00' AS SmallDateTime), 59.3800, 0.0000, 0.0000, 0.0000),
(44, 'Courier Companies, Inc', 'PO Box 5317', '', 'Boston', 'MA', '02206', 'Antavius', 'Troy', '(508) 555-6351', 4, 400, CAST('2015-06-11 00:00:00' AS SmallDateTime), 27462.5600, 0.0000, 29074.2500, 0.0000),
(45, 'Naylor Publications Inc', 'PO Box 40513', '', 'Jacksonville', 'FL', '32231', 'Gerald', 'Kristofer', '(800) 555-6041', 3, 572, CAST('2015-06-11 00:00:00' AS SmallDateTime), 99.5000, 0.0000, 0.0000, 0.0000),
(46, 'Open Horizons Publishing', 'Book Marketing Update', 'PO Box 205', 'Fairfield', 'IA', '52556', 'Damien', 'Deborah', '(515) 555-6130', 2, 540, CAST('2015-06-11 00:00:00' AS SmallDateTime), 94.0000, 0.0000, 60.0000, 0.0000),
(47, 'Baker & Taylor Books', 'Five Lakepointe Plaza, Ste 500', '2709 Water Ridge Parkway', 'Charlotte', 'NC', '28217', 'Bernardo', 'Brittnee', '(704) 555-3500', 3, 572, CAST('2015-06-27 00:00:00' AS SmallDateTime), 531.2500, 0.0000, 0.0000, 0.0000),
(48, 'Fresno County Tax Collector', 'PO Box 1192', '', 'Fresno', 'CA', '93715', 'Brenton', 'Kila', '(209) 555-3482', 3, 574, CAST('2015-06-27 00:00:00' AS SmallDateTime), 378.8800, 0.0000, 678.5800, 0.0000),
(49, 'Mcgraw Hill Companies', 'PO Box 87373', '', 'Chicago', 'IL', '60680', 'Holbrooke', 'Rashad', '(614) 555-3663', 3, 572, CAST('2015-06-27 00:00:00' AS SmallDateTime), 70.4100, 0.0000, 0.0000, 0.0000),
(50, 'Publishers Weekly', 'Box 1979', '', 'Marion', 'OH', '43305', 'Carrollton', 'Priscilla', '(800) 555-1669', 3, 572, CAST('2015-06-27 00:00:00' AS SmallDateTime), 169.0000, 0.0000, 149.0000, 0.0000),
(51, 'Blue Shield of California', 'PO Box 7021', '', 'Anaheim', 'CA', '92850', 'Smith', 'Kylie', '(415) 555-5103', 3, 510, CAST('2015-07-08 00:00:00' AS SmallDateTime), 720.0000, 0.0000, 939.0000, 0.0000),
(52, 'Aztek Label', 'Accounts Payable', '1150 N Tustin Ave', 'Aneheim', 'CA', '92807', 'Griffin', 'Brian', '(714) 555-9000', 3, 551, CAST('2015-07-24 00:00:00' AS SmallDateTime), 267.0000, 0.0000, 813.7000, 0.0000),
(53, 'Gary McKeighan Insurance', '3649 W Beechwood Ave #101', '', 'Fresno', 'CA', '93711', 'Jair', 'Caitlin', '(209) 555-2420', 3, 590, CAST('2015-07-24 00:00:00' AS SmallDateTime), 4745.0000, 0.0000, 4990.1800, 0.0000),
(54, 'Ph Photographic Services', '2384 E Gettysburg', '', 'Fresno', 'CA', '93726', 'Cheyenne', 'Kaylea', '(209) 555-0765', 3, 540, CAST('2015-07-24 00:00:00' AS SmallDateTime), 565.1500, 0.0000, 0.0000, 0.0000),
(55, 'Quality Education Data', 'PO Box 95857', '', 'Chicago', 'IL', '60694', 'Misael', 'Kayle', '(800) 555-5811', 2, 540, CAST('2015-07-24 00:00:00' AS SmallDateTime), 799.0900, 0.0000, 0.0000, 0.0000),
(56, 'Springhouse Corp', 'PO Box 7247-7051', '', 'Philadelphia', 'PA', '19170', 'Maeve', 'Clarence', '(215) 555-8700', 3, 523, CAST('2015-07-29 00:00:00' AS SmallDateTime), 1942.2500, 0.0000, 1215.5000, 0.0000),
(57, 'The Windows Deck', '117 W Micheltorena Top Floor', '', 'Santa Barbara', 'CA', '93101', 'Wood', 'Liam', '(800) 555-3353', 3, 536, CAST('2015-07-29 00:00:00' AS SmallDateTime), 2975.0000, 0.0000, 0.0000, 0.0000),
(58, 'Fresno Rack & Shelving Inc', '4718 N Bendel Ave', '', 'Fresno', 'CA', '93722', 'Baylee', 'Dakota', '', 2, 523, CAST('2015-08-13 00:00:00' AS SmallDateTime), 168.0900, 0.0000, 0.0000, 0.0000),
(59, 'Publishers Marketing Assoc', '627 Aviation Way', '', 'Manhatttan Beach', 'CA', '90266', 'Walker', 'Jovon', '(310) 555-2732', 3, 572, CAST('2015-08-13 00:00:00' AS SmallDateTime), 175.0000, 0.0000, 120.0000, 0.0000),
(60, 'The Mailers Guide Co', 'PO Box 1550', '', 'New Rochelle', 'NY', '10802', 'Lacy', 'Karina', '', 3, 540, CAST('2015-08-13 00:00:00' AS SmallDateTime), 69.0000, 0.0000, 59.0000, 0.0000),
(61, 'American Booksellers Assoc', '828 S Broadway', '', 'Tarrytown', 'NY', '10591', 'Angelica', 'Nashalie', '(800) 555-0037', 3, 574, CAST('2015-08-28 00:00:00' AS SmallDateTime), 775.0000, 0.0000, 175.0000, 0.0000),
(62, 'Cmg Information Services', 'PO Box 2283', '', 'Boston', 'MA', '02107', 'Randall', 'Yash', '(508) 555-7000', 3, 540, CAST('2015-08-28 00:00:00' AS SmallDateTime), 194.5700, 0.0000, 399.8500, 0.0000),
(63, 'Lou Gentile''s Flower Basket', '722 E Olive Ave', '', 'Fresno', 'CA', '93728', 'Anum', 'Trisha', '(209) 555-6643', 1, 570, CAST('2015-08-28 00:00:00' AS SmallDateTime), 74.3500, 0.0000, 0.0000, 0.0000),
(64, 'Texaco', 'PO Box 6070', '', 'Inglewood', 'CA', '90312', 'Oren', 'Grace', '', 3, 582, CAST('2015-08-28 00:00:00' AS SmallDateTime), 62.4800, 0.0000, 476.5000, 0.0000),
(65, 'The Drawing Board', 'PO Box 4758', '', 'Carol Stream', 'IL', '60197', 'Mckayla', 'Jeffery', '', 2, 551, CAST('2015-08-28 00:00:00' AS SmallDateTime), 366.6500, 0.0000, 502.1300, 0.0000),
(66, 'Ascom Hasler Mailing Systems', 'PO Box 895', '', 'Shelton', 'CT', '06484', 'Lewis', 'Darnell', '', 3, 532, CAST('2015-09-12 00:00:00' AS SmallDateTime), 522.2100, 0.0000, 716.6600, 0.0000),
(67, 'Bill Jones', 'Secretary Of State', 'PO Box 944230', 'Sacramento', 'CA', '94244', 'Deasia', 'Tristin', '', 3, 589, CAST('2015-09-12 00:00:00' AS SmallDateTime), 10.0000, 0.0000, 10.0000, 0.0000),
(68, 'Computer Library', '3502 W Greenway #7', '', 'Phoenix', 'AZ', '85023', 'Aryn', 'Leroy', '(602) 547-0331', 3, 540, CAST('2015-09-12 00:00:00' AS SmallDateTime), 196.2000, 0.0000, 0.0000, 0.0000),
(69, 'Frank E Wilber Co', '2437 N Sunnyside', '', 'Fresno', 'CA', '93727', 'Millerton', 'Johnathon', '(209) 555-1881', 3, 532, CAST('2015-09-12 00:00:00' AS SmallDateTime), 1934.7000, 0.0000, 1538.2400, 0.0000),
(70, 'Fresno Credit Bureau', 'PO Box 942', '', 'Fresno', 'CA', '93714', 'Braydon', 'Anne', '(209) 555-7900', 2, 555, CAST('2015-09-12 00:00:00' AS SmallDateTime), 2663.2600, 0.0000, 452.4700, 0.0000),
(71, 'The Fresno Bee', '1626 E Street', '', 'Fresno', 'CA', '93786', 'Colton', 'Leah', '(209) 555-4442', 2, 572, CAST('2015-09-12 00:00:00' AS SmallDateTime), 751.8300, 0.0000, 0.0000, 0.0000),
(72, 'Data Reproductions Corp', '4545 Glenmeade Lane', '', 'Auburn Hills', 'MI', '48326', 'Arodondo', 'Cesar', '(810) 555-3700', 3, 400, CAST('2015-10-09 00:00:00' AS SmallDateTime), 14624.8800, 0.0000, 0.0000, 0.0000),
(73, 'Executive Office Products', '353 E Shaw Ave', '', 'Fresno', 'CA', '93710', 'Danielson', 'Rachael', '(209) 555-1704', 2, 570, CAST('2015-10-09 00:00:00' AS SmallDateTime), 405.4800, 0.0000, 1361.7800, 0.0000),
(74, 'Leslie Company', 'PO Box 610', '', 'Olathe', 'KS', '66061', 'Alondra', 'Zev', '(800) 255-6210', 3, 570, CAST('2015-10-09 00:00:00' AS SmallDateTime), 139.6700, 0.0000, 0.0000, 0.0000),
(75, 'Retirement Plan Consultants', '6435 North Palm Ave, Ste 101', '', 'Fresno', 'CA', '93704', 'Edgardo', 'Salina', '(209) 555-7070', 3, 589, CAST('2015-10-09 00:00:00' AS SmallDateTime), 1660.0000, 0.0000, 1386.0000, 0.0000),
(76, 'Simon Direct Inc', '4 Cornwall Dr Ste 102', '', 'East Brunswick', 'NJ', '08816', 'Bradlee', 'Daniel', '(908) 555-7222', 2, 540, CAST('2015-10-09 00:00:00' AS SmallDateTime), 8662.5000, 0.0000, 0.0000, 0.0000),
(77, 'State Board Of Equalization', 'PO Box 942808', '', 'Sacramento', 'CA', '94208', 'Dean', 'Julissa', '(916) 555-4911', 1, 631, CAST('2015-10-09 00:00:00' AS SmallDateTime), 2433.0000, 0.0000, 3606.8300, 0.0000),
(78, 'The Presort Center', '1627 "E" Street', '', 'Fresno', 'CA', '93706', 'Marissa', 'Kyle', '(209) 555-6151', 3, 540, CAST('2015-10-09 00:00:00' AS SmallDateTime), 2377.4300, 0.0000, 0.0000, 0.0000),
(79, 'Valprint', 'PO Box 12332', '', 'Fresno', 'CA', '93777', 'Warren', 'Quentin', '(209) 555-3112', 3, 551, CAST('2015-10-09 00:00:00' AS SmallDateTime), 44995.7500, 0.0000, 19211.1200, 0.0000),
(80, 'Cardinal Business Media, Inc.', 'P O Box 7247-7844', '', 'Philadelphia', 'PA', '19170', 'Eulalia', 'Kelsey', '(215) 555-1500', 2, 540, CAST('2015-10-23 00:00:00' AS SmallDateTime), 2905.0000, 0.0000, 3867.7500, 0.0000),
(81, 'Wang Laboratories, Inc.', 'P.O. Box 21209', '', 'Pasadena', 'CA', '91185', 'Kapil', 'Robert', '(800) 555-0344', 2, 160, CAST('2015-10-23 00:00:00' AS SmallDateTime), 4343.7600, 0.0000, 24125.3800, 0.0000),
(82, 'Reiter''s Scientific & Pro Books', '2021 K Street Nw', '', 'Washington', 'DC', '20006', 'Rodolfo', 'Carlee', '(202) 555-5561', 2, 572, CAST('2015-10-28 00:00:00' AS SmallDateTime), 600.0000, 0.0000, 600.0000, 0.0000),
(83, 'Ingram', 'PO Box 845361', '', 'Dallas', 'TX', '75284', 'Yobani', 'Trey', '', 2, 572, CAST('2015-11-10 00:00:00' AS SmallDateTime), 4005.0000, 0.0000, 585.0000, 0.0000),
(84, 'Boucher Communications Inc', '1300 Virginia Dr. Ste 400', '', 'Fort Washington', 'PA', '19034', 'Carson', 'Julian', '(215) 555-8000', 3, 540, CAST('2015-11-14 00:00:00' AS SmallDateTime), 1150.0000, 0.0000, 0.0000, 0.0000),
(85, 'Champion Printing Company', '3250 Spring Grove Ave', '', 'Cincinnati', 'OH', '45225', 'Clifford', 'Jillian', '(800) 555-1957', 3, 540, CAST('2015-11-14 00:00:00' AS SmallDateTime), 10729.1400, 0.0000, 8095.2800, 0.0000),
(86, 'Computerworld', 'Department #1872', 'PO Box 61000', 'San Francisco', 'CA', '94161', 'Lloyd', 'Angel', '(617) 555-0700', 1, 572, CAST('2015-11-14 00:00:00' AS SmallDateTime), 11664.5000, 0.0000, 2245.0000, 0.0000),
(87, 'DMV Renewal', 'PO Box 942894', '', 'Sacramento', 'CA', '94294', 'Josey', 'Lorena', '', 4, 568, CAST('2015-11-14 00:00:00' AS SmallDateTime), 923.0000, 0.0000, 779.0000, 0.0000),
(88, 'Edward Data Services', '4775 E Miami River Rd', '', 'Cleves', 'OH', '45002', 'Helena', 'Jeanette', '(513) 555-3043', 1, 540, CAST('2015-11-14 00:00:00' AS SmallDateTime), 413.3600, 0.0000, 0.0000, 0.0000),
(89, 'Evans Executone Inc', '4918 Taylor Ct', '', 'Turlock', 'CA', '95380', 'Royce', 'Hannah', '', 1, 522, CAST('2015-11-14 00:00:00' AS SmallDateTime), 394.5500, 0.0000, 0.0000, 0.0000),
(90, 'Wakefield Co', '295 W Cromwell Ave Ste 106', '', 'Fresno', 'CA', '93711', 'Rothman', 'Nathanael', '(209) 555-4744', 2, 170, CAST('2015-11-14 00:00:00' AS SmallDateTime), 356.4800, 0.0000, 0.0000, 0.0000),
(91, 'McKesson Water Products', 'P O Box 7126', '', 'Pasadena', 'CA', '91109', 'Destin', 'Luciano', '(800) 555-7009', 2, 570, CAST('2015-11-14 00:00:00' AS SmallDateTime), 384.7100, 0.0000, 363.1300, 0.0000),
(92, 'Zip Print & Copy Center', 'PO Box 12332', '', 'Fresno', 'CA', '93777', 'Javen', 'Justin', '(233) 555-6400', 2, 540, CAST('2015-11-14 00:00:00' AS SmallDateTime), 5641.2100, 0.0000, 6186.2700, 0.0000),
(93, 'AT&T', 'PO Box 78225', '', 'Phoenix', 'AZ', '85062', 'Wesley', 'Alisha', '', 3, 522, CAST('2015-11-16 00:00:00' AS SmallDateTime), 15730.9700, 0.0000, 9231.3400, 0.0000),
(94, 'Abbey Office Furnishings', '4150 W Shaw Ave', '', 'Fresno', 'CA', '93722', 'Francis', 'Kyra', '(209) 555-8300', 2, 150, CAST('2015-11-19 00:00:00' AS SmallDateTime), 5515.3500, 0.0000, 1385.0800, 0.0000),
(95, 'Pacific Bell', '', '                              .', 'Sacramento', 'CA', '95887', 'Nickalus', 'Kurt', '(209) 555-7500', 2, 522, CAST('2015-11-25 00:00:00' AS SmallDateTime), 4711.7700, 0.0000, 4296.1200, 0.0000),
(96, 'Wells Fargo Bank', 'Business Mastercard', 'P.O. Box 29479', 'Phoenix', 'AZ', '85038', 'Damion', 'Mikayla', '(947) 555-3900', 2, 160, CAST('2015-11-25 00:00:00' AS SmallDateTime), 1107.3100, 0.0000, 1394.6800, 0.0000),
(97, 'Compuserve', 'Dept L-742', '', 'Columbus', 'OH', '43260', 'Armando', 'Jan', '(614) 555-8600', 2, 572, CAST('2015-11-26 00:00:00' AS SmallDateTime), 109.4500, 0.0000, 145.9600, 0.0000),
(98, 'American Express', 'Box 0001', '', 'Los Angeles', 'CA', '90096', 'Story', 'Kirsten', '(800) 555-3344', 2, 160, CAST('2015-11-28 00:00:00' AS SmallDateTime), 28740.1000, 556.1900, 40780.8300, 0.0000),
(99, 'Bertelsmann Industry Svcs. Inc', '28210 N Avenue Stanford', '', 'Valencia', 'CA', '91355', 'Potter', 'Lance', '(805) 555-0584', 3, 400, CAST('2015-11-28 00:00:00' AS SmallDateTime), 39420.5600, 0.0000, 42499.9400, 0.0000),
(100, 'Cahners Publishing Company', 'Citibank Lock Box 4026', '8725 W Sahara Zone 1127', 'The Lake', 'NV', '89163', 'Jacobsen', 'Samuel', '(301) 555-2162', 4, 540, CAST('2015-11-28 00:00:00' AS SmallDateTime), 2184.5000, 0.0000, 0.0000, 0.0000),
(101, 'California Business Machines', 'Gallery Plz', '5091 N Fresno', 'Fresno', 'CA', '93710', 'Rohansen', 'Anders', '(209) 555-5570', 2, 170, CAST('2015-11-28 00:00:00' AS SmallDateTime), 722.1600, 0.0000, 981.8300, 0.0000),
(102, 'Coffee Break Service', 'PO Box 1091', '', 'Fresno', 'CA', '93714', 'Smitzen', 'Jeffrey', '(209) 555-8700', 4, 570, CAST('2015-11-28 00:00:00' AS SmallDateTime), 706.9000, 0.0000, 816.5000, 0.0000),
(103, 'Dean Witter Reynolds', '9 River Pk Pl E 400', '', 'Fresno', 'CA', '93720', 'Johnson', 'Vance', '(209) 555-7171', 5, 589, CAST('2015-11-28 00:00:00' AS SmallDateTime), 29522.5000, 0.0000, 17967.5000, 0.0000),
(104, 'Digital Dreamworks', '5070 N Sixth Ste. 71', '', 'Fresno', 'CA', '93711', 'Elmert', 'Ron', '', 3, 589, CAST('2015-11-28 00:00:00' AS SmallDateTime), 5000.0000, 0.0000, 0.0000, 0.0000),
(105, 'Dristas Groom & Mccormick', '7112 N Fresno St Ste 200', '', 'Fresno', 'CA', '93720', 'Aaronsen', 'Thom', '(209) 555-8484', 3, 591, CAST('2015-11-28 00:00:00' AS SmallDateTime), 9082.0000, 0.0000, 8173.0000, 0.0000),
(106, 'Ford Motor Credit Company', 'Dept 0419', '', 'Los Angeles', 'CA', '90084', 'Snyder', 'Karen', '(800) 555-7000', 3, 582, CAST('2015-11-28 00:00:00' AS SmallDateTime), 5535.2000, 0.0000, 6039.8100, 0.0000),
(107, 'Franchise Tax Board', 'PO Box 942857', '', 'Sacramento', 'CA', '94257', 'Prado', 'Anita', '', 4, 507, CAST('2015-11-28 00:00:00' AS SmallDateTime), 12632.5000, 0.0000, 800.0000, 0.0000),
(108, 'Gostanian General Building', '427 W Bedford #102', '', 'Fresno', 'CA', '93711', 'Bragg', 'Walter', '(209) 555-5100', 4, 523, CAST('2015-11-28 00:00:00' AS SmallDateTime), 450.0000, 0.0000, 0.0000, 0.0000),
(109, 'Kent H Landsberg Co', 'File No 72686', 'PO Box 61000', 'San Francisco', 'CA', '94160', 'Stevens', 'Wendy', '(916) 555-8100', 3, 540, CAST('2015-11-28 00:00:00' AS SmallDateTime), 5711.1200, 0.0000, 1473.7000, 0.0000),
(110, 'Malloy Lithographing Inc', '5411 Jackson Road', 'PO Box 1124', 'Ann Arbor', 'MI', '48106', 'Regging', 'Abe', '(313) 555-6113', 3, 400, CAST('2015-11-28 00:00:00' AS SmallDateTime), 213039.6500, 0.0000, 192213.0100, 0.0000),
(111, 'Net Asset, Llc', '1315 Van Ness Ave Ste. 103', '', 'Fresno', 'CA', '93721', 'Kraggin', 'Laura', '', 1, 572, CAST('2015-11-28 00:00:00' AS SmallDateTime), 581.2100, 0.0000, 0.0000, 0.0000),
(112, 'Office Depot', 'File No 81901', '', 'Los Angeles', 'CA', '90074', 'Pinsippi', 'Val', '(209) 555-1711', 3, 570, CAST('2015-11-28 00:00:00' AS SmallDateTime), 3916.9100, 0.0000, 4197.3400, 0.0000),
(113, 'Pollstar', '4697 W Jacquelyn Ave', '', 'Fresno', 'CA', '93722', 'Aranovitch', 'Robert', '(209) 555-2631', 5, 520, CAST('2015-11-28 00:00:00' AS SmallDateTime), 17500.0000, 0.0000, 19750.0000, 0.0000),
(114, 'Postmaster', 'Postage Due Technician', '1900 E Street', 'Fresno', 'CA', '93706', 'Finklestein', 'Fyodor', '(209) 555-7785', 1, 552, CAST('2015-11-28 00:00:00' AS SmallDateTime), 1875.0000, 0.0000, 1175.0000, 0.0000),
(115, 'Roadway Package System, Inc', 'Dept La 21095', '', 'Pasadena', 'CA', '91185', 'Smith', 'Sam', '', 4, 553, CAST('2015-11-28 00:00:00' AS SmallDateTime), 614.1500, 0.0000, 327.2300, 0.0000),
(116, 'State of California', 'Employment Development Dept', 'PO Box 826276', 'Sacramento', 'CA', '94230', 'Articunia', 'Mercedez', '(209) 555-5132', 1, 631, CAST('2015-11-28 00:00:00' AS SmallDateTime), 22300.4600, 0.0000, 17568.5800, 0.0000),
(117, 'Suburban Propane', '2874 S Cherry Ave', '', 'Fresno', 'CA', '93706', 'Spivak', 'Harold', '(209) 555-2770', 3, 521, CAST('2015-11-28 00:00:00' AS SmallDateTime), 61.9400, 0.0000, 32.3400, 0.0000),
(118, 'Unocal', 'P.O. Box 860070', '', 'Pasadena', 'CA', '91186', 'Bluzinski', 'Rachael', '(415) 555-7600', 3, 582, CAST('2015-11-28 00:00:00' AS SmallDateTime), 1870.2800, 0.0000, 1140.6700, 0.0000),
(119, 'Yesmed, Inc', 'PO Box 2061', '', 'Fresno', 'CA', '93718', 'Hernandez', 'Reba', '(209) 555-0600', 2, 589, CAST('2015-11-28 00:00:00' AS SmallDateTime), 48690.5100, 0.0000, 51420.0300, 0.0000),
(120, 'Dataforms/West', '1617 W. Shaw Avenue', 'Suite F', 'Fresno', 'CA', '93711', 'Church', 'Charlie', '', 3, 551, CAST('2015-11-30 00:00:00' AS SmallDateTime), 12108.4400, 0.0000, 13232.2200, 0.0000),
(121, 'Zylka Design', '3467 W Shaw Ave #103', '', 'Fresno', 'CA', '93711', 'Ronaldsen', 'Jaime', '(209) 555-8625', 3, 403, CAST('2015-11-30 00:00:00' AS SmallDateTime), 30486.4400, 0.0000, 5311.4800, 0.0000),
(122, 'United Parcel Service', 'P.O. Box 505820', '', 'Reno', 'NV', '88905', 'Beauregard', 'Violet', '(800) 555-0855', 3, 553, CAST('2015-12-02 00:00:00' AS SmallDateTime), 93601.9900, 0.0000, 76238.9600, 0.0000),
(123, 'Federal Express Corporation', 'P.O. Box 1140', 'Dept A', 'Memphis', 'TN', '38101', 'Bucket', 'Charlie', '(209) 555-4091', 3, 553, CAST('2015-12-05 00:00:00' AS SmallDateTime), 29742.9800, 0.0000, 19893.6100, 0.0000)
SET IDENTITY_INSERT Vendors OFF
GO

/****** Object:  Index City    ******/
CREATE NONCLUSTERED INDEX City ON Customers(
	CustCity ASC
)
GO

/****** Object:  Index PostalCode    ******/
CREATE NONCLUSTERED INDEX PostalCode ON Customers(
	CustZip ASC
)
GO

/****** Object:  Index Region    ******/
CREATE NONCLUSTERED INDEX Region ON Customers(
	CustState ASC
)
GO

/****** Object:  Index IX_VendorName    ******/
ALTER TABLE Vendors
ADD CONSTRAINT IX_VendorName UNIQUE NONCLUSTERED (
	VendorName ASC
)
GO

ALTER TABLE Sample
ADD CONSTRAINT DF_Sample_Text  DEFAULT ('None') FOR Color
GO

ALTER TABLE Vendors
ADD CONSTRAINT DF_Vendors_TermsID  DEFAULT (3) FOR TermsID
GO

ALTER TABLE Vendors
ADD CONSTRAINT DF_Vendors_AccountNo  DEFAULT (570) FOR AccountNo
GO

ALTER TABLE Vendors
ADD CONSTRAINT DF_Vendors_YTDPurchases  DEFAULT (0) FOR YTDPurchases
GO

ALTER TABLE Vendors
ADD CONSTRAINT DF_Vendors_YTDReturns  DEFAULT (0) FOR YTDReturns
GO

ALTER TABLE Vendors
ADD CONSTRAINT DF_Vendors_LastYTDPurchases  DEFAULT (0) FOR LastYTDPurchases
GO

ALTER TABLE Vendors
ADD CONSTRAINT DF_Vendors_LastYTDReturns  DEFAULT (0) FOR LastYTDReturns
GO

ALTER TABLE SalesTotals  WITH CHECK
ADD CONSTRAINT FK_SalesTotals_SalesReps FOREIGN KEY(RepID)
REFERENCES SalesReps (RepID)
GO

ALTER TABLE SalesTotals CHECK CONSTRAINT FK_SalesTotals_SalesReps
GO

USE master
GO

ALTER DATABASE Examples SET  READ_WRITE
GO
```

#### For `ProductOrders` Database

```SQL
USE master
GO

/****** Object:  Database ProductOrders     ******/
IF DB_ID('ProductOrders') IS NOT NULL
    DROP DATABASE ProductOrders
GO

CREATE DATABASE ProductOrders
GO

USE ProductOrders
GO

/****** Object:  Table Customers     ******/
CREATE TABLE Customers(
	CustID int IDENTITY(1,1) NOT NULL,
	CustFirstName nvarchar(50) NULL,
	CustLastName nvarchar(50) NOT NULL,
	CustAddress nvarchar(255) NOT NULL,
	CustCity nvarchar(50) NOT NULL,
	CustState nvarchar(20) NOT NULL,
	CustZip nvarchar(20) NOT NULL,
	CustPhone nvarchar(30) NOT NULL,
	CustFax nvarchar(30) NULL,
 CONSTRAINT PK_Customers PRIMARY KEY CLUSTERED(
	CustID ASC
 )
)
GO

/****** Object:  Table Items     ******/
CREATE TABLE Items(
	ItemID int NOT NULL,
	Title nvarchar(50) NOT NULL,
	Artist nvarchar(50) NOT NULL,
	UnitPrice money NOT NULL
)
GO

/****** Object:  Table OrderDetails     ******/
CREATE TABLE OrderDetails(
	OrderID int NOT NULL,
	ItemID int NOT NULL,
	Quantity smallint NOT NULL
)
GO

/****** Object:  Table Orders     ******/
CREATE TABLE Orders(
	OrderID int NOT NULL,
	CustID int NOT NULL,
	OrderDate datetime NOT NULL,
	ShippedDate datetime NULL
)
GO

SET IDENTITY_INSERT Customers ON
INSERT Customers (CustID, CustFirstName, CustLastName, CustAddress, CustCity, CustState, CustZip, CustPhone, CustFax) VALUES
(1, 'Korah', 'Blanca', '1555 W Lane Ave', 'Columbus', 'OH', '43221', '6145554435', '6145553928'),
(2, 'Yash', 'Randall', '11 E Rancho Madera Rd', 'Madison', 'WI', '53707', '2095551205', '2095552262'),
(3, 'Johnathon', 'Millerton', '60 Madison Ave', 'New York', 'NY', '10010', '2125554800', NULL),
(4, 'Mikayla', 'Damion', '2021 K Street Nw', 'Washington', 'DC', '20006', '2025555561', NULL),
(5, 'Kendall', 'Mayte', '4775 E Miami River Rd', 'Cleves', 'OH', '45002', '5135553043', NULL),
(6, 'Kaitlin', 'Hostlery', '3250 Spring Grove Ave', 'Cincinnati', 'OH', '45225', '8005551957', '8005552826'),
(7, 'Derek', 'Chaddick', '9022 E Merchant Wy', 'Fairfield', 'IA', '52556', '5155556130', NULL),
(8, 'Deborah', 'Damien', '415 E Olive Ave', 'Fresno', 'CA', '93728', '5595558060', NULL),
(9, 'Karina', 'Lacy', '882 W Easton Wy', 'Los Angeles', 'CA', '90084', '8005557000', NULL),
(10, 'Kurt', 'Nickalus', '28210 N Avenue Stanford', 'Valencia', 'CA', '91355', '8055550584', '8055556689'),
(11, 'Kelsey', 'Eulalia', '7833 N Ridge Rd', 'Sacramento', 'CA', '95887', '2095557500', '2095551302'),
(12, 'Anders', 'Rohansen', '12345 E 67th Ave NW', 'Takoma Park', 'MD', '24512', '3385556772', NULL),
(13, 'Thalia', 'Neftaly', '2508 W Shaw Ave', 'Fresno', 'CA', '93711', '5595556245', NULL),
(14, 'Gonzalo', 'Keeton', '12 Daniel Road', 'Fairfield', 'NJ', '07004', '2015559742', NULL),
(15, 'Ania', 'Irvin', '1099 N Farcourt St', 'Orange', 'CA', '92807', '7145559000', NULL),
(16, 'Dakota', 'Baylee', '1033 N Sycamore Ave.', 'Los Angeles', 'CA', '90038', '2135554322', NULL),
(17, 'Samuel', 'Jacobsen', '3433 E Widget Ave', 'Palo Alto', 'CA', '92711', '4155553434', NULL),
(18, 'Justin', 'Javen', '828 S Broadway', 'Tarrytown', 'NY', '10591', '8005550037', NULL),
(19, 'Kyle', 'Marissa', '789 E Mercy Ave', 'Phoenix', 'AZ', '85038', '9475553900', NULL),
(20, 'Erick', 'Kaleigh', 'Five Lakepointe Plaza, Ste 500', 'Charlotte', 'NC', '28217', '7045553500', NULL),
(21, 'Marvin', 'Quintin', '2677 Industrial Circle Dr', 'Columbus', 'OH', '43260', '6145558600', '6145557580'),
(22, 'Rashad', 'Holbrooke', '3467 W Shaw Ave #103', 'Fresno', 'CA', '93711', '5595558625', '5595558495'),
(23, 'Trisha', 'Anum', '627 Aviation Way', 'Manhatttan Beach', 'CA', '90266', '3105552732', NULL),
(24, 'Julian', 'Carson', '372 San Quentin', 'San Francisco', 'CA', '94161', '6175550700', NULL),
(25, 'Kirsten', 'Story', '2401 Wisconsin Ave NW', 'Washington', 'DC', '20559', '2065559115', NULL)
SET IDENTITY_INSERT Customers OFF
GO

INSERT Items (ItemID, Title, Artist, UnitPrice) VALUES
(1, 'Umami In Concert', 'Umami', 17.9500),
(2, 'Race Car Sounds', 'The Ubernerds', 13.0000),
(3, 'No Rest For The Weary', 'No Rest For The Weary', 16.9500),
(4, 'More Songs About Structures and Comestibles', 'No Rest For The Weary', 17.9500),
(5, 'On The Road With Burt Ruggles', 'Burt Ruggles', 17.5000),
(6, 'No Fixed Address', 'Sewed the Vest Pocket', 16.9500),
(7, 'Rude Noises', 'Jess & Odie', 13.0000),
(8, 'Burt Ruggles: An Intimate Portrait', 'Burt Ruggles', 17.9500),
(9, 'Zone Out With Umami', 'Umami', 16.9500),
(10, 'Etcetera', 'Onn & Onn', 17.0000)
GO

INSERT OrderDetails (OrderID, ItemID, Quantity) VALUES
(381, 1, 1),
(601, 9, 1),
(442, 1, 1),
(523, 9, 1),
(630, 5, 1),
(778, 1, 1),
(693, 10, 1),
(118, 1, 1),
(264, 7, 1),
(607, 10, 1),
(624, 7, 1),
(658, 1, 1),
(800, 5, 1),
(158, 3, 1),
(321, 10, 1),
(687, 6, 1),
(827, 6, 1),
(144, 3, 1),
(264, 8, 1),
(479, 1, 2),
(630, 6, 2),
(796, 5, 1),
(97, 4, 1),
(601, 5, 1),
(773, 10, 1),
(800, 1, 1),
(29, 10, 1),
(70, 1, 1),
(97, 8, 1),
(165, 4, 1),
(180, 4, 1),
(231, 10, 1),
(392, 8, 1),
(413, 10, 1),
(491, 6, 1),
(494, 2, 1),
(606, 8, 1),
(607, 3, 1),
(651, 3, 1),
(703, 4, 1),
(796, 2, 1),
(802, 2, 1),
(802, 3, 1),
(824, 7, 2),
(829, 1, 1),
(550, 4, 1),
(796, 7, 1),
(829, 2, 1),
(693, 6, 1),
(29, 3, 1),
(32, 7, 1),
(242, 1, 1),
(298, 1, 1),
(479, 4, 1),
(548, 9, 1),
(627, 9, 1),
(778, 3, 1),
(687, 8, 1),
(19, 5, 1),
(89, 4, 1),
(242, 6, 1),
(264, 4, 1),
(550, 1, 1),
(631, 10, 1),
(693, 7, 3),
(824, 3, 1),
(829, 5, 1),
(829, 9, 1)
GO

INSERT Orders (OrderID, CustID, OrderDate, ShippedDate) VALUES
(19, 1, CAST('2014-06-23 00:00:00.000' AS DateTime), CAST('2014-06-28 00:00:00.000' AS DateTime)),
(29, 8, CAST('2014-07-05 00:00:00.000' AS DateTime), CAST('2014-07-11 00:00:00.000' AS DateTime)),
(32, 11, CAST('2014-07-10 00:00:00.000' AS DateTime), CAST('2014-07-13 00:00:00.000' AS DateTime)),
(45, 2, CAST('2014-07-25 00:00:00.000' AS DateTime), CAST('2014-07-30 00:00:00.000' AS DateTime)),
(70, 10, CAST('2014-08-28 00:00:00.000' AS DateTime), CAST('2014-09-07 00:00:00.000' AS DateTime)),
(89, 22, CAST('2014-09-20 00:00:00.000' AS DateTime), CAST('2014-09-22 00:00:00.000' AS DateTime)),
(97, 20, CAST('2014-09-29 00:00:00.000' AS DateTime), CAST('2014-10-02 00:00:00.000' AS DateTime)),
(118, 3, CAST('2014-10-24 00:00:00.000' AS DateTime), CAST('2014-10-28 00:00:00.000' AS DateTime)),
(144, 17, CAST('2014-11-21 00:00:00.000' AS DateTime), CAST('2014-11-29 00:00:00.000' AS DateTime)),
(158, 9, CAST('2014-12-04 00:00:00.000' AS DateTime), CAST('2014-12-20 00:00:00.000' AS DateTime)),
(165, 14, CAST('2014-12-11 00:00:00.000' AS DateTime), CAST('2014-12-13 00:00:00.000' AS DateTime)),
(180, 24, CAST('2014-12-25 00:00:00.000' AS DateTime), CAST('2015-01-30 00:00:00.000' AS DateTime)),
(231, 15, CAST('2015-02-14 00:00:00.000' AS DateTime), CAST('2015-02-22 00:00:00.000' AS DateTime)),
(242, 23, CAST('2015-02-24 00:00:00.000' AS DateTime), CAST('2015-03-06 00:00:00.000' AS DateTime)),
(264, 9, CAST('2015-03-15 00:00:00.000' AS DateTime), CAST('2015-03-18 00:00:00.000' AS DateTime)),
(298, 18, CAST('2015-04-18 00:00:00.000' AS DateTime), CAST('2015-05-22 00:00:00.000' AS DateTime)),
(321, 2, CAST('2015-05-09 00:00:00.000' AS DateTime), CAST('2015-06-05 00:00:00.000' AS DateTime)),
(381, 7, CAST('2015-07-08 00:00:00.000' AS DateTime), CAST('2015-07-16 00:00:00.000' AS DateTime)),
(392, 19, CAST('2015-07-16 00:00:00.000' AS DateTime), CAST('2015-07-23 00:00:00.000' AS DateTime)),
(413, 17, CAST('2015-08-05 00:00:00.000' AS DateTime), CAST('2015-09-11 00:00:00.000' AS DateTime)),
(442, 5, CAST('2015-08-28 00:00:00.000' AS DateTime), CAST('2015-09-03 00:00:00.000' AS DateTime)),
(479, 1, CAST('2015-09-30 00:00:00.000' AS DateTime), CAST('2015-11-03 00:00:00.000' AS DateTime)),
(491, 16, CAST('2015-10-08 00:00:00.000' AS DateTime), CAST('2015-10-14 00:00:00.000' AS DateTime)),
(494, 4, CAST('2015-10-10 00:00:00.000' AS DateTime), CAST('2015-10-14 00:00:00.000' AS DateTime)),
(523, 3, CAST('2015-11-07 00:00:00.000' AS DateTime), CAST('2015-11-15 00:00:00.000' AS DateTime)),
(548, 2, CAST('2015-11-22 00:00:00.000' AS DateTime), CAST('2015-12-18 00:00:00.000' AS DateTime)),
(550, 17, CAST('2015-11-23 00:00:00.000' AS DateTime), CAST('2015-12-03 00:00:00.000' AS DateTime)),
(601, 16, CAST('2015-12-21 00:00:00.000' AS DateTime), CAST('2015-12-27 00:00:00.000' AS DateTime)),
(606, 6, CAST('2015-12-25 00:00:00.000' AS DateTime), CAST('2016-01-02 00:00:00.000' AS DateTime)),
(607, 20, CAST('2015-12-25 00:00:00.000' AS DateTime), CAST('2016-01-04 00:00:00.000' AS DateTime)),
(624, 2, CAST('2016-01-04 00:00:00.000' AS DateTime), CAST('2016-01-09 00:00:00.000' AS DateTime)),
(627, 17, CAST('2016-01-05 00:00:00.000' AS DateTime), CAST('2016-01-10 00:00:00.000' AS DateTime)),
(630, 20, CAST('2016-01-08 00:00:00.000' AS DateTime), CAST('2016-01-18 00:00:00.000' AS DateTime)),
(631, 21, CAST('2016-01-09 00:00:00.000' AS DateTime), CAST('2016-01-11 00:00:00.000' AS DateTime)),
(651, 12, CAST('2016-01-19 00:00:00.000' AS DateTime), CAST('2016-02-02 00:00:00.000' AS DateTime)),
(658, 12, CAST('2016-01-23 00:00:00.000' AS DateTime), CAST('2016-02-02 00:00:00.000' AS DateTime)),
(687, 17, CAST('2016-02-05 00:00:00.000' AS DateTime), CAST('2016-02-08 00:00:00.000' AS DateTime)),
(693, 9, CAST('2016-02-07 00:00:00.000' AS DateTime), CAST('2016-02-19 00:00:00.000' AS DateTime)),
(703, 19, CAST('2016-02-12 00:00:00.000' AS DateTime), CAST('2016-02-19 00:00:00.000' AS DateTime)),
(773, 25, CAST('2016-03-11 00:00:00.000' AS DateTime), CAST('2016-03-13 00:00:00.000' AS DateTime)),
(778, 13, CAST('2016-03-12 00:00:00.000' AS DateTime), CAST('2016-03-21 00:00:00.000' AS DateTime)),
(796, 17, CAST('2016-03-19 00:00:00.000' AS DateTime), CAST('2016-03-26 00:00:00.000' AS DateTime)),
(800, 19, CAST('2016-03-21 00:00:00.000' AS DateTime), CAST('2016-03-28 00:00:00.000' AS DateTime)),
(802, 2, CAST('2016-03-21 00:00:00.000' AS DateTime), CAST('2016-03-31 00:00:00.000' AS DateTime)),
(824, 1, CAST('2016-04-01 00:00:00.000' AS DateTime), NULL),
(827, 18, CAST('2016-04-02 00:00:00.000' AS DateTime), NULL),
(829, 9, CAST('2016-04-02 00:00:00.000' AS DateTime), NULL)
GO

USE master
GO

ALTER DATABASE ProductOrders SET  READ_WRITE
GO
```
