---
title: "Avoid try-catch for manually handling transactions"
date: 2020-05-25T21:31:00+10:00
draft: false
---

### Avoid try-catch
We will show how failing the proper handling of a transaction can leave the database in an inconsistent state.
In this example, we will have a table with a unique constraint and we will try to insert twice in the same stored procedure. The goal is for the store procedure to not insert any of them and also to return an error.
By executing the code below, we can see this message in the output of SQL Server:
`Transaction count after EXECUTE indicates a mismatching number of BEGIN and COMMIT statements. Previous count = 0, current count = 1.`
The problem was a missing ; at the end of the `ROLLBACK TRANSACTION` that made the SQL server execute `ROLLBACK TRANSACTION THROW` in one statement.

```sql
CREATE TABLE ROD (
  ID BIGINT NOT NULL,
  M VARCHAR(50)
)
CREATE UNIQUE INDEX IX_ROD_ID ON dbo.ROD (ID) ON [PRIMARY]
GO

CREATE PROCEDURE dbo.stp_TryCatch
  @ID BIGINT
AS
BEGIN
  BEGIN TRANSACTION
  BEGIN TRY
    INSERT INTO ROD(ID, M) VALUES (@ID, 'stp_TryCatch - First insert')

    INSERT INTO ROD(ID, M) VALUES (@ID, 'stp_TryCatch - Second insert')

    COMMIT TRANSACTION
  END TRY
  BEGIN CATCH
    IF (ERROR_NUMBER() = 2627 OR ERROR_NUMBER() = 2601) -- If error is violation of unique index
      ROLLBACK TRANSACTION -- <--- MISSING ; at the end of this rollback
    THROW;
  END CATCH
END
GO

EXECUTE dbo.stp_TryCatch 1
SELECT * FROM ROD
```

To solve this problem we recommend stop manually handling exceptions but using XACT_ABORT ON.
This doesn't just solve the problem but also leave a cleaner code.

```sql
CREATE PROCEDURE dbo.stp_NoTryCatch
  @ID BIGINT
AS
BEGIN
  SET XACT_ABORT ON -- <--- By using XACT_ABORT ON the transaction will rollback on failure.
  SET NOCOUNT ON

  BEGIN TRANSACTION
  
  INSERT INTO ROD(ID, M) VALUES (@ID, 'stp_NoTryCatch - First insert')

  INSERT INTO ROD(ID, M) VALUES (@ID, 'stp_NoTryCatch - Second insert')

  COMMIT TRANSACTION
END
GO
```