---
title: "Datatypes Precision"
date: 2020-05-13T13:39:02+10:00
draft: false
---

### Money precision
SQL Server uses 8 bytes to represent MONEY giving it a precision from 
 - `-922,337,203,685,477.5808` to 
 - `922,337,203,685,477.5807`

[Reference](https://docs.microsoft.com/en-us/sql/t-sql/data-types/money-and-smallmoney-transact-sql)
After a small test, we realize that `.5808` is not the biggest precision point supported but `.9999`.
We also saw that an overflow in the fractional part won't generate an ERROR. i,e, `.00001` will be truncated to `0`, without throwing an exception.
Meanwhile, an overflow in the integer part will generate an ERROR.

```sql
CREATE TABLE #Money_Precision
(
	ID BIGINT IDENTITY(1,1) NOT NULL,
	Amount MONEY
)

INSERT INTO #Money_Precision (Amount)
VALUES
(0.001),
(0.0001),
(0.5808), -- biggest precision point supported?
(0.9999), -- biggest precision point tested and working
(0.00001), -- rounded as 0
(1.0),
(1000000.0),
(-922337203685477.5808), -- smallest supported number
--(-922337203685477.5809) -- Overflow ERROR
(922337203685477.5807) -- biggest supported number
--,(-922337203685477.5808) -- Overflow ERROR

SELECT * FROM #Money_Precision
```

### Small Money precision
SQL Server uses 4 bytes to represent MONEY giving it a precision from 
 - `-214,748.3648` to 
 - `214,748.3647`

[Reference](https://docs.microsoft.com/en-us/sql/t-sql/data-types/money-and-smallmoney-transact-sql)
After a small test, we realize that `.3648` is not the biggest precision point supported but `.9999`.
We also saw that an overflow in the fractional part won't generate an ERROR. i,e, `.00001` will be truncated to `0`, without throwing an exception.
Meanwhile, an overflow in the integer part will generate an ERROR.

```sql
CREATE TABLE #SmallMoney_Precision
(
	ID BIGINT IDENTITY(1,1) NOT NULL,
	Amount SMALLMONEY
)

INSERT INTO #SmallMoney_Precision (Amount)
VALUES
(0.001),
(0.0001),
(0.3647),
(0.3648), -- biggest precision point supported?
(0.9999), -- biggest precision point tested and working
(0.00001), -- rounded as 0
(1.0),
(100000.0),
(-214748.3648), -- smallest supported number
--(-214748.3649), -- Overflow ERROR
(214748.3647) -- biggest supported number
--,(214748.3648) -- Overflow ERROR

SELECT * FROM #SmallMoney_Precision
```