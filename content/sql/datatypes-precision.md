---
title: "Datatypes Precision"
date: 2020-05-13T13:39:02+10:00
draft: false
---

### Money precision
SQL Server uses 8 bytes to represent MONEY giving it a precision from 
 - `-922,337,203,685,477.5808` to 
 - `922,337,203,685,477.5807`

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
(0.5807), -- biggest precision point supported
(0.5808),
(0.6999),
(0.9999), -- biggest precision point tested and working
(0.00001), -- rounded as 0
(1.0),
(1000000.0),
(922337203685477) -- biggest precision supported
-- ,(922337203685478), -- Overflow ERROR

SELECT * FROM #Money_Precision
```