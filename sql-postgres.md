# SQL and PostgreSQL: Interview Cheat Sheet

Only the most important questions. Say each answer out loud in your own voice.

In the examples I use two tables: `users (id, name, email)` and `orders (id, user_id, total)`.

---

## 1. What is the difference between SQL and NoSQL databases?

- **SQL** (PostgreSQL, MySQL): data is in **tables with rows and columns**, with a **fixed schema** and **relations** between tables. It is strong in **joins** and **ACID transactions**. It is a good choice for related data like users, orders and payments.
- **NoSQL** (MongoDB, Redis, DynamoDB): data is stored in other forms like **documents or key-value**. The schema is **flexible**, and it is usually easier to **scale horizontally**. It is a good choice for fast-changing or unstructured data, caching and huge volumes. Joins are weaker.

Neither one is "better". I choose based on the data and the access pattern. Many apps use both, like PostgreSQL for main data and Redis for cache.

---

## 2. What are the types of JOIN?

- **INNER JOIN:** only rows that **match in both** tables.
- **LEFT JOIN:** **all rows from the left** table, plus matches from the right. If there is no match, the right side is `NULL`.
- **RIGHT JOIN:** the opposite of left join.
- **FULL JOIN:** all rows from **both** tables.

```sql
-- Users who have orders
SELECT u.name, o.total
FROM users u
INNER JOIN orders o ON o.user_id = u.id;

-- All users, even those with no orders
SELECT u.name, o.id AS order_id
FROM users u
LEFT JOIN orders o ON o.user_id = u.id;
```

A useful trick: users with **no orders** are `LEFT JOIN ... WHERE o.id IS NULL`.

---

## 3. What are primary key, foreign key and constraints?

- **Primary key:** uniquely identifies each row. It cannot be `NULL` or duplicate. Postgres makes an index for it automatically.
- **Foreign key:** a column that points to the primary key of another table. It keeps the data **consistent**, so I cannot have an order for a user that does not exist.
- Other constraints: `UNIQUE`, `NOT NULL`, `CHECK`, `DEFAULT`.

```sql
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  total NUMERIC(10, 2) CHECK (total >= 0)
);
```

---

## 4. What is an index? When should I use it?

An index is a **separate data structure (usually a B-tree)** that lets the database find rows **quickly**, without scanning the whole table. It is like the index at the back of a book.

```sql
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

**Use an index on** columns used often in `WHERE`, `JOIN`, and `ORDER BY`. Foreign key columns are a good example. Postgres does **not** create an index for foreign keys automatically.

**The cost:** indexes use disk space and make `INSERT`, `UPDATE` and `DELETE` **slower**. So I do not add them everywhere.

**Composite index:** an index on many columns, like `(user_id, created_at)`. The order matters. It helps queries that use the **first column** (leftmost rule).

**To check a query:** `EXPLAIN ANALYZE SELECT ...` shows if it uses an **Index Scan** (good) or a **Seq Scan** (full table scan, slow on big tables).

---

## 5. What is normalization?

It is organizing tables to **remove duplicate data** and avoid update problems.

- **1NF:** each column holds **one value** (no lists in a cell), and each row is unique.
- **2NF:** 1NF, and every column depends on the **whole primary key**.
- **3NF:** 2NF, and columns depend **only on the key**, not on other non-key columns.

Example: instead of saving the user's name and email in every order row, I save only `user_id` in `orders` and keep the details in `users`.

**Denormalization** means adding some duplicate data on purpose to make reads faster. I do it only when performance needs it.

---

## 6. What are ACID properties and transactions?

A **transaction** is a group of queries that run **as one unit**: all succeed or all fail.

- **Atomicity:** all or nothing.
- **Consistency:** the data always stays valid (constraints are respected).
- **Isolation:** transactions running at the same time do not break each other.
- **Durability:** once committed, the data is saved even if the server crashes.

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;      -- or ROLLBACK if something failed
```

**Isolation levels** control what a transaction can see from others. The Postgres default is **Read Committed**. The problems they prevent: **dirty read** (reading uncommitted data), **non-repeatable read** and **phantom read**.

---

## 7. What is the difference between `WHERE` and `HAVING`? What is `GROUP BY`?

