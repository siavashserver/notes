---
title: Query PLanner
---

## Query Planner

### Role of the query planner in SQL databases

The query planner (optimizer) transforms a parsed SQL statement into a concrete
execution plan the database engine will run. Its main responsibilities:

- Parse SQL into a logical plan and translate it to alternative physical plans
  (index scans, table scans, join algorithms, order-by strategies).
- Estimate costs for each plan based on table/index statistics, row counts, and
  I/O/CPU cost heuristics.
- Choose the lowest-cost plan and pass it to the execution engine.
- Apply transformations such as predicate pushdown, projection pushdown, join
  reordering, and use of materialized results when beneficial.

Why it matters: small SQL changes or stale statistics can make the planner pick
a dramatically worse plan, turning a few-millisecond query into seconds or
minutes.

---

### Common query‑level optimization tips (practical checklist)

1. Select only needed columns

- Use explicit column lists instead of SELECT \* to reduce I/O and network
  transfer.

2. Make WHERE clauses sargable

- Avoid wrapping indexed columns in functions or expressions (e.g., avoid
  DATE(col) = '2025-01-01'); instead rewrite to range predicates so indexes can
  be used.

3. Index strategically

- Index columns used in WHERE, JOIN ON, ORDER BY, and GROUP BY.
- Prefer composite indexes that match the most common query prefix order.
- Keep index columns as small as practical (use appropriate numeric types,
  VARCHAR lengths).

4. Avoid unnecessary full table scans

- If EXPLAIN shows type = ALL, consider appropriate indexes or query rewrite.

5. Optimize joins

- JOIN on indexed primary/foreign keys.
- Push selective predicates early; let the planner filter rows before expensive
  joins.

6. Replace inefficient constructs

- Use EXISTS instead of COUNT(\*) when you only need presence.
- Replace correlated subqueries with JOINs or derived tables when appropriate.
- Use LIMIT for pagination and prefer indexed pagination (seek method) over
  OFFSET for large offsets.

7. Use proper data types and normalization

- Smallest type that fits the data reduces IO and index size.
- Normalize to avoid unnecessary duplication; denormalize only when it
  measurably reduces expensive joins.

8. Avoid large temporary results

- ORDER BY without an index, GROUP BY on many rows, or DISTINCT may create
  on-disk temp tables. Add indexes or rewrite queries.

9. Control result set size

- Use LIMIT and filter early in your pipeline.

10. Be cautious with indexes: balance read vs write

- Each index speeds reads for certain queries but slows writes and increases
  storage.

---

### Server and schema configuration tips (MySQL‑specific highlights)

- innodb_buffer_pool_size: set to ~60–80% of available RAM for dedicated DB
  servers (InnoDB heavy workloads).
- innodb_log_file_size: large enough to avoid frequent flushes but balanced with
  recovery time.
- tmp_table_size / max_heap_table_size: increase when queries create large tmp
  tables in memory.
- sort_buffer_size / join_buffer_size: tune carefully; these are per-connection
  and can exhaust RAM if too large.
- query_cache: removed in newer MySQL versions; don’t rely on it for modern
  MySQL/MariaDB.
- Ensure statistics are collected and up-to-date (ANALYZE TABLE).

---

### How to inspect query performance in MySQL (commands and tools)

1. EXPLAIN (and EXPLAIN ANALYZE)

- EXPLAIN SELECT ... shows the chosen execution plan and key columns:
  - **id**: select identifier for subqueries/derived tables.
  - **select_type**: SIMPLE, PRIMARY, SUBQUERY, DERIVED.
  - **table**: table referenced.
  - **type**: join type (ALL, index, range, ref, eq_ref, const). Closer to
    const/eq_ref is better; ALL means full table scan.
  - **possible_keys**: indexes MySQL could use.
  - **key**: index actually used.
  - **key_len**: bytes of index used.
  - **ref**: columns compared to the index.
  - **rows**: optimizer estimate of rows to examine.
  - **Extra**: useful notes (Using index, Using where, Using temporary, Using
    filesort).
- EXPLAIN ANALYZE SELECT ... (MySQL 8+) executes the query and shows actual rows
  and timing — use it to compare estimates vs reality.

2. SHOW STATUS and performance counters

- SHOW GLOBAL STATUS and SHOW SESSION STATUS give counters (e.g., Questions,
  Slow*queries, Handler_read*\*). Use differences over time to spot hotspots.

3. Slow query log

- Enable slow_query_log and set long_query_time; inspect slow queries. Use
  pt-query-digest or MySQL Enterprise Monitor to aggregate and rank
  slow/expensive queries.

4. Performance Schema

- Enable and query Performance Schema to drill into statement latency, wait
  events, index usage, and I/O by query or user.

5. INFORMATION_SCHEMA and sys schema

- Query INFORMATION_SCHEMA.INNODB_BUFFER_PAGE, sys.schema_table_statistics to
  find hot tables and index usage.

6. SHOW ENGINE INNODB STATUS

- Good for diagnosing lock waits, long transactions, and InnoDB internals.

7. Profiling (deprecated) and EXPLAIN FORMAT=JSON

