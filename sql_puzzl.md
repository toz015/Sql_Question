## SQL PUZZLE
reference link:
https://www.dbrnd.com/sql-interview-the-ultimate-sql-puzzles-and-sql-server-advance-sql-queries/

<!-- MarkdownTOC -->
- [Find the Order basis on thier Status and Step](#Find-the-Order-basis-on-thier-Status-and-Step)
- [Generate the range of data basis on common combination](#Generate-the-range-of-data-basis-on-common-combination)
- [To generate the account balance column for Bank Accounts](#To-generate-the-account-balance-column-for-Bank-Accounts)
- [Place NULL for repeating values](#Place-NULL-for-repeating-values)

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
| --------------- | ---------- | ---- | --------------------- | --------------- |
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



