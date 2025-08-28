---
title: LINQ
---

## LINQ Overview and Operator Categories

LINQ's core strength lies in its **Standard Query Operators**, which are
essentially extension methods (mainly for `IEnumerable<T>` and `IQueryable<T>`)
grouped into several logical categories. Each facilitates a class of data
operations, from filtering and ordering to aggregation and transformation.

### Standard Query Operator Categories

| Category      | Operators (Examples)                                  | SQL Equivalent Concepts          |
| ------------- | ----------------------------------------------------- | -------------------------------- |
| Filtering     | `Where`, `OfType`                                     | `WHERE`, type filter             |
| Projection    | `Select`, `SelectMany`                                | `SELECT`, flatten/join           |
| Sorting       | `OrderBy`, `OrderByDescending`, `ThenBy`, `Reverse`   | `ORDER BY`                       |
| Element       | `First`, `FirstOrDefault`, `Single`, `Last`           | `SELECT TOP 1`, `LIMIT`, `[n]`   |
| Quantifier    | `Any`, `All`, `Contains`                              | `EXISTS`, `ALL`, `IN`            |
| Aggregation   | `Count`, `Sum`, `Min`, `Max`, `Average`, `Aggregate`  | `COUNT()`, `SUM()`, etc.         |
| Grouping      | `GroupBy`, `ToLookup`                                 | `GROUP BY`                       |
| Set           | `Distinct`, `Union`, `Intersect`, `Except`            | `DISTINCT`, `UNION`, `INTERSECT` |
| Joining       | `Join`, `GroupJoin`, `Zip`                            | `INNER JOIN`, `LEFT OUTER JOIN`  |
| Partitioning  | `Skip`, `SkipWhile`, `Take`, `TakeWhile`              | `OFFSET`, `LIMIT`                |
| Generation    | `Range`, `Repeat`, `Empty`                            | Generate series (not all in SQL) |
| Conversion    | `ToList`, `ToArray`, `ToDictionary`, `Cast`, `OfType` | Data table conversions, `CAST`   |
| Concatenation | `Concat`                                              | `UNION ALL`                      |
| Equality      | `SequenceEqual`                                       | Sequence/value comparison        |

**Note:** Some operators in LINQ have no direct SQL equivalent and some SQL
constructs (like sub-queries) map to more than one LINQ operation.

---

## Query Syntax vs Method Syntax: A Practical Comparison

C# supports **query syntax** (similar to SQL, e.g. `from ... where ... select
...`) and **method syntax** (chaining LINQ extension/lambda methods, e.g.
`.Where(...).Select(...)`).

**Query Syntax Example:**

```csharp
var result = from p in products
             where p.Price > 100
             orderby p.Name descending
             select p.Name;
```

**Method Syntax Equivalent:**

```csharp
var result = products
    .Where(p => p.Price > 100)
    .OrderByDescending(p => p.Name)
    .Select(p => p.Name);
```

- **Query syntax** is more readable for those familiar with SQL and often
  preferred for complex queries with joins and groupings.
- **Method syntax** is more flexible and supports all LINQ operators, including
  those not represented by query keywords (such as `Zip`, `Aggregate`).

Most queries are interchangeable between these forms, but highly complex
operators or nuanced data shapes might demand method syntax.

---

## LINQ Methods

### 1. **Element Operators**

Single out individual elements from a collection; throw exceptions or return
defaults based on conditions.

| Operator                           | Description                                           | Sample Usage (Method Syntax)         | SQL Equivalent                      |
| ---------------------------------- | ----------------------------------------------------- | ------------------------------------ | ----------------------------------- |
| `First`                            | Returns first element; error if none                  | `col.First(x => x.Id == 3)`          | `SELECT TOP 1 ... WHERE ...`        |
| `FirstOrDefault`                   | First element or default (null/0) if none             | `col.FirstOrDefault(x => x.Id == 3)` | `SELECT TOP 1 ... WHERE ...`        |
| `Last` / `LastOrDefault`           | Last element or default (list must support indexing)  | `col.Last()`                         | no direct SQL, needs ordering       |
| `Single` / `SingleOrDefault`       | The only element, error/null if not exactly one match | `col.Single(x => x.Name == "John")`  | `SELECT ... WHERE ...` + constraint |
| `ElementAt` / `ElementAtOrDefault` | Returns item at index or default if out of range      | `col.ElementAt(3)`                   | -                                   |

**Sample Code:**

```csharp
var firstExpensive = products.First(p => p.Price > 1000);

// Query syntax alternative:
var firstExpensiveQS = (from p in products
                        where p.Price > 1000
                        select p).First();
```

**SQL Equivalent:**

```sql
SELECT TOP 1 * FROM Products WHERE Price > 1000
```

**Practical Notes:**

- Use `First`/`FirstOrDefault` to retrieve “any” matching data (not guaranteed
  order unless sorted).
- `Single` and `SingleOrDefault` are stricter, triggering errors when multiple
  entries match—ideal for enforcing uniqueness.
- Performance: `First` and friends can stop iterating as soon as a result is
  found, while `Single` must scan all items to ensure only one exists.

---

### 2. **Filtering Operators**

Reduce a dataset to those meeting a predicate or of a specific type.

| Operator      | Description                     | Method Syntax Example           | SQL Equivalent       |
| ------------- | ------------------------------- | ------------------------------- | -------------------- |
| `Where`       | Filters collection by predicate | `col.Where(x => x.Price < 100)` | `WHERE` clause       |
| `OfType<T>()` | Picks only elements of Type T   | `col.OfType<Employee>()`        | No direct equivalent |

**Sample Code:**

```csharp
var cheapItems = items.Where(i => i.Price < 100);
var employees = objects.OfType<Employee>();
```

**Query Syntax:**

```csharp
var cheapItems = from i in items
                 where i.Price < 100
                 select i;
```

**SQL Equivalent:**

```sql
SELECT * FROM Items WHERE Price < 100
```

**Notes:**

- Multiple `Where` clauses can be chained or combined with `&&` inside a single
  lambda; providers usually optimize to a single database query.
- `OfType<T>()` is valuable for filtering from non-generic collections, e.g.,
  those containing mixed types.

---

### 3. **Projection Operators**

Transform each element into a new form, flatten nested collections, or select
specific members.

| Operator     | Description                      | Method Syntax Example           | SQL Equivalent             |
| ------------ | -------------------------------- | ------------------------------- | -------------------------- |
| `Select`     | Projects each item to a new form | `col.Select(x => x.Name)`       | `SELECT column`            |
| `SelectMany` | Flattens nested collections      | `col.SelectMany(x => x.Orders)` | flatten results/cross join |

**Code and SQL:**

```csharp
// Basic projection—get product names
var names = products.Select(p => p.Name);

// Flatten: all orders for all customers
var allOrders = customers.SelectMany(c => c.Orders);
```

**Equivalent SQL:**

```sql
SELECT Name FROM Products;
SELECT * FROM Orders; -- After joining with Customers
```

**Query Syntax Support:**

- `SelectMany` is represented in query syntax using multiple `from` clauses:

```csharp
var ordersFlat = from c in customers
                 from o in c.Orders
                 select o;
```

- Useful for one-to-many relationships (e.g., customer-orders).

---

### 4. **Sorting Operators**

Determine the ordering of the results.

| Operator            | Description                | C# Usage (Method)                 | Query Syntax              | SQL Equivalent      |
| ------------------- | -------------------------- | --------------------------------- | ------------------------- | ------------------- |
| `OrderBy`           | Ascending order by key     | `.OrderBy(x => x.Name)`           | `orderby x.Name`          | `ORDER BY Name ASC` |
| `OrderByDescending` | Descending order           | `.OrderByDescending(x => x.Id)`   | `orderby x.Id descending` | `ORDER BY ... DESC` |
| `ThenBy`            | Secondary ascending order  | `.ThenBy(x => x.Age)`             | `orderby x.Name, x.Age`   | Multi-col ORDER BY  |
| `ThenByDescending`  | Secondary descending order | `.ThenByDescending(x => x.Score)` | ...                       |                     |
| `Reverse`           | Reverses current order     | `.Reverse()`                      | -                         | No direct           |

**Sample:**

```csharp
var sorted = people.OrderBy(p => p.LastName).ThenBy(p => p.Age);
// Or in query syntax:
var sorted = from p in people
             orderby p.LastName, p.Age
             select p;