- EXPLAIN FORMAT=JSON provides detailed optimizer reasoning and index statistics
  used in cost estimates.

8. Client-side timing and sampling

- Use application-level logging, APM, or query sampling to find and reproduce
  slow queries under realistic load.

---

### Systematic process to inspect and optimize a slow query

1. Reproduce and measure

- Capture the query; run it with representative parameters.
- Measure wall time and resource impact.

2. EXPLAIN the query

- Run EXPLAIN and note join order, type, possible_keys, key, rows, Extra.

3. Compare estimates with reality

- Use EXPLAIN ANALYZE to see real rows examined vs optimizer estimates. Large
  estimation errors point to stale or misleading statistics or to data skew.

4. Common fixes (apply iteratively)

- Add or adjust indexes (single or composite) to cover WHERE, JOIN, ORDER BY.
- Rewrite the query to be sargable, remove functions from indexed columns,
  de-correlate subqueries.
- Limit early (add WHERE filters), avoid unnecessary ORDER BY or GROUP BY.
- Consider covering indexes (index contains all needed columns) to avoid
  lookups.
- For pagination, prefer keyset pagination (WHERE id > last_id LIMIT N).

5. Test impact

- Re-run EXPLAIN/EXPLAIN ANALYZE and measure runtime. Ensure changes don’t
  regress other queries.

6. If still slow, check server resources

- Check CPU, IO, swap, InnoDB buffer pool hit ratio; tune server settings or
  scale vertically/horizontally.

7. Plan larger changes

- Partition large tables (range/hash) to reduce scanned rows.
- Archive old data, shard/horizontal scale, or introduce read replicas for
  read-heavy workloads.

---

### Example: reading EXPLAIN and a small optimization

Given: EXPLAIN SELECT o.\* FROM orders o JOIN customers c ON o.customer_id =
c.id WHERE c.country = 'IR' ORDER BY o.created_at DESC LIMIT 50;

Findings:

- If EXPLAIN shows `type = ALL` on orders and `Using where; Using filesort` in
  Extra, likely full scan of orders then sort. Fixes:
- Ensure index on customers.country if filtering by country frequently.
- Prefer a composite index on orders (customer_id, created_at) so the join and
  ORDER BY can use an index to avoid filesort:
  - CREATE INDEX ix_orders_customer_created ON orders(customer_id, created_at
    DESC);
- Re-run EXPLAIN to confirm type moves to ref or index and Extra no longer shows
  filesort.

---

## Interview Questions

#### 1. What is the role of the MySQL query optimizer and how does it choose a plan?

Answer: The optimizer takes a parsed SQL statement, generates alternative
logical/physical plans (join orders, join algorithms, index vs table scan, use
of filesort/temp table), estimates the cost of each using table/index statistics
and cost heuristics, and selects the lowest-cost plan for execution. It applies
rule-based and cost-based transformations (predicate pushdown, projection
pushdown, join reordering). Explain why it matters: plan choice determines I/O,
CPU, memory and thus query latency and throughput.

Why mention in an interview: demonstrates understanding of cost-based decision
making and the importance of accurate statistics.

---

#### 2. How do you interpret EXPLAIN output in MySQL and what are the most important columns to read?

Answer: Key columns: id (query block), select_type, table, type (access method —
ALL, index, range, ref, eq_ref, const), possible_keys, key (index used), key_len
(bytes used), ref, rows (optimizer estimate), Extra (Using where, Using index,
Using temporary, Using filesort). Read type first (closer to const/eq_ref is
better), confirm key is appropriate, check rows to see estimated cardinality,
and inspect Extra for filesort or temporary which often indicate trouble.

Quick tip: use EXPLAIN FORMAT=JSON for deeper reasoning and EXPLAIN ANALYZE to
compare estimates vs actuals (MySQL 8+).

---

#### 3. When EXPLAIN shows type = ALL, what steps do you take?

Answer: ALL indicates full table scan. Steps: check WHERE predicates and whether
an index could be used; inspect cardinality and selectivity; consider adding an
index or a composite index that matches the predicate prefix order; rewrite
predicates to be sargable (avoid functions on indexed columns); check if
partition pruning or limiting could help; re-run EXPLAIN and EXPLAIN ANALYZE to
verify improvement.

Example: WHERE DATE(col)=... is non-sargable; rewrite as col >= '2025-01-01' AND
col < '2025-01-02'.

---

#### 4. Explain cardinality, histograms, and how stale statistics affect the optimizer.

Answer: Cardinality is the optimizer’s estimate of distinct values/row counts
used to predict rows scanned. Histograms (MySQL 8+) capture value distribution
for a column and improve selectivity estimates for skewed data. Stale or missing
statistics (low ANALYZE frequency) lead to poor row estimates and bad plan
choices (e.g., nested loop vs hash join). Fixes: run ANALYZE TABLE, create
histograms for skewed columns, monitor ANALYZE schedules, and use optimizer
trace/EXPLAIN JSON to identify misestimates.

Why it matters: large estimation errors are a top cause of huge performance
regressions.

---

#### 5. When and how do you use composite (multi-column) indexes?

