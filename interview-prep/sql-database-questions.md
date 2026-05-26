# SQL & Database Interview Questions & Answers

Covers RDBMS fundamentals, SQL queries, joins, indexes, transactions, normalization, and common interview-style query problems.

---

## Table of Contents
1. [Database Fundamentals](#1-database-fundamentals)
2. [Keys & Constraints](#2-keys--constraints)
3. [Normalization](#3-normalization)
4. [Joins](#4-joins)
5. [Aggregations & Grouping](#5-aggregations--grouping)
6. [Subqueries & CTEs](#6-subqueries--ctes)
7. [Window Functions](#7-window-functions)
8. [Indexes](#8-indexes)
9. [Transactions & Concurrency](#9-transactions--concurrency)
10. [Performance & Tuning](#10-performance--tuning)
11. [Classic Query Problems](#11-classic-query-problems)

---

## 1. Database Fundamentals

### Q1. What is a database?
An organized collection of structured data, managed by a Database Management System (DBMS) for storage, retrieval, and modification.

### Q2. DBMS vs RDBMS?
- **DBMS** – generic data management (file systems, hierarchical, network).
- **RDBMS** – stores data in *relations* (tables) with rows and columns; enforces ACID, supports SQL, foreign keys, joins (MySQL, PostgreSQL, Oracle, SQL Server).

### Q3. SQL vs NoSQL?
| Aspect | SQL (RDBMS) | NoSQL |
|--------|-------------|-------|
| Schema | Fixed | Flexible/dynamic |
| Data model | Tables/rows | Document, key-value, column, graph |
| Transactions | Strong ACID | Often eventual consistency |
| Scaling | Vertical (mostly) | Horizontal |
| Best for | Complex relations, transactions | Large-scale, unstructured/semi-structured |

### Q4. DDL, DML, DCL, TCL?
- **DDL** (Data Definition) – `CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME`.
- **DML** (Data Manipulation) – `SELECT`, `INSERT`, `UPDATE`, `DELETE`.
- **DCL** (Data Control) – `GRANT`, `REVOKE`.
- **TCL** (Transaction Control) – `COMMIT`, `ROLLBACK`, `SAVEPOINT`.

### Q5. `DELETE` vs `TRUNCATE` vs `DROP`?
| Aspect | DELETE | TRUNCATE | DROP |
|--------|--------|----------|------|
| Type | DML | DDL | DDL |
| Removes | Rows (with WHERE possible) | All rows | Table + structure |
| Rollback | Yes (in transaction) | Usually no (auto-commit) | No |
| Triggers fired | Yes | No | No |
| Speed | Slow (row-by-row) | Fast | Fast |
| Auto-increment | Not reset | Reset | N/A |

### Q6. `WHERE` vs `HAVING`?
- **WHERE** – filters rows *before* grouping. Cannot use aggregates.
- **HAVING** – filters groups *after* `GROUP BY`. Can use aggregates.

```sql
SELECT department, COUNT(*) AS cnt
FROM employees
WHERE active = 1
GROUP BY department
HAVING COUNT(*) > 10;
```

### Q7. `UNION` vs `UNION ALL`?
- **UNION** – combines results, removes duplicates (slower).
- **UNION ALL** – combines results, keeps duplicates (faster).

### Q8. `IN` vs `EXISTS`?
- **IN** – good for small static lists.
- **EXISTS** – good for correlated subqueries; stops at first match.
For large datasets, `EXISTS` often performs better.

---

## 2. Keys & Constraints

### Q9. Types of keys?
- **Primary Key** – uniquely identifies a row (NOT NULL + UNIQUE, only one per table).
- **Candidate Key** – any column(s) eligible to be a primary key.
- **Super Key** – any superset of a candidate key.
- **Foreign Key** – references the primary key of another table.
- **Composite Key** – primary key made of two or more columns.
- **Unique Key** – ensures unique values; allows one NULL.

### Q10. Constraints?
- `NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, `DEFAULT`.

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    age INT CHECK (age >= 18),
    status VARCHAR(20) DEFAULT 'ACTIVE',
    dept_id BIGINT,
    FOREIGN KEY (dept_id) REFERENCES departments(id)
);
```

### Q11. ON DELETE / ON UPDATE actions?
- `CASCADE` – propagate the delete/update.
- `SET NULL` – set FK to NULL.
- `SET DEFAULT` – set FK to default.
- `RESTRICT` / `NO ACTION` – prevent the operation if dependents exist.

---

## 3. Normalization

### Q12. What is normalization?
Process of organizing data to reduce redundancy and improve integrity by splitting large tables into smaller related ones.

### Q13. Normal forms (1NF–BCNF)?
- **1NF** – atomic columns; no repeating groups.
- **2NF** – 1NF + no partial dependency on a composite key.
- **3NF** – 2NF + no transitive dependency on the primary key.
- **BCNF** – 3NF + every determinant is a candidate key.

### Q14. What is denormalization?
Intentionally introducing redundancy to improve read performance (e.g., warehouses, reporting). Trade write performance and consistency for fewer joins.

---

## 4. Joins

### Q15. Types of joins?
- **INNER JOIN** – rows matching in both tables.
- **LEFT (OUTER) JOIN** – all left rows + matches from right (NULL where no match).
- **RIGHT (OUTER) JOIN** – mirror of LEFT.
- **FULL (OUTER) JOIN** – all rows from both, NULL where no match.
- **CROSS JOIN** – Cartesian product.
- **SELF JOIN** – table joined with itself (e.g., employee–manager).

### Q16. INNER JOIN example
```sql
SELECT e.name, d.name AS department
FROM employees e
INNER JOIN departments d ON e.dept_id = d.id;
```

### Q17. SELF JOIN – find employees with their manager
```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

### Q18. Find employees with no department
```sql
SELECT e.* FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id
WHERE d.id IS NULL;
```

---

## 5. Aggregations & Grouping

### Q19. Aggregate functions?
`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, `GROUP_CONCAT` / `STRING_AGG`.

### Q20. `COUNT(*)` vs `COUNT(col)` vs `COUNT(DISTINCT col)`?
- `COUNT(*)` – all rows including NULLs.
- `COUNT(col)` – non-NULL values in `col`.
- `COUNT(DISTINCT col)` – unique non-NULL values.

### Q21. Department-wise employee count
```sql
SELECT department, COUNT(*) AS emp_count
FROM employees
GROUP BY department
ORDER BY emp_count DESC;
```

### Q22. Average salary per department, only departments with > 5 employees
```sql
SELECT department, AVG(salary) AS avg_sal
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
```

---

## 6. Subqueries & CTEs

### Q23. What is a subquery?
A query nested inside another. Can be **scalar**, **row**, **table**, or **correlated**.

### Q24. Find employees earning more than the company average
```sql
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

### Q25. Find departments having at least one employee earning > 100k
```sql
SELECT name FROM departments d
WHERE EXISTS (
    SELECT 1 FROM employees e
    WHERE e.dept_id = d.id AND e.salary > 100000
);
```

### Q26. What is a CTE (Common Table Expression)?
A named temporary result set defined with `WITH`. Improves readability; supports recursion.

```sql
WITH high_earners AS (
    SELECT * FROM employees WHERE salary > 100000
)
SELECT department, COUNT(*) FROM high_earners GROUP BY department;
```

### Q27. Recursive CTE – org hierarchy
```sql
WITH RECURSIVE org AS (
    SELECT id, name, manager_id, 1 AS level
    FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.id, e.name, e.manager_id, o.level + 1
    FROM employees e JOIN org o ON e.manager_id = o.id
)
SELECT * FROM org;
```

---

## 7. Window Functions

### Q28. What are window functions?
Operate over a set of rows ("window") relative to the current row, without collapsing them like `GROUP BY`.

### Q29. Common window functions
- `ROW_NUMBER()` – unique sequential rank, no ties.
- `RANK()` – ties share rank, leaves gaps.
- `DENSE_RANK()` – ties share rank, no gaps.
- `LAG()` / `LEAD()` – previous/next row's value.
- `NTILE(n)` – split into n buckets.
- `SUM()/AVG() OVER(...)` – running totals.

### Q30. Top 3 paid employees per department
```sql
SELECT *
FROM (
    SELECT e.*,
           DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk
    FROM employees e
) ranked
WHERE rnk <= 3;
```

### Q31. Running total of orders per customer
```sql
SELECT customer_id, order_date, amount,
       SUM(amount) OVER (PARTITION BY customer_id ORDER BY order_date) AS running_total
FROM orders;
```

### Q32. Compare each row to previous (LAG)
```sql
SELECT order_date, amount,
       LAG(amount) OVER (ORDER BY order_date) AS prev_amount,
       amount - LAG(amount) OVER (ORDER BY order_date) AS diff
FROM orders;
```

---

## 8. Indexes

### Q33. What is an index?
A data structure (usually B-Tree) that speeds up reads at the cost of storage and write performance. Equivalent to a book's index – avoids scanning every row.

### Q34. Types of indexes?
- **Primary** – on the primary key (clustered in many DBs).
- **Unique** – enforces uniqueness.
- **Composite** – on multiple columns (order matters – leftmost prefix rule).
- **Clustered** – data is physically ordered by the index (only one per table).
- **Non-clustered** – separate structure pointing to rows.
- **Full-text**, **GIN/GiST** (Postgres), **Hash**, **Bitmap** (Oracle).

### Q35. When NOT to index?
- Small tables.
- Columns with very low cardinality (few distinct values, e.g., boolean).
- Columns with frequent writes and few reads.
- When existing indexes already cover the query.

### Q36. What is a covering index?
An index that contains all columns needed by a query, so the DB doesn't need to look up the table at all.

### Q37. How to find slow queries / missing indexes?
- `EXPLAIN` / `EXPLAIN ANALYZE` (Postgres/MySQL).
- DB-specific monitoring (`pg_stat_statements`, MySQL slow query log).
- Look for full table scans, large estimated row counts, and join steps without index use.

---

## 9. Transactions & Concurrency

### Q38. ACID properties?
- **Atomicity** – all-or-nothing.
- **Consistency** – DB moves from one valid state to another.
- **Isolation** – concurrent transactions don't interfere.
- **Durability** – once committed, persists even after crash.

### Q39. Isolation levels?
| Level | Dirty Read | Non-repeatable Read | Phantom Read |
|-------|-----------|---------------------|--------------|
| READ UNCOMMITTED | Yes | Yes | Yes |
| READ COMMITTED | No | Yes | Yes |
| REPEATABLE READ | No | No | Yes (most DBs) |
| SERIALIZABLE | No | No | No |

### Q40. Optimistic vs pessimistic locking?
- **Optimistic** – assumes no conflict; check version on update (e.g., `WHERE version = ?`).
- **Pessimistic** – locks rows up front (`SELECT … FOR UPDATE`).

### Q41. What is a deadlock?
Two transactions waiting on each other's locks indefinitely. DB engines detect and abort one. Avoid by accessing tables in a consistent order, keeping transactions short, and using lower isolation when possible.

---

## 10. Performance & Tuning

### Q42. Query optimization tips?
- Avoid `SELECT *` – fetch only needed columns.
- Use proper indexes; check execution plans.
- Prefer `EXISTS` over `IN` for large subqueries.
- Avoid functions on indexed columns in WHERE (`WHERE YEAR(date) = 2025` blocks index).
- Use pagination (`LIMIT`/`OFFSET`) for large result sets.
- Batch inserts/updates instead of row-by-row.
- Beware of N+1 queries from ORMs.

### Q43. Sargable vs non-sargable queries?
**Sargable** queries can use indexes. Non-sargable queries cannot.
- Bad: `WHERE UPPER(name) = 'JOHN'`.
- Good: store names normalized OR use a functional index.

### Q44. View vs Materialized view?
- **View** – virtual, query is re-run on every access.
- **Materialized view** – physical, refreshed periodically. Faster reads, stale data possible.

### Q45. What is partitioning?
Splitting a large table into smaller physical pieces by a key (range/list/hash). Improves query and maintenance performance.

### Q46. Sharding?
Horizontal splitting across multiple servers/DBs. Used at scale; introduces complexity (cross-shard queries, rebalancing).

---

## 11. Classic Query Problems

### Q47. Find the 2nd highest salary
```sql
-- Approach 1: subquery
SELECT MAX(salary) AS second_highest
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);

-- Approach 2: window function (also handles ties)
SELECT salary FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) t WHERE rnk = 2;
```

### Q48. Find the Nth highest salary
```sql
SELECT salary FROM (
    SELECT DISTINCT salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) t WHERE rnk = N;
```

### Q49. Find duplicate emails
```sql
SELECT email, COUNT(*) AS cnt
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

### Q50. Delete duplicate rows (keep one)
```sql
-- MySQL/Postgres using window function + CTE
WITH ranked AS (
    SELECT id,
           ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) AS rn
    FROM users
)
DELETE FROM users WHERE id IN (SELECT id FROM ranked WHERE rn > 1);
```

### Q51. Customers who placed no orders
```sql
SELECT c.* FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.id IS NULL;
```

### Q52. Department with highest average salary
```sql
SELECT department, AVG(salary) AS avg_sal
FROM employees
GROUP BY department
ORDER BY avg_sal DESC
LIMIT 1;
```

### Q53. Employees earning more than their manager
```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

### Q54. Consecutive days with sales (gaps and islands)
```sql
SELECT MIN(sale_date) AS start_date, MAX(sale_date) AS end_date
FROM (
    SELECT sale_date,
           DATE_SUB(sale_date, INTERVAL ROW_NUMBER() OVER (ORDER BY sale_date) DAY) AS grp
    FROM daily_sales
) t
GROUP BY grp;
```

### Q55. Pivot rows to columns (sum of sales by quarter)
```sql
SELECT product,
       SUM(CASE WHEN quarter = 'Q1' THEN amount ELSE 0 END) AS Q1,
       SUM(CASE WHEN quarter = 'Q2' THEN amount ELSE 0 END) AS Q2,
       SUM(CASE WHEN quarter = 'Q3' THEN amount ELSE 0 END) AS Q3,
       SUM(CASE WHEN quarter = 'Q4' THEN amount ELSE 0 END) AS Q4
FROM sales
GROUP BY product;
```

### Q56. Find top 5 customers by total spend
```sql
SELECT c.id, c.name, SUM(o.amount) AS total
FROM customers c
JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name
ORDER BY total DESC
LIMIT 5;
```

### Q57. Median salary
```sql
-- Postgres
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) AS median FROM employees;

-- Generic
SELECT AVG(salary) AS median FROM (
    SELECT salary,
           ROW_NUMBER() OVER (ORDER BY salary) AS rn,
           COUNT(*) OVER () AS cnt
    FROM employees
) t WHERE rn IN ((cnt + 1) / 2, (cnt + 2) / 2);
```

### Q58. Find customers who bought all products
```sql
SELECT c.id, c.name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM products p
    WHERE NOT EXISTS (
        SELECT 1 FROM orders o
        WHERE o.customer_id = c.id AND o.product_id = p.id
    )
);
```
