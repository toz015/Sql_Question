## SQL PUZZLE
reference link:
https://www.dbrnd.com/sql-interview-the-ultimate-sql-puzzles-and-sql-server-advance-sql-queries/

<!-- MarkdownTOC -->
- [Find the Order basis on thier Status and Step](#Find-the-Order-basis-on-thier-Status-and-Step)
- [Generate the range of data basis on common combination](#Generate-the-range-of-data-basis-on-common-combination)
- [To generate the account balance column for Bank Accounts](#To-generate-the-account-balance-column-for-Bank-Accounts)
- [Place NULL for repeating values](#Place-NULL-for-repeating-values)
- [Recursive CTE](#Get-organisational-level-hierarchy-from-one-table)
- [Replace NULL with Previous Non Null value](#Replace-NULL-with-Previous-Non-Null-value)
- [Generate the data in the RANGE format](#Generate-the-data-in-the-RANGE-format)


<!-- MarkdownTOC -->

#### Find the Order basis on thier Status and Step
Check the below input data and require output data to find order which step is 0 with status D and for the same order other status are P.

Expected Output:

| OrderID|
|--------|
| ABC |

```sql

CREATE TABLE tbl_Orders
(
	OrderID CHAR(5) 
	,Step INTEGER 
	,Status CHAR(1) 
);

INSERT INTO tbl_Orders
VALUES
('ABC', 0, 'D'),
('ABC', 1, 'P'),
('ABC', 2, 'P'),
('ABC', 3, 'P'),
('XYZ', 0, 'D'),
('XYZ', 1, 'D'),
('EFQ', 0, 'D'),
('EFQ', 1, 'D');

```
```sql
---sol1 
SELECT OrderID
FROM tbl_Orders
WHERE OrderID NOT IN
( SELECT tb1.OrderID FROM
tbl_Orders tb1
WHERE
tb1.Step <> 0 AND Status = 'D'
UNION 
SELECT tb2.OrderID FROM
tbl_Orders tb2
WHERE
tb2.Step == 0 AND Status = 'P'

)

---sol2

SELECT OrderID 
FROM tbl_Orders
GROUP BY OrderID
HAVING COUNT(*) =    
COUNT(CASE WHEN Step <> 0 AND Status = 'P' THEN 1 ELSE NULL END) 
+ COUNT(CASE WHEN Step = 0 AND Status = 'D' THEN 1 ELSE NULL END);


```


### Generate the range of data basis on common combination

Check the below input data and expected output to generate the report on range basis on the common combination of SendFlag and ReceiveFlag.

Expected Output:
| startID | EndID| SendFlag | ReceiveFlag|
|---------|------------|--------|---------|
| 1 | 2 | 0 | 0 |
| 3 | 5 | 1 | 0 |
| 6 | 7 | 1 | 1 |
| 8 |  8 |0 | 1 |
```sql
CREATE TABLE Flags
(
	ID INT 
	,CreationDate DATE
	,SendFlag BIT
	,ReceiveFlag BIT)
;
 
INSERT INTO Flags 
VALUES
(1,'20160808',0,0)
,(2,'20161001',0,0)
,(3,'20161126',1,0)
,(4,'20161226',1,0)
,(5,'20170108',1,0)
,(6,'20170310',1,1)
,(7,'20170620',1,1)
,(8,'20170815',0,1);
```

```sql
---sol

WITH CTE1 AS(
SELECT 
ID,
row_number() over(partition by sendFlag, ReceiveFlag order by CreationDate asc) AS rank,
sendFlag, ReceiveFlag
From Flags),

CTE2 AS(
SELECT 
ID,
row_number() over(partition by sendFlag, ReceiveFlag order by CreationDate desc) AS rank,
sendFlag, ReceiveFlag
From Flags)

SELECT c1.ID AS StartID, c2.ID AS EndID , c1.sendflag, c1.receiveflag
FROM CTE1 c1, CTE2 c2 
WHERE c1.rank = 1 and c2.rank = 1 and c1.sendflag = c2.sendflag and c1.receiveflag = c2.receiveflag;


```

### To generate the account balance column for Bank Accounts
Check the below input data and expected output to prepare the report on bank account transaction like generate the data of AccountBalance column where for CR plus the transaction value and for DR minus the transaction value

EXPECTED Output:
|TransactionDate | AccName |    Type Amount |          AccountBalance |
|---------------|----------|---- |---------------------|---------------|
|2017-01-01   |   Anvesh    |  CR  | 60000.00       |       60000.00 |
|2017-02-01   |   Anvesh    |  DB  | 8000.00        |      52000.00 |
|2017-03-01   |   Anvesh    |  CR  | 8000.00        |       60000.00 |
|2017-04-01   |   Anvesh    |  DB  | 5000.00        |       55000.00 |
|2017-01-01   |   Nupur     |  CR  | 10000.00       |       10000.00 |
|2017-02-02   |   Nupur     |  CR  | 8000.00        |       18000.00 |
|2017-03-03   |   Nupur     |  DB  | 8000.00        |       10000.00 |

```sql
DROP TABLE IF EXISTS AccountBalance;

CREATE TABLE AccountBalance
(
	TransactionDate DATE
	,AccName VARCHAR(10)
	,Type VARCHAR(2)
	,Amount MONEY
);
 
INSERT INTO AccountBalance
VALUES
('2017-01-01','Anvesh','CR','60000')
,('2017-02-01','Anvesh','DB','8000')
,('2017-03-01','Anvesh','CR','8000')
,('2017-04-01','Anvesh','DB','5000')
,('2017-01-01','Nupur','CR','10000')
,('2017-02-02','Nupur','CR','8000')
,('2017-03-03','Nupur','DB','8000');
```

```sql
---sol1
WITH CTE AS(
SELECT 
    TransactionDate,
    AccName,
    Type,
    (CASE WHEN Type = 'CR' THEN Amount ELSE -1 * Amount END ) AS Amount
    FROM AccountBalance)
SELECT c2.TransactionDate,
        c2.AccName,
       (select SUM(c1.amount)
        FROM CTE c1
        WHERE c1.TransactionDate <= c2.TransactionDate and c1.AccName = c2.AccName) AS AccountBalance
        FROM CTE c2;
	
```

### Place NULL for repeating values

Check the below input data and expected output to place NULL for repeating values.

Expected Output:

|A |          B        |   C        |   Code |
|-----------|-----------|-----------|----|
|1 |           1 |   |     1  |         A |
| NULL |        NULL  |      2    |       NULL |
| NULL   |     NULL   |     3     |      NULL |
| NULL   |     NULL   |     4     |      NULL |
| 2      |     2      |     1     |      C |
| NULL   |     NULL   |     2     |      NULL |
|NULL   |     NULL    |    3      |     NULL |

```sql
CREATE TABLE Code
(
	A int
	,B Int
	,C int
	,Code CHAR(1) 	
);
 
INSERT INTO Code
VALUES
(1,1,1,'A')
,(1,1,2,'A')
,(1,1,3,'A')
,(1,1,4,'A')
,(2,2,1,'C')
,(2,2,2,'C')
,(2,2,3,'C');
```


```sql
---sol
WITH CTE AS(
SELECT A,
    B,
    C,
    Code,
    row_number() over(partition by A, B, CODE ORDER BY A) AS rnk
    FROM Code)
SELECT
	CASE WHEN rnk = 1 THEN A ELSE NULL END AS A
	,case when rnk = 1 then B ELSE NULL end AS B
    ,C
	,case when rnk = 1 then Code else NULL end AS Code
from CTE
```

### Get organisational level hierarchy from one table

Check the below input data and expected output to prepare the report on Manager -> Employee Nth level hierarchy basis on the ManagerID column.

Expected Output:
|empname	|managername	|level |
|--------------|-------------|-----------|
| Anvesh | NULL |	1 |
| Neevan | NULL |	1 |
| Mukesh | Anvesh | 2 |
| Rajesh | Mukesh | 3 |
| Nupur  | Neevan | 2 |
| Roy | Nupur | 3 |
| Martin | Roy | 4 |
| Manish | Anvesh	 | 2 |
| Eric | Neevan	 | 2 |
| Purv | Eric |	3 |

```sql
DROP TABLE IF EXISTS Employee;
CREATE TABLE Employee
(
	EmpID INT
	,EmpName VARCHAR(10)
	,ManagerID INT
);
 
INSERT INTO Employee 
VALUES
(1,'Anvesh',NULL)
,(2,'Neevan',NULL)
,(3,'Mukesh',1)
,(4,'Rajesh',3)
,(5,'Nupur',2)
,(6,'Roy',5)
,(7,'Martin',6)
,(8,'Manish',1)
,(9,'Eric',2)
,(10,'Purv',9);

```

```sql

---sol

WITH RECURSIVE RCTE AS
( 
    SELECT 
     EmpID AS BaseEmpID, 
     ManagerID AS BaseManagerID,
     1 AS Lvl, 
     EmpID, 
     ManagerID
    FROM employee

    UNION ALL

    SELECT 
     c.BaseEmpID,
     c.BaseManagerID,
     Lvl + 1,
     m.EmpID,
     m.ManagerID
    FROM RCTE c
    JOIN employee m 
      ON m.EmpID = c.ManagerID
)
, EMPLEVELS AS
(
    SELECT 
     BaseEmpID, 
     BaseManagerID, 
     MAX(Lvl) AS Level
    FROM RCTE
    GROUP BY BaseEmpID, BaseManagerID
)
SELECT 
 e.EmpName, 
 m.EmpName AS ManagerName,
 elvl.Level
FROM EMPLEVELS elvl
JOIN employee e ON e.EmpID = elvl.BaseEmpID
LEFT JOIN employee m ON m.EmpID = elvl.BaseManagerID
ORDER BY elvl.BaseEmpID;

```
### Replace NULL with Previous Non Null value

Check the below input data and expected output to replace the NULL value with the previous NON-Null value.

Expected Output:

| Id |          Code | Previous_NonNullCode |
|-----------|----|--------------------|
|1 | ABC | ABC |
|2 | XYZ | XYZ |
|3 | NULL | XYZ |
|4 | NULL | XYZ |
|5 | PQR  | PQR |
|6 | NULL | PQR |
|7 | ERT | ERT |
|8 | NULL | ERT |
```sql
DROP TABLE IF EXISTS tbl_Values;
CREATE TABLE tbl_Values
(
   Id INT
  ,Code VARCHAR(3)
);
 
INSERT INTO tbl_Values 
VALUES
(1,'ABC'),(2,'XYZ'),(3,NULL),(4,NULL)
,(5,'PQR'),(6,NULL),(7,'ERT'),(8,NULL);

```

```sql
---sol
WITH CTE AS (
	SELECT 
		*,
		row_number() OVER(ORDER BY ID) AS rnk
	FROM tbl_Values
		)
SELECT
	Id,
	Code,
	(SELECT Code FROM CTE a WHERE a.rnk = 
		(SELECT MAX(b.rnk) FROM CTE b WHERE b.rnk <= c.rnk AND b.Code IS NOT NULL))
		AS Previous_NonNullCode 
FROM CTE c;
```

### Generate the data in the RANGE format
Check the below input data and expected output to start the range from 0 to first value and the first value to second value so on.

Expected Output:

|LowerRange | UpperRange |
|-----------|-----------|
| 0  |         108 |
| 108  |       116 |
| 116  |       226 | 
| 226  |       308 |
| 308  |       416 |
| 416  |       426 |
| 426  |       NULL |

```sql
DROP TABLE IF EXISTS Ranges;

CREATE TABLE Ranges
(Val INT);
 
INSERT INTO Ranges(Val) 
VALUES 
(108),(116),
(226),(308),
(416),(426);


```


```sql
---sol

WITH CTE AS(
SELECT 
    Val,
    row_number() over() AS rnk FROM
    (select 0 Val union select Val from Ranges) AS T)

SELECT 
    Val AS LowerRange,
    (SELECT b.Val FROM CTE a, CTE b WHERE a.rnk + 1 = b.rnk AND a.rnk = c.rnk) AS UpperRange
    FROM CTE C;
```





