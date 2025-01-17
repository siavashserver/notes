---
title: Functions
---

## Creation

```sql
-- Temporarily change delimiter
DELIMITER $$

CREATE FUNCTION CustomerLevel
(
  credit DECIMAL(10,2)
)
RETURNS VARCHAR(20) -- Functions only return a single value (note: SQLServer can RETURNS TABLE)
DETERMINISTIC -- always returns same value for known value
BEGIN

  DECLARE customerLevel VARCHAR(20);

  IF credit > 50000 THEN
    SET customerLevel = 'PLATINUM';
  ELSEIF (credit >= 50000 AND credit <= 10000) THEN
    SET customerLevel = 'GOLD';
  ELSEIF credit < 10000 THEN
    SET customerLevel = 'SILVER';
  END IF;

  -- return the customer level
  RETURN (customerLevel);

END$$

-- Reset delimiter
DELIMITER ;
```

```sql
SELECT customerName, CustomerLevel(creditLimit)
FROM customers
ORDER BY customerName;
```

## Removal

```sql
DROP FUNCTION IF EXISTS CustomerLevel;
```
