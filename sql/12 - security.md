---
title: Security
---

See [MySQL Security Manual](https://dev.mysql.com/doc/refman/8.0/en/security.html) for more details.

## User Management

### Creating Users

```sql
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
```

### Deleting Users

```sql
DROP USER 'username'@'localhost';
```

---

## Roles

```sql
CREATE ROLE 'app_developer', 'app_read', 'app_write';
```

```sql
GRANT ALL ON app_db.* TO 'app_developer';
GRANT SELECT ON app_db.* TO 'app_read';
GRANT INSERT, UPDATE, DELETE ON app_db.* TO 'app_write';
```

```sql
CREATE USER 'dev1'@'localhost' IDENTIFIED BY 'dev1pass';
CREATE USER 'read_user1'@'localhost' IDENTIFIED BY 'read_user1pass';
CREATE USER 'read_user2'@'localhost' IDENTIFIED BY 'read_user2pass';
CREATE USER 'rw_user1'@'localhost' IDENTIFIED BY 'rw_user1pass';
```

```sql
GRANT 'app_developer' TO 'dev1'@'localhost';
GRANT 'app_read' TO 'read_user1'@'localhost', 'read_user2'@'localhost';
GRANT 'app_read', 'app_write' TO 'rw_user1'@'localhost';
```

```sql
DROP ROLE 'app_read', 'app_write';
```

---

## Setting Stored Procedure Access Levels

```sql
-- Temporarily change delimiter
DELIMITER $$

CREATE DEFINER=root@localhost PROCEDURE InsertMessage
(
  msg VARCHAR(100)
)
-- DEFINER Gains same access rights as DEFINER user
SQL SECURITY DEFINER
BEGIN
  INSERT INTO messages(message)
  VALUES(msg);
END$$

-- Reset delimiter
DELIMITER ;
```

```sql
-- Temporarily change delimiter
DELIMITER $$

CREATE DEFINER=root@localhost PROCEDURE UpdateMessage
(
    msgId INT,
    msg VARCHAR(100)
)
-- INVOKER Get limited to the calling user access rights
SQL SECURITY INVOKER
BEGIN
  UPDATE messages
  SET message = msg
  WHERE id = msgId;
END$$

-- Reset delimiter
DELIMITER ;
```

```sql
-- Grant stored procedure execution rights to user
GRANT EXECUTE ON testdb.* TO dev@localhost;
```

```sql
-- Succedes
CALL InsertMessage('Hello World');
```

```sql
-- Fails
-- dev@localhost does not have any privileges on the messages table
CALL UpdateMessage(1,'Good Bye');
```
