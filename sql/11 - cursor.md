---
title: Cursors
---

## Cursors

```sql
-- Temporarily change delimiter
DELIMITER $$

CREATE PROCEDURE createEmailList
(
  INOUT emailList varchar(4000)
)
BEGIN

  -- all variables should be declared before cursor
  DECLARE finished INTEGER DEFAULT 0;
  DECLARE emailAddress varchar(100) DEFAULT "";

  -- declare cursor for employee email
  DEClARE curEmail CURSOR FOR
  SELECT email FROM employees;

  -- declare NOT FOUND handler
  -- (cursor could not retrieve any more rows)
  DECLARE CONTINUE HANDLER FOR
  NOT FOUND
  SET finished = 1;

  -- open cursor
  OPEN curEmail;

  getEmail: LOOP
    -- extract row from cursor
    FETCH curEmail INTO emailAddress;

    -- check if cursor is out of rows
    IF finished = 1 THEN
      LEAVE getEmail;
    END IF;

    -- build email list
    SET emailList = CONCAT(emailAddress,";",emailList);
  END LOOP getEmail;

  -- close cursor to free-up memory
  CLOSE curEmail;

END$$

-- Reset delimiter
DELIMITER ;
```

```sql
SET @emailList = "";
CALL createEmailList(@emailList);
SELECT @emailList;
```
