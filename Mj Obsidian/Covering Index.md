

## **What is a Covering Index?**

A **covering index** is an index that contains **all the columns needed** for a query, so PostgreSQL can answer the query **just by reading the index**, without having to fetch data from the actual table.

This is useful because:

- Normally, PostgreSQL uses an index to find **row locations** (pointers) and then goes to the table to get the remaining columns.
    
- If the index already has all the columns your query needs, PostgreSQL skips the table lookup (called a **heap fetch**), making the query faster.
    

---

## **How to Create a Covering Index in PostgreSQL**

In PostgreSQL, you create a covering index using the `INCLUDE` clause.

**Example:**

```sql
-- Table
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name TEXT,
    department TEXT,
    salary NUMERIC
);

-- Covering index
CREATE INDEX idx_department_salary
ON employees (department)
INCLUDE (salary);
```

Here:

- `department` is the **indexed column** used for filtering/searching.
    
- `salary` is an **included column** — stored in the index but not used for sorting/searching.
    
- If your query needs `department` and `salary`, it can be answered entirely from the index.
    

---

## **Why It’s Faster**

Without a covering index:

1. PostgreSQL finds matching rows in the index.
    
2. It **goes to the table** for every matching row to get extra columns (heap fetch).
    
3. That’s extra disk I/O.
    

With a covering index:

- All needed columns are **already in the index**.
    
- No heap fetch is required — query reads only from the index.
    
- This is called an **Index-Only Scan**.
    

---

## **Example Query Optimization**

Without covering index:

```sql
EXPLAIN ANALYZE
SELECT department, salary
FROM employees
WHERE department = 'IT';
```

Output might show:

```
Index Scan using idx_department on employees
Heap Fetches: 1000
```

(Meaning it had to go to the table.)

With covering index:

```sql
CREATE INDEX idx_department_salary_cover
ON employees (department) INCLUDE (salary);

EXPLAIN ANALYZE
SELECT department, salary
FROM employees
WHERE department = 'IT';
```

Output:

```
Index Only Scan using idx_department_salary_cover on employees
Heap Fetches: 0
```

(Meaning **no table access** — much faster.)

---

## **When to Use Covering Indexes**

✅ Good for **frequent queries** where you filter on one column but also need extra columns in the result.  
✅ Reduces **disk I/O** and speeds up read-heavy workloads.  
❌ Avoid adding **too many included columns** — it makes the index large and can slow down writes (INSERT/UPDATE).

---

To know **which columns to include in a covering index** in PostgreSQL, you need to think about:

---

## **1️⃣ Identify frequently used queries**

Check which queries are most common and performance-critical.  
Use PostgreSQL’s **`pg_stat_statements`** to see slow or frequently executed queries:

```sql
SELECT query, calls, total_exec_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC;
```

---

## **2️⃣ Look at `WHERE`, `JOIN`, and `ORDER BY` columns**

Covering indexes should **start with columns in your `WHERE` or `JOIN` conditions** (for filtering), because the index is searched in order.

Example:

```sql
SELECT name, salary
FROM employees
WHERE department_id = 5
ORDER BY salary;
```

Here:

- `department_id` → used in `WHERE`
    
- `salary` → used in `ORDER BY`
    

A good **covering index**:

```sql
CREATE INDEX idx_emp_cover
ON employees(department_id, salary)
INCLUDE (name);
```

`INCLUDE (name)` means PostgreSQL will store `name` inside the index so it can return results **without reading the table** (index-only scan).

---

## **3️⃣ Use `EXPLAIN (ANALYZE)` to verify**

Before:

```sql
EXPLAIN (ANALYZE)
SELECT name, salary
FROM employees
WHERE department_id = 5
ORDER BY salary;
```

- If you see `Seq Scan` → it’s reading the whole table.
    
- After adding the covering index, you should see `Index Only Scan`.
    

---

## **4️⃣ Rule of Thumb for Choosing Columns**

✅ **Filtering first** – columns in `WHERE` clause.  
✅ **Sorting second** – columns in `ORDER BY`.  
✅ **Extra columns** – put them in `INCLUDE()` if they are only needed in `SELECT` but not in filtering/sorting.  
✅ **Keep it small** – don’t include too many columns, or the index will become big and slow to update.

---

Let’s go step-by-step with a **covering index** example in PostgreSQL.

---

## **Scenario**

You have an `employees` table:

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name TEXT,
    department TEXT,
    salary NUMERIC
);
```

---

## **Query to Optimize**

Suppose you often run this query:

```sql
SELECT name, salary 
FROM employees
WHERE department = 'IT';
```

---

## **Step 1 — Identify columns for the covering index**

For a **covering index**:

1. **Index key columns** → used in `WHERE` clause (filters).
    
    - Here: `department` is the filter.
        
2. **Included columns** → columns that are selected but not filtered, so PostgreSQL can fetch them **directly from the index** without going to the main table (avoiding a heap lookup).
    
    - Here: `name` and `salary`.
        

---

## **Step 2 — Create a covering index**

```sql
CREATE INDEX idx_department_cover
ON employees (department) INCLUDE (name, salary);
```

---

## **Step 3 — How it works**

- Normally: PostgreSQL uses the index to find rows by `department` but still goes to the **table** to fetch `name` and `salary`.
    
- With covering index:
    
    - The index already **stores `department`, `name`, and `salary`**.
        
    - PostgreSQL **skips** going to the table.
        
    - This is called an **Index-Only Scan**.
        

---

## **Step 4 — Verify with `EXPLAIN ANALYZE`**

```sql
EXPLAIN ANALYZE
SELECT name, salary 
FROM employees
WHERE department = 'IT';
```

If the covering index is used, you’ll see:

```
Index Only Scan using idx_department_cover ...
```

---

✅ **Rule of Thumb for Choosing Columns for a Covering Index**

- **WHERE clause** columns → in the index key.
    
- **SELECT** columns (not in WHERE) → in the INCLUDE clause.
    
- Avoid including large text/JSON columns unless absolutely necessary, as they increase index size.
    