```

**SQL:**

```sql
SELECT * FROM People ORDER BY LastName ASC, Age ASC
```

**Note:** Using `Reverse` is not the same as `OrderByDescending`; `Reverse`
literally reverses the enumeration order in memory.

---

### 5. **Quantifier Methods**

Check if any/all/contains conditions are met—return `bool`.

| Operator   | Purpose                                          | Example                                      | SQL Equivalent        |
| ---------- | ------------------------------------------------ | -------------------------------------------- | --------------------- |
| `Any`      | Checks if any elements exist, or match predicate | `col.Any()`, `col.Any(x => x.Status == "A")` | `EXISTS`, `WHERE ...` |
| `All`      | Checks if all items match predicate              | `col.All(x => x.IsValid)`                    | `ALL()`               |
| `Contains` | Checks if a value exists in the collection       | `col.Contains(5)`                            | `IN (...)`            |

**Code:**

```csharp
bool exists = products.Any(p => p.Stock == 0);
// Equivalent SQL: SELECT CASE WHEN EXISTS (SELECT * FROM Products WHERE Stock = 0) THEN 1 ELSE 0 END
```

**`Contains` for 'IN' semantics:**

```csharp
var validIds = new[] {2, 4, 8};
var selected = people.Where(p => validIds.Contains(p.Id));
```

**SQL:**

```sql
SELECT * FROM People WHERE Id IN (2, 4, 8)
```

**Best Practice:** Use `Any()` over `Count() > 0` for performance, as `Any` can
stop at the first match.

---

### 6. **Aggregation Methods**

Compute summary statistics and custom aggregations.

| Operator    | Usage                                         | Example                                  | SQL Equivalent                |
| ----------- | --------------------------------------------- | ---------------------------------------- | ----------------------------- |
| `Count`     | Number of items (optionally with a predicate) | `orders.Count(o => o.Total > 100)`       | `COUNT(*)` or `COUNT(IF...)`  |
| `Sum`       | Total of values                               | `orders.Sum(o => o.Total)`               | `SUM(Total)`                  |
| `Min`/`Max` | Minimum, maximum value                        | `products.Min(p => p.Price)`             | `MIN`, `MAX`                  |
| `Average`   | Mean                                          | `scores.Average(s => s.Points)`          | `AVG(...)`                    |
| `Aggregate` | Custom aggregation                            | `strs.Aggregate((x, y) => x + ", " + y)` | No direct equivalent (custom) |

**Sample:**

```csharp
var totalSales = orders.Sum(o => o.Amount);
var countActive = users.Count(u => u.IsActive);
```

**SQL:**

```sql
SELECT SUM(Amount) FROM Orders;
SELECT COUNT(*) FROM Users WHERE IsActive = 1;
```

**Note:**

- `Aggregate` is powerful for operations such as computing products,
  concatenations, or custom accumulations—a functional equivalent to custom SQL
  UDFs.

---

### 7. **Grouping Methods**

Organize data by key, returning groups for further aggregation or processing.

| Operator   | Description                            | Example                     | SQL Equivalent        |
| ---------- | -------------------------------------- | --------------------------- | --------------------- |
| `GroupBy`  | Groups items by key selector           | `.GroupBy(x => x.Category)` | `GROUP BY`            |
| `ToLookup` | Groups like `GroupBy`, returns ILookup | `.ToLookup(x => x.Type)`    | (Not directly in SQL) |

**C# Example:**

```csharp
var groups = orders.GroupBy(o => o.CustomerId);
foreach (var group in groups)
{
    Console.WriteLine($"Customer: {group.Key}");
    foreach (var order in group)
        Console.WriteLine(order.Id);
}
```

**SQL:**

```sql
SELECT CustomerId, COUNT(*) FROM Orders GROUP BY CustomerId;
```

**Note:** In LINQ, each group is an `IGrouping<TKey, TElement>`, which can be
further analyzed or projected (e.g., group aggregates).

---

### 8. **Set Operators**

Deal with distinctness and comparison across sets.

| Operator    | Description                       | C# Example             | SQL Equivalent    |
| ----------- | --------------------------------- | ---------------------- | ----------------- |
| `Distinct`  | Removes duplicates                | `col.Distinct()`       | `SELECT DISTINCT` |
| `Union`     | Combines sets, removes duplicates | `col1.Union(col2)`     | `UNION`           |
| `Intersect` | Shared elements only              | `col1.Intersect(col2)` | `INTERSECT`       |
| `Except`    | Elements in first, not second     | `col1.Except(col2)`    | `EXCEPT`          |
| `Concat`    | Appends sets, keeps duplicates    | `col1.Concat(col2)`    | `UNION ALL`       |

**Sample:**

```csharp
var allNames = list1.Union(list2); // unique names
var onlyList1 = list1.Except(list2); // exclusive to list1
```

**SQL:**

```sql
SELECT Name FROM Table1
UNION
SELECT Name FROM Table2

SELECT Name FROM Table1
EXCEPT
SELECT Name FROM Table2
```

**Note:** `Distinct` and friends use `IEqualityComparer` for custom object
equality.

---

### 9. **Joining Operators**

Combine multiple collections based on matching keys—mirror SQL joins.

| Operator    | Description                 | C# Example                                          | SQL Equivalent             |
| ----------- | --------------------------- | --------------------------------------------------- | -------------------------- |
| `Join`      | Inner join, matches on keys | `.Join(other, a => a.Id, b => b.Id, (a, b) => ...)` | `INNER JOIN`               |
| `GroupJoin` | Join, with grouped results  | `.GroupJoin(b, a=>a.Id, b=>b.Id, (a, grp) => ...)`  | `LEFT OUTER JOIN` grouping |
| `Zip`       | Pair by position            | `.Zip(col2, (a,b) => ...)`                          | (Not directly in SQL)      |

**Examples:**

```csharp
// Inner join
var joined = customers.Join(orders, c => c.Id, o => o.CustomerId,
                          (c, o) => new { c.Name, o.OrderDate });

// Group join (LEFT OUTER JOIN)
var groupJoin = departments.GroupJoin(
    employees,
    d => d.Id,
    e => e.DepartmentId,
    (dept, emps) => new { dept.Name, Employees = emps });
```

**Query syntax for Join:**

```csharp
var joined = from c in customers
             join o in orders on c.Id equals o.CustomerId
             select new { c.Name, o.OrderDate };
```

**GroupJoin using 'into':**

```csharp
var deptEmps = from d in departments
               join e in employees on d.Id equals e.DepartmentId into empGroup
               select new { d.Name, Employees = empGroup };
```

**SQL Example:**

```sql
SELECT c.Name, o.OrderDate
FROM Customers c
INNER JOIN Orders o ON c.Id = o.CustomerId;
```

---

### 10. **Partitioning Operators**

Limit and skip over items for paging or isolation.

| Operator    | Description                     | C# Example                          | SQL Equivalent            |
| ----------- | ------------------------------- | ----------------------------------- | ------------------------- |
| `Skip`      | Skips N elements                | `list.Skip(5)`                      | `OFFSET`                  |
| `Take`      | Takes first N elements          | `list.Take(5)`                      | `LIMIT`                   |
| `SkipWhile` | Skips as long as predicate true | `list.SkipWhile(x => x.Score < 10)` | (Not direct; complex SQL) |
| `TakeWhile` | Takes as long as predicate true | `list.TakeWhile(x => x.Age < 18)`   | (Not direct; complex SQL) |

---

### 11. **Generation Methods**

Create sequences programmatically—valuable in test data, projections, or
algorithms.

| Operator     | Purpose                                | C# Example                  | SQL Equivalent               |
| ------------ | -------------------------------------- | --------------------------- | ---------------------------- |
| `Range`      | Sequence of ints from a starting value | `Enumerable.Range(1, 10)`   | Generate_series (PostgreSQL) |
| `Repeat`     | Repeats an element N times             | `Enumerable.Repeat("X", 5)` | (Rare in SQL)                |
| `Empty<T>()` | Empty sequence of given type           | `Enumerable.Empty<int>()`   | -                            |

**Code:**

```csharp
var nums = Enumerable.Range(5, 3); // 5, 6, 7
var repeated = Enumerable.Repeat("hi", 3); // "hi", "hi", "hi"
```

**SQL Equivalent:** Some SQL dialects support generate_series; in most RDBMSs,
synthetic sequences require helper tables or recursion.

---

### 12. **Conversion Methods**

Transform sequence to a new data structure or interface.

| Operator         | Effect                             | C# Sample                     | SQL Equivalent         |
| ---------------- | ---------------------------------- | ----------------------------- | ---------------------- |
| `ToList()`       | Materializes as `List<T>`          | `query.ToList()`              | -                      |
| `ToArray()`      | Converts to native array           | `query.ToArray()`             | -                      |
| `ToDictionary()` | Creates dictionary by key selector | `col.ToDictionary(x => x.Id)` | key-value indexed view |
| `Cast<T>()`      | Casts elements to `T`              | `items.Cast<Employee>()`      | CAST                   |
| `OfType<T>()`    | Picks only those of type `T`       | `items.OfType<string>()`      | (see above)            |
| `AsEnumerable()` | Changes type to `IEnumerable<T>`   | `db.Set<T>().AsEnumerable()`  | -                      |
| `AsQueryable()`  | Adds queryable capability          | `list.AsQueryable()`          | -                      |

**Performance Note:** All conversions that end with immediate structures
(`ToList`, `ToArray`, `ToDictionary`) cause immediate query execution, consuming
the results in-memory and enabling multiple enumerations thereafter (but
potentially loading large result sets all at once).

---

## SQL Equivalents for LINQ Methods

While LINQ is not always a 1:1 map with SQL, the table below summarizes common
mappings:

| LINQ                       | SQL                       |
| -------------------------- | ------------------------- |
| `Where(p => ...)`          | `WHERE ...`               |
| `Select(p => p.Name)`      | `SELECT Name`             |
| `OrderBy(p => p.Col)`      | `ORDER BY Col ASC`        |
| `GroupBy(p => p.Cat)`      | `GROUP BY Cat`            |
| `Join(...)`                | `INNER JOIN ... ON ...`   |
| `GroupJoin(...)` + flatten | `LEFT OUTER JOIN`         |
| `Distinct()`               | `SELECT DISTINCT`         |
| `Count()`                  | `COUNT(*)`                |
| `Sum(p => p.Price)`        | `SUM(Price)`              |
| `Any()`                    | `EXISTS(SELECT 1 ...)`    |
| `Contains(x)`              | `IN (...)`                |
| `Skip/Take`                | `OFFSET`/`LIMIT` or `TOP` |

---

## Key Advanced LINQ Topics

### **Parallel LINQ (PLINQ)**

PLINQ enables parallel processing of LINQ queries for CPU-bound tasks, often
yielding substantial performance boost on large or computation-heavy datasets.
Example:

```csharp
var evenNumbers = numbers.AsParallel().Where(n => n % 2 == 0).ToList();
```

- Add `.AsParallel()` to your collection.
- Use `.AsOrdered()` if ordering is critical (adds cost).
- Tune with `.WithDegreeOfParallelism(n)` and `.WithCancellation(token)` for
  more control.
- **When to use:** Big data, CPU-bound, many-core machines.
- **When _not_ to use:** Small datasets, I/O-bound work, order-critical apps.

### **LINQ Expression Trees**

Expression trees represent code in a tree-like data structure versus
delegates/compiled code. They are central to query providers (such as Entity
Framework) for translating LINQ queries to other languages like SQL.

Example:

```csharp
Expression<Func<User, bool>> expr = u => u.Age > 18;
```

Used in `IQueryable<T>` implementations.

---

### **LINQ Performance Best Practices**

- **Filter Early:** Place `Where` and similar filters at the beginning of a
  query chain to minimize data processed by subsequent steps.
- **Avoid Repeated Enumeration:** Materialize results (e.g., with `ToList()`) if
  you need to iterate them more than once.
- **Use `Any()` not `Count() > 0`:** `Any()` short-circuits on the first match;
  `Count()` always iterates through the entire collection.
- **Defer Execution:** Take advantage of deferred execution; chain transforms
  before materializing with `ToList()`, `ToArray()` etc.
- **Understand `IEnumerable` vs `IQueryable`:** `IQueryable` allows query
  translation and server-side execution; `IEnumerable` works in memory—prefer
  `IQueryable` for database sources.

---

## LINQ for SQL Developers

### Mapping Typical SQL to LINQ

- **`WHERE` clause:** `Where()` method.
- **`IN` clause:** `x => list.Contains(x.Id)`.
- **`JOIN` clause:** `Join()` or `join ... on ... equals ...` query syntax.
- **Grouping and Aggregates:** `GroupBy()` plus `Sum()`/`Count()`/etc.
- **Paging:** `Skip()` and `Take()` together; equivalent to SQL OFFSET/LIMIT.

**Example: Complex Query**

```sql
SELECT Department, COUNT(*) AS EmpCount, AVG(Salary) AS AvgSal
FROM Employees
WHERE IsActive = 1
GROUP BY Department
ORDER BY AvgSal DESC
```

**In LINQ:**

```csharp
var query = employees
    .Where(e => e.IsActive)
    .GroupBy(e => e.Department)
    .Select(g => new {
        Department = g.Key,
        EmpCount = g.Count(),
        AvgSal = g.Average(e => e.Salary)
    })
    .OrderByDescending(x => x.AvgSal);
