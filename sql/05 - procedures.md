---
title: Stored Procedures
---

## Creation

### Variables

```sql
-- Temporarily change delimiter
DELIMITER $$

CREATE PROCEDURE GetTotalOrder()
BEGIN
  DECLARE total_order INT DEFAULT 0;

  SELECT COUNT(*)
  INTO total_order
  FROM orders;

  SELECT total_order;
END$$

-- Reset delimiter
DELIMITER ;
```

```sql
CALL GetTotalOrder();

-- total_order
-- 270
```

### IN / OUT

```sql
-- Temporarily change delimiter
DELIMITER $$

CREATE PROCEDURE GetOrderCountByStatus
(
  IN  order_status VARCHAR(25),
  OUT total INT
)
BEGIN
  SELECT COUNT(order_number)
  INTO total
  FROM orders
  WHERE status = order_status;
END$$

-- Reset delimiter
DELIMITER ;
```

```sql
SET @total = 0;
CALL GetOrderCountByStatus('SHIPPED',@total);
SELECT @total;

-- @total
-- 95
```

### INOUT

```sql
-- Temporarily change delimiter
DELIMITER $$

CREATE PROCEDURE IncrementCounter(
  INOUT counter INT,
  IN    inc INT
)
BEGIN
  SET counter = counter + inc;
END$$

-- Reset delimiter
DELIMITER ;
```

```sql
SET @counter = 1;
CALL IncrementCounter(@counter,1); -- 2
CALL IncrementCounter(@counter,1); -- 3
CALL IncrementCounter(@counter,5); -- 8
SELECT @counter;

-- @counter
-- 8
```

## Deletion

```sql
DROP PROCEDURE ProcedureName;
```

## List All Procedures

```sql
SHOW PROCEDURE STATUS;
```

---

## IF / THEN / ELSEIF / ELSE / END IF

```sql
-- Temporarily change delimiter
DELIMITER $$

CREATE PROCEDURE GetCustomerLevel
(
    IN  pCustomerNumber INT,
    OUT pCustomerLevel  VARCHAR(20)
)
BEGIN
    DECLARE credit DECIMAL DEFAULT 0;

    SELECT creditLimit
    INTO credit
    FROM customers
    WHERE customerNumber = pCustomerNumber;

    IF credit > 50000 THEN
        SET pCustomerLevel = 'PLATINUM';
    ELSEIF credit <= 50000 AND credit > 10000 THEN
        SET pCustomerLevel = 'GOLD';
    ELSE
        SET pCustomerLevel = 'SILVER';
    END IF;
END $$

-- Reset delimiter
DELIMITER ;
```

```sql
CALL GetCustomerLevel(447, @level);
SELECT @level;

-- @level
-- SILVER
```

## CASE / WHEN / THEN / END CASE

```sql
-- Temporarily change delimiter
DELIMITER $$

CREATE PROCEDURE GetDeliveryStatus
(
  IN  pOrderNumber INT,
  OUT pDeliveryStatus VARCHAR(100)
)
BEGIN
  DECLARE waitingDay INT DEFAULT 0;

  SELECT DATEDIFF(requiredDate, shippedDate)
  INTO waitingDay
  FROM orders
  WHERE orderNumber = pOrderNumber;

  CASE
  WHEN waitingDay = 0 THEN
    SET pDeliveryStatus = 'On Time';
  WHEN waitingDay >= 1 AND waitingDay < 5 THEN
    SET pDeliveryStatus = 'Late';
  WHEN waitingDay >= 5 THEN
    SET pDeliveryStatus = 'Very Late';
  ELSE
    SET pDeliveryStatus = 'No Information';
  END CASE;
END$$

-- Reset delimiter
DELIMITER ;
```

```sql
CALL GetDeliveryStatus(10100,@delivery);
SELECT @delivery;

-- @delivery
-- Late
```

---

## LOOP / END LOOP

```sql
-- Temporarily change delimiter
DELIMITER $$

CREATE PROCEDURE LoopDemo()
BEGIN
  DECLARE i   INT;
  DECLARE str VARCHAR(255);

  SET i = 1;
  SET str = '';

  loop_label: LOOP
    IF i > 10 THEN
      -- break loop
      LEAVE loop_label;
    END IF

    SET i = i + 1;

    IF (i mod 2) THEN
      -- continue loop
      ITERATE loop_label;
    ELSE
      SET str = CONCAT(str,i,',');
    END IF;
  END LOOP;

  SELECT str;
END$$

-- Reset delimiter
DELIMITER ;
```

```sql
CALL LoopDemo();

-- str
-- 2,4,6,8,10,
```

---

## WHILE DO / END WHILE

```sql
-- Temporarily change delimiter
DELIMITER $$

CREATE PROCEDURE LoadCalendars(
  startDate DATE,
  day INT
)
BEGIN

  DECLARE counter INT DEFAULT 1;
  DECLARE dt DATE DEFAULT startDate;

  WHILE counter <= day DO
    -- call existing Stored Procedure to insert date
    CALL InsertCalendar(dt);
    SET counter = counter + 1;
    SET dt = DATE_ADD(dt,INTERVAL 1 day);
  END WHILE;

END$$

-- Reset delimiter
DELIMITER ;
```

---

## REPEAT UNTIL / END REPEAT

```sql
-- Temporarily change delimiter
DELIMITER $$

CREATE PROCEDURE RepeatDemo()
BEGIN
  DECLARE counter INT DEFAULT 1;
  DECLARE result VARCHAR(100) DEFAULT '';

  REPEAT
    SET result = CONCAT(result,counter,',');
    SET counter = counter + 1;
  UNTIL counter >= 10
  END REPEAT;

  -- display result
  SELECT result;
END$$

-- Reset delimiter
DELIMITER ;
```
