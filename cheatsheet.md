# SQL Cheat Sheet

## 1. Basic Query Structure

```sql
SELECT column1, column2, …
FROM main_table
[JOIN other_table ON join_condition]
[WHERE filter]
[GROUP BY column_list]
[HAVING group_filter]
[ORDER BY column ASC|DESC]
[LIMIT n OFFSET m];
```

## 2. Set Operations
- UNION / UNION ALL: combine result sets (UNION removes duplicates)
- INTERSECT: intersection of result sets
- EXCEPT (or MINUS): difference between result sets

```sql
SELECT col1, col2
FROM tableA
UNION      -- or UNION ALL / INTERSECT / EXCEPT
SELECT col3, col4
FROM tableB
ORDER BY col1 DESC
LIMIT 10;
```

## 3. Subqueries
### 3.1. Simple Subquery
```sql
SELECT *
FROM sales_associates
WHERE salary > (
  SELECT AVG(revenue)
  FROM sales_associates
);
```

### 3.2. Correlated Subquery

```sql
SELECT *
FROM employees e
WHERE salary > (
  SELECT AVG(revenue_generated)
  FROM employees dept_e
  WHERE dept_e.department = e.department
);

```

## 4. Window Functions & Ranking
### 4.1. DENSE_RANK() vs RANK()
- RANK() skips positions after ties (so after two “1”s you get “3”).
- DENSE_RANK() does not skip (so after two “1”s you get “2”).

Eg:
```sql
WITH scores AS (
  SELECT UNNEST(ARRAY[90, 90, 88, 88, 81, 80, 80, 79]) AS mark
)
SELECT
  mark,
  RANK() OVER (ORDER BY mark DESC)   AS rank,
  DENSE_RANK() OVER (ORDER BY mark DESC)   AS dense_rank
FROM scores;
```
The result :
| mark | rank | dense_rank |
| ---- | ---- | ----------- |
| 90   | 1    | 1           |
| 90   | 1    | 1           |
| 88   | 3    | 2           |
| 88   | 3    | 2           |
| 81   | 5    | 3           |
| 80   | 6    | 4           |
| 80   | 6    | 4           |
| 79   | 8    | 5           |


### 4.2. Aggregates Over Partitions
When you use a regular aggregate (e.g. SUM(), AVG()) in SQL, you normally get a single result for the entire dataset or one result per GROUP BY bucket. A window function lets you “peek” that same aggregate at every row without collapsing your results.
- PARTITION BY (optional) splits the rows into separate groups, each partition gets its own aggregate calculation.
- ORDER BY (optional, inside the window) defines row order within each partition and enables running or cumulative calculations (e.g. running total).

Important: if you include ORDER BY in your window, the aggregate at each row is computed over all prior rows in its partition (plus any ties), rather than over the entire partition. This is how you get true running sums, moving averages, and other “over-time” insights without losing the context of each individual row.

```sql
SELECT depname, empno, salary, AVG(salary)
OVER (
         PARTITION BY depname
         [ORDER BY salary]
     ) AS avg_salary
FROM empsalary;
```
## 5. Common Table Expressions (CTEs)
### 5.1. Simple CTE
```sql
WITH cte_dept AS (
  SELECT dept_id, dept_name
  FROM Department
  WHERE dept_name IN ('IT', 'FINANCE')
)
SELECT e.emp_id, e.first_name, e.last_name, e.gender
FROM employee e
JOIN cte_dept d USING (dept_id);
```
### 5.2.  Recursive CTE

```sql
WITH RECURSIVE cte_tree AS (
  -- Base query
  SELECT id, parent_id, name
  FROM categories
  WHERE parent_id IS NULL

  UNION ALL

  -- Recursive query
  SELECT c.id, c.parent_id, c.name
  FROM categories c
  JOIN cte_tree ct ON c.parent_id = ct.id
)
SELECT *
FROM cte_tree;
```

## 6. Pivoting (PostgreSQL)
PostgreSQL has no native PIVOT. Use:
- CASE statements
- creetab function (enable extension tablefunc)

```sql
-- CASE example
SELECT
  category,
  SUM(CASE WHEN month = 'Jan' THEN amount ELSE 0 END) AS Jan,
  SUM(CASE WHEN month = 'Feb' THEN amount ELSE 0 END) AS Feb
FROM sales
GROUP BY category;
```

## 7. LATERAL JOIN
Useful for querying hierarchical or related row sets. Here is a quick overview : https://www.cybertec-postgresql.com/en/understanding-lateral-joins-in-postgresql/ 


## 8. NULL Handling
- COALESCE(v1, v2, …): returns the first non-NULL argument
- NULLIF(a, b): returns NULL if a = b; otherwise returns a
```sql
SELECT
  COALESCE(NULL, 'default_value') AS result,
  NULLIF(col1, col2) AS diff_null
FROM my_table;

```
## 9. VIEWS
- VIEW: stores query definition; always returns up-to-date data
- MATERIALIZED VIEW: stores query result; must be manually refreshed

```sql
-- Standard view
CREATE VIEW active_clients AS
SELECT *
FROM clients
WHERE status = 'active';

-- Materialized view
CREATE MATERIALIZED VIEW mv_sales AS
SELECT id, AVG(val), COUNT(*)
FROM random_tab
GROUP BY id;

-- Refreshing materialized view
REFRESH MATERIALIZED VIEW mv_sales;
```

## 10. Data Manipulation (DML)
### 10.1. INSERT 
```sql
INSERT INTO MyTable (col1, col2, …)
VALUES
  (v1a, v2a, …),
  (v1b, v2b, …);
```

### 10.2. UPDATE 
```sql
UPDATE MyTable
SET col1 = value1,
    col2 = value2
WHERE condition;
```

### 10.3. DELETE
```sql
DELETE FROM MyTable
WHERE condition;
```

## 11. Schema Definition (DDL)
### 11.1. CREATE TABLE
```sql
CREATE TABLE IF NOT EXISTS MyTable (
  col1   INT     CONSTRAINT pk_mytable PRIMARY KEY,
  col2   TEXT    DEFAULT 'N/A',
  col3   DATE    NOT NULL
);
```
### 11.2. ALTER TABLE
```sql
-- Add a column
ALTER TABLE MyTable
ADD COLUMN new_col VARCHAR(50) DEFAULT 'X';

-- Drop a column
ALTER TABLE MyTable
DROP COLUMN unwanted_col;

-- Rename the table
ALTER TABLE MyTable
RENAME TO new_table_name;
```

### 11.3. DROP TABLE
```sql
DROP TABLE IF EXISTS MyTable;
```


## 11. Advanced Query Patterns
- Aggregations with GROUP BY + HAVING
- Multiple joins and filters
- Pagination using LIMIT … OFFSET …

```sql
SELECT t1.a, t2.b, SUM(t1.c) AS total
FROM db1.table1 t1
JOIN db2.table2 t2 ON t1.id = t2.ref_id
WHERE t1.flag = TRUE
GROUP BY t1.a, t2.b
HAVING SUM(t1.c) > 100
ORDER BY total DESC
LIMIT 20 OFFSET 10;
```



