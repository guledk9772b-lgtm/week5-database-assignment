# PostgreSQL Performance, Transactions & PgBouncer

## Project Overview

This practical demonstrates PostgreSQL performance optimization, transaction isolation, and connection pooling.

The project involves creating a large dataset, analyzing a slow query using `EXPLAIN ANALYZE`, optimizing the query with a targeted index, observing transaction isolation behavior, and configuring PgBouncer for high connection loads.

## Objectives

* Create and populate a 2-million-row `orders` table.
* Analyze query performance using `EXPLAIN ANALYZE`.
* Identify a sequential scan on a large table.
* Create a targeted partial composite index.
* Compare query performance before and after optimization.
* Observe transaction visibility using PostgreSQL isolation levels.
* Install and configure PgBouncer.
* Use transaction pooling to manage database connections efficiently.

## Technologies Used

* PostgreSQL
* SQL
* psql
* PgBouncer
* Linux/Ubuntu

## Step 1: Generate a Big Table

Created an `orders` table containing:

* `id`
* `customer_id`
* `amount`
* `status`
* `created_at`

The table was populated with **2,000,000 records** using `generate_series()` and randomized data.

The table was then analyzed using:

```sql
ANALYZE orders;
```

## Step 2: Measure the Slow Query

Used `EXPLAIN (ANALYZE, BUFFERS)` to examine a query that calculates the total pending order amount for each customer over the last 30 days.

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT customer_id, SUM(amount)
FROM orders
WHERE status = 'pending'
  AND created_at > now() - interval '30 days'
GROUP BY customer_id
ORDER BY SUM(amount) DESC
LIMIT 10;
```

The execution plan was checked for a sequential scan and the execution time was recorded.

## Step 3: Optimize the Query

Created a partial composite index:

```sql
CREATE INDEX idx_pending_recent
ON orders (created_at DESC, customer_id)
WHERE status = 'pending';
```

The same query was executed again with `EXPLAIN ANALYZE` to compare the performance before and after adding the index.

### Performance Comparison

| Test         | Scan Type       | Execution Time |
| ------------ | --------------- | -------------- |
| Before Index | Sequential Scan | __________     |
| After Index  | __________      | __________     |

The results can be filled in using the actual output from PostgreSQL.

## Step 4: Transaction Isolation

Two PostgreSQL sessions were used to observe how transactions see changes made by other transactions.

### READ COMMITTED

The default isolation level was tested by:

```sql
BEGIN;

SELECT amount
FROM orders
WHERE id = 1;
```

A second session updated the same record and committed the change.

The first session then queried the record again to observe the newly committed value.

### REPEATABLE READ

The following transaction was also tested:

```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

SELECT amount
FROM orders
WHERE id = 1;
```

REPEATABLE READ maintains a consistent snapshot during the transaction, so changes committed by another transaction after the snapshot was established are not normally visible to subsequent reads in that transaction.

## Step 5: PgBouncer

PgBouncer was installed to provide connection pooling:

```bash
sudo apt update
sudo apt install -y pgbouncer
```

PgBouncer was configured with transaction pooling:

```ini
[databases]
bootcamp = host=127.0.0.1 port=5432 dbname=bootcamp

[pgbouncer]
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 20
listen_port = 6432
```

PgBouncer was restarted using:

```bash
sudo systemctl restart pgbouncer
```

The database was then accessed through PgBouncer:

```bash
psql -h 127.0.0.1 -p 6432 -U postgres bootcamp
```

## Key Concepts Learned

### Table Bloat

Table bloat occurs when dead row versions from updates and deletes remain in a PostgreSQL table. `VACUUM` helps reclaim space occupied by these dead rows.

### EXPLAIN ANALYZE

`EXPLAIN ANALYZE` executes a query and provides information about its actual execution plan, timing, and resource usage.

### Indexes

Indexes can improve query performance by allowing PostgreSQL to locate relevant rows without scanning the entire table.

### MVCC

PostgreSQL uses Multi-Version Concurrency Control (MVCC) to maintain multiple row versions. This allows readers to see a consistent snapshot while other transactions modify data.

### Transactions

Transactions group multiple SQL statements into a single unit of work. They provide atomicity, meaning changes can either all be committed or rolled back.

### PgBouncer

PgBouncer is a lightweight PostgreSQL connection pooler. Transaction pooling allows many application connections to share a smaller number of PostgreSQL server connections.

## Conclusion

This practical demonstrated how PostgreSQL handles large datasets, query optimization, concurrent transactions, and high connection loads.

The main performance improvement was achieved by identifying the expensive query plan and creating a targeted partial composite index. Transaction isolation demonstrated how PostgreSQL controls data visibility, while PgBouncer provided connection pooling for more efficient database connection management.

## Author

**Name:** Guled Kusow
**Project:** PostgreSQL Performance, Transactions & PgBouncer
