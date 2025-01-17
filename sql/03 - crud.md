---
title: CRUD Operations
---

## Insertion

### New data

```sql
INSERT INTO table_name (col_1, col_n)
VALUES
(1, "A"),
(2, "B"),
(3, "C");
```

### From another query

```sql
INSERT INTO table_name (col_1, col_n)
SELECT col_1, col_n
FROM another_table;
```

---

## Deletion

```sql
DELETE FROM table_name
WHERE id IN (1,2,3);
```

---

## Modification

```sql
UPDATE cars_table
SET model = "BMW", price = 100
WHERE id = 5;
```

---

## Reading

```sql
SELECT *
FROM table_name;
```