- `GROUP BY` groups rows so I can use aggregate functions like `COUNT`, `SUM`, `AVG`, `MAX`.
- `WHERE` filters **rows before** grouping.
- `HAVING` filters **groups after** grouping.

```sql
-- Users who have more than 3 orders
SELECT user_id, COUNT(*) AS order_count, SUM(total) AS spent
FROM orders
WHERE total > 0
GROUP BY user_id
HAVING COUNT(*) > 3
ORDER BY spent DESC;
```

---

## 8. Common SQL questions asked in interviews

**Second highest salary:**

```sql
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;

-- or
SELECT MAX(salary) FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

**Find duplicate emails:**

```sql
SELECT email, COUNT(*)
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

**Top 3 customers by total spending:**

```sql
SELECT u.name, SUM(o.total) AS spent
FROM users u
JOIN orders o ON o.user_id = u.id
GROUP BY u.id, u.name
ORDER BY spent DESC
LIMIT 3;
```

---

## 9. What is the N+1 query problem?

It happens when I run **1 query to get a list**, and then **N more queries** (one per item) to get related data. For example: 1 query for 100 users, then 100 queries for each user's orders. That is 101 queries.

**Fix:** get all the data in one go with a **JOIN**, or one batched query: `WHERE user_id IN (...)`. With an ORM, I use **eager loading** (like `include` or `join`).

---

## 10. How do I optimize a slow query?

1. Run **`EXPLAIN ANALYZE`** to see what is slow.
2. Add the right **index** (on `WHERE`, `JOIN` and `ORDER BY` columns).
3. **Select only the columns I need**, not `SELECT *`.
4. Avoid the **N+1** problem.
5. Use **pagination** (`LIMIT`).
6. Cache the result of heavy and repeated queries (Redis).
7. Use a **connection pool** so I do not open a new connection each time.

**Pagination:** `OFFSET` gets slower for large page numbers, because the database still has to skip all the previous rows. A better way is **keyset pagination**:

```sql
SELECT * FROM orders
WHERE id > 1000        -- the last id from the previous page
ORDER BY id
LIMIT 20;
```

---

## 11. How do I prevent SQL injection?

SQL injection is when an attacker puts SQL code in the input, and my query runs it. The fix is to **never build queries with string concatenation**. I use **parameterized queries**, so the data is never treated as code.

```js
// Bad
db.query(`SELECT * FROM users WHERE email = '${email}'`);

// Good (node-postgres)
db.query('SELECT * FROM users WHERE email = $1', [email]);
```

---

## 12. What is special in PostgreSQL?

- **MVCC:** every transaction sees its own **snapshot** of the data, so **readers do not block writers**. Old row versions are cleaned up by **VACUUM**.
- **JSONB:** I can store and query JSON data, and add an index on it (GIN). It gives me some NoSQL flexibility inside SQL.
- **`RETURNING`:** get the saved row back right away: `INSERT INTO users(name) VALUES ('Aman') RETURNING id;`
- Also has: arrays, full-text search, CTEs (`WITH`), window functions, and extensions like PostGIS.

---

## 13. DELETE vs TRUNCATE vs DROP

- **`DELETE`** removes **rows**, can use `WHERE`, and can be rolled back.
- **`TRUNCATE`** removes **all rows** very fast. No `WHERE`. In Postgres it can be rolled back inside a transaction.
- **`DROP`** removes the **whole table** (structure and data).

---

## Last-Minute Revision

- SQL: structured, joins, ACID. NoSQL: flexible, easy to scale out.
- INNER: matches only. LEFT: all from the left. FULL: everything.
- Index: faster reads, slower writes. Check with `EXPLAIN ANALYZE`. Foreign keys are not indexed automatically in Postgres.
- Normalization: no duplicate data. Denormalize only for speed.
- ACID: Atomicity, Consistency, Isolation, Durability.
- `WHERE` filters rows. `HAVING` filters groups.
- N+1: fix with a JOIN or `IN (...)`.
- Prefer keyset pagination over big `OFFSET`.
- Parameterized queries stop SQL injection.
- Postgres: MVCC, JSONB, `RETURNING`.