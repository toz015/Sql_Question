<!-- MarkdownTOC -->
- [Active User](#Active-User)
  - [Find the month over month percentage change for monthly active users (MAU)](#Find-the-month-over-month-percentage-change-for-monthly-active-users-MAU)
  - [Write a query that gets the number of retained users per month](#Write-a-query-that-gets-the-number-of-retained-users-per-month)
  - [Write a query to find many users last month did not come back this month](#Write-a-query-to-find-many-users-last-month-did-not-come-back-this-month)
  - [Create a table that contains the number of reactivated users per month](#Create-a-table-that-contains-the-number-of-reactivated-users-per-month)
  
- [Cumulative Sums](#Cumulative-Sums)
  - [Write a query to get cumulative cash flow for each day](#Write-a-query-to-get-cumulative-cash-flow-for-each-day-such-that-we-end-up-with-a-table-in-the-form-below)
- [Tree Structure Labeling](#Tree-Structure-Labeling)
  - [Write SQL such that we label each node as a “leaf”, “inner” or “root” node](#Write-SQL-such-that-we-label-each-node-as-a-leaf-inner-or-root--node)
- [Rolling Averages](#Rolling-Averages)
  - [Write a query to get 7-day rolling (preceding) average of daily sign ups](#Write-a-query-to-get-7-day-rolling-preceding-average-of-daily-sign-ups)
- [Multiple Join Conditions](#Multiple-Join-Conditions)
  - [Write a query to get the response time per email (id) sent to zach@g.com](#Write-a-query-to-get-the-response-time-per-email-id-sent-to-zach@g.com)
- [Window Function Pratice Problem](#Window-Function-Practice-Problem)
  - [Get the ID with the highest value](#Get-the-ID-with-the-highest-value)
  - [Average and rank with a window function (multi-part)](#Average-and-rank-with-a-window-function-multi-part)
  - [Adds a column with the rank of each employee based on their salary within their department，where the employee with the highest salary gets the rank of 1](#Adds-a-column-with-the-rank-of-each-employee-based-on-their-salary-within-their-department-where-the-employee-with-the-highest-salary-gets-the-rank-of-1)
- [Histograms](#Histograms)
  -[Buckets and range set](#Buckets-and-range-set)
- [Cross Join](#Cross-join)
  - [Write a query to get the pairs of states with total streaming amounts within 1000 of each other](#Write-a-query-to-get-the-pairs-of-states-with-total-streaming-amounts-within-1000-of-each-other)
  - [Follow up pairs question](#Follow-up-pairs-question)
- [Advancing Counting](#Advancing-Counting)
  - [Write a query to count the number of users in each class](#Write-a-query-to-count-the-number-of-users-in-each-class)
- [GG play](#GG-play)
  - [Count number of transactions in 2017](#Count-number-of-transaction-in-2017)
  - [Count number of transactions smaller than 5 and number of transactions larger than 5](#Count-number-of-transactions-smaller-than-5-and-number-of-transactions-larger-than-5)
  - [Find % of transactions <=$5, and >$5, so should return 33%, 67%  (so query should return ⅓, ⅔)](#Find percentage of transactions smaller than 5 and larger than 5)
  - [Find the days of which total order value larger than 100](#Find the days of which total order value larger than 100)
  - [Top 10 orders of each day by order value, data in 2019](#Top 10 orders of each day by order value data in 2019)
  

<!-- /MarkdownTOC -->

### Active User 
| user_id | date |
|---------|------------|
| 1 | 2018-01-01 |
| 4 | 2018-01-01 |
| 3 | 2018-01-01 |
| 1 | 2018-07-02 |
| ... | ... |
|5 | 2018-03-01 |

``` sql
DROP TABLE IF EXISTS logins;
CREATE TABLE logins (
    user_id            int,
    date             timestamp
);

INSERT INTO logins  VALUES (1, '2018-01-01');
INSERT INTO logins  VALUES (4, '2018-01-01');
INSERT INTO logins  VALUES (3, '2018-01-01');
INSERT INTO logins  VALUES (5, '2018-01-01');
INSERT INTO logins  VALUES (3, '2018-02-01');
INSERT INTO logins  VALUES (4, '2018-02-01');
INSERT INTO logins  VALUES (1, '2018-03-01');
INSERT INTO logins  VALUES (3, '2018-03-01');
INSERT INTO logins  VALUES (2, '2018-03-01');
INSERT INTO logins  VALUES (4, '2018-03-01');
INSERT INTO logins  VALUES (5, '2018-03-01');
```


#### Find the month-over-month percentage change for monthly active users (MAU)
-- solution
```sql
with cte as (
    select count(distinct user_id) as id_count, Date_trunc('Month', date) as Month
    from logins group by Date_trunc('Month', date))

select round(100.0 * (a.id_count - b.id_count)/b.id_count, 2) as percent_change, a.Month as cur_month, b.Month as prev_month
from cte a join cte b on a.Month = b.Month + interval '1 month';

```

#### Write a query that gets the number of retained users per month
In this case, retention for a given month is defined as the number of users who logged in that month who also logged in the immediately previous month
-- solution

```sql

with cte as(
    select user_id, date_trunc( 'Month', date) as Month
    from logins )
select count(distinct a.user_id), a.Month 
    from cte a, cte b where a.user_id = b.user_id and a.Month = b.Month + interval '1 month'
    group by a.Month;
```

#### Write a query to find many users last month did not come back this month
i.e. the number of churned users.
-- solution
```sql
 SELECT
    DATE_TRUNC('month', b.date) month_timestamp,
    COUNT(DISTINCT b.user_id) churned_users
    FROM
    logins a
    FULL OUTER JOIN
    logins b ON a.user_id = b.user_id
    AND DATE_TRUNC('month', a.date) =
        DATE_TRUNC('month', b.date) + interval '1 month'
    WHERE
        a.user_id IS NULL
    GROUP BY DATE_TRUNC('month', b.date)

```

#### Create a table that contains the number of reactivated users per month
-- solution
```sql
SELECT
    DATE_TRUNC('month', a.date) as month_timestamp,
    COUNT(DISTINCT a.user_id) reactivated_users,
    MAX(DATE_TRUNC('month', b.date)) most_recent_active_previously
    FROM
        logins a
    JOIN
        logins b ON a.user_id = b.user_id
    AND
        DATE_TRUNC('month', a.date) > DATE_TRUNC('month', b.date)
    GROUP BY DATE_TRUNC('month', a.date) 
    HAVING
           DATE_TRUNC('month', a.date) > MAX(DATE_TRUNC('month', b.date)) + interval '1 month'

```

### Cumulative Sums
**Context:** Say we have a table transactions in the form:
| date | cash_flow |
|------------|-----------|
| 2018-01-01 | -1000 |
| 2018-01-02 | -100 |
| 2018-01-03 | 50 |
| ... | ... |

Where cash_flow is the revenues minus costs for each day

```sql
DROP TABLE IF EXISTS transactions;
CREATE TABLE transactions (
    cash_flow           int,
    date             timestamp
);

INSERT INTO transactions VALUES (1, '2018-01-01');
INSERT INTO transactions VALUES (2, '2018-01-02');
INSERT INTO transactions VALUES (3, '2018-01-03');
INSERT INTO transactions VALUES (4, '2018-01-04');
INSERT INTO transactions VALUES (5, '2018-01-05');
INSERT INTO transactions VALUES (6, '2018-01-06');
INSERT INTO transactions VALUES (7, '2018-01-07');
INSERT INTO transactions VALUES (8, '2018-01-08');
INSERT INTO transactions VALUES (9, '2018-01-09');
INSERT INTO transactions VALUES (10, '2018-01-10');
```
#### Write a query to get cumulative cash flow for each day such that we end up with a table in the form below

-- solution
```sql
select t1.date, t1.cash_flow, sum(t2.cash_flow)
    from transactions t1, transactions t2
    where t1.date >= t2.date
    group by t1.date, t1.cash_flow
    order by date_trunc('day', t1.date)

```



### Tree Structure Labeling

**Context:**  Say you have a table tree with a column of nodes and a column corresponding parent nodes

|node | label |
|-----|-----|
|1 | Leaf |
|2 | Inner |
|3 | Inner |
|4 | Leaf |
|5 | Root |
```sql
DROP TABLE IF EXISTS TREE;

CREATE TABLE TREE(
    node    int,
    parent  int
    );
  
  
INSERT INTO TREE  VALUES (1, 2);
INSERT INTO TREE  VALUES (2, 5);
INSERT INTO TREE  VALUES (3, 5);
INSERT INTO TREE  VALUES (4, 3);
INSERT INTO TREE  VALUES (5, NULL);
```
#### Write SQL such that we label each node as a “leaf”, “inner” or “Root” node

```sql
select a.node,
       (case 
            when a.parent is NULL and b.parent is null then 'Root'
            when a.node is not NULL and b.parent is not null then 'Leaf'
            else 'Inner'
        end) as Label
     from tree a full join tree b on a.parent = b.node where a.node is not null
```

### Rolling Averages 
Note: there are different ways to compute rolling/moving averages. Here we'll use a preceding average which means that the metric for the 7th day of the month would be the average of the preceding 6 days and that day itself.

| date | sign_ups |
|------------|----------|
| 2018-01-01 | 10 |
| 2018-01-02 | 20 |
| 2018-01-03 | 50 |
| ... | ... |
| 2018-10-01 | 35 |

#### Write a query to get 7-day rolling (preceding) average of daily sign ups


```sql

-- solution 1

select t1.date, t1.sign_ups, (sum(t2.sign_ups) - sum(t3.sign_ups)) / 6
    from sign_ups t1, sign_ups t2, sign_ups t3
    where t1.date >= t2.date and t1.date = t3.date + interval '6 day'
    group by t1.date, t1.sign_ups
    order by date_trunc('day', t1.date);

-- solution 2

SELECT
    a.date,
    AVG(b.sign_ups) average_sign_ups
    FROM
        signups a
    JOIN
        signups b ON a.date <= b.date + interval '6 days' AND
        a.date >= b.date
GROUP BY  a.date

```
### Multiple Join Conditions
**Context:** Say we have a table emails that includes emails sent to and from zach@g.com:
```sql
DROP TABLE IF EXISTS emails;
CREATE TABLE emails(
    id           int,
    subject      Varchar(30),
    email_from        Varchar(60),
    email_to          Varchar(60),
    email_timestamp             timestamp
);

INSERT INTO emails VALUES (1, 'Yosemite', 'zach@g.com', 'thomas@g.com','2018-01-02 12:45:03');
INSERT INTO emails VALUES (2, 'Big Sur', 'sarah@g.com', 'thomas@g.com','2018-01-02 16:30:01');
INSERT INTO emails VALUES (3, 'Yosemite', 'thomas@g.com', 'zach@g.com','2018-01-02 16:35:04');
INSERT INTO emails VALUES (4, 'Running', 'jill@g.com', 'zach@g.com','2018-01-03 08:12:45');
INSERT INTO emails VALUES (5, 'Yosemite', 'zach@g.com', 'thomas@g.com','2018-01-03 14:02:01');
INSERT INTO emails VALUES (6, 'Yosemite', 'thomas@g.com', 'zach@g.com','2018-01-06 15:01:05');
INSERT INTO emails VALUES (7, 'Table', 'zach@g.com', 'thomas@g.com','2018-01-07 17:01:05');
INSERT INTO emails VALUES (8, 'Table', 'thomas@g.com', 'zach@g.com','2018-01-07 16:01:05');
```
#### Write a query to get the response time per email (id) sent to zach@g.com
Do not include ids that did not receive a response from zach@g.com. Assume each email thread has a unique subject. Keep in mind a thread may have multiple responses back-and-forth between zach@g.com and another email address.

| id | subject | from | to | timestamp |
|----|----------|--------------|--------------|---------------------|
| 1 | Yosemite | zach@g.com | thomas@g.com | 2018-01-02 12:45:03 |
| 2 | Big Sur | sarah@g.com | thomas@g.com | 2018-01-02 16:30:01 |
| 3 | Yosemite | thomas@g.com | zach@g.com | 2018-01-02 16:35:04 |
| 4 | Running | jill@g.com | zach@g.com | 2018-01-03 08:12:45 |
| 5 | Yosemite | zach@g.com | thomas@g.com | 2018-01-03 14:02:01 |
| 6 | Yosemite | thomas@g.com | zach@g.com | 2018-01-03 15:01:05 |
| .. | .. | .. | .. | .. |

-- solution
```sql
select a.id,
       min(b.email_timestamp) - a.email_timestamp as response_time
       from emails a , emails b 
           where 
           a.email_from = b.email_to
           and a.email_to = b.email_from
           and a.subject = b.subject
           and a.email_timestamp < b.email_timestamp
           and a.email_to = 'zach@g.com'
       group by 
           a.id, a.email_timestamp
```
### Window Function Practice Problem

**Context:** Say we have a table salaries with data on employee salary and
department in the following format:

|depname | empno | salary |
|-----------|-------|--------|
|develop | 11 | 5200 |
|develop | 7 | 4200 |
|develop | 9 | 4500 |
|develop | 8 | 6000 |
|develop | 10 | 5200 |
|personnel | 5 | 3500 |
|personnel | 2 | 3900 |
|sales | 3 | 4800 |
|sales | 1 | 5000 |
|sales | 4 | 4800 |

```sql
DROP TABLE IF EXISTS salaries;
CREATE TABLE salaries(
    depname           Varchar(30),
    empno     int,
    salary        int
);

INSERT INTO salaries VALUES ('develop', 11, 5200);
INSERT INTO salaries VALUES ('develop', 7, 4200);
INSERT INTO salaries VALUES ('develop', 9, 4500);
INSERT INTO salaries VALUES ('develop', 8, 6000);
INSERT INTO salaries VALUES ('develop', 10, 5200);
INSERT INTO salaries VALUES ('personnel', 5, 3500);
INSERT INTO salaries VALUES ('personnel', 2, 3900);
INSERT INTO salaries VALUES ('sales', 3, 4800);
INSERT INTO salaries VALUES ('sales', 1, 5000);
INSERT INTO salaries VALUES ('sales', 4, 4800);
```

#### Get the ID with the highest value

```sql
-- solution 1
With sal_rank as (select empno,
                         rank() over(order by salary desc) rnk from
                  salaries)
select empno from sal_rank where rnk = 1;

-- solution 2
select empno
    from salaries
    order by salary desc
    limit 1;
```
#### Average and rank with a window function (multi-part)
Write a query that returns the same table, but with a new column that has
average salary per depname. We would expect a table in the form:

|depname | empno | salary | avg_salary |
|-----------|-------|--------|------------|
|develop | 11 | 5200 | 5020 |
|develop | 7 | 4200 | 5020 |
|develop | 9 | 4500 | 5020 |
|develop | 8 | 6000 | 5020 |
|develop | 10 | 5200 | 5020 |
|personnel | 5 | 3500 | 3700 |
|personnel | 2 | 3900 | 3700 |
|sales | 3 | 4800 | 4867 |
|sales | 1 | 5000 | 4867 |
|sales | 4 | 4800 | 4867 |

```sql
-- solution 1
SELECT
*,
round(AVG(salary) OVER (PARTITION BY depname), 0) avg_salary
FROM
salaries;

-- solution 2
select t1.depname, t1.empno, t1.salary, t2.avg_salary
    from salaries t1, (select depname, round(avg(salary),0) as avg_salary
    from salaries 
    group by depname ) as t2
    where t1.depname = t2.depname;
```

#### Adds a column with the rank of each employee based on their salary within their department, where the employee with the highest salary gets the rank of 1
|depname | empno | salary | salary_rank |
|-----------|-------|--------|-------------|
|develop | 8 | 6000 | 1 |
|develop | 10 | 5200 | 2 |
|develop | 11 | 5200 | 2 |
|develop | 9 | 4500 | 4 |
|develop | 7 | 4200 | 5 |
|personnel | 2 | 3900 | 1 |
|personnel | 5 | 3500 | 2 |
|sales | 1 | 5000 | 1 |
|sales | 3 | 4800 | 2 |
|sales | 4 | 4800 | 2 |


```sql
-- solution 1
select t1.depname, t1.empno, t1.salary,  (case when count(t2.salary) != 0 then count(t2.salary) + 1
                                              else 1
                                          end) as salary_rank from salaries t1 full join salaries t2
     on t1.salary < t2.salary and t1.depname = t2.depname
     group by t1.depname, t1.empno, t1.salary order by t1.depname,t1.salary desc;
-- solution 2

select *,
        rank() over (partition by depname order by salary desc) salary_rank
        from salaries;
```
### Histograms
**Context:** Say we have a table sessions where each row is a video streaming session with length in seconds:
| session_id | length_seconds |
|------------|----------------|
| 1 | 23 |
| 2 | 453 |
| 3 | 27 |
| .. | .. |

#### Buckets and range set
Write a query to count the number of sessions that fall into bands of size 5, i.e for the above snippet, produce something akin to
| bucket | count |
|---------|-------|
| 20-25 | 2 |
| 450-455 | 1 |
```sql
-- solution 1

SELECT COUNT(*) AS cnt,
       FLOOR(length_seconds/5) AS prange,
       CONCAT(5*FLOOR(length_seconds/5), '-', 5*FLOOR(length_seconds/5)+4) AS rstr
FROM sessions
GROUP BY prange order by prange;

-- solution 2
WITH cte AS (SELECT
            session_id,
            FLOOR(length_seconds/5) as bin_label 
            FROM sessions
)
SELECT
    CONCAT(bin_label*5, '-', bin_label*5+4) bucket,
    COUNT(DISTINCT session_id) count from cte
    GROUP BY
    bucket
    ORDER BY
    bucket ASC;
```

### Cross Join
**Context:** Say we have a table state_streams where each row is a state and the total number of hours of streaming from a video hosting service:
| state | total_streams |
|-------|---------------|
| NC | 34569 |
| SC | 33999 |
| CA | 98324 || MA | 19345 |
| .. | .. |

#### Write a query to get the pairs of states with total streaming amounts within 1000 of each other

For the snippet above, we would want to see something like:
| state_a | state_b |
|---------|---------|
| NC | SC |
| SC | NC |

```sql
-- solution
SELECT a.state as state_a,
       b.state as state_b
       From state_stream a,
            state_stream b
       where a.state <> b.state
       and abs(a.total_streams - b.total_streams) < 1000;
       
```
#### Follow up pairs question
**Task**: How could you modify the SQL from the solution to Part 1 of this question so that duplicates are removed? For example, if we used the sample table from Part 1, the pair NC and SC should only appear in one row instead of two
```sql
-- solution
SELECT a.state as state_a,
       b.state as state_b
       From state_stream a,
            state_stream b
       where a.state > b.state
       and abs(a.total_streams - b.total_streams) < 1000;
```

### Advancing Counting
**Context:** Say I have a table table in the following form, where a user can be mapped to multiple values of class:
| user | class |
|------|-------|
| 1 | a |
| 1 | b |
| 1 | b |
| 2 | b |
| 3 | a |
#### Write a query to count the number of users in each class
Any user who has label a and b getssorted into b, any user with just a gets sorted into a and any user with just b gets into b
For table that would result in the following table:
| class | count |
|-------|-------|
| a | 1 |
| b | 2 |

```sql
-- solution 1
SELECT
  'a' as class,
  COUNT(DISTINCT user_id) - (SELECT COUNT(DISTINCT user_id) FROM classes WHERE class = 'b') count
  from classes
UNION
SELECT
  'b' as class,
  (SELECT COUNT(DISTINCT user_id) FROM classes WHERE class = 'b') count
  from classes
  order by class asc;

-- solution 2

with cte as (
    SELECT
         user_id,
         SUM(CASE WHEN class = 'b' THEN 1 ELSE 0 END) as num_b
         FROM
             classes
        group by user_id),
    user_label as(
           select user_id,
           case when num_b > 0 then 'b' else 'a' 
        end as class
        from cte)
select class,
       count( distinct user_id) as count
       from user_label 
    group by class
    order by class ASC;
    
-- solution 3 base on b > a 
WITH max_class AS (
    SELECT
    user_id,
    MAX(class) as class
    FROM classes
    GROUP BY
    user_id
)
SELECT
    class,
    COUNT(user_id)
    FROM
        max_class
    GROUP BY
    class
    order by class ASC;
        
                  
```

### GG play

Table: order
|order_time |order_value |
|-------|-------|
| 2/1/19 |  $6  |
| 3/5/19 |  $10 | 
| 4/3/18 |  $4  |


#### Count number of transaction in 2017
```sql
SELECT COUNT(*) AS numTransaction
  FROM order
  WHERE year(order_time) = '2017';
```
#### Count number of transactions smaller than 5 and number of transactions larger than 5

```sql

SELECT 
    SUM(CASE WHEN order_value <=5 THEN 1 ELSE 0) AS small_tran,
    SUM(CASE WHEN order_value > 5 THEN 1 ELSE 0) AS large_tran
FROM order;
 ```

####Find percentage of transactions smaller than 5 and larger than 5

```
SELECT 
    SUM(CASE WHEN order_value <=5 THEN 1 ELSE 0)/COUNT(*) AS small_tran_percent,
    SUM(CASE WHEN order_value > 5 THEN 1 ELSE 0)/COUNT(*) AS large_tran_percent
FROM order;
       
```

#### Find the days of which total order value larger than 100
```
SELECT
    order_time, total_value
FROM (
SELECT
    order_time, sum(order_value) AS total_value
  FROM order 
  GROUP BY 
    order_time) AS tb1
WHERE 
  total_value > 100

```
#### Top 10 orders of each day by order value data in 2019

```
SELECT 
    order_time, order_value
FROM
(
SELECT
  order_time,
  order_value,
  RANK()OVER(PARTITION BY order_time ORDER BY order_value DESC) AS rank
FROM 
  order
  ) AS tb1
WHERE rank <= 10;

```
