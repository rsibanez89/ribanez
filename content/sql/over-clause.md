---
title: "Over clause"
date: 2020-05-26T18:58:39+10:00
draft: false
---

# The OVER clause
Determines the partitioning and ordering of a rowset before the associated window function is applied [Reference](https://docs.microsoft.com/en-us/sql/t-sql/queries/select-over-clause-transact-sql). Here, we will apply the OVER to the SUM function and see how we can solve some big problems.

## The SELECT ... SUM(...) OVER clause
The OVER clause modifies the rows before the SUM is applied. You can specify the columns, the order, and the row or range of rows.

Let's see an example, we have a `Transactions` table that registers which user has made a payment or has been charged in the system.

```sql
CREATE TABLE Transactions (
	ID BIGINT NOT NULL IDENTITY(1,1),
	UserID BIGINT NOT NULL,
	DateTimeStamp DateTime,
	Description VARCHAR(250),
	Amount MONEY
)

INSERT INTO Transactions
(DateTimeStamp, UserID, Description, Amount)
VALUES
('2019-10-09', 1, 'Charges 1', -3.5),
('2019-10-10', 1, 'Charges 2', -3.5),
('2019-10-11', 1, 'Charges 3', -4.5),
('2019-10-13', 1, 'Charges 4', -2.5),
('2019-10-13', 1, 'Payment  ', 5.5),
('2019-10-14', 1, 'Charges 5', -2.5),
('2019-10-15', 1, 'Payment  ', 5.5),

('2019-10-01', 2, 'Charges 1', -1.5),
('2019-10-02', 2, 'Charges 2', -2.5),
('2019-10-08', 2, 'Charges 3', -3.5),
('2019-10-08', 2, 'Charges 4', -2.5),
('2019-10-09', 2, 'Payment  ', 5.5),
('2019-10-12', 2, 'Charges 5', -1.5),
('2019-10-15', 2, 'Payment  ', 5.5)
```

If we want to get their balance at any point in time, the query is as simple as doing the following:
```sql
SELECT t.UserID, SUM(Amount) 'Balance'
FROM Transactions t
GROUP BY t.UserID
```
| UserID | Balance |
|--------|---------|
| 1      | -5.50   |
| 2      | -0.50   |

That same query, can be done by using the OVER clause giving the same result.
```sql
SELECT DISTINCT(t.UserID), SUM(t.Amount) OVER(PARTITION BY t.UserID) 'Balance'
FROM Transactions t
```
Sometimes, this way of writing queries is more readable. Some other times, this way is the only way of writing a query.
For example, if we want to get the running balance at each point in time for each user:
```sql
SELECT 
	t.ID 'TransactionID',
	t.DateTimeStamp,
	t.UserID,
	t.Amount,
	SUM(t.Amount) OVER(PARTITION BY t.UserID ORDER BY t.DateTimeStamp ROWS UNBOUNDED PRECEDING) 'Balance'
FROM Transactions t
```
| TransactionID | DateTimeStamp           | UserID | Amount | Balance |
|---------------|-------------------------|--------|--------|---------|
| 1             | 2019-10-09 00:00:00.000 | 1      | -3.50  | -3.50   |
| 2             | 2019-10-10 00:00:00.000 | 1      | -3.50  | -7.00   |
| 3             | 2019-10-11 00:00:00.000 | 1      | -4.50  | -11.50  |
| 4             | 2019-10-13 00:00:00.000 | 1      | -2.50  | -14.00  |
| 5             | 2019-10-13 00:00:00.000 | 1      | 5.50   | -8.50   |
| 6             | 2019-10-14 00:00:00.000 | 1      | -2.50  | -11.00  |
| 7             | 2019-10-15 00:00:00.000 | 1      | 5.50   | -5.50   |
| 8             | 2019-10-01 00:00:00.000 | 2      | -1.50  | -1.50   |
| 9             | 2019-10-02 00:00:00.000 | 2      | -2.50  | -4.00   |
| 10            | 2019-10-08 00:00:00.000 | 2      | -3.50  | -7.50   |
| 11            | 2019-10-08 00:00:00.000 | 2      | -2.50  | -10.00  |
| 12            | 2019-10-09 00:00:00.000 | 2      | 5.50   | -4.50   |
| 13            | 2019-10-12 00:00:00.000 | 2      | -1.50  | -6.00   |
| 14            | 2019-10-15 00:00:00.000 | 2      | 5.50   | -0.50   |

Imagine having to do that query without using the OVER clause.

What if we also want to add another column to show their previous balance?
```sql
SELECT 
	t.ID 'TransactionID',
	t.DateTimeStamp,
	t.UserID,
	t.Amount,
	SUM(t.Amount) OVER(PARTITION BY t.UserID ORDER BY t.DateTimeStamp ROWS UNBOUNDED PRECEDING) 'Balance',
	SUM(t.Amount) OVER(PARTITION BY t.UserID ORDER BY t.DateTimeStamp ROWS BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING) 'Previous Balance'
FROM Transactions t
```
| TransactionID | DateTimeStamp           | UserID | Amount | Balance | Previous Balance |
|---------------|-------------------------|--------|--------|---------|------------------|
| 1             | 2019-10-09 00:00:00.000 | 1      | -3.50  | -3.50   | NULL             |
| 2             | 2019-10-10 00:00:00.000 | 1      | -3.50  | -7.00   | -3.50            |
| 3             | 2019-10-11 00:00:00.000 | 1      | -4.50  | -11.50  | -7.00            |
| 4             | 2019-10-13 00:00:00.000 | 1      | -2.50  | -14.00  | -11.50           |
| 5             | 2019-10-13 00:00:00.000 | 1      | 5.50   | -8.50   | -14.00           |
| 6             | 2019-10-14 00:00:00.000 | 1      | -2.50  | -11.00  | -8.50            |
| 7             | 2019-10-15 00:00:00.000 | 1      | 5.50   | -5.50   | -11.00           |
| 8             | 2019-10-01 00:00:00.000 | 2      | -1.50  | -1.50   | NULL             |
| 9             | 2019-10-02 00:00:00.000 | 2      | -2.50  | -4.00   | -1.50            |
| 10            | 2019-10-08 00:00:00.000 | 2      | -3.50  | -7.50   | -4.00            |
| 11            | 2019-10-08 00:00:00.000 | 2      | -2.50  | -10.00  | -7.50            |
| 12            | 2019-10-09 00:00:00.000 | 2      | 5.50   | -4.50   | -10.00           |
| 13            | 2019-10-12 00:00:00.000 | 2      | -1.50  | -6.00   | -4.50            |
| 14            | 2019-10-15 00:00:00.000 | 2      | 5.50   | -0.50   | -6.00            |

## Conclusion
Using the OVER clause may improve the readability of the query but some other times the OVER clause is the only way of writing a query.