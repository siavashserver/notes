---
title: Indexing
---

## Data Storage

### Pages

Date (rows, index maps, ...) are stored in _8KB_ sized files, called _Pages_.

Pages are consisted of 3 parts:

- Header: Includes information such as _Page Number_, _Owner Object (table,...)_ and _Page Type_.
- Records: Rows are added sequentially.
- Offset array: Pointers to the starting locations of the rows on the page.

Main page types:

- Data pages
- Index pages
- Large object pages (varchar, text, image, ...)

### Extents

An extent is a _group of 8 physically contiguous pages_ in a data file.

There are 2 types of extents:

- Mixed extents: Contains pages that are allocated for multiple objects. (Small tables containing less than 8 pages, grouped together)
- Uniform extents: All pages are allocated for a single object. (Default behavior since SQL Server 2016, to reduce page allocation contention)

## Index Types

### Heap Table

No indexes set, and data gets stored without any specific order.
Usage: Temporarily tables, ...

### Clustered Index

Data is sorted and stored based on the chosen index key. Index is stored
alongside the table data. Only one clustered index can be set on a table.

### Non-Clustered Index

Secondary indexes created on top of existing _Clustered Indexes_ or _Heap_.
Columns are selected and sorted based on their values, and hold a reference to
the _Heap_ or _Clustered Index_ location of the data that they reference.
Multiple non-clustered indexes can be created for a table.

### Full-Text, etc

Enabling full-text search and other database specific index types.

## How indexes work

Through B-Tree data structures.

## Best Practices

- Use Clustered Indexes on Primary Keys by Default.
  - Should be created on static data
  - Should be created on narrow data types, because the clustered index key for every row is
    included in all non-clustered indexes associated with the table.
  - Should be an increasing value, to reduce B-Tree fragmentation.
  - Avoid *GUID*; Causes more problems than it solves.
- Index Foreign Key Columns

## Index Creation

```sql
CREATE [UNIQUE] [CLUSTERED | NONCLUSTERED | FULLTEXT] INDEX index_name ON table_name
```

## Index Deletion

```sql
ALTER TABLE table_name
DROP INDEX index_name
```
