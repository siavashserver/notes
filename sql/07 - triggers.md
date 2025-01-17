---
title: Triggers
---

## Creation

```sql
CREATE TRIGGER trigger_name
{ BEFORE | AFTER } { INSERT | UPDATE | DELETE }
ON table_name
{ FOR EACH ROW | FOR EACH STATEMENT }
trigger_body;
```

```sql
CREATE OR MODIFY TRIGGER before_insert_item
BEFORE INSERT
ON item_table
FOR EACH ROW
CALL stored_procedure_name;
```

```sql
DELIMITER $$

CREATE TRIGGER before_billing_update
BEFORE UPDATE
ON billings
FOR EACH ROW
BEGIN

  IF NEW.amount > OLD.amount * 10 THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'New amount cannot be 10 times greater than the current amount.';
  END IF;

END$$

DELIMITER ;
```

```sql
DELIMITER $$

CREATE TRIGGER after_members_insert
AFTER INSERT
ON members
FOR EACH ROW
BEGIN

  IF NEW.birthDate IS NULL THEN
    INSERT INTO reminders(memberId, message)
    VALUES(
      NEW.id,
      CONCAT('Hi ', NEW.name, ', please update your date of birth.')
    );
  END IF;

END$$

DELIMITER ;
```

```sql
DELIMITER $$

CREATE TRIGGER before_sales_update
BEFORE UPDATE
ON sales
FOR EACH ROW
BEGIN

  DECLARE errorMessage VARCHAR(255);
  SET errorMessage = CONCAT
  (
    'The new quantity ',
    NEW.quantity,
    ' cannot be 3 times greater than the current quantity ',
    OLD.quantity
  );

  IF NEW.quantity > OLD.quantity * 3 THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = errorMessage;
  END IF;

END $$

DELIMITER ;
```

## Removal

```sql
DROP TRIGGER trigger_name;
```

## List All Triggers

```sql
SHOW TRIGGERS;
```
