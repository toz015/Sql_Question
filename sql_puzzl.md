<!-- MarkdownTOC -->
- [Find the Order basis on thier Status and Step](#Find-the-Order-basis-on-thier-Status-and-Step)
- [Generate the range of data basis on common combination](#Generate-the-range-of-data-basis-on-common-combination)
- [To generate the account balance column for Bank Accounts](#To-generate-the-account-balance-column-for-Bank-Accounts)

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


```sql

```

