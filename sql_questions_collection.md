<!-- MarkdownTOC -->
- [Active User](#Active User)
  - [Find the month over month percentage change for monthly active users (MAU)](Find-the-month-over-month-percentage-change-for-monthly-active-users-MAU)
  - [Write a query that gets the number of retained users per month](Write-a-query-that-gets-the-number-of-retained-users-per-month)
  - [Write a query to find many users last month did not come back this month](Write-a-query-to-find-many-users-last-month-did-not-come-back-this-month)
  -[Create a table that contains the number of reactivated users per month](Create-a-table-that-contains-the-number-of-reactivated-users-per-month)
  
-[Cumulative Sums](#Cumulative Sums)
  -[Write a query to get cumulative cash flow for each day](Write-a-query-to-get-cumulative-cash-flow-for-each-day-such-that-we-end-up-with-a-table-in-the-form-below)
-[Tree Structure Labeling](#Tree Structure Labeling)
  -[Write SQL such that we label each node as a “leaf”, “inner” or “Root” node](Write-SQL-such-that-we-label-each-node-as-a-leaf-inner-or-Root-node)
-[Rolling Averages](#Rolling Averages)
  -[Write a query to get 7-day rolling (preceding) average of daily sign ups](Write-a-query-to-get-7-day-rolling-preceding-average-of-daily-sign-ups)
-[Multiple Join Conditions](#Multiple Join Conditions)
  -[Write a query to get the response time per email (id) sent to zach@g.com](Write-a-query-to-get-the-response-time-per-email-id-sent-to-zach@g.com)


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

|node | label |
|-----|-----|
|1 | Leaf |
|2 | Inner |
|3 | Inner |
|4 | Leaf |
|5 | Root |
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
\*
-- solution 1
*\
select t1.date, t1.sign_ups, (sum(t2.sign_ups) - sum(t3.sign_ups)) / 6
    from sign_ups t1, sign_ups t2, sign_ups t3
    where t1.date >= t2.date and t1.date = t3.date + interval '6 day'
    group by t1.date, t1.sign_ups
    order by date_trunc('day', t1.date);
\*
-- solution 2
*\
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
#### Write a query to get the response time per email (id) sent to zach@g.com
Do not include ids that did not receive a response from zach@g.com. Assume each email thread has a unique subject. Keep in mind a thread may have multiple responses back-and-forth between zach@g.com and another email address.
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