```

Or, in query syntax:

```csharp
var query =
    from e in employees
    where e.IsActive
    group e by e.Department into g
    orderby g.Average(e => e.Salary) descending
    select new {
        Department = g.Key,
        EmpCount = g.Count(),
        AvgSal = g.Average(e => e.Salary)
    };
```

---

## Interview Questions

### What is LINQ and why was it introduced in .NET?

LINQ (Language Integrated Query) brings a uniform, type-safe, and expressive way
to query and manipulate data from different sources (objects, SQL, XML, etc.) in
C#. It integrates querying capabilities directly into the language, bridging the
gap between programming objects and the world of data.

### How can you check if any item in a collection meets a particular condition, and why is `Any()` preferred over `Count() > 0`?

```csharp
if (orders.Any(o => o.Status == "Pending")) { ... }
```

`Any()` stops at the first match (short-circuits), while `Count()` traverses the
whole collection, affecting performance.

### What is deferred execution in LINQ and why is it important?

Deferred execution means the query isn’t executed until its results are
enumerated (iteration, ToList, etc.). This conserves resources and allows query
parts to be composed dynamically.

### When should you use `GroupJoin`, and how does it differ from `Join`?

`GroupJoin` produces a grouped result for one-to-many relationships, akin to
SQL’s GROUP BY plus LEFT JOIN, whereas `Join` flattens to pairs (like INNER
JOIN).

### Compare `IQueryable<T>` and `IEnumerable<T>`, and describe when to use each

- `IEnumerable<T>`: Queries are executed in memory, suitable for in-memory data.
- `IQueryable<T>`: Queries can be translated (e.g., to SQL) and executed by the
  underlying provider (EF, LINQ to SQL). Use `IQueryable` for database queries;
  use `IEnumerable` for collections already in memory.

### Can you show how to use a compiled LINQ query and explain its benefits?

Compiled queries pre-parse LINQ expressions for performance in repetitive
database operations. Example using Entity Framework:

```csharp
static readonly Func<DbContext, int, User> getUserById =
    EF.CompileQuery((DbContext ctx, int id) => ctx.Users.Single(u => u.Id == id));

