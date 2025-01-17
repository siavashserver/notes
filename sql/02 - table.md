---
title: Table Management
---

## List All Tables

```sql
SHOW TABLES
```

## Create Table

```sql
CREATE TABLE table_name
(
  col_name data_type constraints
);
```

## Delete Table

```sql
DROP TABLE table_1, table_2, table_n;
```

## Truncate Table

```sql
TRUNCATE TABLE table_1, table_2, table_n;
```

## Rename Table

```sql
ALTER TABLE table_name
RENAME TO new_table_name;
```

---

## Column Addition

```sql
ALTER TABLE table_name
ADD COLUMN column_1 VARCHAR(50),
ADD COLUMN column_n VARCHAR(50);
```

## Column Removal

```sql
ALTER TABLE table_name
DROP COLUMN column_1,
DROP COLUMN column_n;
```

## Column Modification

```sql
ALTER TABLE table_name
MODIFY column_1 VARCHAR(255);
```

## Rename Column

```sql
ALTER TABLE table_name
RENAME column_name TO new_column_name;
```

---

## Default Value

### During table definition

```sql
CREATE TABLE table_name
(
  column_name INT DEFAULT 0
);
```

### After table definition

```sql
ALTER TABLE table_name
ALTER COLUMN column_name SET DEFAULT 1;
```

### Removal

```sql
ALTER TABLE table_name
ALTER COLUMN column_name DROP DEFAULT;
```

## Primary Key

### During table definition

```sql
CREATE TABLE users
(
  id INT NOT NULL PRIMARY KEY AUTO_INCREMENT
);
```

```sql
CREATE TABLE users
(
  id INT NOT NULL AUTO_INCREMENT,
  CONSTRAINT pk_id PRIMARY KEY (id)
);
```

### After table definition

```sql
ALTER TABLE table_name
ADD CONSTRAINT pk_id PRIMARY KEY (column_name)
```

### Removal

```sql
ALTER TABLE table_name
DROP PRIMARY KEY
```

```sql
ALTER TABLE table_name
DROP CONSTRAINT pk_id;
```

---

## Foreign Key

### During table definition

```sql
CREATE TABLE users
(
  city_id INT FOREIGN KEY REFERENCES city_table(id_column)
);
```

```sql
CREATE TABLE users
(
  city_id INT,
  CONSTRAINT fk_name FOREIGN KEY (city_id) REFERENCES city_table(id_column)
);
```

### After table definition

```sql
ALTER TABLE table_name
ADD CONSTRAINT fk_name FOREIGN KEY (city_id) REFERENCES city_table(id_column);
```

### Removal

```sql
ALTER TABLE table_name
DROP CONSTRAINT fk_name
```

---

## Unique Key

### During table definition

```sql
CREATE TABLE users
(
  username VARCHAR(255) NOT NULL UNIQUE
);
```

```sql
CREATE TABLE users
(
  username VARCHAR(255) NOT NULL,
  CONSTRAINT uk_username UNIQUE (username)
);
```

### After table definition

```sql
ALTER TABLE table_name
ADD CONSTRAINT uk_username UNIQUE (username);
```

### Removal

```sql
ALTER TABLE table_name
DROP CONSTRAINT uk_username
```

---

## Check

### During table definition

```sql
CREATE TABLE car
(
  price NUMERIC(10,2) CHECK (price > 0)
);
```

```sql
CREATE TABLE car
(
  price NUMERIC(10,2),
  CONSTRAINT check_price CHECK (price > 0)
);
```

### After table definition

```sql
ALTER TABLE table_name
ADD CONSTRAINT check_price CHECK (price > 0);
```

### Removal

```sql
ALTER TABLE table_name
DROP CONSTRAINT check_price
```

---

## Indexing

### Creation

```sql
CREATE INDEX index_name
ON table_name(column_1, column_n)
```

```sql
CREATE UNIQUE INDEX index_name
ON table_name(column_1, column_n)
```

### Deletion

```sql
ALTER TABLE table_name
DROP INDEX index_name;
```

### List All Indices

```sql
SHOW INDEX
FROM table_name
FROM database_name;
```
