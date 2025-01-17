---
title: T-SQL
---

## Declaring Variables

```sql
DECLARE @var_name INT = 0;
SET @var_name = @var_name + 5;
SELECT @var_name;
-- 5
```

---

## Unicode Strings

```sql
-- ASCII: CHAR, VARCHAR
DECLARE @text CHAR(10) = 'Hello';

-- Unicode: NCHAR, NVARCHAR
DECLARE @text NVARCHAR(10) = N'سلام';
```

---

## Column Addition

```sql
ALTER TABLE table_name
ADD column_name VARCHAR(50);

-- Unlike MySQL, COLUMN keyword is not required
-- ADD COLUMN column_n VARCHAR(50);
```

## Column Removal

```sql
ALTER TABLE table_name
DROP COLUMN column_name;
```

## Column Modification

```sql
ALTER TABLE table_name
ALTER COLUMN column_name VARCHAR(255);
```

---

## Change Column Default Value

```sql
ALTER TABLE table_name
ADD CONSTRAINT constraint_name DEFAULT 1 FOR column_name;

-- MySQL
-- ALTER TABLE table_name
-- ALTER COLUMN column_name SET DEFAULT 1;
```

---

## LIMIT OFFSET

```sql
-- Skip 0 rows and return only the first 10 rows
SELECT DepartmentID, Name, GroupName
FROM HumanResources.Department
ORDER BY DepartmentID
OFFSET 0 ROWS
FETCH NEXT 10 ROWS ONLY;
```

---

## Functions

### Creating Functions

```sql
CREATE FUNCTION func_Reverse(@Str nvarchar(4000))
RETURNS nvarchar(4000)
AS
BEGIN
  DECLARE @L int = LEN(@Str)
  DECLARE @Result nvarchar(4000) = N''
  DECLARE @i int = @L

  WHILE @i >= 1
  BEGIN
    SET @Result = @Result + SUBSTRING(@Str, @i, 1)
    SET @i = @i - 1
  END

  RETURN @Result
END
```

```sql
SELECT dbo.func_Reverse('Test')
-- tseT
```

### Creating Table Valued Functions

```sql
CREATE FUNCTION func_CustomerList (@Country nvarchar(40))
RETURNS TABLE
AS
RETURN
(
  SELECT CustomerID, Companyname, Country, City
  FROM Customers
  WHERE Country = @Country
);
```

```sql
SELECT *
FROM func_CustomerList(N'Sweden') AS C
INNER JOIN Orders AS O
ON C.CustomerID = O.CustomerID
```

---

## Creating Procedures

```sql
CREATE PROCEDURE procedure_name
(
  @param_name INT = NULL
)
AS
BEGIN
  SELECT name
  FROM users
  WHERE id = @param_name
END
```

## Executing Procedures

```sql
EXECUTE procedure_name @param_name = 1

-- EXECUTE keyword, and specifing param_names are optional
procedure_name 1
```

## Output Params

```sql
CREATE PROCEDURE procedure_name
(
  @in_param VARCHAR(100),
  @out_param1 VARCHAR(100) OUTPUT,
  @out_param2 VARCHAR(100) OUTPUT
)
AS
BEGIN
  SELECT @out_param1 = @in_param + ' p1'
  SELECT @out_param2 = @in_param + ' p2'
  RETURN
END
```

```sql
DECLARE @p1 VARCHAR(100)
DECLARE @p2 VARCHAR(100)
EXECUTE procedure_name 'input', @p1 OUTPUT, @p2 OUTPUT
```

## IF / ELSE

```sql
CREATE PROCEDURE procedure_name
(
  @in_param VARCHAR(100)
)
AS
BEGIN
  IF (condition)
  BEGIN
    -- true case
  END
  ELSE
  BEGIN
    -- false case
  END
END
```

## WHILE

```sql
CREATE PROCEDURE procedure_name
(
  @in_param VARCHAR(100)
)
AS
BEGIN
  WHILE (condition)
  BEGIN
    -- loop body
  END
END
```

---

## Creating Triggers

```sql
CREATE TRIGGER trigger_name
ON table_name
{ FOR | AFTER } { INSERT | UPDATE | DELETE }
AS
BEGIN
-- ACTION
END
```

---

## Transactions

```sql
-- 1. Begin transaction
-- transaction name is optional
-- WITH MARK optionally logs the transaction
BEGIN TRANSACTION transaction_name
WITH MARK N'transaction description'

-- 2. Make changes
UPDATE table_name
SET column_name = 0;

-- 3a. Save changes
COMMIT TRANSACTION transaction_name;

-- 3b. Failure
ROLLBACK TRANSACTION transaction_name;;
```