// Usage
var user = getUserById(context, 42);
```

Benefit: Reduced overhead for repeated queries with variable parameters.

### What are Expression Trees, and why are they important in LINQ?

Expression Trees (`Expression<Func<T, bool>>`) allow LINQ providers (like EF) to
analyze, optimize, and translate queries into other languages (such as SQL),
because the code is represented structurally, not as compiled delegates.

---

## `SelectMany`

### Usage Scenarios

**Typical use cases for SelectMany:**

- **Flattening Nested Collections:** Combine sub-lists (like order items per
  order, skills per employee, or programming languages per student) into a flat
  list.
- **Generating Cartesian or Cross Products:** For scenarios needing every
  possible pairing between two sets.
- **Accessing Deeply Nested Hierarchies:** When models contain multiple levels,
  `SelectMany` cleanly projects all items at a specified depth.
- **Collection Transformation:** When the transformation of each source element
  results in a sequence, and you want to bring all those together.

---

### Method Syntax Examples

#### Flattening Child Collections

Assume:

```csharp
public class Student
{
    public int ID { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public List<string> Programming { get; set; }
}
```

Usage:

```csharp
List<string> allPrograms = students.SelectMany(s => s.Programming).ToList();
foreach (string prog in allPrograms)
{
    Console.WriteLine(prog);
}
// Output: All programming languages across all students, flattened into one list
```

#### Flattening and Removing Duplicates

```csharp
List<string> distinctPrograms = students
    .SelectMany(s => s.Programming)
    .Distinct()
    .ToList();
```

This merges child collections and removes duplicates.

#### Using Result Selector

```csharp
var studentAndProgram = students
    .SelectMany(
        s => s.Programming,
        (student, prog) => new { StudentName = student.Name, Program = prog }
    ).ToList();

foreach (var item in studentAndProgram)
{
    Console.WriteLine($"{item.StudentName} => {item.Program}");
}
```

Here, the result selector produces a projection combining parent and child
info—useful for reporting or for mapping to denormalized data.

#### Combining With Index

```csharp
var indexed = petOwners.SelectMany(
    (owner, idx) => owner.Pets.Select(pet => $"{idx + 1}-{pet}")
);
```

The index argument in `SelectMany` can be used for advanced mapping schemes.

---

### SQL Equivalent

Because SQL has no concept of in-row nested collections, **SelectMany’s**
behavior equates to:

- **CROSS APPLY** or **INNER JOIN** on foreign keys in a normalized schema.
- **UNION ALL** or use of table-valued functions for flattening.

Example analogous SQL:

```sql
SELECT s.Name, sp.Program
FROM Students s
JOIN StudentPrograms sp ON s.ID = sp.StudentID;
```

---

## `Join`

### Usage Scenarios

- **Relational Association:** Combine customers with orders, students with
  departments, or products with categories—whenever matching keys connect two
  datasets.
- **Composite Key Mapping:** When matching requires a compound field (e.g.,
  first name + last name).
- **Data Enrichment:** Supplement a base dataset with additional attributes from
  another source.
- **Multiple Joins:** Multi-table equivalent merges for rich denormalized
  outputs.

---

### Method Syntax Examples

#### Standard Inner Join

Associate customers with orders:

```csharp
var result = customers.Join(
    orders,
    customer => customer.Id,
    order => order.CustomerId,
    (customer, order) => new { customer.Name, order.OrderId }
);

foreach (var row in result)
{
    Console.WriteLine($"{row.Name}, {row.OrderId}");
}
```

**Equivalent SQL:**

```sql
SELECT c.Name, o.OrderId
FROM Customers c
INNER JOIN Orders o ON c.Id = o.CustomerId;
```

#### Composite Key Join

If joining on multiple fields:

```csharp
var result = employees.Join(
    departments,
    e => new { e.DepartmentId, e.Location },
    d => new { d.DepartmentId, d.Location },
    (e, d) => new { e.Name, d.Name }
);
```

**Equivalent SQL:**

```sql
SELECT e.Name, d.Name
FROM Employees e
JOIN Departments d
ON e.DepartmentId = d.DepartmentId AND e.Location = d.Location;
```

#### Three-Way Joins and Chaining

Multi-stage joining:

```csharp
var result = students
    .Join(departments,
        s => s.DepartmentID, d => d.ID,
        (student, department) => new { student, department })
    .Join(teachers,
        sd => sd.department.TeacherID, t => t.ID,
        (sd, teacher) => new
        {
            StudentName = $"{sd.student.FirstName} {sd.student.LastName}",
            DepartmentName = sd.department.Name,
            TeacherName = $"{teacher.First} {teacher.Last}"
        });

foreach (var obj in result)
{
    Console.WriteLine($"{obj.StudentName} studies in {obj.DepartmentName}, led by {obj.TeacherName}");
}
```

**Equivalent SQL:**

```sql
SELECT s.FirstName + ' ' + s.LastName, d.Name, t.First + ' ' + t.Last
FROM Students s
JOIN Departments d ON s.DepartmentID = d.ID
JOIN Teachers t ON d.TeacherID = t.ID;
```

---

### SQL Equivalent

`Join` in LINQ is functionally equivalent to SQL's `INNER JOIN`, returning only
the rows where matching keys are found on both sides.

---

## `GroupJoin`

The **`GroupJoin`** method extends `Join` by pairing each element of an outer
sequence with a collection of matching elements from an inner sequence, based on
keys. The result is a hierarchical, parent-child or master-detail relationship
where every outer item is always present, even if no matching inner elements
exist (in which case the child collection is empty). This closely mirrors SQL’s
`LEFT OUTER JOIN`, though LINQ’s output is nested rather than tabular.

### Usage Scenarios

- **Building Hierarchical Datasets:** Group employees per department, orders per
  customer, or children per parent.
- **Reporting and Grouped Outputs:** Display a list of categories, each with an
  embedded group of products.
- **Left Outer Join Semantics:** Include unmatched parent records gracefully,
  with empty child collections.
- **Efficiency in Parent-Child Traversals:** Rapidly traverse hierarchy
  relationships.

---

### Method Syntax Examples

#### Hierarchy Example: Departments with Employees

Assuming:

```csharp
public class Department { public int ID; public string Name; }
public class Employee { public int ID; public string Name; public int DepartmentId; }
```

```csharp
var groupJoin = departments.GroupJoin(
    employees,
    dept => dept.ID,
    emp => emp.DepartmentId,
    (dept, emps) => new { dept, emps }
);

foreach (var item in groupJoin)
{
    Console.WriteLine("Department :" + item.dept.Name);
    foreach (var employee in item.emps)
    {
        Console.WriteLine(" EmployeeID: " + employee.ID + " , Name: " + employee.Name);
    }
}
```

**Output:** Each department, followed by all employees in the department.

#### Flat Join via GroupJoin + SelectMany

To flatten hierarchical results to a sequence like `Join`:

```csharp
var flatJoin = departments
    .GroupJoin(
        employees,
        dept => dept.ID,
        emp => emp.DepartmentId,
        (dept, emps) => new { dept, emps }
    )
    .SelectMany(
        group => group.emps.DefaultIfEmpty(), // For left-outer join
        (group, emp) => new { Department = group.dept.Name, EmployeeName = emp?.Name ?? "No Employee" }
    );
```

---

### SQL Equivalent

- **Hierarchical group:** `LEFT JOIN` combined with GROUP BY, or simply a left
  join when no aggregation is needed.

```sql
SELECT d.Name AS DepartmentName, e.ID AS EmployeeID, e.Name AS EmployeeName
FROM Departments d
LEFT JOIN Employees e ON d.ID = e.DepartmentId;
```

---

## `GroupBy`

### Usage Scenarios

- **Aggregation by Category:** Total sales per region, employees per department,
  etc.
- **Data Deduplication:** Finding unique elements by property (e.g., first
  product per name).
- **Hierarchical Display:** Multi-level groupings—group by department, then by
  role.
- **Data Summarization:** Producing statistics like min, max, average per group.
- **Analysis and Reporting:** Pivot-style grouping for dashboards.

---

### Method Syntax Examples

#### Single Property Grouping

Group students by branch:

```csharp
var groupedStudents = students.GroupBy(s => s.Branch);
foreach (var group in groupedStudents)
{
    Console.WriteLine($"{group.Key}: {group.Count()}");
    foreach (var student in group)
    {
        Console.WriteLine($" Name: {student.Name}, Age: {student.Age}, Gender: {student.Gender}");
    }
}
```

**SQL Equivalent:**

```sql
SELECT Branch, COUNT(*) FROM Students GROUP BY Branch;
```

#### Multiple Property Grouping (Composite Key)

```csharp
var grouped = sales.GroupBy(s => new { s.ProductId, Month = s.SaleDate.Month });
foreach (var group in grouped)
{
    Console.WriteLine($"Product: {group.Key.ProductId} - Month: {group.Key.Month}");
}
```

**SQL Equivalent:**

```sql
SELECT ProductId, MONTH(SaleDate) AS Month, SUM(Quantity)
FROM Sales
GROUP BY ProductId, MONTH(SaleDate)
ORDER BY ProductId, Month;
```

#### Aggregation

Calculate total amount per group:

```csharp
var totalAmountPerCustomer = orders
    .GroupBy(o => o.CustomerId)
    .Select(g => new { CustomerId = g.Key, TotalAmount = g.Sum(o => o.Amount) });

foreach (var group in totalAmountPerCustomer)
{
    Console.WriteLine($"Customer {group.CustomerId} Total: {group.TotalAmount}");
}
```

**SQL Equivalent:**

```sql
SELECT CustomerId, SUM(Amount) FROM Orders GROUP BY CustomerId;
```

#### Removing Duplicates by Property

```csharp
var distinctProducts = products.GroupBy(p => p.Name).Select(g => g.First()).ToList();
```

#### Nested Grouping (Multi-level)

```csharp
var byDeptAndRole = employees
    .GroupBy(e => e.Department)
    .Select(deptGroup => new
        {
            Department = deptGroup.Key,
            Roles = deptGroup.GroupBy(e => e.Role)
        });
```

---

### SQL Equivalent

- **GroupBy in LINQ** maps directly to SQL’s `GROUP BY`.
- Aggregates are applied as in SQL—with methods like `Sum`, `Count`, `Min`, etc.

---

## `ToLookup`

**ToLookup** creates a one-to-many index (lookup) from an `IEnumerable<T>`,
mapping keys to collections of elements. This is conceptually similar to a
`Dictionary<TKey, TValue>`, but allows to associate multiple values per
key—making it excellent for fast, repeatable key-based retrievals.

**The major difference from GroupBy:** ToLookup executes eagerly/immediately
(consuming the source at invocation); GroupBy is deferred until enumeration.
ToLookup’s results are indexed, supporting efficient, repeated lookups.

### Usage Scenarios

- **Efficient Key-Based Access:** When repeated or fast group lookups by key are
  needed.
- **Grouping Data for Read-Optimized Access:** Serve repeated reads in in-memory
  reporting or API scenarios.
- **Multiple Values per Key, including Duplicates:** Useful when the input can
  have non-unique keys.
- **Avoiding Deferred Execution Issues:** When you require stable groupings
  after source changes.

---

### Method Syntax Examples

#### Group By Single Key

```csharp
var branchLookup = students.ToLookup(s => s.Branch);

foreach (var group in branchLookup)
{
    Console.WriteLine($"{group.Key} : {group.Count()}");
    foreach (var student in group)
    {
        Console.WriteLine($" Name: {student.Name}, Age: {student.Age}, Gender: {student.Gender}");
    }
}
```

#### Group By Multiple Keys

```csharp
var branchGenderLookup = students.ToLookup(s => new { s.Branch, s.Gender });
```

#### Efficient Key Retrieval

```csharp
var departmentLookup = employees.ToLookup(e => e.Department);
var itDepartmentEmployees = departmentLookup["IT"];
```

---

### SQL Equivalent

While SQL lacks a one-to-many dictionary structure, retrieving groups by key is
done with:

```sql
SELECT Branch, Name, Age, Gender FROM Students ORDER BY Branch;
```

Efficient retrieval by key is typically achieved by filtering:

```sql
SELECT * FROM Students WHERE Branch = 'CSE';
```

**GroupBy in SQL follows the same logic but returns aggregates, not
collections.**

---

## SQL `GROUP BY` Clause

### Purpose and Basic Syntax

```sql
SELECT Country, COUNT(*) AS CustomerCount
FROM Customers
GROUP BY Country;
```

- All columns in the `SELECT` list should either be aggregated or appear in
  `GROUP BY`.
- If you omit an aggregate function, `GROUP BY` behaves like `SELECT DISTINCT`
  for those columns but is rarely used this way in practice.
- `GROUP BY` can be used with one or multiple columns (composite/grouping keys).

---

### Grouping by Multiple Columns (Composite Keys)

Grouping by multiple columns increases granularity. Each unique combination of
values constitutes a group.

**Example – Count Orders by Country and City:**

```sql
SELECT Country, City, COUNT(*) AS NumOrders
FROM Orders
GROUP BY Country, City;
```

This produces a row for each unique `(Country, City)` combination, along with
the count.

---

### Common Aggregate Functions Used with `GROUP BY`

| Function | Description             | Example          |
| -------- | ----------------------- | ---------------- |
| COUNT()  | Number of rows in group | COUNT(\*)        |
| SUM()    | Sum of values in group  | SUM(SalesAmount) |
| AVG()    | Average value in group  | AVG(Age)         |
| MIN()    | Smallest value in group | MIN(Price)       |
| MAX()    | Largest value in group  | MAX(Score)       |

**Example with Multiple Aggregates:**

```sql
SELECT Department,
       COUNT(*) AS Headcount,
       SUM(Salary) AS TotalSalary,
       AVG(Salary) AS AvgSalary,
       MIN(HireDate) AS EarliestHire
FROM Employees
GROUP BY Department;
```

---

## SQL `HAVING` Clause

### Purpose and Syntax

The `HAVING` clause filters **groups after aggregation**, unlike `WHERE`, which
filters rows before grouping.

**Basic Syntax:**

```sql
SELECT column1, AGG_FUNC(column2)
FROM table
[WHERE condition]
GROUP BY column1
HAVING AGG_FUNC(column2) operator value
[ORDER BY column1];
```

**Example – Find Products with Sales above 1,000 Units:**

```sql
SELECT ProductID, SUM(Quantity) AS TotalSold
FROM OrderDetails
GROUP BY ProductID
HAVING SUM(Quantity) > 1000;
```

**Key Points:**

- Use `HAVING` with aggregate functions (`SUM`, `COUNT`, etc.).
- Use `WHERE` for filtering before aggregation.
- It’s common to combine `WHERE` and `HAVING` for different filtering stages.

---

### Difference Between `WHERE` and `HAVING`

| Aspect          | WHERE                       | HAVING                            |
| --------------- | --------------------------- | --------------------------------- |
| Applies To      | Individual rows (pre-group) | Groups (post-group)               |
| Aggregate Funcs | Not allowed                 | Allowed (e.g., `SUM(...) > 100`)  |
| Placement       | Before `GROUP BY`           | After `GROUP BY`                  |
| Typical Usage   | Filter raw data             | Filter grouped/aggregated results |

**Example Combining WHERE and HAVING:**

```sql
SELECT Country, SUM(Amount) AS TotalSales
FROM Orders
WHERE OrderDate >= '2024-01-01'
GROUP BY Country
HAVING SUM(Amount) > 5000;
```

---

## LINQ Grouping: Method Syntax Equivalents

### LINQ .NET Overview

In C#, **LINQ** provides the `GroupBy` method for grouping, **mirroring SQL's
`GROUP BY`**. You project and filter on the result using method syntax; **there
is no direct `HAVING`**—instead, you'll use `.Where(...)` after grouping to
filter on aggregate values.

**Basics:**

- `GroupBy` produces an `IGrouping<TKey, TSource>`, an enumerable for each group
  with a `Key` property (the group key).
- Aggregations are run on each group using LINQ extensions: `Count()`, `Sum()`,
  `Average()`, `Min()`, `Max()`, etc.
- To mimic `HAVING`, chain `.Where(...)` after `GroupBy`.

---

### **LINQ Method Syntax: Basic Grouping**

#### Group by a Single Property

```csharp
var groups = data.GroupBy(item => item.Category);
foreach (var group in groups)
{
    Console.WriteLine($"Category: {group.Key}, Count: {group.Count()}");
}
```

#### Group by Composite Key (Multiple Properties)

```csharp
var groups = data.GroupBy(item => new { item.Department, item.ProductType });
foreach (var group in groups)
{
    Console.WriteLine($"Dept: {group.Key.Department}, Type: {group.Key.ProductType}, Count: {group.Count()}");
}
```

- Anonymous type as group key allows grouping by multiple columns, mirroring
  `GROUP BY col1, col2` in SQL.

---

### **LINQ Aggregations Per Group**

To compute aggregates for each group (e.g., total count, sum), project the
results using `Select` after `GroupBy`.

```csharp
var summary = data.GroupBy(item => item.ProductType)
    .Select(g => new
    {
        Type = g.Key,
        Count = g.Count(),
        TotalPrice = g.Sum(item => item.Price),
        AvgPrice = g.Average(item => item.Price)
    });

foreach (var group in summary)
{
    Console.WriteLine($"Type: {group.Type}, Count: {group.Count}, Total: {group.TotalPrice}, Avg: {group.AvgPrice}");
}
```

---

### **Filtering Grouped Results (`HAVING`)**

To filter groups by aggregate properties (i.e., LINQ equivalent of SQL
`HAVING`):

```csharp
var result = data.GroupBy(item => item.ProductType)
    .Where(g => g.Count() > 10)
    .Select(g => new
    {
        Type = g.Key,
        Count = g.Count()
    });
```

Or for composite aggregate conditions:

```csharp
var result = data.GroupBy(item => new { item.Department, item.ProductType })
    .Where(g => g.Sum(item => item.Sales) > 5000 && g.Average(item => item.Price) > 100)
    .Select(g => new
    {
        g.Key.Department,
        g.Key.ProductType,
        TotalSales = g.Sum(item => item.Sales),
        AvgPrice = g.Average(item => item.Price),
    });
```

**This directly parallels `HAVING SUM(Sales) > 5000 AND AVG(Price) > 100` in
SQL.**

---

### **Filter Data Before Grouping (`WHERE`)**

Filter with `.Where` **before** `GroupBy` to restrict the data considered in
grouping, just as `WHERE` is used in SQL:

```csharp
var filteredGroups = data
    .Where(item => item.IsActive)
    .GroupBy(item => item.Department)
    .Select(g => new
    {
        Department = g.Key,
        ActiveCount = g.Count()
    });
```

Equivalent SQL:

```sql
SELECT Department, COUNT(*)
FROM data
WHERE IsActive = 1
GROUP BY Department
```

---

## Overview of SQL Join Types

At its core, a **join** associates rows from two or more tables based on a
specified relationship, typically involving keys. Each join variant has unique
semantics, impacting what data is included in the result set. Understanding the
output shape of each join is a prerequisite to effective SQL and LINQ usage.

The five canonical SQL join types are summarized in the table below:

| SQL Join Type    | Description                                                        |
| ---------------- | ------------------------------------------------------------------ |
| INNER JOIN       | Rows with matching keys in both tables only.                       |
| LEFT OUTER JOIN  | All left table rows; right table data (if matched) or NULLs.       |
| RIGHT OUTER JOIN | All right table rows; left table data (if matched) or NULLs.       |
| FULL OUTER JOIN  | All rows from both tables, combining matched and unmatched.        |
| CROSS JOIN       | Cartesian product: every row of Table A with every row of Table B. |

It is important to note that while SQL natively supports all five, **LINQ** does
not provide direct operators for RIGHT and FULL OUTER JOIN, requiring creative
equivalents using available method syntax building blocks.

---

## INNER JOIN

### SQL INNER JOIN: Syntax and Semantics

**INNER JOIN** returns rows with matching values in both joined tables based on
a specified predicate. Unmatched rows from either table are excluded from the
result.

**SQL Example:**

```sql
SELECT e.EmployeeName, d.DepartmentName
FROM Employees e
INNER JOIN Departments d ON e.DepartmentID = d.DepartmentID;
```

Only employees with a valid department mapping appear in the result.

---

### LINQ Method Syntax Equivalent

In LINQ (method/fluent syntax), the direct equivalent is the `.Join()` method.
This method projects each pair of matched elements from the two collections
having matching keys.

**LINQ Example:**

```csharp
var query = employees.Join(
    departments,
    employee => employee.DepartmentID,
    department => department.DepartmentID,
    (employee, department) => new {
        EmployeeName = employee.Name,
        DepartmentName = department.Name
    }
);
```

This code returns only pairs where the _employee_'s DepartmentID matches a
_department_'s ID—precisely like SQL’s INNER JOIN.

**Pattern Explanation:**

- The join key selectors (`employee => employee.DepartmentID`, `department =>
department.DepartmentID`) define how records relate.
- The result selector (`(employee, department) => ...`) projects the shape of
  the output.
- Only records with key matches in both collections appear in the final
  sequence.

---

**When to Use INNER JOIN (SQL or LINQ):**

Use INNER JOINs when data from two sets/tables should be included only where
there are mutual matches. Examples include finding orders that have a customer,
or students registered in courses.

**Performance Consideration:**

In SQL databases, a well-tuned INNER JOIN typically performs efficiently,
leveraging indexes on joined keys. In LINQ-to-Objects, `.Join()` is implemented
as a hash join, offering O(n + m) performance, but ultimately depends on the
underlying data structure.

For data sources that are ultimately translated to SQL (e.g., via LINQ to SQL or
EF Core), the generated SQL mirrors what you would hand-write in SQL.

---

## LEFT OUTER JOIN

### SQL LEFT OUTER JOIN: Syntax and Semantics

A **LEFT OUTER JOIN** returns all rows from the "left" table, along with
matching rows from the "right" table, or NULLs for right-side columns where no
match exists.

**SQL Example:**

```sql
SELECT c.CustomerName, o.OrderID
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID;
```

All customers are listed, with their orders if present, or NULL if none exist.

---

### LINQ Method Syntax Equivalent

LINQ does not have a single method for left join, but the idiomatic approach
uses a combination of `.GroupJoin()`, `.SelectMany()`, and `.DefaultIfEmpty()`:

**LINQ Example:**

```csharp
var query = customers
    .GroupJoin(
        orders,
        customer => customer.CustomerID,
        order => order.CustomerID,
        (customer, orderGroup) => new { customer, orderGroup }
    )
    .SelectMany(
        x => x.orderGroup.DefaultIfEmpty(),
        (x, order) => new {
            CustomerName = x.customer.Name,
            OrderID = order == null ? (int?)null : order.OrderID
        }
    );
```

**Pattern Explanation:**

- `.GroupJoin()` groups orders for each customer.
- `.SelectMany(..., (outer, inner) => ...)` flattens results, pairing each
  customer with each of their orders.
- `.DefaultIfEmpty()` provides a _null_ when there are no matching orders,
  enabling the "outer" semantics.

**This produces identical results to SQL's LEFT OUTER JOIN: every customer
appears, orders may be null.**

**Variants and Practical Example:**

- Listing all departments, even those with no employees.
- All customers, regardless of whether they have placed orders.

---

**When to Use a LEFT OUTER JOIN:**

Left joins are ideal for preserving all items from your “primary” set, embedding
details from a related set if available. Reporting scenarios (e.g., all users
and last login activity, even if some never logged in) often use left joins.

**Performance Considerations:**

- On SQL Server and most databases, LEFT OUTER JOIN is efficiently implemented,
  but can be more costly than INNER JOIN due to inclusion of non-matching rows.
- In LINQ-to-SQL or EF Core, the above pattern is translated into proper SQL at
  runtime. However, in LINQ-to-Objects, performance is similar to the nested
  loop, but with optimizations through grouping (hashing).
- If you use `.DefaultIfEmpty()` on large groups, be aware this can increase
  memory usage.

**Best Practice:** Filter as early as possible (before the join), not after;
filtering post-join can sometimes inadvertently turn an OUTER join into an INNER
join.

---

## RIGHT OUTER JOIN

### SQL RIGHT OUTER JOIN: Syntax and Semantics

A **RIGHT OUTER JOIN** returns all rows from the “right” table, plus matched
rows from the “left”. Where there is no left match, columns from the left table
are NULL.

**SQL Example:**

```sql
SELECT o.OrderID, c.CustomerName
FROM Orders o
RIGHT JOIN Customers c ON o.CustomerID = c.CustomerID;
```

Equivalent to swapping the left/right roles in a left join; not all databases
support RIGHT JOIN natively, but SQL Server, PostgreSQL, and Oracle all do.

---

### LINQ Method Syntax Equivalent

**LINQ does not directly support RIGHT OUTER JOIN.** The idiomatic solution is
to swap the input sets and apply the left join pattern:

**LINQ Example:**

```csharp
var query = orders
    .GroupJoin(
        customers,
        order => order.CustomerID,
        customer => customer.CustomerID,
        (order, customerGroup) => new { order, customerGroup }
    )
    .SelectMany(
        x => x.customerGroup.DefaultIfEmpty(),
        (x, customer) => new {
            OrderID = x.order.OrderID,
            CustomerName = customer?.Name ?? "No Customer"
        }
    );
```

- Here, `orders` (right table in SQL) becomes the primary source.
- Applying `.GroupJoin` and `.SelectMany` as per left join semantics returns all
  orders, and their customers if any.
- For a “true” right join (as in SQL), you would typically reverse the tables
  from the left join pattern. That is, to achieve `A RIGHT JOIN B`, perform `B
LEFT JOIN A`.

**Use Case:** Report all products, even those never ordered, by right joining
orders to products.

---

**When to Use RIGHT OUTER JOINs:**

Use when you wish to preserve all records from the “secondary” table and include
primary data where available. For example, products without sales, content
without comments.

**Performance and Practical Notes:**

- In SQL, a RIGHT JOIN is just a syntactic mirror of LEFT JOIN; the query
  optimizer can freely reorder.
- In LINQ-to-Objects, there is no functional distinction; reverse the collection
  order to simulate right join.
- In practice, RIGHT JOIN is less commonly needed because its requirements can
  nearly always be met via rewritten LEFT JOINs with table order reversed.

---

## FULL OUTER JOIN

### SQL FULL OUTER JOIN: Syntax and Semantics

A **FULL OUTER JOIN** returns all rows from both tables: matches where possible,
and NULL-filled “outer” rows from both sides where no match exists.

**SQL Example:**

```sql
SELECT a.ID, a.Name, b.Value
FROM TableA a
FULL OUTER JOIN TableB b ON a.ID = b.ID;
```

- All combinations of A and B, with NULLs where no corresponding data exists.

---

### LINQ Method Syntax Equivalent

**LINQ does not have a built-in operator for FULL OUTER JOIN**. It must be
emulated by combining LEFT and RIGHT OUTER JOINs and removing duplicates (i.e.,
union the two sets and filter for uniqueness):

**LINQ Example:**

```csharp
// Left Outer Join: all from A, matches from B
var left = asList
    .GroupJoin(
        bsList,
        a => a.ID,
        b => b.ID,
        (a, groupB) => new { a, groupB }
    )
    .SelectMany(
        ab => ab.groupB.DefaultIfEmpty(),
        (ab, b) => new { A = ab.a, B = b }
    );

// Right Outer Join: all from B, matches from A (to find B's unmatched)
var right = bsList
    .GroupJoin(
        asList,
        b => b.ID,
        a => a.ID,
        (b, groupA) => new { b, groupA }
    )
    .SelectMany(
        ba => ba.groupA.DefaultIfEmpty(),
        (ba, a) => new { A = a, B = ba.b }
    )
    .Where(x => x.A == null);

// FULL OUTER JOIN: union results, avoiding duplicates
var fullOuterJoin = left.Union(right);
```

**Pattern Explanation:**

- Perform a left outer join from A to B, grabbing all A’s and their
  corresponding B’s (or NULL).
- Perform a left outer join from B to A, grabbing all B’s and their A’s, but
  keep only those where A is NULL (i.e., those not already in the previous
  result).
- Union both results for the complete FULL OUTER JOIN effect.

**Use Cases:**

- Comparing datasets from two sources to find overlaps and differences.
- Merging data where neither table can be considered primary.

**Performance Consideration:**

- FULL OUTER JOINs can be expensive since both tables must be scanned.
- In LINQ, care must be taken to avoid unnecessary recomputation; the union
  operation’s performance depends on the key selector and set size.

---

**Notes:**

- In databases not supporting native FULL OUTER JOIN (e.g., MySQL), this pattern
  can be simulated using LEFT JOIN, RIGHT JOIN, and UNION.
- For large datasets, attention to set uniqueness and memory usage is critical.

---

## CROSS JOIN

### SQL CROSS JOIN: Syntax and Semantics

A **CROSS JOIN** produces the Cartesian product: every row of the first table,
paired with every row of the second table.

**SQL Example:**

```sql
SELECT a.Name, b.Value
FROM TableA a
CROSS JOIN TableB b;
```

If A has _m_ rows and B has _n_ rows, the result contains m × n rows.

---

### LINQ Method Syntax Equivalent

LINQ does not require a dedicated method for CROSS JOIN; use `.SelectMany()` on
the outer sequence to project each element to the entire inner sequence,
returning the flattened result:

**LINQ Example:**

```csharp
var crossJoin = listA.SelectMany(
    a => listB,
    (a, b) => new { A = a, B = b }
);
```

**Pattern Explanation:**

- Each element `a` in `listA` is paired with every element `b` in `listB`,
  generating the Cartesian product—a cross join.

**Practical Examples:**

- Generating scheduling slots for every employee and every shift.
- All possible product-category pairs.

**Performance Considerations:**

- For large collections, the cross join creates rapidly growing output (n × m).
  Use cautiously to avoid memory explosions.

**Best Practice:** Avoid unless every possible combination is genuinely
required.

---

## Comparative Table: SQL Joins and LINQ Method Syntax Equivalents

| SQL Join Type    | SQL Example                   | LINQ Method Syntax Example                                                    |
| ---------------- | ----------------------------- | ----------------------------------------------------------------------------- |
| INNER JOIN       | `A INNER JOIN B ON a.ID=b.ID` | `A.Join(B, a => a.ID, b => b.ID, (a, b) => ...)`                              |
| LEFT OUTER JOIN  | `A LEFT JOIN B ON a.ID=b.ID`  | `A.GroupJoin(B, ..., (a, gB) => ...).SelectMany(... gB.DefaultIfEmpty() ...)` |
| RIGHT OUTER JOIN | `A RIGHT JOIN B ON a.ID=b.ID` | `B.GroupJoin(A, ..., (b, gA) => ...).SelectMany(... gA.DefaultIfEmpty() ...)` |
| FULL OUTER JOIN  | `A FULL JOIN B ON a.ID=b.ID`  | Combine above two and Union unique results                                    |
| CROSS JOIN       | `A CROSS JOIN B`              | `A.SelectMany(a => B, (a, b) => ...)`                                         |

Each LINQ code snippet reflects the general idiom but can be adjusted for custom
result projection or key construction.

---

## When to Use Each Join Type and Potential Pitfalls

### INNER JOIN

- Use when only records existing in both collections are meaningful.
- For normalized data and aggregated reporting.
- **Pitfall:** Missing non-matching data; always ensure you don't need unmatched
  records.

### LEFT OUTER JOIN

- Use when you need all records from your principal collection, optionally
  enriched with matches.
- Reporting, soft-matching, optional relationships.
- **Pitfall:** Filtering in the wrong place (after the join) might convert left
  joins to inner joins inadvertently.

### RIGHT OUTER JOIN

- Situations are rare in practice; usually solved by reversing sides of LEFT
  JOIN.
- **Pitfall:** Misunderstanding the directionality of data inclusion.

### FULL OUTER JOIN

- When merging disparate sources, or for comprehensive accuracy checks/audits.
- **Pitfall:** High resource consumption, potential for very sparse (mostly
  null-filled) rows.

### CROSS JOIN

- For all combinations, such as every user × every permission.
- **Pitfall:** Combinatorial explosion—use strict filters or avoid unless
  necessary.

---

## Advanced LINQ Join Patterns

### Composite Key Joins

LINQ supports joins on multiple keys by selecting an anonymous object as a key:

```csharp
A.Join(
    B,
    a => new { a.Key1, a.Key2 },
    b => new { b.Key1, b.Key2 },
    (a, b) => new { a, b }
);
```

_Useful for scenarios where natural/key integrity is based on multiple columns._

---

### Grouped Joins and Hierarchies

`.GroupJoin()` also enables return of hierarchical results: one “parent” object
with a set of “child” objects. This is formally called a grouped join, which is
akin to a SQL aggregation or navigation property:

```csharp
var query = departments.GroupJoin(
    employees,
    dept => dept.DepartmentID,
    emp => emp.DepartmentID,
    (dept, emps) => new { dept, Employees = emps }
);
```

_Hierarchical display of departments and their employees is a typical use case._

---

## Performance Considerations: SQL vs LINQ

### SQL Joins

- The SQL query optimizer chooses the best algorithm (hash join, merge join,
  nested loop) based on data size, indexes, and stats.
- **Indexes on join keys are critical** for efficient execution.
- Using SQL query execution plans (`EXPLAIN`) aids optimization.

**Best Practices:**

- Filter data before joining when possible.
- Avoid selecting unnecessary columns (avoid `SELECT *`).
- Use explicit `JOIN` ... `ON` syntax for clarity and optimizer support.

---

### LINQ Joins

LINQ’s internal join method behaviors depend on the underlying data source:

- **LINQ to Objects:** `.Join()` is implemented efficiently as a hash join—good
  for large in-memory collections.
- **LINQ to SQL/Entities:** LINQ expressions are translated to SQL, so
  performance mapping is as per the target RDBMS.
- **Deferred Execution:** LINQ queries are lazy; they execute only upon
  enumeration, which allows for further chaining of filters or projections.

**Pitfalls:**

- In LINQ-to-Objects, repeated enumeration of collections (in loops or nested
  queries) is slow for large data if not using hashes/dictionaries.
- Beware of `IEnumerable` vs. `IQueryable`: the former operates in memory, the
  latter is translated to SQL—the distinction is huge for data size and network
  round-trips.
- Avoid value manipulations or computations inside LINQ lambda joins—these can
  defeat translations and optimizations.

---

## 1. LINQ Expression Tree Fundamentals

Expression trees in .NET are tree-like data structures representing code—usually
in the form of lambda expressions—as objects, where each node is an expression,
such as a parameter, method call, or binary operation. At runtime, expression
trees can be created, traversed, manipulated, and (in many scenarios) compiled
into executable delegates. This architecture enables a level of meta-programming
and code-as-data, facilitating dynamic behavior hard to achieve through standard
compiled code.

### 1.1. What Is an Expression Tree?

An **expression tree** represents code in the form of a data structure. In
practical C#, this means that something as familiar as a method or lambda
expression can be reflected as a tree of objects—each representing a constant,
variable, method call, or binary operator. This is distinct from simply having
code compiled into IL and executed. The implications are profound: by working
with trees, LINQ providers and libraries can examine, rewrite, and translate
your code before it ever executes.

A typical expression tree for a simple addition could look like:

```csharp
// Automatically created by the C# compiler:
Expression<Func<int, int, int>> expr = (x, y) => x + y;
```

Alternatively, an equivalent expression tree can be constructed **manually**:

```csharp
ParameterExpression x = Expression.Parameter(typeof(int), "x");
ParameterExpression y = Expression.Parameter(typeof(int), "y");
BinaryExpression sum = Expression.Add(x, y);
Expression<Func<int, int, int>> expr = Expression.Lambda<Func<int, int, int>>(sum, x, y);
```

Here, the **Expression** class in `System.Linq.Expressions` acts as a factory
for expression nodes. The resulting tree can then be compiled and invoked.

---

### 1.2. Lambda Expressions, Delegates, and Expression Trees

- **Lambda expressions** in method syntax (`x => x * x`) can be converted to
  either:
  - **Delegates** (i.e., `Func<T>` or `Action<T>`)—runtime executable code.
  - **Expression trees** (i.e., `Expression<Func<T>>`)—a data structure
    describing the code, not immediately executable, but available for
    analysis or translation.

The distinction is critical:

- When you use a LINQ method on an in-memory collection (`IEnumerable<T>`), the
  predicate is a **delegate**.
- When you use LINQ on a queryable provider (`IQueryable<T>`), the predicate is
  an **expression tree**—enabling query translation (e.g., to SQL in EF Core).

**Example:** Assignment to `Func<T, TResult>` creates a compiled delegate.
Assignment to `Expression<Func<T, TResult>>` builds an expression tree.

---

### 1.3. Expression Trees: Method Syntax vs Query Syntax

In C# LINQ, both **method syntax** and **query syntax** ultimately produce the
same underlying expression trees or delegates. However, from the perspective of
expression trees and provider translation, method syntax is often the clearer,
more direct representation; and in code samples below, method syntax is used
exclusively as requested.

**Example:**

```csharp
var results = myQueryable.Where(x => x.IsActive).OrderBy(x => x.Name);
```

This syntax accepts an `Expression<Func<T, bool>>` for `Where` when applied to
an `IQueryable<T>`, and generates expression trees.

---

### 1.4. Automatic Generation of Expression Trees

C# automatically generates expression trees when a lambda expression is assigned
to a variable of type `Expression<TDelegate>`.

```csharp
Expression<Func<Person, bool>> isAdult = person => person.Age >= 18;
```

The compiler builds an expression tree representing this predicate.

---

## 2. Constructing Expression Trees

### 2.1. Automatic Expression Trees from Lambdas

For most common LINQ scenarios, developers rely on the compiler to convert
lambdas into expression trees as arguments to method syntax LINQ operators.

```csharp
IQueryable<Person> query = dbContext.Persons
    .Where(p => p.FirstName == "Alice" && p.Age > 25)
    .OrderBy(p => p.LastName);
```

In LINQ to Entities, this creates a tree rather than a delegate, enabling
translation to SQL.

---

### 2.2. Manual Construction of Expression Trees

For scenarios involving dynamic queries or programmatically constructed logic,
you must build expression trees manually.

**Manual construction follows these steps:**

1. **Create ParameterExpression** objects for lambda arguments:
   ```csharp
   var param = Expression.Parameter(typeof(Person), "p");
   ```
2. **Access Properties** using `Expression.Property`:
   ```csharp
   var age = Expression.Property(param, "Age");
   ```
3. **Create Constants** using `Expression.Constant`:
   ```csharp
   var ageValue = Expression.Constant(30);
   ```
4. **Build Operations** (e.g., comparison):
   ```csharp
   var predicate = Expression.GreaterThanOrEqual(age, ageValue);
   ```
5. **Wrap in a Lambda Expression:**
   ```csharp
   var lambda = Expression.Lambda<Func<Person, bool>>(predicate, param);
   ```

**Practical Example:** Build a filter on any property dynamically:

```csharp
// Filter persons by FirstName at runtime
public static Expression<Func<Person, bool>> CreateFilter(string propertyName, object value)
{
    var param = Expression.Parameter(typeof(Person), "p");
    var member = Expression.Property(param, propertyName);
    var constant = Expression.Constant(value);
    var body = Expression.Equal(member, constant);
    return Expression.Lambda<Func<Person, bool>>(body, param);
}
```

**Usage:**

```csharp
var predicate = CreateFilter("FirstName", "Alice");
var query = persons.Where(predicate);
```

This is essential for building advanced, user-driven, or metadata-driven
filtering interfaces.

---

### 2.3. Complex Dynamic Query Generation

Expression trees are also employed for more complex search scenarios:

- Multiple conditions (AND/OR)
- Nested properties (e.g., `p.Address.City == "Berlin"`)
- Method calls (e.g., `Contains`, `StartsWith`)

**Example – Build a dynamic AND expression:**

```csharp
public static Expression<Func<Person, bool>> BuildAndFilter(IDictionary<string, object> filters)
{
    var param = Expression.Parameter(typeof(Person), "p");
    Expression? body = null;
    foreach (var filter in filters)
    {
        var member = Expression.Property(param, filter.Key);
        var constant = Expression.Constant(filter.Value);
        var equals = Expression.Equal(member, constant);
        body = body == null ? equals : Expression.AndAlso(body, equals);
    }
    return Expression.Lambda<Func<Person, bool>>(body!, param);
}
```

**Usage:**

```csharp
var filters = new Dictionary<string, object> { {"FirstName", "Bob"}, {"Age", 40} };
var expr = BuildAndFilter(filters);
```

This produces a predicate equivalent to `p => p.FirstName == "Bob" && p.Age ==
40`.

**Example – Nested property:**

```csharp
Expression member = param;
foreach (var namePart in "Address.City".Split('.'))
    member = Expression.Property(member, namePart);
```

---

### 2.4. Visiting and Modifying Expression Trees: ExpressionVisitor

Expression trees are **immutable**. To "modify" them (e.g., to rewrite parts for
logging, optimization, or translation), you must construct a new tree by
traversing and replacing nodes.

**System.Linq.Expressions.ExpressionVisitor** provides a powerful and extensible
mechanism for this task.

**Example – Visitor pattern to change all AND to OR:**

```csharp
public class AndToOrExpressionVisitor : ExpressionVisitor
{
    protected override Expression VisitBinary(BinaryExpression node)
    {
        if (node.NodeType == ExpressionType.AndAlso)
            return Expression.MakeBinary(ExpressionType.OrElse, node.Left, node.Right, node.IsLiftedToNull, node.Method);
        return base.VisitBinary(node);
    }
}
```

**Usage:**

```csharp
Expression<Func<Person, bool>> expr = p => p.Age > 30 && p.Name.StartsWith("A");
var modified = new AndToOrExpressionVisitor().Visit(expr);
```

This would yield a predicate where `&&` becomes `||`.

---

### 2.5. Execution and Compilation of Expression Trees

Not all expression trees are directly executable. Only those representing lambda
expressions (`Expression<TDelegate>`) can be compiled and invoked:

```csharp
Expression<Func<int, int, int>> sum = (x, y) => x + y;
Func<int, int, int> compiled = sum.Compile();
int result = compiled(2, 3); // result == 5
```

For scenarios such as in-memory filtering after dynamic construction, this is
invaluable.

**Performance Note:** Creating and compiling expression trees incurs overhead;
thus, caching compiled delegates for reuse is a best practice.

---

### 2.6. Expression Trees in LINQ Providers and Query Translation

When you call a LINQ operator such as `.Where` with a predicate of type
`Expression<Func<T, bool>>` on an `IQueryable<T>`, the provider (e.g., EF Core,
MongoDB Driver) receives the expression tree and parses its nodes to generate an
appropriate query (like SQL or MongoDB query language).

**Translation Example:**

```csharp
var query = dbContext.Persons.Where(p => p.Age >= 18 && p.FirstName.StartsWith("A"));
```

- The expression tree representing the predicate is parsed by the provider,
- Each node (e.g., property access, method call) is mapped to the relevant SQL
  or provider-specific operation,
- The eventual SQL might be:
  ```sql
  SELECT * FROM Persons WHERE Age >= 18 AND FirstName LIKE 'A%'
  ```
- This dynamic enables LINQ’s "write once, run anywhere" philosophy; you can
  build queries against in-memory objects, SQL, XML, or web APIs using identical
  code structures.

---

## 3. Dynamic Query Generation with Expression Trees

### 3.1. Motivation and Real-World Scenarios

Dynamic query generation becomes essential when:

- Query logic depends on user input or metadata,
- Filters, sorts, and projections are not known until runtime,
- Frameworks want to provide extensible querying APIs (e.g., OData,
  Elasticsearch clients, data grid filters),
- Building business rules or plugin architectures where external code or
  policies drive filtering.

**Examples include:**

- Web APIs supporting search/filtering on arbitrary fields,
- Admin portals with metadata-driven dashboards,
- Reporting tools that let users define custom views.

---

### 3.2. Building Complex Queries at Runtime

Expression trees enable dynamic composition:

```csharp
IQueryable<Product> query = context.Products;

if (minPrice.HasValue)
{
    var min = Expression.Constant(minPrice.Value);
    var priceProp = Expression.Property(param, nameof(Product.Price));
    var minPred = Expression.GreaterThanOrEqual(priceProp, min);
    // Combine into lambda and use in Where
}

if (categories != null && categories.Any())
{
    // Build an OrElse chain for each category
}
```

These constructs allow you to progressively extend queries, often using helper
methods to ensure readability and composability.

---

### 3.3. Dynamic LINQ Providers and Query Translation

LINQ providers must traverse (visit) the expression tree for:

- Parsing method calls, operator expressions, and constants,
- Mapping properties and values to data source fields,
- Handling translation differences (e.g., C# `Contains` → SQL `LIKE`, method
  calls on enums),
- Optimizing, rewriting, or inlining expressions for performance.

Custom LINQ providers can be written using this infrastructure; for example,
query translation for custom databases or APIs can be achieved by walking and
rewriting expression trees using visitors or interpreters.

---

## 4. Expression Tree Visitor and Advanced Modification

### 4.1. ExpressionVisitor Class

The `ExpressionVisitor` base class provides traversal and node-rewriting
callbacks for all expression node types (e.g., `VisitBinary`, `VisitMethodCall`,
etc.).

**Use-cases:**

- Optimizing queries (e.g., constant folding, expression rewriting),
- Logging or analyzing query structure,
- Translating expressions to other languages (SQL, JavaScript, Elasticsearch
  DSL),
- Implementing cross-cutting concerns (e.g., security, soft-deletes,
  multi-tenancy).

**Example – SQL translation visitor:**

```csharp
protected override Expression VisitBinary(BinaryExpression b)
{
    sb.Append("(");
    Visit(b.Left);
    switch (b.NodeType)
    {
        case ExpressionType.And: sb.Append(" AND "); break;
        case ExpressionType.Equal: sb.Append(" = "); break;
        // ... others omitted for brevity
    }
    Visit(b.Right);
    sb.Append(")");
    return b;
}
```

**Result:** The visitor produces a SQL WHERE clause equivalent to the original
predicate.

---

### 4.2. Limitations of Expression Trees

While powerful, expression trees are not a full reflection of C# syntax—they do
not support:

- Statement lambdas (multi-line lambdas),
- Loops and control flow,
- Exception-handling blocks,
- C# features such as pattern matching or local functions,
- Async/await,
- Some complex object initializations,
- Delegates with variable/optional/named arguments.

This restriction ensures compatibility and stability for query translation
across different providers and versions.

---

## 5. Expression Tree Execution and Compilation

### 5.1. Compiling and Executing Expression Trees

After building an expression tree (either automatically by the compiler or
manually), you can _compile_ it into a delegate:

```csharp
Expression<Func<int, int, int>> add = (x, y) => x + y;
Func<int, int, int> compiled = add.Compile();
int result = compiled(2, 2); // Output: 4
```

- Expression trees compiled into delegates execute with similar speed as
  idiomatic lambdas
- Compilation incurs upfront cost; cache delegates for repeated invocation.

**Performance:** Compilation is slower than static code execution initially but
negligible once the delegate is cached. Techniques such as
[FastExpressionCompiler](https://github.com/dadhi/FastExpressionCompiler)
significantly reduce compilation overhead for highly dynamic scenarios.

---

### 5.2. Limitations in Execution

- Compiling executes local, in-memory logic. For LINQ providers like Entity
  Framework, only delegates that can be translated to a supported query will
  work—compiling and invoking only operates against in-memory objects
  (IEnumerable), not remote sources (IQueryable).
- If objects referenced by closures are disposed before delegate execution,
  runtime exceptions can occur.
- Expressions referencing types or methods unavailable at runtime (due to
  trimming, AOT, or deployment differences) will fail to compile or execute.

---

## 6. Compiled Queries: Concept, Benefits, and Method Syntax Implementation

### 6.1. What Are Compiled Queries?

A **compiled query** is a LINQ query whose structure is parsed, analyzed, and
compiled (usually into a delegate) just once and then reused multiple times with
different parameter values. This is especially significant in data-access
scenarios—where query parsing and translation to SQL can contribute substantial
startup cost on each call.

**Benefits:**

- **Performance:** Eliminates repeated parsing/translation, especially in large
  enterprise apps with high query volume.
- **Reusability:** For queries with the same structure but different parameter
  values, minimizes system resource usage.

Compiled queries are particularly valuable in ORM frameworks and LINQ providers
(like Entity Framework Core), where each query potentially triggers translation,
plan generation, and caching costs.

---

### 6.2. When to Use Compiled Queries

- The query logic/shape is known and unchanging but parameters may vary.
- The query is _executed frequently_ with different arguments.
- Performance profiling indicates repeated query translation is a significant
  cost.
- Scenarios such as web APIs, background processing, and data pipelines, where
  certain queries are executed on virtually every request.

**Caveats:**

- Compiled queries are less beneficial for highly variable or one-off queries.
- Not all features (e.g., Contains on in-memory collections) are compatible with
  compiled queries.
- There can be a small memory overhead as compiled queries are cached for
  duration in static fields.

---

### 6.3. Implementing Compiled Queries with Method Syntax

**Entity Framework (classic and Core) and LINQ to SQL provide APIs for compiling
queries.**

#### 6.3.1. Entity Framework 6 / LINQ to Entities

```csharp
static readonly Func<MyDbContext, decimal, IQueryable<Order>> compiledQuery =
    CompiledQuery.Compile(
        (MyDbContext ctx, decimal minAmount) =>
            ctx.Orders.Where(order => order.Total >= minAmount)
    );

using (var context = new MyDbContext())
{
    var result = compiledQuery(context, 1000m).ToList();
}
```

- Query is compiled on first use; subsequent invocations reuse the query plan.
- Only 16 parameters allowed; further arguments require a structure.

#### 6.3.2. Entity Framework Core (since 2.0)

```csharp
private static readonly Func<MyDbContext, string, IEnumerable<Customer>> getByCountry =
    EF.CompileQuery(
        (MyDbContext ctx, string country) =>
            ctx.Customers.Where(c => c.Country == country));

using (var context = new MyDbContext())
{
    var result = getByCountry(context, "USA").ToList();
}
```

- Supports both synchronous (`CompileQuery`) and asynchronous
  (`CompileAsyncQuery`)
- Used primarily when queries are invariants across requests.

#### 6.3.3. Manual Compilation Pattern

```csharp
Expression<Func<Person, bool>> expr = p => p.Age > 20;
Func<Person, bool> predicate = expr.Compile(); // now reusable
```

Can be used to precompile any expression tree, not just those part of a query
provider.

---

### 6.4. Compiled Queries: Usage Scenarios and Limitations

| Scenarios            | Best For                                    | Limitations                    |
| -------------------- | ------------------------------------------- | ------------------------------ |
| High-frequency calls | Caching expensive query parsing/translation | Only supports static structure |
| Data access layers   | Strongly-typed reusable query APIs          | Cannot change query shape      |
| Web APIs             | Anonymized, parameterized query logic       | Only primitive parameters      |
| Repos/services       | Separation of query logic/versioning        | Must be called consistently    |

**Critical Note:** Composing further LINQ operators or projections on _results_
of compiled queries (e.g., adding `.Where()` after compiling) often breaks query
caching and should be avoided—always parameterize _in_ the compiled expression
itself.

---

### 6.5. Best Practices

- Declare compiled query delegates as `static readonly Func<...>`.
- Use method syntax and avoid further query chaining after compilation.
- Prefer compiled queries only for well-established, repeated workload paths.
- In EF Core, use pooling (`AddDbContextPool`) to maximize cache re-use.
- Avoid compiled queries for ad-hoc or sporadic queries.

---

## 7. Performance and Limitations of Expression Trees and Compiled Queries

### 7.1. Performance Characteristics

- Expression tree compilation is a one-time cost, making repeated execution
  efficient.
- Execution speed for compiled delegates is close to that of equivalent compiled
  code.
- For high-frequency or performance-critical operations, consider
  high-performance libraries like FastExpressionCompiler, which can reduce
  compilation overhead by 10-40x.
- The cost and benefit curve depends on usage pattern: the more
  repeated/frequent a query or logic, the more benefit from compilation.

**Empirical results:** Benchmarks report negligible runtime difference after
compilation between compiled delegates, expression lambdas, and manually coded
methods (see [Code Maze, Stack Overflow]).

---

### 7.2. Limitations

- Not all C# features are expressible or translatable in expression trees (see
  §4.2 above).
- Expression trees cannot handle certain dynamic or context-specific scenarios.
- Logging, debugging, and stack traces can be harder to read in dynamically
  compiled tree code.
- Compiled queries must be static in shape—dynamic queries cannot take advantage
  of pre-compilation (though NativeAOT and pre-compilation in upcoming .NET
  versions are expected to improve this).

---

## 8. Practical Code Examples (Method Syntax Only)

### 8.1. Simple Filter Using Expression Tree (Method Syntax)

```csharp
Expression<Func<Person, bool>> isOver30 = p => p.Age > 30;
var filtered = context.People.Where(isOver30);
```

### 8.2. Dynamic Filter Construction

```csharp
public static Expression<Func<T, bool>> PropertyEquals<T>(string property, object value)
{
    var param = Expression.Parameter(typeof(T), "x");
    var left = Expression.Property(param, property);
    var right = Expression.Constant(value);
    var body = Expression.Equal(left, right);
    return Expression.Lambda<Func<T, bool>>(body, param);
}

// Usage
var predicate = PropertyEquals<Person>("LastName", "Smith");
var query = people.Where(predicate);
```

### 8.3. Compiled Query in EF Core

```csharp
public static readonly Func<MyDbContext, int, IEnumerable<Customer>> CustomersByCityId =
    EF.CompileQuery(
        (MyDbContext ctx, int cityId) =>
            ctx.Customers.Where(c => c.CityId == cityId));

IEnumerable<Customer> customers = CustomersByCityId(context, 1001);
```

### 8.4. Combining Multiple Dynamic Filters

```csharp
var filters = new Dictionary<string, object>
{
    { "FirstName", "Alice" },
    { "Age", 27 }
};

Expression<Func<Person, bool>> expr = BuildAndFilter(filters);
var ids = people.Where(expr).Select(p => p.Id); // Method syntax
```

### 8.5. Expression Tree Visitor for Custom Analysis

```csharp
public class NodeCountingExpressionVisitor : ExpressionVisitor
{
    private int _count = 0;
    public int NodeCount => _count;

    public override Expression Visit(Expression node)
    {
        if (node != null) _count++;
        return base.Visit(node);
    }
}

// Usage
var expr = (Expression<Func<int, int, int>>)((x, y) => x + y);
var visitor = new NodeCountingExpressionVisitor();
visitor.Visit(expr);
Console.WriteLine($"Node count: {visitor.NodeCount}");
```

---