Answer: Use composite indexes when queries filter or order by multiple columns
with a consistent left-most prefix usage (e.g., WHERE a = ? AND b = ? ORDER BY
c). A composite index matching the WHERE + ORDER BY can become a covering index
and avoid lookups and filesorts. Design by examining the most common query
patterns and their predicate/order exposure; avoid wide composite indexes for
low-selectivity prefixes.

Example: queries frequently use WHERE user_id = ? AND status = ? ORDER BY
created_at DESC → index(user_id, status, created_at DESC).

---

#### 6. How do you decide between adding an index and rewriting the query?

Answer: Measure current cost (EXPLAIN/ANALYZE). If queries are logically simple
but lack an index, adding a targeted index is usually fastest. If predicate
logic is preventing index use (functions, ORs, subqueries), a rewrite (e.g.,
transform OR to UNION, move function off column, use EXISTS instead of IN) may
be preferable. Also weigh write overhead and disk cost of new indexes; for
write-heavy tables, prefer query rewrite and selective indexing.

Decision factors: frequency of query, table size, write QPS, index cardinality,
and coverage benefits.

---

#### 7. What are common optimizer pitfalls in MySQL and how do you mitigate them?

Answer: Pitfalls: stale stats, data skew, function-wrapped indexed columns,
implicit type conversion, poor join order due to bad estimates, oversized
per-connection buffers causing memory exhaustion, and relying on query cache.
Mitigations: ANALYZE TABLE, create histograms, rewrite predicates, ensure
matching column types, use EXPLAIN FORMAT=JSON and EXPLAIN ANALYZE, tune buffer
sizes conservatively, and keep schema and indexing aligned with access patterns.

---

#### 8. Explain EXPLAIN ANALYZE and how you use it in tuning.

Answer: EXPLAIN ANALYZE actually executes the query and returns the execution
plan with real-time row counts and timing per operator. Use it to compare
optimizer estimates (rows) vs actual rows read — large discrepancies indicate
bad stats or data skew and guide whether to add histograms, rewrite query, or
change indexes. Use on representative parameters and beware of side effects (the
query runs).

Practical tip: run on a copy/test dataset or in read-only contexts for
non-idempotent queries.

---

#### 9. How do joins get ordered and what controls join algorithm selection in MySQL?

Answer: The optimizer considers join order permutations based on cost estimates
and tries different join algorithms: nested loop (default with indexes), block
nested loop, hash join (MySQL 8.0.18+ supports semi-hash join for certain
cases), and sort-merge for some engines. Join order is chosen to minimize
intermediate result size — optimizer pushes more selective predicates earlier.
You can influence order with STRAIGHT_JOIN or optimizer_switch hints, but prefer
fixing statistics or indexes so the optimizer chooses correctly.

Why this matters: a bad join order can explode intermediate rows and drastically
increase I/O.

---

#### 10. What are covering indexes and when would you use them?

Answer: A covering index contains all columns needed by a query (WHERE, JOIN,
SELECT, ORDER BY), letting MySQL satisfy the query from the index without
reading the clustered data rows. Use them for high-read, frequent queries to
reduce I/O and avoid lookups. Trade-offs: larger index size and write cost.
Design by adding needed non-key columns as part of the index (in MySQL, include
them in the index key).

Example: SELECT id, status FROM orders WHERE user_id = ? and status = ? →
index(user_id, status, id).

---

#### 11. How do prepared statements, plan caching, and parameter sniffing affect performance in MySQL?

Answer: Prepared statements cache parsed statements and sometimes execution
plans; reuse reduces parsing/optimization overhead. However, parameter
sensitivity (different parameter distributions) can make a cached plan
suboptimal for some parameters. MySQL’s optimizer typically recalculates for
each execution context, but prepared statement plan caching can still surface
parameter-dependent performance issues. Mitigation: use bind variable patterns
that reflect typical load, collect good statistics, or in rare cases re-prepare
with specific hints for pathological parameters.

Mentioning this demonstrates knowledge of dynamic workloads and plan stability.

---

#### 12. Describe a step-by-step approach you would use to optimize a specific slow query in production.

Answer:

1. Reproduce with representative parameters and measure baseline latency and
   resource usage.
2. Run EXPLAIN and EXPLAIN ANALYZE to see chosen plan and actual row
   counts/timings.
3. Check indexes (SHOW INDEX FROM), column types, and histogram/statistics
   freshness (ANALYZE TABLE).
4. Identify hot spots: full table scans, filesorts, temporary tables, large row
   estimates.
5. Try non-invasive rewrites: make predicates sargable, replace IN with EXISTS
   if correlated, reduce selected columns.
6. Add or change indexes (composite or covering) informed by EXPLAIN; avoid
   indexing ad-hoc — consider write cost.
7. Validate improvement with EXPLAIN ANALYZE and measure wall time under
   representative load.
8. If needed, consider schema changes (partitioning, denormalization), caching
   layers, or read-replicas.
9. Monitor over time and rollback if regressions occur.

Why it’s strong interview material: shows methodical, measurement-driven
troubleshooting.
