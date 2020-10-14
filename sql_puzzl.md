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
- [Do basic validation of Email Address](#Do-basic-validation-of-Email-Address)
- [Do the Multiplication for each Group](#Do-the-Multiplication-for-each-Group)
- [Get the Last Sunday of Previous Week](#Get-the-Last-Sunday-of-Previous-Week)
- [Check a String Is Number or Not](#Check-a-String-Is-Number-or-Not)
- [Get the last three Records of a table](#Get-the-last-three-Records-of-a-table)
- [Find Correlation Coefficients for the Run of Cricket Players](#Find-Correlation-Coefficients-for-the-Run-of-Cricket-Players)
- [Delete Duplicate Data without Primary key, ROW_NUMBER()](#Delete-Duplicate-Data-without-Primary-key)
- [Generate Calendar Data for 19th Century](#Generate-Calendar-Data-for-19th-Century)
- [Use Recursive CTE, and list out the Years from Dates](#Use-Recursive-CTE-and-list-out-the-Years-from-Dates)
- [Calculate the Power of Three](#Calculate-the-Power-of-Three)
- [Find the Median Value from the Given Number](#Find-the-Median-Value-from-the-Given-Number)





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
|---------------|----------|----|---------------------|---------------|
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

### Do basic validation of Email Address

Check the below input data and expected output and perform the basic Email Validation using simple LIKE predicate.

```sql
DROP TABLE IF EXISTS tbl_emails;

CREATE TABLE tbl_emails (emailname VARCHAR(400));
INSERT INTO tbl_emails VALUES ('abc@gmail.com'),('@gmail.com'),('abcxyz.gmail.com'),('xyz@yahoo.com');
```
```sql
---sol
SELECT emailname,
       CASE WHEN emailname LIKE '%_@_%_._%' THEN TRUE ELSE FALSE END IsCorrectEmail
       FROM tbl_emails;
```

### Do the Multiplication for each Group
Check the below input data and expected output to get the multiplication for each Group

Expexted Outpu:

|GroupName | GroupValue |
|----------|----------------------|
|A  |      700              |
|B  |       1620000       |
|C  |        12000        |
|D  |         60000       |    

```sql
DROP TABLE IF EXISTS  tbl_GroupNumber;

CREATE TABLE tbl_GroupNumber (GroupName VARCHAR(10), GroupValue INT);
 
INSERT INTO tbl_GroupNumber VALUES
('A',10),('B',15),('C',20),('D',30)
,('B',30),('C',60),('D',40),('D',50)
,('A',70),('B',40),('C',10),('B',90);
```
```sql
---sol1

WITH RECURSIVE cte AS 
(
    SELECT GroupName, GroupValue AS Multiply, row_number() over (partition by GroupName) as rnk
    FROM tbl_GroupNumber
    UNION ALL
    SELECT ff.GroupName, cte.Multiply * ff.GroupValue as multiply, ff.rnk FROM
    ( SELECT GroupName, GroupValue, row_number() over (partition by GroupName) as rnk
    FROM tbl_GroupNumber) ff
        INNER JOIN cte
        ON ff.rnk -1= cte.rnk AND ff.GroupName = cte.GroupName
)

SELECT groupname,
        MAX(multiply) as GroupValue
        FROM CTE
        GROUP BY groupname
        ORDER BY groupname;

---sol2
select gn.groupname, round(EXP(SUM(LN(gn.groupvalue))))
from tbl_GroupNumber gn
group by gn.groupname;


```


### Get the Last Sunday of Previous Week

Check the below input date and expected output to find the last Sunday of the previous week in SQL Server.

```sql
SELECT CURRENT_DATE - case when extract(dow from  current_date) = 0 then 7 * interval '1 DAY' else extract(dow from  current_date)  * interval '1 DAY' end;
```

### Check a String Is Number or Not

```sql
select test_data ~ '^[-+]?[0-9]*\.?[0-9]+([eE][-+]?[0-9]+)?$' ;

```

### Get the last three Records of a table
Check the below input data and expected output to get the last three records of a table.

Expected Output:


|ID |         Name|
|-----------|----------|
|7   |        WER |
|8  |         CVC |
|9 |           ASD|

```sql
Drop Table if exists tbl_TestTable;



CREATE TABLE tbl_TestTable (ID INT, Name VARCHAR(10));
 
INSERT INTO tbl_TestTable 
VALUES (1,'ABC'),(2,'XYZ'),(3,'ERZ')
,(4,'HYU'),(5,'BNM'),(6,'WER')
,(7,'WER'),(8,'CVC'),(9,'ASD');

```
```sql
---sol
SELECT ID,
        Name 
        FROM tbl_TestTable
        WHERE ID + 3 > (SELECT MAX(ID) FROM tbl_TestTable);
```

### Find Correlation Coefficients for the Run of Cricket Players

Check the below input data and expected output to find the Correlation Coefficients for the run of Cricket Players.

Expected Output:


|Player1 |    Player2 |   PlayersCoefficientRuns |
|----------|----------|----------------------|
|Dhoni |     Kohli   |   0.332662790481163 |
|Dhoni  |    Yuvraj   |  -0.311842111156846 |
|Kohli |     Yuvraj   |  -0.915321318268559 |

```sql
CREATE TABLE tbl_Players (PlayerName VARCHAR(10), Runs INT, MatchYear INT);
 
INSERT INTO tbl_Players VALUES 
('Dhoni',1200, 2014),('Kohli',1800, 2014),('Yuvraj',1000, 2014)
,('Dhoni',1300, 2015),('Kohli',1500, 2015),('Yuvraj',900, 2015)
,('Dhoni',1100, 2016),('Kohli',1300, 2016),('Yuvraj',1200, 2016)
,('Dhoni',1800, 2017),('Kohli',1100, 2017),('Yuvraj',1300, 2017)
,('Dhoni',2000, 2018),('Kohli',2200, 2018),('Yuvraj',700, 2018);

```
```sql
---sol1


WITH CTE AS (SELECT a.PlayerName as player1,
       b.PlayerName as player2,
       a.Runs as player1_run,
       b.Runs as player2_run
       FROM tbl_Players a, tbl_Players b
       WHERE a.PlayerName > b.PlayerName
       AND a.MatchYear = b.MatchYear),
CTE2 AS(SELECT 
           player1,
           player2,
           COUNT(*) AS num,
           SUM(player1_run) AS player1_sum,
           SUM(player2_run) AS player2_sum,
           SUM(player1_run * player2_run) AS total_sum,
           SUM(player1_run * player1_run) AS player1_2,
           SUM(player2_run * player2_run) AS player2_2
           FROM CTE GROUP BY player1, player2)
SELECT player1,
       player2,
       (num * (total_sum) - player1_sum * player2_sum)/ sqrt((num * player1_2 - player1_sum * player1_sum)
                                                             * (num * player2_2 - player2_sum * player2_sum))
                                                             AS correlation_coefficient
       FROM CTE2;

---sol2

select
	Player1
	,Player2
	,
    Round((Avg(Runs1 * Runs2) - (Avg(Runs1) * Avg(Runs2))) / 
	(stddev_pop(Runs1) * stddev_pop(Runs2)), 5) as PlayersCoefficientRuns
from  
(
	select 
		a.PlayerName as Player1
		,a.Runs as Runs1
		,b.PlayerName as Player2
		,b.Runs as Runs2
		,a.MatchYear 
	from tbl_Players a
	cross join tbl_Players b
	where b.PlayerName>a.PlayerName and a.MatchYear=b.MatchYear
) as t
group by Player1, Player2
```

### Delete Duplicate Data without Primary key

Check the below input data and expected output for deleting the duplicate data by giving an ID. The table doesn’t have any primary key and writes logic without using ROW_NUMBER().

```sql


CREATE TABLE tbl_TestTable (ID INT, Name VARCHAR(10));
 
INSERT INTO tbl_TestTable 
VALUES (1,'ABC'),(1,'ABC'),(3,'ERZ')
,(4,'HYU'),(5,'BNM'),(1,'ABC')
,(7,'WER'),(3,'ERZ'),(7,'WER');
```
```sql
--sol
DELETE FROM  tbl_TestTable
WHERE id IN
(
SELECT ID
FROM tbl_TestTable
Group By NAME,id
HAVING COUNT(id) >1
)
```


### Generate Calendar Data for 19th Century

```sql

WITH recursive calendar AS (
SELECT to_date('01.01.1900', 'dd.mm.yyyy') as fdate 
UNION ALL
SELECT fdate + 1 FROM calendar
WHERE fdate + 1 <= '12.31.2000'
)
SELECT fdate FROM calendar order by fdate desc;
```


### Use Recursive CTE and list out the Years from Dates

```sql
WITH recursive calendar AS (
SELECT extract(year from to_date('01.01.2010', 'dd.mm.yyyy') ) as fdate 
UNION ALL
SELECT fdate + 1  FROM calendar
WHERE fdate < extract(year from to_date('01.01.2018', 'dd.mm.yyyy'))
)
SELECT fdate FROM calendar order by fdate desc;
```

### Calculate the Power of Three

Check the below input data and expected output to get the power of three within first 100 numbers.

Expected Output:

|PowerOfThree|
|------------|
|1|
|3|
|9|
|27|
|81|

```sql

WITH RECURSIVE CTE AS (
Select 0 as pw, 1 AS cnt Union
Select pw + 1 AS pw, cnt * 3 AS count
    FROM CTE WHERE
    pw < 5) 
SELECT * FROM CTE 
```

### Find the Median Value from the Given Number

Check the below input data and expected output and find the median value from the given range of Numbers.
```sql
CREATE TABLE tbl_TestMedian (Number INT)
INSERT INTO tbl_TestMedian VALUES (1),(5),(10),(16),(20)
```

```sql
---sol1
SELECT NUMBER 
    FROM tbl_TestMedian t1
    WHERE ABS(
        (SELECT COUNT(*) FROM tbl_TestMedian t2 where t2.Number >= t1.Number ) -
        (SELECT COUNT(*) FROM tbl_TestMedian t3 where t3.Number <= t1.Number)) <= 1

---sol2

WITH CTE AS (
    SELECT NUMBER,
           ROW_NUMBER () OVER(ORDER BY NUMBER) AS rnk,
            COUNT(*) OVER() AS total
    FROM tbl_TestMedian)

SELECT NUMBER
    FROM CTE 
    WHERE CASE WHEN total & 1 = 1 THEN rnk = (total + 1) / 2
        ELSE rnk = total / 2 OR rnk = total/ 2 + 1 END
```
